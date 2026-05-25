---
name: handoff-files-managed
description: "don't hand-edit .claude/handoff*.md mid-session; they're managed by the handoff skill + its hooks"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: fba96ba4-0800-400a-b860-509be142f52d
---

Do not ad-hoc edit `.claude/handoff-task.md` or `.claude/handoff.md` during a session. Write `handoff-task.md` ONLY through the handoff skill's protocol (the skill writes that exact path; hooks wipe-before-write and generate `handoff.md`).

**Why:** the user rejected a mid-session direct edit to `handoff-task.md` and noted a PreToolUse guard against such edits belongs in the handoff plugin itself. The files are tooling-managed state, not free-form scratch.

**How to apply:** when you want to refresh handoff content, invoke the handoff skill rather than editing the files; leave them untouched otherwise.
