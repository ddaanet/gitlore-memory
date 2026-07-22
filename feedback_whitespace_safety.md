---
name: feedback_whitespace_safety
description: "a whitespace-unsafe parse is never a nit; test against a spaced input before ranking severity; default to NUL-delimited forms"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5f90d295-aa18-451c-8ed2-514d78c5207b
  modified: 2026-07-20T13:40:18.062Z
---

David: "Whitespace safety is essential, don't be amateurish."

**Why:** said after a code review where a whitespace-unsafe parse was reported
as "low, free to fix" — ranked from *reading* the line rather than running it.
Testing showed it was worse than truncation: `git config --get-regexp` piped to
`awk '{print $2}'`, given `[submodule "org tier"]` with
`path = dir with spaces/tier`, emits `tier.path` — a fragment of the **key**,
presented as a path. The caller then operates on a directory that does not
exist. A "cosmetic" ranking would have shipped that.

**How to apply:** treat splitting on whitespace as a defect on sight, not a nit
to schedule. Before assigning severity to any parsing bug, run it against an
input containing a space — severity claims from reading are guesses. Reach for
the NUL-delimited form by default: `git config -z --get-regexp` (records are
`key\nvalue\0`), `find -print0` / `read -r -d ''`, `find -mindepth 1 -maxdepth 1
-print -quit` instead of `ls -A`. When a residual limit remains (newlines in
paths, say), state the bound in a comment rather than implying full coverage.

Related: [[feedback_no_stderr_suppression]], [[reference_git_stderr_and_parsing]].
