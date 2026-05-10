# Application Design 統合ドキュメント — 町内匿名捜査室

## 設計概要

### アーキテクチャパターン
- **フロントエンド**: 2つの独立した Next.js アプリ（InvestigatorApp / AdminDashboard）
- **バックエンド**: 機能別 Lambda 関数 + API Gateway（5関数）
- **ワークフロー**: Step Functions Express Workflow（1ワークフロー）
- **AI**: 専用 BedrockFunction（action パラメータで処理切り替え）
- **データ**: DynamoDB（7テーブル）+ S3（写真保存）

### 設計決定サマリー

| 決定事項 | 採用案 | 理由 |
|---|---|---|
| フロントエンド構成 | 2アプリ分離 | 捜査員と行政の認証・UX を完全分離 |
| API 構成 | 機能別 Lambda | Unit 境界と一致・タイムアウト独立設定 |
| ワークフロー粒度 | 1ワークフロー | デモ10秒要件・実装シンプル |
| Bedrock 呼び出し | 専用 BedrockFunction | N-07 プロンプト分離準拠 |
| UserProfile 更新 | ワークフロー最後で毎回評価 | デモ即時反映・実装シンプル |
| 優先度分類計算 | 地点統合時に再計算・保存 | 表示5秒以内要件・決定論的ルール |
| S3 キー構造 | `reports/{report_id}/original.jpg` | シンプル・report_id で一意 |
| デモトリガー認証 | DEMO_TOKEN（DemoFunction）/ ADMIN_TOKEN（AdminFunction）を分離 | デモ実行権限と管理権限を独立させる |

---

## コンポーネント一覧

### フロントエンド（2コンポーネント）
- **InvestigatorApp**: 匿名捜査員向け Next.js アプリ
- **AdminDashboard**: 行政担当者向け Next.js アプリ

### API 層（5 Lambda 関数）
- **EventFunction**: 事件 CRUD（`GET /events`, `GET /events/{id}`, `POST /events`）
- **ReportFunction**: 投稿受付（`POST /reports`, `POST /reports/{id}/finalize`）
- **NotificationFunction**: 通知取得（`GET /notifications`）
- **AdminFunction**: 行政ダッシュボード（`GET /admin/locations`, `PUT /admin/locations/{id}/review`）— `ADMIN_TOKEN` 検証
- **DemoFunction**: デモトリガー（`POST /demo/trigger`、タイムアウト30秒）— `DEMO_TOKEN` 検証

### ワークフロー（1ワークフロー）
- **ReportProcessingWorkflow**: 投稿整理 → 地点統合 → UserProfile 評価 → 通知生成

### AI 層（1 Lambda 関数）
- **BedrockFunction**: Bedrock 呼び出し一元管理（`summarize` / `notify`）

### データ層（7 Repository）
- EventRepository, ReportRepository, LocationObservationRepository
- AISummaryRepository, NotificationRepository, UserProfileRepository, ReviewLogRepository

---

## ディレクトリ構成（予定）

```
/                               # ワークスペースルート（アプリケーションコード）
├── app/
│   ├── investigator/           # InvestigatorApp（Next.js）
│   └── admin/                  # AdminDashboard（Next.js）
├── components/
│   ├── investigator/
│   ├── admin/
│   └── shared/
├── lib/
│   ├── events/                 # EventService + EventRepository
│   ├── reports/                # ReportService + ReportRepository
│   ├── notifications/          # NotificationService + NotificationRepository
│   ├── locations/              # LocationObservationRepository
│   ├── ai/                     # BedrockFunction ロジック
│   ├── users/                  # UserProfileRepository
│   └── admin/                  # AdminService + ReviewLogRepository
├── functions/                  # Lambda ハンドラー
│   ├── event/
│   ├── report/
│   ├── notification/
│   ├── admin/
│   ├── demo/
│   └── bedrock/
├── workflows/
│   └── report-processing/      # Step Functions 定義
├── prompts/
│   ├── summarization/
│   ├── tagging/
│   ├── notifications/
│   ├── event-generation/
│   └── guardrails/
├── data/
│   ├── mock-events/            # #T-01 モックデータ
│   └── demo-seeds/             # デモ用シードデータ
├── infrastructure/             # SAM テンプレート（後に CDK へ移行）
└── tests/
    ├── unit/
    ├── integration/
    └── prompt-evals/
```

---

## 詳細ドキュメント参照

- コンポーネント詳細: `components.md`
- メソッドシグネチャ: `component-methods.md`
- サービス定義: `services.md`
- 依存関係・データフロー: `component-dependency.md`
- Unit of Work 分解: `unit-of-work.md`（Units Generation フェーズで生成）
