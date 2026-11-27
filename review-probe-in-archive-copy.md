---
name: review-probe-in-archive-copy
description: "A dispatched reviewer's probe harness that symlinks the checkout's scripts/ lets a bats helper overwrite a real script — brief probes onto git-archive copies"
metadata:
  node_type: memory
  type: feedback
  originSessionId: a3b9c059-11d4-4698-9989-5d4321de1d38
  modified: 2026-11-27T11:55:56.027Z
---

A reviewer or fixer that probes a script must run it from a `git archive HEAD | tar -x` copy (or a temp clone) under `$TMPDIR`, never from a harness that symlinks the checkout's `scripts/`. `stub_commit_script` (tests/helpers/commit-summary.bash) copies `$PLUGIN_ROOT/scripts` with `cp -R`, which copies the link, and its `awk > "$script.new"; mv` then rewrites the real file.

**Why:** 2026-11-27, a Fable release reviewer's race probe rewrote `scripts/commit-memory.sh` in the checkout for seven minutes (mode 644, a stub that commits into `memory/`, the live store). It was caught by another reviewer's `git status` and restored from HEAD; nothing landed.

**How to apply:** every review or fix brief that allows probing names the archive-copy rule, and says not to symlink the checkout. After a review fan-out, check `git status --short scripts bin tests skills` before trusting any run from the checkout.
