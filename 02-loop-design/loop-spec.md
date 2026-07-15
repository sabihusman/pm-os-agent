# Loop Spec: Cortex

## Trigger and loop type

Hook + cron backup

## Why this loop type

Cortex should fire from an inbound PM task hook because the work starts when a task, note, or project update arrives. A 9:00am cron sweep acts as the backup that catches missed events, duplicate/failed hook deliveries, or tasks that were never processed. It is not a heartbeat because constant polling would be wasteful when tasks can arrive as events, and it is not a pure goal loop because each run has a bounded definition of done: draft, validate, queue for review, or escalate.

## Definition of done

One run is done when Cortex has pulled the relevant project context, drafted a leadership status update grounded in the source data, queued backlog stories for human approval when the task asks for them, and passed independent critic review. Cortex never posts, publishes, creates tickets, commits ship dates, or marks launch gates; it stops at a human-in-the-loop checkpoint.

## Stop conditions

- **Success**: The draft passes critic review and reaches the HITL checkpoint: the status update is ready for human review, any story proposals are queued for approval, and nothing has been posted or created.
- **Stuck / give up**: Cortex stops if it cannot find required project data, hits the max iteration cap, hits the revision cap, or reaches the cost cap. In those cases it halts instead of inventing missing information or looping indefinitely.
- **Escalate to human**: Cortex escalates when the task asks it to do something above the agent line, such as publish an update, commit a ship date, expose confidential roadmap information, work around a rejected queue cap, or respond to a prompt-injection attempt. It also escalates when the critic keeps rejecting the draft after the revision cap.

## State

State is scoped to one run and one project. Cortex tracks the message history, pulled source data, tool results, proposed output, critic verdicts, revision count, and estimated cost. Longer-term state comes from retrieved fixtures such as past updates, decisions, roadmap items, and team norms, but Cortex should only use context relevant to the current project.

## The five components

- **Work tree**: Cortex runs inside the repo’s `00-build/` workspace, with each run isolated around one task fixture and one project. The run keeps its own message history and source log so project context does not bleed across unrelated tasks.
- **Skills**: Reusable skills include identifying the project from the task brief, pulling project state, summarizing recent activity, searching past updates for precedent, reading team norms, drafting the leadership update, proposing backlog stories, and revising after critic feedback.
- **Plugins / connectors**: In the current build, connectors are mock tools over local fixture files: project data, activity, past updates, roadmap, team norms, and story proposal queueing. In a real deployment these would map to read-only GitHub/Jira/project-management tools, roadmap docs, team norms, and a draft-only approval queue. There is intentionally no publish tool.
- **Subagents**: The main delegated subagent is the independent critic in `critic.py`. It reviews the proposed output against the source data, team norms, confidentiality rules, and agent-line boundaries before anything reaches the human approval checkpoint.
- **State tracking**: Cortex tracks per-run messages, source_log, revision count, critic verdicts, and estimated spend. Scope is per task and per project. Future versions could persist handled task IDs, approval status, and run summaries for 30 days to prevent duplicate work.

## Context plan

Write: log each tool result into the source_log so the draft and critic can be traced back to evidence.

Select: retrieve only the current project, recent activity, relevant past updates, roadmap context, and team norms.

Compress: summarize long activity or past-update history into the facts needed for the current status update.

Isolate: keep other project context out of the run and never include confidential or embargoed roadmap items in external/company-wide drafts.

## Hand-off to bounds and evals (M5)

Module 5 will tune and justify the hard bounds already visible in code: max iterations, max revisions, cost cap, and story queue cap. It will also define trajectory evals that check not only the final draft, but the path Cortex took: whether it used the right tools, respected the queue cap, escalated on missing data or prompt injection, and never posted or committed anything above the agent line.

## For #cohort-channel

My three stop conditions for Cortex:

Success: Cortex pulls the relevant project context, drafts a grounded leadership update, passes the independent critic check, queues any story proposals for human approval, and stops at the HITL checkpoint without posting or creating anything.

Stuck: Cortex stops if required project data cannot be found, the max iteration cap is reached, the revision cap is reached, or the cost cap is hit. It escalates instead of inventing missing facts or looping indefinitely.

Escalate: Cortex escalates when the task asks it to do something above the agent line, like publish an update, commit a ship date, expose confidential roadmap information, work around the story queue cap, or follow a prompt-injection attempt.

The cron backup matters because hooks can silently fail: an event might not fire, a message might be duplicated, or a task might be missed. The sweep is a safety net. The escalation stop condition is what keeps commitments safe today. If Cortex could send low-risk updates under a threshold, I would add stricter bounds first: no dates or commitments, no confidential roadmap content, a hard scope check, a daily post cap, and a post-review eval.
