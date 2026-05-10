# コンポーネント定義 — 町内匿名捜査室

## フロントエンド層

### InvestigatorApp
- **種別**: Next.js アプリケーション（独立デプロイ）
- **責務**:
  - 匿名捜査員向けの全画面を提供する
  - オンボーディング・事件一覧・事件詳細・投稿フォーム・通知一覧を含む
  - クライアント側での SHA-256 ハッシュ化・Exif 除去を実行する
  - アプリ内通知バッジ・通知一覧を管理する
- **インターフェース**: API Gateway エンドポイントを呼び出す

### AdminDashboard
- **種別**: Next.js アプリケーション（独立デプロイ）
- **責務**:
  - 行政担当者向けの確認候補地点一覧・詳細・レビュー更新画面を提供する
  - 環境変数ベースの固定パスワード認証を実装する
  - デモトリガーボタンを提供する（デモ時のみ使用）
- **インターフェース**: Admin API・Demo API エンドポイントを呼び出す

---

## API 層（Lambda 関数）

### EventFunction
- **種別**: AWS Lambda（TypeScript）
- **責務**:
  - 事件の一覧取得・詳細取得・作成を処理する
  - MVP では #T-01 のモックデータを返す
  - 将来的な Bedrock 動的生成への切り替えポイントを保持する
- **エンドポイント**: `GET /events`, `GET /events/{event_id}`, `POST /events`

### ReportFunction
- **種別**: AWS Lambda（TypeScript）
- **責務**:
  - 捜査報告の下書き作成（`POST /reports`）と確定（`POST /reports/{id}/finalize`）を処理する
  - 写真あり: draft 保存 + Presigned URL 返却 → クライアントが S3 アップロード → finalize で submitted に更新しワークフロー起動
  - 写真なし: submitted で即時保存しワークフロー起動
  - `moderation_status: pending`・`visibility_status: public` で初期保存する
- **エンドポイント**: `POST /reports`, `POST /reports/{report_id}/finalize`

### NotificationFunction
- **種別**: AWS Lambda（TypeScript）
- **責務**:
  - ユーザーの未読通知一覧を返す
  - 通知の既読更新を処理する
- **エンドポイント**: `GET /notifications`

### AdminFunction
- **種別**: AWS Lambda（TypeScript）
- **責務**:
  - 確認候補地点の一覧・詳細を返す
  - 行政担当者のレビュー状態更新を処理する
  - ReviewLog への記録を行う
  - **`Authorization: Bearer {ADMIN_TOKEN}` ヘッダーを検証し、不一致は 401 を返す**
- **エンドポイント**: `GET /admin/locations`, `GET /admin/locations/{location_id}`, `PUT /admin/locations/{location_id}/review`

### DemoFunction
- **種別**: AWS Lambda（TypeScript）、タイムアウト: 30秒
- **責務**:
  - デモ用即時処理トリガーを受け付ける
  - Step Functions Express Workflow を `startSyncExecution` で同期実行する
  - 10秒以内の完了を保証する
  - **`Authorization: Bearer {DEMO_TOKEN}` ヘッダーを検証し、不一致は 401 を返す**
- **エンドポイント**: `POST /demo/trigger`

---

## ワークフロー層

### ReportProcessingWorkflow
- **種別**: AWS Step Functions Express Workflow
- **責務**:
  - 投稿受付後の一連の処理を順次実行する
  - 通常運用: 非同期起動（投稿後にバックグラウンドで実行）
  - デモ時: DemoFunction から `startSyncExecution` で同期実行
- **ステップ順序**:
  1. SummarizeReport（BedrockFunction 呼び出し）
  2. UpdateAISummary（DynamoDB 保存）
  3. UpdateLocationObservation（地点統合・優先度再計算・DynamoDB 保存）
  4. EvaluateUserProfile（role_state 評価・更新）
  5. GenerateNotification（BedrockFunction 呼び出し）
  6. SaveNotification（DynamoDB 保存）

---

## AI 層

### BedrockFunction
- **種別**: AWS Lambda（TypeScript）
- **責務**:
  - Amazon Bedrock の呼び出しを一元管理する
  - `action` パラメータで処理を切り替える（`summarize` / `notify`）
  - `prompts/` ディレクトリからプロンプトファイルを読み込む（N-07 準拠）
  - AI 入出力・使用モデル ID・実行時刻を CloudWatch Logs に記録する（N-04 準拠）
- **アクション**:
  - `summarize`: 投稿の要約・危険要因分類・`has_high_signal` 判定・行政向け摘要生成
  - `notify`: 通知文の生成（`role_state` に応じたトーン調整）

---

## データ層

### EventRepository
- **種別**: DynamoDB アクセスコンポーネント
- **責務**: Event テーブルの CRUD 操作

### ReportRepository
- **種別**: DynamoDB アクセスコンポーネント
- **責務**: Report テーブルの CRUD 操作・`visibility_status`・`moderation_status` 管理

### LocationObservationRepository
- **種別**: DynamoDB アクセスコンポーネント
- **責務**: LocationObservation テーブルの読み書き・`location_key` による検索・`priority_class` 更新

### AISummaryRepository
- **種別**: DynamoDB アクセスコンポーネント
- **責務**: AISummary テーブルの CRUD 操作

### NotificationRepository
- **種別**: DynamoDB アクセスコンポーネント
- **責務**: Notification テーブルの CRUD 操作・未読管理

### UserProfileRepository
- **種別**: DynamoDB アクセスコンポーネント
- **責務**: UserProfile テーブルの読み書き・`role_state` 遷移管理・`valid_observation_count` 更新

### ReviewLogRepository
- **種別**: DynamoDB アクセスコンポーネント
- **責務**: ReviewLog テーブルへの追記（監査ログ）
