---
name: feedback-state-current-truth
description: "present-tense standing truth in memory/docs/comments, no \"used to be\" framing; commit messages exempt"
metadata:
  node_type: memory
  type: feedback
  originSessionId: b02672cf-4763-46ae-8816-16bace1bf471
---

Write memory bodies, code comments, and docs as present-tense standing
truth, not as a correction of a prior version. Phrasings like "no longer
true", "X does now fire (contrary to before)", "not what it was", "the
earlier note said Y" read as museum-keeping: a fresh reader who never held
the old belief doesn't need the rebuttal, and the temporal framing dates
and rots.

**Why:** these artifacts are read as current state, not as a changelog.
Git history already preserves what changed — the diff *is* the changelog.
Repeating "it used to be X" in the standing text just adds noise. David
flagged this twice in one session (a memory body AND a code comment),
explicitly extending it from memories to comments and docs.

**How to apply:** state what is true now, plainly. The one sanctioned
exception is inoculating against a *live* wrong belief still in circulation
— and even then, one present-tense clause ("a widespread belief holds the
opposite; test before trusting it"), never a narration of the old version.
A commit message is exempt: it IS a changelog, so before→after phrasing
belongs there. Distinct but adjacent: don't claim a behaviour *changed*
from a single current-state test — a broken prior config or a different
environment explains it equally, so you can't tell a delta from one
snapshot.

Related: [[feedback_examine_evidence_drift]], [[feedback_fact_check]],
[[feedback_loose_generation]].
