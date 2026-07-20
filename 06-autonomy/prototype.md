# Prototype: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 1, the working agent demo

## What it does

_One paragraph: the agent in action, end to end._

## How you built it

- **Coding agent:** _which one you directed (Claude Code / Cursor / Codex)_
- **Model + bounds:** _model used, max iterations, cost cap, queue cap_
- **Repo / config:** _path to your build in `00-build/`_
- **Live link:** _[shareable URL, optional bonus]_

## Screenshots (required, collected M2 to M6)

Real screenshots of *your* Cortex running. These are the `00-build/CORTEX-ANATOMY.md` set and they are required, a link alone is not enough.

| # | Screenshot | What it shows | From |
|---|---|---|---|
| 1 | _[img]_ | happy-path run: a real drafted update + the HITL checkpoint (queued, not posted) | M2 |
| 2 | [See Module 3 section below](#module-3--critic-rejecting-a-bad-draft) | the critic rejecting a bad draft (revise/block) | M3 |
| 3 | _[img]_ | a grounded update citing pulled activity + a caught hallucination | M4 |
| 4 | _[img]_ | jailbreak refused + escalated | M5 |
| 5 | _[img]_ | an iteration/cost/queue bound halting a runaway | M5 |
| 6 | _[img]_ | end-to-end run | M6 |

## How to run it

_Minimal steps for someone to reproduce the demo (env vars, and the command or the coding-agent prompt you used)._

## Module 3 — Critic rejecting a bad draft

![The critic rejecting Cortex's first draft for invented causation and unsupported claims](critic-rejection-1.png)

Image 1: The critic catching invented causation and unsupported claims in Cortex's first draft (check #2: traceable claims). Fail-action fires — draft goes back for revision 1/2.

![The critic rejecting Cortex's second revision, hitting the full revision cap](critic-rejection-2.png)

Image 2: The critic rejecting the second revision as well, hitting the full revision cap (2/2) — after which the loop stops and escalates to the PM.

Both screenshots are from a live `python agent.py` run on the `task-happy` fixture; see [`03-orchestration/orchestration-map.md`](../03-orchestration/orchestration-map.md) for the design context behind the critic/revision-cap setup.
