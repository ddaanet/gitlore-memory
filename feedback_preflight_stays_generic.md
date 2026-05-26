---
name: feedback-preflight-stays-generic
description: "ddaa:preflight must stay project-agnostic; project-specific release gates live in the project's own Makefile/CI"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 22dc78e0-fb19-470a-aa88-f077c4316f66
---

`ddaa:preflight` is a generic, reusable release-readiness skill shared across all packages. Do NOT add project-specific checks to it. Project-specific release gates (e.g. gitlore's `make check-version` comparing `plugin.json` against the sibling marketplace's `marketplace.json`) belong in that project's own Makefile/CI, invoked at release time.

**Why:** preflight's value is being the same dependable gate everywhere; folding one project's quirks into it makes it lie about other projects and couples the shared skill to a single repo.

**How to apply:** when a new release check is project-specific, wire it into the project's Makefile target (and tests), not into preflight. Only genuinely universal checks (git state, doc audit) go in preflight. Related: [[feedback_automate_default]].
