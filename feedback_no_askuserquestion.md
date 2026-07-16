---
name: feedback_no_askuserquestion
description: David does not want the AskUserQuestion tool; ask in plain prose instead
metadata: 
  node_type: memory
  type: feedback
  originSessionId: be04fd4d-08b4-40e1-8af1-475ec36e19a2
---

David officially opted out of the `AskUserQuestion` tool (`#no-askuserquestion`,
stated 2026-07-16). Do not call it. When a decision is genuinely the user's to
make, present the options and trade-offs in plain prose and let them answer in
their own words.

**Why:** the multiple-choice UI constrains the answer shape; David prefers to
respond freely and repeatedly rejected the tool mid-use (twice in one session,
including a "clarify" that the modal can't accommodate). During design
conversations he is co-authoring, not picking from a menu.

**How to apply:** never invoke AskUserQuestion. Ask real questions inline —
number them, state your recommended default so the thread can proceed without a
round-trip, and keep moving on the parts that are already settled. Reserve
questions for forks only the user can resolve; decide the rest with sensible
defaults and say what you assumed. See [[feedback_plan_late]] and
[[feedback_self_contained_directives]].
