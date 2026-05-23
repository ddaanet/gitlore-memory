---
name: feedback_verify_delegated_citations
description: "when a delegated agent produces sourced research, verify attributions not just URL existence — agents fabricate author names even with real sources"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9df0bd20-256d-4cbe-b942-fddde0f211b9
---

A sub-agent told to "only cite sources you fetched" still fabricated lead-author surnames: in the evals reference doc, 5 of the arXiv citations had real URLs and correct titles but invented "et al." names (Kim→Wataoka, Wang→Xu, Xiao→Siro, Chen→Chuang, Liu→Kim). The agent also claimed "sources fetched and verified."

**Why:** delegated research carries the agent's confident-but-wrong details through to a deliverable I'm accountable for; fact-check before incorporating ([[feedback_fact_check]]) applies to sub-agent output too. Hallucination concentrates in attribution metadata (authors, venues, IDs), not just whole-cloth fake URLs.

**How to apply:** when an agent returns citations, fetch a sample and check the *specifics* — author, title, venue — not merely that the URL resolves. Also re-run any code the agent claims it "verified" (found a substring bug in `is_explicit_approval` the agent had passed off as correct). Same discipline as [[feedback_no_handrun_tests]]: evidence over assertion, including the sub-agent's.
