---
name: cc-memory-retrieval-agentic
description: "recall = tool-gated Read steered by root MEMORY.md (\"Recalled 1 memory\"); nested team/MEMORY.md NOT always-loaded"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7affbb95-e26e-4294-8835-9c686a979955
---

Claude Code's file-based auto-memory RETRIEVAL is agentic, not a mechanical "load MEMORY.md and stop." Established from the reverse-engineered system prompts in `~/code/claude-code-system-prompts/system-prompts/` (ccVersion ~2.1.147–2.1.198):

- A **separate per-query classifier** (`agent-prompt-determine-which-memory-files-to-attach.md`) is handed a manifest of `filename + description` pairs plus the live user query, and returns ≤5 filenames it is *certain* are relevant. Told to be conservative — especially with `user`/`project` memories (match on what the question IS ABOUT, not keyword overlap) — and not to re-select within a conversation.
- Selected files' contents are injected as `Contents of <path>` system-reminders (`system-reminder-memory-file-contents.md`; a **distinct `system-reminder-nested-memory-contents.md`** exists for non-root paths → subdir files ARE attachable).
- **Native two-tier structure exists:** a `team/` subdirectory + a combined index pointing at both `file.md` and `team/file.md` (`system-prompt-combined-memory-index-pointer-instructions.md`, `system-prompt-dream-team-memory-handling.md`). Anthropic's own shared-memory surface. No arbitrary-depth traversal or `[[link]]`-following at retrieval is evidenced.
- `autoMemoryDirectory` is a **single path**; no public `teamMemoryDirectory`/multi-root key (docs confirm). Default when unset: `~/.claude/projects/<hash>/memory/`, project-scoped only — no user-scoped default.

Two unknowns NOT answerable from prompts (build behavior), gating gitlore's global-memory design (see [[gitlore-global-memory-investigation]]): **U1** — does the classifier match on hand-written `MEMORY.md` one-liners or on per-file `description:` frontmatter; **U2** — is a `team/` subdir enumerated, and does a `team/` index pointer bulk-load into the always-loaded ~200-line blob or surface only on-selection. Refines [[feedback_memory_retrieval_in_practice]] (whose "never read spontaneously" was likely observing the classifier's deliberate conservatism, not an absent mechanism).

**RESOLVED empirically (2026-07-14, sentinel-introspection experiment, ccVersion ~2.1.x, via `claude --print --settings autoMemoryDirectory=<scratch>`):**
- **U0/U3** — per-file attachment DOES fire under `--print` and the body is surfaced (positive control 4/4; the actual `Contents of <path>` reminder is what the model quotes). "never read spontaneously" is NOT absolute.
- **U1 — BOTH sources feed the classifier manifest.** A query matching only the frontmatter `description:` (but not the `MEMORY.md` one-liner) attached the file 2/2; a query matching only the `MEMORY.md` one-liner (but not frontmatter) also attached it 2/2. So either lever drives selection.
- **U2a — `team/` IS enumerated** (indexed `team/` file attached 4/4). Depth ≥2 AND index-less pure-frontmatter enumeration also work (unindexed `team/sub/` file attached 3/4).
- **U2b — nothing bulk-loads bodies.** Always-loaded footprint = `MEMORY.md` index blob ONLY; negative-control query loaded zero bodies; introspection confirmed no file body present pre-selection. A `team/` index pointer costs one index line, NOT its body. Global tier is near-free at baseline.
- **Bonus — retrieval reliability tracks whether the file is in `MEMORY.md`.** Indexed files (root or `team/`) surfaced 100% (4/4); an unindexed subdir file 75% (3/4). Practical rule: list every file you want reliably retrieved in `MEMORY.md`; don't rely on pure-frontmatter enumeration for anything important.

**CORRECTION (same 2026-07-14 experiment, tools-disabled follow-up) — the `--print` mechanism is FRONT-AGENT TOOL-READ, not background injection.** Re-ran every probe with file tools hard-disabled (`--disallowedTools "Read,Grep,Glob,Bash,…"`). Body presence collapsed from ~100% (tools on) to ~0% (flumox 0/9, bornix 1/5, quxbar/zorple/narwhal 0/each). Since presence is gated almost entirely on TOOL availability, bodies are FETCHED VIA A TOOL-GATED READ, not pushed through a passive injection channel that bypasses tools.

**INTERACTIVE CONFIRMED (tmux PTY harness, CC v2.1.209) — same mechanism as `--print`, and the exact shape is now pinned down.** Drove a real interactive session against the same scratch dir (`--settings autoMemoryDirectory` overrides gitlore's own hook cleanly — verified the active index was the scratch fixtures, not the real `memory/`). Findings, cross-checked against the session's transcript JSONL:
- **Auto-recall IS real and DOES fire interactively.** Tools-on, a clean "how do I calibrate the flumox regulator?" produced the body sentinel and the UI line **"Recalled 1 memory"**. The transcript shows this = exactly ONE `Read` tool_use of `flumox.md` → tool_result, with an EMPTY thinking block (no model deliberation). So an automatic recall step selects the file and issues a Read on the agent's behalf; "Recalled 1 memory" is the UI label for that recall-Read (distinct from a normal model-decided `Read(path)`).
- **It's tool-gated.** Tools-off (`--disallowedTools Read,Grep,…`), across two turns, the body NEVER entered context — transcript shows ZERO Read calls, and the agent stated plainly the index holds "one line per memory and never carries the memory's content." Disabling Read disables recall itself. So "tools off → no body" reflects recall-Read being blocked, NOT the absence of a recall mechanism.
- **The bodies arrive as tool_results, not passive system-reminders.** There is no separate invisible injection channel; recall == a Read tool call surfaced nicely.
Reliability gap has a clean cause: indexed file = recall has a direct pointer to Read (100%); unindexed/frontmatter-only (quxbar, narwhal-in-`team/sub/`) = must be discovered by Grep/Glob first, also tool-gated, hence less reliable (75%). **Design-robustness (for [[gitlore-global-memory-investigation]]):** a `MEMORY.md` pointer is what makes recall reliable, so composing global files into `MEMORY.md` is the correct and mode-independent lever; the two-tier design holds. Budget fact also holds: only the index is ever always-loaded; bodies are pulled on demand.
