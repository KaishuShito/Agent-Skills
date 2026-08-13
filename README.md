# Agent-Skills

AGI Cockpit のマスターエージェント（統括Chief）向けエージェントスキルのリポジトリ。

## 更新情報

### 2026-08-13 — Cockpit v4.49.0 対応

v4.49.0 でタスク間の自動通知（report-back）が廃止され、タスク管理が「`task create` → `task wait`（非同期）」と「`task run`（同期）」の2方式に整理されたことに合わせて、全スキルを更新しました。

- **子タスクの完了報告をルール化**: 完了が親に自動通知されなくなったため、「ワーカーは作業が終わったら親のLeadに1回だけ報告メッセージを送って終わる」規約を起票テンプレートに組み込み（`WORKER_DONE` 形式）
- **監視はポーリングに一本化**: `task wait --since` / `task run` の使い分けを明記。旧 `--report-back` オプション前提の記述を全削除
- **Leadタスクのトップレベル起票**: エージェントからでも親なし（子タスクにならない）でタスクを作る手順を追記（v4.49.0時点の回避策）

## 収録スキル

| スキル | 概要 |
|---|---|
| [cockpit-master-chief](cockpit-master-chief/SKILL.md) | AGI Cockpit の統括Chiefセッションとしてワーカータスク群を運用する |
| [project-maintainer-orchestrator](project-maintainer-orchestrator/SKILL.md) | プロジェクトの親メンテナー/オーケストレーターとしてワーカー・レビュアーを統括する |
| [chief-handover](chief-handover/SKILL.md) | 新Chiefセッションへの引き継ぎ（レーン再同期・ポーリング監視の継承） |
| [chief-work-report](chief-work-report/SKILL.md) | 完了した作業をObsidian/team-vaultの報告Markdownに残し、Chiefスレッドへ要約を共有する |

## ローカル運用

実体はこのリポジトリに置き、利用するエージェントのスキルディレクトリからシムリンクで参照する:

```sh
ln -s /path/to/Agent-Skills/<skill-name> ~/.agents/skills/<skill-name>
```

スキルディレクトリは利用環境に合わせて読み替える。
