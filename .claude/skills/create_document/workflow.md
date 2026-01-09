# ドキュメント作成ワークフロー

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
2. **ドラフト生成**: `doc-drafter` を `style: structured` で呼び出し（**model: sonnet**） → **ドラフト情報をメタ化**
3. **自己レビュー**: ドラフターが出力時に自己チェック（評価エージェント不使用）
4. **最終版作成**: ドラフト本文を取得し、テンプレート準拠版を作成
5. **保存**: 最終版を `docs/<doc_type>/output/<filename>_<YYYYMMDD>.md` に保存

### 品質モード

1. **入力要約確認**: 同上
2. **ドラフト生成**: `doc-drafter` を `style: structured` で呼び出し（**model: sonnet**） → **メタ化**
3. **評価**: `doc-evaluator` でドラフトを評価（**model: haiku**） → **評価結果をメタ化**
4. **改善**: フィードバックを基に同ドラフターで再生成（1回のみ） → **メタ化**
5. **保存**: 同上

### 高品質モード

1. **入力要約確認**: 同上
2. **ドラフト生成**: `doc-drafter` を2回呼び出し（**model: sonnet**、`style: structured` + doc_type別スタイル、**並列実行**） → **各ドラフト情報をメタ化**
3. **評価**: `doc-evaluator` で両ドラフトをバッチ評価（**model: haiku**） → **評価結果をメタ化**
4. **統合**: `doc-finalizer` で複数ドラフト情報からの最終版作成（**model: haiku**）
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

### ドラフター（doc-drafter）への入力

**model: sonnet**

```
style: <structured | narrative | impact | concise | logical>
doc_type: <doc_type>
目的: <1-2文の要約>
想定読者: <1文>
入力サマリ: docs/00_inputs/summary.md        ← 詳細版を使用
テンプレート: docs/<doc_type>/definitions/template.md
（2回目以降）フィードバック: <改善ポイントの箇条書き>
```

**重要**:
- ファイルパスのみ渡し、内容はエージェントが `read_file` で取得
- ドラフターは **詳細版（summary.md）** を使用

### ドラフター出力のメタ化形式

エージェントが返す形式（`metadata` セクション追加）：

```json
{
  "status": "success",
  "metadata": {
    "draft_id": 1,
    "style": "structured",
    "doc_type": "proposal",
    "word_count": 2341,
    "sections": [
      { "title": "1. 背景", "words": 234 },
      { "title": "2. 課題", "words": 456 }
    ],
    "created_at": "2025-12-17T10:30:45Z"
  },
  "content": "<本文は別途保持>"
}
```

**メイン会話で保持するもの**: `metadata` のみ
**メモリで保持するもの**: `content`（draft_id をキーにしたマップ）

### 評価エージェントへの入力

**model: haiku**

```
doc_type: <doc_type>
評価基準: docs/<doc_type>/definitions/evaluation.md
入力サマリ: docs/00_inputs/summary_compressed.md   ← 圧縮版を使用
drafts: [{ draft_id: 1, style: "structured", content: "<本文>" }, ...]
```

**重要**:
- 評価エージェントは **圧縮版（summary_compressed.md）** を使用
- ドラフト内容がビジネス要件に合致しているか判定する際の参考情報として機能

### 評価エージェント出力のメタ化形式

```json
{
  "evaluations": [
    {
      "draft_id": 1,
      "style": "structured",
      "overall_score": 4,
      "pass": true,
      "criteria_scores": {
        "completeness": 4,
        "clarity": 5,
        "structure": 4
      },
      "improvements": [
        "3章に具体例を追加",
        "用語統一（「システム」→「プラットフォーム」）"
      ]
    }
  ]
}
```

**メイン会話で保持するもの**: 上記JSON形式のメタデータ
**詳細評価内容**: `evaluations_detail` マップに保持（参照必要時のみ）

### 最終化エージェントへの入力

**model: haiku**

```
doc_type: <doc_type>
入力サマリ: docs/00_inputs/summary_compressed.md   ← 圧縮版を使用
drafts: [{ draft_id: 1, style: "structured", content: "<本文>" }, ...]
evaluations_meta: [メタ化された評価結果]
テンプレート: docs/<doc_type>/definitions/template.md
```

**注意**:
- ドラフト本文はメモリマップから取得、評価メタデータのみ参照
- 統合エージェント も **圧縮版（summary_compressed.md）** を使用して、複数ドラフトを比較判定

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

## メモリ管理・コンテキスト最適化

### ドラフト本文の管理

**メイン会話コンテキストに入れない:**
- ドラフト本文そのもの（`content` フィールド）
- 評価の詳細テキスト

**メモリマップで保持:**
```javascript
// メインエージェント内メモリ
drafts_store = {
  1: "<structured ドラフト本文>",
  2: "<narrative ドラフト本文>"
}

evaluations_store = {
  1: "<draft_id=1 の詳細評価内容>",
  2: "<draft_id=2 の詳細評価内容>"
}
```

**メイン会話に含める:**
- `metadata` オブジェクトのみ（JSONメタ）
- 評価スコアと改善ポイントのサマリ（JSON）

---

## 注意事項

1. **プロセス順守**: モードで定義されたステップを省略しない
2. **ファイル保存**: 最終版のみ保存（中間ドラフトはメモリ上で管理）
3. **テンプレート準拠**: 章立ては変更しない
4. **Write ツール使用**: `@save` 記法は使用しない
5. **メタ化ルール**: ドラフト/評価内容はメモリ管理、メイン会話にはメタデータのみ
6. **model 指定**: Task ツール呼び出し時に必ず model パラメータを指定する
