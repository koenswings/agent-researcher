> **Question:** Document the operational flow of the virtual company — triggers, outputs, cross-agent involvement, automation. Compare the standup-driven model with a simpler CEO-driven model. Reframe heartbeats and standup. Include a diagram.

---

## The short version

The standup-driven model (Model A) puts the clock in charge. The CEO is reactive. Every morning five agents wake up and spend tokens whether or not the CEO is available or has anything to decide. The standup chain has complex timing dependencies and a single point of failure.

The CEO-driven model (Model B) puts you in charge. Nothing moves without you. Agents are reactive, not proactive. Cost is proportional to actual work. Complexity disappears because there is no coordination problem to solve — only you can initiate a cycle.

**Model B is the right model for this stage.** The recommendation is to adopt it now and treat Model A (standup) as an optional visibility tool you can trigger manually.

---

## Model B — CEO-driven operational flow

### Core principle

Every work cycle begins with a human decision and ends with a human decision. Agents act only in response to an approved trigger. The system is a decision amplifier, not an autonomous engine.

### The work cycle

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  CEO opens MC chat with Agent A                                             │
│  "Start task: [description]"                                                │
│         │                                                                   │
│         ▼                                                                   │
│  Agent A shows plan (plan mode = always on)                                 │
│         │                                                                   │
│         ├── CEO rejects → revised plan → loop                              │
│         │                                                                   │
│         └── CEO approves                                                    │
│                   │                                                         │
│                   ▼                                                         │
│          Agent A executes                                                   │
│          Produces: PR / design doc / proposal / report                      │
│                   │                                                         │
│                   └── At least once per task:                               │
│                       Agent A creates a review task for Agent B             │
│                       (via MC API during its session)                       │
│                               │                                             │
│                               ▼                                             │
│                    ◀── AUTO-TRIGGER (pi cron, no LLM) ──▶                  │
│                        Detects new task on Agent B's board                  │
│                        Wakes Agent B in isolated session                    │
│                               │                                             │
│                               ▼                                             │
│                    Agent B reviews Agent A's output                         │
│                    Produces: PR comment / annotation / flag                 │
│                    Marks review task done                                   │
│                               │                                             │
│                               ▼                                             │
│                    Agent A's context updated with B's response              │
│                    (written to shared file or PR comment)                   │
│                               │                                             │
│                               ▼                                             │
│          CEO reviews Agent A's complete output                              │
│          (including B's input)                                              │
│                   │                                                         │
│                   ├── Approve → task → Done in MC                          │
│                   ├── Amend   → Agent A revises → new review if needed     │
│                   └── Reject  → task → Cancelled with reason               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Mermaid diagram

```mermaid
flowchart TD
    CEO_START([CEO: start task]) --> PLAN[Agent A: show plan]
    PLAN -->|CEO rejects| PLAN
    PLAN -->|CEO approves| EXEC[Agent A: execute task]
    EXEC --> OUTPUT[Output produced\nPR / doc / proposal]
    OUTPUT --> REVIEW_TASK[Agent A creates\nreview task for Agent B\nvia MC API]
    REVIEW_TASK --> AUTO[pi cron detects new task\nnon-LLM, every 2 min]
    AUTO --> AGENT_B[Agent B isolated session\nreviews output]
    AGENT_B --> B_OUTPUT[Agent B writes response\nPR comment / annotation]
    B_OUTPUT --> CEO_REVIEW[CEO reviews\nAgent A output + B input]
    CEO_REVIEW -->|Approve| DONE([Task: Done in MC])
    CEO_REVIEW -->|Amend| EXEC
    CEO_REVIEW -->|Reject| CANCELLED([Task: Cancelled])

    style CEO_START fill:#2d6a4f,color:#fff
    style DONE fill:#2d6a4f,color:#fff
    style CANCELLED fill:#b5361a,color:#fff
    style AUTO fill:#457b9d,color:#fff
```

---

### Triggers and outputs — complete reference

| Trigger | Who starts it | Agent(s) involved | Output |
|---------|--------------|-------------------|--------|
| CEO message to agent | CEO | 1 primary + 1+ reviewers (auto) | PR, doc, proposal, report |
| CEO approves plan | CEO | Primary agent | Execution begins |
| CEO amends output | CEO | Primary agent (revises) | Updated PR / doc |
| CEO rejects output | CEO | Primary agent | Task cancelled, MC updated |
| Agent creates review task | Primary agent (automated) | Reviewer agent | PR comment, annotation, flag |
| **[auto-trigger]** new task on board | pi cron (automated) | Reviewer agent | Response written to shared output |
| CEO merges PR | CEO | None (git event) | Main branch updated |

---

### The auto-trigger mechanism

This is the one place where the system does something without the CEO. It is narrow, non-LLM, and reversible.

**How it works:**

```
pi cron (every 2 min, ~zero cost):
  for each agent board:
    check MC API for tasks in inbox created in last 3 min
    if found → send isolated trigger to gateway API
               prompt: "You have a new review task. Read it and respond."
               delivery: write response to task comment (no Telegram needed)
```

**Why this is safe:**
- Only fires for newly created tasks (3-minute window prevents double-triggers)
- The agent's response is an isolated session — it reads the task, writes a comment, exits
- No plan mode required for review tasks (they are scoped and bounded)
- CEO sees the review comment as part of Agent A's output before making any decision
- No tokens spent unless there is actually a new task

**What Agent B produces:**
Agent B reads Agent A's PR or document, writes a comment on it (either as a PR review or as an MC task comment), and closes the review task. Agent A's session can then incorporate that comment in a follow-up if needed.

---

## Model A vs Model B — comparison

| | Model A (standup-driven) | Model B (CEO-driven) |
|---|---|---|
| **Primary trigger** | System clock (07:30 cron) | CEO message |
| **Agent activity** | Every morning, regardless | Only when triggered |
| **CEO role** | Reactive (reads standup, then decides) | Active (initiates and gates every cycle) |
| **Cost when idle** | Daily standup tokens even if CEO unavailable | Zero |
| **Complexity** | High: 6 cron jobs, timing chain, skip logic | Low: 1 cron for review task detection |
| **Failure modes** | Standup runs while CEO is away for a week | Nothing happens until CEO returns |
| **Agent visibility** | All agents active every day | Only agents with active tasks |
| **Cross-agent input** | Via standup contributions (sequential) | Via review tasks (auto-triggered) |

---

## Heartbeat — redefined

Heartbeats are **not** a periodic "check in and report" mechanism. They are a **polling probe for external events** that the agent cannot be told about directly.

**Use a heartbeat only when:**
- There is an external system that the agent should react to
- That system cannot push a notification (no webhook available)
- The event is time-sensitive enough to warrant proactive detection

**Current valid heartbeat candidates:**

| External event | Agent | Poll mechanism |
|----------------|-------|----------------|
| CI test failure on `main` | Axle (Engine Dev) | Check GitHub Actions status |
| New grant deadline approaching | Marco (Programme Mgr) | Read `grant-tracker.md` dates |
| Open PR stale > 5 days | Veri (Quality Mgr) | Check GitHub open PRs |

**What heartbeats are NOT for:**
- Checking if there is work to do (that's the CEO's trigger)
- Generating standup content (that's an optional standup mechanism)
- Reporting status (that's what the CEO asks for directly)

**Heartbeat output:** when a heartbeat detects an alert condition, it writes a brief message to Telegram (CEO's Axle group, or a dedicated alerts group). It does NOT produce a full standup section or start any work — it raises a flag for the CEO to decide whether to act.

---

## Standup — redefined

Standup is an **optional visibility mechanism**, not a work trigger.

**CEO-triggered standup:**
```
CEO: /standup   (sends this message in Telegram)
    │
    ▼
Standup script runs on demand (same logic as cron version)
All agents contribute their current status
CEO reviews and decides whether to adjust any agent's direction
```

**What standup is for:**
- Cross-agent visibility when the CEO wants a broad picture
- Surfacing work that has drifted without producing explicit CEO-visible output
- Occasional cadence check (weekly, not daily)

**What standup is NOT for:**
- Starting new tasks (that is the CEO → agent direct trigger)
- Approving work (that happens in the work cycle)
- Replacing the work cycle (standup output does not create MC tasks directly)

**When to run:**
- At the CEO's discretion — weekly or when something feels off
- Before a planning session (to see current state before deciding what to start next)
- NOT on a fixed daily schedule

---

## Putting it together — the full system

```
        ┌─────────────────────────────────────────────────────┐
        │  CEO                                                 │
        │                                                      │
        │  • Starts work cycles (direct agent message)         │
        │  • Approves / amends / rejects output               │
        │  • Optional: /standup for broad visibility           │
        │  • Monitors: Telegram alerts from heartbeats         │
        └─────────┬──────────────────────────────┬────────────┘
                  │ work cycles                  │ visibility
                  ▼                              ▼
        ┌─────────────────┐           ┌──────────────────────┐
        │  Primary agent  │           │  Standup (optional)  │
        │  (Axle / Pixel  │           │  CEO-triggered only  │
        │  / Beacon etc.) │           │  All agents respond  │
        └────────┬────────┘           └──────────────────────┘
                 │ review task
                 ▼
        ┌──────────────────┐
        │  pi cron         │  ← lightweight, no LLM, always running
        │  (every 2 min)   │
        └────────┬─────────┘
                 │ auto-trigger
                 ▼
        ┌──────────────────┐
        │  Reviewer agent  │
        │  (usually Veri)  │
        └──────────────────┘
                 │
                 │ heartbeat alerts (external events only)
                 ▼
        ┌──────────────────┐
        │  Telegram        │
        │  CEO notification│
        └──────────────────┘
```

---

## What to build — ordered by value

| # | What | Why first |
|---|------|-----------|
| 1 | **`check-new-tasks.sh`** — the 2-min cron that auto-triggers reviewer agents | Enables the core cross-agent automation in Model B |
| 2 | **Telegram group bindings** (waiting for group IDs) | Enables CEO → agent direct messaging per agent |
| 3 | **`/standup` command** — CEO sends this in Telegram, triggers standup on demand | Replaces the broken daily standup cron with something useful |
| 4 | **`heartbeat-ci.sh`** — Axle polls GitHub Actions, fires alert to Telegram if `main` fails | First real external event heartbeat |
| 5 | **`heartbeat-grants.sh`** — Marco polls `grant-tracker.md` for approaching deadlines | Second real external event heartbeat |

Items 1 and 3 can be built immediately. Items 2, 4, 5 need Telegram group IDs or grant-tracker.md to exist first.

---

## What does NOT need to be built

- Daily standup cron chain (complex, expensive, fragile — replaced by on-demand `/standup`)
- Per-agent periodic heartbeats at fixed intervals (replaced by narrowly-scoped external event monitors)
- `export-backlog.sh` as a cron job (export on demand, or on MC task update)
