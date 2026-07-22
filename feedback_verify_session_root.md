---
name: feedback-verify-session-root
description: "after a compaction, check PWD / CLAUDE_PROJECT_DIR / gitStatus before acting; the 2026-07-20 wrong-root incident"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 94e0a066-a634-4a25-a32d-b0f53b992c25
  modified: 2026-07-20T20:09:06.450Z
---

A compaction summary describes **what was being done**, not **where the session
lives**. Resuming from one and continuing to act is how you end up editing a
repository this session does not own.

Incident 2026-07-20: a summary described implementing a feature in
`~/code/handoff`. The session was rooted in `~/code/gitlore` — stated plainly in
the environment block from the start. Work continued in `handoff` for many
turns: edits, tests, four commits. A concurrent session owned that repo. The
misplacement only surfaced when the handoff plugin's own cross-project write
guard refused a `handoff-task.md` write, because the path resolved outside the
session root.

The `/handoff` invocation that triggered the guard had already fired the
activation wipe against **gitlore**, deleting its `handoff-task.md` and staging
the deletion — a file holding live D17 work and four open decisions. Restored
from `HEAD`, but nothing about the wipe announced itself as damage.

**Why:** the summary is vivid and the environment block is quiet, so attention
follows the narrative. Every signal that the root was wrong (`PWD`, an empty
`CLAUDE_PROJECT_DIR`, the gitStatus block) was present and unread the whole
time.

**How to apply:** on the first turn after a compaction, check the session root
against the work the summary describes — `PWD`, `CLAUDE_PROJECT_DIR`, and the
gitStatus block. If they disagree, say so and stop; do not reconcile it by
`cd`-ing. Treat a guard refusal or a "modified by the user or a linter" notice
on a file you did not touch as evidence of a concurrent session, not noise.
Related: [[feedback_no_in_place_other_repos]].
