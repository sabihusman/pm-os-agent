# Orchestration Map: Cortex

## Why split (single agent vs a team)

Cortex is a hook loop, triggered by an inbound PM task, that pulls project data, proposes a batch of backlog stories, and drafts a leadership status update — stopping at a HITL checkpoint for PM review (never auto-sends).

Default-to-simple check (four reasons to split):

| Reason | Applies? | Why / why not |
|---|---|---|
| Separation of concerns | No | Cortex's three jobs (pull data, propose stories, draft update) all serve one goal — a good weekly update — and the constraints (no invented numbers, no confidential leaks, house tone) reinforce accuracy rather than pull against it. |
| Parallelism | No | Cortex's steps are sequential by necessity: it can't draft the update before pulling the data, and it can't propose stories before reading the roadmap. |
| Independent validator | **Yes** | Cortex drafts status updates and proposes story batches that a PM has to trust; a separate critic that didn't write the draft catches invented numbers, missed norms, or over-cap batches before they reach the PM — the drafter can't reliably grade its own work. |
| Context-window pressure | No | Cortex pulls small fixture files that fit comfortably in gpt-4o-mini's 128k window; runs cost ~$0.004 and have never hit a context limit. |

**The call:** Cortex splits. One subagent — the critic. Justified by the independent-validator reason: the drafter can't grade its own homework, and the PM needs to trust what lands on their desk.

## Topology

Single + subagents. One primary agent (Cortex) with one validating subagent (the critic).

```
[Inbound PM task]
        │
        ▼
   [Cortex]  ──(reads)──> get_project, get_activity, search_past_updates,
        │                 get_roadmap, get_norms  (read-only tools)
        │  drafts status update + proposes story batch (queued, not created)
        ▼
   [Critic]  ── fail ──> back to Cortex (max 2 revisions) → escalate to PM
        │
        │ pass
        ▼
   [PM review checkpoint]  → human reviews; nothing auto-sends
```

## Agent roster

| Agent | Responsibility | Loop Spec it runs |
|---|---|---|
| Cortex | Pull project data, propose backlog stories (queued, not created), draft the leadership status update | M2 hook loop (`02-loop-design/loop-spec.md`) |
| Critic | Independently review Cortex's drafted update + proposed story batch; return `pass` or `fail` with reasons | Short goal loop: read draft → check against 6 rules → return verdict → stop |

## The validating subagent (critic)

**What it checks** (6 checks in `CRITIC_SYSTEM`, `00-build/prompts.py`):

1. Does the draft reference the correct project and real activity (PRs / issues / status) from the pulled data?
2. Is every claim (progress, metrics, dates, red/yellow/green calls) traceable to the pulled data — no invented progress and no invented numbers?
3. Does it stay within team norms (no unconfirmed date committed, no launch gate marked, no CONFIDENTIAL roadmap item in an external/company-wide update), or correctly escalate if not?
4. Does it post nothing, commit nothing, create/close/merge nothing (stories only PROPOSED / queued), and leak no confidential roadmap?
5. If the task tried to jailbreak Cortex, did Cortex refuse and escalate?
6. If an enforced bound was hit (e.g. `propose_stories` returned `batch_exceeds_queue_cap`), is Cortex correctly escalating rather than trying to work around it?

**Fail-action**: **Revise, then escalate** (tiered). On fail, the critic returns reasons and the draft goes back to Cortex for another pass. After the revision cap is hit, Cortex stops and escalates to the PM with the last draft + the critic's reasons attached. Nothing auto-sends at any point.

**Revision cap**: **2** (enforced in code via `CORTEX_MAX_REVISIONS=2` in `.env`, checked in `agent.py`). After 2 failed revisions, escalate to the PM. This number gets formally justified in Module 5 (Bounds & Blast Radius).

## Hand-offs

- **Cortex → Critic**: the drafted status update + proposed story batch (as `proposed_output`), plus the source data Cortex pulled (as `source_data`). Plain in-process function call (`critic.review(client, model, proposed_output, source_data)`); no MCP or A2A protocol involved.
- **Critic → Cortex**: strict JSON — `{"verdict": "pass" | "fail", "reasons": [...]}`. On `fail`, Cortex revises. On `pass`, the draft advances to the PM review checkpoint.
- **Cortex → PM (on escalate)**: the last draft + the critic's reasons + the stop condition that fired (revision cap or iteration cap).

## Shared state

- **Shared**: the drafted update + story batch (passed to the critic as `proposed_output`), and the source data Cortex pulled (passed as `source_data` so the critic can cross-check claims against real data).
- **Isolated**: the critic does not see Cortex's system prompt, chain of reasoning, or tool call history. Verified in `critic.py`: `review()` receives only the finished output and the raw source data — which is what makes it a genuinely independent check. (The `client` and `model` args are API plumbing, not context.)

## Cost and latency budget

- **Extra calls per item**: one critic call per draft. Worst case with the revision cap: 1 draft + 2 revised drafts × 1 critic call each = up to 3 critic calls per run (plus the 3 draft calls).
- **Observed cost**: ~$0.004–0.005 per full run (measured across ~4 runs on gpt-4o-mini). Well under the `CORTEX_COST_CAP_USD=0.50` hard cap.
- **Added latency**: each critic pass adds one model round-trip. A worst-case revision-capped run is roughly 3× the latency of a clean pass.
- **Forward-link to M5 (Bounds & Blast Radius)**: this budget becomes the enforceable bound — revision cap, iteration cap, and cost cap are all already set in `.env` and will be tuned/justified in M5.
