---
name: feedback-git-status-sandbox
description: Run git/just commands unsandboxed in gitlore repo — sandbox hides dotfiles, blocks .envrc vars, and blocks writes outside the current worktree
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a22fb06e-e320-4708-8c2f-18da35422b19
---

In the gitlore repo, run `git status` and release commands with `dangerouslyDisableSandbox: true`. Two distinct sandbox problems:

1. **Phantom dotfiles:** The sandboxed shell shows spurious untracked files (`.bashrc`, `.zshrc`, `.idea`, `.vscode`, `.claude/agents`, `.claude/commands`, `.claude/settings.json`, `.claude/skills`, `.mcp.json`, `.gitconfig`, `.gitmodules`, `.profile`, `.ripgreprc`, `.bash_profile`, `.zprofile`) that don't actually exist in the working tree.

2. **`.envrc` env vars not loaded:** The sandbox does not run direnv, so vars exported in `.envrc` (e.g. `MARKETPLACE_DIR`) are absent. `just release` will fail with `error: MARKETPLACE_DIR not set` unless run unsandboxed (or the var is passed explicitly on the command line as `MARKETPLACE_DIR=... just release ...`).

3. **Writes outside the current worktree blocked:** The sandbox grants write access only to the current worktree (`.`) plus `.git`. From inside a linked worktree, any git op that writes to a *sibling* path fails with `Read-only file system` — e.g. finishing a branch via `git -C <main-root> merge/push <branch>` (merging the worktree branch into the main worktree's checkout) or `git worktree remove <other>`. The merge half-prints "Updating …" then aborts cleanly (no `MERGE_HEAD` left), so just retry unsandboxed.

**Why:** The sandbox mounts/overlays user dotfiles into the working directory view (problem 1), does not source direnv hooks (problem 2), and scopes its write allowlist to the current worktree (problem 3).

**How to apply:** ALWAYS run `git status`, `just release`, anything depending on `.envrc`, and any git write targeting a sibling worktree (merge/push/worktree-remove into the main worktree) with `dangerouslyDisableSandbox: true` in this repo. Recognition cue for problem 1: untracked `.bashrc`/`.bash_profile` etc. Problem 2: `error: MARKETPLACE_DIR not set` from `just release`. Problem 3: `Read-only file system` from a `git -C <other-worktree>` op. Relevant to [[finishing-a-development-branch]] when merging a worktree branch into main.
