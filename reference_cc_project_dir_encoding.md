---
name: reference-cc-project-dir-encoding
description: "How Claude Code encodes a project's filesystem path into its ~/.claude/projects/<name>/ directory name"
metadata: 
  node_type: memory
  type: reference
  originSessionId: f9147fe1-8a12-4da4-b2ce-6533e7a17e2b
  modified: 2026-07-22T14:43:16.636Z
---

Claude Code derives the per-project directory name under `~/.claude/projects/` by mangling the project's absolute path. The rule, verified against the bundled JS inside the `claude` ELF binary at `~/.local/share/claude/versions/*` and corroborated by empirical inspection of `~/.claude/projects/` entries:

1. `realpath()` then Unicode `NFC` normalize the path.
2. Replace every byte not in `[A-Za-z0-9]` with `-`. Leading `/` becomes leading `-`. `.` becomes `-`. `_`, spaces, and non-ASCII all become `-`. Case is preserved. Consecutive specials accumulate (e.g. `/.foo` → `--foo`).
3. If the result exceeds 200 chars, truncate to 200 and append `-<base36(abs(hash(path)))>`. The hash algorithm (`bRH` in the minified bundle) is not visible from strings; no path on this machine has triggered this branch.

Worked examples (all verified):
- `/Users/david/code/gitlore` → `-Users-david-code-gitlore`
- `/Users/david/.claude` → `-Users-david--claude`
- `/Users/david/code/gitmoji/.test-install-XXXX` → `-Users-david-code-gitmoji--test-install-XXXX`

**Undocumented in official Anthropic docs.** Public references: anthropics/claude-code issues [#19972](https://github.com/anthropics/claude-code/issues/19972) (proposed reverse-engineering), [#7009](https://github.com/anthropics/claude-code/issues/7009) (collision report, closed not-planned). Treat the rule as stable but reverse-engineered.

**Watch for:** `edify`'s `encode_project_path` in `src/edify/paths.py` implements only `rstrip("/").replace("/", "-")` — incomplete. Do not borrow it; it will miss any project dir whose path contained `.`, `_`, spaces, or other non-alphanumeric chars.

Shell one-liner that matches CC's common-case output (ignoring the >200-char fallback):
```sh
printf '%s' "$abs_path" | LC_ALL=C sed 's/[^A-Za-z0-9]/-/g'
```

Used in `gitlore` at `scripts/install/init-submodule.sh` to locate the source for auto-memory migration on install.

**To find a session transcript, don't apply this rule — search by session id.** A
transcript lives at `~/.claude/projects/<mangled>/<session-id>.jsonl`, and session
ids are uuids, so the basename alone is unambiguous across every project dir:

```sh
find "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/projects" -name "$session_id.jsonl" -type f -print -quit
```

That is one filesystem walk against a rule that is reverse-engineered, has an
un-derived >200-char branch, and would be a second place to get wrong. Reach for
the mangling only when you have a *path* and no id — as the installer does.
`tests/evals/lib/claude-runner.sh` takes the id route to capture a transcript for
the eval assertions.
