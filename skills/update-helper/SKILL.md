---
name: update-helper
description: |
  Universal field manual for safely reading, understanding, and patching large source files
  (5,000–30,000+ lines) in ANY language. Windows/IDE environment (PowerShell).
  Covers: Code Comprehension Layer (intent mapping, data flow, architecture summary),
  3-tier backup, anchor-based patching, multi-language syntax check, cascade analysis,
  stale-patch detection, proactive bug hunt, encoding safety (BOM preservation),
  and multi-patch offset strategy.
  Original by PitroyTech. Designed as a ready-to-consult field manual for any agent joining mid-session.
  ALWAYS COMMUNICATE WITH THE USER IN THEIR OWN LANGUAGE — detect the language the user is writing
  in and respond in that language. English only for code identifiers and terminal commands.
---

# Universal Large File Patcher — Complete Field Manual v3.3 (Windows IDE Edition)
### by PitroyTech

You are patching a massive source file (any language, 1,000–30,000+ lines).
**One wrong step can blow up the Context Window or destroy the codebase.**
This is your survival manual. Follow it to the letter.

> 🌐 **LANGUAGE RULE:** Detect the language the user is writing in and **respond in that same language**.
> Vietnamese → Vietnamese. English → English. Other languages → match accordingly.
> English only for code identifiers, variable/function names, and terminal commands.

---

## ⚡ QUICK START — Read this block first, always

> This block is enough to begin working. Deep-dive sections are reference — load on demand.

**5 Non-negotiable rules:**
1. `view_file` requires `StartLine` + `EndLine` — ALWAYS. No exceptions.
2. Never type `TargetContent` by hand — copy verbatim from `view_file` output.
3. Search → Read → Understand → Plan → Patch → Verify. In that order.
4. Create `.bak` at session start. Create `.bak2` before every complex patch.
5. Patch spec from user may be STALE — verify actual code before applying.

**Tool cheat sheet:**

| Need | Tool |
|---|---|
| Find a line | `Select-String -Path "f" -Pattern "keyword" \| Select-Object LineNumber, Line` |
| Read code | `view_file(path, StartLine=N, EndLine=M)` |
| Replace 1 block | `replace_file_content` |
| Replace N blocks | `multi_replace_file_content` |
| Syntax check | See §J1 for your language |
| Backup / Rollback | See §1 |

**Where to go next:**
- Joining mid-session? → §0C (Agent Onboarding)
- Need to understand the file first? → §A1–§A5 (Code Comprehension)
- Know what to patch, need workflow? → §F4 (Smart Patching)
- Got an error? → §E1 (Diagnostic Table)

---

## TABLE OF CONTENTS

```
QUICK START ............. above
TABLE OF CONTENTS ....... this section
HOW TO READ THIS FILE ... §0D

§0   Iron Laws
§0B  Tool Mapping
§0C  Agent Onboarding (mid-session)
§0D  How to Read This File (read ranges)
§0E  Encoding Safety (BOM preservation)
§1   3-Tier Backup Protocol

Part A — Code Comprehension
  §A1  When to Use Comprehension Mode
  §A2  Structural Scan
  §A3  Intent Mapping
  §A4  Data Flow Tracing
  §A5  Architecture Summary

Part B — Search & Navigation
  §B1  Find Code — Priority Order

Part C — Patch Workflow
  §C1  Patch Actions
  §C2  Multi-Patch Offset Strategy

Part D — Logic Awareness
  §D1  Pre-Patch Analysis
  §D2  Stale Patch Detection
  §D3  Cascade Analysis
  §D4  Smart Patch Adaptation
  §D5  Post-Patch Verification
  §D6  Proactive Bug Hunt

Part E — Error Handling
  §E1  "Target Not Found" Diagnostic
  §E2  Check Line Endings
  §E3  Special Character Escaping
  §E4  AllowMultiple
  §E5  Partial Failure in multi_replace
  §E6  Encoding Corruption

Part F — Complete Workflows
  §F1  "Explain what this file does"
  §F2  "Add feature X"
  §F3  "Fix bug Y"
  §F4  Smart Patching Workflow (complex patches)

─── HUMAN REFERENCE ONLY BELOW ───────────────────

Part G — Scoreboard (human metadata — AI: skip)
  §G1  Technique Scoreboard
  §G2  Session Log
  §G3  Default Technique Order (AI-readable summary)

Part H — UI/CSS Guidelines
Part I — Pre-Submit Checklist
Part J — Language Detection Layer
  §J1  Syntax Check by Language
  §J2  Structural Scan by Language
  §J3  Auto-Detect & Dispatch
```

---

## §0D. HOW TO READ THIS FILE

> This document is 700+ lines. Never read it all at once — load only what you need.

| Goal | Read range | Key sections |
|---|---|---|
| Quick orient (new session) | Lines 1–90 | Quick Start + §0 + §0B |
| Mid-session join | Lines 1–90 + §0C | Quick Start + Agent Onboarding |
| Understand unfamiliar codebase | §A1–§A5 | Code Comprehension |
| Execute a patch | §D1–§D5, §F4 | Logic Awareness + Smart Workflow |
| Diagnose a failed patch | §E1–§E5 | Error Handling |
| Syntax check / language tools | §J1–§J3 | Language Detection |
| Pre-submit sanity check | §I | Checklist |

**Read pattern for a typical patch session:**
```
1. Quick Start block (always)
2. §1 (backup created?)
3. §A2 (scan structure if file is new to you)
4. §D1 → §D4 → §F4 (understand → analyze → execute)
5. §D5 + §J1 (verify + syntax check)
6. §I (pre-submit checklist)
```

---

## §0E. ⚠️ ENCODING SAFETY — Read Before ANY File Write

> **Battle-tested lesson:** IDE edit tools (`replace_file_content`, `multi_replace_file_content`) and
> `Set-Content -Encoding UTF8` will **SILENTLY DESTROY** UTF-8 BOM encoding on PowerShell files.
> This corrupts ALL non-ASCII characters (Vietnamese, CJK, accented Latin, etc.) with NO warning.

### Step 1: Detect encoding BEFORE first edit
```powershell
$b = [System.IO.File]::ReadAllBytes("file.ps1")
if ($b[0] -eq 0xEF -and $b[1] -eq 0xBB -and $b[2] -eq 0xBF) { "UTF-8 BOM" }
elseif ($b[0] -eq 0xFF -and $b[1] -eq 0xFE) { "UTF-16 LE" }
else { "UTF-8 no BOM (or ASCII)" }
```

### Step 2: Choose the RIGHT write method

| File has BOM? | Non-ASCII content? | Safe write method |
|---|---|---|
| UTF-8 BOM | Yes (Vietnamese, etc.) | `[System.IO.File]::WriteAllText($path, $content, [System.Text.UTF8Encoding]::new($true))` |
| UTF-8 no BOM | Any | `Set-Content -Encoding UTF8` or IDE tools OK |
| UTF-16 LE | Any | `[System.IO.File]::WriteAllText($path, $content, [System.Text.UnicodeEncoding]::new())` |

### Step 3: NEVER use these on BOM files
```
❌ Set-Content -Encoding UTF8          → strips BOM
❌ replace_file_content (IDE tool)     → strips BOM  
❌ multi_replace_file_content          → strips BOM
❌ Get-Content | Set-Content pipeline  → may strip BOM
```

### Step 4: Safe write pattern for BOM files
```powershell
# READ with .NET (preserves encoding awareness)
$content = [System.IO.File]::ReadAllText($path, [System.Text.Encoding]::UTF8)

# MODIFY via string operations
$content = $content.Replace("old", "new")
# Or via line array:
$lines = $content -split "`r`n"
$lines[41] = "new content"
$content = $lines -join "`r`n"

# WRITE with explicit BOM
$utf8Bom = [System.Text.UTF8Encoding]::new($true)
[System.IO.File]::WriteAllText($path, $content, $utf8Bom)
```

### Step 5: Verify after write
```powershell
$b = [System.IO.File]::ReadAllBytes($path)
$bomOK = ($b[0] -eq 0xEF -and $b[1] -eq 0xBB -and $b[2] -eq 0xBF)
"Encoding preserved: $bomOK"
```

> **Rule of thumb:** If the file contains ANY non-ASCII characters, ALWAYS check BOM first
> and use `[System.IO.File]::WriteAllText()` with explicit encoding.

---

## §0. IRON LAWS

1. **NEVER** use `view_file` without `StartLine` + `EndLine`. Full-file reads blow up Context Window.
2. **Search → Read → Understand → Plan → Patch → Verify.** No exceptions.
3. **NEVER type TargetContent by hand.** Copy verbatim from `view_file` output.
4. **3-tier backup protocol** — see §1 below.
5. **Verify after EVERY patch** — not after a whole batch.
6. **🧠 THINK BEFORE PATCHING** — Understand consequences BEFORE applying.
7. **Patch spec from user may be STALE.** Verify actual code first.
8. **Side effects are YOUR responsibility.** Fix downstream breaks proactively.
9. **DETECT ENCODING BEFORE FIRST EDIT.** If file has BOM or non-ASCII → see §0E. IDE tools WILL corrupt BOM files silently.

---

## §0B. TOOL MAPPING — Windows IDE Environment

| Purpose | Tool |
|---|---|
| Find lines by keyword | PowerShell `Select-String` |
| Read code at coordinates | IDE `view_file` (StartLine/EndLine) |
| Replace 1 block | `replace_file_content` |
| Replace multiple blocks | `multi_replace_file_content` |
| Run commands / syntax check | `run_command` (PowerShell) |
| Backup file | `Copy-Item "file.ext" "file.ext.bak"` |
| Rollback | `Copy-Item "file.ext.bak" "file.ext" -Force` |
| Create new file | IDE file creation tool |

---

## §0C. AGENT ONBOARDING — Mid-Session Join

> Use this when you are joining a session already in progress and need to orient fast.

**Step 1 — Verify backup status**
```powershell
Test-Path "file.ext.bak"   # Must exist. If not → create immediately.
```

**Step 2 — Check if ARCHITECTURE_MAP exists**
- If the user or a previous agent has already produced an `ARCHITECTURE_MAP` → read it first (saves you a full §A2 scan).
- If not → run §A2 Structural Scan before touching any code.

**Step 3 — Confirm tool availability**
```powershell
Get-Command node, python, go, php 2>$null | Select-Object Name, Source
# Know which syntax checkers are available before you need them.
```

**Step 4 — Check for feature_map / knowledge index**
```powershell
# If the project has a feature map or KI file → read it first (§T13 principle).
Get-ChildItem -Filter "*feature_map*","*KI*","*knowledge*" -Recurse | Select-Object FullName
```

**Step 5 — Ask the user**
> "Tôi đã sẵn sàng. Backup ✅. Nhiệm vụ hiện tại là gì?"
> (or in their language: "Ready. Backup ✅. What's the current task?")

**Do NOT start patching until all 5 steps are confirmed.**

---

## §1. 🛡️ 3-TIER BACKUP PROTOCOL

> **Rationale:** AI patch sessions frequently have phases of "apply → error → need to revert fast."
> Having a secondary backup *right before* a complex patch means you can rollback in 1 command
> without hunting for which line the error was introduced.

### Tier 1 — Main Backup (Session Start, once per session)
```powershell
Copy-Item "file.ext" "file.ext.bak"
# Created ONCE at the start of the session. Never overwritten.
# This is your ultimate fallback.
```

### Tier 2 — Secondary Backup (Before each complex/large patch)
Create a secondary backup immediately before any patch that:
- Modifies 3+ non-contiguous blocks
- Changes function signatures or renames variables
- Touches core state/config logic
- Is part of a multi-step refactor

```powershell
Copy-Item "file.ext" "file.ext.bak2"
```

### Tier 3 — Verify & Cleanup (After syntax check passes)
```powershell
# Run syntax check first (see §J1 for language-specific commands)
node -c "file.js"   # example for JS
# If OK → delete secondary backup
Remove-Item "file.ext.bak2"
# If ERROR → rollback to secondary, NOT main (preserve main for ultimate fallback)
Copy-Item "file.ext.bak2" "file.ext" -Force
```

### Rollback Decision Tree
```
Patch failed?
├── Syntax error, just applied → restore .bak2, fix the specific patch, retry
└── Multiple patches broken   → restore .bak (main), start session over
```

---

## ═══════════════════════════════════════
## PART A — CODE COMPREHENSION
## ═══════════════════════════════════════

> This layer ensures the agent **understands the codebase** before acting — not just patching 1-to-1.

---

## §A1. WHEN TO USE COMPREHENSION MODE?

Use comprehension mode when:
- User asks "explain what this file does"
- User asks "add feature X" (unclear where to patch)
- File > 3,000 lines and patch involves complex logic
- Patch spec is vague or doesn't specify location
- Need to understand data flow before refactoring

---

## §A2. STRUCTURAL SCAN — Read the File Map

### Step 1: Count lines & get skeleton
```powershell
(Get-Content "file.ext").Count

# JS/TS: get function map
Select-String -Path "file.js" -Pattern "^function |^const |^class |^async function |^export " |
    Select-Object LineNumber, Line | Format-Table -AutoSize

# Python: get class/def map
Select-String -Path "file.py" -Pattern "^def |^class |^async def " |
    Select-Object LineNumber, Line

# PowerShell: get function map
Select-String -Path "file.ps1" -Pattern "^function |^Function " |
    Select-Object LineNumber, @{L='F';E={$_.Line.TrimStart().Substring(0,[math]::Min(90,$_.Line.TrimStart().Length))}} |
    Format-Table -AutoSize

# PowerShell: find param blocks and script-scope variables
Select-String -Path "file.ps1" -Pattern "^\$script:|param\(|Mandatory" |
    Select-Object LineNumber, Line
```

### Step 2: Find large logic blocks
```powershell
# Find all function definitions
Select-String -Path "file.ext" -Pattern "function |=> {" |
    Where-Object { $_.Line -notmatch "//|console" } |
    Select-Object LineNumber, @{L='Line'; E={$_.Line.TrimStart().Substring(0, 90)}} |
    Format-Table -AutoSize

# Find event listeners (UI flow entry points)
Select-String -Path "file.js" -Pattern "addEventListener|\.on\(" | Select-Object LineNumber, Line

# Find state/config objects
Select-String -Path "file.ext" -Pattern "^const |^let |^var " |
    Where-Object { $_.Line -match "config|state|store|cache|data" } |
    Select-Object LineNumber, Line
```

### Step 3: Map entry points
```powershell
# Find startup points
Select-String -Path "file.js" -Pattern "DOMContentLoaded|window\.onload|\.init\(\)|^main\(\)" |
    Select-Object LineNumber, Line

# Find exports (module public API)
Select-String -Path "file.js" -Pattern "^export |module\.exports|window\." |
    Select-Object -First 30 LineNumber, Line
```

### Expected output — agent summarizes for user:
```
📁 File: xxx.js (~15,000 lines)
🏗️ Main structure:
  - Config/State: aiTransConfig (line 120), userPrefs (line 340)
  - Core functions: translateText() (450), callAPI() (890), saveCache() (1200)
  - UI layer: renderUI() (2100), bindEvents() (2350)
  - Entry point: DOMContentLoaded (14800)
📌 Data flow: user action → bindEvents → translateText → callAPI → renderUI
```

---

## §A3. INTENT MAPPING — Understand the Purpose of Code

Before reading any function in detail, answer 3 questions:

```
❓ WHAT: What does this function/code block DO?
❓ WHY:  Why does it exist? What problem does it solve?
❓ HOW:  HOW does it accomplish that? (processing flow)
```

**Workflow:**
```powershell
# 1. Find function by name
Select-String -Path "file.ext" -Pattern "function translateText|translateText = " | Select-Object LineNumber, Line

# 2. View function header to understand params and intent
# (use view_file with 10-15 line range before reading further)

# 3. Find all callers
Select-String -Path "file.ext" -Pattern "translateText\(" | Select-Object LineNumber, Line

# 4. Find all consumers of return value
# (read the callers found above)
```

---

## §A4. DATA FLOW TRACING — Follow the Data

### Forward trace (from source to destination):
```powershell
# 1. Find where data is CREATED
Select-String -Path "file.ext" -Pattern "apiKey\s*=" |
    Where-Object { $_.Line -notmatch "//|param|function" } | Select-Object LineNumber, Line

# 2. Find where data is MUTATED
Select-String -Path "file.ext" -Pattern "apiKey" |
    Where-Object { $_.Line -match "=|push|pop|splice|delete" } | Select-Object LineNumber, Line

# 3. Find where data is READ/USED
Select-String -Path "file.ext" -Pattern "apiKey" |
    Where-Object { $_.Line -notmatch "=|push|pop" } | Select-Object LineNumber, Line
```

### Backward trace (from bug to root cause):
```powershell
# Start from the problematic location, trace backward to source
Select-String -Path "file.ext" -Pattern "renderUI|innerHTML|textContent" | Select-Object LineNumber, Line
# → find variable read in renderUI
# → grep that variable to find where it's set
```

### End-to-end feature trace template:
```
User clicks button
    → Select-String: "btn.*addEventListener|click.*btn"
    → read handler function
    → what does the handler call?
        → Select-String: "handlerName("
        → read function body
        → how does it call API / set state / update UI?
    → where does the result render?
```

---

## §A5. ARCHITECTURE SUMMARY — Map the Codebase

After scanning, create an internal map used throughout the session:

```
ARCHITECTURE_MAP = {
  config_objects: ["aiTransConfig:120", "userPrefs:340"],
  core_logic: ["translateText:450", "callAPI:890"],
  ui_layer: ["renderUI:2100", "bindEvents:2350"],
  data_flow: "userAction → bindEvents → translateText → callAPI → renderUI",
  entry_point: 14800,
  key_state: ["currentProvider:aiTransConfig.provider", "apiKeys:aiTransConfig.keys"]
}
```

### PowerShell / .ps1 example:
```
ARCHITECTURE_MAP = {
  string_tables: ["VI_Strings:700", "EN_Strings:1400"],
  core_functions: ["Get-WindowsAccountList:6552", "Reset-LocalAccountPassword:6732"],
  ui_wizards: ["Show-CrackPass-Step1:11000", "Show-CrackPass-Step2:11493", "Show-CrackPass-Step3:11781"],
  execute_logic: ["Invoke-CrackPassExecute:12200"],
  data_flow: "Step1(selectDrive) → Step2(selectAccount) → Step3(selectAction) → Step4(confirm) → Execute",
  encoding: "UTF-8 BOM (required for Vietnamese strings)",
  key_state: ["$script:_cpAccountList", "$script:Tokens", "$S (localized strings)"]
}
```

---

## ═══════════════════════════════════════
## PART B — SEARCH & NAVIGATION
## ═══════════════════════════════════════

## §B1. Find Code — Priority Order

### Priority 1: Coordinate Scan (Fastest)
```powershell
Select-String -Path "file.ext" -Pattern "uniqueKeyword" | Select-Object LineNumber, Line

# Filter by line range
Select-String -Path "file.ext" -Pattern "renderApiKeyRows" |
    Where-Object { $_.LineNumber -gt 8000 } | Select-Object LineNumber, Line

# Multi-pattern feature map
Select-String -Path "file.ext" -Pattern "pat1|pat2|pat3" |
    Select-Object LineNumber, @{L='L'; E={$_.Line.TrimStart().Substring(0, 90)}} |
    Format-Table -AutoSize
```

✅ Use: unique identifiers, DOM ids, distinctive log strings
✅ Combine 2 tokens: `"isPaidKey.*paidKeyLabels"`
❌ Avoid: `saveConfig`, `addEventListener`, `data`

### Priority 2: Fuzzy Search (When user's pasted code has wrong indent)

Rule: **NEVER search the full block user provides.**
1. Extract 1 surgical token (rare variable name, unique log string)
2. Search that token → get LineNumber
3. `view_file` at that line to get exact current code

### Priority 3: Structural Scan (When completely lost)
See **§J2** for per-language scan commands.

### Priority 4: Precision View (Final step before any edit)
```
view_file(path, StartLine=line-5, EndLine=line+15)
```
This is the **SINGLE SOURCE OF TRUTH** for `TargetContent`. Copy verbatim.

---

## ═══════════════════════════════════════
## PART C — PATCH WORKFLOW
## ═══════════════════════════════════════

## §C1. Patch Actions — Priority Order

### §C1A. Single Continuous Block → `replace_file_content`
### §C1B. Multiple Non-Contiguous Blocks → `multi_replace_file_content`

**Overlap Warning:** Overlapping TargetContent → merge into 1 chunk.

### §C1C. Insert Code
`TargetContent` = line BEFORE + line AFTER insertion point.
`ReplacementContent` = before + new code + after.

### §C1D. Replace Entire Function
Find opening line + matching `}` or `});` close → use entire block as TargetContent.

### §C1E. Last Resort: PowerShell Direct Replace
> ⚠️ **WARNING:** `Set-Content` strips BOM. For BOM files, use §C1F instead.

```powershell
# ONLY for non-BOM files:
$c = Get-Content "file.ext" -Raw
$c = $c.Replace("old code", "new code")
Set-Content "file.ext" $c -NoNewline
```

### §C1F. Line-Based Array Replacement (Best for BOM files)
> When `string.Replace()` fails due to indent mismatch, or IDE tools corrupt encoding,
> use line-array manipulation — no indent matching needed, just correct line indices.

```powershell
$content = [System.IO.File]::ReadAllText($path, [System.Text.Encoding]::UTF8)
$lines = $content -split "`r`n"

# Find exact line index by searching (never hardcode!)
$targetIdx = -1
for ($i = 0; $i -lt $lines.Count; $i++) {
    if ($lines[$i] -match "unique pattern here") { $targetIdx = $i; break }
}

# Replace range
$patchLines = @("    new line 1", "    new line 2", "    new line 3")
$before = $lines[0..($targetIdx - 1)]
$after  = $lines[($targetIdx + 5)..($lines.Count - 1)]  # skip 5 old lines
$merged = $before + $patchLines + $after

# Write with encoding preserved
$utf8Bom = [System.Text.UTF8Encoding]::new($true)
[System.IO.File]::WriteAllText($path, ($merged -join "`r`n"), $utf8Bom)
```

## §C2. MULTI-PATCH OFFSET STRATEGY

> **Problem:** When applying multiple patches sequentially, earlier patches shift line numbers,
> causing later patches to target wrong lines (replace wrong block → syntax error or data loss).

### Rule: Patch from BOTTOM to TOP
```
File has 3 patches at lines 800, 5000, 12000:
1. Apply patch at line 12000 FIRST  (bottom)
2. Apply patch at line 5000 SECOND  (middle — unaffected by #1)
3. Apply patch at line 800 LAST     (top — unaffected by #1 and #2)
```

### Alternative: All patches in one script
```powershell
$lines = $content -split "`r`n"

# Patch B (lower in file) — apply FIRST
$lines = $before_B + $patchB + $after_B

# Re-search for Patch A anchor (upper) — index unchanged since B was below
$lines = $before_A + $patchA + $after_A
```

### Anti-pattern: Hardcoded line indices
```
❌ $lines[12100..12105]   # WRONG — shifts after every patch
✅ Search for anchor pattern each time before patching
```

### Verify after each patch step
```powershell
# Immediately after each sub-patch, verify the target was correct
Write-Host "Replaced lines $startIdx to $endIdx"
Write-Host "First: $($lines[$startIdx])"
Write-Host "Last:  $($lines[$endIdx])"
```

---

## ═══════════════════════════════════════
## PART D — LOGIC AWARENESS
## ═══════════════════════════════════════

## §D1. PRE-PATCH ANALYSIS (Mandatory for non-trivial patches)

**Step 1: Understand the TARGET**
- WHAT does this code do? (read 20 lines above/below)
- WHO calls it? (search callers)
- WHAT consumes its output? (search consumers)

**Step 2: Map BLAST RADIUS**
- Direct / Indirect / State / UI / Timing effects

**Step 3: Dependency Check**
```powershell
Select-String -Path "file.ext" -Pattern "targetName" | Select-Object LineNumber, Line | Format-Table -AutoSize
# After patch: verify count = expected
Select-String -Path "file.ext" -Pattern "targetName" | Measure-Object
```

## §D2. STALE PATCH DETECTION
```
⚠️ NEVER assume patch spec matches current code.
ALWAYS view actual code first.
```
1. Extract unique tokens from spec's "old code"
2. Search in current file
3. If differs → report diff, adapt patch, ask for confirmation

## §D3. CASCADE ANALYSIS

| Change Type | Must Check |
|---|---|
| Rename function | All call sites |
| Change function signature | All callers — correct args? |
| Change return value format | All consumers |
| Rename/remove state field | All readers + writers |
| Modify CSS class name | All classList.add/remove + CSS rules |
| Change config key | All readers of that key |
| Change config structure | All config readers |

## §D4. SMART PATCH ADAPTATION

```
1. Exact match?               → Apply ✅
2. Minor whitespace diff?     → Re-copy from view_file ✅
3. Logic slightly different?  → STOP — follow steps below ⚠️
4. Completely restructured?   → STOP, ask user ❌
```

**Case 3 — Logic slightly different (mandatory protocol):**
```
a. Show user the diff:
   "Spec says: [X]"
   "Current code is: [Y]"
   "Difference: [explain what changed]"

b. Ask explicitly:
   "Apply spec as-is? / Adapt to current code? / Skip this patch?"

c. NEVER merge silently.
   No confirmation from user = no patch applied.
   If user is absent → leave a TODO comment and move on.
```

## §D5. POST-PATCH VERIFICATION
```powershell
# 1. Syntax check (see §J1 for your language)
node -c "file.js"

# 2. No dangling references
Select-String -Path "file.ext" -Pattern "oldName" | Measure-Object  # Must = 0

# 3. Balanced braces
$c = Get-Content "file.ext" -Raw
($c.ToCharArray() | Where-Object { $_ -eq '{' }).Count
($c.ToCharArray() | Where-Object { $_ -eq '}' }).Count

# 4. Function still exists
Select-String -Path "file.ext" -Pattern "function editedFunc" | Measure-Object  # = 1
```

## §D6. PROACTIVE BUG HUNT

| Bug Pattern | Detection |
|---|---|
| Undefined variable | Declared before use? |
| Function missing | Definition exists? |
| Dead code after refactor | Old name still called? |
| Missing backward-compat | Old config key migrated? |
| Event listener leak | addEventListener without removeEventListener? |
| Hardcoded values | Old strings still in non-config lines? |

---

## ═══════════════════════════════════════
## PART E — ERROR HANDLING
## ═══════════════════════════════════════

## §E1. "Target Not Found" — Diagnostic Table

| Cause | Fix |
|---|---|
| Whitespace mismatch | Re-copy from `view_file` |
| Line numbers shifted | Re-search, get new coordinates |
| CRLF vs LF mismatch | Check encoding (§E2) |
| Pattern too common | Use more specific anchor or AllowMultiple:true |
| Stale patch spec | See §D2 |

## §E2. Check Line Endings
```powershell
(Get-Content "file.ext" -Raw)[0..200] -join '' | Select-String "`r`n"
```

## §E3. Special Character Escaping
| Char | Escape |
|---|---|
| `"` | `\"` |
| `\` | `\\` |
| Backtick, `${...}` | No escape needed |

## §E4. AllowMultiple
- `false` — default (1 location)
- `true` — same pattern, ALL occurrences need same change

## §E5. Partial Failure in multi_replace
Other chunks ALREADY applied. Fix only the failed chunk — do NOT rollback all.

## §E6. Encoding Corruption — Detection & Recovery

### Symptoms
- Vietnamese/CJK characters become `?`, `�`, or garbage after edit
- PowerShell parser reports errors on previously-valid string literals
- File size changes unexpectedly (BOM = 3 bytes difference)

### Diagnosis
```powershell
# Check if BOM was stripped
$b = [System.IO.File]::ReadAllBytes("file.ps1")
$hasBOM = ($b[0] -eq 0xEF -and $b[1] -eq 0xBB -and $b[2] -eq 0xBF)
"Has BOM: $hasBOM"

# Quick visual check for corruption
$sample = [System.IO.File]::ReadAllText("file.ps1", [System.Text.Encoding]::UTF8)
# Search for known Vietnamese string — if garbled, encoding is wrong
$sample -match "xoá|thành|khoả|không"  # should be True for VI files
```

### Recovery
```powershell
# If .bak2 exists (pre-patch backup) → restore it
Copy-Item "file.ps1.bak2" "file.ps1" -Force

# If only .bak exists → restore and re-apply patches using safe method
Copy-Item "file.ps1.bak" "file.ps1" -Force
# Then re-apply ALL patches using [System.IO.File]::WriteAllText() only
```

### Prevention (add to §0E checklist)
```
Before EVERY edit session on non-ASCII files:
1. Detect encoding → record it
2. Choose write method based on §0E
3. After write → verify BOM + spot-check non-ASCII content
```

---

## ═══════════════════════════════════════
## PART F — COMPLETE WORKFLOWS
## ═══════════════════════════════════════

## §F1. "Explain what this file does"
```
1. Count lines → get total
2. Structural scan → functions/classes skeleton
3. Find entry points
4. Find config/state objects → data model
5. Trace 1-2 main flows
6. Summarize: purpose, structure, data flow
```

## §F2. "Add feature X"
```
1. Understand feature X
2. Scan file for related code
3. Identify intervention point
4. Map blast radius
5. Draft patch
6. Apply (anchor-based replace)
7. Verify syntax + references + cascade
8. Report
```

## §F3. "Fix bug Y"
```
1. Backward trace from symptom to root cause
2. Verify root cause by reading surrounding code
3. Check fix doesn't cause side effects
4. Apply fix
5. Verify
6. Proactive check: does the fix expose other bugs?
```

## §F4. SMART PATCHING WORKFLOW (Complex patches)
```
1.  RECEIVE spec
2.  EXTRACT unique tokens
3.  SEARCH → match current code?
    → NO: report diff, adapt patch, confirm (see §D4 Case 3)
4.  CREATE secondary backup (.bak2)
5.  VIEW actual code → copy TargetContent
6.  ANALYZE blast radius (§D1)
7.  APPLY patch
8.  VERIFY syntax (§J1 for your language)
    → OK: delete .bak2
    → FAIL: restore .bak2, fix, retry
9.  VERIFY references (no dangling)
10. VERIFY cascade effects (§D3)
11. REPORT to user
```

### Report Format
```
✅ PATCH-03 applied.

📋 Changes:
- X → Y at N locations

🛡️ Backup:
- .bak2 created before patch → deleted after syntax OK

🔍 Cascade check:
- caller_1() ✅  caller_2() ✅

⚠️ Extra findings:
- [proactive bugs found and fixed]

🔧 Syntax check: OK
```

---

## ─────────────────────────────────────────────
## HUMAN REFERENCE ONLY — Sections G onwards
## AI agent: §I (Pre-Submit Checklist) and §J (Language Tools) still apply to you.
## ─────────────────────────────────────────────

## ═══════════════════════════════════════
## PART G — SCOREBOARD
## ═══════════════════════════════════════

> ⚠️ **HUMAN METADATA — AI agent: skip §G1 and §G2. Go to §G3 for actionable technique guidance.**

## §G1. Technique Scoreboard

| ID | Technique | Base | Accumulated | Total |
|:---|:---|:---:|:---:|:---:|
| T1 ⭐⭐⭐⭐ | Coordinate Scan | 95 | 55 | **150** |
| T2 ⭐⭐ | Fuzzy Search | 90 | 0 | **90** |
| T3 ⭐ | Structural Scan | 70 | 0 | **70** |
| T4 ⭐⭐⭐⭐⭐ | Precision View | 100 | 115 | **215** |
| T5 ⭐⭐⭐ | Anchor-Based Replace | 95 | 45 | **140** |
| T6 ⭐ | Context-Wrapped Insert | 85 | 0 | **85** |
| T7 ⭐⭐ | Multi-Replace | 80 | 45 | **125** |
| T8 ⭐ | PowerShell Direct Replace | 75 | 0 | **75** |
| T9 ⭐⭐ | 3-Tier Backup | 95 | 0 | **95** |
| T10 ⭐⭐⭐ | Double Verify | 90 | 20 | **110** |
| T11 ⭐ | Narrow with Where-Object | 80 | 0 | **80** |
| T12 ⭐ | Reuse existing CSS classes | 65 | 0 | **65** |
| T13 ⭐⭐ | Read feature_map / KIs first | 70 | 20 | **90** |
| T14 ⭐⭐⭐ | 🧠 Blast Radius Analysis | 90 | 0 | **90** |
| T15 ⭐⭐ | 🧠 Stale Patch Detection | 85 | 0 | **85** |
| T16 ⭐⭐⭐ | 🧠 Cascade Verification | 90 | 0 | **90** |
| T17 ⭐⭐ | 🧠 Proactive Bug Hunt | 80 | 0 | **80** |
| T18 ⭐⭐⭐⭐ | 🧠 Intent Mapping | 90 | 30 | **120** |
| T19 ⭐⭐⭐⭐ | 🧠 Data Flow Tracing | 85 | 30 | **115** |
| T20 ⭐⭐⭐ | 🧠 Architecture Summary | 80 | 20 | **100** |
| T21 ⭐⭐⭐⭐⭐ | 🛡️ Encoding Safety | 100 | 30 | **130** |

## §G2. Session Log

| Date | Agent | 🥇 Best | 🥈 Second | 🥉 Third | Notes |
|:---|:---|:---|:---|:---|:---|
| 2025-07-15 | Claude | T4 (+30) | T1 (+25) | T5 (+20) | Initial |
| 2026-03-23 | Antigravity | T4 (+30) | T1 (+25) | T5 (+20) | First update |
| 2026-03-23 | Antigravity | T4 (+30) | T7 (+25) | T10 (+20) | 6-module cascade safety |
| 2026-03-23 | Antigravity | T1 (+30) | T4 (+25) | T13 (+20) | 20-30k line arch analysis |
| 2026-03-28 | Antigravity | T4 (+30) | T5 (+25) | T7 (+20) | MSS bug fix, youtube-sub.js |
| 2026-03-28 | Antigravity | T4 (+30) | T1 (+25) | T10 (+20) | Loading notice, context prompt |
| 2026-04-18 | Claude v3 | T18 (+30) | T19 (+25) | T20 (+20) | Code Comprehension Layer |
| 2026-04-23 | Antigravity | T9 (+30) | T4 (+25) | T1 (+20) | v3.1: multi-lang + 3-tier backup |
| 2026-04-23 | Claude v3.3 | F-01 F-02 F-03 | S-01–S-04 | P-01–P-03 | v3.2: AI-readability overhaul |
| 2026-05-03 | Antigravity | T21 (+30) | T4 (+25) | T9 (+20) | v3.3: Encoding safety, PS1 examples, offset strategy |

## §G3. DEFAULT TECHNIQUE ORDER — For AI Agent

> This is the prescriptive version of the scoreboard — use this to decide which technique to reach for first.

```
Every read:        T4  Precision View       → view_file with StartLine/EndLine, always
Before any patch:  T1  Coordinate Scan      → find the exact line first
Complex patches:   T5  Anchor-Based Replace → preferred over PowerShell direct (T8)
Multi-block:       T7  Multi-Replace        → batch non-contiguous changes in 1 call
After patch:       T10 Double Verify        → syntax check + reference check
New session:       T9  3-Tier Backup        → create .bak before anything else
BOM/non-ASCII:     T21 Encoding Safety     → detect BOM, use .NET WriteAllText, NEVER IDE tools
New codebase:      T18 Intent Mapping       → WHAT/WHY/HOW before touching code
                   T19 Data Flow Tracing    → follow data from creation to consumption
                   T20 Architecture Summary → produce ARCHITECTURE_MAP first
Large file:        T13 Read feature_map     → if project has KI/feature_map, read it first
```

**Fallback order for search:** T1 → T2 → T3 → T4 (never skip T4 before editing)

---

## ═══════════════════════════════════════
## PART H — UI/CSS GUIDELINES
## ═══════════════════════════════════════

- Do NOT create new CSS files unless instructed.
- Reuse existing classes — check architecture first.
- For quick fixes: inline style.
- CSS sibling selectors (`~`) can replace JS state sync.

---

## ═══════════════════════════════════════
## PART I — PRE-SUBMIT CHECKLIST
## ═══════════════════════════════════════

> ✅ AI agent: this section applies to you.

- [ ] Encoding detected (§0E) — BOM status recorded
- [ ] Main backup (.bak) created at session start
- [ ] Secondary backup (.bak2) created before this patch
- [ ] TargetContent copied exactly from view_file
- [ ] Line endings match file format
- [ ] AllowMultiple: false unless clear reason
- [ ] Syntax check passed → .bak2 deleted
- [ ] No dangling references to old names
- [ ] Blast radius analyzed
- [ ] Cascade verified — callers pass correct args
- [ ] Proactive bug hunt completed
- [ ] 🧠 Intent understood before patching
- [ ] 🧠 Data flow consistent after patch

---

## ═══════════════════════════════════════
## PART J — 🌐 LANGUAGE DETECTION LAYER
## ═══════════════════════════════════════

> ✅ AI agent: this section applies to you. Auto-detect language from file extension and use the correct commands below.

## §J1. Syntax Check by Language

| Language | Extension | Syntax Check Command |
|---|---|---|
| JavaScript | `.js`, `.mjs` | `node -c file.js` |
| TypeScript | `.ts`, `.tsx` | `npx tsc --noEmit` |
| Python | `.py` | `python -m py_compile file.py` |
| Go | `.go` | `go vet ./...` |
| Rust | `.rs` | `cargo check` |
| PHP | `.php` | `php -l file.php` |
| PowerShell | `.ps1` | `$null = [System.Management.Automation.Language.Parser]::ParseFile('file.ps1', [ref]$null, [ref]$errs); $errs` |
| Bash | `.sh` | `bash -n file.sh` |
| Ruby | `.rb` | `ruby -c file.rb` |
| C/C++ | `.c`, `.cpp` | `gcc -fsyntax-only file.c` |
| CSS/SCSS | `.css` | (manual review or stylelint) |

## §J2. Structural Scan by Language

| Language | Scan Command |
|---|---|
| JavaScript | `Select-String -Path "file.js" -Pattern "function \|=>"` |
| TypeScript | `Select-String -Path "file.ts" -Pattern "function \|class \|interface \|=>"` |
| Python | `Select-String -Path "file.py" -Pattern "^def \|^class "` |
| Go | `Select-String -Path "file.go" -Pattern "^func "` |
| Rust | `Select-String -Path "file.rs" -Pattern "^fn \|^pub fn \|^impl "` |
| PHP | `Select-String -Path "file.php" -Pattern "function \|class "` |
| PowerShell | `Select-String -Path "file.ps1" -Pattern "^function \|^Function "` |

## §J3. Auto-Detect & Dispatch
```powershell
$ext = [System.IO.Path]::GetExtension("file.xyz")
switch ($ext) {
    ".js"  { node -c "file.xyz" }
    ".ts"  { npx tsc --noEmit }
    ".py"  { python -m py_compile "file.xyz" }
    ".go"  { go vet "./..." }
    ".rs"  { cargo check }
    ".php" { php -l "file.xyz" }
    ".rb"  { ruby -c "file.xyz" }
    ".sh"  { bash -n "file.xyz" }
    ".ps1" {
        $e=$null; $t=$null
        [System.Management.Automation.Language.Parser]::ParseFile(
            (Resolve-Path "file.xyz").Path, [ref]$t, [ref]$e) | Out-Null
        if ($e.Count -eq 0) { "Syntax OK" } else { $e | ForEach-Object { "ERR L$($_.Extent.StartLineNumber): $($_.Message)" } }
    }
    default { Write-Warning "Unknown language: $ext — manual review required" }
}
```

---

*update-helper v3.3 — Windows IDE Edition by PitroyTech · Last updated: 2026-05-03*
