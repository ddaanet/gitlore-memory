---
name: feedback-spec-vs-code
description: Treat the writing-plans phase as where spec assumptions get fact-checked against the codebase — not just transcribed
metadata:
  type: feedback
---

When transitioning from a brainstormed spec to an implementation plan, read the actual codebase symbols the spec refers to before writing test fixtures for them. The spec is generated under conversational pressure and can carry concepts that don't match production semantics.

**Why:** In gitlore Plan 02's brainstorm, I proposed a `gitlore.prepushCommand` config key by analogy with the existing `gitlore.precommitCommand`. The user approved that section via revdiff without annotations. When I started drafting the implementation plan and read `scripts/cc-hooks/post-tool-use.sh`, I found `gitlore.precommitCommand` is not an executor — it's a *prefix-match trigger* for the PostToolUse hook to detect commit-intent. The analogy I'd drawn was meaningless. If I'd transcribed the spec into tests verbatim, I'd have written test fixtures for a config key that has no place in the architecture. Caught it at the writing-plans step only because I read `post-tool-use.sh` to ground the plan in real symbols.

**How to apply:** During writing-plans, when about to write a test fixture or function signature referring to a symbol from the spec, grep for that symbol's actual usage in the codebase first. If the spec's concept doesn't map to anything concrete, fix the spec inline (with a flag in a `> **Architectural correction:**` block) and proceed. Related: [[feedback-fact-check]] (verify user corrections), [[feedback-outside-in-tdd]] (black-box tests survive refactors — but only if the box's interface matches reality).
