---
name: feedback-test-the-invocation-path
description: "green means nothing until you know what ran: a 100644 hook shipped past 16 tests invoking it as `bash script.sh`, and 5 suites (21 tests, incl. the FR11 gate) sat unlisted in `make test`; assert discovery + `[ -x ]`, and distrust counts from pipes/concurrent logs"
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

**Same family, one layer up — a suite that is not *run* cannot fail.** 2026-07-15:
`make test` hand-listed every `.bats` file, and five suites (21 tests) had drifted
off the list — including the D12/FR11 memory-gate cover. A load-bearing invariant
sat with zero regression cover and nothing reported it: an unlisted suite is
indistinguishable from a passing one. Fixed by globbing (`wildcard`/`filter-out`)
so discovery cannot drift. Generalization: **green means nothing until you know
what ran** — check the count/plan lines, not just the exit status. Two contracts
worth asserting, not assuming: the suite is *reached* (discovery) and the code is
*invoked as production invokes it* (mode/shebang/PATH).

Corollary on measuring: a pipe (`make test | tail`) reports the *last* stage's exit
status, so a failing suite reads as exit 0; and two concurrent runs redirected to
the same log interleave into a garbage count (both hit here — a bogus "393 green").
Redirect to distinct files, echo `$?` before any pipe, count `^ok`/`^not ok`, and
sanity-check the delta: a number that moves the wrong way is a broken measurement,
not a surprise.

Related: [[feedback_dogfood_early]] (same failure family — fixtures/tests miss what
the real target hits), [[feedback_no_handrun_tests]].
