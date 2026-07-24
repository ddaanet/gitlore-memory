---
name: nested-submodule-tier-mechanics
description: "tier gitdir at `.git/modules/gitlore-memory/modules/<tier>`; `fetch origin live:live` is ff-only for free (works only because `live` is never checked out as a branch); a D11 linked memory worktree gets its OWN independent tier clone under `worktrees/<wt>/modules/<tier>`; seed a tier remote with default branch `main` and `live` alongside — a `live` default gets checked out by the mount and the ff-only fetch then refuses; mounting needs no commit inside memory"
metadata: 
  node_type: memory
  type: reference
  originSessionId: d5a70bd9-0134-401c-b513-66b2bfbfbd23
  modified: 2026-07-24T13:21:30.552Z
---

Characterized by spike + pinned in `tests/tier_discovery.bats` (D17 slice 3-i-a).

- **Nested gitdir:** `git -C memory submodule add` puts the tier's store at
  `<parent>/.git/modules/gitlore-memory/modules/<tier>`; `memory/<tier>/.git`
  reads `gitdir: ../../.git/modules/gitlore-memory/modules/<tier>`. The tier is
  registered in the memory store's OWN `.gitmodules` only — the parent's is
  untouched, which is what makes discovery-by-enclosure work.
- **`git fetch origin live:live` is fast-forward-only for free** — a refspec
  fetch into a branch ref refuses a non-ff without `+` (exit 1, `! [rejected]
  … (non-fast-forward)`; `origin/live` still advances). It works only because a
  tier never checks `live` out AS a branch — git refuses to fetch-update a
  checked-out branch. That is the propagation-in primitive; callers must
  tolerate the non-zero exit.
- **A fresh mount lands on the remote's default branch** (`main`), and no local
  `live` ref exists until the fetch above creates it. Consequence for seeding a
  tier remote: keep the default branch `main` and push `live` *alongside* it.
  A `live` default gets checked out as a branch by the mount, and the ff-only
  fetch above then refuses to update it — propagation-in dies at the first hop.
- **Mounting a tier requires no commit inside the memory store.**
  `gitlore_tier_paths` reads `memory/.gitmodules` from the *working tree*, so a
  `submodule add` left staged is already discoverable; the FR11 gate stays the
  sole committer.
- **A D11 linked memory worktree gets an INDEPENDENT tier clone** at
  `.git/modules/gitlore-memory/worktrees/<wt>/modules/<tier>` — separate object
  store and separate refs from the primary checkout's tier. `submodule update
  --init` from the linked worktree succeeds; it just re-clones. Each worktree
  therefore fast-forwards from the remote on its own. Two worktrees can diverge
  once tier writes land, but that is **not** a new design problem (settled
  2026-07-21): memory worktrees *share* one `refs/heads/live`, so their
  divergence is caught locally and offline by `push . HEAD:live` (`head-vs-live`);
  a tier's refs are not shared, so the same divergence simply surfaces one step
  later, at `push origin HEAD:live` (`head-vs-remote`). Both flavors exist after
  the branch-model unification and funnel into the same `gitlore_prepare_merge`,
  so a per-worktree tier clone behaves exactly like the already-solved
  two-machines-one-remote case. It wants a lockstep test, not a decision.
- **A tier's `live` reaches the shared remote only via the PARENT repo's
  `pre-push`** (tiers push before memory, `HEAD:live`). So a tier promotion that
  is committed-but-unpushed in the origin repo is INVISIBLE to a
  `/gitlore:add-tier` mount elsewhere: the mount's `fetch origin live:live` reads
  whatever `live` the remote had at mount time. Diagnosed on cwd-safety
  2026-07-24 — it mounted `ddaanet` while gitlore's Step-0 61-file `live` was
  still local, and got the 6/7-file pre-promotion tier. `origin/live` reflog is
  the timeline (`update by push` entries). Also: the remote's DEFAULT branch
  `main` stays at the seed forever by design (the ff-only-fetch landing handle),
  so a fresh `clone`/web view shows the seed, not `live` — a red herring, since
  `add-tier.sh` detaches at `live`, not the default. Rule for the migration:
  **push the origin repo before mounting the tier in any sibling.** A stale mount
  self-heals on the sibling's next SessionStart (`fetch origin live:live` +
  detach), provided the newer `live` is a fast-forward (it is, promotions build
  on the seed).
- **Checking a stale memory branch out drops `memory/.gitmodules`**, so the tier
  looks unmounted and leaves an untracked `<tier>/` dir behind (`warning: unable
  to rmdir`). Mounting a tier must advance the memory trunk, not one branch.

See [[submodule-escape-to-parent]], [[git-hook-env-leak]],
[[gitlore-global-memory-investigation]].
