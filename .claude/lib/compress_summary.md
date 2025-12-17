---
title: "Summary 圧縮ユーティリティ関数集"
description: "詳細版→圧縮版の変換ロジック"
---

# Summary 圧縮ユーティリティ

このドキュメントは、`@summarize_inputs` コマンド内で使用する圧縮ロジック関数のリファレンスです。

---

## 1. セクション抽出関数

### `extract_section(text, keywords)`

指定されたキーワードに合致するセクションを抽出します。

```
入力:
  text: Markdown テキスト（詳細版）
  keywords: ["背景", "課題", "背景説明"] 等

処理:
  1. 見出し（## または ### ）を探索
  2. キーワードに合致する見出しを特定
  3. その見出し以降のコンテンツを抽出
  4. 次の見出しが出現したら終了

出力:
  セクション内のテキスト
```

### `extract_bullet_points(section_text)`

セクションから箇条書きを抽出します。

```
入力:
  section_text: セクション内のテキスト

処理:
  1. "- " または "* " で始まる行を特定
  2. インデント構造を保持
  3. 空行で区切られたグループを識別

出力:
  [[item1, item2, ...], [item3, item4, ...], ...]
```

---

## 2. 圧縮ロジック関数

### `compress_background(section_text, max_words=150, max_points=3)`

背景・課題セクションを圧縮します。

```
入力:
  section_text: "## ビジネス背景" 以降のテキスト
  max_words: 出力の最大文字数
  max_points: 抽出するポイント数

処理:
  1. 箇条書きを抽出
  2. 文字数 50～100 字のポイントを選定
  3. 定量的データ（%、数値）を優先
  4. 修飾表現を削除（「非常に」「重要な」）
  5. ビジネス目標を 1～2 文で別途抽出

出力:
  - 背景ポイント (最大 3 個)
  - ビジネス目標 (1～2 文)
```

**例**:

```markdown
## ビジネス背景・課題

### 入力（詳細版）:
- 営業データ集計に手作業が多く、集計担当者の負荷が高い
- 現在、営業チーム全体で月 200 時間の手作業が発生
- ヒューマンエラーにより、月平均 5～10 件のデータ誤りが発生
- データ誤りによる営業判断の遅延が課題
- AI を活用することで効率化を目指す
- 目標: 手作業を 80% 削減し、データ精度を 99% 以上達成

### 出力（圧縮版）:
- 営業データ集計に月 200 時間の手作業が必要
- ヒューマンエラーが月 5～10 件発生
- 目標: AI 導入により手作業 80% 削減、精度 99% 以上
```

### `compress_users(section_text, max_items=3)`

ユーザー・ステークホルダーセクションを圧縮します。

```
入力:
  section_text: "## ユーザー" または "## ステークホルダー" セクション

処理:
  1. ロールごとに情報をグループ化
  2. 各ロール 1 行（20～30 字以下）にまとめる
  3. 意思決定者→実行担当→その他の優先順
  4. 説明的な部分は削除

出力:
  [
    "営業部長（意思決定者・CEO へのレポート責任）" → "営業部長（意思決定者）",
    "営業チーム（データ利用者）",
    "IT 部門（システム保守担当）"
  ]
```

### `extract_scope_what(section_text, max_items=5)`

スコープの「対象」セクション（WHAT）を抽出します。

```
入力:
  section_text: "## 要件" または "## スコープ" セクション

処理:
  1. "対象" / "機能" / "スコープに含む" など のセクション探索
  2. 箇条書きを抽出
  3. 重要度順にソート
     - ビジネス目標に直結するもの
     - ユーザーの主要ニーズ
     - サポート機能
  4. 上位 5 個を選定
  5. 重複・冗長性を排除

出力:
  [
    "営業データの自動集計",
    "週次レポートの自動生成",
    "顧客セグメント別分析",
    "売上予測",
    "営業パイプライン可視化"
  ]
```

### `extract_scope_what_not(section_text, max_items=3)`

スコープの「対象外」セクション（WHAT NOT）を抽出します。

```
入力:
  section_text: "## 要件" セクション

処理:
  1. "対象外" / "スコープに含まない" など のセクション探索
  2. 明示的に「○○は対象外」と書かれたものを優先
  3. 暗黙的な除外項目（新規開発等）を検出
  4. 重要度順に上位 3 個を選定

出力:
  [
    "CRM システムの改修",
    "営業システムインフラの刷新",
    "営業プロセス自体の変更"
  ]
```

### `extract_constraints(section_text, max_items=5)`

制約・前提条件セクションを抽出します。

```
入力:
  section_text: "## 制約" または "## 前提条件" セクション

処理:
  1. "制約" / "前提" / "前置き" など のセクション探索
  2. 実装に関わる制約を優先
     - 期間、予算、リソース、技術要件
  3. ビジネス制約を次点
     - 市場条件、組織制限
  4. 背景説明（「～なので」）は削除
  5. 上位 5 個を選定

出力:
  [
    "予算上限: 500 万円",
    "実装期間: 6 ヶ月以内",
    "既存営業システムとの連携が必須",
    "営業チーム（15 名）への研修実施必須",
    "月次レポート提出時には稼働状態を要する"
  ]
```

### `extract_top_benefits(section_text, max_items=3)`

メリット・期待効果セクションを抽出します。

```
入力:
  section_text: "## 期待効果" または "## メリット" セクション

処理:
  1. "期待効果" / "メリット" / "得られるもの" など のセクション探索
  2. 定量的メリット（%、数値）を優先
  3. ビジネスインパクトが大きいもの順
  4. 抽象的な表現は削除
  5. 上位 3 個を選定

出力:
  [
    "営業データ集計の手作業削減：50%",
    "営業意思決定速度の向上：40% 短縮",
    "営業チーム満足度向上により離職率低下"
  ]
```

---

## 3. テンプレート適用関数

### `format_compressed_summary(data_dict)`

圧縮データを最終的なテンプレート形式に整形します。

```
入力:
  {
    "background": "...",
    "business_goal": "...",
    "users": [...],
    "scope_what": [...],
    "scope_what_not": [...],
    "constraints": [...],
    "benefits": [...]
  }

処理:
  1. Markdown テンプレートに値を埋め込む
  2. 行数・文字数をチェック
  3. 見出しのハイフン等を自動生成
  4. 最終フォーマット: UTF-8 Markdown

出力:
  完成したMarkdown テキスト
```

---

## 4. 検証関数

### `validate_compressed_summary(text, detailed_text)`

圧縮版が正しく生成されたかを検証します。

```
入力:
  text: 生成した圧縮版テキスト
  detailed_text: 元の詳細版テキスト

検証項目:
  1. 圧縮率が 20～30% か確認
     compression_ratio = len(text) / len(detailed_text)
  2. 必須セクション（背景、ユーザー等）が揃っているか
  3. 行数が異常でないか（< 300 行）
  4. テンプレート構造が正しいか

出力:
  {
    "is_valid": True/False,
    "compression_ratio": 0.25,
    "missing_sections": [...],
    "warnings": [...]
  }
```

### `count_tokens_estimate(text)`

トークン数を推定します（目安）。

```
入力: Markdown テキスト

処理:
  1. 日本語: 文字数 ÷ 3.5 ≈ トークン数
  2. 英語: 単語数 ÷ 1.3 ≈ トークン数
  3. 全体: (日本語トークン) + (英語トークン)

出力:
  推定トークン数
```

---

## 5. 実装例

### 標準的な圧縮フロー

```python
# Step 1: ファイル読み込み
detailed = read_file("docs/00_inputs/summary.md")

# Step 2: 各セクションを抽出・圧縮
background_section = extract_section(detailed, ["背景", "課題"])
background = compress_background(background_section)

users_section = extract_section(detailed, ["ユーザー", "ステークホルダー"])
users = compress_users(users_section)

req_section = extract_section(detailed, ["要件", "スコープ"])
scope_what = extract_scope_what(req_section)
scope_what_not = extract_scope_what_not(req_section)

constraint_section = extract_section(detailed, ["制約", "前提"])
constraints = extract_constraints(constraint_section)

benefit_section = extract_section(detailed, ["期待効果", "メリット"])
benefits = extract_top_benefits(benefit_section)

# Step 3: テンプレート適用
compressed = format_compressed_summary({
    "background": background.points,
    "business_goal": background.goal,
    "users": users,
    "scope_what": scope_what,
    "scope_what_not": scope_what_not,
    "constraints": constraints,
    "benefits": benefits
})

# Step 4: 検証
validation = validate_compressed_summary(compressed, detailed)
if not validation.is_valid:
    print("警告:", validation.warnings)

# Step 5: 保存
write_file("docs/00_inputs/summary_compressed.md", compressed)
print(f"✓ 圧縮率: {validation.compression_ratio:.1%}")
```

---

## 注意事項

### セクション名の多様性への対応

Markdown セクションの見出しは多様になりがちです。以下のキーワード拡張で対応：

```
"背景": ["背景", "背景・課題", "ビジネス背景", "背景説明", "状況"]
"ユーザー": ["ユーザー", "ステークホルダー", "想定読者", "利用者"]
"要件": ["要件", "スコープ", "機能要件", "対象"]
"制約": ["制約", "前提条件", "前置き", "留意事項", "制限事項"]
"メリット": ["期待効果", "メリット", "得られるもの", "ビジネスインパクト"]
```

### 抽出の失敗ハンドリング

セクションが見つからない場合の動作：

```
- セクションが複数ある場合: 最初のものを採用
- セクションがない場合: 空のプレースホルダーを使用
- 内容が不足する場合: ユーザーに手動編集を促す
```

