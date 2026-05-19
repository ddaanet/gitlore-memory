---
name: loose generation + post-hoc fix for output constraints
description: Don't pressure generation to suppress scaffolding; enforce output constraints in a second pass
type: feedback
originSessionId: cf8270f1-627f-41f0-96b6-295fcfe5ef90
---
For constrained agent output, let generation be loose (scaffolding tokens, priming labels like `**Analysis:**`, etc.) and enforce output constraints via a separate post-hoc pass using the full generated context.

**Why:** Training pressure against scaffolding forces the generator to either compensate with silent reasoning (extra cost) or produce lower-quality output. Scaffolding isn't waste — it's the attention mechanism doing its work; the priming label is attention drafting the content that follows. Separating generator from validator follows the compiler model (parsing vs typechecking). Don't conflate them.

**How to apply:** When designing agent workflows that produce artifacts, don't try to suppress scaffolding at generation time. Agent drafts freely; apply cleanup/linting/review at the artifact boundary (file write, commit, user presentation). Self-review is the cheap form; separate lint is the thorough form. Exception: derived artifacts (compressed summaries from existing context, e.g., commit messages) don't need this — they're already compression, not fresh generation. The compression IS the review.
