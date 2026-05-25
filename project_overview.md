---
name: gitlore project overview
description: What gitlore is, current state, and next steps
type: project
originSessionId: 44430e43-5ab7-4730-8696-8993292db475
---
gitlore is a Claude Code plugin that makes Claude's auto-memory system versioned, shared, and git-backed. Memory lives in a git submodule inside the project repo, synchronized across worktrees via a local `live` trunk branch.

**Why:** Replace the default per-machine `~/.claude/projects/<hash>/memory/` with a tracked, pushable, multi-session memory store.

**Current state (2026-05-25):** Plans 01-05 shipped. Plan 05 (Memory Redirect Launcher / D10) is complete and dogfooded in this repo: the `claude` shim (`scripts/install/launcher-shim`) injects `--settings autoMemoryDirectory` so CC auto-memory lands in the submodule; Placement A (direnv via `.gitlore/bin/claude` + `.envrc`) and Placement B (global via `/gitlore:install-launcher`) both ship; SessionStart emits a launcher guard when `GITLORE_LAUNCHED` is unset; the dead `settings.local.json` `autoMemoryDirectory` writes are removed. 124 tests pass. This session runs with `GITLORE_LAUNCHED=1`.

**Stranded-dir migration (done 2026-05-25):** the 6 orphan files in the pre-launcher `~/.claude/projects/-Users-david-code-gitlore/memory/` dir were migrated into the submodule `memory/` (all 30 shared files were already byte-identical); index lines added to `MEMORY.md`. No longer divergent.

**Remote state (2026-05-25):** `ddaanet/gitlore` and the `gitlore-memory` submodule are both pushed to current `main` — the launcher is on GitHub, so the marketplace can serve it. Formal marketplace-installed end-to-end verification (push → `/plugin update` → fresh session → `/gitlore:install`) is deferred, not yet run.

**Next step:** Plan 06 — `WorktreeCreate`/`WorktreeRemove` hooks (deferred from Plan 05 scope), and/or the deferred marketplace verification above. Write the plan as late as possible (see [[plan-as-late-as-possible]]).

**How to apply:** When resuming work, read `docs/design.md` first. Note that it's a living design doc — functional reqs, non-functional reqs, architecture, design decisions, rejected alternatives, changelog. Design review already completed; treat the doc as implementation-ready unless a new gap surfaces.
