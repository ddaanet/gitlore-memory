---
name: plan-length-matches-work
description: "Plan doc length should match the executable work — don't write 700 lines for 7 commands"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ce981dd4-19be-4e5d-baa0-0c4f080d1af5
---

When the work is concrete ("edit N files + run M commands"), the plan doc should be that small. Padding with architecture diagrams, lessons-learned openers, multi-tier dogfood narratives, and self-review checklists feels productive but adds review burden without changing the work.

**Why:** Plan 04 (`docs/plans/2026-05-22-04-marketplace-install.md`) was 700 lines for what reduces to 7 executable steps (edit two manifests, validate, push twice, dogfood twice). User reaction: "why 700 when 7 would do?" The exposition didn't help the LLM execute — it already has context — and it defeated review because nobody can eyeball 700 lines.

**How to apply:**
- Before writing a plan doc, list the actual executable steps. If <10 commands and no genuine design decisions, skip the spec-style template entirely.
- Plans 01–03 had real architectural decisions (FRs, NFRs, alternative analysis) — the spec template fit them. Plan 04 was packaging only. Same template didn't fit; I forced it.
- For packaging/configuration/distribution plans: a short checklist (≤30 lines) with steps + open decisions + scope notes is plenty.
- Reserve full spec docs for plans with non-trivial design decisions worth recording across sessions.

Related: [[feedback-plan-late]] (don't plan beyond next iteration); [[feedback-large-docs-review]] (large docs defeat eyeball review).
