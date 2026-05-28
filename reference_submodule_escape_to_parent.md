---
name: reference-submodule-escape-to-parent
description: git -C / cd into an unchecked-out submodule silently escapes to the parent repo; how to detect linked worktrees
metadata: 
  node_type: memory
  type: reference
  originSessionId: 4a2946a3-91d0-46ec-a916-73075b08702f
---

When a submodule working-tree path exists but has **no `.git`** (empty/unchecked-out
dir — fresh clone before SessionStart, a linked worktree whose submodule was never
checked out, or after `git submodule deinit`), every `git -C "$subpath" …` or
`cd "$subpath"; git …` walks **up** the directory tree, finds the enclosing repo's
`.git`, and operates on the **parent**. Symptoms in gitlore: `git -C memory rev-parse HEAD`
returns the parent's HEAD (then staged as a bogus 160000 gitlink → unclonable
superproject), and `git branch live` creates the reserved trunk in the parent.

Guards:
- Before any submodule git op, assert `[ -e "$subpath/.git" ]`.
- Detect a **linked worktree**: `git rev-parse --git-dir` (resolved) differs from
  `git rev-parse --git-common-dir` (resolved). They're equal only in the main worktree.

Same failure family as [[reference-plugin-recurse-clone]] (unreachable gitlink SHA)
and related to [[reference-git-hook-env-leak]] (GIT_DIR/GIT_INDEX_FILE leak into
`git -C <submodule>`). Fixed in `scripts/install/run.sh` (main-worktree gate) and
`scripts/install/init-submodule.sh` (no-`.git` refuse).
