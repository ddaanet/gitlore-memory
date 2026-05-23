---
name: large-docs-review
description: "For multi-hundred-line plan/spec docs, don't lean on the user to eyeball the whole thing — surface high-risk sections explicitly first"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ce981dd4-19be-4e5d-baa0-0c4f080d1af5
---

When presenting a large planning or design document for user review, do NOT just commit it and ask "look this over." The user's reaction: "documents are too large for eyeballs" — large docs hide errors in volume, and reviewers (human or LLM) skim instead of inspect.

**Why:** Plan 04 (`docs/plans/2026-05-22-04-marketplace-install.md`) was ~680 lines. The user's first revdiff pass surfaced exactly one annotation (a redundant test layer). They later said they "should have looked for changes and did not" — meaning the volume defeated the review.

**How to apply:**
- Before asking the user to review a long doc, do a fresh-eyes pass yourself and flag specific high-risk sections (concrete shell snippets, cross-repo paths, file lists, anything that requires checking against the world rather than reading prose).
- Present those sections explicitly: "Lines X–Y do Z — worth verifying" beats "review the whole plan."
- For doc edits, prefer `revdiff HEAD~1` (revision-only diff) over `HEAD~N` (cumulative) when the user just needs to verify a targeted change.
- Consider splitting structurally: spec → its own commit; implementation plan → its own commit. Smaller diffs review better than one large drop.
- The brainstorming skill's "spec self-review" step exists for exactly this reason — actually do it with rigor, don't just tick the checkboxes.

Related: [[feedback-fact-check]] (verify claims rather than assert), [[feedback-self-contained-directives]] (the same "be explicit" principle for sub-agent instructions).
