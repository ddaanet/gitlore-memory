---
name: git-hook-env-leak
description: "git invokes hooks with repo-local GIT_* vars; submodule-aware hooks must clear the FULL set (`unset $(git rev-parse --local-env-vars)`), not just GIT_DIR/INDEX/WORK_TREE — else GIT_COMMON_DIR retargets the parent store in linked worktrees; EXCEPT GIT_INDEX_FILE, which a staging hook must capture before the unset and restore for its `git add` — git hands the hook `.git/index` (plain), `.git/index.lock` (`-a`), or `.git/next-index-*.lock` (pathspec), and a bare `git add` under the lock flavors fails \"index.lock: File exists\" and blocks the commit"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 39084536-e084-480e-981f-18bb2e894d63
  modified: 2026-07-21T07:48:50.678Z
---

When `git commit` (or any porcelain command) invokes a hook, it exports `GIT_DIR=.git`, `GIT_INDEX_FILE=.git/index`, `GIT_WORK_TREE=.` (relative to the parent repo). Any `git -C <submodule> <cmd>` inside the hook inherits those env vars; the relative paths resolve under the *submodule's* CWD where `.git` is a gitfile, not a directory. Symptom: `fatal: .git/index: index file open failed: Not a directory`.

Confirmed Plan 04 Step 3 (2026-05-22) in `scripts/git-hooks/pre-commit` / `pre-push`. Reproduces deterministically by exporting the three vars before running the hook standalone.

**Fix (corrected 2026-06-12):** at the top of any submodule-aware git hook, clear the *complete* set, not a hand-picked subset: `unset $(git rev-parse --local-env-vars)` (needs `# shellcheck disable=SC2046` for the intended word-split). The old 4-var `unset GIT_DIR GIT_INDEX_FILE GIT_WORK_TREE GIT_PREFIX` left `GIT_COMMON_DIR`/`GIT_OBJECT_DIRECTORY`/`GIT_CONFIG_PARAMETERS` behind. Those aren't exported in a plain main-worktree commit (so 4-var tests passed), but in a **linked parent worktree** git exports absolute `GIT_DIR` *and* `GIT_COMMON_DIR`; clearing only `GIT_DIR` lets `GIT_COMMON_DIR` silently redirect the submodule's refs/objects to the parent's common dir. gitlore supports linked parent worktrees, so this was a real corruption path. Applied to `scripts/git-hooks/pre-commit`, `pre-push`, and `scripts/commit-memory.sh`. Standalone runs (no git parent) won't have these set; production runs will. Note: `--local-env-vars` does NOT include `GIT_AUTHOR_*`, so an in-hook submodule commit inherits the parent commit's author/date — intended lockstep here, but a footgun for hooks committing into unrelated repos. Regression test pattern: export the leaked vars (both the index-breaking trio AND a bogus `GIT_COMMON_DIR`/`GIT_OBJECT_DIRECTORY`) and assert the hook exits 0, the index-error string is absent, and no refs landed in the bogus common dir.

**`GIT_INDEX_FILE` is the one leaked var a pre-commit hook may need to KEEP** (verified git 2.47.3, 2026-07-21). Git hands the hook a different index depending on how the commit was invoked:

| invocation | `GIT_INDEX_FILE` |
|---|---|
| `git commit` | `.git/index` |
| `git commit -a` | `.git/index.lock` |
| `git commit -- <paths>` / `-p` | `.git/next-index-<pid>.lock` |

A hook that stages something (`git add`) must target *that* index or the commit will not see it. After a blanket `unset $(git rev-parse --local-env-vars)`, a bare `git add` does not merely miss under the two lock flavors — it **fails hard** with `fatal: Unable to create '.git/index.lock': File exists` (git already holds the lock), and under `set -e` that **blocks the commit**. So capture `GIT_INDEX_FILE` *before* the unset and restore it for that one call: `GIT_INDEX_FILE="$saved" git add -- <path>`. With it restored, all three modes record the staged change correctly. Caveat for the pathspec mode: the add lands in the temp index (so it *is* in the commit) but not in the real index, so the path still shows modified afterwards — the documented partial-commit artifact.

**Why standalone tests miss this:** running `bash hooks/pre-commit` directly from the shell doesn't set the leaked vars — and never exercises the index handoff at all, so a staging bug is invisible to it. Tests that only exercise the hook this way will pass while production fails silently. Always include one test that simulates the leaked env, and one that drives a **real `git commit`** in each of the three modes ([[feedback_test_the_invocation_path]]).
