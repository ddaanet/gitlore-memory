---
name: reference_hook_cannot_inject_tool_calls
description: "no hook output field can force a tool call; `additionalContext` is the substitute — the hook reads the files itself and injects the bytes"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 9eca9443-3081-46d5-a1d8-7f881c0f4d41
  modified: 2026-07-22T06:33:07.511Z
---

No Claude Code hook can inject a `tool_use`, force a `Read`, or enqueue a tool
call. Only the agentic loop generates tool calls. Hooks can block
(`PreToolUse.permissionDecision`, `Stop`/`PostToolBatch` `decision: "block"`),
rewrite input (`updatedInput`), rewrite a result (`updatedToolOutput`), and
inject text (`additionalContext`) — nothing more.

Verified 2026-07-22 against the shipped binary
(`~/.local/share/claude/versions/2.1.217`), not only the docs: zero occurrences
of `injectToolCall`, `requestTool`, `forceRead`, `injectTool`, `forceToolUse`,
`requestedTools`, `injectedToolCalls`. The four `triggerRead` hits are Node
stream internals (`stream._read = triggerRead`). Controls present in the same
grep: `additionalContext` 180, `updatedInput` 249, `permissionDecision` 34,
`PostToolBatch` 46, `InstructionsLoaded` 27, `systemMessage` 54.

**How to apply:** when you want file contents in the model's context
deterministically, don't try to make the agent Read them — have the hook read
them and emit the bytes as `additionalContext`. That is strictly better than a
forced Read: one round trip instead of two, no tool-permission surface, and the
selection stays in a script rather than in agent judgement
([[feedback_agent_executes]]). `additionalContext` is honoured at `PreToolUse`
as well as `PostToolUse` and arrives even when the tool call fails
([[reference_plugin_hook_user_channel]]), so a deny-reason and the injected
bodies can land in the same turn.

The complement — *obliging* the agent to reach the checkpoint at all — is a
separate mechanism: a `PreToolUse` deny on the first durable write of an
episode is self-scoping (no edit, no gate), where a `Stop` block would fire on
every turn including conversational ones.

`InstructionsLoaded` fires when `CLAUDE.md` and `.claude/rules/*.md` load
(matchers `session_start`, `nested_traversal`, `path_glob_match`, `include`,
`compact`) but is side-effects-only: no decision control, exit code ignored.
