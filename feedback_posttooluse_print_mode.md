---
name: feedback_posttooluse_print_mode
description: PostToolUse hooks don't fire at all in --print mode; hookSpecificOutput wrapping required for additionalContext
metadata: 
  node_type: memory
  type: feedback
  originSessionId: c1fe8547-2e6f-4cb6-b1a3-c378f4e05ed4
---

**`claude --print` suppresses all hooks entirely** — PreToolUse, PostToolUse, everything. Confirmed by GitHub issue [anthropics/claude-code#37559](https://github.com/anthropics/claude-code/issues/37559) (closed as not planned). The hook script never executes; there is nothing to debug.

**`additionalContext` requires `hookSpecificOutput` wrapping.** A shell command hook outputting `{"additionalContext": "..."}` at the top level is silently dropped. The correct format is:
```json
{
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "additionalContext": "your text"
  }
}
```
This applies in both interactive CC and the Agent SDK. `post-tool-use.sh` was fixed to use this format on 2026-05-28.

**Agent SDK (`claude-agent-sdk`) fires hooks correctly** — use `setting_sources=["project"]` to load filesystem hooks from the repo's `.claude/settings.json`. Also requires `permission_mode="bypassPermissions"` for unattended eval runs (default mode blocks file writes with a permission prompt).

**Two-turn SDK flow via `resume=session_id`.** For scenarios where Claude must summarise before committing (hook instructs it to await confirmation), use two calls to `query()`: turn 1 captures the session_id from `ResultMessage.session_id`; turn 2 passes `resume=<session_id>` in `ClaudeAgentOptions`. Do NOT use `session_id=` — that field identifies the current session and causes "already in use" errors.

**How to apply:** For any eval of hook-driven flows, use the Agent SDK, not `claude --print`. See the eval runner at `tests/evals/lib/sdk-runner.py` on the feat-evals branch. See [[reference_cc_subagent_approval]] for context on two-turn patterns.
