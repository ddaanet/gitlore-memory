---
name: feedback-outside-in-tdd
description: "When architecture is fixed, prefer outside-in TDD (red e2e first) over bottom-up unit tests, because outside-in produces black-box tests coupled to the contract rather than internals"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7dd65fa7-001f-45cd-a975-fcab4433bdf5
---

When the architecture is already settled (e.g., decided in a design doc), start TDD from a red end-to-end test that names the user-facing contract, then let its failure modes drive what to build next. Don't default to bottom-up unit tests first.

**Why:** Bottom-up unit tests can pass while the integration contract is still broken — they verify internal consistency, not the behavior the user cares about. Outside-in red-first tests are black-box by construction (they assert through the boundary), so:
- the test survives refactors of the internals
- the next failure tells you the next file to write — no speculative helpers
- integration risk (fixtures, stubs, env setup) gets exercised on day 1 instead of being deferred

The user pushed back on a bottom-up recommendation for Plan 02 with this reasoning. Plan 01 had its e2e test added last (`ca1d733`) — so its passing only confirmed unit-test self-consistency, not the contract.

**How to apply:** When the user has a design doc and you're proposing a plan structure, default to outside-in: e2e (red) → fill in by following failures → backfill unit tests at the seams the e2e doesn't reach. Only fall back to bottom-up when the architecture itself is still being discovered. See related: [[feedback-loose-generation]] (constrain at the artifact boundary, not mid-stream).
