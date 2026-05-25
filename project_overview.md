---
name: gitlore project overview
description: What gitlore is, current state, and next steps
type: project
originSessionId: 44430e43-5ab7-4730-8696-8993292db475
---
gitlore is a Claude Code plugin that makes Claude's auto-memory system versioned, shared, and git-backed. Memory lives in a git submodule inside the project repo, synchronized across worktrees via a local `live` trunk branch.

**Why:** Replace the default per-machine `~/.claude/projects/<hash>/memory/` with a tracked, pushable, multi-session memory store.

**Current state (2026-05-25):** Plans 01-05 shipped. Plan 05 (Memory Redirect Launcher / D10) is complete and dogfooded in this repo: the `claude` shim (`scripts/install/launcher-shim`) injects `--settings autoMemoryDirectory` so CC auto-memory lands in the submodule; Placement A (direnv via `.gitlore/bin/claude` + `.envrc`) and Placement B (global via `/gitlore:install-launcher`) both ship; SessionStart emits a launcher guard when `GITLORE_LAUNCHED` is unset; the dead `settings.local.json` `autoMemoryDirectory` writes are removed. 124 tests pass. This session runs with `GITLORE_LAUNCHED=1`.

**Stranded memory migrated (2026-05-25):** the pre-launcher `~/.claude/projects/-Users-david-code-gitlore/memory/` dir held 6 orphan files never synced into the submodule (the 30 common files were byte-identical). All 6 migrated into `memory/` + indexed; submodule `38e5870`, root gitlink `f376e65`. Divergence resolved.

**Root + submodule pushed (2026-05-25):** `origin/main` (ddaanet/gitlore) now at `f376e65` with the launcher; submodule pushed to `ddaanet/gitlore-memory`. The marketplace cache (`0.1.0`) is still stale (pre-launcher) — refreshing needs a user `/plugin update gitlore` and a *new* session (see [[plugin-cache-staleness]]).

**Deferred — marketplace-installed verification:** the full marketplace path (`/plugin update` → fresh session → `/gitlore:install` in a clean repo → confirm launcher emitted + memory redirects) was deliberately deferred (user chose "just push"). This is the residual half of the Plan 04 outer-loop dogfood, now that the launcher is on GitHub.

**Next step:** either the deferred marketplace verification above, or Plan 06 — `WorktreeCreate`/`WorktreeRemove` hooks (deferred from Plan 05 scope). Write the plan as late as possible (see [[plan-as-late-as-possible]]).

**How to apply:** When resuming work, read `docs/design.md` first. Note that it's a living design doc — functional reqs, non-functional reqs, architecture, design decisions, rejected alternatives, changelog. Design review already completed; treat the doc as implementation-ready unless a new gap surfaces.
