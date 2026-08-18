---
name: chief-work-report
description: Save completed work as a concise project or team-vault Markdown report and share a short summary with the designated Chief task. Use when asked to preserve a work record or report completed work upstream.
---

# Chief Work Report

Create two outputs: one factual Markdown report and one short Chief summary. Do not invent status or include unrelated cleanup.

## Workflow

1. Resolve the Chief task and report destination from the request or project instructions. Do not guess either one; ask one short question if needed.
2. Read applicable `AGENTS.md` files and inspect repository status. Preserve unrelated changes and touch only the report file when possible.
3. Write a date-prefixed report with verified facts, evidence, decisions, current status, and next actions. Record rationale, not private reasoning.
4. Send the Chief a compact summary containing the absolute report path, completion state, remaining work, and any decision needed. Do not paste the whole report.
5. Read back the report and verify its repository status. Tell the user what was written and whether the Chief message was sent.

## Report Shape

Use only the sections the task needs:

```markdown
# YYYY-MM-DD Task Name 作業報告

## 概要

1-3 sentences.

## 実施したこと

| 項目 | 実施内容 | 補足 |
|---|---|---|

## 根拠 / 参照

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

Skip empty sections. Tiny tasks need only `概要`, `実施したこと`, and `次`.

## Guardrails

- Do not commit, stage, or contact external channels unless explicitly requested.
- Do not expose secrets, credentials, or unnecessary personal data.
- State live external changes exactly; never turn an unverified claim into completion.
- Preserve project-specific scope and approval gates.
