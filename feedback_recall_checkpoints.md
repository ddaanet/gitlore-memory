---
name: feedback_recall_checkpoints
description: deliberate recall needs embedded checkpoints in calling skills; two natural points in design tasks — early and late
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8a4f5b31-d418-4b46-ad38-b5635e446776
---

Spontaneous recall is nil in practice. Explicit checkpoints embedded in task skills are required, not optional.

**Why:** Confirmed from experience both pre- and post-automemory. The agent does not voluntarily traverse the memory index without explicit instruction at the right moment.

**How to apply:** A recall skill is a composable utility — other skills call it at designated points, not the other way around. For design tasks, two natural checkpoints:
- **Early**: after receiving the spec/input, before exploration. Broad traversal — "what do I already know about this domain?"
- **Late**: after codebase/problem exploration, before writing the design doc. Targeted traversal — context is richest here, so relevance judgments are best.

The late recall matters most: the agent knows what it found during exploration and can match memories to the actual problem shape, not just the initial description.
