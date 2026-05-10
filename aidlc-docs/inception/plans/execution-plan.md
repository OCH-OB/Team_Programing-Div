# 実行計画 — 町内匿名捜査室

## 詳細分析サマリー

### 変更スコープ
- **変換タイプ**: Greenfield 新規開発
- **主要変更**: 9 Unit 相当（Phase 1: 7 Unit、Phase 2: 2 Unit）の新規実装
- **関連コンポーネント**: フロントエンド・バックエンド API・AI ワークフロー・行政ダッシュボード・デモ基盤

### 変更影響評価
- **ユーザー向け変更**: Yes — 匿名捜査員向け Web アプリ（事件一覧・詳細・投稿・通知）
- **構造的変更**: Yes — サーバレスアーキテクチャ（Lambda・Step Functions・DynamoDB・S3）
- **データモデル変更**: Yes — 7テーブル（Event・Report・LocationObservation・AISummary・Notification・UserProfile・ReviewLog）
- **API 変更**: Yes — 10エンドポイント新規作成
- **NFR 影響**: Yes — レスポンス時間・プライバシー保護・監査可能性

### リスク評価
- **リスクレベル**: 中〜高
- **ロールバック複雑度**: 中（サーバレス・ステートレス設計のため比較的容易）
- **テスト複雑度**: 複雑（AI 統合・Step Functions ワークフロー・Bedrock 依存）
- **主なリスク**:
  - Bedrock モデルの利用可能性（Construction 開始前に確認必須）
  - Step Functions + Bedrock の10秒以内完了（デモ要件）
  - 高齢層 UX の3タップ以内投稿完了

---

## ワークフロー可視化

```
INCEPTION PHASE
+---------------------------+
| [x] Workspace Detection   |  COMPLETED
| [x] Reverse Engineering   |  SKIP (Greenfield)
| [x] Requirements Analysis |  COMPLETED
| [x] User Stories          |  COMPLETED
| [x] Workflow Planning     |  COMPLETED
| [x] Application Design    |  COMPLETED
| [x] Units Generation      |  COMPLETED
+---------------------------+
             |
             v
CONSTRUCTION PHASE (per-unit loop)
+---------------------------+
| [ ] Functional Design     |  EXECUTE (per unit)
| [ ] NFR Requirements      |  EXECUTE (per unit)
| [ ] NFR Design            |  EXECUTE (per unit)
| [ ] Infrastructure Design |  EXECUTE (per unit)
| [ ] Code Generation       |  EXECUTE (per unit)
+---------------------------+
             |
             v
+---------------------------+
| [ ] Build and Test        |  EXECUTE
+---------------------------+
             |
             v
OPERATIONS PHASE
+---------------------------+
| [ ] Operations            |  PLACEHOLDER
+---------------------------+
```

---

## 実行するフェーズ

### 🔵 INCEPTION PHASE
- [x] Workspace Detection（COMPLETED）
- [x] Reverse Engineering（SKIP — Greenfield）
- [x] Requirements Analysis（COMPLETED）
- [x] User Stories（COMPLETED）
- [x] Workflow Planning（COMPLETED）
- [x] Application Design — **COMPLETED**
- [x] Units Generation — **COMPLETED**

### 🟢 CONSTRUCTION PHASE（per-unit）
- [ ] Functional Design — **EXECUTE**
  - **理由**: 各 Unit のビジネスロジック・ドメインモデル・業務ルールの詳細設計が必要
- [ ] NFR Requirements — **EXECUTE**
  - **理由**: Bedrock モデル選定・DynamoDB 設計・S3 設計・Step Functions 設計が必要
- [ ] NFR Design — **EXECUTE**
  - **理由**: サーバレスパターン・リトライ戦略・タイムアウト設計が必要
- [ ] Infrastructure Design — **EXECUTE**
  - **理由**: AWS リソースマッピング（Lambda・API Gateway・DynamoDB・S3・Step Functions・Amplify）が必要
- [ ] Code Generation — **EXECUTE**（ALWAYS）
- [ ] Build and Test — **EXECUTE**（ALWAYS）

### 🟡 OPERATIONS PHASE
- [ ] Operations — PLACEHOLDER

---

## Unit of Work（Phase 1: 7 Unit）

Units Generation フェーズで Unit 5+6 統合・Unit 7+8 統合を決定。
**unit-of-work.md が正とする。**

### Phase 1（MVP デモ必須・7 Unit）

| Unit | 名称 | 対応ストーリー | 優先度 |
|---|---|---|---|
| Unit 1 | 事件化エンジン | — | ★★★ |
| Unit 3 | 匿名捜査員向けフロントエンド | US-01〜03, US-07 | ★★★ |
| Unit 4 | 投稿受付・保存基盤 | US-04 | ★★★ |
| Unit 5 | 投稿整理AI + 地点統合 + Workflow | US-07, US-09, US-10 | ★★★ |
| Unit 7 | 役立っている通知 + 進化予兆 | US-05, US-06 | ★★★ |
| Unit 9 | 行政ダッシュボード | US-08〜11 | ★★★ |
| Unit 11 | デモ用即時処理トリガー | US-12, US-13 | ★★★ |

### Phase 2（将来拡張）

| Unit | 名称 | 優先度 |
|---|---|---|
| Unit 2 | 事件ガードレール | ★★☆ |
| Unit 10 | 投稿モデレーション / 安全制御 | ★☆☆ |

---

## 推奨 Unit 実装順序

```
Unit 1 → Unit 3 → Unit 4 → Unit 5 → Unit 7 → Unit 9 → Unit 11
```

クリティカルパス: Unit 1 → Unit 4 → Unit 5 → Unit 11

---

## 推定タイムライン

- **合計フェーズ数**: INCEPTION 完了済み + CONSTRUCTION（7 Unit × 5 ステージ + Build and Test）
- **推定期間**: ハッカソン期間内（数日〜1週間）
- **クリティカルパス**: Unit 1 → Unit 4 → Unit 5 → Unit 11（デモループ）

---

## 成功基準

- **主要目標**: 15分以内のデモで「捜査員投稿 → AI 整理 → 行政ダッシュボード → 通知」のループを実証
- **主要成果物**:
  - 匿名捜査員向け Web アプリ（Next.js）
  - バックエンド API（Lambda + API Gateway）
  - AI 整理ワークフロー（Step Functions + Bedrock）
  - 行政ダッシュボード
  - デモ用即時処理トリガー
- **品質ゲート**:
  - 投稿完了まで3タップ以内
  - デモ用即時処理10秒以内
  - 個人情報がサーバに保存されないこと
