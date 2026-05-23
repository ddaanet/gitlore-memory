---
name: feedback-automate-default
description: Default to automated tests; manual tests only when automation costs disproportionately more than the value it adds
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 03a0d3e5-4434-42b0-9d92-9fb1800fa8e8
---

Default to automated tests; reserve manual ones for cases where automation requires disproportionate effort.

**Why:** Manual tests don't compound — each is a recurring tax. Automated tests pay once and run forever. Manual is also where "we forgot to retest that" lives. Yesterday's "too hard to automate" often becomes trivial once a stub or fixture exists.

**How to apply:** When a plan includes a "dogfood" / manual gate, first sketch the automated equivalent (stub the LLM-driven part, mock external services, build a hermetic fixture). Pick manual only when the automation effort is meaningfully larger than the value it adds — typically the kind of judgment that can't be expressed deterministically (e.g., "is the merged content semantically coherent?"). Re-evaluate at each plan: yesterday's manual gate may now be cheap to automate because the surrounding infrastructure exists.

Related: [[feedback-dogfood-early]] — dogfooding still catches the unknown unknowns automation can't anticipate, but its findings should be encoded as fixtures in the next plan rather than re-run manually forever.
