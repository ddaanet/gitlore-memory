---
name: dogfood-bin-on-path
description: "`command -v gitlore-commit` (or any gitlore bin) resolves to `dist/plugin/bin/`, not the checkout's `bin/`: an edit to `bin/` does not run in-repo until `just dogfood` promotes it"
metadata:
  node_type: memory
  type: project
  originSessionId: 97c3a880-f831-444f-bbad-3f6b10e13e07
  modified: 2026-10-30T14:42:34.932Z
---

In a session launched through `plugin-dev/bin/claude`, every gitlore command
on PATH — `gitlore-commit` and the rest — is the copy under
`dist/plugin/bin/`, which `just dogfood` promotes from the checkout. The
checkout's `bin/` is not on PATH.

**Why:** the launcher appends `<root>/dist/plugin/bin` to PATH unconditionally
(`docs/references/memory-commit-ownership.md`), so a branch that changes a
script in `bin/`, or the gate a script talks to, keeps running the last
promoted build. Nothing errors; the old behaviour simply persists.

**How to apply:** after changing anything under `bin/` that the session itself
calls — committing memory through `gitlore-commit` included — run
`just dogfood` before relying on the new behaviour, and check with
`command -v` which copy a call reaches.
