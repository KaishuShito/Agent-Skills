# Agent-Skills

AGI Cockpit のマスターエージェント（統括Chief）向けエージェントスキルのリポジトリ。

## 収録スキル

| スキル | 概要 |
|---|---|
| [cockpit-master-chief](cockpit-master-chief/SKILL.md) | AGI Cockpit の統括Chiefセッションとしてワーカータスク群を運用する |
| [project-maintainer-orchestrator](project-maintainer-orchestrator/SKILL.md) | プロジェクトの親メンテナー/オーケストレーターとしてワーカー・レビュアーを統括する |
| [chief-handover](chief-handover/SKILL.md) | 新Chiefセッションへの引き継ぎ（レーン再同期・report-back先の付け替え） |

## ローカル運用

実体はこのリポジトリに置き、`~/.claude/skills/` からシムリンクで参照する:

```sh
ln -s ~/Develop/Agent-Skills/<skill-name> ~/.claude/skills/<skill-name>
```

Claude Code からは従来どおり `~/.claude/skills/` 配下のスキルとして見える。
