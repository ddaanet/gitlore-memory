---
name: feedback_evals_happy_path
description: "evals cover the ordinary flow bats can't; `just precommit` fast, `just prerelease` slow; sentinels in `scripts/run-gate.sh`; write them once D17 slice 3 lands"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7f6454fc-a230-4fec-989c-c057366f5934
  modified: 2026-07-21T11:15:31.118Z
---

Use the eval harness (`tests/evals/`) for end-to-end testing that exercises the
**happy paths** — the ordinary flow a user actually walks, driven through the
real agent, not through a script. Run them at two moments: **on release**, and
**whenever a prompt changes** (skill, agent, hook `additionalContext`,
orientation block).

**Why:** the bugs that keep escaping are happy-path bugs in the seam between the
agent and the shell, and bats structurally cannot see them. Two from 2026-07-21
alone: the tier fetch ran with `-q`, so its divergence arm was dead code; and
`pre-commit` never staged the memory gitlink, so every parent commit recorded a
stale memory SHA. Both survived a green suite because the suite called each
script the convenient way rather than the way production does
([[feedback_test_the_invocation_path]]) — and no bats test can drive "a session
starts, the agent edits memory, the user approves, the commit lands." Only an
eval walks that. The prompt-change trigger is the other half: a prompt is
untestable by assertion, so the eval *is* its regression test — and prompts are
edited far more casually than code.

**How to apply:** write happy-path scenarios, not edge cases — bats already owns
the edges, and an eval's value is proving the whole chain fits together. Keep
them in the `pass^k` shape the harness already uses, so an agent-side flake is
distinguishable from a regression.

**Two gates, split by cost (decided 2026-07-21).** `just precommit` is the
**fast, frequent** per-change gate (`check-version`, `lint`, `test`) and evals
must NOT go in it. `just prerelease` is the **slow, rare** gate — it runs
`precommit` plus `just evals` — so the expensive run sits next to the last
irreversible step (wiring in the last paragraph). `ddaa:preflight` probes
`just precommit` first and therefore stays fast; it is a readiness check, not
the eval gate. Naming: `nightly` is the widely recognized term but names a
*clock* trigger this has none of; `acceptance` names the test *kind*, not the
moment. `precommit`/`prerelease` names the trigger honestly in the vocabulary
this repo already uses. Either way the wiring lives in the project's
justfile/Makefile, never in the shared skill
([[feedback_preflight_stays_generic]]).

**Redundant re-runs are prevented by content-addressed sentinels — BUILT
2026-07-21** as `scripts/run-gate.sh NAME CMD...`, one sentinel per gate under
`$(git rev-parse --git-path gitlore/gates)/NAME`, recorded only on success.
`precommit` and `evals` each own one; `prerelease` depends on both, so it
re-runs only the evals after a green `precommit`. The resolved hash inputs:

- **Whole tree, not a per-gate input set.** A narrower set skips more often but
  a forgotten input yields a *stale green*, the one failure a gate must not
  have. Over-running is the acceptable direction.
- **Content-addressed, not HEAD-addressed** — throwaway index: `cp` the real
  index, `git add -A`, `git write-tree`. A release commits *after* precommit
  goes green, and that commit must not invalidate the sentinel.
- **Untracked non-ignored files count** (hence `add -A`, not `add -u`):
  `make test` globs `tests/*.bats`, so an unstaged new suite changes what runs.
- **A failed hash records nothing and runs the gate.** Not hypothetical — under
  a sandbox surfacing phantom home dotfiles `git add -A` dies outright, and the
  first draft recorded the half-updated index's hash, which would have skipped
  the *next* run ([[feedback_git_status_sandbox]]).
- Escape hatch: `GITLORE_GATE_FORCE=1` — which `tests/gate_sentinel.bats` must
  `unset` in `setup`, because the suite runs *inside* a gate (`just precommit`
  → `make test`) and the ambient value otherwise reaches every gate under test
  and makes all the skip cases silently unprovable. A suite that runs inside
  the thing it tests has to neutralize the ambient environment first.

**`release` still depends on `precommit` alone**, because that dependency lives
in the vendored `plugin-dev/release.just` and editing it here would drift from
upstream ([[feedback_preflight_stays_generic]], [[feedback_no_in_place_other_repos]]).
Release via **`just prerelease release`**: the evals run once and release's own
`precommit` is a sentinel skip. Making `release` depend on a `prerelease` the
plugin defines is a generic change that belongs upstream in the toolkit.

**Scheduled:** write these evals once nested memory is complete (D17 slice 3 —
after 3-ii composition and 3-iii `/add-tier`), so the scenarios cover the
finished tier flow rather than a moving target
([[project_gitlore_global_memory]]). Sharpens
[[feedback_no_handrun_tests]]'s "prompt contracts are not bats-testable" from
"dogfood it by hand" into a repeatable harness.
