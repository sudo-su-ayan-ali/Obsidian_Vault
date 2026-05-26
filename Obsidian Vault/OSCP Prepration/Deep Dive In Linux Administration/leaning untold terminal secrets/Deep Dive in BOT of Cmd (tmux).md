Here’s a **practical, beginner-friendly guide to using `tmux`** on Linux (including Arch).

---

## 1. What is tmux?

`tmux` is a **terminal multiplexer**:

- Multiple terminals in one window
    
- Split panes
    
- Persistent sessions (survive SSH disconnects)
    

---

## 2. Install tmux

```bash
sudo pacman -S tmux
```

---

## 3. Start tmux

```bash
tmux
```

You’re now inside a tmux session.

---

## 4. The tmux prefix key

By default:

```
Ctrl + b
```

You press **Ctrl+b**, release, then press another key.

Example:

```
Ctrl+b then c
```

---

## 5. Basic commands (ESSENTIAL)

### Create / manage windows

| Action          | Keys               |
| --------------- | ------------------ |
| New window      | `Ctrl+b c`         |
| Next window     | `Ctrl+b n`         |
| Previous window | `Ctrl+b p`         |
| List windows    | `Ctrl+b w`         |
| Rename window   | `Ctrl+b ,`         |
| Close window    | `exit` or `Ctrl+d` |

---

### Split panes

| Action           | Keys                  |
| ---------------- | --------------------- |
| Vertical split   | `Ctrl+b %`            |
| Horizontal split | `Ctrl+b "`            |
| Switch panes     | `Ctrl+b` + arrow keys |
| Close pane       | `exit`                |

---

### Resize panes

Hold:

```
Ctrl+b
```

Then press:

```
↑ ↓ ← →
```

(You can repeat arrows without re-pressing Ctrl+b)

---

## 6. Sessions (the killer feature)

### Create a named session

```bash
tmux new -s mysession
```

### Detach from session (leave it running)

```
Ctrl+b d
```

### List sessions

```bash
tmux ls
```

### Reattach

```bash
tmux attach -t mysession
```

---

## 7. Scroll & copy mode

Enter copy mode:

```
Ctrl+b [
```

- Scroll with arrow keys or PgUp/PgDn
    
- Exit: `q`
    

Copy text:

1. `Ctrl+b [`
    
2. Move cursor
    
3. Press `Space` (start selection)
    
4. Move cursor
    
5. Press `Enter` (copy)
    

Paste:

```
Ctrl+b ]
```

---

## 8. Kill sessions

Kill current session:

```bash
tmux kill-session
```

Kill specific session:

```bash
tmux kill-session -t mysession
```

Kill **all** sessions:

```bash
tmux kill-server
```

---

## 9. tmux config (optional but recommended)

Create config file:

```bash
nano ~/.tmux.conf
```

Minimal useful config:

```conf
# Enable mouse
set -g mouse on

# Faster escape
set -sg escape-time 0

# Better colors
set -g default-terminal "screen-256color"
```

Reload config:

```
Ctrl+b :
source-file ~/.tmux.conf
```

---

## 10. Everyday tmux workflow example

```bash
tmux new -s work
Ctrl+b %
Ctrl+b "
Ctrl+b d
tmux attach -t work
```

---

## Quick cheat sheet 🧠

```
Ctrl+b c   → new window
Ctrl+b %   → split vertical
Ctrl+b "   → split horizontal
Ctrl+b d   → detach
Ctrl+b [   → scroll
Ctrl+b ]   → paste
```

---

If you want:

- a **custom keybinding setup**
    
- a **tmux workflow for programming**
    
- or **tmux + SSH usage**
    

tell me what you’re using tmux for and I’ll tailor it 👍