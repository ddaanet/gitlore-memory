---
name: vibe-release-cc-only
description: "the gitlore release has no executor in a Vibe session: `edify:release` is a Claude Code skill and is unmounted under Vibe, so asked to release from a Vibe session, report the cascade as my human partner's action to run in a CC session — approximating it (dispatched review children, `just release`) is not the release"
metadata:
  node_type: memory
  type: reference
---

A Vibe session cannot perform gitlore's release. `edify:release` is a Claude
Code skill and is unmounted under Vibe — the session has no cascade to load
and no authority to invent one. Asked to release from a Vibe session, report
the cascade (`just release`, the review rounds, the triage) as my human
partner's action to run in a CC session; do not approximate it with dispatched
review children or a bare `just release`. The plugin's Vibe port
(`vibe-native-plugins`) ships skills and hooks to users; it does not carry the
release tooling into the Vibe session, and the port's completion is not a
release.
