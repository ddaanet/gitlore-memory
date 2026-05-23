---
name: git-hook-env-leak
description: "git invokes hooks with GIT_DIR/GIT_INDEX_FILE/GIT_WORK_TREE set to relative paths; `git -C <submodule>` inherits them and fails on the submodule's gitfile — unset at the top of the hook"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 39084536-e084-480e-981f-18bb2e894d63
---

When `git commit` (or any porcelain command) invokes a hook, it exports `GIT_DIR=.git`, `GIT_INDEX_FILE=.git/index`, `GIT_WORK_TREE=.` (relative to the parent repo). Any `git -C <submodule> <cmd>` inside the hook inherits those env vars; the relative paths resolve under the *submodule's* CWD where `.git` is a gitfile, not a directory. Symptom: `fatal: .git/index: index file open failed: Not a directory`.

Confirmed Plan 04 Step 3 (2026-05-22) in `scripts/git-hooks/pre-commit` / `pre-push`. Reproduces deterministically by exporting the three vars before running the hook standalone.

**Fix:** at the top of any submodule-aware git hook, `unset GIT_DIR GIT_INDEX_FILE GIT_WORK_TREE GIT_PREFIX`. Standalone hook runs (without git as parent) won't have them set; production runs will. Regression test pattern: export the leaked vars in the test, assert hook exits 0 and the index-error string isn't in stderr.

**Why standalone tests miss this:** running `bash hooks/pre-commit` directly from the shell doesn't set the leaked vars. Tests that only exercise the hook this way will pass while production fails silently. Always include one test that simulates the leaked env.
