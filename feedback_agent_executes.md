---
name: agent executes, scripts decide
description: Detection and decision logic must live in shell scripts, not in agent reasoning
type: feedback
originSessionId: 44430e43-5ab7-4730-8696-8993292db475
---
Detection logic, branching, and decision-making must live in shell scripts. The agent reads structured script output and acts on it — it does not reason its way to a decision.

**Why:** Ensures deterministic, testable, auditable behavior that doesn't vary with model version or context. Applies to: hook manager detection, remote provider detection, merge state checks, worktree detection.

**How to apply:** When implementing any detection or decision flow, write a shell script that outputs a structured result. The skill/hook calls the script and acts on its output. Never let the agent inspect files and decide — delegate that to the script.
