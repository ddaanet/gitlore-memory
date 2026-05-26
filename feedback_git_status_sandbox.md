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

**How to apply:** ALWAYS run `git status` (and other git inspection commands like `git ls-files --others`) with `dangerouslyDisableSandbox: true` in this repo — not "default to," always. Recognition cue: if a `git status` shows untracked `.bashrc`/`.bash_profile`/`.claude/agents`/`.claude/commands`/`.claude/skills`/`.zshrc`/`.idea`/`.vscode`/`.mcp.json`/`.gitconfig` etc., you ran it sandboxed — those are the artifacts; re-run unsandboxed before reporting tree state or commit counts (sandboxed `??` noise also obscures the real ahead/behind picture). Don't waste a turn showing the user the polluted output first, and don't rationalize the dotfiles away as "just home-dir files."
