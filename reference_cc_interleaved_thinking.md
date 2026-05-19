---
name: Claude Code interleaved thinking
description: CC supports interleaved thinking blocks hidden from the TUI; model support varies
type: reference
originSessionId: cf8270f1-627f-41f0-96b6-295fcfe5ef90
---
Claude Code supports interleaved thinking — multiple thinking blocks mixed with text/tool_use blocks in a single assistant response. The TUI hides thinking blocks from the user by default; only text and tool_use blocks render.

**Model support (confirmed 2026-04-23, may drift):**
- Opus 4.7, Sonnet 4.6: adaptive thinking enabled by default
- Sonnet 4.5, Haiku 4.5: require `alwaysThinkingEnabled: true` in settings.json
- Claude Mythos Preview: interleaved thinking automatic

**How to apply:** When designing agent skills where internal scaffolding would clutter the user's view, use interleaved thinking as the hidden channel. No special config needed on supported models. For older models, set `alwaysThinkingEnabled` in settings.json. Verify current availability before relying on it in a design (feature flags and model versions evolve).

Docs: https://platform.claude.com/docs/en/build-with-claude/extended-thinking
