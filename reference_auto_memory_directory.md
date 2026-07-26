---
name: reference_auto_memory_directory
description: "gitlore points CC's autoMemoryDirectory at the memory/ submodule; set by install + session-start hook, effective next session"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 9df0bd20-256d-4cbe-b942-fddde0f211b9
---

A repo with a gitlore `memory/` submodule also has CC's `autoMemoryDirectory` setting pointing at it — they are inseparable. gitlore writes the setting into `.claude/settings.local.json`:
- `scripts/install/write-settings.sh` — at install.
- `scripts/cc-hooks/session-start.sh` — re-asserted every session start.

So when the auto-memory dir resolves to the submodule, memory writes land there and [[feedback_memory_before_root_commit]] governs your own edits, not just leftovers.

**Non-obvious gotcha:** the harness resolves the memory directory at session *start*, but `session-start.sh` writes `autoMemoryDirectory` during that same startup — so a freshly-installed or just-changed value takes effect only the **next** session. In the interim the harness uses the default encoded path (`~/.claude/projects/<encoded>/memory/`, see [[reference_cc_project_dir_encoding]]). Same start-time-read caching trap as [[reference_cc_agent_discovery]]. Don't conclude the setting is broken from one session's behavior — check whether it predates the session.
