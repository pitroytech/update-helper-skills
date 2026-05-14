<p align="right">
  <b>🇺🇸 English</b> &nbsp;|&nbsp;
  <a href="README.vi.md">🇻🇳 Tiếng Việt</a>
</p>

# 🦾 Update Helper v5 — by PitroyTech

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-5.0-blue.svg)](SKILL.md)

> **The safe and efficient way for AI agents to update existing code.**
> No blind patching. No broken builds. No burned context windows.

**Search first. Read less. Patch narrow. Verify thoroughly. Rollback clean.**

---

## 🤔 What is the problem?

AI agents write new code very fast. But updating existing code is much harder.

Real projects are full of hidden contracts: legacy state, UI handlers, config files, cached data, build artifacts, encoding rules, and decisions made by people who no longer work on the project. Without a standard protocol, an agent can confidently ship a patch that looks correct but breaks the entire real workflow.

Without a clear protocol, agents tend to:

**— Patching without understanding —**

① AI starts patching before knowing what actually controls the behavior → fixes the symptom, misses the cause.

② AI reads the entire file just to find one thing to change → burns tokens, slow, expensive.

③ New session starts → AI no longer knows what was done, which files were changed, or where things stopped → starts over or guesses.

**— Right spot, not enough —**

④ AI makes the UI look right → clicking the button does nothing because the logic behind it was never touched.

⑤ AI fixes one thing → something else crashes because it depended on what just changed.

⑥ AI removes a feature → many other places still call it, silently breaking things.

**— Unintended behavior change —**

⑦ AI finishes a refactor → behavior changes that nobody asked for.

⑧ AI patches based on what you described, not what is actually running → applied to a version that has already changed since then.

**— Every fix makes it worse —**

⑨ One fix fails → AI stacks another fix on top of already broken code → drifts further from the original.

⑩ Something breaks → nothing to rollback to, no clean version to compare against and start over.

**— Encoding —**

⑪ File has Vietnamese / emoji → AI has no proper encoding workflow from the start → corrupted silently, cause unclear, no clear restore path.

**— No Verification —**

⑫ AI says "done" → the file has a syntax error or a line mismatch from a shifted edit → AI does not know, does not check → the build output is broken.

With Update Helper, the agent works completely differently:

✅ Finds the real source of truth before patching: source file, generated output, config, runtime state, wrapper, or reference file

✅ Identifies the exact broken layer before patching: collect → transform → validate → call → parse → apply → persist → render

✅ Preserves encoding and carefully verifies text-sensitive files after writing

✅ Keeps a rollback path while patching, and only cleans up backups after verification passes

✅ Finds anchors, then reads only the 40–160 lines that actually matter instead of loading the entire repo

✅ Maps all behavior before deleting or renaming anything: render → handler → state → storage → verification

✅ Compares assumptions against current code and treats divergence as evidence, not noise

---

## 🔄 How does the agent work differently?

```
User: "fix this" / "remove old UI" / "last patch broke something"
        ↓
Update Helper
        ↓
[Lite]  Find anchor → read range → identify owner →
        state intended edit → backup if needed → patch →
        syntax check → invariant search → report

[Full]  + classify source-of-truth → trace data flow →
        assess blast radius → write with correct encoding →
        UI symmetry checklist → cleanup backup
        ↓
Result: right file, right layer, build pass, clean invariants, rollback path intact
```

**Lite** — small, clear task, 1 file, no encoding risk, no generated/source confusion.

**Full Protocol** — large file, multiple modules, encoding risk, refactor, potentially stale spec, or another agent/human has already touched the repo.

| Situation | What agents usually do | With Update Helper |
|---|---|---|
| UI bug | Fix what is visible | Trace render → handler → state → config before patching |
| Provider/API bug | Swap model, swap key | Separate request / response / post-processing → find the actual failure point |
| Generated file | Edit the running file directly | Find source-of-truth, patch source, rebuild output |
| Large file | Read the whole file | Search anchor → read bounded range |
| Vietnamese/CJK file | Replace with whatever tool is handy | Detect BOM before writing, verify after writing |
| Broken patch | Stack another workaround | Restore `.bak2`, re-read range, patch smaller |
| New agent joins | Read everything from the start | Read existing map/backup/KI → continue immediately |
| Stale spec + new code | Apply old spec to refactored code | Compare spec vs reality, ask before merging |

---

## 🛠️ Installation

### Antigravity / OpenClaw (skills folder)

Clone the repo:

```bash
git clone https://github.com/pitroytech/update-helper-skills.git
```

Copy to your agent's skills directory:

```bash
xcopy /E /I update-helper-skills\skills\update-helper %USERPROFILE%\.gemini\antigravity\skills\update-helper
```

### Claude Code / Cursor / Cline (`.skill` file)

1. Download [`update-helper.skill`](update-helper.skill) from this repo.
2. Import into your agent's skill manager.

### Manual (any agent)

Copy the contents of [`skills/update-helper/SKILL.md`](skills/update-helper/SKILL.md) into your system prompt or `AGENTS.md`.

Once loaded, the agent automatically activates the protocol on triggers: `fix this`, `remove old UI`, `refactor`, `port this`, `last patch broke it`.

---

## 🎯 Real scenarios

### Scenario 1: Remove an old UI feature

> Task: remove the Batch tuning group from the API settings page.

An agent without the skill only removes the visible HTML. Result: the input disappears on screen, but the code behind it still reads the old value, causing errors when saving settings.

An agent with Update Helper runs the full checklist:

```
Search: batch-gemini, batch-zhipu, api-advanced
Read:   render block, CSS, Gemini handler, Zhipu handler, shared config save function
Patch:  remove visible control + default buttons + save reads + null-crash handlers
Verify: npm run verify
Check:  rg "batch-gemini|batch-zhipu|api-advanced" src dist → 0 results
```

### Scenario 2: Patching the output file instead of the source

> Task: change a setting but the userscript does not reflect the change.

An agent without the skill finds the running file and edits it directly. Tests pass. Rebuild — everything is gone.

An agent with Update Helper finds the build script first and classifies the files:

```
src/module.js          → SOURCE — this is the file to edit
dist/app.user.js       → GENERATED — build output, do not edit directly
src/app.user.js        → REFERENCE — old monolith, do not touch

→ Patch source → run npm run build → inspect dist
```

### Scenario 3: Dropdown loses models after testing a provider

> Task: after testing Groq then running RACE with Gemini, the dropdown only shows Gemini models.

An agent without the skill edits the dropdown labels. No effect — labels are not the cause.

An agent with Update Helper traces the data flow:

```
RACE → providerModelPool[provider] → sync flat modelPool → dropdown → scheduler

→ Found: RACE was overwriting the entire modelPool instead of only updating that provider
→ Fix: RACE only updates providerModelPool[provider], dropdown hydrates from the aggregate store
```

### Scenario 4: API returns valid data but still reports failure

> Task: logs show a valid response, but the scheduler still reports failed.

An agent without the skill keeps swapping models, testing keys, switching providers. Nothing works.

An agent with Update Helper separates three layers:

```
Request sent?        → Yes ✅
Response received?   → Yes, valid JSON ✅
Post-processing?     → Crash at apply translation step ❌

→ Root cause: the apply function was deleted in a previous refactor, call site never updated
```

---

## 📸 Real results

### ❌ Before — agent without the skill (12 failed searches in a row)

<img src="images/before-12-searches.png" width="33%" alt="Agent without skill: 12 failed searches">

The agent used `grep_search` which could not handle a 14,000-line UTF-8 BOM file. Conclusion: *"file has an encoding issue, trying something else"* → guessing, burning tokens.

### ✅ After — agent with Update Helper (3 searches, root cause found)

<img src="images/after-trace-success.png" width="33%" alt="Agent with skill: bug found immediately">

Detailed Data Flow Tracing process:

<img src="images/comparison-table.png" width="33%" alt="Detailed Data Flow Tracing process">

```
Search "Reset-LocalAccountPassword"  → line 6732  ✅
Trace caller                         → line 12273 ✅
Root cause: $acc.Username empty → ConvertMsa crash ✅
```

### 📋 Quick comparison

| | Without skill | With Update Helper |
|---|---|---|
| Tool | `grep_search` (IDE) | `Select-String` (PowerShell native) |
| Searches | 12+ failed | 3 targeted → bug found |
| Method | Random guessing | Search → Read → Understand → Trace |
| Result | "encoding issue" | Root cause + 3 bugs identified |
| Encoding | Cannot handle UTF-8 BOM | BOM preserved correctly |

---

## 💰 Real token savings

<img src="images/token-savings.png" width="33%" alt="Token savings estimate with Update Helper v5">

| Task type | Without protocol | With Update Helper |
|---|---|---|
| 500KB+ JS file | Easily 100k–180k tokens | Anchor + bounded range → ~15k–30k tokens |
| UI removal | 2–4 rounds due to missed handlers | 1 round if checklist is complete |
| Source/dist confusion | Patch wrong file, test passes, fix lost on rebuild | Classify first → patch the right file |
| Encoding corruption | May lose the entire session recovering | Detect first, clear restore path |
| Multi-agent handoff | Next agent reads everything from scratch | Continue from existing map/backup |

Strong models still benefit — not because they don't know code, but because the protocol keeps them from small mistakes that are expensive to undo.

---

## 🧪 Proven on

* 15,000+ line userscript: provider routing, scheduler, model pool, floating settings UI
* Translation tool using Gemini, Groq, OpenRouter, SambaNova
* PowerShell/JS files with Vietnamese, CJK, emoji — UTF-8 BOM and no-BOM
* Repos with `src/`, `dist/`, generated userscript, and old monolith reference
* Multi-agent sessions: one agent maps, one patches, one recovers

Every rule in this skill exists because a session without it went wrong.

---

## 🧩 What is in `SKILL.md`?

This README is for humans. `SKILL.md` is what the agent reads when doing real work.

| Feature | Description |
|---|---|
| **Tool Cheat Sheet** | Quick dual-platform command reference: Linux/Claude Code and Windows/PowerShell |
| **Lite Workflow** | 9 steps for clear tasks, with example commands at each step |
| **Full Protocol** | For large files, multiple modules, encoding risk, refactor, or a previously broken patch |
| **Source-of-Truth Detection** | Distinguish source, generated output, and old reference files |
| **Pre-Submit Checklist** | 11 mandatory checks before reporting a patch as done |
| **UI Removal Checklist** | Render, CSS, handler, config, status, consumers, dist |
| **Refactor & Port Protocol** | Leaf → caller → parent → entry point; 5-step port workflow |
| **Backup Cleanup** | Separate process for git workspaces and non-git workspaces |
| **Failure Recovery** | Restore `.bak2`, re-read range, patch smaller |
| **Multi-Agent Handoff** | Read map/backup/KI before continuing, do not overwrite backups |
| **Final Report** | Changed files, behavior, verification, backup state, remaining risk |

---

## 🕰️ Version history

| Version | Changes |
|---|---|
| **v5.0** | Split Lite/Full. Source-of-truth detection. Tool cheat sheet + str_replace diagnostics. Pre-submit checklist. UI removal checklist. Refactor/port protocol. Backup cleanup for git and non-git. feature_map/KI check in multi-agent. Full dual-platform command examples. |
| v4.0 | Self-contained. Encoding-safe .NET write. Multi-agent onboarding. Cascade analysis. Proactive bug hunt. |
| v3.4 | Stale spec detection. Improved BOM write pattern. |
| v3.3 | Data-flow tracing. Architecture summary. JS UI patterns. |
| v3.x | First public releases. |

---

## ⚖️ License

MIT License. **Concept, design, and content © PitroyTech.**

Built from real sessions — not from theory about how agents should work.
