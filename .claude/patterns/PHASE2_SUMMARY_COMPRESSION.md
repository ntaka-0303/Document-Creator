---
title: "Phase 3: Summary 圧縮版の実装ガイド"
description: "evaluator/finalizer のコンテキスト削減 + 10～15% を実現"
---

# Phase 3: Summary 圧縮版の実装ガイド

**目的**: `summary.md`（詳細版）から `summary_compressed.md`（圧縮版）を自動生成し、評価・統合エージェントのコンテキスト使用量を追加削減。

**期待効果**: evaluator/finalizer への Summary 参照で **80% 削減** → 総コンテキスト削減率 **26%**

---

## 1. 仕様概要

### Phase 1～2 との組み合わせ

```
Phase 1: ドラフト本文メモリ管理
  ├─ ドラフト本文をメモリに保持（メイン会話から除外）
  └─ 効果: 15～20% 削減

Phase 2: 評価エージェント出力メタ化
  ├─ ドラフター以外の出力をメタ化
  └─ 効果: 追加 5～10% 削減

Phase 3: Summary 圧縮版の活用  ← **今ここ**
  ├─ evaluator/finalizer が圧縮版を参照
  ├─ 詳細版: 2,000～4,000 tokens（ドラフター向け）
  ├─ 圧縮版: 400～600 tokens（評価・統合向け）
  └─ 効果: 追加 10～15% 削減
```

### ファイル構造

```
docs/00_inputs/
├─ summary.md                    # 詳細版（ドラフター用）
│  └─ 2,000～4,000 tokens
├─ summary_compressed.md         # 圧縮版（評価・統合用）← 新規生成
│  └─ 400～600 tokens
└─ SUMMARY_COMPRESSED_TEMPLATE.md # テンプレート・ガイド
```

---

## 2. Summary 圧縮版の生成ルール

### 圧縮の基本方針

**目標**: 詳細版の 20～30% に圧縮

| 詳細版セクション | 圧縮ルール | 圧縮版での表現 |
|---|---|---|
| **背景説明**（500～800w） | 要点のみ 3 ポイント | 100～150 words |
| **インタビュー内容**（複数） | 実装・投資に関連する制約のみ抽出 | 箇条書き（最大 5 項目） |
| **要件リスト**（複数個） | スコープの WHAT/WHAT NOT に集約 | 箇条書き（最大 10 項目） |
| **想定結果・メリット**（複数） | トップ 3 を厳選 | 箇条書き（最大 3 項目） |

### 圧縮版テンプレート構造

```markdown
# 圧縮サマリ

## ビジネス背景・課題（100～150 words）
- 市場/業界: [1文]
- 現状の課題: [最大3点]
- ビジネス目標: [1～2文]

## 想定ユーザー/読者
- 決定権者: [1文]
- 実行担当者: [1文]
- その他: [1文]

## スコープ（WHAT/WHAT NOT）
- 対象: [最大3項目]
- 対象外: [最大2項目]

## 提案/計画の要点
- ソリューション概要: [1～2文]
- 主なメリット: [最大3点]
- 実装方式/フェーズ: [1行箇条]
- 投資額/リソース: [1～2行]

## 制約・前提条件
- [最大3項目]
```

---

## 3. 実装パターン（@summarize_inputs コマンド拡張）

### 現在の処理フロー

```
@summarize_inputs 実行
  ↓
Step 1: docs/00_inputs/**.md を読み込み
  ↓
Step 2: 構造化サマリを作成
  ↓
Step 3: Write で docs/00_inputs/summary.md に保存
  ↓
終了
```

### 拡張後の処理フロー

```
@summarize_inputs 実行
  ↓
Step 1: docs/00_inputs/**.md を読み込み
  ↓
Step 2: 構造化サマリを作成
  ↓
Step 3: Write で docs/00_inputs/summary.md に保存（詳細版）
  ↓
Step 4: 詳細版を入力として圧縮処理実行 ← **新規追加**
  ├─ 背景 → 要点抽出
  ├─ インタビュー → 制約抽出
  ├─ 要件 → スコープ集約
  └─ 結果 → 上位3件抽出
  ↓
Step 5: Write で docs/00_inputs/summary_compressed.md に保存（圧縮版）← **新規追加**
  ↓
✓ 完了（両ファイル生成）
```

### 実装例（Pseudo-code）

```python
# Step 1-3: 詳細版生成（既存）
detailed_summary = generate_detailed_summary(input_files)
write_file("docs/00_inputs/summary.md", detailed_summary)

# Step 4-5: 圧縮版生成（新規）
compressed_summary = compress_summary(detailed_summary, {
    "background_max_words": 150,
    "constraint_max_items": 5,
    "scope_what_max": 3,
    "scope_what_not_max": 2,
    "benefits_max_items": 3,
    "constraints_max_items": 3
})

write_file("docs/00_inputs/summary_compressed.md", compressed_summary)

print("✓ summary.md を生成")
print("✓ summary_compressed.md を生成")
```

### 圧縮ロジックの詳細

```python
def compress_summary(detailed_text, limits):
    """
    詳細版 Summary を圧縮版に変換

    処理フロー:
    1. セクションごとにテキスト抽出
    2. キーワード・キーセンテンス抽出
    3. 単語数/アイテム数制限を適用
    4. テンプレート構造で再構成
    """

    compressed = {}

    # 背景セクション
    background = extract_section(detailed_text, "背景")
    compressed["background"] = extract_top_sentences(
        background,
        max_words=limits["background_max_words"]
    )

    # インタビュー→制約
    interviews = extract_section(detailed_text, "インタビュー")
    compressed["constraints"] = extract_implementation_constraints(
        interviews,
        max_items=limits["constraint_max_items"]
    )

    # 要件→スコープ
    requirements = extract_section(detailed_text, "要件")
    compressed["scope_what"] = extract_what_items(
        requirements,
        max_items=limits["scope_what_max"]
    )
    compressed["scope_what_not"] = extract_what_not_items(
        requirements,
        max_items=limits["scope_what_not_max"]
    )

    # 結果→トップメリット
    results = extract_section(detailed_text, "想定結果")
    compressed["benefits"] = extract_top_benefits(
        results,
        max_items=limits["benefits_max_items"]
    )

    return format_compressed_template(compressed)
```

---

## 4. エージェント呼び出し仕様の変更

### ドラフター（変更なし）

```
入力サマリ: docs/00_inputs/summary.md  ← 詳細版を使用
```

### 評価エージェント（**新規追加**）

```
入力サマリ: docs/00_inputs/summary_compressed.md  ← 圧縮版を使用
評価基準: docs/<doc_type>/definitions/evaluation.md
drafts: [{ draft_id, style, content }, ...]
```

**使用方法**:
- ドラフトが要件に適合しているか判定する際の参考情報
- 詳細版では冗長なため、圧縮版でスコープ確認→採点

### 統合エージェント（**新規追加**）

```
入力サマリ: docs/00_inputs/summary_compressed.md  ← 圧縮版を使用
評価メタデータ: [{ draft_id, score, improvements }, ...]
drafts: [{ draft_id, style, content }, ...]
テンプレート: docs/<doc_type>/definitions/template.md
```

**使用方法**:
- 複数ドラフトの比較・統合時の参考情報
- 要件・スコープを簡潔に参照してドラフト選別

---

## 5. コンテキスト削減の定量化

### Summary 参照のコンテキスト変化

| エージェント | 詳細版使用時 | 圧縮版使用時 | 削減率 |
|---|---|---|---|
| doc-evaluator | 3K tokens | 600 tokens | 80% |
| doc-finalizer | 3K tokens | 600 tokens | 80% |
| **合計** | **6K tokens** | **1.2K tokens** | **80%** |

### 総合削減効果（高品質モード）

```
Phase 1 + 2 適用後:        12.6K tokens
Phase 3 適用後（圧縮版）:   ?

計算:
  ドラフター ×2             7K tokens（変更なし）
  評価エージェント          2.8K → 2.2K tokens（summary 600→600）
  統合エージェント          2.8K → 2.2K tokens（summary 600→600）
  ─────────────────────────
  合計: 11.4K tokens

総削減率: (17K - 11.4K) / 17K = 33%
  ※ Phase 1+2: 26%, Phase 3: +7%
```

**最終効果**:
- Phase 1 のみ: 15～20% 削減
- Phase 1+2: 26% 削減
- Phase 1+2+3: **33% 削減** ← ここまで実装予定

---

## 6. チェックリスト（実装者向け）

### @summarize_inputs コマンド拡張

- [ ] `summarize_inputs.md` に圧縮処理の説明を追加
- [ ] 詳細版と圧縮版の粒度定義を明記
- [ ] エージェント実装に、Step 4-5 を追加
  - [ ] 圧縮ロジック関数の実装
  - [ ] テンプレート適用関数の実装
  - [ ] Write で `summary_compressed.md` に保存

### /create_document コマンド実装

- [ ] コマンド開始時に圧縮版の存在確認
- [ ] 圧縮版が見つからない場合の警告処理
- [ ] evaluator 呼び出しに圧縮版 Summary パス追加
- [ ] finalizer 呼び出しに圧縮版 Summary パス追加

### テスト

- [ ] `summary_compressed.md` が正しく生成されるか
- [ ] トークン削減効果の測定（6K → 1.2K の確認）
- [ ] ドラフト内容の評価が圧縮版でも適切か
- [ ] 統合結果に差異がないか（詳細版との比較）

---

## 7. 実装時の注意事項

### Summary 圧縮時に失うもの（許容範囲）

- ❌ 詳細なビジネスシナリオ → ✅ 背景要点のみ
- ❌ 個別のインタビュー内容 → ✅ 制約・実装上の課題のみ
- ❌ 全要件の列挙 → ✅ WHAT/WHAT NOT のみ
- ❌ 期待効果の全詳細 → ✅ トップ 3 メリットのみ

### Summary 圧縮時に保つもの（重要）

- ✅ ビジネス目標
- ✅ スコープ（対象・対象外）
- ✅ 想定ユーザー
- ✅ 制約条件
- ✅ 最重要メリット

---

## 8. 次のステップ

### Phase 4: キャッシング最適化（オプション・未実装）

- ドラフト本文のハッシュ化
- 同一入力値での再生成時のキャッシュ利用
- 再実行時の **30～50%** 削減

### Phase 5: 複数モデル並列化（オプション・未実装）

- 標準/品質モード: Haiku 継続
- 高品質モード: Opus への段階的移行
- 精度向上 + コスト最適化の両立

---

## 参考資料

- **仕様**: `.claude/commands/summarize_inputs.md`
- **テンプレート**: `docs/00_inputs/SUMMARY_COMPRESSED_TEMPLATE.md`
- **実装ガイド**: `.claude/patterns/create_document_implementation.md`
- **管理ガイド**: `.claude/patterns/README.md`

