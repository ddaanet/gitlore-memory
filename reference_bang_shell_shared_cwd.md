---
name: reference_bang_shell_shared_cwd
description: "the user's `!`-prefixed shell and the agent's Bash tool share one working directory; a lingering agent `cd` silently relocates the user's command"
metadata: 
  node_type: memory
  type: reference
  originSessionId: c6448e7d-daa1-41ce-83a9-ec2174c57e83
---

The user's `!`-prefixed interactive commands and the agent's `Bash` tool run in the **same shell working directory**. A `cd` left over from an earlier agent `Bash` call therefore relocates the user's next `!` command.

Bit us 2026-05-25: an earlier `cd .../gitlore/memory` (submodule) persisted; the user's `! git push origin main` then ran *inside the submodule* (already pushed → "Everything up-to-date") instead of the root repo. Looked done; the root's 15 commits were still unpushed. `git ls-remote origin refs/heads/main` exposed the truth — the remote head hadn't moved.

**How to apply:** when handing a `!` command to the user (or running your own Bash), make it cwd-independent — `git -C <abs-path> …`, absolute paths, explicit `cd` at the front. Never assume the shell is where you think. Reinforces [[self_contained_directives]]. Related push-classifier facts: [[reference_cross_repo_push_auth]] (also: the auto-mode classifier denies pushing to the default branch `main` even after a generic "just push" — it wants explicit main-push authorization or an allow rule).
