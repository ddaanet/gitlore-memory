---
name: skill-eval-exercises-the-trigger
description: "a live-agent eval that names the skill in its prompt, or leaves the trigger token in the prompt where CC's prompt-time classifier fires, passes with the skill absent; assert the `Skill` tool_use in `transcript.jsonl`, and the order against the call that surfaced the trigger"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 862d9831-f887-4bc6-86c7-e1d36349ec07
  modified: 2026-08-08T22:19:08.013Z
---

An eval that drives the real CLI grades a skill only if the skill could have
been the difference. Three shapes look like coverage and are not:

- **The prompt names the skill.** "Use the `foo:bar` skill" removes the
  judgement being graded — the description's triggering is the thing under
  test. Phrase the prompt the way a user would, in the description's own
  conversational wording.
- **The trigger token is in the user prompt.** Claude Code's own prompt-time
  classifier fires there, so the harness can satisfy every assertion with the
  skill absent. Put the trigger where only the running task reaches it — the
  output of a command the agent runs — and assert **order**: the fetch came
  after the call that surfaced it. Order is the only thing separating a
  mid-task fetch from a prompt-time one; the answer and the repo are identical.
- **The assertion keys on an artifact of the mechanism** — an IPC file the
  agent writes, a hook's marker. It proves the mechanism ran, so it stops
  discriminating the moment the mechanism changes, and a suite that still
  passes reads as coverage that is no longer there.

Assert the `Skill` tool_use itself: an agent-invoked skill appears in the
transcript as one ([[jsonl-slash-command-shape]]), and without that assertion a
model that simply does the right thing unaided scores as a pass.

A skill body edited in the working tree is not what a live session invokes —
the plugin cache serves the old one ([[stale-plugin-code]]) — but an eval that
copies skills into its fixture repo runs the new body, so the two disagree on
purpose. Related: [[test-the-invocation-path]], [[green-is-not-evidence]].
