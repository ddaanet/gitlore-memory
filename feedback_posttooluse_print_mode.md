---
name: feedback_posttooluse_print_mode
description: "under --print, PostToolUse hooks fire and wrapped hookSpecificOutput.additionalContext injects (tested 2.1.212); --print --resume does scripted multi-turn; the SDK (tests/evals on main) is an efficiency choice"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: c1fe8547-2e6f-4cb6-b1a3-c378f4e05ed4
---

**Under `claude --print`, PostToolUse hooks fire and a properly-wrapped `additionalContext` injects.** Verified by direct test on CC 2.1.212: a PostToolUse hook ran under `--print` and its `additionalContext` steered the model's reply. A widespread belief holds the opposite (GitHub [anthropics/claude-code#37559](https://github.com/anthropics/claude-code/issues/37559) reads that way) — test before trusting it.

**`additionalContext` requires `hookSpecificOutput` wrapping.** A hook printing top-level `{"additionalContext": "..."}` is silently dropped; the correct shape is:
```json
{ "hookSpecificOutput": { "hookEventName": "PostToolUse", "additionalContext": "your text" } }
```
This holds in both interactive CC and the Agent SDK. The silent top-level drop is the subtle failure to watch for.

**Scripted multi-turn works under `--print` via `--resume`.** Capture `session_id` from turn 1 (`claude --print --output-format json`), then `claude --print --resume <session_id>` for turn 2; conversation state carries (verified 2026-07-17).

**SDK vs `--print` for evals — an efficiency choice.** Both fire hooks and both do multi-turn. The SDK holds one process across both turns, whereas each `--print` spawn re-primes the full session context (~40k cache-creation tokens) + ~10s startup per turn — so the SDK wins across a scenario × trial matrix. The gitlore eval harness lives in `tests/evals/` on `main` and uses the SDK for that reason.

**Agent SDK details.** `setting_sources=["project"]` loads the repo's `.claude/settings.json` hooks; `permission_mode="bypassPermissions"` is needed for unattended runs (default blocks writes). Two-turn: turn 1 captures `ResultMessage.session_id`, turn 2 passes `resume=<session_id>` in `ClaudeAgentOptions` — NOT `session_id=` (that identifies the current session → "already in use").

**How to apply:** `--print` supports hook-driven flows (2.1.212). Pick the SDK for eval efficiency at scale, `--print --resume` for a lighter scripted harness. See [[reference_cc_subagent_approval]] for two-turn patterns.
