---
description: "ドキュメント作成コマンド（トークン最適化版）"
---

# ドキュメント作成オーケストレーター

ユーザーの依頼に応じてドキュメントを生成します。

---

## 対話フロー

### Step 1: ドキュメント種別選択

```
作成するドキュメント種別を選択してください：

1. 提案書
2. 計画書
3. 要件定義書
4. To-Beワークフロー / 業務フロー
5. 機能仕様書 / システム仕様書

番号を入力してください（1-5）:
```

番号に対応する `doc_type`: 1→`proposal`, 2→`plan`, 3→`requirement`, 4→`to-be_workflow`, 5→`spec`

### Step 2: 依頼内容の確認

- どのようなドキュメントを作成したいか確認
- 不足情報があれば質問

### Step 3: モード選択

```
実行モードを選択してください：

1. 標準モード（デフォルト・最速）
   - 1ドラフター → 自己レビュー → 完了

2. 品質モード
   - 1ドラフター → 評価 → 改善 → 完了

3. 高品質モード
   - 2ドラフター → 評価 → 統合 → 完了

番号を入力してください（1-3、デフォルトは1）:
```

---

## モード別実行フロー

### 標準モード（デフォルト）

1. **入力要約確認**: `docs/00_inputs/summary.md` がなければ `@summarize_inputs` を実行
2. **ドラフト生成**: `doc-drafter-structured` を呼び出し
3. **自己レビュー**: ドラフターが出力時に自己チェック（評価エージェント不使用）
4. **保存**: 最終版を `docs/<doc_type>/output/<filename>_<YYYYMMDD>.md` に保存

### 品質モード

1. **入力要約確認**: 同上
2. **ドラフト生成**: `doc-drafter-structured` を呼び出し
3. **評価**: `doc-evaluator` でドラフトを評価
4. **改善**: フィードバックを基に同ドラフターで再生成（1回のみ）
5. **保存**: 同上

### 高品質モード

1. **入力要約確認**: 同上
2. **ドラフト生成**: 2つのドラフターを並列実行（`structured` + doc_type別）
3. **評価**: `doc-evaluator` で両ドラフトをバッチ評価
4. **統合**: `doc-finalizer` で複数ドラフトを統合
5. **保存**: 同上

---

## ドラフター選択ルール

### doc_type別のデフォルトペア（高品質モード用）

| doc_type | ドラフター1 | ドラフター2 |
|----------|------------|------------|
| proposal | structured | narrative |
| plan | structured | logical |
| requirement | structured | logical |
| to-be_workflow | structured | narrative |
| spec | structured | logical |

### キーワードによる上書き

ユーザー発言に以下が含まれる場合、対応ドラフターを優先：
- 「ストーリー」「読み物」→ narrative
- 「経営層」「インパクト」→ impact
- 「簡潔」「短く」→ concise
- 「論理構成」「Why/What/How」→ logical

---

## エージェント呼び出し仕様

### ドラフターへの入力

```
doc_type: <doc_type>
目的: <1-2文の要約>
想定読者: <1文>
入力サマリ: docs/00_inputs/summary.md
テンプレート: docs/<doc_type>/definitions/template.md
（2回目以降）フィードバック: <改善ポイントの箇条書き>
```

**重要**: ファイルパスのみ渡し、内容はエージェントが `read_file` で取得

### 評価エージェントへの入力

```
doc_type: <doc_type>
評価基準: docs/<doc_type>/definitions/evaluation.md
drafts: [{ id: 1, style: "structured", content: "<本文>" }, ...]
```

### 最終化エージェントへの入力

```
doc_type: <doc_type>
drafts: [{ id: 1, style: "structured", content: "<本文>" }, ...]
evaluations: [{ draft_id: 1, content: "<評価結果>" }, ...]
テンプレート: docs/<doc_type>/definitions/template.md
```

---

## ファイル保存ルール

### 最終版ファイル

- パス: `docs/<doc_type>/output/<filename>_<YYYYMMDD>.md`
- `<filename>`: template.md の FILE_NAME コメントから取得
- 保存は必ず **Write ツール** を使用

### ディレクトリ対応表

| doc_type | ディレクトリ |
|----------|-------------|
| proposal | 01_proposal |
| plan | 02_plan |
| requirement | 03_requirement |
| to-be_workflow | 04_to-be_workflow |
| spec | 05_spec |

---

## 注意事項

1. **プロセス順守**: モードで定義されたステップを省略しない
2. **ファイル保存**: 最終版のみ保存（中間ドラフトはメモリ上で管理）
3. **テンプレート準拠**: 章立ては変更しない
4. **Write ツール使用**: `@save` 記法は使用しない
