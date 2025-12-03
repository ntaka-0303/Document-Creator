# DocumentCreator プロジェクトガイド

このドキュメントは、DocumentCreator プロジェクトでエージェントを安定稼働させるための重要な情報をまとめています。

## プロジェクト概要

DocumentCreator は、AIによる高品質なドキュメント自動生成システムです。

**コアコンセプト:**
- 複数のスタイル（構造化、ナラティブ、論理的など）でドラフトを生成
- 定義された評価基準で自動評価
- 合格基準を満たすまで反復改善
- 複数のドラフトを評価基準に基づいて統合し、最終版を作成
- Write ツールを使用した明示的なファイル保存

## アーキテクチャ

### システム構成

```
/create_document コマンド（オーケストレーター）
  ↓
  ├─ doc-drafter-structured (構造化・網羅性重視)
  ├─ doc-drafter-narrative (ストーリー性重視)
  ├─ doc-drafter-impact (経営層向け・インパクト重視)
  ├─ doc-drafter-concise (簡潔・事実ベース)
  ├─ doc-drafter-logical (論理構成重視)
  ├─ doc-evaluator (評価エージェント)
  └─ doc-finalizer (最終化エージェント)
```

### 実行フロー

1. **引数解析** → doc_type と依頼内容を決定
2. **ドラフター選択** → doc_type に応じて2-3個のドラフターを選択
3. **Step1: 並列ドラフト生成** → 各ドラフターを呼び出し（独立実行）
4. **ファイル保存** → Write ツールで `docs/<doc_type>/output/draft/` に保存
5. **Step2: 評価とフィードバック抽出** → doc-evaluator で各ドラフトを評価
6. **Step3: フィードバック付き再ドラフト生成** → 各ドラフターがフィードバックをもとにドラフトを修正
7. **再評価** → 修正されたドラフトを再度評価
8. **Step4: 最終化** → 1つ以上のドラフトが pass の場合、doc-finalizer が複数のドラフトを統合
9. **最終版保存** → Write ツールで `docs/<doc_type>/output/` に保存

**注意**: すべてのドラフトが fail の場合、Step3を繰り返します（最大2-3回）。

## ディレクトリ構造

```
docs/
├── 00_inputs/              # 全ドキュメント共通の入力情報
│   └── *.md               # 要件、背景、制約など
│
├── 01_proposal/           # 提案書
├── 02_plan/               # 計画書
├── 03_requirement/        # 要件定義書
├── 04_to-be_workflow/     # To-Beワークフロー
└── 05_spec/               # 仕様書
    ├── definitions/
    │   ├── template.md    # 章立てテンプレート
    │   └── evaluation.md  # 評価基準
    └── output/
        ├── draft/         # ドラフト格納
        │   ├── draft-1.md
        │   ├── draft-2.md
        │   └── draft-3.md
        └── <filename>_<YYYYMMDD>.md  # 最終版
```

n**ドキュメント種別の正式定義**: `@.claude/registry/document_types.md`
## エージェント間の連携ルール

### 1. ドラフターエージェント

**入力:**
- ユーザー依頼の要約（テキスト）
- `@docs/00_inputs/` 配下のすべての `.md` ファイル（@ 記法で参照）
- `@docs/<doc_type>/definitions/template.md`
- （2回目以降）前回の評価フィードバック（テキスト）

**出力:**
- Markdown形式のドラフト本文のみ
- 余計な前置き・後書きは含めない
- テンプレートの章構成を遵守

**重要:**
- ドラフターはファイル保存を行わない（本文のみを返す）
- 保存は呼び出し元（create_document）が Write ツールで実行

### 2. 評価エージェント (doc-evaluator)

**入力:**
- 各ドラフトファイル（@ 記法で参照）
- `@docs/<doc_type>/definitions/evaluation.md`
- doc_type（テキスト）

**出力形式:**
```markdown
# Evaluation Result

- Document type: <doc_type>
- Target draft: Draft N (<Style>)
- Overall score: X / 5
- Pass/Fail: pass|fail

## Scores by criteria

- 観点1: X / 5
  - コメント
- 観点2: X / 5
  - コメント
...

## Improvement suggestions

- 改善ポイント1
- 改善ポイント2
...
```

**スコアリング基準:**
- 1: 非常に不十分
- 2: 不十分
- 3: 実務でギリギリ使えるレベル
- 4: 良好
- 5: 非常に優れている

### 3. 最終化エージェント (doc-finalizer)

**入力:**
- 複数の修正済みドラフトファイル（すべて、@ 記法で参照）
- 各ドラフトの評価結果ファイル（すべて、@ 記法で参照）
- `@docs/<doc_type>/definitions/evaluation.md`
- `@docs/<doc_type>/definitions/template.md`
- `@docs/00_inputs/` 配下のすべてのファイル（@ 記法で参照）
- doc_type（テキスト）

**出力:**
- Markdown形式の最終版ドキュメント本文のみ
- 余計な前置き・後書きは含めない
- テンプレートの章構成を遵守

**統合方針:**
- 各ドラフトの評価結果を分析し、各観点で優れているドラフトを特定
- 評価基準に基づいて、各章・各セクションごとに最良の内容を選択
- 複数のドラフトから良い部分を組み合わせて統合
- 用語・表記ルールを統一し、一貫性を確保

**重要:**
- 最終化エージェントはファイル保存を行わない（本文のみを返す）
- 保存は呼び出し元（create_document）が Write ツールで実行

## ファイル命名規則

### ドラフトファイル
- パス: `docs/<doc_type>/output/draft/draft-<n>.md`
- 例: `docs/05_spec/output/draft/draft-1.md`
- 連番は 1 から開始

### 最終版ファイル
- パス: `docs/<doc_type>/output/<template_filename>_<YYYYMMDD>.md`
- 例: `docs/05_spec/output/機能仕様書_20251201.md`
- `<template_filename>` は `template.md` で定義
- `<YYYYMMDD>` は保存日（8桁の数字）

## 利用可能な doc_type

| ID | 説明 | デフォルトドラフター |
|----|------|-------------------|
| `spec` | 機能仕様書 | structured, logical, concise |
| `proposal` | 提案書/PoC計画書 | structured, narrative, impact |
| `plan` | 計画書 | structured, logical, concise |
| `requirement` | 要件定義書 | structured, logical, concise |
| `to-be_workflow` | To-Beワークフロー | structured, narrative, logical |
n**重要**: 以下は要約です。実装時は必ず `@.claude/registry/document_types.md` を参照してください。

## ドラフター選択ロジック

**重要**: 以下は要約です。実装時は必ず `@.claude/registry/drafter_agents.md` を参照してください。
### ユーザーの明示的希望（最優先）

| キーワード | 使用ドラフター |
|-----------|--------------|
| 「ストーリーで」「読み物として」 | doc-drafter-narrative |
| 「経営層向け」「インパクト」 | doc-drafter-impact |
| 「簡潔に」「短く」 | doc-drafter-concise |
| 「論理構成」「Why/What/How」 | doc-drafter-logical |

**推奨:** 構造の安定性のため、可能な限り `doc-drafter-structured` を含める

## 改善ループの制御

### ループ回数
- **ユーザー指定がある場合**: その回数に従う
- **指定なしの場合**: 最大2-3回を推奨（実行者判断）

### ループ継続条件
- すべてのドラフトが `fail` の場合のみループ
- 1つでも `pass` があればループ終了し、Step4（最終化）に進む

### 最終手段
- 最大ループ回数に達しても `pass` がない場合
- 最も評価の高い案を選択
- 可能な範囲で改善を加える
- 「現時点でのベスト案」として保存

## 重要な制約事項

### ファイル操作

1. **保存処理は必ず Write ツールを使用**
   - `@save` 記法は使用しない
   - Write ツールで明示的に保存

2. **保存パスの厳守**
   - ドラフト: `docs/<doc_type>/output/draft/draft-<n>.md`
   - 最終版: `docs/<doc_type>/output/<template_filename>_<YYYYMMDD>.md`

3. **ファイル参照は @ 記法を使用**
   - 例: `@docs/00_inputs/requirements.md`
   - 例: `@docs/05_spec/definitions/template.md`

### ドキュメント作成

1. **テンプレート準拠**
   - `template.md` の章立ては基本的に変更しない
   - 小見出しの追加は可、大見出しの変更は不可

2. **入力情報の扱い**
   - `docs/00_inputs/` の内容は改変しない
   - 足りない情報はユーザーに確認

3. **評価基準の尊重**
   - `evaluation.md` の観点・合格条件を厳守
   - スコアリングは客観的に実施

## トラブルシューティング

### ドラフト生成が失敗する場合

1. **入力ファイルの確認**
   - `docs/00_inputs/` にファイルが存在するか
   - ファイルの内容が適切か

2. **テンプレートの確認**
   - `docs/<doc_type>/definitions/template.md` が存在するか
   - フォーマットが正しいか

3. **ドラフター選択の確認**
   - 選択されたドラフターが適切か
   - ユーザーの希望とマッチしているか

### 評価が正しく動作しない場合

1. **ドラフトファイルの確認**
   - `docs/<doc_type>/output/draft/` にファイルが保存されているか
   - ファイルの内容が適切か

2. **評価基準の確認**
   - `docs/<doc_type>/definitions/evaluation.md` が存在するか
   - 合格条件が明確に定義されているか

3. **出力フォーマットの確認**
   - 評価結果が規定のフォーマットに従っているか

### 保存が失敗する場合

1. **ディレクトリの確認**
   - `docs/<doc_type>/output/draft/` ディレクトリが存在するか
   - 存在しない場合は作成が必要

2. **パスの確認**
   - 相対パスではなく、プロジェクトルートからの正しいパスを使用
   - 例: `docs/05_spec/output/draft/draft-1.md`

3. **Write ツールの使用確認**
   - Write ツールを正しく呼び出しているか
   - ファイルパスとコンテンツが正しく指定されているか

## ベストプラクティス

### 1. 並列実行の活用
- 複数のドラフターは独立して動作
- 可能な限り並列で実行して効率化

### 2. エラーハンドリング
- エージェント呼び出しの失敗を適切に検出
- ユーザーにわかりやすいエラーメッセージを提供

### 3. 進捗の可視化
- ドラフト生成中、評価中などの状態をユーザーに通知
- 保存完了後は保存先パスを明示

### 4. 品質の確保
- 評価基準を満たすまで改善を繰り返す
- ただし無限ループは避ける（最大2-3回）

### 5. ユーザー体験の最適化
- 不明な点はユーザーに確認
- 曖昧な依頼は選択肢を提示して明確化

## 拡張時の注意事項

### 新しい doc_type を追加する場合

1. **レジストリファイルの更新**
   - `.claude/registry/document_types.md` のテーブルに新しい行を追加

2. **ディレクトリ構造の作成**
   ```
   docs/<XX_new_doc_type>/
   ├── definitions/
   │   ├── template.md      # FILE_NAME コメントを必ず追加
   │   └── evaluation.md
   └── output/
       └── draft/
   ```

3. **README.md の更新**（オプション）
   - 利用可能な doc_type テーブルに追加

### 新しいドラフタースタイルを追加する場合

1. **エージェント定義の作成**
   - `.claude/agents/doc-drafter-<style>.md`
   - frontmatter に `name`, `description`, `color` を定義
   - 入力・出力フォーマットを既存のドラフターと統一

2. **レジストリファイルの更新**
   - `.claude/registry/drafter_agents.md` のテーブルに新しい行を追加
   - 必要に応じてキーワードマッピングテーブルにも追加

3. **README.md の更新**（オプション）
   - ドラフター一覧に追加

**重要**: 拡張時は必ずレジストリファイルを更新してください。これが単一真実源（Single Source of Truth）です。
## まとめ

このプロジェクトの成功の鍵は：

1. **明確な責任分離**: オーケストレーター（create_document）、ドラフター、評価器、最終化器の役割を明確に分離
2. **標準化されたインターフェース**: すべてのエージェントが同じ入出力形式を使用
3. **明示的なファイル操作**: Write ツールを使用した確実な保存処理
4. **反復改善プロセス**: 評価→フィードバック→再生成のサイクル
5. **統合による品質向上**: 複数のドラフトの良い部分を統合して最終版を作成
6. **柔軟性と拡張性**: 新しい doc_type やドラフタースタイルを容易に追加可能

これらの原則を守ることで、安定した高品質なドキュメント生成を実現できます。
