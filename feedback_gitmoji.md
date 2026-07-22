---
name: gitmoji-commit-convention
description: "conventional prefix → emoji, via a plugin-managed commit-msg hook"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a22fb06e-e320-4708-8c2f-18da35422b19
---

Commits in this repo use conventional commit prefixes (`feat:`, `fix:`, `docs:`, `chore:`, `test:`, etc.). A commit-msg hook prepends a gitmoji emoji based on the prefix word and then leaves the message otherwise intact. The hook is installed and managed by the gitmoji plugin: it materializes `.git/hooks/commit-msg` (marked `gitmoji-plugin-installed`) on SessionStart, with the bundled `.git/hooks/gitmoji.sh` + `.git/hooks/gitmoji.cfg` alongside. It is not a tracked file in any repo. (Previously a homegrown `devddaanet/scripts/gitmoji.sh` served this role; that was retired when the plugin took over.)

**Mappings observed in this repo's history:**
- `feat:` → `✨`
- `fix:`  → `🐛`
- `test:` → `✅` (also written as `🧪` by some authors — the hook produces `✅`)
- `docs:` → `📝`
- `chore:`→ `🔧`

**Why this matters when writing commit messages:**
- Don't prepend your own emoji to the subject — the hook prepends one and you'll end up with two (observed: `✅ 🧪 add end-to-end happy-path integration test` after an agent typed `test: 🧪 ...`).
- The conventional prefix word is the source of truth for which emoji you get. To pick a specific emoji, pick the matching prefix word, not the emoji directly.
- Unknown prefixes cause the commit to fail — stick to: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert, hotfix.

**How to apply:** Write commit messages as `<prefix>: <subject>`. Let the hook handle the emoji. If a particular emoji is desired, work backward from the mapping table (or `gitmoji.cfg`) to pick the right prefix word.

Note: hook installation may be missing on fresh worktrees — verify `.git/hooks/commit-msg` exists before relying on the rewrite (gitmoji install command lives in the gitmoji plugin).
