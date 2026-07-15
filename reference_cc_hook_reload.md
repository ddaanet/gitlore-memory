---
name: reference-cc-hook-reload
description: "CC re-reads hook config from settings(.local).json mid-session, but NOT a plugin's hooks.json — there, script bodies are live while event registration is frozen at session start"
metadata:
  node_type: memory
  type: reference
  originSessionId: a95591b9-8bc1-40e4-8ed3-e54dbed7b4ca
---

Claude Code re-reads hook configuration from `settings.local.json` **mid-session** —
a hook added during a live session takes effect on the **very next** matching tool
call, with no `/clear` or restart. Verified 2026-06-10 (CC 2.1.x): added a `PostToolUse
matcher:"Bash"` hook via an Edit, then a Bash call immediately fired it (its payload
captured the same call's `tool_input`).

This **corrects** the common assumption (and a prior gitlore handoff) that "hook
matchers are read only at session start, so you can't test a hook-config change in the
session that makes it." You can: edit the settings hook block, then trigger the matched
tool, then inspect the result — all in one session. This is how D15's
`EnterWorktree|ExitWorktree` matcher firing was confirmed without a restart (see
[[reference_cc_worktree_memory_freeze]]).

Caveat on timing: a `PostToolUse` hook fires **after** the triggering tool returns, so a
single Bash call cannot observe its own hook in the same invocation — check the
side-effect (log file) on the *next* call. (Distinct from `autoMemoryDirectory`, which
genuinely resolves once at startup and no hook can change — see
[[reference_auto_memory_directory]].)

## Plugin `hooks.json` does NOT reload — and the split is not obvious

The reload above is a claim about **`settings.local.json`**. A **plugin's own
`hooks/hooks.json` does not reload mid-session**, even under `--plugin-dir .`, where the
plugin loads straight from the checkout you are editing. Observed 2026-07-15 (CC 2.1.210)
while moving gitlore's `index-sync-post.sh` from `PostToolUse(Write|Edit)` to
`PostToolBatch`: after editing `hooks.json`, a live `Edit` to `memory/MEMORY.md` fired the
(unchanged) `PreToolUse` hook and left its stash, but the post hook never ran as
`PostToolBatch` — no frontmatter rewrite, stash unconsumed.

The split that makes this confusing:

- **Script bodies are read per invocation** — edit a hook script and the next firing runs
  the new code, no restart. My rewritten script was live.
- **Event registration is read at session start** — so the *code* was new while the *event
  routing it to* was old. The hook was handed a `PostToolUse`-shaped payload, found no
  `.tool_calls[]`, and silently no-op'd.

Failure signature: a hook that looks dead for no reason right after you move it to a
different event. It isn't dead; it's being called on the old event with the old payload
shape. Don't debug the script — start a new session.

Corollary for probing: you cannot test a plugin hook-*event* change in the session that
makes it, only a hook-*logic* change. Testing an event change via `settings.local.json`
instead is blocked by the auto-mode classifier as self-modification (it reads as the agent
editing its own config), so that route needs the user's explicit go-ahead.

Not verified: whether the *old* registration still fires after the edit. A no-op and a
non-call look identical from outside, and both were consistent with what I saw.

See [[reference_plugin_hook_user_channel]], [[reference_plugin_cache_staleness]].
