---
name: nested-submodule-tier-mechanics
description: "git mechanics of a submodule nested inside a submodule (gitlore memory tiers), characterized 2026-07-19: nested gitdir path, ff-only `fetch live:live`, and the D11 linked worktree getting its OWN independent tier clone"
metadata: 
  node_type: memory
  type: reference
  originSessionId: d5a70bd9-0134-401c-b513-66b2bfbfbd23
  modified: 2026-07-19T17:47:32.091Z
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
  `live` ref exists until the fetch above creates it.
- **A D11 linked memory worktree gets an INDEPENDENT tier clone** at
  `.git/modules/gitlore-memory/worktrees/<wt>/modules/<tier>` — separate object
  store and separate refs from the primary checkout's tier. `submodule update
  --init` from the linked worktree succeeds; it just re-clones. Each worktree
  therefore fast-forwards from the remote on its own (fine for propagation-in;
  two worktrees can diverge once tier writes land — input to the lockstep slice).
- **Checking a stale memory branch out drops `memory/.gitmodules`**, so the tier
  looks unmounted and leaves an untracked `<tier>/` dir behind (`warning: unable
  to rmdir`). Mounting a tier must advance the memory trunk, not one branch.

See [[submodule-escape-to-parent]], [[git-hook-env-leak]],
[[gitlore-global-memory-investigation]].
