---
name: reference-submodule-config-visibility
description: "A git hook firing inside the memory submodule reads the submodule's own config, not the parent's — parent-pinned keys must be mirrored in."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7f413441-5cdb-4f10-8189-9563a5960e41
---

A git hook that fires under `git -C <mempath> commit` (i.e. *inside* the memory
submodule) runs with the submodule as `GIT_DIR`, so `git config <key>` resolves
against the **submodule's** config (`.git/modules/gitlore-memory/config`), NOT
the parent repo's `.git/config` where SessionStart pins `gitlore.hooksDir`. The
parent wrappers never hit this because they run in the parent context.

Consequence for the D12 submodule commit gate: `emit-memory-gate.sh` must mirror
the parent's `gitlore.hooksDir` into the submodule config each session, else the
gate wrapper reads an empty key and degrades to "hooks not installed" — silently
letting a naked commit through. Symptom when debugging: the wrapper blocks when
invoked directly (parent cwd) but is skipped when invoked by `git -C memory
commit`. The submodule's *common* config is shared across submodule worktrees, so
one write covers linked worktrees too.

Related: [[reference_git_hook_env_leak]] (GIT_DIR/GIT_INDEX_FILE leak across the
boundary), [[reference_submodule_escape_to_parent]].
