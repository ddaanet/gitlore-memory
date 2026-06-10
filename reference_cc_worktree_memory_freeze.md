---
name: reference-cc-worktree-memory-freeze
description: "CC in-process EnterWorktree moves cwd but freezes the launch env (PATH/autoMemoryDirectory/CLAUDE_PROJECT_DIR); auto-memory strands on the launch repo; PostToolUse cwd vs CLAUDE_PROJECT_DIR is the drift signal (D15 basis)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 6fdf0ecf-96bd-4ca8-aab2-d54dc98d915a
---

Claude Code's in-process worktree feature (the `EnterWorktree` tool) is incompatible
with gitlore's launch-time `--settings autoMemoryDirectory` redirect (D10). Verified
empirically 2026-06-10 (CC 2.1.170):

- **`autoMemoryDirectory` resolves once at process startup, absolute** (relative is
  undefined). No hook can change it mid-session.
- **`EnterWorktree` stays in the same process.** It moves the session cwd to the
  worktree, but PATH (front = launch repo's `.gitlore/bin`), `DIRENV_DIR`,
  `CLAUDE_PROJECT_DIR`, and the launch `--settings` `autoMemoryDirectory` all stay
  FROZEN at the launch repo. So auto-memory written from the worktree lands on the
  LAUNCH repo's memory tree/branch, not the worktree's.
- **A fresh `claude` whose cwd is the worktree resolves correctly** — the shim
  recomputes `git rev-parse --show-toplevel`. The only broken path is the in-process
  tool; any fresh shell launch from the worktree dir is fine.
- **Detection (D15):** `PostToolUse` payloads carry the live `cwd` and the hook runs
  in it; `toplevel(cwd) != CLAUDE_PROJECT_DIR` (same git common-dir) flags the drift.
  Shim env (e.g. `GITLORE_LAUNCHED`) IS inherited by hooks, so an exported
  `GITLORE_LAUNCH_ROOT` would also serve as the exact memory-target marker. UNVERIFIED:
  whether `PostToolUse` fires for `EnterWorktree` itself (capture only ever saw
  `tool_name:"Bash"`) — decides targeted `matcher:"EnterWorktree|ExitWorktree"` vs `"*"`.
- **Conversation namespace is per sanitized cwd**, so CC worktrees get their own
  `~/.claude/projects/<encoded>/` (worktrees live under `.claude/worktrees/<name>`).
  `--continue`/`-c` is cwd-project-scoped (won't find a parent-dir session from a
  worktree); `--resume <id>` resumes by id; `--fork-session` forks. CC has native
  `--worktree`/`--tmux` flags.

See [[reference_cc_project_dir_encoding]], [[reference_auto_memory_directory]],
[[reference_plugin_hook_user_channel]].
