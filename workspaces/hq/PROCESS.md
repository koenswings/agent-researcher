# PROCESS.md — How We Work

## The Backlog

`BACKLOG.md` is the single source of truth for all approved work. Items here have been reviewed
and approved by the CEO. Only work on unapproved items when explicitly instructed in a session.

---

## Growing the Backlog

Anyone on the team — or the CEO — can propose a new item. Here is the process:

### Step 1: Propose

Create a file in `hq/proposals/` with the naming convention:

```
YYYY-MM-DD-<topic>.md
```

Open a PR to the `hq` repo with your proposal. A good proposal answers:

- **What**: What is being proposed?
- **Why**: What problem does it solve or opportunity does it create?
- **Impact**: Which areas of the project are affected?
- **Dependencies**: Does this depend on other work already in the backlog?
- **Open questions**: What do you need input on from other roles?

### Step 2: Cross-team input

Tag the relevant agents in the PR description. Examples:

- A fundraising observation about system usability → tag `console-dev` for technical feasibility,
  `quality-manager` for consistency review
- A teacher guide gap identified during school feedback → tag `engine-dev` if a feature is needed,
  `communications` if public messaging is involved
- A website content need → tag `communications` (to write) and `site-dev` (to implement)

Agents review the PR and contribute by:
- Commenting with technical context, estimates, or concerns
- Drafting a sub-proposal if the idea needs to be broken down first
- Flagging dependencies on existing backlog items

The **Quality Manager** reviews all proposals for cross-project consistency and flags
architectural concerns before the CEO sees it.

### Step 3: CEO approval

The CEO reviews the enriched proposal and either:

- **Merges the PR** → the item moves to `BACKLOG.md` in the appropriate section
- **Requests changes** → PR comments; the proposing agent revises
- **Declines** → closes the PR with a comment explaining why

### Step 4: Backlog entry

On merge, the proposing agent (or CEO) adds a concise entry to `BACKLOG.md` in the relevant
section, including:
- A short title
- A link to the merged proposal for full context
- Which role should pick it up
- Priority or sequencing notes if relevant

---

## Worked Example

> **Scenario**: The fundraising agent notices that grant applications often ask for student usage
> data, but the Engine currently records none.

1. Fundraising creates `proposals/2026-03-01-usage-analytics.md`:
   - What: Add per-student usage tracking to the Engine
   - Why: Required evidence for EU development fund applications
   - Impact: Engine (data model), Console (display), Teacher guides (privacy notice for schools)
   - Open questions: What data is appropriate to collect? How is it stored offline?

2. PR opened on hq repo. Tagged: `engine-dev`, `console-dev`, `quality-manager`.

3. `engine-dev` comments: "We can store usage events in the Automerge doc. Syncs across devices
   when on the school network. Estimated effort: ~2 weeks."

4. `quality-manager` comments: "Privacy implications need a design doc before implementation.
   Recommend a minimal data approach — session counts only, no personal identifiers."

5. CEO reviews, merges with note: "Start with design doc. QM concern about privacy is right."

6. Backlog updated:
   ```
   - [ ] Usage analytics — design doc required first (QM: minimal data, no PII)
         [→ proposal](link) | owner: engine-dev
   ```

---

## The CEO Approval Chain

**For all work:**
- Plan mode is always on — every agent shows its plan before executing
- CEO approves or modifies before execution proceeds

**For code and document changes:**
- All changes land via PRs on feature branches
- No agent merges to `main` — CEO only

**For complex engine or console features:**
1. Design doc PR → CEO approves (merges to hq/design/)
2. Implementation PR → CEO reviews and merges

This is not bureaucracy. It is how we ensure that systems running in unattended schools with no
IT support are built carefully.
