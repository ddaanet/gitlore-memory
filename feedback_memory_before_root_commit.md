---
name: feedback_memory_before_root_commit
description: "commit memory/ before the root commit, push it alongside; lockstep is checked on `live`, not `main`"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9df0bd20-256d-4cbe-b942-fddde0f211b9
---

Keep the `memory/` submodule in lockstep with the parent repo across the full persist cycle:
1. **Commit** pending submodule changes **before** the root commit — don't root-commit while `git -C memory status` is dirty.
2. **Push** the submodule **whenever you push the parent** — don't leave `memory/` ahead of its remote after the root push lands.

**Why:** gitlore's model is git-backed memory — the submodule is the source of truth and should be persisted before the pointer that references it. A root commit made over a dirty submodule leaves memory uncommitted and the recorded pointer stale. Likewise, pushing the parent but not the submodule publishes a pointer to commits no one else can fetch — the shared memory is unreachable, defeating the versioning the tool exists to provide.

**How to apply:** before any root `git commit`, run `git -C memory status` (unsandboxed per [[feedback_git_status_sandbox]]). If dirty, resolve/commit inside the submodule first (the pre-commit/pre-push hook machinery handles the worktree→live commit-and-push; see [[reference_git_hook_env_leak]]), then stage the updated submodule pointer in the root commit. Whenever you push the parent (or it gets pushed for you, e.g. by a release recipe), also `git -C memory push` so both remotes advance together. The parent pre-push hook pushes the submodule's **`live`** branch, so verify lockstep by comparing `live` vs `origin/live` (`git -C memory rev-parse live origin/live`) — **not** `main` vs `origin/main`. The memory tree's local `main` is the worktree branch and may legitimately sit ahead of `origin/main`; only `live` is gitlore's canonical synced branch, and the parent gitlink must be reachable on `origin/live`. Cross-repo/submodule pushes are classifier-gated — if denied, surface the push command for the user rather than dropping it ([[reference_cross_repo_push_auth]]). If the submodule changes are unfamiliar leftovers, surface them rather than blindly committing — [[feedback_verify_handoff_pending]].
