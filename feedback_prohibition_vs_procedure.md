---
name: feedback_prohibition_vs_procedure
description: "split always-on disclosures by timing — front-load prohibitions, defer procedures to their trigger"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: f816f5f1-5339-4cd9-964a-75fe6f06682a
---

When deciding what an always-on context channel (SessionStart additionalContext, a system prompt, a standing reminder) should carry, split the content by *timing need*, not by topic:

- **Prohibitions** — guardrails against an action the agent would otherwise take unprompted (e.g. "never `git commit` inside the memory submodule"). MUST be front-loaded: nothing downstream would trigger their disclosure in time.
- **Procedures** — recipes only needed at the moment of acting (the 4-step persist flow at commit time). Do NOT preload them. A recipe resident in context reads as "a process you must run," so the agent runs ceremony — e.g. pausing for explicit user approval *before even writing a memory file* — when the happy path is meant to be seamless. Surface procedures just-in-time, at their actual trigger (the pre-commit hook's own output when a commit is blocked, `/gitlore:resolve` on divergence).

**Why:** the gitlore SessionStart disclosure front-loaded both the prohibition and the four-step persist procedure; the procedure distorted behavior (over-worrying about memory updates, ceremony before an ordinary edit). User: "why do you worry so much about memory updates all the time? The gitlore thing is intended to be mostly seamless."

**How to apply:** keep the prohibition line in the always-on channel; replace a preloaded procedure with at most a one-line reassurance of the seamless happy path (which defuses over-worry rather than feeding it), and route the full recipe to the surface that fires when the action actually happens. Generalizes [[feedback_self_contained_directives]] and [[reference_plugin_hook_user_channel]]; precedent is gitlore's [[feedback_recall_checkpoints]] / surface-detail-when-trigger-fires pattern.
