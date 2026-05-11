# SESSION HYGIENE PROTOCOL
*How to open and close Claude sessions to maximize continuity and minimize token waste*

---

## Why This Exists

Claude has no persistent memory between sessions beyond what's in `userMemories` (auto-generated summaries) and `memory_user_edits` (permanent injected facts). Project bibles exist to bridge the gap. Without session hygiene, you re-explain context every session and burn tokens on catch-up.

---

## SESSION OPEN — Do This Every Time

### 1. Paste the relevant project bible(s)
At the very start of the conversation, paste the content of the relevant project bible(s). Don't assume Claude will search for them — paste them directly.

For Spider work: paste `SPIDER_PROJECT_BIBLE.md`
For Skope work: paste `SKOPE_PROJECT_BIBLE.md`  
For Cented work: paste `CENTED_PROJECT_BIBLE.md`
For multi-project sessions: paste all relevant bibles

### 2. State the session goal in one sentence
"Today we are: [fixing X bug in Spider / designing Skope v1 architecture / building the Helius feed for Cented]"

This prevents scope drift before it starts.

### 3. Upload current files if doing code work
Claude cannot see your local files. If working on Spider: upload `spider.py`, `content.js`, `background.js`. If you don't upload current files, Claude is working from memory and will produce stale code.

### 4. State any constraints upfront
"Don't touch the scoring logic today — UI only."
"Architecture discussion only — no code until I say go."
"We're continuing from where we left off — the bug is in the pre-migration fallback."

---

## SESSION CLOSE — Do This Every Time for Significant Sessions

### 1. Generate a handover doc
Use the `session-handover` skill. It will capture:
- What was decided
- What was built/changed
- What was explicitly deferred
- Exact next steps
- Any open questions

Save this as a dated file: `SPIDER_HANDOVER_2026-05-11.md`

### 2. Update the relevant project bible if something significant changed
If a major architectural decision was made, a significant bug was found and fixed, or a "never do this again" lesson emerged — update the project bible immediately before closing.

### 3. Update memory_user_edits if needed
If a permanent fact changed (e.g., project status changed, a critical decision was made), update the relevant memory edit. Don't let the memory get stale.

---

## QUICK REFERENCE — What Lives Where

| What | Where | How to access |
|---|---|---|
| Permanent facts (project status, settled decisions, working style) | `memory_user_edits` | Injected automatically every session |
| Auto-generated context (project history, people, timeline) | `userMemories` | Injected automatically every session |
| Project-specific knowledge (architecture, decisions, failures) | Project bibles (markdown files) | **Paste manually at session start** |
| How to work on a specific project (rules, constraints) | Skills in `/mnt/skills/user/` | Auto-triggered by skill descriptions |
| Session-to-session continuity | Handover docs | **Paste manually at session start** |

---

## TOKEN EFFICIENCY RULES

1. **Never paste large documents unless Claude needs to read them.** Reference the project bible instead — it's pre-compressed.
2. **Upload current code files instead of describing what they do.** "Here's spider.py" is worth 500 words of explanation.
3. **State what you want at the start, not the end.** Don't discover the goal through conversation — state it upfront.
4. **Use the arch-spec-writer skill for new features.** It forces architecture-first before any code is written, preventing expensive rework.
5. **If Claude starts drifting scope, interrupt immediately.** "Stop — that's out of scope" costs 5 tokens. Letting Claude write 200 lines of code you didn't ask for costs 2000.

---

## PROJECT BIBLE MAINTENANCE

Project bibles go stale. Keep them current:

**Update after any session that:**
- Changes a fundamental architectural decision
- Adds a "things we tried that failed" lesson
- Changes project status
- Resolves an open question
- Discovers a non-obvious technical constraint

**Don't update for:**
- Minor bug fixes
- UI tweaks
- Routine feature additions that don't change architecture

---

## CURRENT PROJECT BIBLES

| Project | File | Last Updated |
|---|---|---|
| Spider | `SPIDER_PROJECT_BIBLE.md` | May 2026 |
| Skope | `SKOPE_PROJECT_BIBLE.md` | May 2026 |
| Cented Classifier | `CENTED_PROJECT_BIBLE.md` | May 2026 |
| Polymarket Copy Bot | *Not yet written* | — |
| TACO Watch | *Not yet written* | — |
| Beat / Tide | *Not yet written* | — |

---

## SIGNS A SESSION IS GOING WRONG

- Claude is explaining something you already know → paste the project bible, it's missing context
- Claude suggested reverting a settled decision → project bible needs updating, or wasn't pasted
- Claude is writing code you didn't ask for → interrupt, restate scope
- You're re-explaining the same context for the third time → that context needs to be in memory_user_edits
- The session has been going 2 hours with no concrete output → scope was too wide, split into two sessions
