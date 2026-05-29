---
name: feedback_handoff_with_commit
description: "when doing handoff + commit, fold the handoff files into the code commit — no separate snapshot commit"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 2a96fe05-a9e9-4a9d-ad56-4f2e36ad3403
---

When the user asks for "handoff and commit" (or handoff alongside committing work), stage the handoff files (`.claude/handoff-task.md`, `.claude/handoff.md`) into the **same** commit as the code work. Do not make a separate `📝 handoff snapshot` commit.

**Why:** The user wants one commit per unit of work, with the handoff snapshot riding along — not a trailing snapshot commit cluttering history.

**How to apply:** Run the handoff skill to write the task file (hook generates `handoff.md`), then `git add` the handoff files together with the code changes and make a single commit with the feature/fix message. See [[feedback_handoff_files_managed]] — still write handoff files only via the skill, never hand-edit.
