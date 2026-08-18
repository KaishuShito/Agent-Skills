# Agent-Skills

AGI Cockpit の Chief / Lead 運用を補助する、薄いエージェントスキル集。

Cockpit Master の `AGENTS.md` と現在の CLI ヘルプを実行時仕様の正本とします。このリポジトリは、変化しやすいコマンド一覧や状態遷移を複製せず、役割分担・委譲・レビュー・引き継ぎ・報告の判断だけを扱います。

## 収録スキル

| スキル | 概要 |
|---|---|
| [cockpit-ops](cockpit-ops/SKILL.md) | Chief / Lead / Worker の責任分界、委譲、レビュー、Chief交代を統合する |
| [chief-work-report](chief-work-report/SKILL.md) | 完了した作業をObsidian/team-vaultの報告Markdownに残し、Chiefタスクへ要約を共有する |

旧 `cockpit-master-chief`、`project-maintainer-orchestrator`、`chief-handover` は `cockpit-ops` に統合しました。既存のシムリンクを使っている場合は、旧3件を外して `cockpit-ops` へのリンクを作り直してください。

## ローカル運用

実体はこのリポジトリに置き、利用するエージェントのスキルディレクトリからシムリンクで参照する:

```sh
ln -s /path/to/Agent-Skills/<skill-name> ~/.agents/skills/<skill-name>
```

スキルディレクトリは利用環境に合わせて読み替える。
