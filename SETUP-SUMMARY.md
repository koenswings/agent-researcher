# IDEA Virtual Company — Setup Summary

This document summarises the full agent configuration prepared in this proposal project.
Everything here awaits CEO approval and implementation in `/home/pi/openclaw`.

---

## What Has Been Prepared

### 1. `openclaw.json`

A complete replacement for the OpenClaw agent configuration, defining 7 agents:

| Agent ID | Workspace (in container) | Role |
|----------|--------------------------|------|
| `engine-dev` | `/home/node/workspace/engine` | Engine software developer |
| `console-dev` | `/home/node/workspace/console-ui` | Console UI developer (Solid.js, Chrome Extension) |
| `site-dev` | `/home/node/workspace/website` | Public website (Astro/Hugo, GitHub Pages) |
| `quality-manager` | `/home/node/workspace/hq/quality-manager` | Cross-project PR reviewer |
| `teacher` | `/home/node/workspace/hq/teacher` | Teacher guides for rural schools |
| `fundraising` | `/home/node/workspace/hq/fundraising` | Grant research and proposal drafts |
| `communications` | `/home/node/workspace/hq/communications` | Brand voice, all external content |

All agents use `claude-sonnet-4-6` and per-agent sandbox isolation. The gateway token
placeholder must be filled from the existing `openclaw.json` before applying.

---

### 2. `workspaces/` — AGENTS.md files

Each workspace contains an `AGENTS.md` that defines the agent's role, tech stack, workflow,
and constraints. These files are copied into the workspace root of the corresponding project
directory on the Pi (e.g. `/home/pi/projects/engine/AGENTS.md`).

```
workspaces/
  engine/AGENTS.md
  console-ui/AGENTS.md
  website/AGENTS.md
  hq/
    ROLES.md          ← agent roster and role boundaries
    BACKLOG.md        ← approved work items (starting state)
    PROCESS.md        ← how proposals become backlog items
    quality-manager/AGENTS.md
    teacher/AGENTS.md
    fundraising/AGENTS.md
    communications/AGENTS.md
```

The `hq/` files go into `/home/pi/projects/hq/` — a new git repo to be created.

---

### 3. `sandboxes/` — Agent sandbox files

Each agent has 6 sandbox files, placed in its OpenClaw sandbox directory
(`/home/pi/openclaw/app-disk/instances/openclaw-dev/sandboxes/<agent-id>/`):

| File | Purpose |
|------|---------|
| `IDENTITY.md` | Agent's confirmed name, emoji, and vibe (filled during BOOTSTRAP) |
| `SOUL.md` | Shared values, mission framing, team rules, boundaries — identical across all agents |
| `USER.md` | Notes about the CEO: Koen, preferences, working style |
| `TOOLS.md` | Agent-specific tool notes and constraints |
| `HEARTBEAT.md` | Periodic checks the agent runs between sessions (PRs, tests, backlog, memory) |
| `BOOTSTRAP.md` | First-session script — self-deletes after setup is complete |

**Suggested agent identities** (confirmed during BOOTSTRAP sessions):

| Agent | Name | Emoji |
|-------|------|-------|
| `engine-dev` | Axle | ⚙️ |
| `console-dev` | Pixel | 🖥️ |
| `site-dev` | Beacon | 🌐 |
| `quality-manager` | Veri | 🔍 |
| `teacher` | Sage | 📚 |
| `fundraising` | Scout | 🔭 |
| `communications` | Herald | 📣 |

---

## CEO Approval Mechanism

Two layers are built in:

1. **Plan mode** (`DEFAULT_PERMISSION_MODE=plan`, already set in `compose.yaml`): every agent
   shows its plan before acting. The CEO types "go ahead" to proceed.

2. **GitHub PRs**: all code and document changes land on feature branches. Agents open PRs;
   the CEO reviews and merges. Branch protection on `main` enforces this mechanically.

---

## Implementation Steps (in order)

1. **Review** — read and approve the files in this proposal project
2. **Create GitHub org** — `idea-edu-africa`; transfer `engine`, `openclaw`, `idea-proposal`; create `console-ui`, `website`, `hq`
3. **Create Pi directories** — `console-ui/`, `website/`, `hq/` with subdirectories
4. **Copy AGENTS.md files** from `workspaces/` into each project directory
5. **Initialise hq repo** — copy `hq/ROLES.md`, `BACKLOG.md`, `PROCESS.md` into `/home/pi/projects/hq/`
6. **Apply openclaw.json** — copy to `/home/pi/openclaw/openclaw.json` (fill gateway token first)
7. **Copy sandbox files** — copy each agent's folder from `sandboxes/` into the OpenClaw sandbox directory
8. **Restart OpenClaw** — `sudo docker restart openclaw-gateway`
9. **Set branch protection** — CEO-only merge on `main` in every GitHub repo
10. **Run BOOTSTRAP sessions** — open each agent tab in the OpenClaw UI and run the first session

---

## Key Paths (on the Pi)

| Item | Path |
|------|------|
| OpenClaw config | `/home/pi/openclaw/openclaw.json` |
| OpenClaw sandbox dir | `/home/pi/openclaw/app-disk/instances/openclaw-dev/sandboxes/` |
| Projects root | `/home/pi/projects/` |
| HQ repo (to create) | `/home/pi/projects/hq/` |
| OpenClaw web UI | `https://openclaw-pi.tail2d60.ts.net` |
