---
name: feedback-git-status-sandbox
description: "sandbox surfaces phantom home dotfiles → sandboxed working-tree reads misreport; git status auto-unsandboxed by the unsandbox-git-status plugin (best-effort); sibling-worktree writes still blocked"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a22fb06e-e320-4708-8c2f-18da35422b19
---

In the gitlore repo the command sandbox distorts some git/filesystem operations (verified 2026-07-17).

1. **Phantom dotfiles.** A sandboxed working-tree read surfaces user-home dotfiles absent from the repo — `.bashrc`, `.zshrc`, `.bash_profile`, `.zprofile`, `.profile`, `.gitconfig`, `.ripgreprc`, `.idea`, `.vscode`, `.mcp.json`. So a sandboxed `git status` reports spurious untracked files. Ground truth: the repo's real dotfiles are only `.claude .claude-plugin .editorconfig .envrc .git .gitignore .gitlore .gitmodules`.

2. **Sibling-worktree writes blocked.** The sandbox grants write access only to the current worktree (`.`) plus `.git`. From a linked worktree, a git op writing a *sibling* path fails with `Read-only file system` — e.g. `git -C <main-root> merge/push <branch>` (finishing a worktree branch into main) or `git worktree remove <other>`. The merge half-prints "Updating …" then aborts cleanly (no `MERGE_HEAD`), so retry unsandboxed.

`.envrc` vars pass through the sandbox: `MARKETPLACE_DIR` and the `DIRENV_*` vars are present in the sandboxed env (direnv runs in the launching shell), so `.envrc`-dependent commands like `just release` work sandboxed.

**`git status` is auto-handled.** The `unsandbox-git-status` plugin (a PreToolUse Bash hook) detects a `git status` command and flips `dangerouslyDisableSandbox` in place, so it runs unsandboxed transparently — no need to remember. Caveats: detection is best-effort (an unrecognised `git status` stays sandboxed), and it deliberately does NOT cover `git add -A` (also fails sandboxed, but too dangerous to auto-unsandbox). Its notice goes to `systemMessage` (user channel, not model context), so the agent may never learn a command was auto-unsandboxed — do NOT read a clean sandboxed `git status` as evidence the sandbox itself is clean.

**How to apply:** run any *other* sandboxed command that reads the working tree, or that writes to a sibling worktree, with `dangerouslyDisableSandbox: true`. Cues: phantom `.bashrc`/`.zshrc` in a listing; `Read-only file system` from a `git -C <other-worktree>` op. Relevant to [[finishing-a-development-branch]] when merging a worktree branch into main.
