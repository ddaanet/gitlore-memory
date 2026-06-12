---
name: reference_release_uninit_memory_skip
description: an uninitialized memory submodule makes pre-push skip the memory push; init it before any release/parent push or you publish a gitlink missing from the remote
metadata: 
  node_type: memory
  type: reference
  originSessionId: b0680788-fc7d-46c3-9543-cb0ab766a61f
---

`scripts/git-hooks/pre-push:25-27` (the D11 corollary) exits 0 **without
pushing memory** when `memory/.git` is absent — by design, so a session-less
worktree never blocks the parent push. During a release this is a trap:
`just release` runs `git push`, the hook skips memory, and origin/main then
publishes a gitlink commit that is not on the memory remote — broken for
anyone cloning.

Before releasing (or any parent push), verify the submodule is checked out:
`git submodule update --init memory`. Once it is initialized at the pinned
commit, the same pre-push hook fast-forwards memory `live` to origin in
lockstep and the published gitlink is consistent.

Confirmed 2026-06-12 during the 0.2.8 release: the submodule was uninitialized
(an `-`-prefixed `git submodule status`, empty `memory/`), pinned `90b100e` sat
exactly one ff-commit ahead of origin/live, and the release push would have
stranded it. `submodule update --init` restored lockstep before `just release`.

Related: [[feedback_memory_before_root_commit]].
