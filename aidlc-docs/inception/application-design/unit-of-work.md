# Unit of Work 定義 — 町内匿名捜査室

## Unit 分解方針

- **ペルソナ・ストーリー・コンポーネント**の3軸で Unit を定義する
- **Phase 1**: MVP デモに必須の Unit（7 Unit）
- **Phase 2**: 将来拡張の Unit（2 Unit）
- Unit 5+6 を統合、Unit 7+8 を統合した結果、Phase 1 は7 Unit（Unit 1,3,4,5,7,9,11）に整理

---

## Phase 1 Units（MVP デモ必須）

### Unit 1: 事件化エンジン

**目的**: 事件データを提供し、将来の動的生成への拡張ポイントを確保する

**責務**:
- 事件 #T-01 のモックデータを DynamoDB に投入・管理する
- `GET /events`・`GET /events/{event_id}` を提供する
- 将来の Bedrock 動的生成への切り替えポイント（`POST /events`）を保持する

**主要コンポーネント**:
- EventFunction（Lambda）
- EventRepository（DynamoDB）
- `data/mock-events/` モックデータ

**コード配置**: `functions/event/`・`lib/events/`・`data/mock-events/`

**完了条件**:
- `GET /events` が #T-01 を返す
- `GET /events/{event_id}` が事件詳細を返す
- DynamoDB の Event テーブルスキーマが確定している

---

### Unit 3: 匿名捜査員向けフロントエンド

**目的**: 匿名捜査員が事件を発見・確認・投稿・通知受取できる Web アプリを提供する

**責務**:
- オンボーディング画面（1〜2画面・安全ガイドライン含む）
- 事件一覧画面（新着順・16px 以上フォント）
- 事件詳細画面（選択式回答集計・投稿ボタン）
- 投稿フォーム（写真または一文・推理オプション・3タップ以内）
- クライアント側 SHA-256 ハッシュ化・Exif 除去
- 通知一覧画面（未読バッジ・通知タップで事件詳細へ）

**主要コンポーネント**:
- InvestigatorApp（Next.js）
- `app/investigator/`・`components/investigator/`

**コード配置**: `app/investigator/`・`components/investigator/`・`components/shared/`

**完了条件**:
- オンボーディング → 事件一覧 → 事件詳細 → 投稿 → 通知の画面遷移が動作する
- 3タップ以内で投稿完了できる
- 16px 以上のフォントサイズ
- Exif 除去ロジックが動作する

---

### Unit 4: 投稿受付・保存基盤

**目的**: 捜査報告を安全に受け付け・保存し、処理ワークフローを起動する

**責務**:
- `POST /reports`（draft 作成）: テキスト・選択肢を保存し `report_id` と `upload_url`（写真あり時）を返す
- `POST /reports/{id}/finalize`: S3 アップロード完了を確認し `status: submitted` に更新してワークフローを起動する
- テキストのみ投稿は `POST /reports` で即時 `status: submitted` に保存しワークフローを起動する
- `moderation_status: pending`・`visibility_status: public` で初期保存する
- 投稿受付（draft 作成）を3秒以内に完了する
- **ワークフローは `status: submitted` の Report に対してのみ起動する**

**主要コンポーネント**:
- ReportFunction（Lambda）
- ReportRepository（DynamoDB）
- S3（`reports/{report_id}/original.jpg`）

**コード配置**: `functions/report/`・`lib/reports/`

**完了条件**:
- `POST /reports` が3秒以内に `report_id`（と写真あり時は `upload_url`）を返す
- `POST /reports/{id}/finalize` が S3 存在確認後にワークフローを起動する
- テキストのみ投稿が即時 submitted になる
- DynamoDB に Report が正しいステータスで保存される

---

### Unit 5: 投稿整理AI + 地点統合（ReportProcessingWorkflow）

**目的**: 投稿を AI で整理し、地点単位に統合して行政が使えるデータを生成する

**責務**:
- ReportProcessingWorkflow（Step Functions Express Workflow）を定義・実装する
- BedrockFunction（`summarize` アクション）で投稿を要約・危険要因分類する
- AISummary を DynamoDB に保存する
- `location_key = {event_id}#{selected_choice}` で LocationObservation を統合する
- `observation_count`・`risk_tags`・`priority_class` を更新する
- `prompts/summarization/`・`prompts/tagging/` からプロンプトを読み込む（N-07）
- AI 入出力・モデル ID・実行時刻を CloudWatch Logs に記録する（N-04）

**主要コンポーネント**:
- ReportProcessingWorkflow（Step Functions Express Workflow）
- BedrockFunction（Lambda・summarize アクション）
- AISummaryRepository（DynamoDB）
- LocationObservationRepository（DynamoDB）
- `prompts/summarization/`・`prompts/tagging/`

**コード配置**: `functions/bedrock/`・`lib/ai/`・`lib/locations/`・`workflows/report-processing/`・`prompts/`

**完了条件**:
- ワークフローが投稿を受け取り AISummary を生成する
- LocationObservation が作成・更新される
- `priority_class` がルールベースで正しく計算される
- `has_high_signal` フラグが設定される

---

### Unit 7: 役立っている通知エンジン + 進化予兆

**目的**: 有効観察に対して通知を生成し、role_state の進化予兆を通知文で表現する

**責務**:
- UserProfile の `valid_observation_count` を更新する
- `role_state` 遷移を評価・更新する（anonymous → trusted_candidate → special_candidate）
- BedrockFunction（`notify` アクション）で role_state に応じた通知文を生成する（**3種類全て MVP で実装**）
- Notification を DynamoDB に保存する
- `GET /notifications` で未読通知一覧を返す
- `prompts/notifications/` からプロンプトを読み込む（N-07）
- UI 上に役職名・バッジは表示しない（通知文のニュアンスのみ）

**主要コンポーネント**:
- NotificationFunction（Lambda）
- BedrockFunction（Lambda・notify アクション）
- NotificationRepository（DynamoDB）
- UserProfileRepository（DynamoDB）
- `prompts/notifications/`

**コード配置**: `functions/notification/`・`lib/notifications/`・`lib/users/`・`prompts/notifications/`

**完了条件**:
- ワークフロー完了後に Notification が生成される
- `role_state` に応じて通知文のトーンが変わる
- `GET /notifications` が未読通知を返す
- 通知タップで事件詳細へ遷移できる

---

### Unit 9: 行政ダッシュボード

**目的**: 行政担当者が確認候補地点を一覧・詳細確認・レビュー更新できる画面を提供する

**責務**:
- AdminDashboard（Next.js）を実装する
- **2層認証を実装する**: Next.js 固定パスワード（画面保護）+ `ADMIN_TOKEN` ヘッダー（API 保護）
- `GET /admin/locations` で確認候補地点一覧を返す（新着順）
- `GET /admin/locations/{id}` で地点詳細を返す
- `PUT /admin/locations/{id}/review` でレビュー状態を更新する
- ReviewLog に変更履歴を記録する
- デモトリガーボタンを提供する

**主要コンポーネント**:
- AdminDashboard（Next.js）
- AdminFunction（Lambda）
- LocationObservationRepository（DynamoDB）
- ReviewLogRepository（DynamoDB）

**コード配置**: `app/admin/`・`components/admin/`・`functions/admin/`・`lib/admin/`

**完了条件**:
- **2層認証**が動作する（Next.js パスワード + Lambda トークン検証）
- 確認候補地点一覧が新着順で表示される
- 地点詳細（写真・要約・危険要因・優先度）が表示される
- レビュー状態を更新できる
- ReviewLog に記録される

---

### Unit 11: デモ用即時処理トリガー

**目的**: デモ時に投稿整理→地点統合→通知生成を10秒以内に完結させる

**責務**:
- `POST /demo/trigger` を実装する
- DemoFunction（Lambda・タイムアウト30秒）を実装する
- ReportProcessingWorkflow を `startSyncExecution` で同期実行する
- デモ用シードデータ投入・リセットスクリプトを提供する
- `DEMO_TOKEN` による認証を実装する（AdminFunction の `ADMIN_TOKEN` とは独立して管理）

**主要コンポーネント**:
- DemoFunction（Lambda）
- `data/demo-seeds/` シードデータ

**コード配置**: `functions/demo/`・`data/demo-seeds/`

**完了条件**:
- `POST /demo/trigger` が10秒以内に完了する
- シードデータ投入スクリプトが **DynamoDB Local 向け**と **AWS DynamoDB 向け**の両方で動作する
- デモ後のリセットスクリプトが動作する
- DynamoDB Local でフロントエンド開発用のシードデータを確認できる
- AWS DynamoDB でデモ本番用のシードデータを確認できる

---

## Phase 2 Units（将来拡張）

### Unit 2: 事件ガードレール
- AI 生成事件の安全性チェック・承認フロー
- `prompts/guardrails/` の実装
- MVP では #T-01 固定のため不要

### Unit 8（統合済み → Unit 7 に含まれる）
- role_state 評価・進化予兆は Unit 7 に統合済み

### Unit 10: 投稿モデレーション / 安全制御
- `moderation_status` の管理 UI・自動チェック
- MVP では `moderation_status` フラグのみ（UI なし）

---

## Phase 1 実装順序

```
Unit 1（事件化エンジン）
    ↓
Unit 3（フロントエンド）  ← Unit 1 のデータを表示
    ↓
Unit 4（投稿受付）        ← Unit 3 の投稿フォームから呼ばれる
    ↓
Unit 5（投稿整理AI+地点統合） ← Unit 4 の保存後に起動
    ↓
Unit 7（通知+進化予兆）   ← Unit 5 のワークフロー内で実行
    ↓
Unit 9（行政ダッシュボード） ← Unit 5 の LocationObservation を表示
    ↓
Unit 11（デモトリガー）   ← 全 Unit を統合してデモループを実行
```
