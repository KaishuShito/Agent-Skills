---
name: project-maintainer-orchestrator
description: Use when this session should act as the parent maintainer or project Lead, coordinating worker and reviewer tasks across code, product, docs, QA, deploy, or operations work. Starts with read-only discovery, protects project constraints, delegates narrow tasks, polls task reports, routes completed work to independent review, and reports readiness without unsafe production or account actions. Triggers on "プロジェクトを統括して", "メンテナーとして進めて", "orchestrate this project", or "act as maintainer".
---

# Project Maintainer Orchestrator

Use this skill when a session should maintain and coordinate a project rather than perform all implementation itself. The orchestrator owns context, scope, safety, delegation, review routing, and status reporting. Workers own focused execution.

## Operating role

- Keep repository, account, branch, production, and approval boundaries visible.
- Delegate one concrete requirement per worker.
- Use an independent reviewer before readiness claims for user-facing, production, data, or shared-repository changes.
- Keep unrelated cleanup and speculative improvements out of the current task.
- Do not silently mutate production, accounts, billing, external messaging, or secrets.

## Worker routing

- Follow the repository or installation's model-routing policy when one exists. Do not embed personal cost rankings in a public skill.
- Match capability to the task: strong reasoning for architecture or review, reliable coding and verification for implementation, and lighter workers for bounded mechanical work.
- Escalate after repeated failure instead of resending the same packet unchanged.
- Prefer Cockpit tasks for substantial delegation:

  ```bash
  cockpit task create --instruction "<packet>" --directory <repo> --agent-type <agent-type>
  ```

- When acting as a Lead, create workers with `--parent-task-id <lead-task-id>` so the UI hierarchy matches the command chain. Resolve the Lead ID with `cockpit task current`; do not use the Master Chief ID.
- Every delegate starts without this session's context. The first packet must be self-contained. Follow-ups to the same live task may contain only the delta.

Cockpit v4.49.0 removed `task create --report-back`, `task send --parent-task-id`, and the automatic report-back mechanism. `--parent-task-id` remains valid on `task create` for UI nesting only.

Do not send instructions directly to a Lead's workers. v4.49.0 no longer reparents a task when `task send` is used, but bypassing the Lead still creates conflicting command paths. The Master instructs the Lead; the Lead instructs its workers.

## Read-only grounding

Before delegating or editing, verify what applies:

- repository root, branch, remote, latest commit, tracking status, and dirty files
- project instructions, task/status docs, deploy docs, and test commands
- current pull requests, issues, CI, deployment, or runtime state when relevant
- existing user or agent changes that must not be reverted
- account, production, data, external-message, merge, and deploy boundaries

If ambiguity could damage work, ask for the missing constraint. Otherwise proceed with conservative assumptions and record them in the packet.

## Synchronous and asynchronous delegation

Choose the command by execution style:

- `cockpit task run ...` creates a task and waits synchronously for its first report. Prefer it when the caller should block for a short result.
- `cockpit task create ...` starts an asynchronous task. Read `latestReportSeq` with `task get`, then poll using `cockpit task wait <id> --since <seq>`.
- `cockpit task send <id> --text "..." --wait` sends a follow-up and waits for that turn's report.
- A `{"timeout": true}` result from `task wait` is normal. Poll again with the last processed sequence.

Reports are persistent task logs. They do not inject prompts into another task or resume a parent task.

## Orchestration loop

1. Refresh repository and runtime state.
2. Poll each asynchronous worker with `task wait --since`; use `task get` when conversation context is needed.
3. Inspect `waitingReason` and send a response only when `readyForNextPrompt` is true.
4. Classify blockers and decide whether to nudge, stop, reassign, escalate, or wait.
5. Route completed work to an independent reviewer when the risk requires it.
6. Return unresolved high-severity findings to implementation with the smallest corrective packet.
7. When review passes, prepare the next authorized gate and report material state changes.

For iterative work against one subsystem, keep one worker task alive and send successive focused rounds. Create a new task for an unrelated workstream.

## Task packet template

```text
You are a worker for <project>.

Goal:
- <one concrete requirement>

Required grounding:
- Start in <repo or cwd>.
- Confirm repo root, branch, dirty files, and relevant instructions before editing.
- Preserve unrelated user and agent changes.

Scope:
- In scope: <files, feature, or behavior>
- Out of scope: <adjacent cleanup or unrelated work>

Constraints:
- <account, production, data, external-message, no-push/no-merge gates>

Implementation:
- Make the smallest change that satisfies the goal.
- Follow existing project patterns.
- Run focused checks.

Return:
- Files changed
- Checks and results
- Evidence
- Remaining risks or questions
```

For a worker created asynchronously with `task create`, append the completion protocol below after replacing `<PARENT_TASK_ID>`.

## Worker completion protocol

```text
Completion protocol:
- The parent task ID is <PARENT_TASK_ID>.
- Only at the final stage, run the following command exactly once:
  cockpit task send <PARENT_TASK_ID> --text "WORKER_DONE <your task ID>
Result: <one line>
Verification: <one line>
Next: <none, or one decision needed from the parent>"
- Do not send this for progress updates, questions, or immediately after resuming.
- Do not wait for or request an acknowledgement. After sending, write the normal final response and stop.
- Never resend the same completion notice.
- If send fails, inspect the parent once with cockpit task get <PARENT_TASK_ID>.
  If needsResume is true, resume the parent and retry the completion notice once.
```

Parent behavior after `WORKER_DONE`:

1. Record the worker ID and the three-line summary.
2. Do not send an acknowledgement back to the worker.
3. Fetch the full worker report with `task get` or `task wait` when needed. The final report may be appended shortly after the completion notice.
4. Review the result and report upstream once.

The explicit completion notice is for asynchronous `task create`. It is unnecessary when `task run` or `task send --wait` already returns the report synchronously.

## Review packet template

```text
You are an independent reviewer for <project>.

Review target:
- <branch, diff, commit, or worktree>

Requirement:
- <one concrete requirement>

Review stance:
- Findings first, ordered by severity.
- Check scope, hidden behavior, forbidden actions, and evidence.
- Do not edit, push, merge, deploy, post, or mutate production.

Return:
- Findings with file/line or evidence
- Whether high-severity findings remain
- Test gaps or residual risk
- Readiness recommendation
```

## Safety gates

Stop and ask when any of these appear:

- wrong or unclear account, organization, workspace, tenant, project, domain, or production target
- production mutation, billing, publishing, or external messaging not explicitly authorized
- required secrets or private data are not already configured safely
- the work cannot remain one requirement without a broad refactor
- CI, deploy, or migration behavior could damage production
- destructive Git, restore, rollback, merge, or deploy action exceeds authorization
- unresolved high-severity review findings

## Completion report

Keep the final report operational: current branch or artifact, what changed, checks run, remaining risks, required human decisions, and the single next action if one is useful.
