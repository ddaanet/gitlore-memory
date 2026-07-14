---
name: gitlore-global-memory-investigation
description: Design (2026-07-14) for a gitlore global/org memory tier alongside per-project memory — CONVERGED, captured as FR15 + D17 in docs/design.md
metadata: 
  node_type: memory
  type: project
  originSessionId: 7affbb95-e26e-4294-8835-9c686a979955
---

Investigating whether gitlore should support a GLOBAL/organizational memory tier alongside its per-project memory. Motivation: David has ~16 sibling repos each with a gitlore `memory/` submodule; `user` + CC-platform `reference` + portable `feedback` memories are trapped and duplicated per-repo, while `project` memories are correctly per-repo.

**Conclusion so far:** a flat merge-everything store is WRONG — it blows the ~200-line always-loaded `MEMORY.md` budget and loads 15 irrelevant projects every session. Right shape is **two-tier**: a global tier (user + CC-platform reference + portable feedback) loaded everywhere, plus per-project tiers staying local. This aligns with a structure Anthropic ALREADY ships — the `team/` subdir + combined index (see [[cc-memory-retrieval-agentic]]). **Design principle: keep Anthropic's memory structure, upgrade only gitlore's GENERATION step — that structure is Anthropic's surface.**

**Load-bearing facts.** Retrieval is agentic (≤5 selective attach), so a large global tier costs little context IF surfaced on-selection rather than bulk-loaded. gitlore today: flat memory submodule, hand-edited `MEMORY.md` (NO generation step anywhere), write path is a uniform `git add -A` (`scripts/lib/resolve.sh:158-161`) with ZERO per-file/per-type routing. `docs/design.md:637` already rejected a global `userSettings` `autoMemoryDirectory` (all projects → one repo). NOTE: the early "write-routing = a classifier" framing was SUPERSEDED during design — routing is the agent picking a directory (see converged design below), no content classifier.

**Experiment DONE (2026-07-14) — both unknowns resolved, the clean two-tier design is confirmed viable** (probe details in [[cc-memory-retrieval-agentic]]; protocol was `~/.claude/plans/go-precious-crystal.md`, sentinel-introspection via `claude --print --settings autoMemoryDirectory=<scratch>`, no real-memory contact):
- Retrieval attaches file BODIES on-selection under `--print` (mechanism fires; not "never read").
- **U1:** both the `MEMORY.md` one-liner AND the per-file `description:` frontmatter feed the classifier — either is a viable generation target.
- **U2a:** a `team/`-style subdir IS enumerated natively (100% for indexed subdir files); depth ≥2 works too.
- **U2b:** subdir/`team/` bodies do NOT bulk-load — the always-loaded footprint is the `MEMORY.md` index blob only. A global tier of N files costs N index one-liners, NOT N bodies. Near-free at baseline as long as the index stays under ~200 lines.
- **Reliability:** indexed files retrieve 100%; an unindexed pure-frontmatter file retrieved 75%. → the generation step MUST emit `MEMORY.md` pointers for global files, not lean on frontmatter enumeration alone.

**CONVERGED DESIGN — captured as D17 in `docs/design.md` (design only; not yet implemented or planned).** Generalized from "two-tier" to **N named tiers**:
- **Materialization:** each tier (`memory/lore` global, `memory/ddaanet` org, …; names configurable) is a **nested git submodule** inside the project memory submodule, reusing the existing init / FR11 gate / push-lockstep framework. `autoMemoryDirectory` is single-path, so tier content is physically present as nested checkouts under it.
- **Routing = agent picks the directory**, guided by an addition to the standing SessionStart `additionalContext` (D12 block): portable → `memory/<tier>/`, project → `memory/`. No content classifier — the directory *is* the tier *is* the submodule; the FR11 gate commits whichever submodule the file landed in. (Routing guidance rides `additionalContext` → may not inject under `--print` per D14; a misfile is self-correctable, never corruption.)
- **Index generation:** a `PostToolUse(Write|Edit)` hook (no-op outside the memory dir) derives the one-liner *description* from the file's frontmatter (single source of truth, no drift) and recomposes the **root `MEMORY.md`** — tier blocks in precedence order (global first), project bare-path lines last. Two writers, partitioned by path prefix, never collide. The pointer is DERIVED from file location, so the agent only needs to pick the right directory. (Distinct from the rejected "PostToolUse for commit-prep" — this is index composition, cheap/idempotent.)
- **Propagation + pinning:** SessionStart ff's the nested tier submodule(s) so cross-project global facts arrive; the recompose leaves `memory/` DIRTY — expected — and rides the next parent commit (float, but commits naturally; no scheduled sync-commit, no gate churn).
- **Conflicts:** `**/MEMORY.md merge=union` concatenates same-position insertions with no markers; recompose dedups by path. No append-only constraint; conflicts rare (more global → more stable).

**Empirical basis** for "compose into root index": interactive PTY + `--print` both confirm recall = tool-gated `Read` steered by the ROOT `MEMORY.md`; nested `team/MEMORY.md` NOT auto-loaded; index-listed → ~100% recall, unindexed → ~75%. See [[cc-memory-retrieval-agentic]].

**Next step (when scheduled):** promote D17 to an FR + write an implementation plan. Seams: `scripts/lib/resolve.sh:158-161` (uniform `git add -A`), `scripts/cc-hooks/session-start.sh` (composition + ff), a new PostToolUse(Write|Edit) hook.
