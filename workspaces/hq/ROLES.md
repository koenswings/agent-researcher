# ROLES.md — IDEA Team

IDEA (Initiative for Digital Education in Africa) operates as a virtual company.
Each role is an AI agent with a defined workspace and scope. The CEO (Koen) is the sole human:
he approves all plans before execution and merges all PRs.

---

## Agent Roster

| Agent | Workspace | AGENTS.md location | Scope |
|-------|-----------|-------------------|-------|
| `engine-dev` | `/projects/engine` | `/projects/engine/AGENTS.md` | Engine software: TypeScript, Automerge, Docker, Raspberry Pi |
| `console-dev` | `/projects/console-ui` | `/projects/console-ui/AGENTS.md` | Console UI: Solid.js, Chrome Extension |
| `site-dev` | `/projects/website` | `/projects/website/AGENTS.md` | Public website: Astro/Hugo, GitHub Pages |
| `quality-manager` | `/projects/hq/quality-manager` | `/projects/hq/quality-manager/AGENTS.md` | PR review across all code repos; cross-project consistency |
| `teacher` | `/projects/hq/teacher` | `/projects/hq/teacher/AGENTS.md` | Teacher guides for rural schools; all three delivery formats |
| `fundraising` | `/projects/hq/fundraising` | `/projects/hq/fundraising/AGENTS.md` | Grant research, tracking, proposal drafts |
| `communications` | `/projects/hq/communications` | `/projects/hq/communications/AGENTS.md` | All external communication, brand voice, content |

### Why config files are split across repos

**Developer agents** (`engine-dev`, `console-dev`, `site-dev`) have their `AGENTS.md` inside their
own code repository. This is the standard Claude Code convention — the file is discovered
automatically from the project root. Role instructions belong alongside the code they govern.

**Coordination agents** (`quality-manager`, `teacher`, `fundraising`, `communications`) produce
documents, not code. They have no code repository of their own, so their `AGENTS.md` lives in the
`hq/` repo — the only repository they work in.

This file (`ROLES.md`) is the single navigation point: wherever the config lives, you can always
find it from here.

---

## Shared Knowledge

All agents read `hq/CONTEXT.md` at the start of each session. That file covers:
- IDEA's mission and the problem it solves
- How the system works: Engine, Console, App Disks, offline-first, data synchronization
- Guiding principles shared across all roles

`SOUL.md` (in each agent's sandbox) covers values and behaviour.
`AGENTS.md` (per agent) covers role-specific instructions.
`CONTEXT.md` (this repo) covers factual product and mission knowledge.

---

## How the Team Works

**Plan mode is always on.** No agent acts without first showing its plan to the CEO.

**All changes go via PR.** No agent commits directly to `main` in any repo.

**The backlog is the source of truth.** Agents only work on approved items in `BACKLOG.md`,
unless given explicit in-session instruction.

**Proposals are how ideas grow.** Any agent can open a proposal PR in `hq/proposals/`.
See `PROCESS.md` for the full pipeline.

---

## Role Boundaries

- `engine-dev` and `console-dev` write code. They do not write external content.
- `quality-manager` reviews code and documents. It does not write features or content.
- `teacher` writes guides for school users. It does not write donor or press content.
- `fundraising` researches and drafts grant applications. It does not make external contact.
- `communications` drafts all external content. It does not send anything without CEO approval.
- `site-dev` builds the website. It does not write content — that comes from `communications`.

When a task spans roles (e.g., fundraising needs a UX assessment from console-dev), it goes
through a proposal PR. See `PROCESS.md`.
