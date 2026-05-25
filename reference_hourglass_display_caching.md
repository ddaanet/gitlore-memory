---
name: reference_hourglass_display_caching
description: CC hourglass displays last-API-call + 1h, which is the conversation-turn TTL (60min); a +5min display is an anomaly, not the rule
metadata: 
  node_type: memory
  type: reference
  originSessionId: c6448e7d-daa1-41ce-83a9-ec2174c57e83
---

The time displayed after the hourglass = **last API call + 1 hour**. That +1h is not arbitrary — it *is* the Claude Code conversation-turn TTL (60 min): a stopped/idle state stays cached for the hour, and the displayed time is that TTL projected off the last API activity. So "+1h displayed" and "~60min cache" are the **same number**, not two separate facts.

**Anomaly to ignore:** a hourglass showing **+5min** (observed 2026-05-25: last call ~11:43 → display 11:48 instead of the correct 12:43) is unrelated to this rule — "whatever, not related to here and now." Don't treat a +5min display as the behavior; the rule is +1h.

**Why it matters:** the displayed time tells you when the cached conversation-turn state expires (last activity + the 60-min TTL). Reason about freshness from +1h, not from a transient +5min readout.
