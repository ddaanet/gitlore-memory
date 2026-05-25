---
name: revdiff-iterm-tmux-cc-popup
description: "revdiff launcher's tmux display-popup is invisible under iTerm tmux-CC control-mode + multi-client setup; use tmux new-window instead"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 585314ef-4d01-4c5a-b974-557581da793c
---

revdiff's launcher (`launch-revdiff.sh`) uses `tmux display-popup` when `$TMUX` is set. In this user's setup it renders invisibly when:

- multiple tmux clients are attached to the same session (e.g. a stale long-running `tmux attach` on pts/N plus the active iTerm tmux-CC client), and/or
- the active client is iTerm in tmux control-mode (`tmux -CC`), which doesn't reliably render display-popup overlays as iTerm windows

Symptom: launcher reports the popup opened, `ps` shows `revdiff` running attached to the tmux server, but the user sees no new window.

Diagnostic:
```sh
tmux list-clients   # multiple clients on same session is the smell
ps -ef | grep revdiff   # confirms revdiff is alive but invisible
```

**Workaround:** open revdiff in a new tmux window so iTerm tmux-CC renders it as a new iTerm window/tab. **Use a script file, not an inline command** — the harness's bash uses `eval` with nested-quoting that silently mangles an inline `tmux new-window "cmd"` (window-creation appears to succeed, `tmux list-windows` shows nothing, and the outer `wait-for` hangs forever). Two abandoned-session attempts hung this exact way before swapping to a script file made it work.

Launcher script (write to a temp file, chmod +x):
```sh
#!/bin/bash
# args: $1=output $2=chan $3=desc-file $4=ref
set -u
OUTPUT="$1"; CHAN="$2"; DESC="$3"; REF="$4"
trap 'tmux wait-for -S "$CHAN" 2>/dev/null' EXIT   # signal even on user kill-window
REVDIFF_EXIT_CODE_ON_ANNOTATIONS=true /usr/local/bin/revdiff \
  --output="$OUTPUT" --description-file="$DESC" "$REF"
```

Caller (Bash tool with `dangerouslyDisableSandbox: true`, timeout 600000):
```sh
output="/tmp/revdiff-output-plan05-$$.md"
chan="revdiff-done-plan05-$$"
tmux new-window -t claude-rc: -n revdiff -P -F '#{window_id} #{window_index}' \
  /tmp/revdiff-launch.sh "$output" "$chan" "$DESC" "$REF"
tmux wait-for "$chan"      # blocks until script's EXIT trap fires
cat "$output" 2>/dev/null  # may not exist — revdiff only writes on annotations
```

**Gotchas:**
- The output file is **only created when there are annotations**. `cat "$output"` returning exit 1 is the no-annotations case — treat as "review complete, approved", not failure. **Empty revdiff exit is the user's explicit approval signal** (they intentionally `q` without annotating); don't second-guess it by asking "did you really mean to approve?".
- The `trap … EXIT` is the fix for the kill-window-instead-of-`q` hang in the old workaround.
- The harness auto-backgrounds long-running Bash tasks; that's fine — verify the window is open with `tmux list-windows -t claude-rc` and wait for the task-notification.

Related: [[reference-zellij-ipc-sandbox]] — same family of TUI-overlay-fails-under-sandbox/multi-client issues. Bash invocations need `dangerouslyDisableSandbox: true` for tmux IPC socket access.
