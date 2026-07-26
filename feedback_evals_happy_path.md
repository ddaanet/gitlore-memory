---
name: feedback_evals_happy_path
description: "`just evals` is separate/opt-in, NOT run by `prerelease`/`release`; gate sentinels hash declared inputs, in the justfile's `bash_prolog`; the held 0.4.2 owes a grid run before the tag as part of the vanished-pointer dig — `03`/`04` walk the reworked compose merge"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7f6454fc-a230-4fec-989c-c057366f5934
  modified: 2026-07-26T18:09:55.380Z
---

Use the eval harness (`tests/evals/`) for end-to-end testing that exercises the
**happy paths** — the ordinary flow a user actually walks, driven through the
real agent, not through a script. Run them explicitly with `just evals` —
not wired into `precommit`/`prerelease`/`release`, since the 5-round grid costs
real time and money (decided 2026-07-24: `prerelease` had been `precommit
evals`, but that made every release pay for a grid most releases don't touch).
Run on demand **before a release that touches eval-covered flows**, and
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

**Two gates, split by cost — revised 2026-07-24.** `just precommit` is the
**fast, frequent** per-change gate (`check-version`, `lint`, `test`) and evals
must NOT go in it. `just evals` is the **slow, rare, opt-in** gate — run by
name, not pulled in by `prerelease` or `release`. (2026-07-21 had wired
`prerelease: precommit evals` so the expensive run sat next to the last
irreversible step; 2026-07-24 unwired it — a 5-round eval grid on every
release was wasteful more often than it was warranted, so `prerelease` reverted
to plain `precommit` and evals moved to a manual call.) `ddaa:preflight` probes
`just precommit` first and therefore stays fast; it is a readiness check, not
the eval gate. Naming: `nightly` is the widely recognized term but names a
*clock* trigger this has none of; `acceptance` names the test *kind*, not the
moment. `precommit`/`prerelease` names the trigger honestly in the vocabulary
this repo already uses. Either way the wiring lives in the project's
justfile/Makefile, never in the shared skill
([[feedback_preflight_stays_generic]]).

**Redundant re-runs are prevented by content-addressed sentinels**, one per gate
under `$(git rev-parse --git-path gitlore/gates)/NAME`, recorded only on success.
`precommit` and `evals` each own one, independently skippable; `prerelease` is
plain `precommit` (2026-07-24), so it never touches the evals sentinel. The
logic is `check-sentinel` / `record-sentinel` / `gate-inputs-hash`, bash
functions in the justfile's `bash_prolog` that every shebang recipe expands
(2026-07-26 — it was `scripts/run-gate.sh` until then, and a separate script
bought nothing once the Makefile was gone). The resolved hash inputs:

- **Declared inputs, not the whole tree.** `precommit_inputs` names what the
  checks read; `evals_inputs` adds `agents commands skills`, which only the
  evals reach. `memory/` and `docs/` are in neither. This reverses the
  2026-07-21 whole-tree call, whose fear — a forgotten input yielding a *stale
  green* — is real, so the allow-list is guarded instead: every declared path
  must exist, and every top-level entry must be declared or on a written-down
  exclusion list. Both directions narrow silently otherwise.
- **Content-addressed, not HEAD-addressed**: `git ls-files -z --cached --others
  --exclude-standard` over the declared pathspecs, each name then its contents,
  through `cksum`, with the unpinned tool versions in the same stream. A release
  commits *after* precommit goes green, and that commit must not invalidate the
  sentinel.
- **Untracked non-ignored files count** (hence `--others`): `just test` globs
  `tests/*.bats`, so an unstaged new suite changes what runs.
- **A failed hash records nothing and runs the gate.** Not hypothetical. The
  first design built the hash with `git add -A` into a throwaway index, which
  dies outright on any working-tree path git will not index — the phantom home
  dotfiles the sandbox surfaces ([[reference_sandbox_effects]]) are exactly
  that, and this repo briefly had real ones too. An empty hash suppressed the
  *record* step as well as the skip, so the gate was not degraded but inert,
  never skipping and never recording, for a day, with the only diagnostic on
  stderr where nothing captured it. Hence also: every component of the stream
  fails explicitly, since bash suspends errexit inside the command substitution
  the hash runs in.
- Escape hatch: `GITLORE_GATE_FORCE=1` — which `tests/justfile_gates.bats` must
  `unset` in the subshell it loads the prolog into, because the suite runs
  *inside* a gate and the ambient value otherwise reaches every gate under test
  and makes all the skip cases silently unprovable. A suite that runs inside
  the thing it tests has to neutralize the ambient environment first.

**`release` depends on `prerelease`**, and since 2026-07-24 `prerelease` is
plain `precommit` in this repo — `just release` no longer runs evals. That
`release`→`prerelease` dependency lives in the vendored
`plugin-dev/release.just` (fixed upstream in `ddaanet/claude-plugin-dev`
v0.4.0, vendored here 2026-07-23) rather than patched in the vendored copy —
the generic recipe is the toolkit's, and editing it here would drift from
upstream ([[feedback_preflight_stays_generic]],
[[feedback_no_in_place_other_repos]]). The toolkit makes `prerelease`
**mandatory**: just rejects a justfile whose dependency names a missing
recipe, so a consumer that hasn't defined it fails immediately rather than at
release time — gitlore satisfies that with `prerelease: precommit`, the
"most plugins" default the toolkit docs describe, not the widened
`prerelease: precommit evals` it had between 2026-07-21 and 2026-07-24.

**Five scenarios exist** (written 2026-07-22, once the tier flow stopped being a
moving target): `01`/`02` on the memory-commit flow, `03-add-tier`,
`04-tier-write`, `05-recall`. They sharpen
[[feedback_no_handrun_tests]]'s "prompt contracts are not bats-testable" from
"dogfood it by hand" into a repeatable harness
([[project_gitlore_global_memory]]).

**Owed on the held 0.4.2 (2026-07-25):** the path-keyed three-way root↔carrier
compose merge is committed but untagged, and `03-add-tier` / `04-tier-write`
are exactly what walk it end to end. The unit suite covers the merge, but a
pointer line still vanished from the live store during ordinary index edits and
could not be reproduced from the committed blobs
([[project_gitlore_global_memory]]) — which is the case for the grid, not
against it: the loss happened in the agent-and-shell seam that bats cannot see,
and that seam is the whole reason these evals exist. Run `just evals` as part
of that investigation, before the tag.
