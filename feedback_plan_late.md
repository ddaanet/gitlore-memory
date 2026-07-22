---
name: feedback-plan-late
description: "defer until evidence arrives, act before inaction decides by default"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a22fb06e-e320-4708-8c2f-18da35422b19
---

**The general principle: "Take decisions as late as possible, but not later."** The longer you defer a commitment, the more information you have, so the better it will be — but there is a hard deadline, the point where continuing without deciding lets the decision be made *by default* (through momentum, accreted code, or an emergent default nobody chose). Both halves bind: premature decisions bake in a direction before the evidence that would justify it exists and then constrain everything downstream; deferring past the deadline is not neutral either — the system drifts into a de-facto choice no one reasoned about. Applies to design decisions as much as to plans: when a question isn't yet forced, name it as open, state what evidence would settle it (e.g. "needs a log analysis of how X drifts"), and defer — do NOT build code that presupposes the unsettled answer (that decides by default) — but watch the deadline and act before inaction makes the call. Live example — D17's presence-authority question (file set vs. index authoritative over a line's *presence*): left open pending log evidence, and the coverage/prune recompose that presupposed one answer was reverted as premature (see [[gitlore-global-memory-investigation]]).

Two coupled rules that apply this principle to **planning** work in gitlore (and likely beyond):

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
