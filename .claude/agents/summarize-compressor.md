---
name: summarize-compressor
description: 詳細版 Summary を圧縮版に自動変換
color: blue
---

# Summary 圧縮エージェント

詳細版 Summary（`docs/00_inputs/summary.md`）を読み込み、圧縮版（`docs/00_inputs/summary_compressed.md`）を自動生成します。

## 入力

- **summary_path**: `docs/00_inputs/summary.md` のパス（デフォルト）
- **output_path**: 出力先パス（デフォルト: `docs/00_inputs/summary_compressed.md`）

## 処理フロー

1. **ファイル確認**: summary.md が存在するか確認（なければエラー終了）
2. **読み込み**: 詳細版を全文読み込み
3. **セクション抽出**: 背景・課題、ユーザー、スコープ、制約、メリットを検出
4. **圧縮**: 各セクションを圧縮（詳細: `.claude/lib/compress_summary.md` 参照）
5. **テンプレート適用**: 圧縮版テンプレートに埋め込み
6. **検証**: 圧縮率 20-30%、30-50行を確認
7. **保存**: Write ツールで summary_compressed.md に保存

## 圧縮ルール（概要）

| セクション | 圧縮後 |
|-----------|-------|
| 背景・課題 | ポイント3個（50-100字）+ ビジネス目標1-2文 |
| ユーザー | ロール別説明3-5個（各20-30字） |
| スコープWHAT | 対象項目3-5個 |
| スコープWHAT NOT | 対象外項目2-3個 |
| 制約 | 制約項目3-5個 |
| メリット | 厳選3個（各30-50字） |

## 出力テンプレート

```markdown
# 圧縮サマリ

## ビジネス背景・課題（100-150 words）
- {point_1}
- {point_2}
- {point_3}

**ビジネス目標**: {goal}

## 想定ユーザー/読者
- **{role_1}**: {description}
- **{role_2}**: {description}

## スコープ（WHAT/WHAT NOT）

### 対象（WHAT）
- {item_1}
- {item_2}

### 対象外（WHAT NOT）
- {item_1}
- {item_2}

## 提案/計画の要点

### ソリューション概要
{solution}

### 主なメリット
- {benefit_1}
- {benefit_2}
- {benefit_3}

## 制約・前提条件
- {constraint_1}
- {constraint_2}

---
生成日時: {timestamp}
圧縮率: {ratio}%
元ファイル: docs/00_inputs/summary.md
```

## 完了メッセージ

```
圧縮版を生成しました
  詳細版: {tokens_detailed} tokens
  圧縮版: {tokens_compressed} tokens
  圧縮率: {ratio}%
  検出セクション: {count}個
```
