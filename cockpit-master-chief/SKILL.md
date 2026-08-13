---
name: cockpit-master-chief
description: >
  Operate as the top-level Master Chief in AGI Cockpit's three-layer workflow.
  Use when a Master Agent session needs to create and supervise project Leads,
  poll their report logs, review their results, or hand them over. Project Lead
  and worker sessions should not use this skill to create sibling masters.
---

# Cockpit Master Chief

Use this skill only from the top-level Master Agent. The Master is the human's single coordination window; a project Lead owns one workstream; workers execute focused tasks.

## Three layers

| Layer | Responsibility | Does not do |
|---|---|---|
| Master Chief | Intake, prioritization, Lead creation, supervision, review, and consolidated reporting | Routine implementation |
| Project Lead | Plan and supervise one project, delegate focused work, review results | Create sibling Leads or perform unapproved external actions |
| Worker | Execute one self-contained requirement and return evidence | Broaden scope or declare unreviewed work ready |

## 1. Intake

Clarify the outcome, decide whether the work needs a persistent Lead or a one-shot task, and keep account, production, publishing, and destructive-action gates explicit.

## 2. Create the right task

Create long-running Leads as top-level tasks so their workers remain beneath them in the UI hierarchy. As of Cockpit v4.49.0, unset the caller context variables to do this:

```bash
env -u AGI_COCKPIT_TASK_ID -u AGI_COCKPIT_CONTEXT_TASK_ID -u AGI_COCKPIT_CONTEXT_LOOKUP \
  cockpit task create --agent-type <agent-type> \
  --name "<project> Lead" --directory <repo> --instruction "<packet>"
cockpit task pin <new-id>
```

This is a v4.49.0 workaround; a future Cockpit release may provide an official flag.

Create a short one-shot task normally. It may remain nested under the caller:

```bash
cockpit task create --agent-type <agent-type> \
  --name "<task>" --directory <repo> --instruction "<packet>"
```

The initial packet must be self-contained. Include the role, one concrete requirement, repository and branch facts, scope boundaries, approval gates, expected evidence, and output format. For an asynchronously created worker, also include the completion protocol from `project-maintainer-orchestrator`.

Cockpit v4.49.0 removed `task create --report-back`, `task send --parent-task-id`, and the report-back prompt-injection/resume mechanism. Do not use those options or wait for automatic parent notifications.

## 3. Monitor by polling

Choose the command by execution style:

- `cockpit task run ...` creates a task and waits synchronously for its first report. Prefer it when the caller should block for the result.
- `cockpit task create ...` is asynchronous. Read `latestReportSeq` from `task get`, then poll with `cockpit task wait <id> --since <seq>`.
- `cockpit task send <id> --text "..." --wait` sends a follow-up and waits for that turn's report.
- `{"timeout": true}` from `task wait` is normal; repeat with the last processed sequence.

Reports are persistent task logs. They do not inject prompts into another task or resume a parent.

Do not send instructions directly to a Lead's workers. Cockpit v4.49.0 no longer reparents tasks on send, but the rule remains necessary to preserve one command path: the Master instructs the Lead, and the Lead instructs its workers.

## 4. Review before relaying

- Verify referenced branches, commits, files, URLs, screenshots, and test results.
- Use an independent reviewer for user-facing, production, data, or shared-repository changes.
- Return unresolved high-severity findings to the Lead before calling the work ready.
- Consolidate decisions for the human; do not forward raw worker claims as proof.

## 5. Handover

Before context runs low, ask the Lead to write a handoff packet containing current state, artifact locations, task IDs, latest processed report sequences, decisions, blockers, and next actions. Use `chief-handover` when replacing the Master itself.

## Never

- Do not complete or remove tasks without explicit authorization.
- Do not publish, message externally, deploy, bill, or mutate production without explicit authorization.
- Do not create multiple Master Chiefs.
- Do not treat a worker's completion claim as reviewed evidence.

## Related skills

- `chief-handover` — transfer a Master session and resume polling existing lanes
- `project-maintainer-orchestrator` — Lead and worker orchestration, including the worker completion protocol
