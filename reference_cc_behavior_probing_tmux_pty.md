---
name: cc-behavior-probing-tmux-pty
description: "How to probe interactive-only CC behavior (that --print can't reach) — tmux PTY harness + session transcript JSONL inspection + --disallowedTools tool-gating"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7affbb95-e26e-4294-8835-9c686a979955
---

To test Claude Code behavior that `--print` can't reach (interactive-only recall, UI affordances like "Recalled 1 memory"), drive a real interactive `claude` in a tmux pane and read the ground truth from the session transcript.

**Harness (all tmux calls need `dangerouslyDisableSandbox: true`):**
- `tmux new-session -d -s S -x 400 -y 50 bash` — run the pane under **bash**, not the login shell, and make it **wide**: fish wraps long paths and mangles `send-keys`, executing truncated commands. Put the launch invocation in a small launcher script to dodge nested-quote hell (`--settings '{"autoMemoryDirectory":…}'` survives bash→shell handoff badly).
- `send-keys -l "bash launch.sh"` then a separate `send-keys Enter`; `sleep ~15` for boot; send the query; `sleep ~30` for processing; `capture-pane -p -S -80`. Kill with `tmux kill-session`.

**Ground truth = the transcript JSONL**, not the pane scrape. `~/.claude/projects/<encoded-cwd>/<uuid>.jsonl`; the interactive session is the newest file *excluding your own* (clock skew makes absolute mtimes confusing — use relative order). Grep for `"type":"tool_use"` / `tool_result` and the `Read` input to distinguish an agent-issued/auto-recall **Read** from a passive injected context block. That distinction is what settled recall = tool-gated Read (see [[cc-memory-retrieval-agentic]]).

**Two levers:**
- `--disallowedTools "Read,Grep,Glob,Bash,…"` hard-disables tools to test whether a behavior is tool-gated. **Comma-separated, placed LAST** on the line — it's variadic and will otherwise swallow the prompt as tool names. Invalid names (LS, NotebookRead) warn but don't abort.
- `--settings '{"autoMemoryDirectory":"<scratch>"}'` cleanly **overrides gitlore's own session-start hook** (verified: the active index was the scratch fixtures despite the "memory ready" banner), so you can probe throwaway fixtures with zero real-memory contact.

Related tmux-under-CC gotchas: [[zellij-ipc-sandbox]], [[revdiff-iterm-tmux-cc]].
