---
name: feedback_memory_before_root_commit
description: "commit the memory submodule's changes before the root repo commit, so root records an advanced (not stale/dirty) submodule pointer"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9df0bd20-256d-4cbe-b942-fddde0f211b9
---

When committing the root gitlore repo, commit pending changes in the `memory/` submodule **first**, then make the root commit. Do not root-commit while `git -C memory status` is dirty.

**Why:** gitlore's model is git-backed memory — the submodule is the source of truth and should be persisted before the pointer that references it. A root commit made over a dirty submodule leaves memory uncommitted and the recorded pointer stale, defeating the versioning the tool exists to provide.

**How to apply:** before any root `git commit`, run `git -C memory status` (unsandboxed per [[feedback_git_status_sandbox]]). If dirty, resolve/commit inside the submodule first (the pre-commit/pre-push hook machinery handles the worktree→live commit-and-push; see [[reference_git_hook_env_leak]]), then stage the updated submodule pointer in the root commit. If the submodule changes are unfamiliar leftovers, surface them rather than blindly committing — [[feedback_verify_handoff_pending]].
