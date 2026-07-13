# Agent-Line Map — Cortex (PM Chief-of-Staff Agent)

**Module 1 deliverable** · Decisions broken out, scored on three axes, and placed above or below the agent line.

## The framework

- **Reversibility** — if Cortex gets this wrong, how easily do we undo it?
- **Blast radius** — how much damage does a wrong call cause before someone catches it?
- **Measurability** — can we reliably tell, after the fact, whether it was right?

**Golden rule:** High reversibility + low blast radius + high measurability → **Below** the line (Cortex owns). Low reversibility, OR high blast radius, OR low measurability → **Above** the line (human owns), or **HITL** (Cortex does the work, a human approves before it proceeds).

## Scored decision table

| # | Decision / action | Reversibility | Blast radius | Measurability | Verdict | One-line justification |
|---|---|---|---|---|---|---|
| 1 | Pull project state / recent activity | High | Low | High | Below | Read-only and re-runnable, so a wrong pull costs nothing — Cortex owns it. |
| 2 | Decide which past context is relevant | High | Low | Med | Below | Cheap to correct and low-risk; a bad pick only weakens a draft nobody's sent. |
| 3 | Draft the leadership status update | High | Low | High | Below | Nothing leaves the building until a human sends, so drafting sits safely below. |
| 4 | Decide tone / commitment level (e.g. promising a date) | Med | Med | Low | HITL | A commitment is hard to walk back and tone is fuzzy to score, so a human approves. |
| 5 | Flag a project at-risk / issue as likely escalation | High | Low | Med | HITL | A wrong at-risk flag erodes trust in the system, so a human confirms it before it stands. |
| 6 | Choose which risk call to escalate to a human | Med | Med | Med | HITL | All three axes are middling, so a human confirms which call actually goes up. |
| 7 | Propose a story batch within the cap | Med | Med | Low | HITL | Reversible but its justification is hard to verify, so a human clears the batch. |
| 8 | Post an update / approve a company-wide one | Low | High | Low | Above | Near-impossible to reverse with a high blast radius — a human owns this, always. |

**Split:** 3 Below · 4 HITL · 1 Above.

## Hardest above-vs-below call

**Propose a story batch within the cap (#7).** I went back and forth on this one. Reversibility pulls it toward Below — a queued proposal undoes easily, and in a real run Cortex only queued the batch `for_approval` and created nothing in the tracker. But a strict reading pulled it toward Above, because you can't reliably tell after the fact whether it proposed the *right* stories. **Measurability** is the axis that settled it: because nothing else can verify whether the batch is correct, a human has to clear it — so it lands at HITL, not Below and not Above.
