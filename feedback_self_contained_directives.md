---
name: self-contained-directives
description: "When emitting commands a sub-agent will run, include absolute paths and an explicit cd — assume nothing about the sub-agent's env or CWD"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 42606cdb-1aa3-4a00-a992-3119f3bbf6d4
---

When a script emits a command for a sub-agent (or any downstream actor) to execute, the command must be **self-contained**: absolute paths only, explicit `cd "<dir>" && ...` if a particular CWD is needed, no `$ENV_VAR` references that the sub-agent might not have set.

**Why:** Sub-agent shells don't reliably inherit the parent's environment or CWD. A directive that reads `bash "$CLAUDE_PLUGIN_ROOT/scripts/foo.sh"` expands to `bash "/scripts/foo.sh"` (file not found) in any shell where the var isn't set. A continuation that depends on `git config --file .gitmodules` (CWD-relative) crashes if the sub-agent's CWD is `/tmp` instead of the repo root. Plan 03's dogfood (commit `dcaaf75`) caught all three variants — `$CLAUDE_PLUGIN_ROOT` not propagated, CWD not preserved, and incomplete file lists from `git diff base...HEAD` after a checkout.

**How to apply:** Whenever you write code that emits a command for another agent or process to run:
- Resolve every `$VAR` at emit time, not run time.
- If the command needs a particular working directory, prefix it with `cd "<absolute-path>" && `.
- Test the directive by running it verbatim in a shell with `env -i` (no inherited env) from an unrelated CWD.

Bats tests with `set -e` and pre-set env vars routinely mask these issues; live dogfood reliably surfaces them. See [[feedback-dogfood-early]].
