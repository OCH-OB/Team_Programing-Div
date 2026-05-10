# Application Design 質問ファイル — 町内匿名捜査室

requirements.md・stories.md を踏まえ、コンポーネント設計の判断に必要な事項を確認します。
各質問の `[Answer]:` タグに回答を記入してください。

---

## Question 1: フロントエンドのアプリ構成

Next.js アプリを何個のアプリとして構成しますか？

A) 1アプリ（匿名捜査員・行政ダッシュボードを同一 Next.js アプリ内のルートで分ける）
B) 2アプリ（匿名捜査員用と行政ダッシュボード用を別々の Next.js アプリとして分ける）
C) 1アプリ + サブパス（`/` が捜査員向け、`/admin` が行政向け・同一デプロイ）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 2: API の構成単位

バックエンド API をどう構成しますか？

A) 機能別に Lambda 関数を分ける（EventFunction・ReportFunction・NotificationFunction 等）
B) 1つの Lambda 関数でルーティング（モノリシック Lambda + API Gateway）
C) ドメイン別に Lambda 関数を分ける（InvestigatorFunction・AdminFunction・WorkflowFunction）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question 3: Step Functions ワークフローの粒度

投稿処理の Step Functions ワークフローをどう設計しますか？

A) 1つのワークフロー（投稿整理 → 地点統合 → 通知生成 → UserProfile 更新を1ワークフローで）
B) 2つのワークフロー（投稿整理ワークフロー と 通知・進化ワークフローを分ける）
C) 3つのワークフロー（投稿整理 / 地点統合 / 通知生成をそれぞれ独立させる）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question 4: Bedrock 呼び出しの責務配置

Amazon Bedrock の呼び出しをどのコンポーネントが担いますか？

A) Step Functions の各ステートから直接 Bedrock を呼ぶ（Lambda なし）
B) 専用の Lambda 関数（SummarizationFunction・NotificationFunction）経由で呼ぶ
C) Step Functions から Lambda を呼び、Lambda が Bedrock を呼ぶ（B と同じだが Lambda を汎用化）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 5: UserProfile の更新タイミング

`role_state`（`anonymous` → `trusted_candidate` → `special_candidate`）の評価・更新をいつ行いますか？

A) 投稿整理ワークフローの最後のステップで毎回評価・更新する
B) 通知生成の前に評価し、遷移条件を満たした場合のみ更新する
C) 独立したバッチ処理（1日1回）で評価・更新する
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question 6: 優先度分類（即応候補 / 中期確認候補）の計算タイミング

LocationObservation の優先度分類をいつ計算しますか？

A) 地点統合処理のたびに再計算して LocationObservation に保存する
B) 行政ダッシュボードの表示時にリアルタイムで計算する（DB には保存しない）
C) 投稿が追加されるたびに非同期で再計算する
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question 7: 写真の S3 保存構造

S3 バケットのオブジェクトキー構造をどうしますか？

A) `reports/{report_id}/original.jpg`（シンプル・report_id で管理）
B) `reports/{event_id}/{report_id}/original.jpg`（event_id でグループ化）
C) `reports/{year}/{month}/{report_id}/original.jpg`（日付でパーティション）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question 8: デモトリガーの実装方式

`POST /demo/trigger` の実装をどうしますか？

A) Step Functions Express Workflow を同期実行（`startSyncExecution`）する専用 Lambda
B) 通常の非同期ワークフローを起動し、ポーリングで完了を待つ
C) 通常の投稿処理ワークフローをそのまま同期モードで呼ぶ（専用ワークフローなし）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

すべての質問に回答したら「回答完了」とお知らせください。
