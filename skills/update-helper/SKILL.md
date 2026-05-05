---
name: update-helper
version: "4.0"
description: |
  Universal protocol for safely reading, understanding, and patching large source files
  (1,000–30,000+ lines) in any language on Windows/PowerShell environments.
  Use whenever the user wants to: edit, patch, refactor, debug, or understand a large file —
  especially when the file is too big to read at once, when the user says "fix this",
  "add feature X", "why is Y broken", or when a previous patch needs to be verified or rolled back.
  Covers: code comprehension, architecture mapping, data flow tracing, encoding-safe writes
  (Vietnamese/CJK/BOM), 2-tier backup, anchor-based patching, cascade analysis, stale-patch
  detection, proactive bug hunt, multi-agent mid-session onboarding, and JS UI patterns.
  Self-contained — no external reference files needed.
  Original by PitroyTech. Updated v4.0.
---

# update-helper v4.0

Purpose: execute large-file edits with minimum context burn, low corruption risk, and repeatable verification. Self-contained — never needs external files loaded.

Respond in the user's language. Keep code identifiers and terminal commands in English.

---

## 0. Hard Rules

1. Never read a whole large file. Search first, then read bounded ranges (40–160 lines).
2. Before the first write to any target file: detect encoding and create a session backup.
3. Immediately before each write: create/refresh `file.ext.bak2`.
4. Preserve original encoding exactly: UTF-8 BOM stays BOM; UTF-8 no-BOM stays no-BOM.
5. Prefer `apply_patch` for normal repo edits. For encoding-sensitive files, use .NET `ReadAllText/WriteAllText`.
6. After every write: run the syntax check for the file's language (see Section 6).
7. If verification passes: delete `.bak2`. If it fails: restore `.bak2`, fix, verify again.
8. Patch spec from user may be STALE — verify actual current code before applying.
9. Never revert unrelated user changes. Work around them or ask only if blocked.
10. Final response must mention: changed files, verification result, backup state, remaining risk.

---

## 1. Fast Workflow

Execute in this exact order:

1. **Locate anchors** — use `rg` when available; otherwise `Select-String`.
2. **Read bounded range** — 40–160 lines around each anchor.
3. **Structural Scan** (Section 4A) → **Data Flow Trace** (Section 4B) → **Blast Radius** (Section 4C). Mandatory before coding.
4. **Detect encoding** (Section 2).
5. **Create session backup** if not yet created for this target.
6. **Create `.bak2`** immediately before writing.
7. **Patch narrowly**.
8. **Verify** syntax + searched invariants.
9. **Delete `.bak2`** only after verification passes.
10. **Report** concise result (Section 9).

---

## 2. Encoding-Safe Write Pattern

Use for: any file with Vietnamese/CJK/emoji, any UTF-8 BOM file, any large external file, any file with known encoding risk.

```powershell
$path = "C:\path\file.ext"
$b = [System.IO.File]::ReadAllBytes($path)
$hasBom = $b.Length -ge 3 -and $b[0] -eq 0xEF -and $b[1] -eq 0xBB -and $b[2] -eq 0xBF
$content = [System.IO.File]::ReadAllText($path, [System.Text.Encoding]::UTF8)
$nl = if ($content.Contains("`r`n")) { "`r`n" } else { "`n" }

# Modify $content using exact ASCII anchors where possible.
# Never rely on terminal-rendered Vietnamese when console may show mojibake.

$utf8 = [System.Text.UTF8Encoding]::new($hasBom)
[System.IO.File]::WriteAllText($path, $content, $utf8)
```

**Forbidden for encoding-sensitive files:**
```
Get-Content | Set-Content
Set-Content -Encoding UTF8
WriteAllLines after Get-Content
IDE replace tools that may strip/add BOM
```

**Encoding check:**
```powershell
$b = [System.IO.File]::ReadAllBytes($path)
if ($b.Length -ge 3 -and $b[0] -eq 0xEF -and $b[1] -eq 0xBB -and $b[2] -eq 0xBF) { "UTF-8 BOM" }
elseif ($b.Length -ge 2 -and $b[0] -eq 0xFF -and $b[1] -eq 0xFE) { "UTF-16 LE" }
else { "UTF-8 no BOM or ASCII" }
```

---

## 3. Backup Protocol

### Session backup — once per target file per work stream. Never overwrite.
```powershell
$sessionBak = "file.ext.bak.codex-session-YYYYMMDD-topic"
if (-not (Test-Path -LiteralPath $sessionBak)) {
    Copy-Item -LiteralPath "file.ext" -Destination $sessionBak -Force
}
```

### Current-request backup — immediately before every write.
```powershell
Copy-Item -LiteralPath "file.ext" -Destination "file.ext.bak2" -Force
```

### Rollback decision tree
```
Patch failed?
├── Syntax error, just applied  →  restore .bak2, fix the patch, retry
└── Multiple patches broken     →  restore session backup, restart from clean state
```

**On failure:**
```powershell
Copy-Item -LiteralPath "file.ext.bak2" -Destination "file.ext" -Force
# Re-apply a smaller safer patch, then verify again.
```

**After success:**
```powershell
Remove-Item -LiteralPath "file.ext.bak2" -Force
```

---

## 4. Code Comprehension, Search & Architecture

Do not patch blindly. Map the architecture first to prevent breaking dependencies.

### A. Structural Scan — get the skeleton
```powershell
# Count lines
(Get-Content "file.ext").Count

# JS/TS: function map
Select-String -LiteralPath "file.js" -Pattern "^function |^const |^class |^async function |^export " |
    Select-Object LineNumber, Line | Format-Table -AutoSize

# Python: class/def map
Select-String -LiteralPath "file.py" -Pattern "^def |^class |^async def " | Select-Object LineNumber, Line

# Event listeners / UI entry points
Select-String -LiteralPath "file.js" -Pattern "addEventListener|\.on\(" | Select-Object LineNumber, Line

# Startup / bootstrap
Select-String -LiteralPath "file.js" -Pattern "DOMContentLoaded|window\.onload|\.init\(\)|^main\(\)" |
    Select-Object LineNumber, Line
```

After scanning, produce an ARCHITECTURE_MAP and share it with the user:
```
ARCHITECTURE_MAP:
  config/state   : aiTransConfig (line 120), userPrefs (line 340)
  core logic     : translateText() (450), callAPI() (890)
  ui layer       : renderUI() (2100), bindEvents() (2350)
  data flow      : userAction → bindEvents → translateText → callAPI → renderUI
  entry point    : DOMContentLoaded (14800)
```

### B. Data Flow Tracing

Forward trace (source → destination):
```powershell
# Where is the variable CREATED
Select-String -LiteralPath "file.ext" -Pattern "apiKey\s*=" |
    Where-Object { $_.Line -notmatch "//|param|function" } | Select-Object LineNumber, Line

# Where is it MUTATED
Select-String -LiteralPath "file.ext" -Pattern "apiKey" |
    Where-Object { $_.Line -match "=|push|pop|splice|delete" } | Select-Object LineNumber, Line

# Where is it READ
Select-String -LiteralPath "file.ext" -Pattern "apiKey" |
    Where-Object { $_.Line -notmatch "=|push|pop" } | Select-Object LineNumber, Line
```

Backward trace (bug → root cause):
```powershell
# Start from broken UI output → find render function → find state variable → find where state is set
Select-String -LiteralPath "file.ext" -Pattern "renderUI|innerHTML|textContent" | Select-Object LineNumber, Line
```

### C. Blast Radius Assessment

Before writing any patch, answer:
- **Direct:** what code am I changing?
- **Indirect:** what functions call the changed code?
- **State:** what global/shared variables are read or written?
- **UI:** what DOM elements or event handlers are affected?
- **Timing:** any async callbacks, debounce, or event queues involved?

```powershell
# Find all references to changed symbol before patching
Select-String -LiteralPath "file.ext" -Pattern "targetName" | Select-Object LineNumber, Line
```

### D. Anchor Search

Always use unique tokens. Never use the full block from user (may be stale).

```powershell
# Search unique token
Select-String -LiteralPath "file.js" -Pattern "uniqueLogString|rareVarName" | Select-Object LineNumber, Line

# Filter by line range when too many hits
Select-String -LiteralPath "file.js" -Pattern "renderApiKey" |
    Where-Object { $_.LineNumber -gt 8000 } | Select-Object LineNumber, Line
```

### E. Read Bounded Range

Never read the whole file. Read 40–160 lines around the anchor.

```powershell
$lines = [System.IO.File]::ReadAllLines("file.js", [System.Text.Encoding]::UTF8)
for ($i = 1200; $i -le 1280; $i++) { '{0,6}: {1}' -f $i, $lines[$i-1] }
```

---

## 5. Patch Strategy

### Prefer smallest reliable change.

Use `apply_patch` when:
- File is inside writable workspace
- Patch is small/normal code text
- Encoding risk is low

Use .NET scripted replacement (Section 2) when:
- File has BOM/no-BOM that must be preserved
- Terminal display may corrupt Vietnamese/emoji
- Need ASCII anchors around non-ASCII strings

For multiple patches in one file:
- Prefer one validated script when encoding-sensitive
- Otherwise patch bottom-to-top to avoid line drift
- Verify after each meaningful write, not at the end of a chain

### Stale spec handling

When spec differs from current code — mandatory protocol, never merge silently:

```
1. Show user the diff:
   "Spec says: [X]"
   "Current code is: [Y]"
   "Difference: [explain]"

2. Ask explicitly:
   "Apply spec as-is? / Adapt to current code? / Skip this patch?"

3. No confirmation = no patch.
   If user is absent → leave a TODO comment, continue to next item.
```

When spec is completely unrecognizable (not found anywhere): STOP, ask user — file may have been refactored.

---

## 6. Verification

### Syntax check by language

| Language | Command |
|---|---|
| JavaScript `.js` | `node --check file.js` |
| TypeScript `.ts` | `npx tsc --noEmit` |
| Python `.py` | `python -m py_compile file.py` |
| Go `.go` | `go vet ./...` |
| Rust `.rs` | `cargo check` |
| PHP `.php` | `php -l file.php` |
| Ruby `.rb` | `ruby -c file.rb` |
| Bash `.sh` | `bash -n file.sh` |
| PowerShell `.ps1` | `$e=$null; [System.Management.Automation.Language.Parser]::ParseFile((Resolve-Path $path),[ref]$null,[ref]$e); $e` |
| C/C++ `.c/.cpp` | `gcc -fsyntax-only file.c` |

Auto-detect and check:
```powershell
Get-Command node, python, go, php, ruby 2>$null | Select-Object Name, Source
```

### Post-patch invariants

```powershell
# 1. No dangling references to old name (must = 0)
Select-String -LiteralPath "file.ext" -Pattern "oldName" | Measure-Object

# 2. New name exists (must = expected count)
Select-String -LiteralPath "file.ext" -Pattern "newName" | Measure-Object

# 3. Balanced braces (JS/TS)
$c = Get-Content "file.ext" -Raw
($c.ToCharArray() | Where-Object { $_ -eq '{' }).Count
($c.ToCharArray() | Where-Object { $_ -eq '}' }).Count

# 4. Re-read edited range to confirm result
$lines = [System.IO.File]::ReadAllLines("file.ext", [System.Text.Encoding]::UTF8)
for ($i = $editStart; $i -le $editEnd; $i++) { '{0,6}: {1}' -f $i, $lines[$i-1] }

# 5. Re-check encoding after write
$b = [System.IO.File]::ReadAllBytes($path)
if ($b[0] -eq 0xEF -and $b[1] -eq 0xBB -and $b[2] -eq 0xBF) { "BOM preserved OK" } else { "UTF-8 no BOM" }
```

### Cascade analysis — run after every non-trivial patch

| Change Type | Must Verify |
|---|---|
| Rename function | All call sites updated |
| Change signature | All callers pass correct args |
| Change return format | All consumers handle new format |
| Rename/remove state field | All readers + writers |
| Rename CSS class | All classList.add/remove + CSS rules |
| Change config key | All readers of that key |

### Proactive bug hunt

After each patch, scan for these common side effects:

| Pattern | Detection |
|---|---|
| Undefined variable | Declared before use? |
| Missing function definition | Definition still exists after rename? |
| Dead code after refactor | Old name still called anywhere? |
| Missing backward-compat | Old config key migrated everywhere? |
| Event listener leak | Every `addEventListener` has matching `removeEventListener`? |
| Hardcoded stale values | Old string literals in non-config lines? |

---

## 7. Multi-Agent Mid-Session Onboarding

Use when joining a session already in progress.

```powershell
# Step 1 — Verify session backup exists. Create immediately if missing.
Test-Path "file.ext.bak*"

# Step 2 — Check for ARCHITECTURE_MAP from previous agent
# If found → read it first. If not → run Section 4A Structural Scan.

# Step 3 — Confirm tool availability
Get-Command node, python, go, php 2>$null | Select-Object Name, Source

# Step 4 — Check for feature_map or knowledge index
Get-ChildItem -Filter "*feature_map*","*knowledge*","*KI*" -Recurse | Select-Object FullName
# If found → read before touching any code.
```

Do NOT start patching until all steps are confirmed. Then ask the user:
> "Ready. Session backup confirmed. What's the current task?"

---

## 8. Large JS UI Files

Safe orientation order:
1. Find config/state key (line ~top of file)
2. Find UI render block (search `renderUI|innerHTML|textContent`)
3. Find event handlers and save path (search `addEventListener|saveConfig`)
4. Find initialization/bootstrap (search `DOMContentLoaded`)
5. Find CSS if visual issue (search class name in style blocks)
6. Patch all affected paths or explicitly state why not
7. Run `node --check`
8. Search old IDs/classes/functions to confirm removal

For dynamic dropdown/model lists:
- Keep API IDs separate from display names
- If provider returns live IDs, use those for API calls
- Hardcoded names are friendly labels and priority ordering only
- Unknown live models can be shown with generated labels if filtered for correct modality

For popup/scroll UI:
- If a guard must always be visible, make it part of the scroll container frame (`border`, `padding`, `sticky`), not only an element at the bottom of scroll content
- If a placeholder/status has no real information, hide it by default and show only when populated

---

## 9. Failure Recovery

If syntax check fails immediately after a patch:
1. Do not edit the broken file further
2. Restore `.bak2`
3. Re-read the exact range
4. Apply a smaller, simpler patch
5. Verify again

If encoding looks corrupted:
1. Stop editing immediately
2. Restore `.bak2` if available; otherwise restore session backup
3. Re-apply minimal changes using .NET explicit encoding (Section 2)
4. Verify encoding and a known non-ASCII string after write

---

## 10. Concise Final Report

Include only high-signal items:

```
Files changed   : file.js (lines X–Y, Z–W)
Behavior change : [what changed and why]
Verification    : node --check → OK | FAIL
Encoding        : UTF-8 BOM preserved / UTF-8 no BOM preserved
Backup state    : session backup kept at file.js.bak.codex-session-...
                  .bak2 deleted (verification passed) / kept (verification failed)
Remaining risk  : [any manual browser check or known limitation]
```

---

## 11. Do Not Over-Expand

This is the complete active protocol. Act on it directly. Do not narrate the skill loading or explain the workflow to the user unless asked. The default is to execute.
