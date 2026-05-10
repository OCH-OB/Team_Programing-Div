# コンポーネントメソッド定義 — 町内匿名捜査室

※ 詳細なビジネスロジックは Construction フェーズの Functional Design で定義する。
※ ここでは各コンポーネントのメソッドシグネチャと高レベルの目的を定義する。

---

## EventFunction

```typescript
// 事件一覧取得
listEvents(query: { status?: string; area_tag?: string }): Promise<Event[]>

// 事件詳細取得
getEvent(eventId: string): Promise<Event>

// 事件作成（将来の動的生成用）
createEvent(input: CreateEventInput): Promise<Event>
```

---

## ReportFunction

```typescript
// 捜査報告の下書き作成（写真あり・なし共通の第1ステップ）
createDraftReport(input: CreateDraftInput): Promise<{ report_id: string; upload_url?: string }>
// input: { event_id, user_id, selected_choice?, raw_text?, report_type, has_photo: boolean }
// has_photo=true の場合: status=draft で保存し upload_url を返す
// has_photo=false の場合: status=submitted で保存しワークフローを起動する

// 投稿の確定（写真アップロード完了後に呼ぶ）
finalizeReport(reportId: string, userId: string): Promise<void>
// status を draft → submitted に更新し、ReportProcessingWorkflow を非同期起動する
// アップロード未完了（S3 オブジェクト不在）の場合は 400 を返す
```

---

## NotificationFunction

```typescript
// 未読通知一覧取得
listNotifications(userId: string): Promise<Notification[]>

// 通知既読更新
markAsRead(notificationId: string, userId: string): Promise<void>
```

---

## AdminFunction

```typescript
// 確認候補地点一覧取得
listLocations(query: { priority_class?: string; admin_review_status?: string }): Promise<LocationObservation[]>

// 確認候補地点詳細取得
getLocation(locationId: string): Promise<LocationObservationDetail>

// レビュー状態更新
updateReviewStatus(locationId: string, input: UpdateReviewInput): Promise<void>
// input: { admin_review_status, review_comment? }
// 副作用: ReviewLog に記録する
```

---

## DemoFunction

```typescript
// デモ用即時処理トリガー
triggerDemo(input: { report_id: string }): Promise<DemoResult>
// Step Functions Express Workflow を startSyncExecution で同期実行
// タイムアウト: 30秒（10秒以内完了を目標）
```

---

## BedrockFunction

```typescript
// 投稿の要約・危険要因分類
summarizeReport(input: SummarizeInput): Promise<SummarizeOutput>
// input: { report_id, raw_text, selected_choice, event_context }
// output: { summary_text, risk_classification, has_high_signal, followup_hint, admin_excerpt }

// 通知文の生成
generateNotification(input: NotifyInput): Promise<NotifyOutput>
// input: { user_id, role_state, event_id, report_id, has_high_signal }
// output: { message, notification_type }
// note: role_state に応じてトーンを変える（anonymous / trusted_candidate / special_candidate）
```

---

## ReportProcessingWorkflow（Step Functions ステート）

```typescript
// ステート1: 投稿要約・分類
SummarizeReport: BedrockFunction.summarizeReport(report)

// ステート2: AISummary 保存
UpdateAISummary: AISummaryRepository.save(summary)

// ステート3: 地点統合・優先度再計算
UpdateLocationObservation:
  LocationObservationRepository.upsertByLocationKey(locationKey, summary)
  // location_key = {event_id}#{selected_choice}
  // observation_count +1
  // risk_tags 更新
  // priority_class 再計算（ルールベース）

// ステート4: UserProfile 評価・更新
EvaluateUserProfile:
  UserProfileRepository.evaluateAndUpdate(userId)
  // valid_observation_count +1（has_high_signal=true の場合）
  // role_state 遷移評価
  //   anonymous → trusted_candidate: valid_observation_count >= 3
  //   trusted_candidate → special_candidate: count >= 5 AND similar_case_ref_count >= 1

// ステート5: 通知文生成
GenerateNotification: BedrockFunction.generateNotification(context)

// ステート6: 通知保存
SaveNotification: NotificationRepository.save(notification)
```

---

## データ層メソッド（主要なもの）

### LocationObservationRepository

```typescript
// location_key で検索・なければ新規作成・あれば更新
upsertByLocationKey(locationKey: string, data: LocationUpdateData): Promise<LocationObservation>

// 優先度分類の再計算（ルールベース）
// 即応候補: count >= 3 OR (risk_tags に視認遅延/死角あり AND count >= 2)
// 中期確認候補: count >= 1 でそれ以外
calculatePriorityClass(observation: LocationObservation): PriorityClass
```

### UserProfileRepository

```typescript
// UserProfile の取得・なければ初期作成
getOrCreate(userId: string): Promise<UserProfile>

// valid_observation_count 更新・role_state 遷移評価
evaluateAndUpdate(userId: string, hasHighSignal: boolean): Promise<UserProfile>
```

### ReviewLogRepository

```typescript
// レビュー操作の監査ログ追記
append(input: ReviewLogInput): Promise<void>
// input: { target_type, target_id, review_status, reviewer_type, review_comment? }
```
