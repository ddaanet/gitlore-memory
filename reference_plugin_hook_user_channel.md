---
name: reference-plugin-hook-user-channel
description: "CC hook output channels — systemMessage is the user-visible field, additionalContext is model-only, stderr only surfaces on non-zero exit"
metadata: 
  node_type: memory
  type: reference
  originSessionId: f781c2e5-7dc1-474b-818d-0ba97fea0231
---

Claude Code hook output channels, by destination (verified 2026-06-10):

- **`systemMessage`** (top-level field) — the user-visible channel. Works for the
  events gitlore uses (SessionStart, PostToolUse); gitlore's SessionStart
  launcher-guard warning already rides it successfully. This is the field to use
  for anything the human should see.
- **`hookSpecificOutput.additionalContext`** — injected into the model's context
  only; **never echoed to the user**. Silent by design.
- **stdout** — consumed as JSON; not echoed.
- **stderr** — ignored on `exit 0`; on non-zero exit, shown to the user only with
  exit code **2**, or with `--verbose` for other codes (a "hook error" label
  appears). SessionStart is non-blocking — continues regardless of exit code.

Consequence for gitlore (D14): route user-facing notices through `systemMessage`,
not stderr-on-exit-0 and **not** via the agent (`additionalContext` "tell the
user…" is model-dependent on the hot path, against NFR1/D7). A `systemMessage`
can sit alongside an `additionalContext` in the same hook JSON.

See [[feedback_agent_executes]], [[feedback_verify_delegated_citations]].
