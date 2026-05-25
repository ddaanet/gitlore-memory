---
name: cc-command-namespacing
description: plugin slash-command name = /<plugin>:<path-under-commands>; nesting commands/<subdir>/x.md double-prefixes to /<plugin>:<subdir>:x — keep commands flat
metadata: 
  node_type: memory
  type: reference
  originSessionId: fba96ba4-0800-400a-b860-509be142f52d
---

A CC plugin slash command's invocation name is `/<plugin>:<path-relative-to-commands-dir>`. So:
- `commands/install.md` → `/gitlore:install`
- `commands/gitlore/install.md` → `/gitlore:gitlore:install` (the `gitlore/` subdir becomes an extra namespace segment — the ugly double prefix)

Keep command files **flat** directly under `commands/` unless you deliberately want a sub-namespace. Discovered Plan 04 (2026-05-23): gitlore shipped `commands/gitlore/{install,resolve}.md`, flattened to `commands/{install,resolve}.md`.

Watch for collisions with skills: a skill and a command that resolve to the same `<plugin>:<name>` token clash. gitlore had both a `skills/install/SKILL.md` (thin pointer) and the command; flattening the command to `/gitlore:install` forced removing the redundant skill. See [[cc-agent-discovery]] for the parallel agent-frontmatter rules.
