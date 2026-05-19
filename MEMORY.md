# Memory Index

- [gitlore project overview](project_overview.md) — current state, next steps, design doc location
- [gitmoji commit convention](feedback_gitmoji.md) — conventional commit prefixes required, hook from devddaanet
- [living design doc structure](feedback_design_doc_structure.md) — six-section pattern for docs/design.md
- [agent executes, scripts decide](feedback_agent_executes.md) — detection logic in shell scripts, not agent reasoning
- [fact-check before incorporating](feedback_fact_check.md) — verify user corrections against training knowledge, don't just defer
- [loose generation + post-hoc fix](feedback_loose_generation.md) — let generator scaffold freely, constrain at artifact boundary
- [user: technical depth on LLM mechanism](user_technical_depth.md) — expects rigorous attention/cost/training claims, not hand-waving
- [CC interleaved thinking](reference_cc_interleaved_thinking.md) — hidden scaffolding channel; model-dependent config
- [git status sandbox artifacts](feedback_git_status_sandbox.md) — run git status unsandboxed in this repo
- [plan as late as possible](feedback_plan_late.md) — don't plan beyond next iteration; write Plan N+1 after Plan N ships
- [CC project-dir encoding](reference_cc_project_dir_encoding.md) — `~/.claude/projects/<name>/` mangling rule; reverse-engineered, edify's impl is incomplete
- [outside-in TDD when architecture is fixed](feedback_outside_in_tdd.md) — red e2e first, drive units by failure; black-box tests survive refactors
