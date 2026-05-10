# Unit of Work 依存マトリクス — 町内匿名捜査室

## 依存マトリクス

| Unit | 依存先 Unit | 依存種別 | 理由 |
|---|---|---|---|
| Unit 1 | なし | — | 独立して実装可能 |
| Unit 3 | Unit 1 | データ依存 | 事件データがないと画面が空になる |
| Unit 4 | Unit 1 | データ依存 | report に event_id が必要 |
| Unit 5 | Unit 4 | 実行依存 | 投稿がないと整理できない |
| Unit 5 | Unit 1 | データ依存 | event_context を参照する |
| Unit 7 | Unit 5 | 実行依存 | ワークフロー内で Unit 5 の後に実行 |
| Unit 9 | Unit 5 | データ依存 | LocationObservation を表示する |
| Unit 9 | Unit 7 | データ依存 | Notification の存在を前提とする |
| Unit 11 | Unit 1〜9 全て | 統合依存 | 全 Unit が揃ってデモループが成立 |

## 依存グラフ

```
Unit 1
  |
  +---> Unit 3（フロントエンド）
  |
  +---> Unit 4（投稿受付）
              |
              v
          Unit 5（投稿整理AI+地点統合）
              |
              +---> Unit 7（通知+進化予兆）
              |
              +---> Unit 9（行政ダッシュボード）
                          |
                          v
                      Unit 11（デモトリガー）
```

## 並行開発の可能性

| 並行可能な組み合わせ | 条件 |
|---|---|
| Unit 1 + Unit 3 の画面骨格 | Unit 3 は API モックで先行開発可能 |
| Unit 4 + Unit 9 の画面骨格 | Unit 9 は LocationObservation モックで先行開発可能 |
| Unit 5 の Bedrock プロンプト調整 | Unit 4 完了後に並行して調整可能 |

## クリティカルパス

```
Unit 1 → Unit 4 → Unit 5 → Unit 11
```

このパスが最短でデモループを成立させる最小構成。
Unit 3・7・9 はデモの完成度を高めるが、クリティカルパスではない。

## ブロッカーリスク

| リスク | 影響 Unit | 対策 |
|---|---|---|
| Bedrock モデル利用不可 | Unit 5・7 | Construction 開始前にモデルアクセス確認必須 |
| Step Functions 10秒超過 | Unit 11 | Express Workflow のステップ数を最小化 |
| DynamoDB スキーマ変更 | 全 Unit | Unit 1 でスキーマを先に確定させる |
