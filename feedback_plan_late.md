---
name: feedback-plan-late
description: "Write plans as late as possible; don't plan beyond the next iteration"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a22fb06e-e320-4708-8c2f-18da35422b19
---

Two coupled rules for planning work in gitlore (and likely beyond):

1. **Write plans as late as possible.** You have the most information right before you execute. Plans written far ahead drift, encode wrong assumptions, and waste effort when reality differs.
2. **Agile means not planning ahead of the next iteration.** Plan the iteration in front of you; the one after that gets planned after this one ships.

**Why:** When I asked "shall I draft Plans 02–05 so the full roadmap exists before any code is written?" after delivering Plan 01, the user pushed back. Pre-writing Plans 02-05 would bake in assumptions from a state where zero code exists. The right time to write Plan 02 is when Plan 01 is done — by then, the implementation has revealed which design assumptions held and which didn't.

**How to apply:**
- After completing one implementation plan, **stop**. Don't offer to draft the next plan. Don't outline the next plan. Don't sketch the next plan.
- If asked "what's next?", answer in one or two sentences (the next iteration's *deliverable*), not as a fleshed-out plan.
- When proposing a multi-plan decomposition up-front, present it as a *map of subsystems* (one line each), not as plans-in-waiting.
- When the current plan ships, *then* write the next plan, informed by what was learned.
- Pre-writing is OK only when explicitly asked.

Related: [[feedback-loose-generation]] (let the generator scaffold freely, constrain at the artifact boundary) — the analogue here is "don't constrain future iterations with present knowledge."
