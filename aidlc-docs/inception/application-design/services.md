# サービス定義 — 町内匿名捜査室

## サービス層の役割

サービス層は複数のコンポーネントをオーケストレーションし、ビジネスユースケースを実現する。
Lambda 関数内のビジネスロジックをリポジトリ層から分離するために使用する。

---

## EventService

**責務**: 事件の取得・提供に関するビジネスロジック

```typescript
// 事件一覧を取得する（MVP では #T-01 固定）
getEventList(): Promise<Event[]>

// 事件詳細と他の捜査員の集計情報を取得する
getEventDetail(eventId: string): Promise<EventDetailWithStats>
// EventDetail + 選択式回答の集計（個人特定情報なし）
```

---

## ReportService

**責務**: 捜査報告の受付・保存・ワークフロー起動

```typescript
// 下書き作成（写真あり: draft保存+URL返却 / 写真なし: submitted保存+ワークフロー起動）
createDraftReport(input: CreateDraftInput, userId: string): Promise<{ report_id: string; upload_url?: string }>
// has_photo=true:
//   1. Report を DynamoDB に保存（status: draft）
//   2. S3 Presigned URL を生成して返す（s3_key = reports/{report_id}/original.jpg）
//   3. ワークフローは起動しない
// has_photo=false:
//   1. Report を DynamoDB に保存（status: submitted）
//   2. ReportProcessingWorkflow を非同期起動
//   3. report_id を返す

// 投稿確定（写真アップロード完了後に呼ぶ）
finalizeReport(reportId: string, userId: string): Promise<void>
// 1. S3 オブジェクトの存在確認（未アップロードなら 400）
// 2. Report の status を draft → submitted に更新
// 3. ReportProcessingWorkflow を非同期起動
```

---

## NotificationService

**責務**: 通知の取得・既読管理

```typescript
// ユーザーの未読通知を取得する
getNotifications(userId: string): Promise<Notification[]>

// 通知を既読にする
markNotificationRead(notificationId: string, userId: string): Promise<void>
```

---

## AdminService

**責務**: 行政ダッシュボード向けのデータ取得・レビュー管理

```typescript
// 確認候補地点一覧を取得する（新着順）
getLocationList(): Promise<LocationObservation[]>

// 確認候補地点の詳細を取得する（写真・要約・危険要因・時間帯傾向を含む）
getLocationDetail(locationId: string): Promise<LocationObservationDetail>

// レビュー状態を更新し、ReviewLog に記録する
updateReviewStatus(locationId: string, status: AdminReviewStatus, comment?: string): Promise<void>
```

---

## ReportProcessingService（Step Functions ステート内）

**責務**: 投稿整理ワークフローの各ステップのビジネスロジック

```typescript
// 投稿を Bedrock で要約・分類し AISummary を保存する
processSummarization(reportId: string): Promise<AISummary>

// AISummary を元に LocationObservation を更新・優先度を再計算する
processLocationUpdate(reportId: string, summary: AISummary): Promise<LocationObservation>

// UserProfile の role_state を評価・更新する
processUserProfileEvaluation(userId: string, hasHighSignal: boolean): Promise<UserProfile>

// 通知文を生成して Notification を保存する
processNotificationGeneration(userId: string, reportId: string, profile: UserProfile): Promise<Notification>
```

---

## DemoService

**責務**: デモ用即時処理の実行管理

```typescript
// Express Workflow を同期実行してデモループを完結させる
executeDemoTrigger(reportId: string): Promise<DemoResult>
// startSyncExecution で ReportProcessingWorkflow を同期実行
// 完了後: { location_id, notification_id, role_state } を返す
```

---

## サービス間の依存関係

```
ReportService
  → ReportRepository（保存）
  → ReportProcessingWorkflow（非同期起動）

ReportProcessingService（ワークフロー内）
  → BedrockFunction（AI 呼び出し）
  → AISummaryRepository
  → LocationObservationRepository
  → UserProfileRepository
  → NotificationRepository

AdminService
  → LocationObservationRepository
  → ReviewLogRepository

DemoService
  → Step Functions（startSyncExecution）
```
