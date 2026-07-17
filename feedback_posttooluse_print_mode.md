---
name: feedback_posttooluse_print_mode
description: "under --print, PostToolUse hooks fire and wrapped hookSpecificOutput.additionalContext injects (tested 2.1.212); --print --resume does scripted multi-turn; SDK `query()` is stateless (spawns per call, `resume` replays) so BOTH harnesses re-prime every turn — measured 2026-07-17, SDK $0.379/14.4s vs --print --resume $0.425/24.3s per two-turn trial; pick on dependency footprint, not context reuse"
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

**SDK vs `--print` for evals — a small startup-latency edge, nothing more.** Both fire hooks and both do multi-turn. The SDK's `query()` is **one-shot and stateless** by its own docstring ("each query is independent, no conversation state") — it spawns a subprocess per call and `resume` replays the session, exactly like `--print --resume`. `ClaudeSDKClient` is the SDK's stateful API; gitlore's `tests/evals/lib/sdk-runner.py` does not use it. So **both harnesses re-prime the project context on every turn**; a claim that the SDK "holds one process so context primes once" is false for any `query()`-based runner.

Measured 2026-07-17 (gitlore cwd, trivial two-turn probe, warm cache, per two-turn trial): SDK $0.379 / 14.4s vs `--print --resume` $0.425 / 24.3s. The delta is ~5s startup per turn plus ~7k more context (the CLI loads user settings; the runner passes `setting_sources=["project"]`) — fixed overhead that does not scale with turn length. Cold-cache turn 1 creates ~40k either way, which is where the "~40k per `--print` spawn" figure came from — it is per *turn*, not per harness.

**Agent SDK details.** `setting_sources=["project"]` loads the repo's `.claude/settings.json` hooks; `permission_mode="bypassPermissions"` is needed for unattended runs (default blocks writes). Two-turn: turn 1 captures `ResultMessage.session_id`, turn 2 passes `resume=<session_id>` in `ClaudeAgentOptions` — NOT `session_id=` (that identifies the current session → "already in use").

**How to apply:** `--print` supports hook-driven flows (2.1.212). Both harnesses are viable; pick on dependency footprint, not on context reuse — the SDK buys ~$0.05 and ~10s per two-turn trial and costs a `uv` + Python + `claude-agent-sdk` dependency in an otherwise bash/bats suite. Realising the SDK's *persistent-process* advantage means rewriting the runner onto `ClaudeSDKClient`. See [[reference_cc_subagent_approval]] for two-turn patterns, and [[feedback_verify_delegated_citations]] — this rationale survived two reviews because nobody read `query()`'s docstring.
