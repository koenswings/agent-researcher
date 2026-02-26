# agent-researcher — Compass (Strategic Advisor)

**Agent id:** `researcher`
**Project name:** Compass
**tmux session:** `claude-agent-researcher`

## What This Is

This is the workspace for **Compass**, the strategic advisor to the IDEA CEO. Its purpose is to
help the CEO think through organisational structure, agent design, governance, and long-term
direction for the virtual company.

This is not an operational agent. It does not take tasks from Mission Control, open PRs on
operational repos, or participate in daily standups. It is consulted — the CEO comes here to
think, not to dispatch work.

## Key Documents

- **[openclaw-brainstorm.md](openclaw-brainstorm.md)** — Full configuration plan: agent roster,
  directory structure, workflows, architectural decisions, backlog.
- **[openclaw.json](openclaw.json)** — Proposed OpenClaw agent configuration (to be applied when
  implementing the virtual company).
- **[workspaces/](workspaces/)** — Staging area: approved AGENTS.md files and org root content,
  ready to be deployed to their final locations.
- **[sandboxes/](sandboxes/)** — Staging area: OpenClaw sandbox files (IDENTITY, SOUL, USER,
  TOOLS, HEARTBEAT, BOOTSTRAP) for each agent.

## Directory Structure

```
agent-researcher/
  CLAUDE.md                    ← agent instructions (role, scope, memory isolation rules)
  openclaw-brainstorm.md       ← main planning document
  openclaw.json                ← proposed openclaw configuration
  workspaces/
    idea/                      ← org root content (deploy to /home/pi/idea/)
      CONTEXT.md, ROLES.md, PROCESS.md, BACKLOG.md
    agents/
      agent-engine-dev/AGENTS.md
      agent-console-dev/AGENTS.md
      agent-site-dev/AGENTS.md
      agent-quality-manager/AGENTS.md
      agent-programme-manager/AGENTS.md
  sandboxes/
    engine-dev/
    console-dev/
    site-dev/
    quality-manager/
    programme-manager/
    researcher/
```

## Memory

This agent's Claude memory is private to the CEO. Operational agents do not read it and must
not be directed here. See `CLAUDE.md` for the full memory isolation rules.
