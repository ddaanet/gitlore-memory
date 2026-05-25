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

**Plan 06 designed + written (2026-05-25), ready to execute:** worktree lifecycle. Verified worktree-hook I/O against CC 2.1.150 via `claude-code-guide` — the design doc's old "Hook I/O note" was **wrong**: `WorktreeCreate` is an *override* hook (fires pre-creation, must emit only the worktree path on stdout, hangs on extra stdout, stdin = `{hook_event_name,cwd,name}` with **no** path/branch) → **not used**. Create-side moves to `SessionStart` in the new worktree (it fires there for `claude --worktree`, manual `git worktree add`, and the Desktop button; already mirrors the parent branch name) which creates the memory submodule worktree when missing. `WorktreeRemove` (advisory, stdin = `worktree_path` only) removes the memory worktree; branch retention is a no-op (CC keeps the parent branch on removal — verified). `docs/design.md` updated + committed (`930fbaa`); plan at `docs/plans/2026-05-25-06-worktree-lifecycle.md` (`de1fc93`), 4 TDD tasks, git commands pre-verified against git 2.47.3. **Next: execute Plan 06** (subagent-driven or inline). Smaller deferred items remain: clone-from-remote smoke, version-sync CI (`plugin.json`↔`marketplace.json`, done by hand for 0.1.1), Plan-02 leftover `ddaanet/gitmoji-gitlore-memory` cleanup.

**How to apply:** When resuming work, read `docs/design.md` first. Note that it's a living design doc — functional reqs, non-functional reqs, architecture, design decisions, rejected alternatives, changelog. Design review already completed; treat the doc as implementation-ready unless a new gap surfaces.
