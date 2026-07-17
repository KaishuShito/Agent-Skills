---
name: chief-work-report
description: Create a concise Markdown work report in the appropriate Obsidian or team-vault folder and share a summary to Kai's Chief/Chief秘書 Codex thread. Use when Kai asks to report completed work to the pinned Chief, summarize an operational task into md, hand off status to a Chief thread, or preserve a team-facing AGIラボ work log in Obsidian before closing a task.
---

# Chief Work Report

## Overview

Turn a completed work session into two durable outputs: a short Markdown report saved in the right Obsidian/team-vault location, and a concise message sent to the Chief/Chief秘書 Codex thread.

Keep the report factual and operational. Do not invent status, over-explain, or fold in unrelated cleanup.

## Workflow

1. Identify the reporting target.
   - If Kai gives a thread id, use it.
   - If Kai says "pinned Chief" or "チーフ", use Codex thread tools when available: first discover them with `tool_search`, then confirm the target with `read_thread`.
   - If no thread can be identified safely, ask one short question.

2. Identify the vault destination.
   - For AGIラボ team work, default to `/Users/kai/repo/agi-tools/agilab-team-vault`.
   - `~/Documents/Obsidian/01_Projects/AGIラボ-Team` is a compatibility symlink; do not use it as the entry point for new work.
   - Save under the most relevant project docs folder, usually `Projects/AGI-lab-community/Docs/` for Discord/community operations.
   - Use lowercase date-prefixed filenames such as `2026-06-25-feedback-loop-discord-report.md`.
   - If the repo has an `AGENTS.md`, follow it before writing.

3. Inspect before writing.
   - Run `git status --short` in the vault/repo.
   - Treat existing changes as user-owned. Do not revert, reformat, stage, or clean unrelated files.
   - If adding one new report file is enough, touch only that file.

4. Write the Markdown report.
   - Use a Japanese H1 with the date and task name.
   - Prefer tables for facts, IDs, links, decisions, and next actions.
   - Include enough evidence for future agents: local file paths, Discord/channel/message IDs, PR numbers, URLs, or thread IDs when relevant.
   - Keep internal reasoning out. Record decisions and rationale, not private scratchwork.
   - End with current status and clear next-task candidates.

5. Share to the Chief thread.
   - Send a concise summary with the saved absolute file path.
   - Mention what is complete, what remains, and what should become a separate task.
   - Do not paste the whole report unless Kai asks.
   - Do not post to Discord, Slack, Gmail, or other external channels unless explicitly requested.

6. Verify and report back.
   - Read back the created file or enough of it to catch obvious mistakes.
   - Check `git status --short -- <report-file>` to confirm the intended file is present.
   - Tell Kai the file path and that the Chief message was sent.

## Report Shape

Use this structure unless the task calls for something smaller:

```markdown
# YYYY-MM-DD Task Name 作業報告

## 概要

1-3 sentences.

## 実施したこと

| 項目 | 実施内容 | 補足 |
|---|---|---|

## 主なID / 参照

| 種別 | 名前 | ID / URL / Path |
|---|---|---|

## 判断メモ

| 論点 | 判断 | 理由 |
|---|---|---|

## 残タスク候補

| 優先 | タスク | メモ |
|---|---|---|

## 現時点の結論

Close/continue recommendation.
```

Skip empty sections. For tiny tasks, use only `概要`, `実施したこと`, and `次`.

## Chief Message Shape

Use a compact handoff:

```text
<task name> の作業報告です。

Team Vaultに報告Markdownを保存しました:
<absolute path>

要点:
- ...
- ...
- ...

次タスク候補:
1. ...
2. ...
```

## Guardrails

- Do not commit or stage unless Kai explicitly asks.
- Do not expose secrets, tokens, private credentials, or raw personal data.
- Use absolute paths in user-facing responses.
- Preserve `1 PR = 1 requirement` for AGIラボ shared/team work.
- When the task involved live external changes, state exactly what was changed and where.
