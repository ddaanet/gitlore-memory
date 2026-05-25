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

**Plan 06 superseded → D11 design (2026-05-25):** executing Plan 06 (worktree lifecycle) surfaced a deeper flaw, verified empirically against git 2.47.3: gitlore's hook wrappers hardcoded the relative path `.git/gitlore-<hook>`, which only resolves when `.git` is a directory. In a linked worktree `.git` is a gitlink *file*, so the write aborts SessionStart and the shared wired hook's `exec` **blocks the commit outright** — linked worktrees were unusable at a more basic level than "memory missing." Fix is **D11** (in `docs/design.md`, commit `98f1511`): anchor wrappers at `$(git rev-parse --git-common-dir)/gitlore-<hook>` (shared across worktrees) on both write and exec sides, across **all five** hook managers (direct via `--git-path hooks/<hook>`; husky `exec`; lefthook `run:`; overcommit `sh -c` array; manual). Plus early `[ -e "$mempath/.git" ] || exit 0` guards in `pre-commit`/`pre-push`. Plan 06's two deliverables (SessionStart memory-worktree creation; advisory `WorktreeRemove`) are absorbed. The old Plan-06 file (`docs/plans/2026-05-25-06-worktree-lifecycle.md`, `de1fc93`) is **stale** — do not execute it.

**D11 implemented and tested (2026-05-25):** Plan 07 executed — all 10 tasks committed, full suite green (134 tests + 1 integration). All five hook managers anchor wrappers at `$(git rev-parse --git-common-dir)/gitlore-<hook>`; SessionStart lazily creates the memory worktree in linked worktrees; advisory `WorktreeRemove` hook removes it. Smaller deferred items: clone-from-remote smoke, version-sync CI (`plugin.json`↔`marketplace.json`), Plan-02 leftover `ddaanet/gitmoji-gitlore-memory` cleanup, locked-worktree test for `WorktreeRemove` (nice-to-have follow-up).

**How to apply:** When resuming work, read `docs/design.md` first. Note that it's a living design doc — functional reqs, non-functional reqs, architecture, design decisions, rejected alternatives, changelog. Design review already completed; treat the doc as implementation-ready unless a new gap surfaces.
