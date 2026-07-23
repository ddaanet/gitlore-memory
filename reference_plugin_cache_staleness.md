---
name: reference_plugin_cache_staleness
description: "`~/.claude/plugins/cache/<owner>/<plugin>/<version>/` is a flat extraction keyed by version, so `/plugin update` no-ops when the version string is unchanged; verify loaded source before trusting any plugin-code verification"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 4c347c63-b84a-48f3-b4fb-2b90c78a02a9
---

`~/.claude/plugins/cache/<owner>/<plugin>/<version>/` is a flat file extraction from a past push, NOT a git checkout (no HEAD). A marketplace-backed session loads whatever the cache last held — it can lag the repo by many commits even though the version string (e.g. `0.1.0`) is unchanged. Two things gate a refresh: (1) **`/plugin update` only re-fetches when the version string changed** — the cache dir is keyed by `<version>`, so pushing new commits under the *same* `0.1.0` leaves the cache stale and update no-ops (confirmed live 2026-05-25: pushed the launcher under `0.1.0`, update kept the old extraction; bumping to `0.1.1` forced the re-fetch). (2) even after a refresh, only a *new* session picks up the change; agent definitions in particular are cached at session start. So shipping a code change to marketplace users requires a **version bump**, not just a push.

Discovered closing Plan 04 Step 6 (2026-05-24): a marketplace session was about to "verify" the memory-merger two-turn flow, but the cache still had the pre-flatten `commands/gitlore/` layout AND the broken agent frontmatter (no `name:`, `allowed-tools:`) — i.e. it would have tested old code that the repo had already fixed (`e205aa6`, `27af3b8`). Redone under `--plugin-dir` (loads the local repo) to test current `main`.

**Before trusting any verification of plugin code, confirm the loaded source matches the repo:** `find ~/.claude/plugins/cache/<owner>/<plugin>/<ver>/commands` for the command layout, and check `agents/*.md` frontmatter. The tell in this project: a flat `/gitlore:resolve` in the skill list means `--plugin-dir`/fresh-cache is active; the double-prefixed `/gitlore:gitlore:resolve` means a stale nested cache.

Related: [[reference_cc_agent_discovery]], [[reference_cc_command_namespacing]], [[feedback_dogfood_early]].
