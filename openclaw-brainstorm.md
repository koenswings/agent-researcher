# IDEA Virtual Company — Brainstorm

*Brainstorming session — no configuration changes made yet.*

**OpenClaw** is the self-hosted AI assistant platform already running on this Pi at `/home/pi/openclaw`. It manages multiple AI agents, each tied to a workspace (a codebase directory), and is accessible from any device on the Tailscale network at `https://openclaw-pi.tail2d60.ts.net`.

**IDEA** (Initiative for Digital Education in Africa) is the charity this virtual company serves. It deploys Raspberry Pi-based offline school computers running Engine and Console into rural African schools.

The goal is to configure OpenClaw to run the IDEA virtual company: multiple agents, each playing a specific role, coordinating through shared files and a CEO approval loop.

---

## How OpenClaw Already Maps to the Virtual Company

OpenClaw's agent model is a direct fit:

| OpenClaw concept | Virtual company concept |
|------------------|-------------------------|
| Agent (`id` in openclaw.json) | A team member / role |
| `workspace` | The codebase or work area that role owns |
| `AGENTS.md` in the workspace | The role definition — instructions, responsibilities, constraints |
| `DEFAULT_PERMISSION_MODE=plan` (already set in compose.yaml) | CEO approval loop — agents always show their plan before acting |
| Browser UI at tailscale URL | The "office" — you open an agent's tab to interact with that role |

**The plan permission mode is the key insight.** It is already set. Agents never act unilaterally — they always propose what they intend to do and wait for approval. This IS the CEO approval mechanism, built in.

For code changes specifically, the additional layer is **GitHub branch protection**: agents open PRs on feature branches, and only you can merge to `main`. This prevents any code from landing without explicit review.

---

## The Agent Roster

Each IDEA role becomes one agent entry in `openclaw.json`, with its own workspace and `AGENTS.md`.

**Full agent roster:**

| Agent id | Workspace (host path) | Role |
|----------|----------------------|------|
| `engine-dev` | `/home/pi/projects/engine` | Engine software developer |
| `console-dev` | `/home/pi/projects/console-ui` | Console UI developer (Solid.js, Chrome Extension) |
| `site-dev` | `/home/pi/projects/website` | Builds and maintains the IDEA public website |
| `quality-manager` | `/home/pi/projects/hq/quality-manager` | Cross-project quality and consistency reviewer |
| `teacher` | `/home/pi/projects/hq/teacher` | Creates offline teacher guides for rural schools |
| `fundraising` | `/home/pi/projects/hq/fundraising` | Researches grants, tracks donors, drafts proposals |
| `communications` | `/home/pi/projects/hq/communications` | All external communication and public presence |

*Note: existing `engine` and `console-ui` entries in openclaw.json must be renamed to `engine-dev` and `console-dev`.*

The `quality-manager`, `teacher`, `fundraising`, and `communications` agents are given subdirectories inside `hq/` as their workspace. This gives each its own `AGENTS.md` while keeping them inside the shared HQ directory tree. Because `/home/pi/projects` is fully mounted into the container as `/home/node/workspace`, every agent can read any other project if its AGENTS.md instructs it to — the QM in particular needs this to review code across all repos.

---

## The HQ Directory

`/home/pi/projects/hq/` is the company's coordination hub. It is a git repo in its own right. Structure:

```
hq/
  BACKLOG.md                      ← master backlog, covers all projects
  ROLES.md                        ← agent roster with links to all AGENTS.md files (explains split)
  CONTEXT.md                      ← shared knowledge: mission, solution, key concepts (read by all agents)
  PROCESS.md                      ← how ideas become backlog items
  standups/
    2026-02-20-morning.md
  discussions/                    ← multi-agent dialogue threads (open until CEO closes)
  design/                         ← RFC-style design docs for complex features
  proposals/                      ← new ideas awaiting CEO approval
    YYYY-MM-DD-<topic>.md
  quality-manager/
    AGENTS.md                     ← quality manager role definition
  teacher/
    AGENTS.md                     ← teacher role definition
    getting-started.md
    kolibri-guide.md
    nextcloud-guide.md
  fundraising/
    AGENTS.md                     ← fundraising role definition
    opportunities.md
    grant-tracker.md
    proposals/
  communications/
    AGENTS.md                     ← communications manager role definition
    brand/
      tone-of-voice.md            ← how IDEA writes and speaks
      key-messages.md             ← what IDEA is, why it matters, our asks
    website/                      ← content drafts for site-dev
    donors/                       ← newsletter and impact report templates
    grants/narratives/            ← grant application narrative sections
    partners/                     ← partner outreach templates
    press/                        ← press releases and media kit
```

The `BACKLOG.md` is the single source of truth for what needs to be done across all projects. Any agent can propose additions; only the CEO approves them.

---

## AGENTS.md — The Role Definition File

Each workspace has an `AGENTS.md` that shapes the agent's behaviour. For example:

**`/home/pi/projects/engine/AGENTS.md`** (Engine Developer):
- You are the Engine software developer for IDEA
- Tech stack: TypeScript, Node.js, Automerge, Docker, zx
- Your work lives on feature branches; open a PR for every change
- Every PR must include: code changes, tests, and updated documentation
- For complex features, propose a design doc in `hq/design/` first
- The engine runs unattended in rural schools with no IT support — reliability is paramount
- Consult `hq/BACKLOG.md` for approved work items

**`/home/pi/projects/hq/quality-manager/AGENTS.md`** (Quality Manager):
- You are the Quality Manager for IDEA
- Review open PRs across `/home/node/workspace/engine` and `/home/node/workspace/console-ui`
- Check: tests present, docs updated, change consistent with architecture, offline resilience preserved
- Raise concerns in PR comments; never approve or merge — that is the CEO's role
- Read the latest standup from `../standups/` before each review session

**`/home/pi/projects/hq/fundraising/AGENTS.md`** (Fundraising Manager):
- You are the Fundraising Manager for IDEA, a charity deploying offline school computers in rural Africa
- Research and track grant opportunities: EU development funds, UNESCO, UNICEF, Gates Foundation, Raspberry Pi Foundation, national development agencies
- All outputs are documents for CEO review — never make external contact autonomously
- Maintain `opportunities.md` and `grant-tracker.md`
- Draft proposals in `proposals/` as PRs for CEO approval before any submission
- Hand narrative writing to the Communications Manager with a clear brief

**`/home/pi/projects/hq/communications/AGENTS.md`** (Communications Manager):
- You are the Communications Manager for IDEA, a charity deploying offline school computers in rural Africa
- Define and maintain IDEA's brand voice (`brand/tone-of-voice.md`, `brand/key-messages.md`)
- Draft all external-facing content: website, donor newsletters, grant narratives, partner outreach, press
- All outputs are drafts for CEO review — never send, post, or publish anything externally
- Website content goes in `website/content-drafts/` as PRs for `site-dev` to implement
- Grant narratives are written from briefs provided by the Fundraising Manager in `hq/fundraising/proposals/`
- The Quality Manager reviews your drafts for factual consistency with project documents

**`/home/pi/projects/hq/teacher/AGENTS.md`** (Teacher):
- You are the Teacher documentation specialist for IDEA
- Audience: teachers in rural African schools, limited technology experience, no internet
- Guides must work fully offline — they will be served from the Engine or printed
- Cover the deployed apps: Kolibri, Nextcloud, offline Wikipedia
- Keep language simple and concrete. Use screenshots or diagrams where possible.
- Flag any guide that needs review by an actual teacher before deployment

---

## CEO Approval — Two Layers

**Layer 1 — Plan mode (already active):** Every agent shows its plan before acting. You approve or modify before it executes. This applies to all work.

**Layer 2 — GitHub PRs (to be set up):** All code and document changes land on feature branches. The agent opens a PR. You review on GitHub and merge (or request changes). Branch protection on `main` in every repo enforces this mechanically.

For complex engine or console changes, a third gate applies:
1. Agent proposes a design doc in `hq/design/` → you approve via PR merge
2. Implementation PR is raised only after the design is merged

---

## CEO Tools & Daily Workflow

### Tool Stack

| Tool | Purpose | When you use it |
|------|---------|-----------------|
| **OpenClaw Web UI** | Direct agents, approve plans, run standups | Daily — primary interface |
| **GitHub** | Review and merge PRs (code and documents) | Whenever agents raise PRs |
| **Terminal (SSH / Tailscale SSH)** | Pi administration, Docker, logs | Occasional — infrastructure changes only |

### Using the OpenClaw Web UI

Access at `https://openclaw-pi.tail2d60.ts.net` from any device on the Tailscale network.

The UI presents each agent as a separate chat tab. Your workflow in each tab:

| Step | What you do | What happens |
|------|-------------|--------------|
| **Direct** | Open an agent's tab, type a task | e.g. "Pick the next backlog item and propose your approach" |
| **Review plan** | Agent proposes exactly what it will do before acting | This is `DEFAULT_PERMISSION_MODE=plan` — the agent always stops here |
| **Approve / redirect** | Type "go ahead" or modify the plan | Agent executes only after your explicit approval |
| **Observe** | Watch tool calls, file edits, git operations stream in real time | You can interrupt at any point |

The "A return + waiting emoticon" in the chat is the agent presenting its plan and waiting for your go/no-go. It is not a bug — it is the CEO approval loop working as intended.

**Which agent tab to use for what:**

| Agent tab | Use it to... |
|-----------|-------------|
| `engine-dev` | Assign Engine coding tasks, review technical proposals |
| `console-dev` | Assign Console UI tasks |
| `site-dev` | Assign website content and build tasks |
| `quality-manager` | Request a cross-project review or PR analysis |
| `fundraising` | "What grants should we be applying for right now?" |
| `communications` | "Draft a donor update for this month" |
| `teacher` | "Write a Kolibri getting-started guide for teachers" |

### A Typical CEO Day

```
Morning
  └─ OpenClaw: standup agent → "run morning standup" → approve plan → read result
  └─ GitHub: review any PRs raised overnight → merge or comment

During the day
  └─ OpenClaw: fundraising → "what grants close this quarter?" → approve → review output
  └─ OpenClaw: engine-dev → "pick the next backlog item" → approve → it opens a PR
  └─ GitHub: quality-manager has commented on a PR → read, decide, merge

As needed
  └─ OpenClaw: communications → "draft a donor update from this month's commits" → approve
  └─ GitHub: review the comms draft PR, edit inline, merge when satisfied
```

**Key mental model:** OpenClaw is where you direct work and approve plans. GitHub is where you review and accept finished work.

---

## Complementary Open Source Tools

OpenClaw's web UI is the primary dashboard — there are no third-party dashboards built specifically for it. These tools add useful capability around it:

### Portainer — Docker management UI
The most immediately useful addition. Gives a web UI to see all running containers, their health, logs, and resource usage — without needing SSH.

- Runs as a Docker container alongside OpenClaw
- Useful for monitoring the OpenClaw container itself and checking logs

### Plane — project management (open source Jira/Linear alternative)
If `hq/BACKLOG.md` as a markdown file starts to feel limiting, Plane provides a proper board and backlog UI with issues, cycles, and modules. Self-hostable on the Pi.

### n8n — workflow automation
Useful for automating the standup trigger, scheduled agent prompts, or GitHub notifications. Rather than running `./standup morning` manually, n8n can trigger it on a schedule.

### Grafana — monitoring dashboards
Visibility into Pi health (CPU, memory, temperature, disk usage) — relevant for an always-on device deployed in a rural school. Pairs with Prometheus for metrics collection.

---

## Mission Control — An Alternative Operating Layer

[openclaw-mission-control](https://github.com/abhi1693/openclaw-mission-control) is a purpose-built
dashboard for running OpenClaw at team scale. It adds a Kanban board, structured task dispatch,
real-time agent monitoring, and built-in approval flows on top of the OpenClaw gateway. It runs as
a Next.js application (port 4000), connects to the OpenClaw gateway via WebSocket (port 18789),
persists state in SQLite, and deploys as a Docker container. The project is under active development
— production-grade architecture, but expect occasional breaking changes.

### What Changes in the Setup

**Additions:**
- Mission Control runs as an additional Docker container alongside OpenClaw, added to `compose.yaml`
- A bearer token (`LOCAL_AUTH_TOKEN`, minimum 50 characters) links it to the OpenClaw gateway
- A one-time board hierarchy is configured in the MC UI: **IDEA org → Board Groups (Engineering, HQ)
  → Boards per agent or project → Tasks** (the backlog items)
- A second Tailscale-accessible URL for the MC UI (e.g. `https://openclaw-pi.tail2d60.ts.net:4000`)

**Unchanged:**
- `openclaw.json` — agent config, workspaces, plan mode, and sandbox settings are untouched
- `AGENTS.md` files — role definitions are unaffected
- Sandbox files — IDENTITY, SOUL, USER, TOOLS, HEARTBEAT, BOOTSTRAP are unaffected
- The HQ directory structure on disk

**Retired:**
- `hq/BACKLOG.md` as the live task view — the MC Kanban board replaces it. BACKLOG.md could be
  kept as a git-native audit log or retired entirely once MC is the source of truth.

### What Changes in Daily Operations

**Task dispatch moves to the Kanban board.** Instead of opening an agent's chat tab and typing
"pick the next backlog item", you create a task in MC and assign it to the relevant agent. The board
columns — Planning → Inbox → Assigned → In Progress → Review → Done — give a single-screen view of
all 7 agents' work simultaneously.

**Plan approval still happens in individual agent tabs.** Mission Control dispatches work, but
`DEFAULT_PERMISSION_MODE=plan` is unchanged. After assigning a task, you go to that agent's chat
tab to read and approve its plan before anything executes. MC does not replace this layer.

**The activity timeline partially replaces the standup script.** MC provides a real-time SSE-fed
activity log across all agents. The morning standup becomes a scan of that timeline rather than
triggering a script to generate `hq/standups/YYYY-MM-DD-morning.md`. The standup script can still
run if a written record is preferred.

**The proposal/PR flow is unchanged.** MC tasks are the implementation-level view. The GitHub
PR-based proposal and review process remains the approval layer for finished work.

### Revised CEO Tool Stack

| Tool | Purpose | When you use it |
|------|---------|-----------------|
| **Mission Control** | Live Kanban across all 7 agents; task creation and assignment; activity timeline | Daily — primary dispatch surface |
| **Individual agent tabs (OpenClaw UI)** | Approve plans, have in-depth sessions, run ad-hoc tasks | After each task assignment, and for complex conversations |
| **GitHub** | Review and merge PRs (code and documents) | Whenever agents raise PRs |
| **Terminal (SSH / Tailscale SSH)** | Pi administration, Docker, logs | Occasional |

### Revised Typical CEO Day

```
Morning
  └─ Mission Control: scan activity timeline → see what ran overnight
  └─ GitHub: review any PRs raised overnight → merge or comment

During the day
  └─ Mission Control: create task for fundraising → assign → open agent tab → approve plan
  └─ Mission Control: move engine-dev task to "In Progress" → open tab → review progress
  └─ GitHub: quality-manager has commented on a PR → read, decide, merge

As needed
  └─ Mission Control: assign comms task ("donor update") → open tab → approve plan
  └─ GitHub: review the comms draft PR, edit inline, merge when satisfied
```

**Key mental model with MC:** Mission Control is where you see the full picture and assign work.
Agent tabs are where you have the conversation and approve plans. GitHub is where you accept
finished work.

### Fit for IDEA

**Pros:**
- Single-screen visibility across all 7 agents — no tab-switching just to see what's in progress
- Built-in approval flows and RBAC formalise the CEO role in the UI itself
- REST API makes standup automation and task state queries straightforward without parsing markdown
- Directly replaces the need for Plane (from the Complementary Tools list above) and reduces the
  need for Portainer
- Purpose-built for OpenClaw — won't drift out of sync as the gateway evolves

**Cons:**
- Additional process on a constrained Raspberry Pi (memory, CPU — worth benchmarking)
- SQLite state lives outside git — less auditable than commits to `BACKLOG.md`
- Under active development — breaking changes possible between updates
- One more URL and port to route through Tailscale
- One-time cost to migrate the existing backlog items into the MC board hierarchy

**Verdict:** Mission Control is a clear upgrade to the operating experience once the base 7-agent
setup is stable. It is not a prerequisite for launch — the base setup (plan mode + GitHub PRs +
`BACKLOG.md`) works without it. The recommended path is to launch with the base setup, run it for
a few weeks, then add Mission Control when the workflow is understood and the Pi's headroom is
confirmed.

---

## Multi-Agent Dialogue — Standups and Discussion Threads

### The vision

A standup is not a summary. It is a forum where each agent shares what they are working on,
what new insights they have developed, and what questions or concerns they have for the team.
Other agents complement, critique, or build on what was said. New ideas are explored through
dialogue between stakeholders. The belief is that this kind of multi-stakeholder conversation
drives innovation and quality that no individual agent — or any single summary — can produce.

### The architectural reality

OpenClaw agents run in separate Claude Code sessions. They cannot talk to each other directly
or simultaneously. The only shared medium is the filesystem (all repos mounted into the same
container directory at `/home/node/workspace/`).

"Dialogue" is approximated through three mechanisms:
1. **Sequential contribution to a shared document** — each agent reads what came before, then
   adds their voice, so later agents naturally respond to earlier ones
2. **Persistent discussion threads** — topic-based files in `hq/discussions/` that any agent
   can contribute to over time
3. **@-mention convention** — agents signal when they need a specific other agent's input,
   giving the CEO a clear guide for which tab to open next

The CEO acts as **facilitator**, not just audience. They decide which agents participate, in
what order, and when a dialogue has reached a useful conclusion.

---

### Mechanism 1: The Roundtable Standup

A structured shared document where agents contribute sequentially. Each agent reads the whole
file before writing — so later agents naturally respond to earlier ones.

#### Document format

```markdown
# Standup — YYYY-MM-DD

## Opening (CEO)
Focus for today, current priorities, anything agents should keep in mind.

---

## engine-dev (Axle ⚙️)

### Working on
What I am currently doing and why it matters.

### New insight — may affect others
Something I discovered that others should know. (Leave blank if nothing.)

### Question for the team
A specific question or concern directed at another agent or the group.
Use @agent-id to be explicit.

### Response to [agent name]
(Only if a previous agent raised something relevant to my domain.)

---

## console-dev (Pixel 🖥️)
[same four sections]

...

## CEO close
Decisions made. Actions added to backlog. Discussion threads opened.
```

#### Flow

1. CEO (or the standup script) creates the file with the opening section and today's context
   (recent commits, open PRs, backlog priorities)
2. CEO opens Agent 1's tab: *"Add your voice to today's standup at
   `hq/standups/YYYY-MM-DD.md` — read the whole file first, then contribute"*
3. Agent reads the whole file, fills in their four sections, commits
4. CEO opens Agent 2's tab — same instruction; Agent 2 reads Agent 1's contribution and may
   respond directly to it
5. Repeat for all participating agents
6. CEO reads the completed document and adds the closing section: decisions, new backlog
   entries, discussion threads to open
7. Any agent whose @-mention needs a follow-up response gets another pass

#### Depth is adjustable

Not every standup needs all seven agents. The CEO decides the scope:

| Mode | Who participates | When |
|------|-----------------|------|
| **Daily light** | Agents with active backlog items (2–4) | Most days |
| **Full roundtable** | All 7 | After a milestone; when something cross-cutting happened |
| **Issue-triggered** | Relevant 2–3 agents | When any agent flags a cross-cutting concern |

#### Why this creates genuine dialogue

- Each agent explicitly reads what came before writing
- The template prompts agents to share *insights* (not just status) and ask *questions*
- Later agents respond directly to earlier ones — emergent cross-pollination
- The CEO can re-open any agent tab mid-sequence to follow up on an @-mention
- The completed document is committed to `hq/standups/` — the dialogue is permanent and
  searchable

---

### Mechanism 2: Discussion Threads — hq/discussions/

Standups surface ideas and concerns. Some need more depth than a standup can give. Discussion
threads provide a persistent, topic-based forum for this deeper exploration.

**How a thread works:**

1. Any agent (or CEO) opens: `hq/discussions/YYYY-MM-DD-<topic>.md`
2. The opening section states the question or proposal clearly, plus the opener's initial view
3. Tagged agents (`@agent-id`) add their perspective when the CEO opens their tab and points
   them at the thread
4. The thread stays open across sessions and days until the CEO closes it
5. The CEO closes a thread with a bottom section: *Resolved — [decision / added to backlog /
   no-action — reason]*
6. Closed threads become institutional memory: they explain *why* decisions were made, not
   just what was decided

**Examples:**

- **Teacher** discovers guides assume USB ports are labelled: *"Is this true for our hardware?"*
  → tags `@engine-dev` → Axle clarifies → Teacher updates guides, thread closed
- **Fundraising** finds a grant requiring offline impact metrics → tags `@engine-dev`,
  `@console-dev` → dialogue about what is feasible to measure → thread feeds into a backlog
  proposal
- **Engine-dev** hits an Automerge edge case → tags `@quality-manager`, `@teacher` → QM
  documents as a known limitation, Teacher adds a troubleshooting note → thread closed
- **CEO** wants to explore whether a new app category is worth supporting → opens thread,
  tags all relevant agents → full multi-stakeholder exploration before any backlog commitment

---

### Mechanism 3: @-mention convention

In any shared document — standup, discussion thread, proposal — agents use `@agent-id` to
signal that a specific agent's input is needed:

```
@engine-dev — does the disk detection code handle this edge case already?
@quality-manager — is this a pattern we should flag in the review checklist?
```

The CEO reads @-mentions as a guide for which tab to open next. The standup script will scan
the day's standup file after each agent's contribution and print a suggested follow-up list.

---

### Options considered

**Option A — Current proposal (one agent generates a summary):** A script reads context and
one agent produces a structured status document.
- ✅ Fast; zero orchestration overhead
- ❌ Not dialogue; one voice; no cross-pollination; ideas don't improve through challenge

**Option B — Sequential roundtable standup (selected):** Shared document, agents contribute
in sequence, each reads and responds to previous contributions.
- ✅ Genuine multi-voice dialogue within OpenClaw's architectural constraints
- ✅ CEO facilitates rather than relays — agents read each other directly via the file
- ✅ Depth adjustable; git-preserved; no new tooling required
- ❌ Takes longer than a single-agent summary (multiple tabs)
- ❌ Introduces some asymmetry: later agents see more than earlier ones

**Option C — Multi-round dialogue (two explicit passes):** Everyone writes updates first, then
everyone does a response pass after reading all updates.
- ✅ More symmetric — everyone reads everyone before responding
- ❌ Doubles agent sessions per standup; expensive in time and CEO effort

**Option D — Async discussion threads only, no standup:** Replace the standup with
topic-based threads. Agents contribute when relevant, not on a schedule.
- ✅ Organic; discussions happen when needed
- ❌ No cadence; easy to drift; CEO loses the daily pulse

**Option E — Automated orchestration via subagents:** A standup orchestrator agent uses the
Task tool to spawn each other agent as a subagent, collecting contributions automatically.
- ✅ Zero CEO tab-switching
- ❌ Subagents lack each agent's real memory and sandbox state — it simulates the agents
  rather than engaging the genuine ones; bypasses the CEO's facilitator role

### Decision ✅

**Option B as the standup format, with Option D's discussion threads as a complementary
mechanism.** Together they provide daily rhythm (roundtable) and deep-topic forums
(discussions). The @-mention convention connects them.

---

## File System Structure — Agent Config Location

### The observed split

Developer agents (`engine-dev`, `console-dev`, `site-dev`) have their `AGENTS.md` inside their own
code repositories. Coordination agents (`quality-manager`, `teacher`, `fundraising`,
`communications`) have theirs under `hq/`. This can feel inconsistent at first glance.

### Why it is actually intentional

**Developer agents are tied to a codebase.** Co-locating `AGENTS.md` in the repo is the standard
Claude Code convention — the file is discovered automatically from the project root and tells the
agent how to work in that specific repo. The config belongs alongside the code it governs, just like
`package.json` or a `Makefile`.

**Coordination agents produce documents, not code.** They have no code repository of their own.
The `hq/` repo is the only repo they work in, so their `AGENTS.md` naturally lives there as a
subdirectory.

### Options considered

**Option A — Co-location (current as-is):** `AGENTS.md` in each agent's primary workspace;
coordination agents in `hq/` subdirs.
- ✅ Natural home; standard Claude Code convention for developer agents
- ❌ Three separate repos to scan when looking for all agent configs; non-obvious to new contributors

**Option B — Centralise all in HQ:** Move all `AGENTS.md` to `hq/engine-dev/`, `hq/console-dev/`,
`hq/site-dev/` alongside the coordination agent subdirs.
- ✅ Single repo to find all agent definitions
- ❌ Breaks co-location convention; developer agents must look in a different repo for their own instructions; code repos become "bare" with no configuration

**Option C — Hybrid: co-location + explicit HQ index (selected):** Keep `AGENTS.md` in each
workspace (code repos for developers, HQ subdirs for coordinators). Improve `hq/ROLES.md` to
explicitly document where each agent's config lives and why, with links — making it the single
navigation point even though configs are distributed.
- ✅ No duplication; no broken convention; developer agents find instructions in their own repo
- ✅ `ROLES.md` already serves as the team roster — it just needs the explanation and links

**Option D — One dedicated repo per agent + one shared company repo:** Every agent gets their own
git repository. Coordination agents (`quality-manager`, `teacher`, `fundraising`, `communications`)
move from `hq/` subdirs into dedicated standalone repos. The `hq/` repo is stripped to purely
shared coordination content (CONTEXT.md, ROLES.md, BACKLOG.md, PROCESS.md, standups/, design/,
proposals/) — no agent workspace content at all.

Repo count: 7 agent repos + 1 company repo = **8 repos total** (vs 4 in Options A/C).

```
idea-edu-africa/
  engine           ← engine-dev workspace (unchanged)
  console-ui       ← console-dev workspace (unchanged)
  website          ← site-dev workspace (unchanged)
  quality-manager  ← NEW: own repo (currently hq/quality-manager/)
  teacher          ← NEW: own repo (currently hq/teacher/)
  fundraising      ← NEW: own repo (currently hq/fundraising/)
  communications   ← NEW: own repo (currently hq/communications/)
  hq               ← company repo: CONTEXT.md, ROLES.md, BACKLOG.md,
                      PROCESS.md, standups/, design/, proposals/
```

Key consequence: `AGENTS.md` co-location now works **uniformly for all 7 agents** — every agent's
config is at the root of their own repo. The developer/coordinator split disappears entirely.

- ✅ Structural confusion disappears — every agent has the exact same setup (own repo + company repo)
- ✅ Standard Claude Code convention (`AGENTS.md` at repo root) applies to all agents without exceptions
- ✅ Clean git history per agent: teacher guide commits live only in the teacher repo
- ✅ Independent per-repo access control if needed (e.g. make teacher guides public separately)
- ✅ Company repo is purely coordination — no workspace noise
- ✅ Pattern scales cleanly: adding a new agent = create a new repo
- ❌ Four additional GitHub repos to create, configure, and maintain (branch protection ×4)
- ❌ Coordination agents' path to shared files changes: BACKLOG.md is now in the company repo
  (`/home/node/workspace/hq/BACKLOG.md`) rather than `../BACKLOG.md` — a minor AGENTS.md update,
  not a real obstacle since all repos are mounted in the same container directory
- ❌ One-time migration effort: move content out of `hq/` subdirs into new repos

### Decision ✅

**Currently: Option C.** The split is documented in `ROLES.md` with links and explanation.

**Note for review:** Option D directly addresses the original concern — the structural confusion —
rather than patching it with documentation. The cost is 4 additional repos and a one-time migration.
If conceptual uniformity matters more than minimising repo count, Option D is the stronger choice.
This decision is worth revisiting before the full implementation starts.

---

## Shared Agent Knowledge — CONTEXT.md

### The problem

Technical knowledge about App Disks, Engine, Console, offline-first operation, and data
synchronization currently lives primarily in `engine-dev/AGENTS.md`. But every agent needs this
understanding:

- `teacher` must explain app usage accurately to non-technical school staff
- `fundraising` must describe the solution credibly to grant reviewers
- `communications` must explain how the system works in public-facing writing
- `quality-manager` must catch when changes break architectural principles (e.g. offline-first)
- `site-dev` must present the product accurately on the website

`SOUL.md` (shared across all agents) covers values and behaviour but not factual product knowledge.
Individual `AGENTS.md` files describe roles, not the product. There is no shared knowledge base.

### Options considered

**Option A — Extend SOUL.md:** Add "What we build" and "How it works" sections to the existing
shared SOUL.md.
- ✅ Already loaded by all agents; zero new files
- ❌ Conflates values/behaviour with factual product knowledge; SOUL.md becomes harder to maintain
and update as the product evolves

**Option B — Per-agent CONTEXT.md in each sandbox:** Write a tailored `CONTEXT.md` per agent,
calibrated to each role's depth of technical knowledge.
- ✅ Can be tailored (engine-dev gets deep technical detail; fundraising gets high-level)
- ❌ Seven copies to maintain; any factual update requires seven edits; drift is inevitable

**Option C — Single `hq/CONTEXT.md` read by all agents (selected):** One file in the HQ repo.
All agents read it at the start of each session (referenced in HEARTBEAT.md). Covers:
1. **Mission** — Who IDEA serves, what problem it solves, why it matters
2. **Solution overview** — What the system does at a high level
3. **Key concepts** — Engine, Console, App Disks, offline-first, data synchronization, physical web
   app management
4. **Guiding principles** — Reliability > features, no internet dependency, teacher-friendly
- ✅ Single source of truth; one edit propagates to all agents via normal git/PR workflow
- ✅ Factual knowledge cleanly separated from values (SOUL.md) and role instructions (AGENTS.md)
- ✅ Maintained like any other HQ document — QM reviews for factual accuracy

**Option D — Embed context in each AGENTS.md:** Expand the "Context" section in every role
definition file. Some AGENTS.md files already partially do this.
- ✅ No new files
- ❌ Duplication across seven files; no guarantee coordination agents are updated when technical
  concepts evolve; becomes inconsistent over time

### Decision ✅

**Option C.** Add `hq/CONTEXT.md` as the shared knowledge base for all agents. Keep SOUL.md for
values/behaviour and AGENTS.md for role-specific instructions. All agents read CONTEXT.md at the
start of each session via HEARTBEAT.md.

---

## Decisions

1. **Rename `engine` → `engine-dev`?** ✅ Yes.
2. **Rename `console-ui` → `console-dev`?** ✅ Yes. Website agent is `site-dev`.
3. **Standup: automated or manual?** ✅ Manual trigger for now — `./standup morning`.
4. **HQ repo: public or private?** ✅ Public. All repos under the GitHub org will be public.
5. **GitHub Organisation name?** → **`idea-edu-africa`** (not yet created). Repos currently live under the personal account `koenswings` and will be transferred to the org when it is created. See availability table:

   | Name | Available? |
   |------|-----------|
   | `idea-africa` | ❌ Taken (existing "IDEA Africa" org) |
   | `idea-edu-africa` | ✅ Available |
   | `idea-offline` | ✅ Available |
   | `offline-schools` | ✅ Available |
   | `appdocker` | ❌ Taken |

6. **Teacher guides delivery?** ✅ All three: served from Engine, embedded in Console UI, printable PDFs.
7. **Website technology?** ✅ Static site (Astro or Hugo — TBC) hosted on GitHub Pages.

---

## Backlog Growth Process

Growing the backlog is a collaborative, PR-driven process. Full details in `hq/PROCESS.md`. Summary:

### The pipeline

1. **Anyone proposes** — any agent (or CEO) creates `hq/proposals/YYYY-MM-DD-<topic>.md` and opens a PR.

2. **Cross-team refinement** — relevant agents are tagged in the PR. Examples:
   - Fundraising identifies a need for usage analytics → tags `engine-dev` for feasibility,
     `console-dev` for UI impact, `quality-manager` for privacy review
   - Teacher spots a missing app feature → tags `engine-dev` to scope it
   - Communications needs new content → tags `site-dev` for implementation assessment

   Agents comment on the PR with technical context, estimates, concerns, or sub-proposals.
   The **Quality Manager** reviews all proposals for cross-project consistency.

3. **CEO decides** — merges (approved → backlog), requests changes, or closes (declined).

4. **Backlog entry** — on merge, a concise entry is added to `hq/BACKLOG.md` with a link to the proposal.

### Why this works

- Any team member can surface a need regardless of role
- Ideas get cross-functional input before the CEO sees them
- Nothing lands in the backlog without explicit approval
- The full proposal history is preserved in git

---

## Project Repositories

| Repo | Current URL | Notes |
|------|------------|-------|
| `engine` | https://github.com/koenswings/engine | Engine software |
| `openclaw` | https://github.com/koenswings/openclaw | OpenClaw platform config |
| `idea-proposal` | https://github.com/koenswings/idea-proposal | This planning doc and proposals |

All repos currently under personal account `koenswings`. Will transfer to `idea-edu-africa` org when it is created.

Repos still to create (after org setup): `console-ui`, `website`, `hq`.

---

## What Needs to Happen (in order)

1. ✅ Decide answers to the open questions above
2. ✅ Set up project repos: `engine`, `openclaw`, `idea-proposal` on GitHub
3. ✅ Set up VS Code / Claude Code / tmux per-project session pattern across all three projects
4. Review and approve the full proposal in `/home/pi/projects/idea-proposal/` (AGENTS.md files, sandbox files, openclaw.json)
5. Create GitHub Organisation (`idea-edu-africa`) and transfer existing repos; create new repos (`console-ui`, `website`, `hq`)
6. Create project directories on the Pi: `console-ui/`, `website/`, `hq/` with subdirs
7. Copy approved `AGENTS.md` files from proposal into each workspace
8. Apply updated `openclaw.json` (rename existing agents + add new ones)
9. Copy sandbox files (IDENTITY, SOUL, USER, TOOLS, HEARTBEAT, BOOTSTRAP) into each agent's OpenClaw sandbox
10. Restart OpenClaw: `sudo docker restart openclaw-gateway`
11. Set up branch protection on `main` in each GitHub repo (CEO-only merge)
12. Pair your browser with each new agent in the OpenClaw UI
13. Run the BOOTSTRAP session for each new agent to confirm identity and orientation

---

## Current Backlog

### HQ / Setup
- [x] Decide GitHub org name → `idea-edu-africa`
- [x] Set up `engine`, `openclaw`, `idea-proposal` repos on GitHub (currently under `koenswings`)
- [x] Set up VS Code / Claude Code / tmux per-project session pattern
- [x] Decide AGENTS.md file structure → co-location + HQ index (see File System Structure section)
- [x] Decide shared knowledge approach → single `hq/CONTEXT.md` (see Shared Agent Knowledge section)
- [x] Decide standup model → roundtable format + discussion threads (see Multi-Agent Dialogue section)
- [ ] Review and approve proposal in `/home/pi/projects/idea-proposal/`
- [ ] Create `hq/CONTEXT.md` — draft covering mission, solution overview, key concepts, guiding principles
- [ ] Update `hq/ROLES.md` to explain the AGENTS.md split with links to each agent's config
- [ ] Design standup template (`hq/standups/TEMPLATE.md`) and enhance `./standup` script: seed file with context, support @-mention scanning after each agent pass
- [ ] Create GitHub organisation (`idea-edu-africa`) and transfer repos; create `console-ui`, `website`, `hq`
- [ ] Create project directories on Pi: `console-ui/`, `website/`, `hq/` with subdirs
- [ ] Configure OpenClaw agents (rename engine→engine-dev, console-ui→console-dev; add 5 new agents)
- [ ] Copy sandbox files (IDENTITY, SOUL, USER, TOOLS, HEARTBEAT, BOOTSTRAP) to each agent
- [ ] Set up branch protection on `main` across all repos
- [ ] BOOTSTRAP sessions for all new agents

### Engine
- [ ] Review and improve Solution Description
- [ ] Update Architecture doc from Solution Description
- [ ] Remove Docker dev environment support from docs and code
- [ ] Test setup design review — automated tests, simulate disk dock/undock, multi-engine scenarios
- [ ] Refactor `script/` to `scripts/`
- [ ] Scan Solution Description for unimplemented features
- [ ] Review run architecture: which user? File ownership and permissions?

### Console UI
- [ ] Create repo and AGENTS.md
- [ ] Document architecture: Solid.js, Chrome Extension, Engine API contract
- [ ] First version of UI from Solution Description outline

### Website
- [x] Decide technology → static site on GitHub Pages
- [ ] Confirm framework: Astro or Hugo
- [ ] Create repo and AGENTS.md
- [ ] Set up GitHub Actions deploy to GitHub Pages
- [ ] First version: mission, how it works, how to support

### Teacher Guides
- [x] Decide delivery → all three (Engine-served, Console-embedded, printable PDF)
- [ ] Define delivery pipeline and PDF generation approach
- [ ] Getting Started guide
- [ ] App guides: Kolibri, Nextcloud, Wikipedia

### Fundraising
- [ ] Research applicable grant programmes
- [ ] Create grant tracking document
- [ ] Draft first funding opportunity brief

### Communications
- [ ] Define brand voice and key messages (`brand/tone-of-voice.md`, `brand/key-messages.md`)
- [ ] Draft website content: mission, how it works, how to support
- [ ] Create donor newsletter template
- [ ] Create impact report template
