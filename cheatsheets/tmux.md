# tmux Cheat Sheet

_Last updated: 2026-01-08_

## Notation

- **prefix** = `C-b` (default)  
- `prefix x` means: press `C-b`, release, then press `x`
- `C-x` = **Ctrl** + `x` (similarly, `M-x` = **Alt/Meta** + `x`)
- `prefix :` opens the tmux command prompt (type a tmux command, then press Enter)

## Start / attach (shell)

```sh
tmux                      # start a new session
tmux new -s NAME          # start a named session
tmux new -A -s NAME       # attach if it exists, otherwise create (recommended)
tmux ls                   # list sessions
tmux attach -t NAME       # attach to a session
tmux kill-session -t NAME # kill a session
tmux kill-server          # kill all sessions (stop tmux server)
```

## Essentials

- `prefix ?` — help (show current key bindings)
- `prefix d` — detach (leave session running)
- `prefix :` — command prompt
- `prefix r` — redraw / refresh

## Sessions

- `prefix s` — list / switch sessions
- `prefix $` — rename current session

Command prompt examples:

- `:new -s NAME`
- `:rename-session -t OLD NEW`
- `:kill-session -t NAME`

## Windows (tabs)

- `prefix c` — create window
- `prefix w` — list / choose windows
- `prefix n` / `prefix p` — next / previous window
- `prefix 0..9` — jump to window by number
- `prefix ,` — rename window
- `prefix &` — kill window (confirm)

## Panes (splits)

Create splits:

- `prefix %` — split left/right
- `prefix "` — split top/bottom

Navigate:

- `prefix ←/→/↑/↓` — move to pane in that direction
- `prefix o` — next pane
- `prefix ;` — last pane (toggle)

Manage:

- `prefix x` — kill pane (confirm)
- `prefix z` — zoom/unzoom pane (temporary fullscreen)
- `prefix !` — move pane to its own window
- `prefix {` / `prefix }` — swap pane with previous / next
- `prefix Space` — cycle pane layouts

Resize:

- `prefix :resize-pane -L 5` — resize left by 5 cells (also `-R`, `-U`, `-D`)

## Copy mode (scrollback) and paste

- `prefix [` — enter copy mode (scrollback)
- `q` — quit copy mode
- `prefix ]` — paste the most recent buffer

Copy-mode key bindings depend on the active key table (**emacs** vs **vi**).  
To see the exact keys for your setup:

- `prefix ?` (general help), or
- `prefix :list-keys -T copy-mode` (emacs-style table), or
- `prefix :list-keys -T copy-mode-vi` (vi-style table)

If you enable vi-style copy mode (`setw -g mode-keys vi`), the most common keys are:

- `/` — search forward
- `Space` — begin selection
- `Enter` — copy selection

## Useful command prompt snippets (`prefix :`)

- `:list-keys` — show all bindings
- `:source-file ~/.tmux.conf` — reload config
- `:set -w synchronize-panes on` — broadcast input to all panes in this window
- `:set -w synchronize-panes off` — stop broadcasting

## Common options (`~/.tmux.conf`)

```tmux
# Larger scrollback
set -g history-limit 10000

# Mouse: scroll, select, resize (optional)
set -g mouse on

# vi keys in copy mode (optional)
setw -g mode-keys vi
```

## GNU screen mental map (optional)

- screen **session** ≈ tmux **session**
- screen **window**  ≈ tmux **window**
- screen **split**   ≈ tmux **pane**

Common screen-style prefix remap (optional):

```tmux
unbind C-b
set -g prefix C-a
bind C-a send-prefix
```
