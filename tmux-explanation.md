# tmux Configuration Explanation

## What is tmux?

**tmux** = "Terminal MUltipleXer". It's a program that runs **inside** your terminal and lets you:

1. **Have multiple terminal windows/panes** in a single SSH connection
2. **Keep sessions alive** even after you disconnect — processes keep running in the background
3. **Re-attach** to a running session from anywhere (SSH, VS Code, another device)

Think of it like this: without tmux, if your SSH connection drops, everything running in that terminal dies. With tmux, your session persists on the Raspberry Pi and you just re-attach.

Key concepts:
- **Session** — a named collection of windows (e.g. `claude-idea`)
- **Window** — like a tab inside the session
- **Pane** — a split within a window

---

## The Full Flow

### 1. `~/.tmux.conf` — Global tmux config (loaded when tmux starts)

```
set mouse on
set -g terminal-overrides 'xterm*:smcup@:rmcup@'
```

| Line | What it does |
|------|-------------|
| `set mouse on` | Enables mouse support globally — you can click to switch panes, scroll with the wheel, and resize panes by dragging borders |
| `set -g terminal-overrides 'xterm*:smcup@:rmcup@'` | Disables the "alternate screen" feature. This means when you scroll your mouse wheel, it scrolls the terminal's scroll buffer instead of tmux's own scroll mode. Without this, scrolling in tmux can feel weird in some terminals |

This file is loaded automatically whenever any tmux server starts.

---

### 2. `~/.bashrc` — The SSH auto-attach logic

```bash
if [ -z "$TMUX" ] && [ -n "$SSH_CONNECTION" ]; then
    session="claude-$(basename "$PWD")"
    tmux has-session -t "$session" 2>/dev/null || tmux new-session -d -s "$session" -c "$PWD" "claude"
    tmux attach-session -t "$session"
fi
```

This runs every time you open a bash shell. Breaking it down:

| Condition | Meaning |
|-----------|---------|
| `[ -z "$TMUX" ]` | "Am I NOT already inside tmux?" — prevents nesting tmux inside tmux |
| `[ -n "$SSH_CONNECTION" ]` | "Did I connect via SSH?" — only runs for SSH connections, not local terminals |

If both are true:

1. **`session="claude-$(basename "$PWD")"`** — builds a session name from the current directory. If you're in `/home/pi/idea-proposal`, the session becomes `claude-idea-proposal`

2. **`tmux has-session -t "$session" 2>/dev/null || tmux new-session -d -s "$session" -c "$PWD" "claude"`**
   - Check if a session with that name already exists
   - If it does NOT exist: create one (`-d` = detached/background, `-s` = session name, `-c` = starting directory, `"claude"` = run the `claude` command inside it)
   - If it already exists: skip creation

3. **`tmux attach-session -t "$session"`** — attach your SSH terminal to that session so you can see and interact with it

**Result:** The moment you SSH into the Pi, you are automatically dropped into the right tmux session with `claude` running, named after your project folder.

---

### 3. VS Code `settings.json` — Per-project terminal config

**`idea-proposal/.vscode/settings.json`:**
```json
{
    "terminal.integrated.defaultProfile.linux": "tmux",
    "terminal.integrated.profiles.linux": {
        "tmux": {
            "path": "bash",
            "args": ["-c", "tmux attach -t claude-idea 2>/dev/null || tmux new -s claude-idea; tmux set -g mouse on"]
        }
    }
}
```

This tells VS Code: "When you open a terminal in this project, don't open a plain bash — open this custom `tmux` profile instead."

The command VS Code actually runs is:
```bash
bash -c "tmux attach -t claude-idea 2>/dev/null || tmux new -s claude-idea; tmux set -g mouse on"
```

Step by step:
1. `tmux attach -t claude-idea 2>/dev/null` — try to attach to the existing `claude-idea` session (suppress errors if it doesn't exist)
2. `|| tmux new -s claude-idea` — if attach failed (session didn't exist), create a new session named `claude-idea`
3. `; tmux set -g mouse on` — enable mouse support (redundant with `.tmux.conf` but safe to repeat)

**`engine/.vscode/settings.json`** does the same but for the `claude-engine` session — each project gets its own named session.

---

## The Big Picture — End to End

```
You SSH in from your laptop
        │
        ▼
.bashrc runs
        │
        ├── Are we in SSH? ─── No → normal bash, no tmux
        │
        └── Yes ──► Build session name from $PWD
                          │
                          ├── Session exists? → attach to it
                          │
                          └── Doesn't exist? → create it, run `claude`, then attach
                                                    │
                                                    ▼
                                         You land inside tmux session
                                         with claude already running
                                         Session survives if you disconnect


You open VS Code terminal (in idea-proposal project)
        │
        ▼
settings.json tells VS Code to use the "tmux" profile
        │
        ▼
VS Code runs: bash -c "tmux attach -t claude-idea || tmux new -s claude-idea"
        │
        ├── Session running (from SSH)? → attach → you see the SAME session SSH sees
        │
        └── No session? → create one fresh
```

---

## Why This Setup Exists

The key insight is that **the SSH session and the VS Code terminal share the same tmux session**. If you:
- SSH in → you're in `claude-idea`
- Open VS Code terminal → you attach to the same `claude-idea`
- Both see the same process, same output

If your SSH drops, tmux keeps running on the Pi. When you reconnect (SSH or VS Code), you re-attach and see everything exactly as you left it — no lost work, no interrupted `claude` sessions.
