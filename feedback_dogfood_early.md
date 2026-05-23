---
name: feedback-dogfood-early
description: Run new code on the actual production target as soon as it ships — not just on test fixtures — because self-application surfaces bugs that synthetic harnesses miss
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7dd65fa7-001f-45cd-a975-fcab4433bdf5
---

When a self-contained slice ships, run it on the real target the same day. Don't wait for "the next plan that builds on this" to validate the previous one. Treat surprises as patches to the just-shipped plan, not as scope for the next.

**Why:** Test fixtures encode what you *expected* to vary. The real environment includes things you didn't think to parameterize. Plan 01 of gitlore shipped with 64 passing tests; within minutes of running `/gitlore:install` on the gitlore repo itself, three bugs surfaced that the fixture-based suite missed entirely:

1. Path-mangling assumption (CC's `~/.claude/projects/<name>/` rule was reverse-engineered correctly only because dogfooding forced the question — fixture tests used a synthetic mangling that matched the implementation, not the real CC behavior).
2. Install's "review the staged changes" contract was broken — `.claude/settings.json`, `.gitignore`, and `.claude/gitlore-hook-setup` were written but never staged. Tests asserted file existence, not staging state.
3. `git add memory` silently refused to stage as gitlink in modern git, emitting the "embedded git repository" advice. Fixtures didn't notice because they checked the final `.gitmodules` content, not the staging path.

Each one would have multiplied complexity into the next plan if not caught — Plan 02 would have built on top of a broken Plan 01.

**How to apply:** Tests answer "did the contract hold for my mocks?" Dogfooding answers "did the contract hold for reality?" Both matter. After shipping a unit big enough to be useful on its own, find the smallest real target and run it. Related: [[feedback-outside-in-tdd]] — dogfooding is the production-target case of black-box validation.
