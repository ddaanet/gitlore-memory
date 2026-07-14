---
name: examine-evidence-for-drift-direction-don-t-assume-a-source-of-truth
description: "When two co-maintained artifacts may have diverged, establish which is authoritative from git history, not by assuming a direction"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: b4fe98c4-c5d9-4e86-9302-1be83a10806c
---

When two artifacts are supposed to agree but can diverge — a memory file's `MEMORY.md` index one-liner vs its frontmatter `description`, a spec vs code, a doc vs its implementation — do NOT assume a priori which one is fresher or authoritative. Pull the git history and let the commit pattern show the drift direction: which surface a given change actually touched, and when.

**Why:** In one session I twice asserted a source-of-truth direction from plausible reasoning (first "frontmatter is stale," then "the index one-liner is the living copy") and was corrected both times — "don't assume either way, examine evidence." A `git log -S/-p/--stat` audit then showed the drift is **bidirectional**: sometimes the index line is fresh and the frontmatter stale, sometimes the reverse. Any assumed direction is wrong a large fraction of the time, and building a sync/regeneration on the wrong assumption silently propagates staleness the wrong way.

**How to apply:** For any "which of these two is right?" question about co-maintained content, run the history on *both* surfaces before designing a fix, and present concrete commits (this file changed here, that one didn't) rather than a theory. Only after the evidence establishes the direction — or that there is no single direction — design the mechanism. Related: [[fact-check before incorporating corrections]], [[feedback-spec-vs-code]], [[feedback-verify-handoff-pending]].
