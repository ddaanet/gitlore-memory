---
name: zellij-ipc-sandbox
description: zellij run/action commands need dangerouslyDisableSandbox=true — IPC unix socket is outside the sandbox filesystem view
metadata: 
  node_type: memory
  type: reference
  originSessionId: ce981dd4-19be-4e5d-baa0-0c4f080d1af5
---

In a zellij session (`ZELLIJ_SESSION_NAME=...`), calling `zellij run --floating`, `zellij action new-pane`, etc. from the Claude Code Bash tool returns "There is no active session!" under the default sandbox, even with explicit `--session <name>`. The session exists (`zellij list-sessions` shows "current") but zellij's IPC unix socket (typically under `/tmp/zellij-<uid>/zellij-<session>/`) is not reachable from the sandbox.

**Fix:** call Bash with `dangerouslyDisableSandbox: true`. The first symptom is "no active session" from any zellij subcommand that needs to talk to the running daemon (verified with `zellij --version` 0.44.3).

**Affects:**
- `/revdiff:revdiff` skill launcher (`launch-revdiff.sh`) — it picks the zellij branch when `ZELLIJ` is set and falls silently because the launcher pipes `zellij run` stderr to `/dev/null`. Sandbox failure looks like "exit 1 with no output."
- Any future skill/script that drives zellij from inside CC.

Once sandbox-disabled, the launcher works and the floating pane appears immediately.

Related: [[feedback-git-status-sandbox]] (the same kind of sandbox/host divergence, but for git+submodule artifacts).
