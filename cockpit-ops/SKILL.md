---
name: cockpit-ops
description: Coordinate AGI Cockpit work as a Master Chief or project Lead, including scoped delegation, independent review, lane recovery, and Chief handover. Use when asked to orchestrate projects, supervise agents, take over a Chief session, or reconcile active lanes.
---

# Cockpit Ops

Use the Cockpit Master `AGENTS.md` and the installed CLI help as the authority for commands, statuses, and runtime behavior. This skill supplies workflow decisions only. Never replace or rewrite the Master `AGENTS.md` from this skill.

## Choose the role

| Role | Owns | Boundary |
|---|---|---|
| Master Chief | Priorities, Lead creation, cross-project gates, consolidated decisions | Delegates routine implementation |
| Project Lead | One workstream, worker packets, review, readiness | Does not create sibling Leads or bypass approval gates |
| Worker | One concrete requirement and its evidence | Does not broaden scope or self-approve |

Keep one command path: the Master directs Leads; Leads direct their workers. Use a one-shot worker instead of a persistent Lead when the work has one bounded outcome.

## Ground before delegating

Confirm the exact repository or workspace, instructions, branch and dirty state, current task state, and relevant account or production target. Preserve user-owned changes. Carry forward explicit no-push, no-merge, no-deploy, no-send, data, billing, and destructive-action gates.

## Delegate a complete packet

Every new task starts blind. Include:

```text
Role and goal: <one concrete requirement>
Start here: <repository or workspace>
Grounding: <instructions and current facts to verify>
In scope: <files, behavior, or workstream>
Out of scope: <adjacent work>
Gates: <account, production, publish, merge, or send limits>
Verify: <focused checks and evidence>
Return: <changes, results, residual risks, decision needed>
```

Use the current Master instructions to choose synchronous or asynchronous execution and to inspect waiting tasks. Do not add a second notification protocol: persistent task reports and the Master's documented wait flow are the coordination mechanism.

## Review and report

- Inspect returned artifacts and checks before relaying a completion claim.
- Use an independent reviewer for user-facing, production, shared-data, or shared-repository changes.
- Ask reviewers for findings first, ordered by severity, with file/line or source evidence.
- Return unresolved material findings to the responsible Lead or worker.
- Report facts, unknowns, required human decisions, and the next authorized action separately.

## Reconcile or hand over a Chief

Treat saved state as a snapshot, not live truth.

1. Record priorities, approval gates, active task IDs, latest processed report cursors, artifacts, decisions, blockers, and next actions in a private handoff packet.
2. The successor reads the packet, resolves its own identity, and refreshes every listed task using the Master `AGENTS.md` procedures.
3. Compare the packet with the current task inventory. Do not infer ownership from UI nesting alone.
4. Notify a Lead only when its operating context changed; do not redirect its workers.
5. Produce one compact lane table with state, blocker, decision needed, and next cursor.

Do not complete, remove, publish, deploy, message externally, bill, or mutate production unless the user explicitly authorizes that action.
