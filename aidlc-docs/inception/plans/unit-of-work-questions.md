# Unit of Work 質問ファイル — 町内匿名捜査室

Application Design の成果物を踏まえ、Unit 分解の最終確認をします。
各質問の `[Answer]:` タグに回答を記入してください。

---

## Question 1: Unit 5 と Unit 6 の境界

BedrockFunction（要約・分類）と LocationObservation（地点統合）は
ReportProcessingWorkflow の中で連続して実行されます。
Unit として分けますか？

A) 分ける（Unit 5 = AI 要約・分類、Unit 6 = 地点統合・優先度計算）
B) まとめる（Unit 5+6 = 投稿整理 AI + 地点統合を1 Unit にする）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 2: Unit 7 と Unit 8 の実装タイミング

通知エンジン（Unit 7）と進化予兆エンジン（Unit 8）は密接に関連しています。
Phase 1 での扱いをどうしますか？

A) Unit 7（通知）のみ Phase 1 に含める。Unit 8（進化予兆）は Phase 2
B) Unit 7 と Unit 8 を Phase 1 でまとめて実装する（role_state 評価も含める）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 3: ReportProcessingWorkflow の Unit 帰属

Step Functions の ReportProcessingWorkflow はどの Unit に帰属させますか？

A) Unit 11（デモ用即時処理トリガー）に含める
B) Unit 5（投稿整理 AI）に含める
C) 独立した Unit として扱う（Unit 4.5 的な位置づけ）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 4: Phase 1 の実装順序の確認

execution-plan.md で提案した実装順序（Unit 1 → 3 → 4 → 5 → 6 → 7 → 9 → 11）を
そのまま採用しますか？

A) そのまま採用する
B) 順序を変更したい（変更後の順序を [Answer] 欄に記載してください）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

すべての質問に回答したら「回答完了」とお知らせください。
