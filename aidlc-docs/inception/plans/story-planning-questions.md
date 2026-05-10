# ストーリー計画 質問ファイル — 町内匿名捜査室

requirements.md とアドバイスの内容を踏まえ、ストーリー生成の質を高めるために確認します。
各質問の `[Answer]:` タグに回答を記入してください。

---

## Question 1: 匿名捜査員の「最初の一歩」の設計

初めてアプリを開いたユーザーが最初に見る画面・体験をどう設計しますか？

A) 事件一覧をすぐ表示する（登録・チュートリアルなし・即参加）
B) 簡単なオンボーディング（1〜2画面）を経てから事件一覧へ
C) 事件 #T-01 の詳細を直接表示する（1事件固定 MVP に最適化）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 2: 投稿フローの最小構成

「3タップ以内で投稿完了」を実現するための最小投稿フローはどれですか？

A) 選択式推理 → 写真または一文テキスト → 送信（推理必須）
B) 写真または一文テキスト → 送信（推理はオプション）
C) 推理選択 → 送信（テキスト・写真はオプション）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 3: 「役立っている通知」の受け取り方

ユーザーはどこで通知を受け取りますか？

A) アプリ内通知のみ（次回アプリを開いたときに表示）
B) ブラウザのプッシュ通知（PWA）
C) アプリ内通知 + 将来的に LINE 連携（MVP はアプリ内のみ）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question 4: 行政担当者のダッシュボード操作の主な動線

行政担当者が週次レビューで最もよく使う操作はどれだと想定しますか？

A) 確認候補地点一覧を見て、気になる地点の詳細を開く
B) 優先度（即応候補 / 中期確認候補）でフィルタリングして確認する
C) 新着順で並べて上から順に確認する
X) Other (please describe after [Answer]: tag below)

[Answer]: 
C
---

## Question 5: 「他の捜査員の報告」の見せ方

事件詳細画面で他の捜査員の報告をどう表示しますか？

A) 選択式回答の集計（例：「B を選んだ捜査員が最多」）のみ表示
B) 一文報告の断片を匿名で数件表示（個人特定情報なし）
C) 選択式集計 + 一文報告断片の両方を表示
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question 6: 「特命予兆」の表現方法（MVP 範囲内）

MVP では正式昇格なしで「予兆」だけを通知文で表現します。
どのタイミングで予兆通知を出しますか？

A) 有効観察が3件に達したとき（`trusted_candidate` 遷移時）
B) 有効観察が5件 + 類似案件再参照1件に達したとき（`special_candidate` 遷移時）
C) MVP では予兆通知は実装せず、通常の有効観察通知のみ
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 7: 運営者・モデレーターのストーリーの MVP 範囲

運営者向けのストーリーをどこまで MVP に含めますか？

A) 含めない（運営者向け UI は MVP 外・`moderation_status` フラグのみ）
B) 最小限含める（`moderation_status` の手動更新 API のみ・UI なし）
C) 簡易モデレーション画面を含める
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question 8: デモシナリオとストーリーの対応

idea.md の「Scene 0〜7」のデモシナリオに対して、ストーリーはどう対応させますか？

A) デモシナリオの各 Scene を1つのストーリーとして対応させる
B) デモシナリオとは独立してストーリーを生成し、後でマッピングする
C) デモシナリオを「受容基準」として各ストーリーに組み込む
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

すべての質問に回答したら「回答完了」とお知らせください。
