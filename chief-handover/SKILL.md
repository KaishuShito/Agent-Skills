---
name: chief-handover
description: >
  Transfer an AGI Cockpit Master Chief session: create the successor as a
  top-level task, load a handoff packet, rediscover live lanes, continue polling
  their persistent report logs, and produce a consolidated decision queue.
  Use for Chief replacement or lane resynchronization, not ordinary delegation.
---

# Chief Handover

Cockpit v4.49.0 removed report-back and `task send --parent-task-id`. A handover no longer reparents lanes or changes a report destination. The successor discovers the existing task IDs and polls each task's persistent report log.

## Outgoing Chief

1. Write a handoff packet in a project-appropriate private location. Include:
   - outgoing Chief task ID and successor instructions
   - current priorities and approval gates
   - every live Lead/task ID, status, and latest processed report sequence
   - artifact paths, decisions, blockers, deadlines, and next actions
   - a note that lane state is a snapshot to be refreshed by the successor
2. Rename and unpin the outgoing Chief if that matches the local operating convention.
3. Create the successor as a top-level task:

   ```bash
   env -u AGI_COCKPIT_TASK_ID -u AGI_COCKPIT_CONTEXT_TASK_ID -u AGI_COCKPIT_CONTEXT_LOOKUP \
     cockpit task create --directory master --agent-type <agent-type> --name "Chief" \
     --instruction "Read <handoff-packet>, run chief-handover, refresh every live lane, and present a consolidated brief."
   cockpit task pin <new-id>
   ```

   This is a v4.49.0 workaround. An official top-level creation flag is proposed in [tempi-tech/AGICockpit#301](https://github.com/tempi-tech/AGICockpit/issues/301).
4. Stop accepting new work after the successor has been created. Do not complete or remove the outgoing task without explicit authorization.

## Incoming Chief

### 1. Load context

Read the complete handoff packet and resolve the current task ID:

```bash
cockpit task current
```

### 2. Rediscover live lanes

Use the packet's task IDs as the primary inventory, then compare them with:

```bash
cockpit task list --all
```

Top-level Leads may have no `parentMasterId`, so do not rely on parent-based filtering. Exclude completed lanes that have no remaining follow-up. If the old Chief ID or the intended lane set is ambiguous, ask the human before taking over.

### 3. Notify without reparents

When a lane needs the new operating context, send a normal message:

```bash
cockpit task send <id> --text "The Master session has changed. Continue the existing scope and approval rules. Summarize current state, blockers, and decisions needed in three lines or fewer."
```

This message does not alter task hierarchy or report routing. Do not use the removed `--parent-task-id` option.

### 4. Resume polling

For each lane, read its current state and sequence:

```bash
cockpit task get <id> --turns 2 --max-lines 100
cockpit task wait <id> --since <last-processed-seq> --timeout 110
```

Treat `{"timeout": true}` as a normal timeout and poll again. Advance the saved sequence only after processing a report. Use `task get` when recent conversation context is needed.

### 5. Consolidate

Report lanes, current state, blockers, and human decisions in one compact table. Include the next polling cursor for each active lane in the durable handoff state.

## Guardrails

- There is no report-back destination to transfer in v4.49.0.
- Do not assume UI nesting identifies every Lead.
- Do not send directly to a Lead's workers. Reparenting no longer occurs, but bypassing the Lead still breaks the command chain.
- Do not acknowledge `WORKER_DONE` with another send; inspect the worker log and report upstream once.
- Do not complete or remove tasks without explicit authorization.
