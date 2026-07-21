---
name: feedback_evals_happy_path
description: "evals are the net bats can't be: cover the ordinary flow not the edges, run on release + on every prompt change; `just precommit` stays fast/frequent, new `just prerelease` (= precommit + evals) is slow/rare and `release` depends on it; content-addressed sentinels on both gates prevent redundant re-runs (unbuilt); write the evals once nested memory (D17 slice 3) is done"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7f6454fc-a230-4fec-989c-c057366f5934
  modified: 2026-07-21T08:53:43.739Z
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
must NOT go in it. A new `just prerelease` is the **slow, rare** gate — it runs
`precommit` plus `make evals` — and **`release` depends on `prerelease`**, so
the expensive run sits on the last irreversible step. `ddaa:preflight` probes
`just precommit` first and therefore stays fast; it is a readiness check, not
the eval gate. Naming: `nightly` is the widely recognized term but names a
*clock* trigger this has none of; `acceptance` names the test *kind*, not the
moment. `precommit`/`prerelease` names the trigger honestly in the vocabulary
this repo already uses. Either way the wiring lives in the project's
justfile/Makefile, never in the shared skill
([[feedback_preflight_stays_generic]]).

**Redundant re-runs are prevented by content-addressed sentinels** on both
`precommit` and `prerelease`: derive a hash from the content the gate actually
covers, record it when the gate passes, and skip the run when the current hash
matches a recorded one. So a `release` right after a green `precommit` re-runs
only what `precommit` did not already cover, and repeated invocations on an
unchanged tree cost nothing. NOT YET BUILT — design the hash inputs when it is.

**Scheduled:** write these evals once nested memory is complete (D17 slice 3 —
after 3-ii composition and 3-iii `/add-tier`), so the scenarios cover the
finished tier flow rather than a moving target
([[project_gitlore_global_memory]]). Sharpens
[[feedback_no_handrun_tests]]'s "prompt contracts are not bats-testable" from
"dogfood it by hand" into a repeatable harness.
