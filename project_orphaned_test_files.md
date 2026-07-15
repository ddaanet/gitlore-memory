---
name: project-orphaned-test-files
description: "five bats files exist but are unreferenced by the Makefile, so make test never runs them — includes the D12/FR11 memory-gate suite; all 21 pass, so the gap is silent"
metadata: 
  node_type: memory
  type: project
  originSessionId: 06f8aba2-a8f8-40d2-90ad-d1c6ed14ec84
---

Found 2026-07-15 (while chasing a 223-vs-211 test-count discrepancy between two
agent reports). Five `tests/*.bats` files exist on disk but are **not referenced
by the Makefile**, so `make test` never runs them:

```
tests/commit_memory.bats              (2f805c9, 2026-06-12)
tests/emit_memory_gate.bats           (f8a6ea5, 2026-06-09 — D12 FR11-bypass gate)
tests/git_hook_memory_pre_commit.bats (f88c7bf, 2026-06-12)
tests/integration_memory_gate.bats    (f88c7bf, 2026-06-12)
tests/write_settings.bats             (ca3a43c, 2026-06-12)
```

**Why it matters:** three of them cover the **D12 / FR11 memory-commit gate** — the
safety mechanism the whole memory-approval design rests on — and it therefore has
zero regression protection in `make test`/CI. They were added across 2026-06-09..12
and never wired in.

**Not currently broken:** verified all 21 tests pass
(`bats tests/commit_memory.bats tests/emit_memory_gate.bats
tests/git_hook_memory_pre_commit.bats tests/write_settings.bats
tests/integration_memory_gate.bats` → exit 0, 21 ok, 0 not ok). So this is a
missing *guard*, not a live failure — which is exactly why it stayed invisible.

**Fix:** one-line Makefile change — append the four unit files to `test-unit` and
`integration_memory_gate.bats` to `test-integration`. Deliberately left undone on
the D17 slice-1 branch to keep those commits scoped.

**How to detect recurrence:** `comm -23` of (basenames of `tests/*.bats`) against
(basenames grepped out of the Makefile). Worth considering as a lint/test guard so
a new bats file can't be added without being wired in.

Related: [[feedback_no_handrun_tests]], [[reference_cc_hook_user_channel]].
