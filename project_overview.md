---
name: gitlore project overview
description: What gitlore is, current state, and next steps
type: project
originSessionId: 44430e43-5ab7-4730-8696-8993292db475
---
gitlore is a Claude Code plugin that makes Claude's auto-memory system versioned, shared, and git-backed. Memory lives in a git submodule inside the project repo, synchronized across worktrees via a local `live` trunk branch.

**Why:** Replace the default per-machine `~/.claude/projects/<hash>/memory/` with a tracked, pushable, multi-session memory store.

**Current state (2026-04-23):** Design doc fully reviewed and updated at `docs/design.md`. 25-item structured review via `ddaa:proof` produced 61 decisions, all applied. Doc expanded from 260 to ~503 lines: added FRs (install-time disclosure, per-commit review gate, coexistence, recovery), NFRs (graceful degradation, overrides), rewrote branch model, install flow, resolve skill (now covers branch-vs-live + local-vs-remote divergence with sub-agent synthesis), configuration, hooks. Added D9 (sub-agent + experimental flag).

**Next step:** New session → invoke `superpowers:writing-plans` skill pointing at `docs/design.md` → implementation plan in `docs/plans/`.

**How to apply:** When resuming work, read `docs/design.md` first. Note that it's a living design doc — functional reqs, non-functional reqs, architecture, design decisions, rejected alternatives, changelog. Design review already completed; treat the doc as implementation-ready unless a new gap surfaces.
