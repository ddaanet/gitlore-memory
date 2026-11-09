---
name: probe-state-normal-path-before-fixing
description: "Before fixing a TODO item that names a state, check whether normal operation produces it; a hand-made fixture is not evidence that it does"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 418c4f51-9898-43be-b4f5-544718d0bba3
  modified: 2026-11-09T14:26:09.731Z
---

A TODO item that names a bad state ("a tier ahead of its pin re-adopts a
deleted line") is not evidence that normal operation reaches that state. Before
designing a fix, name the operations that produce the state and check each
against the code; a bats fixture that constructs the state by hand (a
`reset --hard` in a store, an `update-index --cacheinfo`) proves the symptom,
not the path to it.

**Why:** A six-round review loop landed staging, a pre-move reset and a dirty
gate for a state that only a killed run or hand git produces; my human partner
challenged the algorithm's premise afterwards and the slice was reversed. Each
round had real probes, so the round count read as progress.

**How to apply:** Open the slice by listing the producers of the state, with
the window each leaves, and say which are normal operation. If none is, the
question is whether to refuse the state with a named remedy, not how to merge
it. Put that fork to my human partner before the first commit. Related:
[[fix-is-the-rule-when-cases-are-open]].
