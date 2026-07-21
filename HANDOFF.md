# Handoff: Cortex (pm-os-agent)

> Living handoff doc, not a per-module snapshot. Update this as the project moves
> instead of writing a new one each module. Last refreshed: end of Module 4
> (Memory & Context), before starting Module 5 (Bounds & Evals).

## Project summary

Cortex is a PM chief-of-staff agent built in `00-build/` as the course's hands-on
build. It's a transparent, hand-written tool-calling loop (no framework) around the
OpenAI API: one primary agent (Cortex) plus one validating subagent (the critic).
Given a PM task brief, it pulls project context via read-only tools, drafts a
leadership status update, proposes next-sprint backlog stories (queued, never
created), and stops at a human-in-the-loop checkpoint. It never posts, publishes,
creates/merges tickets, or commits a date — those tools don't exist.

Repo: `sabihusman/pm-os-agent` (GitHub), cloned locally at
`C:\Users\sabih\OneDrive\Desktop\VSCode\AI Agents Course\pm-os-agent`.

## Environment

- **Python**: 3.14.0 (`C:\Users\sabih\AppData\Local\Programs\Python\Python314`)
- **pip**: 26.0.1
- **No virtualenv** — dependencies are installed to the system Python install.
  If you set up a venv later, reinstall `00-build/requirements.txt` plus the
  packages below into it.
- **Installed beyond `requirements.txt`**: `langsmith` (0.10.2), `pip-system-certs`
  (5.3, see friction points below). `openai` (2.45.0) and `python-dotenv` (1.2.2)
  were already present.
- **OS**: Windows 11. Shell used for repo work: Git Bash / PowerShell.

## Recurring friction points

- **SSL cert verification fails on this machine** — Avast antivirus's HTTPS-scanning
  proxy filter (`aswMonFltProxy`, visible via the `SSLKEYLOGFILE` env var) intercepts
  TLS and re-signs with its own root cert, which Python's bundled `certifi` doesn't
  trust. Symptom: `CERTIFICATE_VERIFY_FAILED: unable to get local issuer certificate`
  on both the OpenAI and LangSmith endpoints. **Fixed** by installing
  `pip-system-certs`, which patches Python to trust the Windows OS certificate store
  instead (Windows already trusts Avast's injected cert). If this box is ever
  reprovisioned or a fresh venv is created, this fix needs to be reapplied.
- **Known unfixed bug**: `agent.py`'s `banner()` crashes with
  `UnicodeEncodeError: 'charmap' codec can't encode character '≈'` on the final
  "Run cost ≈ $X" banner, because Windows' default console codepage (cp1252) can't
  encode `≈`. This only hits the *last* print of a run (HITL checkpoint pass, or the
  max-iterations escalate) — everything before it, including critic verdicts, prints
  fine. Not yet fixed; flagged to the user twice, not yet actioned. Trivial fix
  (reconfigure stdout encoding to UTF-8, or swap `≈` for `~`) whenever it's worth
  doing.

## `.env` (`00-build/.env`) — variable names only, no secret values here

`.env` is confirmed gitignored (`.gitignore:8:.env`) and has never appeared in
`git status`. Do not paste real key values into this file or into chat.

```
OPENAI_API_KEY=<set by user>
CORTEX_MODEL=gpt-4o-mini
CORTEX_MAX_ITERATIONS=8
CORTEX_MAX_REVISIONS=2
CORTEX_COST_CAP_USD=0.50
CORTEX_MAX_QUEUE_ITEMS=10
CORTEX_PRICE_IN_PER_M=0.15
CORTEX_PRICE_OUT_PER_M=0.60
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=<set by user>
LANGSMITH_PROJECT=cortex
```

The provider-side hard spend cap (separate from `CORTEX_COST_CAP_USD`, which is a
code-enforced runtime bound) is set in the OpenAI dashboard
(Settings → Billing → Limits) — not in this repo.

## What Cortex does

1. Reads a PM task brief (fixtures: `task-happy`, `task-missing-data`,
   `task-jailbreak`).
2. Calls read-only tools (`00-build/tools.py`): `get_project`, `get_activity`,
   `search_past_updates`, `get_roadmap`, `get_norms`.
3. Optionally calls `propose_stories` to **queue** backlog stories for approval
   (rejected if the batch exceeds `CORTEX_MAX_QUEUE_ITEMS`; never creates anything).
4. Drafts a status update, grounded in the pulled data.
5. The **critic** (`00-build/critic.py`, `review()`) independently checks the draft
   against 6 rules in `CRITIC_SYSTEM` (`00-build/prompts.py`) — traceable claims,
   norms compliance, nothing posted/committed, jailbreak refusal, bound-trip
   handling — and returns `{"verdict": "pass"|"fail", "reasons": [...]}`.
6. On `fail`, Cortex revises (cap: `CORTEX_MAX_REVISIONS=2`, enforced in `agent.py`).
   After the cap, or after `CORTEX_MAX_ITERATIONS=8` total steps, it escalates to a
   human instead of looping.
7. On `pass`, it stops at the HITL checkpoint — update + any proposed stories
   queued for PM review, nothing auto-sent.

**LangSmith tracing** (added post-M1, committed `0a9a0ab`): the OpenAI client in
`agent.py` is wrapped with `wrap_openai()`; `run()` (`agent.py`) and `review()`
(`critic.py`) are both decorated `@traceable`, giving a nested trace tree (parent
run → tool calls → critic call) with per-step cost/latency, visible under the
`cortex` project in the LangSmith dashboard.

## Repo structure

```
pm-os-agent/
├── HANDOFF.md                          ← this file
├── 00-build/                           Module 1–6 build: agent.py, critic.py,
│   │                                    prompts.py, tools.py, requirements.txt,
│   │                                    .env (gitignored), .env.example,
│   │                                    RUNBOOK.md, PROMPTS.md, CORTEX-ANATOMY.md,
│   │                                    DEMO-SCRIPT.md
│   └── fixtures/                       task-happy.md, task-missing-data.md,
│                                        task-jailbreak.md, + mock data JSON/MD
├── 01-agent-line/agent-line-map.md     M1 deliverable — done
├── 02-loop-design/loop-spec.md         M2 deliverable — done
├── 03-orchestration/orchestration-map.md  M3 deliverable — done
├── 04-memory-context/memory-and-context.md  M4 deliverable — done
├── 05-bounds-evals/bounds-and-evals.md M5 — not started
├── 06-autonomy/
│   ├── prototype.md                    M6 deliverable — Module 3 section filled in
│   │                                    (2 critic-rejection screenshots embedded);
│   │                                    screenshot table rows 1, 3, 4, 5, 6 are
│   │                                    still open `_[img]_` placeholders
│   ├── critic-rejection-1.png          M3 evidence: critic fails draft 1 (check #2,
│   │                                    invented causation) → revision 1/2
│   ├── critic-rejection-2.png          M3 evidence: critic fails draft 2 → hits
│   │                                    revision cap 2/2 → escalate
│   ├── build-insights.md               not started
│   └── governance-and-strategy.md      not started
├── LICENSE
└── README.md
```

## Progress (commits, all pushed to `origin/main`)

```
e6b05ae Module 4: memory & context analysis, verified hallucination catch
ee55392 Add HANDOFF.md living handoff doc through Module 3
7eb0e05 Module 3: orchestration map + critic rejection screenshots
e126a56 Add Module 2 loop spec
0a9a0ab Add LangSmith tracing with nested traces (wrap_openai + @traceable)
f9ebeca Module 1: revise agent-line map formatting
972a97a Module 1: agent-line map for Cortex
```

- **M1** (agent line) — done, committed.
- **M2** (loop spec) — done, committed (`e126a56`).
- **M3** (orchestration map + critic) — done, committed (`7eb0e05`). Verified via a
  live `python agent.py` run on `task-happy`: critic rejected two drafts in a row
  (revision 1/2, then 2/2) before the run hit `MAX_ITERATIONS` — both rejections
  screenshotted and embedded in `prototype.md`.
- **LangSmith tracing** — done, committed (`0a9a0ab`), predates M2/M3 commits.
- **M4** (memory & context) — done, committed (`e6b05ae`). Filled in via close
  reading of `agent.py`/`tools.py`/`critic.py` plus 3 live `task-happy` runs, not
  just prose: found and documented a real Cortex/critic drift on green-vs-yellow
  thresholds, a `search_past_updates` fallback that silently returns irrelevant
  precedent when nothing matches, and that `get_roadmap`'s confidentiality bound
  is prompt-enforced (soft) unlike the hard-blocked publish/create/date bounds.
  Verified live that withholding the `activation_rate` source did NOT cause
  Cortex to invent a number (3 revisions, all qualitative); separately confirmed
  the critic's traceability check catches an invented metric via a targeted
  adversarial draft fed straight to `critic.review()`.
- **M5** — not started.
- **M6 (`prototype.md`)** — Module 3 and Module 4 sections filled in (Module 4
  as text/JSON evidence, not a screenshot — no screenshot tool was available this
  session); the top screenshot table still needs rows 1, 4, 5, 6.

## Open tasks

1. **Verify the M2 cohort-channel response was actually posted** — unconfirmed,
   check before assuming it's done.
2. **Post the M3 cohort-channel response** (orchestration map + critic:
   fail-action = revise then escalate, revision cap = 2).
3. **Start Module 5 (Bounds & Evals)** — `05-bounds-evals/bounds-and-evals.md`
   is still the unedited template; bounds themselves are already set in `.env`
   (see "Decisions already made" below) but not yet formally justified there.
   Also consider: M4 surfaced a real Cortex/critic drift on green-vs-yellow
   thresholds — tightening that rubric is arguably M3/M5-adjacent cleanup, not
   yet actioned, flag to the user before touching `CORTEX_SYSTEM`/`CRITIC_SYSTEM`
   since M1–M3 are marked done and not up for casual re-litigation.
4. **Fill remaining `prototype.md` screenshot rows** — 1 (happy-path/HITL), 4
   (jailbreak refusal, M5), 5 (bound trip, M5), 6 (end-to-end, M6). Row 3 (M4)
   is filled with text/JSON evidence, not an image — no screenshot tool was
   available; if a real screenshot becomes possible later, consider swapping it
   in to match rows 1/2's format.

## User working-style notes

- Wants **diffs/status shown before committing**, and staged files reviewed against
  what's actually intended — has caught and corrected scope drift before (e.g.
  confirming exactly which files get staged, never blanket `git add -A`).
- Explicitly values **accuracy over convenience** — when told about a minor
  inaccuracy (e.g. a doc claiming a function "takes only X and Y" when it also took
  two more plumbing args), the answer was "fix it," not "leave it, it's close enough."
- Prefers **secrets never touch the chat** — keys are pasted directly into `.env` by
  the user, not shared in conversation; `.env`'s gitignore status gets re-verified
  after every edit that touches it.
- Asks **pointed clarifying questions before acting** on ambiguous claims (e.g.
  questioned whether a "critic rejected it naturally" claim was a real captured run
  or a hypothetical, before deciding whether a rerun was even necessary) — reward
  precision, don't round up.
- Runs this project through a **coding agent by design** (per `RUNBOOK.md` — "you
  never have to hand-write it"), but treats the coding agent's output as something
  to verify, not trust blindly.

## Course arc & grading (from `00-build/CORTEX-ANATOMY.md`, `RUNBOOK.md`)

Modules: **M1** agent line (what Cortex may/may not do) → **M2** loop + definition
of done → **M3** fleet + critic (orchestration) → **M4** memory & context
(grounding, hallucination catching) → **M5** bounds (iteration/cost/queue caps,
jailbreak refusal) → **M6** autonomy (`prototype.md`, the full demo).

Seven things a submission must prove (each needs a real screenshot/snippet, not
just a claim): loop + definition of done; the read-only tool list and the
deliberately absent post/create/merge tools; an independent critic with a
fail-action and revision cap; a hard iteration cap that stops a runaway; a cost +
commitment bound enforced outside the model; a HITL checkpoint (queued, never
posted); and a jailbreak refusal.

Four gated runs the screenshots come from: **happy path** (`task-happy`),
**stuck/escalate** (`task-missing-data`), **jailbreak** (`task-jailbreak`), and a
**bound trip** (lowering `CORTEX_MAX_ITERATIONS`, `CORTEX_COST_CAP_USD`, or
`CORTEX_MAX_QUEUE_ITEMS` below what the run needs, to prove the loop halts on the
bound rather than on success).
