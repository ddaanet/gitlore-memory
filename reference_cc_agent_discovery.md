---
name: cc-agent-discovery
description: "cwd `agents/` not auto-loaded; plugin must be marketplace-installed OR `--plugin-dir`-loaded for sub-agent dispatch; address as `<plugin>:<agent>`; defs cached per-session"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 42606cdb-1aa3-4a00-a992-3119f3bbf6d4
---

CC plugin sub-agents (files under `<plugin-root>/agents/<name>.md`) are NOT auto-discovered from the cwd alone. The plugin must be reachable via one of:
1. **Marketplace install** (`/plugin install <name>@<scope>` or `enabledPlugins` in `~/.claude/settings.json`), or
2. **`--plugin-dir <path>` flag** on `claude` launch (dev iteration).

**Namespace:** dispatch must use `<plugin-name>:<agent-name>`, not the bare agent name. Confirmed Plan 04 Step 1 (2026-05-22): in a `claude --plugin-dir /Users/david/code/gitlore` session, `subagent_type: "memory-merger"` returned "Agent type not found", but `subagent_type: "gitlore:memory-merger"` dispatched cleanly. The available-agents list in the error message showed `gitlore:memory-merger` alongside the built-ins. So agent discovery via `--plugin-dir` works the same way marketplace install does — plugin prefix is mandatory either way.

Earlier observation (Plan 03 Task 7) that "both forms failed" was from a session that had neither marketplace install nor `--plugin-dir`, so the plugin simply wasn't loaded for agent discovery.

**For other plugins under development:** dev iteration via `claude --plugin-dir <path>` is the cleanest way to test sub-agent dispatch without a full marketplace install. Slash commands and agents both become visible (slash commands appear as `<plugin>:<command>`; agents are addressed as `<plugin>:<agent>`).

**Definitions are cached per-session.** Edits to `agents/<name>.md` or `commands/<plugin>/<name>.md` after session start do NOT take effect for new dispatches in the same session — the harness reads them at start. To verify a contract change end-to-end you must restart `claude`. Discovered Plan 04 Step 3 (2026-05-22): a fresh dispatch after editing the sub-agent file still ran against the old contract text.
