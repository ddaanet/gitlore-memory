---
name: reference_git_replay_hooks
description: "`git rebase --continue` does NOT fire pre-commit (2.47.3) but cherry-pick/revert `--continue` and a hand-made commit at a rebase stop DO; a hook exiting non-zero during a replay aborts the REBASE not the commit; detect replay via `--git-path rebase-merge|rebase-apply|CHERRY_PICK_HEAD|REVERT_HEAD` — MERGE_HEAD is not a replay"
metadata: 
  node_type: memory
  type: reference
  originSessionId: b75fba34-d0f7-49a6-acd4-8c79f04f6b8d
  modified: 2026-07-21T10:55:56.302Z
---

Verified against git 2.47.3 (2026-07-21) while building gitlore's replay guard.

**Which replay paths actually fire `pre-commit`:**

| path | fires `pre-commit`? |
|---|---|
| `git rebase --continue` (merge backend, after conflict) | **no** |
| `git commit` by hand at a conflict stop | yes |
| `git commit --amend` at an interactive `edit` stop | yes |
| `git cherry-pick --continue` | yes |
| `git revert --continue` | yes |
| `git commit` resolving a conflicted merge | yes |

The `rebase --continue` gap matters for tests: a guard driven through
`--continue` passes with **no guard at all** and asserts nothing. Drive the
hand-made commit at the stop instead — it is both the path that reaches the
hook and the one users actually take ([[feedback_test_the_invocation_path]]).

**A non-zero exit from `pre-commit` during a replay aborts the REBASE, not a
commit.** There is no commit to reject; git unwinds the sequence and leaves
history half-replayed for the user to clean up by hand. So any hook that can
fail — an approval gate, a lint — is far more destructive mid-replay than it is
on an ordinary commit, which is reason enough to stand the whole hook down
rather than just the part that writes.

**Detecting a replay:** test for `rebase-merge`, `rebase-apply`,
`CHERRY_PICK_HEAD`, `REVERT_HEAD` via `git rev-parse --git-path <name>`.
`--git-path` is per-worktree, which is what replay state is. Resolve them
*before* any `unset $(git rev-parse --local-env-vars)`, same reason
`GIT_INDEX_FILE` is captured there ([[reference_git_hook_env_leak]]).

**`MERGE_HEAD` is not a replay marker.** A merge commit is authored now and
must record current state; so must a plain `--amend` on the tip. Both belong in
the test suite as negative controls, or the guard silently over-reaches.

**Why this bites a submodule pointer specifically:** a submodule's worktree does
not move when the superproject's HEAD does. So a hook that stages the gitlink
during a replay re-pins the *replayed* commit to the submodule's **current**
SHA rather than the one it recorded — the commit's own history is rewritten
under it. Plumbing (`commit-tree`) runs no hooks and never touches the
worktree, which is the escape hatch when surgery is unavoidable.
