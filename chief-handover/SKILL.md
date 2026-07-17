---
name: chief-handover
description: >
  Take over as the new Chief session in AGI Cockpit Master: boot from the handoff pack,
  re-link child-task report-back targets from the previous Chief session to this session,
  resync every live lane, and report a consolidated "awaiting Kai's decision" list.
  Use when Kai says 「Chief交代」「taskId XXX を引き継いでChiefになってください」「朝のChief起動」
  「レーンを同期して」, when a new Master/Chief session starts while worker tasks still report
  to an old Chief task, or as a morning routine to resync all lanes. Don't use for creating
  new worker tasks or for handoffs outside Cockpit.
---

# Chief Handover

新しいChiefセッションが旧Chiefのレーン群を引き継ぎ、報告先を自分に付け替え、全レーンの現在地と「Kaiの判断待ち」を1枚にして報告する。

## 送り出し側の手順（現Chiefがコンテキスト残量低下などで自分から交代する場合）

現Chiefのコンテキストが逼迫してきたら（目安: 残30%以下、または大きな作業を終えた区切り）、この手順で後継に引き継ぐ。

1. **引き継ぎパックを書く**: `~/.agi-tools/data/cockpit/master/handoffs/YYYY-MM-DD-chief-handoff*.md`。構成は §0起動手順（このスキルの実行指示＋旧ChiefID）／§1いまの優先事項／§2レーン表（ID・状態・待ち）／§3確立した運用・今日学んだ罠／§4直近の期日／§5ポインタ。レーン状態は「交代時点の知識」と明記（正確な同期は後継がやる）
2. **自分をリネーム**: `./cockpit task rename $ME --name "Chief migrated"`
3. **自分のピンを外す**: `./cockpit task unpin $ME`
4. **新Chiefを起票**（masterディレクトリ・claude）:
   ```bash
   ./cockpit task create --directory master --agent-type claude --name "Chief" \
     --instruction "あなたはAGI Cockpitの新しい統括Chief（Fable 5想定。起動時にまず自分のモデルを1行報告。違えば明記して続行）。①引き継ぎパック <パック絶対パス> を全部読む ②パック§0に従い cockpit-master-chiefスキル→/chief-handoverスキルを実行（旧Chief = $ME）。レーンの報告先を自分に付け替え＋全レーン同期 ③Kaiに現況ブリーフを提示。以後はKaiとの直接対話で進める。"
   ```
5. **新Chiefをピン留め**: `./cockpit task pin <新ID>`
   - 注意: 新Chiefは作成者（旧Chief）の子としてぶら下がるため、**ピン留めしてもサイドバーのピン欄に出ない**（親持ちタスクはツリー下に畳まれる仕様/バグ）。KaiにUIの三点メニュー「**子タスクから外す**」で親を切ってもらうと表示される
6. 以後、旧Chief（Chief migrated）は新規作業を受けない。残ヒントは無視し、後継からのhandover連絡（付け替えsend）に短く応答するだけ

## 手順

### 1. 文脈の起動（新セッションの初回のみ）

1. 恒久メモリの `project-fable-strategy-week-handoff` が指す引き継ぎパック（`~/Documents/Obsidian/01_Projects/Fable-AGI-Lab-Strategy/16_handoff-pack.md`）を読む。§0の起動手順に従う
2. `./cockpit task current` で自分のタスクIDを確認（以下 `$ME`）

すでにChiefとして稼働中の朝の再同期なら、この節は省略して2へ。

### 2. 旧Chiefと子タスクの特定

- 旧Chief IDはKaiの指示から取る。指示がなければ `./cockpit task list --all` で name が `Chief` 系の直近タスクを候補としてKaiに確認
- 子タスクの列挙:

```bash
./cockpit task list --all 2>/dev/null | python3 -c "
import json,sys
OLD='<旧ChiefID>'
for t in json.load(sys.stdin)['data']:
    if t.get('parentMasterId')==OLD:
        print(t['id'],'|',t['status'],'|',t.get('needsResume'),'|',t['name'][:50])
"
```

- 付け替え対象は**生きているレーンだけ**。completed と役目を終えたタスク（成果物納品済みで次の仕事がないもの）は除外

### 3. 報告先の付け替え

Cockpitに親変更の専用APIはないが、**send --parent-task-id すると相手のreport-back先と `parentMasterId`（UIツリー表示）が送信元へ付け替わる**（作成時固定ではない。2026-07-09実測）。自分から1回sendすれば以後の報告は自分に届く。

各レーンへ逐次（2秒間隔）送信:

```bash
./cockpit task send <id> --parent-task-id $ME --text "【Chief交代のお知らせ】旧Chief（<旧ID>）は役目を終え、新Chief（タスクID $ME）に引き継がれました。以後の報告はこの送信により自動的に新Chiefへ届きます。作業内容・これまでの指示・運用ルールに変更はありません。確認のため、①現在の状態 ②Kai/Chief側の判断・入力待ちになっている点、を3行以内で返してください。"
```

- `{"ok":false,"error":"タスクが見つかりません"}` = プロセス停止（`needsResume: true`）。`./cockpit task resume <id>` → 8秒待って再send

### 4. 返答の回収

send直後のレポートは自分の送信文のechoになることがあり、`task wait --since` が噛み合わないことがある。確実な方法:

```bash
./cockpit task get <id> --turns 2 --max-lines 100
```

で assistant の最後の発言を読む。返答生成に1〜3分かかるので、全レーンに送り終えてから回収する。

### 5. 統合報告

レーン×状態×判断待ちのテーブルでKaiに提示する。判断待ちが共通のボトルネック（例: Yotaさんとの接点、Kaiの承認1件）に集約されているなら、それを解く一手まで提案して締める。

## 注意

- `parentMasterId` は send のたびに送信元へ付け替わる（ドキュメントの「作成時固定」は誤り・2026-07-09実測）。**子に照会するときは「回答は task send で返さず、ターン完了報告で返す」を必ず明記** — 子からsendさせるとChief側の親・報告先が子に付け替わり、ヒントの無限往復ループが起きる（詳細: メモリ reference_cockpit_reportback_loop）
- KaiがCockpit UIから子タスクへ直接送信すると、そのタスクの報告先はユーザーにリセットされる（仕様）。次に自分がsendすれば戻る
- タスクの complete / remove はKaiの明示指示なしにしない
