# Application Design Plan — 町内匿名捜査室

## 実行チェックリスト

- [x] Step 1: コンテキスト分析（requirements.md・stories.md 読み込み済み）
- [x] Step 2: Application Design Plan 作成（本ファイル）
- [x] Step 3: 質問生成（application-design-questions.md 作成）
- [ ] Step 4: ユーザーの回答収集
- [ ] Step 5: 回答の曖昧さ分析
- [x] Step 6: 設計成果物の生成
  - [x] components.md
  - [x] component-methods.md
  - [x] services.md
  - [x] component-dependency.md
  - [x] application-design.md（統合版）
- [ ] Step 7: 承認取得

---

## 設計スコープ

### 対象コンポーネント群（Unit 候補から導出）

**フロントエンド層**
- InvestigatorApp（匿名捜査員向け Next.js アプリ）
- AdminDashboard（行政ダッシュボード Next.js アプリ）

**API 層**
- EventAPI（事件 CRUD）
- ReportAPI（投稿受付・S3 Presigned URL）
- NotificationAPI（通知取得）
- AdminAPI（行政ダッシュボード・レビュー更新）
- DemoAPI（デモ用即時処理トリガー）

**ワークフロー層**
- ReportProcessingWorkflow（Step Functions: 投稿整理 → 地点統合 → 通知生成）

**AI 層**
- SummarizationService（Bedrock: 投稿要約・危険要因分類）
- NotificationGenerationService（Bedrock: 通知文生成）

**データ層**
- EventRepository
- ReportRepository
- LocationObservationRepository
- AISummaryRepository
- NotificationRepository
- UserProfileRepository
- ReviewLogRepository

---

## 質問ファイル参照
`aidlc-docs/inception/plans/application-design-questions.md` を参照
