---
name: feedback-no-in-place-other-repos
description: "don't edit another repo in place — investigate and propose; David applies changes to his other projects himself"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7ffb103e-27dd-4424-9ec0-a51c5fe7d743
---

When a finding implicates one of David's *other* repos (e.g. `unsandbox-git-status`,
`handoff`, `cwd-safety`), do not edit that repo in place — even when the fix is
small, verified, and the session established the need for it. Investigate, verify,
and *propose*; he applies it himself.

**Why:** the session's working directory is the scope of consent. Another repo has
its own release discipline, its own in-flight work, and its own review path; a
drive-by edit lands uncommitted changes in a tree he did not open. Being right
about the fix does not authorize the edit. Note this is a *broader* rule than
[[reference-cross-repo-push-auth]] (which is about push auth being
classifier-denied): the boundary starts at the working tree, not at the push.

**How to apply:** stay read-only outside the session's project. Report the
diagnosis, the exact change, and the evidence — a patch or quoted snippet in the
reply, not a Write to that path. If the change really must be made from here, ask
first. Corollary: once he says "I'll take it from here," stop touching it; leave
what exists rather than reverting, and say what state it is in.
