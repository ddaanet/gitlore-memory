---
name: feedback-git-status-sandbox
description: Run git status unsandboxed in gitlore repo — sandbox shows phantom untracked dotfiles
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a22fb06e-e320-4708-8c2f-18da35422b19
---

In the gitlore repo, run `git status` with `dangerouslyDisableSandbox: true`. The sandboxed shell shows spurious untracked files (`.bashrc`, `.zshrc`, `.idea`, `.vscode`, `.claude/agents`, `.claude/commands`, `.claude/settings.json`, `.claude/skills`, `.mcp.json`, `.gitconfig`, `.gitmodules`, `.profile`, `.ripgreprc`, `.bash_profile`, `.zprofile`) that don't actually exist in the working tree — they're sandbox artifacts, not real repo state.

**Why:** The sandbox mounts/overlays user dotfiles into the working directory view, so `git status` reports them as untracked. Unsandboxed, the working tree is actually clean.

**How to apply:** Default to `dangerouslyDisableSandbox: true` for `git status` (and likely other git inspection commands like `git ls-files --others`) in this repo. Don't waste a turn showing the user the polluted output first.
