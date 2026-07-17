---
name: feedback_decide_as_late_as_possible
description: "take decisions as late as possible but not later — defer until evidence arrives, yet act before inaction decides by default"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 254e0719-055b-4ee9-93ea-5aafd4d48993
---

"Take decisions as late as possible, but not later." A software-engineering
principle David holds: the longer you defer a decision, the more information
you have, so the better the decision — but there is a hard deadline, the point
where continuing without deciding lets the decision be made *by default*
(through momentum, accreted code, or an emergent default nobody chose).

**Why:** premature decisions bake in a direction before the evidence that
would justify it exists; a hook, schema, or abstraction committed too early
then constrains everything downstream. But deferring past the deadline is not
neutral — the system drifts into a de-facto choice no one reasoned about.

**How to apply:** when a design question isn't yet forced, name it as open,
state what evidence would settle it (e.g. "needs a log analysis of how X
drifts"), and defer — don't manufacture a principle to justify a choice you
haven't earned. Do NOT build code that presupposes the unsettled answer; that
decides by default. But watch the deadline: act before inaction or accreting
code makes the call for you. Live example — D17's presence-authority question
(is the file set or the index authoritative over a line's *presence*?): left
open pending log evidence, and the coverage/prune recompose that presupposed
one answer was reverted as premature (see [[gitlore-global-memory-investigation]]).
Related but distinct: [[feedback_plan_late]] (don't write plans beyond the next
iteration) is the planning-cadence corollary of this decision-timing rule.
