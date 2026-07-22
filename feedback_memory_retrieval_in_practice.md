---
name: feedback_memory_retrieval_in_practice
description: "what CC's passive recall classifier actually attaches; the index is a routing table, not the content"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8a4f5b31-d418-4b46-ad38-b5635e446776
  modified: 2026-07-22T06:21:51.256Z
---

In practice, individual memory files are rarely if ever read spontaneously — the agent answers from what is already in context (MEMORY.md). Update (2026-07-14): the mechanism is now known — CC runs a *separate per-query classifier agent* that attaches ≤5 relevant files, deliberately conservative (see [[cc-memory-retrieval-agentic]]). So the practical under-retrieval is by-design selectivity, not an absent instruction — confirmed empirically (2026-07-14): attachment DOES fire (positive control 100%), but the classifier is probabilistic and conservative (an unindexed subdir file selected only 75%). The classifier keys on BOTH the MEMORY.md one-liners AND per-file frontmatter descriptions; listing a file in MEMORY.md materially boosts its retrieval reliability. See [[cc-memory-retrieval-agentic]] and [[gitlore-global-memory-investigation]].

**Why:** The agent has no observable obligation to Read anything before responding. Pattern-matching descriptions to task context may be instructed by CC's hidden system prompt, but if so the agent does not reliably act on it.

**How to apply:** Write each index line as a *routing key*, not as the fact. It needs the trigger keywords (the error string, the flag, the tool name) so both the classifier and a deliberate scan can decide read-or-skip; it does not need the explanation, which belongs in the body. Passive recall keys on the user prompt and does not re-select later in a conversation, so anything whose trigger emerges mid-task — from a tool result rather than the prompt — reaches context only through an explicit recall checkpoint ([[feedback_recall_checkpoints]], and the checkpoint written into gitlore's `CLAUDE.md`). Unconditional directives don't belong in the index at all: they go to `CLAUDE.md`, or to a path-scoped `.claude/rules/` file when the trigger is "Claude read a matching file."

Corollary: the binding constraint is bytes, not lines — 25KB, hit at 75 lines when lines carry paragraphs. Measured 2026-07-22: one project-state line was 3.3KB (18% of the whole index) and the top five were 31%, while every behavioral directive line together was a small fraction. Curation effort belongs on the long state and reference lines.
