---
name: feedback-harden-human-gates
description: "Apply input-recognition hardening (word-boundary/escape, negation handling) at human free-text gates, not on channels whose producer you control"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1f5cfbea-7d20-4feb-837d-f0a29957091c
---

When tightening how an approval/verdict signal is recognized, first ask **who produces the signal at that point**. Harden where a *human* types free-text; do not over-engineer a channel whose producer is a component you control.

**Why:** In gitlore the approval signal is interpreted at three points: (A) the human confirms a commit summary, (B) the parent escalates a synthesis to the human, (C) the parent agent resumes the `memory-merger` sub-agent with a controlled string. The instruction "add word boundary and escape to approval" (lifted from `docs/references/evals-best-practices.md` §10.4's `is_explicit_approval`, which uses `\b…\b` + `re.escape` + a negation list) first got applied to C — wrong target. At C the producer is the parent agent, which we instruct to send exactly `"approved"`/`"rejected:…"`; fuzzy NL matching there is theatrical. The technique earns its keep only at A and B, where a person might type "not yet", "disapprove", or a hedge that must not be misread as approval (misreading → unapproved memory commits, the load-bearing gate in design.md FR 11).

**How to apply:** Locate every point where the signal is interpreted and label each producer human vs. controlled-component. For controlled producers, prefer an exact reserved sentinel over fuzzy matching (or leave the lean original). For human free-text, apply whole-word + negation-aware recognition: a hedge, a question, or any negation is a rejection; on anything unclear, re-ask rather than proceed. Note the rendering constraint: if no script sits between the human and the agent (CC conversation), the hardening is an *agent instruction*, not literal regex. Related: [[feedback-agent-executes]] (detection logic in scripts when there's a script to host it), [[feedback-spec-vs-code]] (ground spec concepts in real symbols before wiring).
