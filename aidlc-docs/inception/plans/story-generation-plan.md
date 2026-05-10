# Story Generation Plan — 町内匿名捜査室

## 実行チェックリスト

### Part 1: Planning

- [x] Step 1: User Stories の必要性評価（user-stories-assessment.md 作成済み）
- [x] Step 2: ストーリー生成計画の作成（本ファイル）
- [x] Step 3: 質問の生成（story-planning-questions.md 作成）
- [ ] Step 4: ユーザーの回答収集
- [ ] Step 5: 回答の曖昧さ分析
- [ ] Step 6: 承認取得

### Part 2: Generation

- [x] Step 7: ペルソナ生成（personas.md）
- [x] Step 8: ユーザーストーリー生成（stories.md）
- [x] Step 9: 受容基準の付与
- [x] Step 10: ペルソナとストーリーのマッピング
- [ ] Step 11: 完了確認・承認取得

---

## ストーリー生成方針

### 採用するブレイクダウンアプローチ
**ペルソナベース × ユーザージャーニーベースのハイブリッド**

- ペルソナ（匿名捜査員・行政担当者・運営者・開発者）ごとにストーリーをグループ化
- 各ペルソナ内ではユーザージャーニー順（最初の接触 → 参加 → 継続 → 深化）に並べる
- 理由: idea.md に「佐藤さん（68歳）」という具体的な主人公が存在し、
  ペルソナ起点の整理が最も自然かつ実装優先度の判断に直結するため

### INVEST 基準の適用
- Independent: 各ストーリーは独立して実装・テスト可能にする
- Negotiable: 受容基準は実装前に調整可能な形で記述
- Valuable: ビジネス価値・ユーザー価値を明示
- Estimable: 実装規模が推定できる粒度
- Small: 1スプリント（数日）で完了できる粒度
- Testable: 受容基準が検証可能な形で記述

### ストーリーフォーマット
```
As a [ペルソナ],
I want to [行動・機能],
So that [動機・価値].

Acceptance Criteria:
- [ ] [検証可能な条件1]
- [ ] [検証可能な条件2]
```

---

## 質問ファイル参照
`aidlc-docs/inception/plans/story-planning-questions.md` を参照
