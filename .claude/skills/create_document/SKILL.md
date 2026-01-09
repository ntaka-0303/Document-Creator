---
name: create_document
description: >
  ドキュメントを自動生成します。提案書、計画書、要件定義書、
  To-Beワークフロー、機能仕様書に対応。
---

# ドキュメント作成スキル

詳細は [workflow.md](workflow.md) を参照。

## 対話フロー

1. ドキュメント種別選択を表示
2. 依頼内容をヒアリング
3. モード選択（標準/品質/高品質）
4. Task ツールでサブエージェントを呼び出し
5. 最終版をファイル保存

## サブエージェント呼び出し（model 指定）

### doc-drafter
- **model: sonnet**（品質重視の本文生成）
- 入力: style, doc_type, summary.md（詳細版）

### doc-evaluator
- **model: haiku**（コスト効率）
- 入力: drafts, summary_compressed.md（圧縮版）

### doc-finalizer
- **model: haiku**（コスト効率）
- 入力: drafts, evaluations_meta, summary_compressed.md（圧縮版）

### summarize-compressor
- **model: haiku**（シンプルな圧縮タスク）
