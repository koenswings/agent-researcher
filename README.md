# IDEA Proposal

Planning and proposal documents for configuring OpenClaw to run the IDEA virtual company.

**IDEA** (Initiative for Digital Education in Africa) is a charity deploying Raspberry Pi-based offline school computers into rural African schools.

## What this project is

This is the pre-approval workspace. Documents here describe how the IDEA virtual company will be structured and operated through OpenClaw — the agent roster, HQ directory layout, CEO approval workflows, and component backlogs.

Once the CEO approves a proposal, it is implemented and its documentation migrates to the appropriate operational project (primarily `/home/pi/openclaw`).

## Key documents

- **[openclaw-brainstorm.md](openclaw-brainstorm.md)**: Main planning document — agent roster, HQ structure, daily workflows, decisions log, and current backlog.

## Claude session

This project uses the `claude-idea` tmux session. Opening this folder in VS Code via Remote-SSH automatically attaches to (or creates) that session, with project-specific Claude memory isolated from other projects.

See the engine project's README for the full explanation of the per-project session convention.
