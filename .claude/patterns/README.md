# パターン集（Patterns）

このディレクトリは、Document Creator の実装パターンと参照ガイドを管理します。

## ファイル構成

### `create_document_implementation.md`

**`/create_document` コマンド実装の具体的フロー**

- **対象**: メインエージェント（Haiku 4.5）
- **内容**:
  - メモリ構造の初期化
  - 標準/品質/高品質モード の実装例
  - ユーティリティ関数の実装パターン
  - テスト方法

- **関連ドキュメント**:
  - `../.claude/commands/create_document.md` （仕様）
  - `../.claude/agents/doc-drafter.md` （ドラフター出力仕様）
  - `../.claude/agents/doc-evaluator.md` （評価出力仕様）

---

## メタ化・コンテキスト最適化の実装概要

### 概念図

```
[メイン会話コンテキスト]（制限あり）
├─ ドキュメント種別・モード情報
├─ ドラフトメタデータ（JSON）    ✓ ここのみ
├─ 評価メタデータ（JSON）         ✓ ここのみ
└─ ユーザー指示

[セッションメモリ]（無制限）
├─ drafts_store = { draft_id → 本文 }
├─ evaluations_store = { draft_id → 詳細評価 }
└─ メタデータのマップ
```

### コンテキスト削減の実装ステップ

1. **Task 実行後** → JSON メタ抽出
2. **本文をメモリに** → `drafts_store[draft_id] = content`
3. **メイン会話に追加** → メタデータのみ
4. **必要時にメモリから取得** → ユーザー表示・次エージェントへの入力

### 効果

| 項目 | 削減前 | 削減後 | 削減率 |
|------|--------|--------|--------|
| ドラフト本文（2個） | 8K tokens | 0 | 100% |
| 評価詳細 | 3K tokens | 200 tokens | 93% |
| 総削減効果 | 11K tokens | 200 tokens | **98%** |

---

## 実装チェックリスト

メインエージェント（`/create_document` コマンド実装）の実装時：

### Phase 1: メモリ管理の実装

- [ ] `session` オブジェクト初期化の実装
- [ ] `drafts_store`, `evaluations_store` の辞書管理
- [ ] `extract_json_metadata()` ユーティリティの実装
- [ ] `extract_content_after_json()` ユーティリティの実装

### Phase 2: Summary 管理の実装

- [ ] コマンド開始時に `summary.md`（詳細版）をロード
- [ ] `summary_compressed.md` の存在確認（なければ警告）
- [ ] 詳細版をドラフター向けに保持
- [ ] 圧縮版をメモリで保持（evaluator/finalizer 向け）

### Phase 3: 標準モード

- [ ] ドラフト生成 Task の実装
- [ ] メタ化・メモリ保存の実装
- [ ] ユーザー表示（メモリから本文取得）
- [ ] 最終版作成・保存

### Phase 4: 品質モード

- [ ] ドラフト生成 + 評価 Task の実装（圧縮版 Summary を参照）
- [ ] 評価結果のメタ化・メモリ保存
- [ ] 合格/不合格判定ロジック
- [ ] 不合格時の再生成フロー

### Phase 5: 高品質モード

- [ ] 並列ドラフト生成（`run_in_background=True`）
- [ ] 並列タスク待機（`TaskOutput(...).wait()`）
- [ ] バッチ評価の実装（圧縮版 Summary を参照）
- [ ] 統合 Task の実装（圧縮版 Summary を参照）

### Phase 6: テスト

- [ ] メモリ隔離テスト
- [ ] メタデータ抽出テスト
- [ ] 並列実行パフォーマンステスト
- [ ] Summary 圧縮版のコンテキスト削減効果測定
- [ ] メモリクリーンアップテスト

---

## 次のステップ（オプション・長期目標）

### Phase 3（現在実装中）: Summary 圧縮

**実装内容**:
- `@summarize_inputs` の拡張 → `summary_compressed.md` の自動生成
- ドラフター以外のエージェント（evaluator, finalizer）に圧縮版を供給

**期待効果**: 評価時のコンテキスト削減 さらに 10～15%

**参照**: `../docs/00_inputs/SUMMARY_COMPRESSED_TEMPLATE.md`

### Phase 4: キャッシング最適化

**実装内容**:
- ドラフト本文の内容ハッシュ化
- 同じ入力値での再生成時のキャッシュ利用
- 評価結果のキャッシュ（同一ドラフト・同一評価基準）

**期待効果**: 再実行時のトークン削減 30～50%

### 推奨実装順序

```
✅ Phase 1: メモリ管理（完了）
✅ Phase 2: エージェント出力メタ化（完了）
⏳ Phase 3: Summary 圧縮（実装中）
  ↓
🔄 Phase 4（オプション）: キャッシング最適化
  ↓
🔄 Phase 5（オプション）: 複数モデル並列化（Opus/Sonnet との使い分け）
```

---

## トラブルシューティング

### Q: メタデータの抽出に失敗する

**A**: Task 結果の形式を確認してください。

- ドラフターは JSON メタ + Markdown 本文形式を返す必要があります
- `extract_json_metadata()` で最初の `{...}` ブロックを探しています
- JSON が複数行に渡る場合、括弧カウントで終端を判定しています

### Q: メモリ使用量が増え続ける

**A**: セッション終了時にメモリをクリアしているか確認してください。

```python
# セッション終了時
del session.drafts_store
del session.evaluations_store
```

### Q: 並列実行の効果がない

**A**: `run_in_background=True` と `TaskOutput().wait()` の組み合わせを確認してください。

```python
# ✓ 正しい
task1 = Task(..., run_in_background=True)
task2 = Task(..., run_in_background=True)
result1 = TaskOutput(task1.task_id).wait()
result2 = TaskOutput(task2.task_id).wait()

# ✗ 間違い
task1 = Task(...)  # 即座に待機（背景実行ではない）
task2 = Task(...)
```

---

## 参考資料

- **仕様書**: `../.claude/commands/create_document.md`
- **ドラフター**: `../.claude/agents/doc-drafter.md`
- **評価エージェント**: `../.claude/agents/doc-evaluator.md`
- **最終化エージェント**: `../.claude/agents/doc-finalizer.md`
- **Summary テンプレート**: `../docs/00_inputs/SUMMARY_COMPRESSED_TEMPLATE.md`

