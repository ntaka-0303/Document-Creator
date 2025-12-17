---
title: "Phase 3 実装サマリ：Summary 圧縮版の自動生成"
description: "詳細版→圧縮版の自動生成機能の完全実装仕様"
---

# Phase 3 実装サマリ

## 概要

Phase 3 では、`@summarize_inputs` コマンドに **圧縮版 Summary の自動生成機能** を追加実装します。

**目標**: 評価エージェント・統合エージェントへの Summary 参照時に **80% のコンテキスト削減** を実現

---

## 実装スコープ

### 生成ファイル一覧

#### 更新されたファイル
- ✅ `.claude/commands/summarize_inputs.md` （フロー定義・実装仕様追加）
- ✅ `.claude/commands/create_document.md` （圧縮版参照の明記）
- ✅ `.claude/agents/doc-evaluator.md` （圧縮版参照の明記）
- ✅ `.claude/agents/doc-finalizer.md` （圧縮版参照の明記）
- ✅ `.claude/patterns/PHASE2_SUMMARY_COMPRESSION.md` （Phase 3 ガイド）
- ✅ `.claude/patterns/create_document_implementation.md` （実装パターン更新）
- ✅ `.claude/patterns/README.md` （チェックリスト更新）

#### 新規作成ファイル

**`.claude/lib/` ディレクトリ内：**

1. **`compress_summary.md`**
   - セクション抽出関数の仕様
   - 圧縮ロジック関数の仕様
   - テンプレート適用関数の仕様
   - 検証関数の仕様

2. **`compressed_summary_template.md`**
   - テンプレート構造の定義
   - 各セクションの詳細ルール
   - 文字数・行数制限
   - 生成関数の Pseudo-code

3. **`IMPLEMENTATION_GUIDE.md`**
   - 実装方法の詳細説明
   - エージェント定義の例
   - 実装チェックリスト
   - トラブルシューティング
   - 動作確認方法

**`.claude/` ルート内：**

4. **`PHASE3_IMPLEMENTATION_SUMMARY.md`** （このファイル）
   - Phase 3 全体のサマリ
   - 実装の流れ
   - ファイル構成

---

## 実装の流れ

### Stage 1: 詳細版 Summary 生成（既存）

```
入力: docs/00_inputs/ の全ファイル
↓
処理: セクション別サマリ生成
↓
出力: docs/00_inputs/summary.md
  約 2,000～4,000 tokens
```

### Stage 2: 圧縮版 Summary 生成（新規・当実装）

```
入力: docs/00_inputs/summary.md
↓
処理: 圧縮ロジック実行
  1. セクション抽出（背景、ユーザー、スコープ等）
  2. 圧縮ルール適用
     - 背景: 3 ポイント抽出
     - ユーザー: ロール別 3～5 個
     - スコープ: WHAT 3～5 個、WHAT NOT 2～3 個
     - 制約: 3～5 個
     - メリット: TOP 3
  3. テンプレート適用
  4. 検証（圧縮率 20～30% か確認）
↓
出力: docs/00_inputs/summary_compressed.md
  約 400～600 tokens
```

### 効果

```
詳細版：2,500 tokens
圧縮版：  625 tokens
─────────────────
削減率：  75% (25% に圧縮)

evaluator/finalizer が圧縮版を参照
→ 6K tokens → 1.2K tokens（80% 削減）
```

---

## ファイル体系

### 仕様・設計ファイル（ユーザー向け）

```
.claude/commands/
├─ summarize_inputs.md
│  ├─ 実行手順（Step 1-5）
│  ├─ 圧縮版の粒度定義
│  ├─ 圧縮ロジック詳細
│  ├─ 出力例
│  └─ 注意事項
└─ create_document.md
   ├─ エージェント呼び出し仕様
   ├─ 詳細版・圧縮版の使い分け
   └─ コンテキスト削減効果

.claude/agents/
├─ doc-evaluator.md
│  ├─ 入力に summary_compressed.md を追加
│  └─ 実行手順を更新
└─ doc-finalizer.md
   ├─ 入力に summary_compressed.md を追加
   └─ 実行手順を更新
```

### 実装ガイド（実装者向け）

```
.claude/lib/
├─ compress_summary.md
│  ├─ セクション抽出関数
│  ├─ 圧縮ロジック関数
│  ├─ テンプレート適用関数
│  ├─ 検証関数
│  └─ 実装例
│
├─ compressed_summary_template.md
│  ├─ テンプレート構造
│  ├─ 各セクションの詳細
│  ├─ 文字数・行数制限
│  ├─ 生成関数 Pseudo-code
│  └─ バリデーション
│
└─ IMPLEMENTATION_GUIDE.md
   ├─ 実装方法（パターン A・B）
   ├─ Step-by-Step 実装手順
   ├─ エージェント定義例
   ├─ 実装チェックリスト
   ├─ 注意事項
   ├─ トラブルシューティング
   ├─ 動作確認方法
   └─ 検証リスト
```

### パターン・ガイド（プロジェクト全体）

```
.claude/patterns/
├─ PHASE2_SUMMARY_COMPRESSION.md
│  ├─ 仕様概要
│  ├─ Summary 圧縮版の生成ルール
│  ├─ 実装パターン（Pseudo-code）
│  ├─ エージェント呼び出し仕様
│  ├─ コンテキスト削減の定量化
│  ├─ チェックリスト
│  └─ 注意事項
│
├─ create_document_implementation.md
│  ├─ メモリ構造に summary_* を追加
│  ├─ Summary 事前ロード処理
│  └─ 各モード実装例（圧縮版参照）
│
└─ README.md
   ├─ Phase 2 チェックリスト追加
   ├─ 推奨実装順序更新
   └─ 参考資料更新
```

---

## 実装の優先順位

### Phase 3a: 準備（ドキュメント）⏳ 現在ここ

1. ✅ `.claude/commands/summarize_inputs.md` に実装フロー追加
2. ✅ `.claude/lib/compress_summary.md` に関数仕様記述
3. ✅ `.claude/lib/compressed_summary_template.md` にテンプレート定義
4. ✅ `.claude/lib/IMPLEMENTATION_GUIDE.md` に実装ガイド作成

### Phase 3b: 実装（実装者が実施）

1. `.claude/agents/summarize-compressor.md` を作成（新規エージェント）
2. `@summarize_inputs` の実装を拡張（Stage 2 処理追加）
3. セクション抽出・圧縮ロジック関数を実装
4. テンプレート適用・検証関数を実装

### Phase 3c: テスト・検証

1. 既存の summary.md で圧縮版を生成
2. 圧縮率が 20～30% か確認
3. テンプレート構造が正しいか確認
4. /create_document で圧縮版が参照されるか確認
5. コンテキスト削減効果を測定

---

## 主要な仕様

### 圧縮ルール

| セクション | 詳細版 | 圧縮版 | 削減 |
|---|---|---|---|
| 背景・課題 | 500～800w | 100～150w | **80%** |
| ユーザー | 200～300w | 60～90w | **70%** |
| スコープ | 300～400w | 50～120w | **75%** |
| メリット | 150～200w | 50～80w | **70%** |
| 制約 | 200～300w | 60～150w | **60%** |
| **合計** | **2,000～4,000w** | **400～600w** | **80%** |

### テンプレート構造

```markdown
# 圧縮サマリ

## ビジネス背景・課題
- [背景ポイント ×3]
- **ビジネス目標**: [1～2文]

## 想定ユーザー/読者
- **[ロール1]**: [説明]
- **[ロール2]**: [説明]
- **[ロール3]**: [説明]

## スコープ（WHAT/WHAT NOT）
### 対象（WHAT）
- [項目1] × 3～5個

### 対象外（WHAT NOT）
- [項目1] × 2～3個

## 提案/計画の要点
### ソリューション概要
[1～2文]

### 主なメリット
- [メリット1]
- [メリット2]
- [メリット3]

## 制約・前提条件
- [項目1] × 3～5個
```

---

## 実装完了時の期待効果

### コンテキスト削減

```
高品質モード（2ドラフト + 評価 + 統合）

削減前: 17K tokens
Phase 1（メタ化）: 17K → 12.6K (26% 削減)
Phase 3（圧縮版）: 12.6K → 11.4K (33% 削減)

⟹ 総削減率: **33%** のコンテキスト削減
```

### 費用削減

```
Haiku の料金体系（仮定）
入力: $0.80 / 100K tokens
出力: $4.00 / 100K tokens

平均的な高品質モード実行:
  削減前: 17K tokens × $0.80 / 100K = $0.136
  削減後: 11.4K tokens × $0.80 / 100K = $0.091

1 実行あたり: $0.136 → $0.091 (33% 削減)
100 実行: $13.60 → $9.10 (26% 削減)
```

### パフォーマンス

```
@summarize_inputs 実行時間:
- Stage 1（詳細版）: 1～2 分
- Stage 2（圧縮版）: 30 秒～1 分
- 合計: 1.5～3 分

コンテキスト削減による /create_document 実行時間短縮:
- 削減前: ~3 分（高品質モード）
- 削減後: ~2.5 分（評価・統合が高速化）
- 削減効果: ~15% 時間短縮
```

---

## 次のステップ

### Phase 3 実装後

- [ ] エージェント実装（summarize-compressor）
- [ ] @summarize_inputs コマンド拡張
- [ ] テスト・検証
- [ ] ドキュメント最終化

### Phase 4（オプション・将来）

- キャッシング最適化（再実行時 30～50% 削減）
- 複数モデル並列化（Opus/Sonnet との使い分け）

---

## ドキュメント構成図

```
ユーザー（プロジェクトマネージャ等）
  ↓
【仕様理解】
  ├─ .claude/commands/summarize_inputs.md（何をするか）
  ├─ .claude/commands/create_document.md（圧縮版の使用法）
  └─ CLAUDE.md（プロジェクト概要）
  ↓
【実装者向け】
  ├─ .claude/lib/IMPLEMENTATION_GUIDE.md（実装方法）
  ├─ .claude/lib/compress_summary.md（関数仕様）
  ├─ .claude/lib/compressed_summary_template.md（テンプレート）
  ├─ .claude/patterns/PHASE2_SUMMARY_COMPRESSION.md（詳細ガイド）
  └─ .claude/patterns/create_document_implementation.md（実装パターン）
  ↓
【実装】
  ├─ summarize-compressor エージェント作成
  ├─ 圧縮ロジック関数実装
  └─ @summarize_inputs 拡張
  ↓
【テスト】
  ├─ 既存ファイルで動作確認
  ├─ 圧縮率の検証
  └─ コンテキスト削減効果の測定
```

---

## チェックリスト（最終確認用）

```
ドキュメント完成度:
 ✅ Phase 3 ガイド（PHASE2_SUMMARY_COMPRESSION.md）
 ✅ 実装ガイド（IMPLEMENTATION_GUIDE.md）
 ✅ ユーティリティ仕様（compress_summary.md）
 ✅ テンプレート定義（compressed_summary_template.md）
 ✅ コマンド仕様更新（summarize_inputs.md）
 ✅ エージェント仕様更新（doc-evaluator.md, doc-finalizer.md）

実装準備:
 ✅ 実装チェックリスト作成
 ✅ トラブルシューティング記載
 ✅ 動作確認方法記載

残作業（実装者向け）:
 ⏳ summarize-compressor エージェント実装
 ⏳ @summarize_inputs コマンド拡張
 ⏳ テスト・検証
```

---

## 参考資料

- **Phase 2 ガイド**: `.claude/patterns/PHASE2_SUMMARY_COMPRESSION.md`
- **実装ガイド**: `.claude/lib/IMPLEMENTATION_GUIDE.md`
- **ユーティリティ**: `.claude/lib/compress_summary.md`
- **テンプレート**: `.claude/lib/compressed_summary_template.md`
- **仕様書**: `.claude/commands/summarize_inputs.md`

---

**実装完了予定**: 実装者が IMPLEMENTATION_GUIDE.md に従い Phase 3b・3c を実施

