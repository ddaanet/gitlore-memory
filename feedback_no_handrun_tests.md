---
name: feedback_no_handrun_tests
description: "never hand-run verification scripts; encode behavior in the bats suite, dogfood the real product"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9df0bd20-256d-4cbe-b942-fddde0f211b9
---

Do not verify script behavior by hand-building a throwaway fixture and running the script with manual assertions in the shell. If a behavior is worth checking, it goes in the test suite (bats here).

**Why:** hand-run checks aren't repeatable, aren't reviewed, and rot; the suite is the durable record. A throwaway fixture that duplicates existing coverage is pure waste.

**How to apply:** before manufacturing a fixture, grep the suite — the path is often already covered (e.g. `resolve_merge_branch.bats` drives divergence→directive→continuation via `run_stub_synth`). Reserve manual runs for *dogfooding the real product* end-to-end ([[feedback_dogfood_early]]), which is distinct from testing a script. Sharpens [[feedback_automate_default]]. Note: LLM prompt contracts (e.g. the memory-merger two-turn handshake) are not bats-testable — verify those only via a real-skill dogfood.
