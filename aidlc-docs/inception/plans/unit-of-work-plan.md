# Unit of Work Plan — 町内匿名捜査室

## 実行チェックリスト

### Part 1: Planning
- [x] Step 1: コンテキスト分析（requirements.md・stories.md・application-design 読み込み済み）
- [x] Step 2: Unit of Work Plan 作成（本ファイル）
- [x] Step 3: 質問生成（unit-of-work-questions.md 作成）
- [x] Step 4: ユーザーの回答収集
- [x] Step 5: 回答の曖昧さ分析
- [x] Step 6: 承認取得

### Part 2: Generation
- [x] Step 7: unit-of-work.md 生成
- [x] Step 8: unit-of-work-dependency.md 生成
- [x] Step 9: unit-of-work-story-map.md 生成
- [x] Step 10: 完了確認・承認取得

---

## 分解方針

Application Design で確定した以下の構造を Unit に対応させる。
**Unit 5+6 統合・Unit 7+8 統合済み。unit-of-work.md が正。**

| Unit | 名称 | 主要コンポーネント | Phase |
|---|---|---|---|
| Unit 1 | 事件化エンジン | EventFunction + EventRepository + モックデータ | Phase 1 |
| Unit 3 | 匿名捜査員向けフロントエンド | InvestigatorApp | Phase 1 |
| Unit 4 | 投稿受付・保存基盤 | ReportFunction + ReportRepository + S3 | Phase 1 |
| Unit 5 | 投稿整理AI + 地点統合 + Workflow | BedrockFunction（summarize）+ AISummaryRepository + LocationObservationRepository + ReportProcessingWorkflow | Phase 1 |
| Unit 7 | 役立っている通知 + 進化予兆 | BedrockFunction（notify）+ NotificationRepository + UserProfileRepository | Phase 1 |
| Unit 9 | 行政ダッシュボード | AdminDashboard + AdminFunction + ReviewLogRepository | Phase 1 |
| Unit 11 | デモ用即時処理トリガー | DemoFunction + シードスクリプト | Phase 1 |
| Unit 2 | 事件ガードレール | ガードレールプロンプト + 承認フロー | Phase 2 |
| Unit 10 | 投稿モデレーション / 安全制御 | moderation_status 管理 | Phase 2 |

---

## 質問ファイル参照
`aidlc-docs/inception/plans/unit-of-work-questions.md` を参照
