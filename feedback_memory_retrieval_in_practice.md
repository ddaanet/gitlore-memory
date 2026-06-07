---
name: feedback_memory_retrieval_in_practice
description: MEMORY.md one-liners are the real memory content; individual files are never read spontaneously — only when explicitly prompted
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8a4f5b31-d418-4b46-ad38-b5635e446776
---

In practice, individual memory files are rarely if ever read spontaneously — the agent answers from what is already in context (MEMORY.md). Whether this is because CC provides no retrieval instruction, or because the agent's execution of that instruction is unreliable, is unknown: the full CC system prompt around memory is not visible to the agent.

**Why:** The agent has no observable obligation to Read anything before responding. Pattern-matching descriptions to task context may be instructed by CC's hidden system prompt, but if so the agent does not reliably act on it.

**How to apply:** Treat MEMORY.md one-liners as the effective memory content for practical purposes. Write descriptions dense enough to be actionable on their own. Don't rely on individual file body details being surfaced without explicit user prompting. Reserve detailed file bodies for cases where the user or a skill explicitly triggers a read.

Corollary: the 200-line MEMORY.md cap is the real binding constraint on usable memory in practice.
