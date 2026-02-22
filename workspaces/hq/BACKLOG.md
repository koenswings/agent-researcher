# BACKLOG.md — IDEA Master Backlog

All items here are approved by the CEO. Work on unapproved items only with explicit instruction.
To propose a new item, follow the process in `PROCESS.md`.

---

## HQ / Setup

- [ ] Create GitHub organisation (`idea-edu-africa`) and all repos
- [ ] Create project directories on Pi: `console-ui/`, `website/`, `hq/` with subdirs
- [ ] Write AGENTS.md files for each workspace
- [ ] Update openclaw.json (rename engine→engine-dev, console-ui→console-dev; add new agents)
- [ ] Restart OpenClaw after config change
- [ ] Set up branch protection on `main` in each GitHub repo (CEO-only merge)
- [ ] Pair browser with each new agent in the OpenClaw UI
- [ ] Run BOOTSTRAP session for each new agent

---

## Engine

- [ ] Review and improve Solution Description
- [ ] Update Architecture doc from Solution Description
- [ ] Remove Docker dev environment support from docs and code
- [ ] Test setup design — automated tests, simulate disk dock/undock, multi-engine scenarios
- [ ] Refactor `script/` to `scripts/`
- [ ] Scan Solution Description for unimplemented features
- [ ] Review run architecture: which user? File ownership and permissions?

---

## Console UI

- [ ] Create repo and initial AGENTS.md
- [ ] Document Console UI architecture: Solid.js, Chrome Extension, Engine API contract
- [ ] First version of UI from Solution Description outline

---

## Website

- [ ] Confirm framework: Astro or Hugo
- [ ] Create repo and initial AGENTS.md
- [ ] Set up GitHub Pages deployment via GitHub Actions
- [ ] First version: mission, how it works, how to support

---

## Teacher Guides

- [ ] Define delivery pipeline: Engine-served + Console-embedded + printable PDF
- [ ] Getting Started guide (hardware setup, first boot)
- [ ] App guides: Kolibri, Nextcloud, offline Wikipedia
- [ ] Privacy notice template for schools

---

## Fundraising

- [ ] Research applicable grant programmes (EU development, UNESCO, UNICEF, Gates, RPi Foundation)
- [ ] Create grant tracking document (`grant-tracker.md`)
- [ ] Draft first funding opportunity brief

---

## Communications

- [ ] Define brand voice and key messages (`brand/tone-of-voice.md`, `brand/key-messages.md`)
- [ ] Draft website content: mission page, how it works, how to support
- [ ] Create donor newsletter template
- [ ] Create impact report template
