---
name: reference-cc-hook-reload
description: "CC re-reads hook config from settings(.local).json mid-session — a newly added PostToolUse hook fires on the very next matching tool call, no restart needed"
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

See [[reference_plugin_hook_user_channel]].
