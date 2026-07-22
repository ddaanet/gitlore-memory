---
name: feedback_no_stderr_suppression
description: "drop `2>/dev/null`; prefer a guard, then capture-and-match, then a redirect justified inline; never guess in place of git's message"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5f90d295-aa18-451c-8ed2-514d78c5207b
  modified: 2026-07-20T13:40:10.449Z
---

David: "Remove `2>/dev/null` everywhere, unless failure is normal and error
message expected... and even there, it would be safer to surface the error
message unless it matches expectation."

**Why:** a blanket redirect silences one anticipated message and every
unanticipated one with it. Two real defects in gitlore hid behind exactly that:
both `git push` sites assumed failure meant unreachable-or-divergence, so a
protected branch or pre-receive decline started a merge resolution that could
not help — with git's explanation discarded; and a suppressed `ls -A` let an
unreadable directory read as "empty" so the installer proceeded into it.

**How to apply:** prefer, in order —
1. **Guard away the expected failure** so no redirect is needed and what remains
   on stderr is real: `[ -f "$file" ]` before a reader, `[ -e path/.git ]` before
   a submodule `git -C`, `rev-parse -q --verify MERGE_HEAD` before `merge
   --abort`, `-q --verify` instead of bare `rev-parse`.
2. **Capture and match** when the command has several failure modes you treat
   differently — `err=$(cmd 2>&1)`, then `case "$err"` on a verified
   discriminator (see [[reference_git_stderr_and_parsing]]).
3. **Keep the redirect only when provoking the error IS the mechanism** (a write
   probe, `python3 -c 'import yaml'` feature detection) or when every failure
   mode is normal and already answered by a safe default (`gh repo view` for
   visibility, a shim wrapping every invocation outside any repo) — and write the
   reason inline.

Never replace git's message with a guess: worktree-remove appended "(locked or
uncommitted)" to every failure, which was wrong whenever it wasn't that.

Related: [[feedback_whitespace_safety]], [[reference_git_stderr_and_parsing]].
