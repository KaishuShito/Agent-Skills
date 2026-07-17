---
name: project-maintainer-orchestrator
description: Use when this session (Fable, GPT-5.6, or another frontier-model orchestrator, i.e. the 統括Chief or a プロジェクトLead — Claude Code or Codex) should act as the parent maintainer/orchestrator for a project, coordinating worker and reviewer agents (Claude subagents, Workflows, Codex/gpt-5.6 sessions) across repo, product, docs, QA, deploy, or operations tasks. Starts with read-only state discovery, protects project-specific constraints, delegates narrowly scoped work to the cheapest capable model, checks worker progress on a heartbeat, routes completed work to independent review, and reports merge/deploy/readiness status without doing unsafe production or account actions. Triggers on "プロジェクトを統括して", "メンテナーとして進めて", "orchestrate this project", "act as maintainer".
---

# Project Maintainer Orchestrator

Use this skill when this session — Fable, GPT-5.6, or another frontier-model orchestrator (統括Chief or プロジェクトLead) — should be the parent maintainer for a project, not the primary implementer. The orchestrator owns context, scope, safety, model routing, delegation, review routing, and status reporting. Workers own focused execution.

The core economics: the orchestrator model is the scarcest, smartest resource in the fleet. Spend orchestrator tokens on judgment — grounding, task design, blocker triage, review verdicts, and reporting. Spend delegate tokens on everything else. (When the orchestrator itself is gpt-5.6, delegation still pays: workers keep the orchestrator context free for judgment.)

## Operating Role

- Act as the project steward and traffic controller. Stay at high effort; do not switch yourself to xhigh/max for orchestration work.
- Keep the project-specific context, account boundaries, branch/worktree facts, production risks, and human approval gates visible.
- Delegate focused work to worker agents. Never implement inline what a worker can do — exception: trivial edits where writing the task packet costs more than the edit.
- Route completed work to an independent reviewer before merge/deploy/readiness claims.
- Prefer one requirement per worker/PR. Split adjacent improvements into later tasks.
- Do not silently mutate production, accounts, CMS data, billing, external messaging, or secrets.

## Model Routing

Follow the "Picking the right models for workflows and subagents" section in `~/.claude/CLAUDE.md`. Summary for this skill's worker types:

| Worker type | Default model | Mechanism |
|---|---|---|
| Discovery / codebase analysis | gpt-5.6 (read-only) or Explore agent | Cockpit task (`--agent-type codex`), `codex exec -s read-only` for quick sweeps, or Agent tool |
| Implementation (well-spec'd) | gpt-5.6 | Cockpit task (`--agent-type codex`, `--worktree` for isolation); `codex:codex-rescue` inside workflows |
| Implementation (taste-sensitive: UI, copy, API design) | opus-4.8 or fable-5 | Agent/Workflow `model` parameter, or Cockpit task (`--agent-type claude`) |
| QA / computer use / browser verification | gpt-5.6 | Codex via Cockpit task — it is notably better and cheaper at computer use |
| Review of plans/implementations | fable-5 or opus-4.8 (+ optional gpt-5.6 second opinion) | Agent tool; `codex review` or delegator Code Reviewer for the extra perspective |
| Docs / reports / mechanical sweeps | sonnet-5 | Agent/Workflow `model` parameter |

Routing rules:

- Defaults, not limits. If a cheap model's output misses the bar, redo with a smarter model without asking. Judge the output, not the price tag.
- Never use Haiku.
- **Prefer Cockpit tasks for substantial delegation** (use the `/cockpit` skill): `cockpit task create --instruction "<packet>" --directory <repo> --agent-type codex` (or `claude`). Workers become visible, resumable, user-inspectable tasks — the right default for an orchestrator. Monitor with `cockpit task list` / `cockpit task get <id>`, answer worker questions with `cockpit task send <id> --text "..."`, isolate parallel implementation with `--worktree <branch>`.
- Every delegate starts blind to this session's context. The initial packet must be fully self-contained (context, scope, constraints, output format) — the templates below satisfy this. Within a live Cockpit task, follow-ups via `cockpit task send` retain the worker's history, so send only the delta. When re-delegating to a fresh worker (new task, `codex exec`, MCP), include the prior attempt and how it failed.
- Inside Workflow scripts, reach gpt-5.6 through the thin wrapper: `agent(prompt, {agentType: 'codex:codex-rescue'})`. The wrapper forwards to the Codex runtime and the orchestrator is not involved until the work is done. (Codex orchestrators skip the wrapper and spawn Codex workers directly via Cockpit tasks.)
- Reserve direct `codex exec` via Bash for quick, bounded, foreground checks not worth a Cockpit task.
- Claude workers get the `model` (and `effort`) parameters on Agent/Workflow calls. Use `effort: 'low'` for mechanical stages, higher tiers only for verify/judge stages. (Agent/Workflow tools exist only in Claude Code sessions; a Codex orchestrator reaches Claude workers via Cockpit tasks with `--agent-type claude` instead.)

## First Move: Read-Only Grounding

Before delegating or editing, build a short state packet from the actual current environment. Do this yourself (it shapes every downstream packet) unless the repo is large — then delegate the sweep to a discovery worker and verify its summary against `git status`/`gh` yourself.

Check what applies:

- Current cwd, repo root, branch, remote, latest commit, tracking status, and dirty files.
- Whether this session's cwd/branch differs from the user's referenced thread or repo.
- Project docs: `CLAUDE.md`, `AGENTS.md`, `README.md`, task/status docs, deploy docs, test docs.
- Open PRs, issues, CI status, deployment status, and recent commits (`gh pr list`, `gh run list`) when relevant.
- Product/runtime state: local server, public URL, admin/CMS URL, database/source status, or API health when relevant.
- Existing user/agent changes that must not be reverted.
- Explicit boundaries: accounts not to use, production data not to touch, no-notify/no-post rules, merge/deploy gates.

If the state packet reveals ambiguity that could damage work, pause delegation and ask for the missing constraint. Otherwise proceed with conservative assumptions and record them.

## Orchestration Loop

Track every worker as a task via TaskCreate/TaskUpdate so state survives context compaction and the user can see progress. Cockpit workers are already visible tasks in the Cockpit UI; mirror only the orchestration-level state.

Heartbeat mechanics:

- Background Agent/Workflow completions re-invoke you automatically — that is the primary wake signal. Do not poll for them.
- Cockpit worker tasks and other external waits (CI runs, deploys, long Codex background jobs) are not harness-tracked: use ScheduleWakeup or Monitor with a cadence matched to how fast that state changes. If the user gave no cadence: ~5 minutes for active multi-agent work, 15–30 minutes for slow CI/deploy waiting.

Each heartbeat:

1. Refresh repo/runtime state read-only.
2. Check each worker state (`cockpit task list` / `cockpit task get <id>`, TaskList, `gh run list`). For Cockpit workers in `waiting_confirmation`, read `waitingReason` and answer with `cockpit task send <id>` when `readyForNextPrompt` is true.
3. Classify blockers and decide whether to nudge, stop, reassign, or wait. Two failed attempts by a cheap worker → escalate the model, don't re-send the same packet.
4. If implementation is complete, send it to a reviewer worker (different model or at least a fresh session — never the implementer reviewing itself).
5. If review finds P0/P1, route back to implementation with the smallest corrective task, including the reviewer's findings verbatim in the new packet.
6. If review passes, prepare the next gate: PR, merge decision, deploy decision, post-deploy proof, or final report.
7. Send a concise status update to the user when material state changes.

## Resident Worker Pattern (Iterative Calibration Rounds)

Proven 2026-07-06 on Daifukucho M6 (e-NAVI connector: M6→M6b→M6c→M6d, one worker, four rounds, ~70 tests, all merged same day).

For one milestone-stream against a real external system (scraper calibration, API integration, migration against live data), keep **one Cockpit worker task alive and send successive rounds via `cockpit task send`** instead of creating fresh tasks:

- The worker accumulates codebase knowledge and prior design decisions; each follow-up needs only the delta — no re-onboarding packet, and the whole audit trail lives in one task.
- The round loop: worker implements with tests → orchestrator reviews the diff AND re-runs tests/tsc/lint independently → orchestrator runs the real-environment E2E the worker cannot reach (authenticated sessions, real data, local DBs) → reality produces new facts (schema drift, format surprises, misclassified errors) → orchestrator distills them into the next round's instruction → repeat until the loop closes end-to-end.
- Instruction format that worked per round: a `【実測で確定した新事実】` section carrying verbatim observed facts ("field X arrives as number, e.g. 202607", "cell value is '-' for N/A") + numbered MUST DO + MUST NOT. Label rounds (M6b, M6c…) so commits and reports stay traceable.
- Division of labor: the worker takes subsystem-sized changes with tests; the orchestrator makes tiny calibration edits directly (1-line schema/TDZ fixes) when the round-trip costs more than the change, and owns merges, E2E runs, and one-time DB backfills.
- Scope rule: same subsystem/milestone → same task; unrelated workstream → new task (parallel, own worktree).

Caveats learned the hard way:

- Worker unit tests cannot catch module-level CLI runtime bugs (e.g. TDZ on a `const` declared after top-level `parseAsync`) nor real-data drift — the orchestrator's live E2E is a mandatory gate before declaring a round done.
- `idle_timeout` re-reports from the same task are often duplicates of the last report — check the report seq before treating one as new work.
- A behavior fix that lands mid-stream (e.g. review auto-resolution) does not repair state created before it — plan a one-time backfill as an ops action, not worker code.

## Worker States

- `not_started`: Task is defined but not delegated.
- `running`: Worker is actively working.
- `waiting`: Worker waits on CI, auth, human input, external service, or a long-running process.
- `completed`: Worker returned artifacts, diff, PR, tests, or report.
- `reviewing`: Reviewer is checking completed work.
- `needs_fix`: Reviewer found issues that should return to implementation.
- `needs_human_judgment`: Scope, account, product, legal, data, or merge/deploy judgment needs the user.
- `failed`: Tooling, environment, permissions, or task design failed.
- `ready`: No blocking findings remain and the next user/merge/deploy action is clear.

## Task Packet Template

Send workers enough context to succeed without leaking unnecessary history. For Codex workers this packet is the entire shared state — include everything.

```text
You are a worker for <project>.

Goal:
- <one concrete requirement>

Required grounding:
- Start in/read <repo or cwd>.
- Confirm repo root, branch, dirty files, and relevant docs before editing.
- Treat existing user/agent changes as intentional; do not revert unrelated work.

Scope:
- In scope: <files/features/behavior>
- Out of scope: <adjacent cleanup, unrelated UI, schema, deploy, data edits>

Constraints:
- <account boundaries>
- <production/data/secret/no-post/no-merge rules>
- <one PR = one requirement if applicable>

Implementation:
- Make the smallest code/docs/config change that satisfies the goal.
- Follow existing project patterns.
- Run relevant focused tests/checks.

Return:
- Files changed
- Commands/tests run and results
- Screenshots/URLs/logs if relevant
- Remaining risks or questions
```

## Review Packet Template

Use a separate reviewer when the implementation can affect users, production, shared repos, billing/data, or a visible UI. Default reviewer: fable-5 or opus-4.8; add a gpt-5.6 pass (`codex review` or delegator Code Reviewer) as an extra independent perspective for high-stakes changes.

```text
You are a reviewer for <project>.

Review target:
- <branch/PR/diff/commit/worktree>

Requirement:
- <one concrete requirement>

Review stance:
- Findings first, ordered P0/P1/P2.
- Verify scope: no unrelated changes, no hidden behavior, no forbidden account/data/deploy actions.
- Verify tests/evidence match the requirement.
- For UI: check relevant breakpoints, interactions, layout, copy, and console errors.
- For backend/data: check validation, migration safety, auth, concurrency, and rollback/recovery.
- Do not edit, push, merge, deploy, post, or mutate production.

Return:
- Findings with file/line or evidence
- Whether P0/P1 remain
- Test gaps or residual risk
- Merge/deploy/readiness recommendation
```

## Common Worker Types

Choose only the workers needed for the current task. Model defaults per the routing table above.

- **Discovery worker**: read-only repo/product investigation and risk map. → gpt-5.6 read-only or Explore agent.
- **Implementation worker**: one focused code/content/config change. → gpt-5.6 for well-spec'd work; opus-4.8/fable-5 when taste matters.
- **QA worker**: browser/app/API regression testing against expected behavior. → gpt-5.6 (computer use strength).
- **Reviewer worker**: independent code/product review. → fable-5/opus-4.8, optional gpt-5.6 second opinion.
- **Deploy/proof worker**: post-merge CI/deploy/production verification without repeating unsafe operations. → gpt-5.6 read-only or sonnet-5.
- **Docs/report worker**: user story, runbook, status sheet, release notes, or orchestrator-style work report. → sonnet-5.

## Safety Gates

Stop and ask the user, or mark `needs_human_judgment`, when any of these appear:

- Wrong or unclear account, org, workspace, tenant, project, domain, or production target.
- Production data mutation, billing action, external posting, messaging, emailing, or user notification not explicitly authorized.
- Secrets, credentials, private data, or admin access are required but not already safely configured.
- Scope cannot remain one requirement or requires broad refactor.
- CI/deploy/migration/pipeline behavior is unclear or failing in a way that could damage production.
- Worker proposes merge, deploy, destructive Git operation, data restore, or rollback beyond authorization.
- Reviewer finds P0/P1 that have not been fixed and re-reviewed.

## Completion Report

Keep the final report short and operational:

- Current state and branch/PR/deploy URL if relevant.
- Workers used, their models, and their outcomes.
- What changed, if anything.
- Verification performed.
- Remaining P0/P1/P2 or explicit "no P0/P1 known".
- Required user decision, if any.
- Next suggested action only if it materially advances the project.

## Project-Specific Adapter

At the start of each orchestration, write a compact adapter for the project. Include only facts verified in the current session or explicitly supplied by the user.

```text
Project:
Repo/cwd:
Branch/remote:
Primary URLs:
Accounts/tenants allowed:
Accounts/tenants forbidden:
Production/data mutation policy:
Merge/deploy policy:
Known protected work:
Current requirement:
Heartbeat cadence:
Worker plan (type → model → mechanism):
Review gate:
Stop conditions:
```

Use the adapter in every worker/reviewer packet so separate sessions do not accidentally inherit the wrong branch, workspace, account, or product assumptions. For Codex workers, paste the adapter into the packet itself — they cannot see anything outside the prompt.
