---
name: gitlore project overview
description: What gitlore is, current state, and next steps
type: project
originSessionId: 44430e43-5ab7-4730-8696-8993292db475
---
gitlore is a Claude Code plugin that makes Claude's auto-memory system versioned, shared, and git-backed. Memory lives in a git submodule inside the project repo, synchronized across worktrees via a local `live` trunk branch.

**Why:** Replace the default per-machine `~/.claude/projects/<hash>/memory/` with a tracked, pushable, multi-session memory store.

**Current state (2026-05-25):** Plans 01-05 shipped. Plan 05 (Memory Redirect Launcher / D10) is complete and dogfooded in this repo: the `claude` shim (`scripts/install/launcher-shim`) injects `--settings autoMemoryDirectory` so CC auto-memory lands in the submodule; Placement A (direnv via `.gitlore/bin/claude` + `.envrc`) and Placement B (global via `/gitlore:install-launcher`) both ship; SessionStart emits a launcher guard when `GITLORE_LAUNCHED` is unset; the dead `settings.local.json` `autoMemoryDirectory` writes are removed. 124 tests pass. This session runs with `GITLORE_LAUNCHED=1`.

**Known live divergence (out of Plan 05 scope):** the stranded `~/.claude/projects/-Users-david-code-gitlore/memory/` dir (36 files) is ahead of the submodule `memory/` (30 files) from pre-launcher sessions. Migrating it is a separate one-off.

**Next step:** Plan 06 — `WorktreeCreate`/`WorktreeRemove` hooks (deferred from Plan 05 scope). Plan 04 outer-loop marketplace dogfood is also still pending. Write the plan as late as possible (see [[plan-as-late-as-possible]]).

**How to apply:** When resuming work, read `docs/design.md` first. Note that it's a living design doc — functional reqs, non-functional reqs, architecture, design decisions, rejected alternatives, changelog. Design review already completed; treat the doc as implementation-ready unless a new gap surfaces.
