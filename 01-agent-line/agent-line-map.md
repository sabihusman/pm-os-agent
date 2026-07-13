# Agent-Line Map: Cortex

## Decisions, scored

| # | Decision | Reversibility | Blast radius | Measurability | Verdict |
|---|---|---|---|---|---|
| 1 | Pull project state / activity | High | Low | High | Below |
| 2 | Read-only and re-runnable, so a wrong pull costs nothing — Cortex owns it. | High | Low | Med | Below |
| 3 | Draft the leadership status update | High | Low | High | Below |
| 4 | Decide tone / commitment level | Med | Med | Low | Above |
| 5 | Flag a project at-risk / issue as likely escalation | High | Low | Med | HITL |
| 6 | Choose which risk call to escalate | Med | Med | Med | HITL |
| 7 | Propose a story batch within the cap | Med | Med | Low | HITL |
| 8 | Post an update / approve a company-wide one | Low | High | Low | Above |

## One-line justifications

1. **Pull project state / activity** (Below): Read-only and re-runnable, so a wrong pull costs nothing — Cortex owns it.
2. **Read-only and re-runnable, so a wrong pull costs nothing — Cortex owns it.** (Below): Cheap to correct and low-risk; a bad pick only weakens a draft nobody's sent.
3. **Draft the leadership status update** (Below): Nothing leaves the building until a human sends, so drafting sits safely below.
4. **Decide tone / commitment level** (Above): A commitment is hard to walk back and tone is fuzzy to score, so a human approves.
5. **Flag a project at-risk / issue as likely escalation** (HITL): A wrong at-risk flag erodes trust in the system, so a human confirms it before it stands.
6. **Choose which risk call to escalate** (HITL): All three axes are middling, so a human confirms which call actually goes up.
7. **Propose a story batch within the cap** (HITL): Propose a story batch within the cap
8. **Post an update / approve a company-wide one** (Above): Near-impossible to reverse with a high blast radius — a human owns this, always.

## Hardest above-vs-below call

row 7 (propose stories) is the natural pick. Reversibility pulls it toward Below (a queued proposal undoes easily — you watched Cortex queue 5 stories and create nothing), but measurability is what drags it to HITL — you can't reliably tell after the fact whether it proposed the right stories. If that matches what you actually wrestled with, the deciding axis is measurability.
