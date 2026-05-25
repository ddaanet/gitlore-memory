---
name: session-title-via-custom-title-transcript-entry
description: "how CC stores/sets a session title; no hook-output field exists, but a hook can write the transcript"
metadata: 
  node_type: memory
  type: reference
  originSessionId: a7e28213-e5b0-4230-a6b4-882fe813e66e
---

A Claude Code session's title is the `customTitle`, persisted as a line in the session transcript `.jsonl`:

```json
{"type":"custom-title","customTitle":"my title","sessionId":"<id>","uuid":"<uuid>","timestamp":"<iso>"}
```

`/rename <name>` writes exactly this (binary fn appends the entry, sets in-memory `currentSessionTitle`, emits `tengu_session_renamed`). On load/resume CC derives the title by scanning the transcript for the last `type:"custom-title"` entry. CLI flag `-n <name>` sets it at startup; picker rename is Ctrl+R.

**No hook *output* field exists** — there is no `sessionName`/`sessionTitle` in any `hookSpecificOutput` (`SessionStart` outputs are only `additionalContext`/`initialUserMessage`/`watchPaths`). The feature was requested and closed `not_planned` (issue #21859, via #24119/#24872). Confirmed both from docs and by grepping the v2.1.150 binary: `customTitle` appears only in session-storage/transcript/resume code, never near hook-output parsing.

**Obscure mechanism (the "hook can set the title" trick):** every hook receives `session_id` + `transcript_path` on stdin, so a hook script can append a `custom-title` line to `transcript_path` to set the title programmatically. It takes effect on the next read/resume (appending does not mutate the live in-memory `currentSessionTitle` the way `/rename` does). Transcript path follows the [[reference_cc_project_dir_encoding]] mangling under `~/.claude/projects/<encoded>/<session-id>.jsonl`. Verified against CC 2.1.150 via `claude-code-guide` + binary inspection, 2026-05-25.
