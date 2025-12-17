---
title: "@summarize_inputs コマンド実装ガイド（Phase 3）"
description: "詳細版→圧縮版の自動生成機能を実装するための詳細ガイド"
---

# @summarize_inputs コマンド実装ガイド

このドキュメントは、`@summarize_inputs` コマンドに圧縮版 Summary 生成機能を追加実装する際の手順書です。

---

## 実装方法

### パターン A: Claude Agent として実装（推奨）

`@summarize_inputs` を実行すると、以下の 2 段階の Task が発動します：

```
ユーザー: @summarize_inputs 実行
  ↓
Stage 1: 詳細版 Summary 生成（既存）
  ├─ docs/00_inputs/ の全ファイルを読み込み
  ├─ Markdown でサマリ作成
  └─ docs/00_inputs/summary.md に保存
  ↓
Stage 2: 圧縮版 Summary 生成（新規・当実装）
  ├─ summary.md を入力
  ├─ 圧縮ロジック実行
  └─ docs/00_inputs/summary_compressed.md に保存
  ↓
✓ 完了
```

### パターン B: スクリプト呼び出しとして実装

詳細版生成後、圧縮処理用の独立したスクリプトを呼び出す方法。

```
Stage 1 で summary.md 生成
  ↓
Task: compress_summary スクリプト実行
  ├─ 圧縮ロジック適用
  └─ summary_compressed.md に保存
```

---

## 実装の具体的な流れ

### Step 1: @summarize_inputs の実装構造

```markdown
# 入力要約コマンド

`docs/00_inputs/` 配下の全 `.md` ファイルを要約し、以下 2 つを生成：

1. docs/00_inputs/summary.md（詳細版）
2. docs/00_inputs/summary_compressed.md（圧縮版）

---

## 実装ワークフロー

[ここに詳細版・圧縮版生成のワークフローを記述]

### ステップ 1-3: 詳細版生成（既存通り）

[既存の実装を記述]

### ステップ 4-5: 圧縮版生成（新規）

以下のエージェントを Task として呼び出す：

エージェント: summarize-compressor
入力:
  - summary_path: docs/00_inputs/summary.md
  - output_path: docs/00_inputs/summary_compressed.md

出力:
  - 圧縮版 Summary
  - 圧縮率メッセージ
```

### Step 2: Compressor エージェントの定義

新しいエージェント `.claude/agents/summarize-compressor.md` を作成：

```markdown
---
name: summarize-compressor
description: 詳細版 Summary を圧縮版に変換
color: blue
---

# Summary 圧縮エージェント

詳細版 Summary を読み込み、圧縮版を生成します。

## 入力

- **summary_path**: docs/00_inputs/summary.md のパス
- **output_path**: 出力先パス（デフォルト: docs/00_inputs/summary_compressed.md）

## 処理フロー

1. summary.md を読み込む
2. セクション別に圧縮ロジックを適用
3. テンプレートに適用
4. 検証実行
5. ファイル保存

## 出力

- Markdown 形式の圧縮版 Summary
- メタ情報（圧縮率、トークン数）
```

### Step 3: 圧縮エージェントの実装

エージェント内で以下を実行：

```python
# 1. ファイル読み込み
detailed_summary = read_file("docs/00_inputs/summary.md")

# 2. セクション抽出・圧縮
background = compress_background(
    extract_section(detailed_summary, ["背景", "課題"])
)
users = compress_users(
    extract_section(detailed_summary, ["ユーザー", "ステークホルダー"])
)
scope_what = extract_scope_what(
    extract_section(detailed_summary, ["要件", "スコープ"])
)
scope_what_not = extract_scope_what_not(
    extract_section(detailed_summary, ["要件", "スコープ"])
)
constraints = extract_constraints(
    extract_section(detailed_summary, ["制約", "前提"])
)
benefits = extract_top_benefits(
    extract_section(detailed_summary, ["期待効果", "メリット"])
)

# 3. テンプレート適用
compressed = format_compressed_summary({
    "background": background.points,
    "business_goal": background.goal,
    "users": users,
    "scope_what": scope_what,
    "scope_what_not": scope_what_not,
    "constraints": constraints,
    "benefits": benefits
})

# 4. 検証
validation = validate_compressed_summary(compressed, detailed_summary)
if not validation.is_valid:
    print("警告:", validation.warnings)

# 5. 保存
write_file("docs/00_inputs/summary_compressed.md", compressed)

# 6. メタ情報出力
tokens_detailed = count_tokens_estimate(detailed_summary)
tokens_compressed = count_tokens_estimate(compressed)
compression_ratio = tokens_compressed / tokens_detailed

print(f"✓ 圧縮版を生成しました")
print(f"  詳細版: {tokens_detailed} tokens")
print(f"  圧縮版: {tokens_compressed} tokens")
print(f"  圧縮率: {compression_ratio:.1%}")
```

---

## ファイル構成の確認

実装に必要なファイル：

```
.claude/
├─ commands/
│  └─ summarize_inputs.md      ← 更新済み（フロー定義）
├─ agents/
│  └─ summarize-compressor.md  ← 新規（圧縮エージェント定義）
└─ lib/
   ├─ compress_summary.md      ← 新規（ユーティリティ関数）
   ├─ compressed_summary_template.md ← 新規（テンプレート定義）
   └─ IMPLEMENTATION_GUIDE.md   ← このファイル
```

---

## 実装チェックリスト

### フェーズ 1: ドキュメント作成（✅ 完了）

- [x] summarize_inputs.md にフロー定義を追加
- [x] compress_summary.md にユーティリティ関数を定義
- [x] compressed_summary_template.md にテンプレートを定義
- [x] このガイドを作成

### フェーズ 2: エージェント実装（実装待ち）

- [ ] `.claude/agents/summarize-compressor.md` を作成
- [ ] セクション抽出関数を実装
- [ ] 圧縮ロジック関数を実装
- [ ] テンプレート適用関数を実装
- [ ] 検証関数を実装

### フェーズ 3: コマンド統合（実装待ち）

- [ ] @summarize_inputs のフロー内に圧縮処理を追加
- [ ] Task の並列実行設定（詳細版の生成完了後に圧縮版生成）
- [ ] エラーハンドリングを実装
- [ ] ユーザーメッセージを実装

### フェーズ 4: テスト（実装待ち）

- [ ] 既存の summary.md ファイルで圧縮版を生成
- [ ] 圧縮率が 20～30% か確認
- [ ] テンプレート構造が正しいか確認
- [ ] トークン削減効果を測定

---

## 実装上の注意事項

### 1. セクション抽出の堅牢性

Markdown の見出しが多様である可能性があるため：

```python
# キーワード辞書を使用
SECTION_KEYWORDS = {
    "background": ["背景", "背景・課題", "ビジネス背景", "状況"],
    "users": ["ユーザー", "ステークホルダー", "想定読者", "利用者"],
    "requirements": ["要件", "スコープ", "機能要件", "対象"],
    "constraints": ["制約", "前提条件", "前置き", "制限事項"],
    "benefits": ["期待効果", "メリット", "得られるもの", "ビジネスインパクト"]
}

# セクション探索時は複数キーワードを試す
def find_section(text, keyword_type):
    for keyword in SECTION_KEYWORDS[keyword_type]:
        section = extract_by_keyword(text, keyword)
        if section:
            return section
    return None  # セクション見つからない場合
```

### 2. 抽出失敗時のハンドリング

セクションが見つからない場合：

```python
if not find_section(text, "background"):
    # 警告を出力
    print("警告: ビジネス背景セクションが見つかりません")
    print("手動で以下を追加してください:")
    print("## ビジネス背景")
    print("- [背景ポイント 1]")
    print("- [背景ポイント 2]")
    print("")
    # プレースホルダーを使用
    background = ["[背景情報を入力してください]"]
```

### 3. 圧縮率の最適化

目標は 20～30% ですが、実装でズレが生じる可能性があります：

```python
# 圧縮率チェック
ratio = len(compressed) / len(detailed)
if ratio > 0.35:
    print(f"⚠️  圧縮率が高すぎます: {ratio:.1%}（目標: 20～30%）")
    print("以下を検討してください:")
    print("- 背景ポイント: 5個→3個に削減")
    print("- ユーザー: 4個→3個に削減")
elif ratio < 0.15:
    print(f"⚠️  圧縮率が低すぎます: {ratio:.1%}（目標: 20～30%）")
```

### 4. トークン数の推定

日本語トークン化は複雑なため、推定値を使用：

```python
def estimate_tokens(text):
    """日本語テキストのトークン数を推定"""
    import re

    # 日本語文字を数える
    japanese_chars = len(re.findall(r'[\u4e00-\u9fff\u3040-\u309f\u30a0-\u30ff]', text))
    japanese_tokens = japanese_chars / 3.5  # 平均 3.5 文字/トークン

    # 英数字を単語数で推定
    english_words = len(text.split())
    english_tokens = english_words / 1.3  # 平均 1.3 単語/トークン

    total = japanese_tokens + english_tokens
    return int(total)
```

---

## トラブルシューティング

### Q1: セクションが見つからない

**原因**: Markdown の見出しレベルが # ではなく ## や ### の場合

**解決策**:
```python
# 複数の見出しレベルに対応
def extract_section(text, keywords):
    for level in ['#', '##', '###']:
        for keyword in keywords:
            pattern = f"^{level} .*{keyword}"
            if re.search(pattern, text, re.MULTILINE):
                # セクション抽出
                return extract_by_pattern(text, pattern)
    return None
```

### Q2: 圧縮率がおかしい

**原因**: 単語/文字のカウント方法が異なる可能性

**解決策**:
- 生成後に必ず `validate_compressed_summary()` で確認
- テンプレート構造に問題がないか確認
- セクションの重複削除を確認

### Q3: テンプレートが崩れる

**原因**: Markdown の箇条書き・インデントのフォーマットの問題

**解決策**:
```python
# テンプレート適用時に正規化
def normalize_markdown(text):
    # 連続する空行を 1 行に統一
    text = re.sub(r'\n\n+', '\n\n', text)
    # 行末の空白を削除
    text = re.sub(r' +$', '', text, flags=re.MULTILINE)
    return text
```

---

## 動作確認方法

### 基本的な動作確認

```bash
# 既存の summary.md がある状態で実行
@summarize_inputs

# 以下の出力が期待される:
# ✓ summary.md を生成
# ✓ summary_compressed.md を生成
#   詳細版: 2,340 tokens
#   圧縮版: 586 tokens
#   圧縮率: 25.0%
```

### 詳細な確認

```bash
# 生成されたファイルを確認
cat docs/00_inputs/summary_compressed.md

# 行数・文字数を確認
wc -l docs/00_inputs/summary_compressed.md
# 出力例: 45 docs/00_inputs/summary_compressed.md（行数 30～50 が目標）

# セクションが完全か確認
grep "^##" docs/00_inputs/summary_compressed.md
# 出力例:
# ## ビジネス背景・課題
# ## 想定ユーザー/読者
# ## スコープ（WHAT/WHAT NOT）
# ## 提案/計画の要点
# ## 制約・前提条件
```

---

## 参考資料

- **仕様書**: `.claude/commands/summarize_inputs.md`
- **ユーティリティ**: `.claude/lib/compress_summary.md`
- **テンプレート**: `.claude/lib/compressed_summary_template.md`
- **Phase 3 ガイド**: `.claude/patterns/PHASE2_SUMMARY_COMPRESSION.md`

---

## 実装完了後の検証リスト

実装完了時に以下を確認してください：

```
実装完了チェック:
 ✓ @summarize_inputs 実行で詳細版と圧縮版の両方が生成される
 ✓ summary_compressed.md が正しいテンプレート構造を持つ
 ✓ 圧縮率が 20～30% の範囲内
 ✓ 行数が 30～50 行程度
 ✓ 文字数が 350～800 字程度
 ✓ すべての必須セクションが存在する
 ✓ 日本語・英語の混在がない
 ✓ メタ情報（生成日時、圧縮率）が付与されている

動作確認:
 ✓ /create_document で圧縮版 Summary が参照される
 ✓ evaluator/finalizer が圧縮版を問題なく処理できる
 ✓ コンテキスト削減効果が実現している
```

