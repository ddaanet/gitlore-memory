---
name: reference_hourglass_display_caching
description: CC hourglass/idle display time = last API call + 1h; stopped states are empirically cached for 60 min
metadata: 
  node_type: memory
  type: reference
  originSessionId: c6448e7d-daa1-41ce-83a9-ec2174c57e83
---

The time displayed after the hourglass (CC's idle/stopped indicator) is computed as **last API call timestamp + 1 hour**. Stopped states are, empirically, **cached for 60 minutes**.

**Why it matters:** the displayed "next" time is an offset off the last API activity, not an independent clock — and a stopped state persists in cache for ~60 min, so re-checks within that window see the cached state rather than a freshly computed one.

User-reported empirical observation (2026-05-25); not yet cross-checked against harness source. Verify before relying on the exact 1h/60min figures.
