<p align="right">
  <b>🇺🇸 English</b> &nbsp;|&nbsp;
  <a href="README.vi.md">🇻🇳 Tiếng Việt</a>
</p>

# 🦾 Update Helper v5 — by PitroyTech

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-5.0.1-blue.svg)](SKILL.md)

The safe, smart way for AI agents to update existing code.
No blind patching. No broken builds. No burned context windows.

Search first. Read less. Patch narrow. Verify hard. Rollback clean.

[🚨 Problem](#-what-is-the-problem) • [🛠️ Installation](#️-installation) • [🧬 Agent Workflow](#-how-does-the-agent-work-differently) • [🎯 Scenarios](#-real-scenarios) • [📸 Results](#-real-results)

---

## 🚨 What is the problem?

AI agents write new code very fast. But updating existing code is much harder.

> Every real project hides a tangled web that's invisible at first glance: legacy state, UI handlers, config, cache, build artifacts, encoding rules — and decisions made by developers or agents from sessions long past. A new agent can't possibly know all of it upfront. And that's exactly when a patch that *looks correct* brings down the entire workflow.
>
> **Update Helper fixes that.** Any agent joining mid-project can follow the protocol, map the hidden web, and patch safely — no memory of previous sessions required.

Without a clear protocol, agents tend to:

**— Patching without understanding —**
- ① AI starts patching before knowing what actually controls the behavior → fixes the symptom, misses the cause.
- ② AI reads the entire file just to find one thing to change → burns tokens, slow, expensive.
- ③ New session starts → AI no longer knows what was done, which files were changed, or where things stopped → starts over or guesses.

**— Right spot, not enough —**
- ④ AI makes the UI look right → clicking the button does nothing because the logic behind it was never touched.
- ⑤ AI fixes one thing → something else crashes because it depended on what just changed.
- ⑥ AI removes a feature → many other places still call it, silently breaking things.

**— Unintended behavior change —**
- ⑦ AI finishes a refactor → behavior changes that nobody asked for.
- ⑧ AI patches based on what you described, not what is actually running → applied to a version that has already changed since then.

**— Every fix makes it worse —**
- ⑨ One fix fails → AI stacks another fix on top of already broken code → drifts further from the original.
- ⑩ Something breaks → nothing to rollback to, no clean version to compare against and start over.

**— Encoding & Verification —**
- ⑪ File has Vietnamese / emoji → AI has no proper encoding workflow from the start → corrupted silently, cause unclear, no clear restore path.
- ⑫ AI says "done" → the file has a syntax error or a line mismatch from a shifted edit → AI does not know, does not check → the build output is broken.

With Update Helper, the agent works completely differently:
- ✅ Finds the real source of truth before patching: source file, generated output, config, runtime state, wrapper, or reference file
- ✅ Identifies the exact broken layer before patching: collect → transform → validate → call → parse → apply → persist → render
- ✅ Preserves encoding and carefully verifies text-sensitive files after writing
- ✅ Keeps a rollback path while patching, and only cleans up backups after verification passes
- ✅ Finds anchors, then reads only the 40–160 lines that actually matter instead of loading the entire repo
- ✅ Maps all behavior before deleting or renaming anything: render → handler → state → storage → verification
- ✅ Compares assumptions against current code and treats divergence as evidence, not noise

---

## 🛠️ Installation

### 1. Agentic IDEs & Local Agents (Cursor, Cline, Windsurf, Antigravity, OpenClaw)

*The easiest way: just give the repo link to your AI agent and let it install itself.*

**Prompt:**
> "Install the update-helper skill from `https://github.com/pitroytech/update-helper-skills.git`"

The agent will automatically clone the repo and copy it to the correct location (e.g., `.cursorrules`, `AGENTS.md`, or its `skills/` folder).

*(For manual setup via terminal)*:
```bash
git clone https://github.com/pitroytech/update-helper-skills.git
```
*Antigravity/OpenClaw:*
```bash
xcopy /E /I update-helper-skills\skills\update-helper %USERPROFILE%\.gemini\antigravity\skills\update-helper
```

### 2. Claude Desktop / Web

1. Download [`update-helper.skill`](update-helper.skill) from this repo.
2. Import it into the built-in **Skill Manager**.

### 3. Manual usage

Copy the contents of [`skills/update-helper/SKILL.md`](skills/update-helper/SKILL.md) and paste it into your system prompt.

### 🔄 Update to latest version

**Via agent (recommended):**
> "Update the update-helper skill from `https://github.com/pitroytech/update-helper-skills.git`"

**Manual update:**
```bash
cd update-helper-skills
git pull origin main
```
*Antigravity/OpenClaw:*
```bash
xcopy /E /I /Y update-helper-skills\skills\update-helper %USERPROFILE%\.gemini\antigravity\skills\update-helper
```

> **⚠️ Note:** After updating, start a **new conversation** for the agent to load the latest version. The current session may still use the cached old version.

> **💡 Note:** Once loaded, the agent automatically activates the protocol on triggers like: `fix this`, `remove old UI`, `refactor`, `port this`, `last patch broke it`.

---

## 🧬 How does the agent work differently?

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

- **Lite** — small, clear task, 1 file, no encoding risk, no generated/source confusion.
- **Full Protocol** — large file, multiple modules, encoding risk, refactor, potentially stale spec, or another agent/human has already touched the repo.

| Situation | What agents usually do | With Update Helper |
|---|---|---|
| **UI bug** | Fix what is visible | Trace render → handler → state → config before patching |
| **Provider/API bug** | Swap model, swap key | Separate request / response / post-processing → find the actual failure point |
| **Generated file** | Edit the running file directly | Find source-of-truth, patch source, rebuild output |
| **Large file** | Read the whole file | Search anchor → read bounded range |
| **Vietnamese/CJK file** | Replace with whatever tool is handy | Detect BOM before writing, verify after writing |
| **Broken patch** | Stack another workaround | Restore `.bak2`, re-read range, patch smaller |
| **New agent joins** | Read everything from the start | Read existing map/backup/KI → continue immediately |
| **Stale spec + new code** | Apply old spec to refactored code | Compare spec vs reality, ask before merging |

---

## 🎯 Real scenarios

### Scenario 1: Remove an old UI feature
> **Task:** remove the Batch tuning group from the API settings page.

**❌ Without skill:** Agent only removes the visible HTML. Result: the input disappears on screen, but the code behind it still reads the old value, causing errors when saving settings.

**✅ With Update Helper:** Agent runs the full checklist:
```
Search: batch-gemini, batch-zhipu, api-advanced
Read:   render block, CSS, Gemini handler, Zhipu handler, shared config save function
Patch:  remove visible control + default buttons + save reads + null-crash handlers
Verify: npm run verify
Check:  rg "batch-gemini|batch-zhipu|api-advanced" src dist → 0 results
```

### Scenario 2: Patching the output file instead of the source
> **Task:** change a setting but the userscript does not reflect the change.

**❌ Without skill:** Agent finds the running file and edits it directly. Tests pass. Rebuild — everything is gone.

**✅ With Update Helper:** Agent finds the build script first and classifies the files:
```
src/module.js          → SOURCE — this is the file to edit
dist/app.user.js       → GENERATED — build output, do not edit directly
src/app.user.js        → REFERENCE — old monolith, do not touch

→ Patch source → run npm run build → inspect dist
```

### Scenario 3: New session, old decisions, wrong file risk
> **Task:** continue a complex translation project after several prior sessions touched source chunks, generated output, context docs, and backups.

**❌ Without skill:** Agent trusts the newest user note or the visible file name, edits the wrong surface, and accidentally drifts away from the previous architecture. The build may pass, but the next session no longer knows which file is source, generated, reference, or stale.

**✅ With Update Helper:** Agent starts by rebuilding the map:
```
Read first: SESSION_BRIEF / CTO_HANDOFF / TASK_LIST / TEST_REPORT
Classify:   src chunks → SOURCE, dist → GENERATED, monolith → REFERENCE
Check:      git status / untracked files / existing backups
Patch:      source only, then rebuild output
Report:     changed files + verification + backup state

→ Result: the next agent continues the work instead of re-discovering the whole project
```

### Scenario 4: API returns valid data, but translation still fails
> **Task:** logs show a valid AI response, but nothing appears correctly in the UI.

**❌ Without skill:** Agent keeps swapping models, testing keys, changing prompts, or switching providers. Nothing works because the API was never the broken layer.

**✅ With Update Helper:** Agent separates the pipeline:
```
Collect input?       → Yes ✅
Request sent?        → Yes ✅
Response received?   → Yes, valid JSON ✅
Parse/validate?      → Yes ✅
Apply/cache/render?  → Fails at exact DOM apply ❌

→ Root cause: response handling was fine; the DOM target/hash no longer matched after a previous refactor
→ Fix: patch the apply/cache layer, not the provider/model layer
```

### Scenario 5: Backup files pile up after long debugging
> **Task:** after several sessions, the repo has `.bak2`, `.bak.codex-session-*`, and renamed skill files everywhere.

**❌ Without skill:** Agent silently deletes backups to make the workspace look clean — including the only good copy of an untracked file.

**✅ With Update Helper:** Agent treats cleanup as a safety-gated step:
```
List:    exact backup candidates, no broad wildcard delete
Verify:  build/check passes
Ask:     user confirms the current behavior is good
Clean:   delete only reviewed backup artifacts
Report:  what was removed and what was intentionally kept

→ Result: rollback safety during active work, clean workspace after the user has tested
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

| Feature | Without skill | With Update Helper |
|---|---|---|
| **Tool** | `grep_search` (IDE) | `Select-String` (PowerShell native) |
| **Searches** | 12+ failed | 3 targeted → bug found |
| **Method** | Random guessing | Search → Read → Understand → Trace |
| **Result** | "encoding issue" | Root cause + 3 bugs identified |
| **Encoding** | Cannot handle UTF-8 BOM | BOM preserved correctly |

### 🗣️ The AI's Own Review (GPT-5.5 class)

After a complex refactoring and porting session spanning dozens of files, an advanced AI agent gave this exact feedback on working with Update Helper:

<img src="images/ai-review-proof.png" width="100%" alt="Real review from AI Agent about Update Helper">

> *"Update Helper does not make me **'code faster'**, but it makes me **break less, verify stronger, and recover better.** Overall efficiency increased by 25-35%."*
> 
> **Core differences:**
> - **Less guessing:** Forces searching anchors, reading ranges, and understanding flows before patching. (Normally, AI jumps in too fast and misses edge cases).
> - **Strict verification:** Always runs verification commands + static searches instead of just assuming "the code looks fine".
> - **Clear Source/Dist boundaries:** Edits `src` and rebuilds `dist`, never patching generated files directly.
> - **Safe in dirty repos:** Prevents accidental reverts of human edits, strictly checks `git status`.
> - **Prevents over-refactoring:** Contains changes to the specific module instead of unnecessarily rewriting the entire system.

---

## 💰 Real token savings

<img src="images/token-savings.png" width="33%" alt="Token savings estimate with Update Helper v5">

### 🗣️ Real evaluation from a massive session (Codex 5.5 High)

During a task to **completely rebuild software from a legacy codebase (over 20,000 lines of code)**, which required deep tracing, port parity, doc updates, and multi-round verification. The AI (Codex 5.5 High) evaluated the actual impact of following the Update Helper protocol:

<img src="images/codex-55-high-review.png" width="100%" alt="Review from Codex 5.5 High">

> **Actual estimates for the recent massive session:**
> - **Productivity increase:** around 35-50%
> - **Wasted token reduction:** around 25-40%
> - **Mis-patch / file corruption risk reduction:** around 50-70%
> 
> *"Main reason: `update-helper` forces the cycle of `search -> read small range -> map v10/v11 -> narrow patch -> verify`. With a large codebase like AIDICH, this avoids mindlessly reading entire 3,000-5,000 line files, prevents 'flow guessing', and reduces the number of re-edits.*
> 
> *It didn't drastically reduce the absolute total tokens in this session, because your request required deep trace + port parity + docs update + multiple verification rounds. Meaning tokens are still high, but tokens are far more 'useful'. Without `update-helper`, this session would highly likely cost around 1.3x-1.6x more time/context, and errors like patching the wrong module or missing dist/docs would be significantly higher."*

### 📋 Savings by task type

| Task type | Without protocol | With Update Helper |
|---|---|---|
| **500KB+ JS file** | Easily 100k–180k tokens | Anchor + bounded range → ~15k–30k tokens |
| **UI removal** | 2–4 rounds due to missed handlers | 1 round if checklist is complete |
| **Source/dist confusion** | Patch wrong file, test passes, fix lost on rebuild | Classify first → patch the right file |
| **Encoding corruption** | May lose the entire session recovering | Detect first, clear restore path |
| **Multi-agent handoff** | Next agent reads everything from scratch | Continue from existing map/backup |

> **💡 Note:** Strong models still benefit — not because they don't know code, but because the protocol keeps them from small mistakes that are expensive to undo.

---

## 🧪 Proven on
- `15,000+` line userscript: provider routing, scheduler, model pool, floating settings UI
- Translation tool using Gemini, Groq, OpenRouter, SambaNova
- PowerShell/JS files with Vietnamese, CJK, emoji — UTF-8 BOM and no-BOM
- Repos with `src/`, `dist/`, generated userscript, and old monolith reference
- Multi-agent sessions: one agent maps, one patches, one recovers

*Every rule in this skill exists because a session without it went wrong.*

---

## 🧠 What is in `SKILL.md`?

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
| **v5.0.1** | **Strict Backup Discipline:** Added Backup Decision Matrix. Mandatory session backups for untracked files (treated as non-git). Prohibited deletion of the only known-good copy of untracked files. Added failure recovery steps for missing backups. Proactive backup cleanup suggestion with strict safety gates after long sessions. |
| **v5.0** | Split Lite/Full. Source-of-truth detection. Tool cheat sheet + str_replace diagnostics. Pre-submit checklist. UI removal checklist. Refactor/port protocol. Backup cleanup for git and non-git. feature_map/KI check in multi-agent. Full dual-platform command examples. |
| **v4.0** | Self-contained. Encoding-safe .NET write. Multi-agent onboarding. Cascade analysis. Proactive bug hunt. |
| **v3.4** | Stale spec detection. Improved BOM write pattern. |
| **v3.3** | Data-flow tracing. Architecture summary. JS UI patterns. |
| **v3.x** | First public releases. |

---

## ⚖️ License
MIT License. **Concept, design, and content © PitroyTech.**

*Built from real sessions — not from theory about how agents should work.*
