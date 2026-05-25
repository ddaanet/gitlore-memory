---
name: reference_cross_repo_push_auth
description: "pushing to a repo outside the session's working dirs is blocked by the auto-mode classifier; /add-dir authorizes it — removing ask rules does not"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6dbcf2d7-32f4-41a7-ba25-1cb68fa46c28
---

When working in gitlore, pushing to the sibling `~/code/claude-plugins` marketplace repo (needed whenever a plugin entry or version changes) is **denied by the auto-mode classifier** — reason "external repo outside the trusted source control org" — even though pushes to gitlore itself (the session cwd) go through.

What does *not* fix it: removing the `Bash(git push:*)` **ask** directives from user config. Ask rules only control prompting; their absence is not authorization, so the classifier still denies.

What fixes it: either an explicit **allow** rule (`"permissions":{"allow":["Bash(git push:*)"]}`), or — cleanest — `/add-dir ~/code/claude-plugins`, which makes the repo a session working directory and the classifier then trusts pushes to it. Confirmed 2026-05-23: after `/add-dir`, the identical `git -C ~/code/claude-plugins push origin main` succeeded.
