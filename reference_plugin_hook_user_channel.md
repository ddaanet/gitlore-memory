---
name: reference-plugin-hook-user-channel
description: "systemMessage is the user-visible channel (works for SessionStart/PostToolUse); additionalContext is model-only; stderr only on non-zero exit (code 2 or --verbose); basis for D14. To report a hook ERROR use systemMessage + exit 0 — stdout JSON parses only on exit 0, so non-zero DISCARDS the message; only exit 2 blocks, and only at PreToolUse (PostToolUse can never block/undo)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: f781c2e5-7dc1-474b-818d-0ba97fea0231
---

Claude Code hook output channels, by destination (verified 2026-06-10):

- **`systemMessage`** (top-level field) — the user-visible channel. Works for the
  events gitlore uses (SessionStart, PostToolUse); gitlore's SessionStart
  launcher-guard warning already rides it successfully. This is the field to use
  for anything the human should see.
- **`hookSpecificOutput.additionalContext`** — injected into the model's context
  only; **never echoed to the user**. Silent by design.
- **stdout** — consumed as JSON; not echoed. **Parsed ONLY on `exit 0`.**
- **stderr** — ignored on `exit 0`; on non-zero exit, shown to the user only with
  exit code **2**, or with `--verbose` for other codes (a "hook error" label
  appears). SessionStart is non-blocking — continues regardless of exit code.

Consequence for gitlore (D14): route user-facing notices through `systemMessage`,
not stderr-on-exit-0 and **not** via the agent (`additionalContext` "tell the
user…" is model-dependent on the hot path, against NFR1/D7). A `systemMessage`
can sit alongside an `additionalContext` in the same hook JSON.

## Reporting an ERROR from a hook: `exit 0` is required, not merely tolerated (2026-07-15)

The naive instinct — "fail loudly ⇒ exit non-zero" — is **backwards here**, and the
two facts above are why:

- stdout JSON parses **only on `exit 0`**, so a non-zero exit **discards your
  `systemMessage`** — the only reliably user-visible channel. Exiting non-zero to
  "surface" an error makes it *less* visible, leaving only stderr (exit 2 /
  `--verbose` / debug log).
- So the honest error path is **`systemMessage` + `exit 0`** — which is exactly
  what D14 concluded when it moved SessionStart's errors off `stderr` + `exit 1`.

**Blocking asymmetry** (don't conflate "don't exit 2" with "don't error"):

| event | can it block? |
|---|---|
| `PreToolUse` | **`exit 2` blocks the tool outright.** Other non-zero → action proceeds + "hook error" notice. |
| `PostToolUse` | **Nothing can block** — docs: *"PostToolUse hooks can't undo actions since the tool has already executed."* `exit 2` is mere feedback to the model; the exit code buys nothing. |

So a spec saying "exit 0 on every path so we never block a Write" is over-broad
reasoning that lands on the right answer: `exit 0` is right, but for *visibility*,
not for non-blocking. `|| true` and a bare `|| exit 0` on a fallible command are
dishonest error paths — check status explicitly, report on `systemMessage`, exit 0.
Applied in D17's index-sync hooks; see [[feedback_prohibition_vs_procedure]].

**Unresolved tension — re-verify before relying on it.** The 2026-06-10 finding above
says stderr reaches the user only on exit 2 / `--verbose`. The current docs
(fetched 2026-07-15, `code.claude.com/docs/en/hooks-guide.md`) instead describe *any*
non-zero as showing a `<hook name> hook error` notice + the first stderr line in the
transcript, with full stderr to the debug log. Possibly a CC behavior change, or
"transcript" meaning the transcript view rather than the inline UI. It does not
change the conclusion — `systemMessage` + `exit 0` is correct under **both**
readings — but the stderr claim itself is stale-suspect.

See [[feedback_agent_executes]], [[feedback_verify_delegated_citations]].
