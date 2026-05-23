---
name: feedback-dogfood-b
description: "Concrete bugs Dogfood B (gitlore install on the gitmoji repo) surfaced that the bats suite did not — a second instance of [[feedback-dogfood-early]]"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a6677797-2e34-4b21-98e2-e25fd30753fb
---

Plan 02's Dogfood B on `/Users/david/code/gitmoji` surfaced two bugs the unit + integration suites missed.

**Why:** the surprises follow the [[feedback-dogfood-early]] pattern — fixtures encode what the author *expected* to vary; real-world deviations live elsewhere.

**How to apply:** when writing fixtures for filesystem-affecting installers, do not assume the parent repo's `.gitignore` is empty, and do not assume command-line tools cope with non-default git layouts (gitfile-pointed worktrees, submodules, etc.).

## Surprise 1 — `.gitmodules` was in the target repo's `.gitignore`

`gitmoji/.gitignore` listed `.gitmodules` under a "Claude Code sandbox artifacts" comment — added to quiet `git status` against sandbox-induced churn. `git add .gitmodules` silently refuses to stage gitignored files; the install aborted halfway.

**Fix:** `scripts/install/init-submodule.sh` now detects (`git check-ignore -q .gitmodules`) the case, strips the `.gitmodules` line from the top-level `.gitignore` (preserving other entries), and proceeds. A regression test covers it.

**What the bats suite missed:** every fixture in `tests/install_run.bats` ran against a freshly-`git-init`'d repo with no `.gitignore`. Real users have populated gitignores, and the sandbox-quieting convention is something we ourselves established (see [[feedback-git-status-sandbox]]) — so we should have expected it.

## Surprise 2 — `gh repo create --source=. --push` rejects gitfile-pointed submodules

Plan 02's spec used `gh repo create <name> --private --source=. --push` from inside the memory submodule worktree. The submodule's working tree has a `.git` *file* (`gitdir: ../.git/modules/gitlore-memory`), not a `.git/` dir. gh 2.88.1 rejected this with "current directory is not a git repository. Run `git init` to initialize it." `git rev-parse --is-inside-work-tree` returns true from the same dir; `gh repo view --json sshUrl` also works. The issue is specifically gh's `--source` handling.

**Fix:** `scripts/install/create-remote.sh` no longer uses `--source` or `--push`. It splits the work:
1. `gh repo create <name> --private` (create empty remote).
2. `gh repo view <name> --json sshUrl -q .sshUrl` (resolve URL — respects the user's gh-configured protocol).
3. `git -C <mempath> remote add origin <url>` (wire local).
4. `git -C <mempath> push -u origin live` (push).

This is more explicit, doesn't depend on gh's source-discovery, and the mock surface shrank (no more side-effect simulation block).

**What the bats suite missed:** the `gh-mock.bash` shim recognized `--source=. --push` and simulated the side effects directly without exercising gh's real source-discovery path. We were testing our own contract against ourselves.

## Process lesson

Two real-world bugs, both caught within minutes of the first dogfood attempt; both invisible to 89 passing unit/integration tests. This is the same shape as Plan 01's three dogfood-only bugs ([[feedback-dogfood-early]]). The pattern is now established enough to call it a rule: filesystem-touching installers must dogfood on at least one non-trivial real repo *before* the plan is considered shipped.
