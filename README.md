<p align="right">
  <b>🇺🇸 English</b> &nbsp;|&nbsp;
  <a href="README.vi.md">🇻🇳 Tiếng Việt</a>
</p>

# 🛠️ Update Helper — Large File Patcher Protocol

> **The definitive AI-agent skill for safely reading, understanding, and patching large source files — without burning context windows or corrupting encoding.**

An original protocol created, refined, and battle-tested by **PitroyTech** through thousands of real-world edits on large, complex codebases.

---

## ⚡ Why Does This Exist?

Every AI coding agent eventually hits the same wall:

- File is **5,000–30,000+ lines**. Reading it all blows the context window.
- You patch line 800, then line 5,200 is now **line 5,203** — agent patches the wrong block.
- The file has **Vietnamese / CJK / emoji**. A `Set-Content` strips the BOM. Silent corruption, no warning.
- A new agent joins mid-session. It has **zero context** about what was already done.
- Two agents work in **parallel**. One overwrites the other's backup. Both lose rollback.

**Update Helper solves all of these — with a single, self-contained protocol.**

---

## 📊 Real-World: Before vs. After

### ❌ Without Update Helper
Agent using `grep_search` (IDE tool) — **12 failed searches in a row:**

![Agent without skill: 12 failed searches](images/before-12-searches.png?v=3)

→ Agent gives up: *"Hmm, file có vấn đề encoding. Thử cách khác"*  
→ `grep_search` cannot handle 14,000+ line UTF-8 BOM files with Vietnamese.  
→ Agent guesses randomly, no system, burns tokens.

---

### ✅ With Update Helper
Agent switches to `Select-String` (PowerShell native) following **§A4 Data Flow Tracing:**

![Agent with skill: finds bug immediately and traces root cause](images/after-trace-success.png?v=3)

Detailed trace from error → root cause:

![Detailed Data Flow Tracing process](images/comparison-table.png?v=3)

**In 3 targeted searches:**
- Found `Reset-LocalAccountPassword` at line 6732 ✅
- Traced caller at line 12273 ✅
- Identified root cause: `$acc.Username` is empty → `ConvertMsa` crash ✅

---

### 📋 Side-by-Side Comparison

| | Without skill | With `/update-helper` |
|---|---|---|
| Tool | `grep_search` (IDE) | `Select-String` (PowerShell native) |
| Searches | 12+ FAILED | 3 targeted → bug found |
| Method | Random guessing | Search → Read → Understand → Trace |
| Result | "encoding issue" | Root cause + 3 bugs identified |
| Encoding | Cannot handle UTF-8 BOM | BOM preserved correctly |

---

## 💰 Token Savings Estimate

![Token savings estimate with Update Helper v4](images/token-savings.png?v=3)

- **578 KB JS file**: Reading entire file = ~120k–180k tokens. With Update Helper, only anchor zones → ~15k–30k tokens.
- **Standard/weak agents**: **70–90% token savings**
- **Strong models** that already use range reading: **15–30% savings** — main benefit is eliminating encoding corruption, enforcing `.bak2` discipline, and never forgetting `node --check`

---

## ✨ What's Inside

### `skills/update-helper/` — The Core Protocol (v4.0)

A single `SKILL.md` that covers the complete lifecycle of a safe large-file edit:

| Section | What it handles |
|---|---|
| **Hard Rules** | 11 non-negotiable laws that prevent data loss |
| **Fast Workflow** | Exact 10-step execution order for every session |
| **Encoding-Safe Write Pattern** | BOM-preserving `.NET ReadAllText/WriteAllText` for Vietnamese/CJK/emoji files |
| **Backup Protocol** | 2-tier: session backup (stable) + `.bak2` (disposable per-edit) |
| **Code Comprehension** | Structural Scan → Data Flow Trace → Blast Radius Assessment |
| **Anchor Search** | Unique-token search strategy — never trust full-block spec from user |
| **Patch Strategy** | Bottom-to-top multi-patch, stale spec handling, silent merge prevention |
| **Verification** | Syntax check table for 10 languages + post-patch invariant checks |
| **Cascade Analysis** | Every change type mapped to what must be verified downstream |
| **Multi-Agent Onboarding** | 4-step protocol for a new agent joining mid-session |
| **Failure Recovery** | Step-by-step restore for syntax failures and encoding corruption |

### `update-helper.skill` — Claude Code / Cursor / Cline Format

Pre-packaged `.skill` file for direct import into Claude Code, Cursor, or any agent platform that supports skill files. No manual copy-paste needed.

---

## 🤝 Multi-Agent & Multi-Account Use Cases

This is where Update Helper really shines.

### Scenario 1: Agent Handoff
**Agent A** starts patching a 15,000-line JS file, creates a session backup, maps the architecture. **Agent B** joins 30 minutes later with zero memory. Without a protocol, Agent B reads the entire file (context blown), patches wrong lines, overwrites Agent A's backup.

**With Update Helper:** Agent B runs the 4-step onboarding checklist (Section 7):
```
1. Verify session backup exists → found ✅
2. Read ARCHITECTURE_MAP from Agent A → loaded ✅
3. Confirm tools available → node, python ✅
4. Check feature_map / KI files → read ✅
→ "Ready. Session backup confirmed. What's the current task?"
```
No context waste. No broken backups. No duplicate work.

### Scenario 2: Parallel Agents on the Same Codebase
Two agents fix different bugs in the same large file simultaneously. Without coordination, both create `.bak` files with the same name — one silently overwrites the other.

**With Update Helper:** Session backups are timestamped and topic-named:
```
file.js.bak.codex-session-20260505-fix-auth
file.js.bak.codex-session-20260505-fix-ui
```
Both agents can roll back independently. Merging is safe.

### Scenario 3: AI Agent + Human Developer
Human makes a fix directly in the file while agent is mid-session. Agent's `TargetContent` is now stale. Without the protocol, agent patches wrong block, merges silently, produces syntactically valid but logically broken code.

**With Update Helper:** The stale-spec detection rule triggers:
```
⚠️ Spec says: [X]
   Current code is: [Y]
   Difference: [explain]
→ "Apply spec as-is? / Adapt to current code? / Skip this patch?"
No confirmation = no patch.
```

---

## 📦 Installation

### For Antigravity / OpenClaw (Skills folder)
```bash
git clone https://github.com/pitroytech/update-helper-skills.git
# Copy to your agent's skills directory:
xcopy /E /I update-helper-skills\skills\update-helper %USERPROFILE%\.gemini\antigravity\skills\update-helper
```

### For Claude Code / Cursor / Cline (`.skill` file)
1. Download `update-helper.skill` from this repo.
2. Import it into your agent's skill manager.

### Manual (Any agent with system prompt)
Copy the content of `skills/update-helper/SKILL.md` directly into your system prompt or AGENTS.md.

---

## 🧪 Proven Track Record

Built from real sessions patching:
- **30,000+ line PowerShell scripts** with Vietnamese UI strings (UTF-8 BOM required)
- **15,000+ line JavaScript** single-file apps (subtitles, translation overlays)
- **Complex Objective-C Logos tweaks** for iOS jailbreak development
- **Multi-language codebases** (JS, PS1, Python, C/ObjC, Bash)

Every rule in this protocol was added because a session **without it** went wrong — not as a theoretical precaution.

---

## 📋 Quick Reference

```
Hard Rules     → Section 0   (read this first, always)
Fast Workflow  → Section 1   (10-step execution order)
Encoding       → Section 2   (BOM-safe write pattern)
Backups        → Section 3   (2-tier protocol)
Architecture   → Section 4   (Scan → Trace → Blast Radius)
Patching       → Section 5   (stale spec, bottom-to-top)
Verification   → Section 6   (syntax + invariants + cascade)
Multi-Agent    → Section 7   (onboarding protocol)
JS UI Files    → Section 8   (UI-specific patterns)
Recovery       → Section 9   (restore workflow)
Final Report   → Section 10  (what to include)
```

---

## 🔖 Version History

| Version | Changes |
|---|---|
| v4.0 | Refactored to self-contained single file. Upgraded encoding pattern. Hardened multi-agent onboarding. Added proactive bug hunt and cascade analysis tables. |
| v3.4 | Added stale-spec detection protocol. Improved BOM write pattern. |
| v3.3 | Added Data Flow Tracing, Architecture Summary, JS UI patterns. |
| v3.x | Initial public releases. |

---

## 📄 License

MIT License.

**Original concept, design, and all content © PitroyTech.**
Created independently and refined through thousands of real-world large-file edit sessions.
