# コンポーネント依存関係 — 町内匿名捜査室

## 依存マトリクス

| コンポーネント | 依存先 | 依存種別 |
|---|---|---|
| InvestigatorApp | EventFunction, ReportFunction, NotificationFunction | HTTP（API Gateway） |
| AdminDashboard | AdminFunction, DemoFunction | HTTP（API Gateway） |
| EventFunction | EventRepository | 直接呼び出し |
| ReportFunction | ReportRepository, S3 | 直接呼び出し |
| ReportFunction | ReportProcessingWorkflow | 非同期起動（finalize 時のみ） |
| NotificationFunction | NotificationRepository | 直接呼び出し |
| AdminFunction | LocationObservationRepository, ReviewLogRepository | 直接呼び出し |
| DemoFunction | ReportProcessingWorkflow | 同期実行（startSyncExecution） |
| ReportProcessingWorkflow | BedrockFunction | Lambda 呼び出し |
| ReportProcessingWorkflow | AISummaryRepository | 直接呼び出し |
| ReportProcessingWorkflow | LocationObservationRepository | 直接呼び出し |
| ReportProcessingWorkflow | UserProfileRepository | 直接呼び出し |
| ReportProcessingWorkflow | NotificationRepository | 直接呼び出し |
| BedrockFunction | Amazon Bedrock | AWS SDK |
| BedrockFunction | prompts/（ファイル） | ファイル読み込み |
| 全 Repository | Amazon DynamoDB | AWS SDK |
| ReportFunction | Amazon S3 | AWS SDK（Presigned URL） |

---

## データフロー図

### 捜査員投稿フロー（通常・写真あり）

```
InvestigatorApp
  |
  | POST /reports（has_photo: true）
  v
ReportFunction
  |-- ReportRepository --> DynamoDB（Report 保存・status: draft）
  |-- S3（Presigned URL 生成）
  |-- { report_id, upload_url } を返す
  |
  | ※ クライアントが S3 へ直接アップロード（Exif 除去済み）
  |
  | POST /reports/{report_id}/finalize
  v
ReportFunction
  |-- S3 オブジェクト存在確認
  |-- ReportRepository --> DynamoDB（status: draft → submitted）
  |
  | 非同期起動
  v
ReportProcessingWorkflow（Step Functions Express）
  |
  +--> BedrockFunction（action: summarize）
  |      |-- prompts/summarization/ 読み込み
  |      |-- Amazon Bedrock 呼び出し
  |      v
  |    AISummaryRepository --> DynamoDB（AISummary 保存）
  |
  +--> LocationObservationRepository --> DynamoDB（地点統合・優先度再計算）
  |
  +--> UserProfileRepository --> DynamoDB（role_state 評価・更新）
  |
  +--> BedrockFunction（action: notify）
  |      |-- prompts/notifications/ 読み込み
  |      |-- Amazon Bedrock 呼び出し
  |      v
  |    NotificationRepository --> DynamoDB（Notification 保存）
  |
  v
完了
```

### 捜査員投稿フロー（通常・テキストのみ）

```
InvestigatorApp
  |
  | POST /reports（has_photo: false）
  v
ReportFunction
  |-- ReportRepository --> DynamoDB（Report 保存・status: submitted）
  |
  | 非同期起動（即時）
  v
ReportProcessingWorkflow（上記と同じステップ）
```

### デモトリガーフロー

```
AdminDashboard
  |
  | POST /demo/trigger
  v
DemoFunction（タイムアウト: 30秒）
  |
  | startSyncExecution（同期・10秒以内）
  v
ReportProcessingWorkflow（上記と同じステップ）
  |
  v
DemoResult（location_id, notification_id, role_state）
  |
  v
AdminDashboard（ダッシュボード更新）
InvestigatorApp（通知バッジ更新）
```

### 行政ダッシュボードフロー

```
AdminDashboard
  |
  | GET /admin/locations
  v
AdminFunction
  |-- LocationObservationRepository --> DynamoDB（一覧取得）
  v
確認候補地点一覧（新着順・優先度分類付き）

  |
  | PUT /admin/locations/{id}/review
  v
AdminFunction
  |-- LocationObservationRepository --> DynamoDB（レビュー状態更新）
  |-- ReviewLogRepository --> DynamoDB（監査ログ追記）
  v
更新完了
```

---

## 通信パターン

| パターン | 使用箇所 | 理由 |
|---|---|---|
| 同期 HTTP | フロントエンド → API Gateway → Lambda | ユーザー操作への即時応答 |
| 非同期起動 | ReportFunction → Step Functions | 投稿受付を3秒以内に完了させるため |
| 同期実行 | DemoFunction → Step Functions | デモの10秒以内完了要件 |
| Lambda 呼び出し | Step Functions → BedrockFunction | プロンプト管理の分離（N-07） |
| AWS SDK | Lambda → DynamoDB/S3/Bedrock | サーバレス標準パターン |

---

## 循環依存なし確認

- フロントエンド → API → ワークフロー → AI/データ の一方向フロー
- データ層（Repository）は他のコンポーネントに依存しない
- BedrockFunction は他の Lambda を呼ばない
- 循環依存: なし
