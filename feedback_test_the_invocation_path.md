---
name: feedback-test-the-invocation-path
description: "a green suite proves nothing about how production invokes the code — tests calling `bash script.sh` bypass the exec bit that hooks.json depends on; assert the real invocation contract"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 06f8aba2-a8f8-40d2-90ad-d1c6ed14ec84
---

2026-07-15, D17 slice-1: `scripts/cc-hooks/index-sync-post.sh` shipped through 16
green tests, three task reviews, and a lint pass while committed **mode 100644**.
`hooks/hooks.json` invokes hooks **by path**, so in production it would have failed
`Permission denied` (rc=126) on *every* `Write`/`Edit` in *every* gitlore-installed
project. Caught only by the final whole-branch review.

**Why:** every test invoked it as `bash "$POST"`, which **bypasses the executable
bit entirely**. The tests exercised the code; nothing exercised *how production
actually calls it*. The suite was green precisely because it tested a different
invocation path than the real one.

**Why:** a test harness usually invokes code the most convenient way, not the way
the runtime does. Every difference between those two is an untested contract — and
mode bits, shebangs, `$PATH` resolution, and CWD are invisible to a green suite.

**How to apply:** when code is invoked by *something other than your test* (a hook
manifest, cron, systemd, a shell exec), assert the invocation contract itself, not
just the behavior. Concretely here: `[ -x path ]` per hook script. The repo already
had the pattern at `tests/plugin_distribution.bats:69` (added for D15 for this exact
reason) — the lesson had been learned once and not generalized.

Corollary: `git ls-files -s` (recorded mode), not `ls -l` (local filesystem mode) —
a marketplace clone reproduces git's mode, and a local `chmod` can mask the bug.

Related: [[feedback_dogfood_early]] (same failure family — fixtures/tests miss what
the real target hits), [[feedback_no_handrun_tests]].
