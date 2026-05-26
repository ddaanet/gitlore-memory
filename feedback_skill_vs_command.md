---
name: skill-vs-command
description: When to use a self-triggering skill vs an explicit command for plugin agent actions
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6779ad0c-a404-41bb-9407-f4f60e20647b
---

Use a **skill with a trigger description** when the invocation condition is mechanical and detectable from output (e.g. a hook failure pattern in stderr). Use a **command** only when the user must explicitly initiate the action.

For gitlore: `resolve` is a skill because its trigger (`gitlore: memory merge prepared` in hook stderr) is mechanical — the agent recognizes it and invokes the skill automatically during commit flow. The skill's `description` frontmatter names the trigger pattern explicitly.

**Why:** Commands require the user to know when to run them. Skills with trigger patterns invoke themselves when the condition is met — zero user intervention, consistent with D7 (scripts decide, agent handles language). Missed triggers can't happen.

**How to apply:** When designing a new plugin command, ask: "Is there a detectable mechanical condition that should trigger this?" If yes, make it a skill with that condition in the description. If the user must consciously decide to run it (install, one-time setup), keep it a command. Skills and commands both live in `commands/` in CC — the distinction is in the frontmatter `description` and whether the skill documents entry modes vs a linear procedure.

See also: [[agent executes, scripts decide]]
