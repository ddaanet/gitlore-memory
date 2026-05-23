---
name: feedback-verify-handoff-pending
description: "Before claiming prior-session feedback is un-actioned, verify against current code/commits — handoff narrative can lag behind the work"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: d4dae832-0bb2-45de-aaf9-12d3e771fc94
---

When a handoff says "user gave feedback X, action pending," do not treat X as outstanding work before checking:

1. The commit log for fixes landed *after* the feedback timestamp.
2. The current state of the relevant code against the described "correct shape."

If both match, the work is already done; the handoff was just written before the previous agent could clean up the narrative.

**Why:** This session's handoff listed the user's "Wrong solution shape" message as the final user prompt, implying the fix in `192d7e8` was rejected and un-revised. In reality, `192d7e8` *was* the corrected shape — the handoff was written right after the corrected commit landed, and the prior agent didn't get to record the resolution. I reported "needs revision" without checking, and the user had to push back ("phase 2 is done done?") to prompt verification.

**How to apply:** On session start, when the handoff names a pending revision/fix/decision, before promising to act on it: (a) `git log` since the feedback timestamp, (b) read the file the feedback was about, (c) compare against the described shape. Only then report status. This is a specific instance of [[feedback-fact-check]] applied to handoff narratives rather than user corrections in real time. Related: [[feedback-plan-late]] — don't carry forward prior-session assumptions about what's "pending"; re-check.
