# 要件確認質問 — 町内匿名捜査室

`idea.md` の内容を分析した結果、以下の点について確認が必要です。
各質問の `[Answer]:` タグに回答を記入してください。

---

## Question 1: Exif 除去方式の決定

`idea.md` に記載の通り、写真アップロード時の Exif 除去方式に不整合があります。どちらの方式を採用しますか？

A) クライアント側 JS で Exif を除去してから Presigned URL で S3 へ直接アップロードする（実装コスト低・MVP 向き）
B) S3 イベントで Lambda を起動し、非同期に Exif 除去済み画像を生成する（確実性高・実装コスト高）
C) MVP では Exif 除去を実装せず、UI 注意喚起のみとする（最小実装・後続フェーズで対応）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question 2: インフラ構成管理ツールの選択

バックエンドのインフラ構成管理に何を使いますか？

A) AWS SAM（ローカル実行が容易・Lambda/API Gateway 中心の構成に向く）
B) AWS CDK TypeScript（TypeScript 統一・Step Functions の扱いやすさ・長期保守性）
C) AWS SAM でプロトタイプ後、CDK へ移行する
X) Other (please describe after [Answer]: tag below)

[Answer]: 
C
---

## Question 3: 行政ダッシュボードの認証方式

MVP での行政ダッシュボードへのアクセス制御をどうしますか？

A) Next.js 側で環境変数ベースの固定パスワード（最小実装・デモ向き）
B) API Gateway + Lambda オーソライザーでトークン検証
C) 認証なし（デモ環境のみ・URL 知っている人だけアクセス可）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question 4: MVP Phase 1 の Unit 6 / Unit 7 の扱い

デモで「役立っている通知」と「行政ダッシュボードへの確認候補地点表示」を見せるには、Unit 6（地点統合）と Unit 7（通知エンジン）が必要です。どう扱いますか？

A) Unit 6 / Unit 7 の最小版を Phase 1 に含める（デモ完全性を優先）
B) Unit 11（デモ用即時処理）に地点統合と通知生成の最小実装を含め、Unit 6 / Unit 7 は Phase 2 以降
C) Unit 5（投稿整理 AI）の出力を直接行政ダッシュボードに表示し、Unit 6 は簡略化する
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question 5: 画像モデレーションの MVP 範囲

投稿写真の安全性チェックをどこまで MVP に含めますか？

A) UI 注意喚起のみ（「顔・ナンバーが写った写真は投稿しないでください」の表示）
B) UI 注意喚起 + 運営が手動で確認できる `moderation_status` フラグのみ
C) Bedrock の画像解析で自動チェック（実装コスト高・MVP 外候補）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 6: 通知の種類と境界

MVP で実装する通知を明確にしてください。

A) 「投稿整理完了通知」のみ（最小実装）
B) 「投稿整理完了通知」＋「有効観察通知」（役立っている通知）
C) 「投稿整理完了通知」＋「有効観察通知」＋「特命予兆通知」（idea.md 記載の全通知）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 7: Bedrock モデルの利用可能性確認

現在の AWS アカウントで利用可能な Bedrock モデルとリージョンを確認していますか？

A) はい、確認済み（利用可能なモデル名とリージョンを [Answer] 欄に記載してください）
B) まだ確認していない（Construction フェーズ開始前に確認する）
C) ローカル開発中はモックを使い、デプロイ時に確認する
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 8: セキュリティ拡張の適用

このプロジェクトにセキュリティベースラインルール（SECURITY-01〜15）を適用しますか？

A) Yes — すべての SECURITY ルールをブロッキング制約として適用する（本番グレードのアプリケーション向け）
B) No — SECURITY ルールをスキップする（PoC・プロトタイプ・ハッカソン MVP 向け）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 9: プロパティベーステストの適用

このプロジェクトにプロパティベーステスト（PBT）ルールを適用しますか？

A) Yes — すべての PBT ルールをブロッキング制約として適用する
B) Partial — 純粋関数とシリアライゼーションのラウンドトリップのみに適用する
C) No — PBT ルールをスキップする（シンプルな CRUD・UI 中心・薄い統合レイヤー向け）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
C
---

## Question 10: フロントエンドのデプロイ先

Next.js フロントエンドのデプロイ先はどこを想定していますか？

A) AWS Amplify（idea.md 記載のデモ保険候補・簡単デプロイ）
B) Vercel（Next.js との親和性高・無料枠あり）
C) AWS CloudFront + S3（静的エクスポート）
D) AWS App Runner または ECS（SSR が必要な場合）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question 11: ローカル開発環境の優先度

ローカル開発環境（DynamoDB Local、SAM Local 等）のセットアップをどこまで重視しますか？

A) 完全なローカル環境を構築する（DynamoDB Local + SAM Local + Step Functions Local）
B) DynamoDB Local のみローカル、Lambda は AWS 実環境を使う
C) すべて AWS 実環境を使う（ローカル環境は最小限）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 12: MVP のデモ対象者

このハッカソン MVP のデモは主に誰に向けて行いますか？

A) ハッカソン審査員（技術的な深さと完成度を重視）
B) 行政担当者（実用性と行政接続の価値を重視）
C) 一般ユーザー（UX と没入感を重視）
D) 上記すべて（バランスよく）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

すべての質問に回答したら「回答完了」とお知らせください。
