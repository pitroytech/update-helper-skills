---
name: update-helper
version: "5.0"
description: |
  Safe, high-efficiency code update protocol for AI agents. Use for editing,
  patching, debugging, refactoring, porting, or recovering code changes in real
  projects, especially when files are large, generated/source files may differ,
  UI render and handlers must stay in sync, non-English/BOM text is present,
  multiple agents/humans touched the repo, or a previous update broke behavior.
  Triggers: "fix this", "add feature", "why is this broken", "remove old UI",
  "refactor", "port this", "continue previous agent", "last patch broke it".
  Original by PitroyTech. v5.0.
---

# update-helper v5.0

Purpose: update existing code without code blindness. Search first, read bounded ranges, understand enough flow, patch narrowly, verify with syntax/tests and invariants, and keep a clean recovery path.

Respond in the user's language. Keep code identifiers and commands in English. Default is to execute, not explain the protocol, unless the user asks for a plan or review.

---

## Tool Cheat Sheet

| Need | Linux / Claude Code | Windows / PowerShell |
|---|---|---|
| Find line by keyword | `rg -n "token" file` or `grep -n` | `rg -n` or `Select-String` |
| Read bounded range | `nl -ba file \| sed -n 'N,Mp'` | `$lines[N..M]` via ReadAllLines |
| Replace unique block | `str_replace` (Claude) / patch tool | `apply_patch` / .NET Replace |
| Replace all occurrences | `sed -i 's/old/new/g' file` for simple ASCII only | `-replace` in .NET content string after encoding check |
| Syntax check JS | `node --check file.js` | `node --check file.js` |
| Backup before write | `cp file file.bak2` | `Copy-Item -LiteralPath` |
| Detect encoding | `file -bi file; xxd -l 3 file` | Read first 3 bytes via ReadAllBytes |
| Search in src+dist | `rg -n "token" src dist` | `rg -n "token" src dist` |
| Count brace balance | `grep -o '{' f \| wc -l` | `($c.ToCharArray() \| ? {$_ -eq '{'}).Count` |
| Rebuild artifact | `npm run build` / `npm run verify` | `cmd /c npm run build` |

Do not use `sed -i` or blind global replace for Vietnamese/CJK/emoji/BOM-sensitive files. For those, detect encoding first and use explicit encoding-preserving writes.

**str_replace / patch failure quick fix:**

| Symptom | Cause | Fix |
|---|---|---|
| "string not found" | Whitespace / indent mismatch | Re-read exact range, copy verbatim |
| "string not found" | Line numbers shifted after earlier patch | Re-run search to get new coordinates |
| "string not found" | Included line-number prefix from view | Strip the `N:` or `    N\t` prefix — display only |
| "appears more than once" | Pattern too short | Add 2–3 surrounding lines to make unique |
| "appears more than once" | Want all simple ASCII occurrences | Use a mechanical global replace instead |

---

## Quick Decision

Use **Lite Workflow** only when all are true:

- Small local change: 1 file, 1-3 functions/blocks.
- Target is obvious and current code matches the request.
- No generated/source ambiguity.
- No non-English/BOM/emoji encoding risk.
- No previous failed patch in this session.
- Verification command is clear.

Read and follow the **Full Protocol** when any are true:

- File is 1,000+ lines or unfamiliar.
- Task touches multiple files/modules.
- User reports runtime/build error.
- Previous patch broke something.
- File contains Vietnamese/CJK/emoji/BOM-sensitive text.
- Refactor, port, migration, or legacy cleanup.
- User spec may be stale.
- Another agent/human changed files.
- You cannot explain data flow before editing.

---

## Lite Workflow

Skip nothing.

1. Search anchors with `rg`, `grep -n`, or `Select-String`.
2. Read bounded range around anchors, usually 40-160 lines.
3. Identify owner function/module and direct callers.
4. Say the intended edit in one sentence before writing.
5. Backup if risk is non-trivial: create `.bak2`.
6. Patch narrowly.
7. Run syntax/build/test check.
8. Re-read edited range or search changed anchors.
9. Report changed files, verification, backup state, remaining risk.

Lite hard rules:

- Do not read whole large files.
- Do not trust user-quoted code until current code is verified.
- Do not report done without verification.
- If verification fails, stop normal patching and enter Failure Recovery.

### Lite Examples By Step

These examples are intentionally concrete. Use the same pattern on Windows/PowerShell or Linux/Bash/Claude Code. Prefer `rg`; if unavailable, use `grep -n` or `Select-String`.

#### 1. Search anchors

Use exact symbols from the bug, not broad words.

Windows:

```powershell
rg -n "providerModelPool|modelPool|RaceResults|buildPriorityZoneOptions|renderModelPoolUI" src
rg -n "updateLastActiveTimestamp|Structured failed|translateStructured|apply.*translation" src dist
rg -n "batch-gemini|batch-zhipu|api-advanced|geminiBatchInput|zhipuBatchInput" src dist
```

Linux:

```bash
rg -n "providerModelPool|modelPool|RaceResults|buildPriorityZoneOptions|renderModelPoolUI" src
rg -n "updateLastActiveTimestamp|Structured failed|translateStructured|apply.*translation" src dist
rg -n "batch-gemini|batch-zhipu|api-advanced|geminiBatchInput|zhipuBatchInput" src dist
```

Good cases:

- Dropdown loses Groq after Gemini RACE -> search model pool, provider pool, dropdown render.
- Prompt/response exists but apply fails -> search exact runtime error and apply path.
- Remove old batch/warmup UI -> search old DOM ids, config keys, handlers, status text.

#### 2. Read bounded range

Read only the range around the anchor. Do not read a whole large file.

Windows:

```powershell
$path = "src/dich_taobao/07-floating-ui-settings.js"
$lines = [System.IO.File]::ReadAllLines($path, [System.Text.Encoding]::UTF8)
for ($i = 3200; $i -le 3330; $i++) { '{0,5}: {1}' -f $i, $lines[$i-1] }
```

Linux:

```bash
nl -ba src/dich_taobao/07-floating-ui-settings.js | sed -n '3200,3330p'
```

For runtime translation failures:

```powershell
$path = "src/dich_taobao/priority-zone-engine.js"
$lines = [System.IO.File]::ReadAllLines($path, [System.Text.Encoding]::UTF8)
for ($i = 11800; $i -le 12380; $i++) { '{0,5}: {1}' -f $i, $lines[$i-1] }
```

```bash
nl -ba src/dich_taobao/priority-zone-engine.js | sed -n '11800,12380p'
```

#### 3. Identify owner function/module and direct callers

Map writer -> store -> reader -> UI/runtime consumer before patching.

Windows:

```powershell
rg -n "function .*Pool|const .*Pool|setProviderPool|syncModelPoolFromProviderStore|getActivePool" src/dich_taobao
rg -n "buildPriorityZoneOptions|chat-popup-model|priority-zone-engine" src/dich_taobao
rg -n "07-floating-ui-settings|priority-zone-engine|dich_taobao.user" scripts src dist
```

Linux:

```bash
rg -n "function .*Pool|const .*Pool|setProviderPool|syncModelPoolFromProviderStore|getActivePool" src/dich_taobao
rg -n "buildPriorityZoneOptions|chat-popup-model|priority-zone-engine" src/dich_taobao
rg -n "07-floating-ui-settings|priority-zone-engine|dich_taobao.user" scripts src dist
```

Expected map examples:

```text
RACE -> providerModelPool[provider] -> sync flat modelPool -> dropdown options -> scheduler
TRANSLATE response -> parse JSON -> apply translation -> update UI/cache/logs
Settings render -> DOM id -> event binding -> config save/default/reset
```

#### 4. Say intended edit before writing

Write one sentence to the user or your scratchpad. This prevents blind edits.

Good:

```text
I will make providerModelPool the source of truth, refresh modelPool from all providers after each RACE, and keep dropdown rendering from the aggregate list.
```

Good:

```text
I will remove the old Batch tuning control across render, CSS, event binding, config defaults, and stale status text, then verify old ids have no active hits.
```

Bad:

```text
I will tweak the dropdown.
```

#### 5. Backup when risk is non-trivial

For Lite, backup is optional only when the patch is truly tiny and git can recover it. Create `.bak2` for Unicode, generated artifacts, large files, or any risky UI/config edit.

Windows:

```powershell
Copy-Item -LiteralPath "src/dich_taobao/07-floating-ui-settings.js" -Destination "src/dich_taobao/07-floating-ui-settings.js.bak2" -Force
```

Linux:

```bash
cp src/dich_taobao/07-floating-ui-settings.js src/dich_taobao/07-floating-ui-settings.js.bak2
```

If there is no git, also keep a session backup:

```powershell
Copy-Item -LiteralPath "file.ext" -Destination "file.ext.bak.codex-session-YYYYMMDD-topic" -Force
```

```bash
cp file.ext "file.ext.bak.codex-session-$(date +%Y%m%d)-topic"
```

#### 6. Patch narrowly

Patch the owner block, not every nearby string. For normal repo edits, use the patch tool. For encoding-sensitive full-file rewrites, preserve BOM/no-BOM exactly.

Windows encoding check before risky rewrite:

```powershell
$path = "src/dich_taobao/07-floating-ui-settings.js"
$b = [System.IO.File]::ReadAllBytes($path)
if ($b.Length -ge 3 -and $b[0] -eq 0xEF -and $b[1] -eq 0xBB -and $b[2] -eq 0xBF) { "UTF-8 BOM" }
elseif ($b.Length -ge 2 -and $b[0] -eq 0xFF -and $b[1] -eq 0xFE) { "UTF-16 LE" }
else { "UTF-8 no BOM or ASCII" }
```

Linux encoding check:

```bash
file -bi src/dich_taobao/07-floating-ui-settings.js
xxd -l 3 src/dich_taobao/07-floating-ui-settings.js
```

Patch examples:

- Dropdown bug: patch provider pool writer and dropdown hydration, not only option labels.
- Runtime error after response: patch missing function/call path after response parsing, not API keys.
- UI removal: patch render + handler + config + status text together.

#### 7. Run syntax/build/test check

Prefer the project verification command when known. On JS repos, `cmd /c npm run verify` is often safer in PowerShell than plain `npx`.

Windows:

```powershell
node --check src/dich_taobao/07-floating-ui-settings.js
node --check dist/dich_taobao.user.js
cmd /c npm run verify
```

Linux:

```bash
node --check src/dich_taobao/07-floating-ui-settings.js
node --check dist/dich_taobao.user.js
npm run verify
```

Other language quick checks:

```bash
python3 -m py_compile file.py
go test ./...
cargo check
php -l file.php
ruby -c file.rb
bash -n script.sh
```

#### 8. Re-read edited range or search changed anchors

Prove the intended change exists and stale behavior is gone.

Windows:

```powershell
rg -n "providerModelPool|setProviderPool|syncModelPoolFromProviderStore" src/dich_taobao dist/dich_taobao.user.js
rg -n "clearPoolByProvider\('gemini'\)|clearPoolByProvider\('groq'\)|config\.modelPool = \[" src/dich_taobao
rg -n "batch-gemini|batch-zhipu|api-advanced|geminiBatchInput|zhipuBatchInput" src dist
```

Linux:

```bash
rg -n "providerModelPool|setProviderPool|syncModelPoolFromProviderStore" src/dich_taobao dist/dich_taobao.user.js
rg -n "clearPoolByProvider\('gemini'\)|clearPoolByProvider\('groq'\)|config\.modelPool = \[" src/dich_taobao
rg -n "batch-gemini|batch-zhipu|api-advanced|geminiBatchInput|zhipuBatchInput" src dist
```

Interpretation:

- New owner symbols should exist in source and generated output if generated output is used.
- Removed UI ids/functions should have zero active hits or only documented migration notes.
- If a model returned JSON, failures after that point are runtime/apply failures, not key failures.

#### 9. Report changed files, verification, backup state, risk

Keep final report short and concrete.

```text
Changed       : src/dich_taobao/07-floating-ui-settings.js, dist/dich_taobao.user.js
Behavior      : provider RACE now updates providerModelPool and dropdown hydrates from aggregate store.
Verification  : npm run verify -> OK; invariant search found no old shared-pool overwrite path.
Backup state  : .bak2 deleted after pass; session backup kept at file.ext.bak.codex-session-YYYYMMDD-topic.
Remaining risk: browser click-test recommended for settings dropdown.
```

---

## Pre-Submit Checklist

Run before marking any patch done. Skip only items that provably don't apply.

```text
[ ] Session backup exists for risky, non-git, generated, or encoding-sensitive modified files
[ ] .bak2 created before this write (or git can recover trivially)
[ ] old_str / patch anchor copied from actual file, not user-quoted spec
[ ] Anchor is unique in file (or context added until unique)
[ ] Encoding verified for non-ASCII files; BOM/no-BOM preserved after write
[ ] Re-read edited range — intended change is visible in file
[ ] Syntax/build check passed
[ ] .bak2 deleted after pass (or kept if failed)
[ ] Invariant search: old ids/functions -> 0 hits (or documented exception)
[ ] Cascade check: all callers, consumers, generated output updated
[ ] Report includes: changed files, behavior, verification, backup state, remaining risk
```

---

## FULL PROTOCOL BOUNDARY

Everything below is the full protocol. Use it for risky, unclear, multi-file, generated-file, encoding-sensitive, refactor/port, UI, or recovery work.

---

## 0. Hard Rules

1. Search before reading. Read bounded ranges, not whole large files.
2. Before writing: identify source-of-truth files versus generated outputs/reference files.
3. Detect encoding and create a session backup before the first risky write.
4. Create/refresh `.bak2` immediately before each write.
5. Preserve encoding exactly: UTF-8 BOM stays BOM, UTF-8 no-BOM stays no-BOM.
6. Use stable ASCII anchors when terminal output may show mojibake.
7. Patch render + handler + state + save/config + CSS together when UI controls change.
8. Run syntax/build/tests and invariant searches after meaningful edits.
9. Delete `.bak2` only after verification passes. Clean session backups only under Backup Cleanup Protocol.
10. Never revert unrelated user/human/agent changes.

---

## 1. Source-of-Truth Detection

Before patching, determine which file actually controls runtime.

Search build scripts and generated hints with `rg "build|dist|bundle|generated|do not edit" package.json scripts src .`, then classify files:

```text
SOURCE        : files humans should edit
GENERATED     : build output; regenerate after source edits
REFERENCE     : old monolith/spec/sample; do not patch unless requested
UNKNOWN       : inspect build path before writing
```

Do not patch generated output as the only fix unless the user explicitly wants a one-off artifact edit. If user-run artifact is generated, patch source then rebuild output.

Example:

```text
src/dich_taobao/07-floating-ui-settings.js   SOURCE
src/dich_taobao.user.js                      REFERENCE/stale monolith
dist/dich_taobao.user.js                     GENERATED user artifact
```

---

## 2. Search And Bounded Read

```bash
rg -n "renderApiProviderCard|batch-gemini|api-advanced" src/
grep -n "uniqueToken\|rareFunction" file.js
```

```powershell
Select-String -LiteralPath "file.js" -Pattern "uniqueToken|rareFunction" | Select-Object LineNumber, Line
$lines = [System.IO.File]::ReadAllLines("file.js", [System.Text.Encoding]::UTF8)
for ($i = 1200; $i -le 1280; $i++) { '{0,6}: {1}' -f $i, $lines[$i-1] }
```

If the requested anchor is not found anywhere, stop and ask. The code may have been refactored.

---

## 3. Architecture And Flow Map

Before non-trivial edits, produce a compact map for yourself and, when helpful, share it.

Structural scan:

```bash
rg -n "^function |^const |^class |^async function |^export " file.js
rg -n "addEventListener|\.onclick|\.on\(|saveConfig|render" file.js
rg -n "targetSymbol" src/
```

```powershell
Select-String -LiteralPath "file.js" -Pattern "^function |^const |^class |^async function |^export " | Select-Object LineNumber, Line
Select-String -LiteralPath "file.js" -Pattern "addEventListener|\.onclick|saveConfig|render" | Select-Object LineNumber, Line
```

Map format:

```text
ARCHITECTURE_MAP:
  source-of-truth : src/module.js -> dist/app.js generated by npm run build
  config/state    : config.apiKeys, config.modelPool
  render          : renderSettings()
  handlers        : bindSettingsEvents(), saveConfig()
  flow            : UI input -> state -> API call -> cache/render
  blast radius    : dropdown IDs, event handlers, scheduler consumers
```

Blast-radius questions:

- Direct: what code changes?
- Indirect: who calls/consumes it?
- State: what shared config/global fields are read/written?
- UI: what DOM IDs/classes/events are affected?
- Timing: async callbacks/debounce/race?
- Build: what output must be regenerated?

---

## 4. Safe Edit Protocol

Encoding detection: inspect first bytes for UTF-8 BOM / UTF-16 LE / UTF-8 no BOM before risky rewrites. Session backup, once per target/work stream:

```bash
BAK="file.ext.bak.session-$(date +%Y%m%d)-topic"
[ ! -f "$BAK" ] && cp file.ext "$BAK"
```

```powershell
$sessionBak = "file.ext.bak.codex-session-YYYYMMDD-topic"
if (-not (Test-Path -LiteralPath $sessionBak)) {
    Copy-Item -LiteralPath "file.ext" -Destination $sessionBak -Force
}
```

Pre-write backup:

```bash
cp file.ext file.ext.bak2
```

```powershell
Copy-Item -LiteralPath "file.ext" -Destination "file.ext.bak2" -Force
```

After success:

```bash
rm file.ext.bak2
```

```powershell
Remove-Item -LiteralPath "file.ext.bak2" -Force
```

### Backup Cleanup Protocol

Goal: keep rollback while editing, then remove backup noise only when it is safe.

Always:

- Keep `.bak2` while the current write is unverified.
- Delete `.bak2` only after syntax/build/tests and invariant searches pass.
- If verification fails, restore `.bak2`, re-read the exact range, patch smaller, verify again.
- Never delete the only known-good copy in a non-git workspace.

Git workspace cleanup:

```bash
git status --short
git diff -- file.ext
npm run verify
rm file.ext.bak2
```

```powershell
git status --short
git diff -- "file.ext"
cmd /c npm run verify
Remove-Item -LiteralPath "file.ext.bak2" -Force
```

Session backup cleanup is allowed only when all are true:

- `git status --short` shows the intended changed files and no surprise target-file changes.
- `git diff -- file.ext` is readable and contains the intended patch.
- Verification passed.
- The final answer reports changed files and backup state.

Then either keep the session backup for the user to delete later, or delete it if the user asked for a clean workspace:

```bash
rm "file.ext.bak.codex-session-YYYYMMDD-topic"
```

```powershell
Remove-Item -LiteralPath "file.ext.bak.codex-session-YYYYMMDD-topic" -Force
```

Non-git workspace cleanup:

```bash
sha256sum "file.ext.bak.codex-session-YYYYMMDD-topic" file.ext
npm run verify
rm file.ext.bak2
```

```powershell
Get-FileHash -Algorithm SHA256 "file.ext.bak.codex-session-YYYYMMDD-topic"
Get-FileHash -Algorithm SHA256 "file.ext"
cmd /c npm run verify
Remove-Item -LiteralPath "file.ext.bak2" -Force
```

Keep the session backup by default in non-git workspaces. Delete it only when the user explicitly asks for cleanup or you created a newer verified archival backup. In the final answer, tell the user where the session backup is.

Generated files:

- If source was patched and generated output rebuilt, delete `.bak2` for both only after source verification and generated artifact verification pass.
- Do not keep backup artifacts inside `dist/` longer than the current turn unless the user asks; generated files can usually be rebuilt.

Tool selection: use normal patch tools for small code edits; use explicit Python/.NET encoding writes for full-file or Unicode-sensitive rewrites; patch source then rebuild generated artifacts. Windows encoding-safe rewrite:

```powershell
$path = "C:\path\file.ext"
$b = [System.IO.File]::ReadAllBytes($path)
$hasBom = $b.Length -ge 3 -and $b[0] -eq 0xEF -and $b[1] -eq 0xBB -and $b[2] -eq 0xBF
$content = [System.IO.File]::ReadAllText($path, [System.Text.Encoding]::UTF8)
$nl = if ($content.Contains("`r`n")) { "`r`n" } else { "`n" }

# Modify content using stable ASCII anchors. Avoid terminal-rendered mojibake text.

$utf8 = [System.Text.UTF8Encoding]::new($hasBom)
[System.IO.File]::WriteAllText($path, $content, $utf8)
```

Forbidden for encoding-sensitive files:

```text
Get-Content | Set-Content
Set-Content -Encoding UTF8
WriteAllLines after Get-Content
copying terminal mojibake into source
```

---

## 5. UI Removal And Settings Changes

When adding/removing/changing a UI control, patch all matching paths.

Checklist:

```text
Render/HTML       : element, id, class, label
CSS               : selectors and layout rules
Event binding     : querySelector, onclick, addEventListener
State/config      : save path, defaults, migration, reset buttons
Status/logging    : user feedback and debug output
Consumers         : scheduler/API/cache/dropdowns using that state
Generated output  : rebuild dist if needed
Invariant search  : old id/class/function should be zero unless intentionally kept
```

Removal invariant example:

```bash
rg -n "batch-gemini|batch-zhipu|api-advanced|geminiBatchInput|zhipuBatchInput" src dist
```

If a removed DOM id still appears in active code, the task is not done.

For dynamic API/model settings:

- Keep raw API model IDs separate from friendly display names.
- Do not show key labels A/B/C as the primary model identity unless the user needs key debugging.
- Live provider IDs are for API calls; labels are for UI only.
- If a provider is removed from active registry, check settings UI, scheduler, planner dropdowns, model pool, and cache keys.
- After RACE/test/reload buttons, status should say what happened, not just change the button text.

---

## 6. Refactor And Port Protocol

Use for behavior-preserving restructure, cross-module changes, or moving a feature between projects.

Pre-work:

```bash
rg -n "oldFunction|oldPattern" src/
rg -n "functionBeingChanged" src/
rg -n "import .*module|require\(" src/
```

Build a map before editing:

```text
REFACTOR_MAP:
  source behavior : what currently works
  target behavior : what must remain/change
  dependencies    : imports, globals, config keys, env vars, CSS classes
  call order      : leaf -> callers -> entry point
  verification    : syntax, tests, invariant searches
```

Execution order:

```text
Correct: leaf function -> direct caller -> parent flow -> entry point
Wrong  : entry point first, then chase breakage
```

For ports:

1. List dependencies in source project.
2. Map each dependency to target project equivalent.
3. Identify missing helpers/config/styles.
4. Patch target in small slices.
5. Verify target behavior, not just compile success.

---

## 7. Stale Spec Handling

User specs and previous-agent notes can be stale.

If spec differs from current code:

```text
Spec says      : old shape
Current code   : actual shape
Difference     : what changed
Recommendation : adapt / apply as-is / skip
```

Ask if applying stale spec could destroy current behavior. If the safe adaptation is obvious and local, adapt and mention it. If not obvious, stop and ask.

Never silently merge stale instructions into unfamiliar code.

---

## 8. Failure Recovery

Rollback decision tree:

```text
Syntax error just introduced
  -> restore .bak2 -> re-read exact range -> smaller patch -> verify

Build/test fails after patch
  -> do not patch randomly -> trace failure to changed symbol -> targeted fix -> verify

Runtime wrong behavior, no error
  -> backward trace from bad output -> state mutation -> handler/render path -> targeted fix

Encoding/mojibake corruption
  -> stop -> restore .bak2/session backup -> detect encoding -> rewrite with explicit encoding

Multiple files broken
  -> list broken files -> restore each from its own backup -> redo blast radius before patching
```

Rules:

- Do not add a second workaround on top of a broken first patch.
- Re-read after restore.
- If the same patch fails twice, stop and ask.

---

## 9. Verification

Syntax/build checks: JS `node --check`, TS `npx tsc --noEmit`, Python `python3 -m py_compile`, Go `go test ./...`, Rust `cargo check`, PHP `php -l`, Ruby `ruby -c`, Bash `bash -n`, PowerShell parser. Project verification beats generic syntax check when available:

```bash
npm run verify
npm test
npm run build
```

Post-patch invariants:

```bash
rg -n "oldId|oldFunction|oldConfigKey" src dist       # expect 0 unless intentional
rg -n "newId|newFunction|newConfigKey" src dist       # expect expected hits
```

Cascade checks:

| Change | Verify |
|---|---|
| Rename function | All call sites |
| Change signature | All callers pass correct args |
| Change return shape | All consumers parse new shape |
| Remove DOM id/class | Render, CSS, handlers, tests, dist |
| Change config key | Defaults, migration, readers/writers |
| Change provider/model | UI dropdowns, API calls, scheduler, cache |
| Generated output | Build regenerated artifact |

---

## 10. Multi-Agent Handoff

When joining an in-progress session, run these steps before patching:

1. Verify session backup exists. Create immediately if missing.
2. Check for ARCHITECTURE_MAP or notes from previous agent. If found, read before touching code. If not, run Section 3 Structural Scan.
3. Check for feature_map / knowledge index files:
   ```bash
   find . -maxdepth 3 \( -iname "*feature_map*" -o -iname "*KI*" -o -iname "*knowledge*" \) 2>/dev/null
   ```
   ```powershell
   Get-ChildItem -Recurse | Where-Object { $_.Name -match 'feature_map|knowledge|KI' } | Select-Object FullName
   ```
   If found, read before writing any code.
4. Confirm tool availability: `node`, `python3`, `go`, `npm run verify` as needed.
5. Check git status for changes by other agents/humans: `git status --short`. Re-verify anchors if target files are dirty.

Rules:

- Do not overwrite another agent's session backup.
- If a human changed a file, re-check anchors before applying pending patches.
- Treat previous notes as hints, not truth. Verify against live code.
- If worktree is dirty, do not revert unrelated changes.

---

## 11. Example Scenarios

### A. Remove old settings control safely

Task: remove Batch tuning from API settings.

Correct flow:

```text
Search: batch-gemini, batch-zhipu, api-advanced
Read : render block, CSS block, Gemini handler, Zhipu handler, generic provider save
Patch: remove visible controls, remove default buttons, remove save reads, remove null-crash handlers
Verify: npm run verify
Invariant: rg "batch-gemini|batch-zhipu|api-advanced|geminiBatchInput|zhipuBatchInput" src dist -> 0
```

### B. Fix generated/source confusion

Task: userscript does not reflect settings change.

Correct flow:

```text
Find build script -> identify source chunks -> patch source chunk -> run npm run build/verify -> inspect dist -> leave stale reference untouched
```

### C. Recover from mojibake display

Task: patch Vietnamese UI string, terminal displays garbage.

Correct flow:

```text
Do not patch using garbled terminal text.
Use ASCII anchors around the block.
Detect BOM/no-BOM.
Write with explicit encoding.
Verify known string or invariant after write.
```

### D. API provider cleanup

Task: remove provider or weak models.

Correct flow:

```text
Registry -> settings tabs -> planner provider dropdown -> model filters -> scheduler candidates -> cache/model pool -> status logs -> dist build
```

---

## 12. Concise Final Report

Final response should include only high-signal items:

```text
Changed       : file.js, dist/app.js
Behavior      : removed old Batch UI and dead handlers; moved key visibility toggle beside Add key
Verification  : npm run verify -> OK
Invariants    : old batch IDs/functions -> 0 hits in source/dist
Backup state  : .bak2 deleted after pass; session backup kept/removed per user preference
Remaining risk: browser visual check if UI layout changed
```
