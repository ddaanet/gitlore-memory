---
name: feedback_strict_sandbox_git
description: "In strict sandbox mode, git write operations fail even with dangerouslyDisableSandbox; user must run them via !"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9ac5ddb7-c3a0-4f04-b949-4008965560dd
---

When the user has enabled strict sandbox mode (`/sandbox` → "Strict sandbox mode"), `dangerouslyDisableSandbox: true` is blocked. Git write operations (`git add`, `git commit`) then fail with "Read-only file system" on `.git/index.lock`.

**Why:** Strict mode disables the sandbox override mechanism entirely; the filesystem restriction is enforced at the OS level and cannot be bypassed by the agent.

**How to apply:** When strict sandbox is active and a git write is needed, either ask the user to run the command via `! git …`, or ask them to toggle sandbox mode first (`/sandbox` → unsandboxed fallback). Do NOT retry the same sandboxed command or try workarounds.
