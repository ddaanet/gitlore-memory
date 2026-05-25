---
name: plugin-recurse-clone-submodule
description: "/plugin install clones a plugin repo with --recurse-submodules; any committed submodule must have an absolute, publicly-fetchable url or install aborts"
metadata: 
  node_type: memory
  type: reference
  originSessionId: fba96ba4-0800-400a-b860-509be142f52d
---

`/plugin install <name>@<scope>` clones the plugin's GitHub repo with `git clone --recurse-submodules`. Any submodule committed in that repo is therefore cloned into every installer's `~/.claude/plugins/cache/...`.

**A relative or local-only submodule url breaks install.** gitlore committed `.gitmodules` with `url = ./.git/gitlore-placeholder` (the local-only sentinel) plus a gitlink. On install, git resolved the relative url against the remote → `git@github.com:ddaanet/gitlore.git/.git/gitlore-placeholder` → "not a valid repository name" → install aborts. Discovered Plan 04 Step 6 (2026-05-23) — the inner-loop dogfood could not catch it because `--plugin-dir` does not recurse-clone.

Requirements for a distributable plugin repo with a self-hosted submodule:
- `.gitmodules` url must be **absolute** and use a real scheme (`https://`, `git@…`, `ssh://`). Use **https** so anonymous installers can read a public repo without SSH keys.
- The submodule's recorded gitlink SHA must be **pushed and fetchable** on that remote.
- For install to succeed for everyone, the submodule remote must be **public** (a private submodule → install works only for those with access).

Fix shipped: created public `ddaanet/gitlore-memory`, pushed the submodule, set `.gitmodules` url to `https://github.com/ddaanet/gitlore-memory.git`. Verified by `git clone --recurse-submodules git@github.com:ddaanet/gitlore.git` succeeding. Regression guard: `tests/plugin_distribution.bats` asserts the submodule url is an absolute scheme. The local-only placeholder mechanism still applies to **user** installs (`/gitlore:install` in a target repo) — it's only the plugin repo's *own* self-dogfood submodule that needs a real remote. See [[cc-agent-discovery]].
