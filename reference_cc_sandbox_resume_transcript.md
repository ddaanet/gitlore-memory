---
name: reference-cc-sandbox-resume-transcript
description: "sandbox makes `~/.claude/projects/` read-only → turn-1 transcript never persists → `--resume` finds no session; single-turn probe passes and misses it; two-turn/N-turn flows must run unsandboxed, so `just prerelease`/`just release` do too — `No conversation found with session ID`, `no session transcript captured` and `direnv: … read-only file system` are the signatures, 0/5 not flaky"
metadata: 
  node_type: memory
  type: reference
  originSessionId: ba06765d-68cc-40dc-a513-1ff1599e7277
  modified: 2026-07-23T15:33:56.703Z
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

**So `just prerelease` and `just release` must run unsandboxed** — `release`
depends on `prerelease`, which runs `just evals`, which is exactly such a flow.
Sandboxed it fails at `pass^5` in a way that looks like it indicts the scenarios
or the code, and does not: every run of an affected scenario fails identically,
0/5 rather than flaky. Two signatures name the cause. Every two-turn scenario
reports `No conversation found with session ID: … turn 2 yielded no result`, and
`05-recall` reports `no session transcript captured` from its assertion, which
reads the transcript to tell an injected body from a self-issued Read. A third
tell is upstream of the evals and visible on every trial: `direnv: error open …
read-only file system` from the install step. Diagnose from these before
suspecting a scenario — and note that piping the gate into `tail` reports the
pipeline's exit status, so a failing run looks like exit 0.
