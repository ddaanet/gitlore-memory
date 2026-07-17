---
name: reference-cc-sandbox-resume-transcript
description: CC sandbox makes ~/.claude/projects/ read-only → turn-1 transcript never persists → --resume finds no session; a single-turn probe passes and misses it
metadata: 
  node_type: memory
  type: reference
  originSessionId: ba06765d-68cc-40dc-a513-1ff1599e7277
---

Inside the Claude Code command sandbox the API is reachable, but
`~/.claude/projects/` (where session transcripts are written) is read-only. So a
`claude --print` turn completes and returns a `session_id`, yet its transcript is
never persisted — and a following `claude --print --resume <id>` fails with
"No conversation found with session ID". The Agent SDK's `query()` has the same
failure (it also spawns a subprocess and resumes by id). Verified 2026-07-17:
`touch ~/.claude/projects/.probe` returns "Read-only file system"; the identical
two-turn resume that failed sandboxed returned `subtype: success` unsandboxed.

**Consequence for eval/harness design:** any two-turn (or N-turn) flow that
relies on `--resume` must run **unsandboxed** (`dangerouslyDisableSandbox`, or
David's own shell). A single-turn connectivity **probe cannot detect this** — it
never resumes, so it passes sandboxed while the real flow cannot. gitlore's
`tests/evals/run-evals.sh` documents this in its header and its probe comment.
Related: [[reference-memory-gate-commit-path]] (the gitdir is likewise write-
blocked, which is why the memory-message IPC file lives in the parent worktree).
