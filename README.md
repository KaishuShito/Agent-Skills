# Agent-Skills

AGI Cockpit のマスターエージェント（統括Chief）向けエージェントスキルのリポジトリ。

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
