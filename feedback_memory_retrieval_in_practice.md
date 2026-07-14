---
name: feedback_memory_retrieval_in_practice
description: MEMORY.md one-liners are the real memory content; individual files are never read spontaneously — only when explicitly prompted
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8a4f5b31-d418-4b46-ad38-b5635e446776
---

In practice, individual memory files are rarely if ever read spontaneously — the agent answers from what is already in context (MEMORY.md). Update (2026-07-14): the mechanism is now known — CC runs a *separate per-query classifier agent* that attaches ≤5 relevant files, deliberately conservative (see [[cc-memory-retrieval-agentic]]). So the practical under-retrieval is by-design selectivity, not an absent instruction — confirmed empirically (2026-07-14): attachment DOES fire (positive control 100%), but the classifier is probabilistic and conservative (an unindexed subdir file selected only 75%). The classifier keys on BOTH the MEMORY.md one-liners AND per-file frontmatter descriptions; listing a file in MEMORY.md materially boosts its retrieval reliability. See [[cc-memory-retrieval-agentic]] and [[gitlore-global-memory-investigation]].

**Why:** The agent has no observable obligation to Read anything before responding. Pattern-matching descriptions to task context may be instructed by CC's hidden system prompt, but if so the agent does not reliably act on it.

**How to apply:** Treat MEMORY.md one-liners as the effective memory content for practical purposes. Write descriptions dense enough to be actionable on their own. Don't rely on individual file body details being surfaced without explicit user prompting. Reserve detailed file bodies for cases where the user or a skill explicitly triggers a read.

Corollary: the 200-line MEMORY.md cap is the real binding constraint on usable memory in practice.
