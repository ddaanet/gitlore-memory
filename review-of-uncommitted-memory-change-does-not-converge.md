---
name: review-of-uncommitted-memory-change-does-not-converge
description: "`/code-review medium` over an uncommitted memory-store change re-reviews the whole diff each round, so fix rounds never reach lows-only: triage by the fix set and stop by the drop-the-claim rule"
metadata:
  node_type: memory
  type: project
  originSessionId: d2b35b73-4308-478a-846a-8540f8ee9344
  modified: 2026-11-18T10:42:41.084Z
---

A memory-store change (a tier cut, a sweep under `memory/`) cannot land its
fixes as separate commits, because every memory commit goes through the
approved-summary gate, so the per-commit review loop has no fix commit to
review. Each `/code-review medium <path>` pass then covers the whole
uncommitted diff again and mines new ground in the original edit instead of
checking the previous round's fixes: five passes over the 2026-11-17 ddaanet
cut returned 10, 10, 5, 9 and 9 findings, mostly on removals the earlier
rounds had already accepted.

**Why:** the loop's convergence test ("a pass returns only lows") assumes the
reviewed delta shrinks to the fix commit; with one working-tree delta it never
shrinks, and a reviewer asked for findings always returns some.

**How to apply:** triage each pass against the previous round's fix set only,
treat findings on untouched removals as the trade already accepted (or as
briefs to the destination plugin), and when a sentence draws a third round,
drop the claim rather than refine it. Present the summary after that instead of
chasing a lows-only pass. Related: [[feedback-atomic-commits]].
