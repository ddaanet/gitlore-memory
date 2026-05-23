---
name: cc-subagent-approval
description: "Sub-agent approval gates should use a two-turn return → SendMessage(approval) pattern, not SendMessage-to-parent — sub-agents have no addressable parent in one-shot dispatch"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 39084536-e084-480e-981f-18bb2e894d63
---

When a parent dispatches a sub-agent via `Agent({subagent_type, prompt})` (one-shot), the sub-agent has **no addressable parent** for `SendMessage`. Attempting `SendMessage to: "parent"` or `"team-lead"` returns `No agent named 'X' is currently addressable`. A sub-agent that needs parent approval and tries to SendMessage will fail the call, and (without a strict "stop on failure" clause) will rationalize proceeding — exactly what the gate was meant to prevent.

**Correct pattern: two turns.**

1. **Turn 1 (dispatch):** parent calls `Agent({subagent_type, prompt})`. Sub-agent does prep work (no destructive ops), then **returns** the approval request as its final message. Parent captures the returned `agentId`.
2. **Turn 2 (resume):** parent reads the return message, decides, then `SendMessage({to: <agentId>, message: "approved" | "rejected: <reason>"})`. The sub-agent resumes from the SendMessage and either runs the destructive op (on approval) or re-does the prep with feedback (on rejection).

This works because SendMessage *to a known agentId* succeeds even after the sub-agent's first task completes; the harness resumes from transcript.

**Contract must include:** "If you are resumed without a clear approval/rejection signal, do not proceed. Report and stop." Plus an unconditional "never run the gated op in turn 1" — otherwise a sub-agent under auto-mode pressure will rationalize a "trivial case" exception.

Confirmed Plan 04 Step 3 (2026-05-22) — the old `agents/memory-merger.md` contract had the failed pattern and the sub-agent's own post-mortem identified the gap. Rewritten to two-turn flow; `commands/gitlore/resolve.md` dispatcher updated to match.

Related: [[cc-agent-discovery]] (the namespacing + caching rules that constrain how the contract can be tested in-session).
