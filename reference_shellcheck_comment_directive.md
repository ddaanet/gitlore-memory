---
name: reference_shellcheck_comment_directive
description: "a comment starting `# shellcheck` parses as a malformed directive and fails lint; .bats is linted too"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 43b30b7a-8283-4436-a17a-ca3109131521
---

ShellCheck treats any comment whose first token is `# shellcheck` as a directive, not prose. A descriptive comment like `# shellcheck missing on a shell file = ...` triggers SC1072/SC1073 ("Expected '=' after directive key") and fails the lint. Reword such comments so they don't begin with the word `shellcheck` (e.g. "Missing shellcheck ..."), or it's a self-inflicted lint error.

ShellCheck has linted Bats test files since v0.7.0 (2019); the 0.10.0 man page auto-detects `.bash`/`.bats`/`.dash`/`.ksh` extensions plus shell directive/shebang. So bats suites (gitlore's `tests/*.bats`) are lintable. Related: [[reference_plugin_hook_user_channel]] for hook exit-2/systemMessage channels (the shell-scripting plugin's shellcheck-on-edit hook uses both).
