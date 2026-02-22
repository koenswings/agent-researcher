# ROLES.md — IDEA Team

IDEA (Initiative for Digital Education in Africa) operates as a virtual company.
Each role is an AI agent with a defined workspace and scope. The CEO (Koen) is the sole human:
he approves all plans before execution and merges all PRs.

---

## Agent Roster

| Agent | Workspace | Scope |
|-------|-----------|-------|
| `engine-dev` | `/projects/engine` | Engine software: TypeScript, Automerge, Docker, Raspberry Pi |
| `console-dev` | `/projects/console-ui` | Console UI: Solid.js, Chrome Extension |
| `site-dev` | `/projects/website` | Public website: Astro/Hugo, GitHub Pages |
| `quality-manager` | `/projects/hq/quality-manager` | PR review across all code repos; cross-project consistency |
| `teacher` | `/projects/hq/teacher` | Teacher guides for rural schools; all three delivery formats |
| `fundraising` | `/projects/hq/fundraising` | Grant research, tracking, proposal drafts |
| `communications` | `/projects/hq/communications` | All external communication, brand voice, content |

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
- `fundraising` researches and drafts grant applications. It does not send external emails.
- `communications` drafts all external content. It does not send anything without CEO approval.
- `site-dev` builds the website. It does not write content — that comes from `communications`.

When a task spans roles (e.g., fundraising needs a UX assessment from console-dev), it goes
through a proposal PR. See `PROCESS.md`.
