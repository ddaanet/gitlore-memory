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

**Released `0.1.1` (2026-05-25):** `ddaanet/gitlore` + `gitlore-memory` submodule pushed to current `main`; `plugin.json`/`marketplace.json` bumped `0.1.0`→`0.1.1` and pushed. `/plugin update` + `/reload-plugins` confirmed live — the flat skill names (`gitlore:install`, `gitlore:install-launcher`, `gitlore:resolve`, no double-prefix) prove the marketplace now serves current code with the launcher (the version bump was required to bust the version-keyed cache — see [[reference_plugin_cache_staleness]]). The only unrun proof is a clean-repo `/gitlore:install` watching launcher emission end-to-end (optional).

**Next step:** Plan 06 — `WorktreeCreate`/`WorktreeRemove` hooks (deferred from Plan 05 scope): memory branches mirror CC's parent-branch policy on worktree create/remove. Branch model already specified in `docs/design.md`; see its "Hook I/O note" (command hooks read `worktree_path`/`worktree_branch` from input; `hookSpecificOutput.worktreePath` is HTTP-hooks-only). Brainstorm scope before writing the plan; write it as late as possible (see [[plan-as-late-as-possible]]). Smaller deferred items: clone-from-remote smoke, version-sync CI (`plugin.json`↔`marketplace.json`, done by hand for 0.1.1), Plan-02 leftover `ddaanet/gitmoji-gitlore-memory` cleanup.

**How to apply:** When resuming work, read `docs/design.md` first. Note that it's a living design doc — functional reqs, non-functional reqs, architecture, design decisions, rejected alternatives, changelog. Design review already completed; treat the doc as implementation-ready unless a new gap surfaces.
