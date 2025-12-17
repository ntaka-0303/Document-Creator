---
title: "create_document コマンド実装パターン（メタ化版）"
description: "コンテキスト最適化を前提とした実装の具体的なフロー"
---

# create_document コマンド実装パターン

## 概要

このドキュメントは、`create_document` コマンドの `/create_document` 実行時における、**メイン会話エージェント** の実装パターンを定義します。

- **目的**: メモリ管理・コンテキスト最適化の原則に基づいた実装
- **対象**: メインエージェント（Haiku 4.5）内での Task オーケストレーション
- **効果**: コンテキスト使用量 15～20% 削減 + 並列実行維持

---

## 1. メモリ構造初期化

コマンド実行開始時、以下のメモリマップを初期化：

```javascript
// セッション内メモリ（ローカル変数）
const session = {
  drafts_store: {},           // draft_id → draft_content
  evaluations_store: {},      // draft_id → detailed_evaluation
  draft_metadata: [],         // [{ draft_id, style, ... }]
  evaluation_metadata: [],    // [{ draft_id, score, ... }]
  summary_detailed: null,     // summary.md の内容（ドラフター用）
  summary_compressed: null,   // summary_compressed.md の内容（evaluator/finalizer用）
  mode: null,                 // "standard" | "quality" | "high-quality"
  doc_type: null
};
```

**メイン会話に追加しない** - セッション内ローカルメモリで管理

### Summary の事前ロード

```python
# コマンド実行直後
session.summary_detailed = read_file("docs/00_inputs/summary.md")
if not exists("docs/00_inputs/summary_compressed.md"):
    print("警告: 圧縮版 Summary が見つかりません")
    print("@summarize_inputs を実行してください")
    return
session.summary_compressed = read_file("docs/00_inputs/summary_compressed.md")
```

---

## 2. 標準モード実装フロー

```
┌─ ドキュメント種別選択
├─ 依頼内容確認
├─ モード選択（デフォルト: 標準）
├─ Summary 確認/生成
├─ ドラフト生成（Task）
│  └─ result.metadata → draft_metadata に追加
│  └─ result.content → drafts_store[id] に保存
├─ ドラフト内容確認（UI表示用）
│  └─ drafts_store から本文を取得して表示
├─ 最終版作成（ドラフト本文から）
└─ ファイル保存 → 終了
```

### 標準モード実装例

```python
# Step 1: Summary 確認
if not exists("docs/00_inputs/summary.md"):
    run_slash_command("@summarize_inputs")

# Step 2: ドラフト生成
print("ドラフトを生成中...")
draft_result = Task(
    subagent_type="doc-drafter",
    description="ドキュメント作成",
    prompt=f"""
    style: structured
    doc_type: {session.doc_type}
    目的: ユーザーが提示した目的
    想定読者: ユーザーが提示した読者
    入力サマリ: docs/00_inputs/summary.md
    テンプレート: docs/{session.doc_type}/definitions/template.md
    """
).wait()

# メモリに保存（メイン会話に入れない）
meta = extract_json_metadata(draft_result)
session.drafts_store[meta.draft_id] = extract_content_after_json(draft_result)
session.draft_metadata.append(meta)

# UI表示用にメタのみを会話に追加
print(f"✓ ドラフト生成完了 ({meta.word_count} words)")
print(f"  Sections: {', '.join([s.title for s in meta.sections])}")

# Step 3: ユーザー確認
print("\n---\nドラフト内容：\n---\n")
print(session.drafts_store[meta.draft_id])  # 本文表示
confirm = ask_user("このドラフトで進めますか？ (y/n): ")

if confirm != "y":
    print("キャンセルしました")
    return

# Step 4: 最終版作成（ドラフト本文から）
final_content = create_final_version(
    doc_type=session.doc_type,
    template_path=f"docs/{session.doc_type}/definitions/template.md",
    draft_content=session.drafts_store[meta.draft_id]
)

# Step 5: 保存
filename = extract_filename_from_template(...)
save_path = f"docs/{session.doc_type}/output/{filename}_{today_yyyymmdd}.md"
write_file(save_path, final_content)

print(f"✓ 保存完了: {save_path}")
```

---

## 3. 品質モード実装フロー

```
┌─ Summary 確認/生成
├─ ドラフト生成（Task）
│  └─ メタ化 + メモリ保存
├─ 評価（Task）
│  └─ メタ化 + メモリ保存
├─ 改善の判定
│  ├─ 合格 → 最終版作成へ
│  └─ 不合格 → 再生成へ
├─ 再生成（Task）
│  └─ メタ化 + メモリ保存
└─ 最終版作成 → 保存
```

### 品質モード実装例

```python
# Step 1-2: ドラフト生成（標準モードと同じ）
draft_result = Task(...).wait()
meta = extract_json_metadata(draft_result)
session.drafts_store[meta.draft_id] = extract_content_after_json(draft_result)
session.draft_metadata.append(meta)

# Step 3: 評価（メモリから本文を取得、圧縮版 Summary を参照）
print("評価中...")
eval_result = Task(
    subagent_type="doc-evaluator",
    description="ドラフト評価",
    prompt=f"""
    doc_type: {session.doc_type}
    評価基準: docs/{session.doc_type}/definitions/evaluation.md
    入力サマリ: docs/00_inputs/summary_compressed.md
    drafts: [{{
      "draft_id": {meta.draft_id},
      "style": "{meta.style}",
      "content": \"\"\"
{session.drafts_store[meta.draft_id]}
      \"\"\"
    }}]
    """
).wait()

# 評価結果をメタ化・保存
eval_meta = extract_json_metadata(eval_result)
eval_detail = extract_detailed_text(eval_result)
session.evaluation_metadata.append(eval_meta)
session.evaluations_store[meta.draft_id] = eval_detail

# Step 4: 合格判定
score_data = eval_meta.evaluations[0]
if score_data.pass:
    print(f"✓ 評価完了 (Score: {score_data.overall_score}/5, 合格)")
    final_content = create_final_version(...)
else:
    print(f"✗ 評価完了 (Score: {score_data.overall_score}/5, 不合格)")
    print("改善内容:", score_data.improvement_summary)

    # Step 5: 再生成
    print("\n再生成中...")
    improvements_text = "\n".join(score_data.improvement_summary)

    redraft_result = Task(
        subagent_type="doc-drafter",
        prompt=f"""
        ... (基本設定は同じ)
        フィードバック:
        {improvements_text}
        """
    ).wait()

    # メモリ更新
    meta_v2 = extract_json_metadata(redraft_result)
    session.drafts_store[meta_v2.draft_id] = extract_content_after_json(redraft_result)
    session.draft_metadata.append(meta_v2)

    final_content = create_final_version(..., draft_content=session.drafts_store[meta_v2.draft_id])

# Step 6: 保存
save_file(final_content)
```

---

## 4. 高品質モード実装フロー

```
┌─ Summary 確認/生成
├─ ドラフト生成（並列 ×2）
│  ├─ Task 1: structured style
│  └─ Task 2: doc_type別スタイル
│  ※ 両方とも並列実行、並列待機
│  └─ 各ドラフト → メタ化 + メモリ保存
├─ 評価（バッチ）
│  └─ メモリから両ドラフト本文を取得
│  └─ メタ化 + メモリ保存
├─ 統合（Task）
│  └─ メモリから本文 + メタ評価から評価結果を組成
│  └─ 最終版ドキュメント生成
└─ 保存
```

### 高品質モード実装例

```python
# Step 1-2: ドラフト生成（並列）
print("ドラフトを2つ並列生成中...")

# 並列 Task 1: structured
task1 = Task(
    subagent_type="doc-drafter",
    prompt=f"style: structured\n...",
    run_in_background=True
)

# 並列 Task 2: doc_type別スタイル
style2 = get_style_for_doc_type(session.doc_type)  # narrative, logical, etc.
task2 = Task(
    subagent_type="doc-drafter",
    prompt=f"style: {style2}\n...",
    run_in_background=True
)

# 両タスクを待機
result1 = TaskOutput(task1.task_id).wait()
result2 = TaskOutput(task2.task_id).wait()

# メモリに保存
for result in [result1, result2]:
    meta = extract_json_metadata(result)
    session.drafts_store[meta.draft_id] = extract_content_after_json(result)
    session.draft_metadata.append(meta)

print(f"✓ ドラフト生成完了 (Draft 1, 2)")

# Step 3: バッチ評価（圧縮版 Summary を参照）
print("評価中...")
drafts_for_eval = [
    {
        "draft_id": meta.draft_id,
        "style": meta.style,
        "content": session.drafts_store[meta.draft_id]
    }
    for meta in session.draft_metadata
]

eval_result = Task(
    subagent_type="doc-evaluator",
    prompt=f"""
    doc_type: {session.doc_type}
    評価基準: docs/{session.doc_type}/definitions/evaluation.md
    入力サマリ: docs/00_inputs/summary_compressed.md
    evaluating {len(drafts_for_eval)} drafts...
    drafts: {json.dumps(drafts_for_eval)}
    """
).wait()

# 評価メタ保存
eval_meta = extract_json_metadata(eval_result)
for eval_item in eval_meta.evaluations:
    session.evaluation_metadata.append(eval_item)
    session.evaluations_store[eval_item.draft_id] = extract_detail_for_draft(eval_result, eval_item.draft_id)

# Step 4: 統合（圧縮版 Summary を参照）
print("統合中...")
final_result = Task(
    subagent_type="doc-finalizer",
    prompt=f"""
    doc_type: {session.doc_type}
    入力サマリ: docs/00_inputs/summary_compressed.md
    drafts: [
      {{"draft_id": 1, "style": "structured", "content": "{session.drafts_store[1]}"}},
      {{"draft_id": 2, "style": "{style2}", "content": "{session.drafts_store[2]}"}}
    ]
    evaluations_meta: {json.dumps(eval_meta.evaluations)}
    テンプレート: docs/{session.doc_type}/definitions/template.md
    """
).wait()

# Step 5: 保存
final_content = extract_content_after_json(final_result)
save_file(final_content)

print("✓ 完了")
```

---

## 5. ユーティリティ関数

これらの関数はメインエージェント内で実装：

### JSON メタデータ抽出

```python
def extract_json_metadata(task_result):
    """
    Task 結果から JSON メタデータセクションを抽出

    Returns: Parsed JSON dict
    """
    import json
    text = task_result.text if hasattr(task_result, 'text') else str(task_result)

    # JSON ブロックを探す
    lines = text.split('\n')
    in_json = False
    json_lines = []

    for line in lines:
        if line.strip() == '{':
            in_json = True
        if in_json:
            json_lines.append(line)
        if line.strip() == '}' and in_json:
            break

    if json_lines:
        return json.loads('\n'.join(json_lines))
    return None

def extract_content_after_json(task_result):
    """
    Task 結果から JSON 後の Markdown 本文を抽出
    """
    text = task_result.text if hasattr(task_result, 'text') else str(task_result)
    lines = text.split('\n')

    # JSON ブロック終端を探す
    brace_count = 0
    json_end_idx = 0
    for i, line in enumerate(lines):
        for char in line:
            if char == '{':
                brace_count += 1
            elif char == '}':
                brace_count -= 1
        if brace_count == 0 and json_end_idx == 0 and i > 0:
            json_end_idx = i + 1
            break

    return '\n'.join(lines[json_end_idx:])
```

### ドラフト本文の取得と表示

```python
def display_draft(draft_id):
    """ドラフト本文をユーザーに表示"""
    content = session.drafts_store.get(draft_id)
    if content:
        print("---\n")
        print(content)
        print("\n---")
    else:
        print(f"Draft {draft_id} not found in memory")

def get_draft_summary(draft_id):
    """ドラフトのメタサマリを取得"""
    for meta in session.draft_metadata:
        if meta.draft_id == draft_id:
            return {
                "style": meta.style,
                "words": meta.word_count,
                "sections": [s.title for s in meta.sections]
            }
    return None
```

---

## 6. 注意事項

### コンテキスト管理

- ❌ ドラフト本文をメイン会話に追加しない
- ❌ 評価の詳細テキストをメイン会話に追加しない
- ✅ メタデータ（JSON）のみを会話に参照
- ✅ ユーザー表示時のみメモリから本文を取得

### メモリ安全性

- セッション終了時にメモリをクリア
- 大容量ドラフトの場合、不要な draft_id は削除
- `drafts_store` は辞書でアクセス時の O(1) 性能を確保

### 並列実行の保証

- 標準モード：Task 1個 → 結果待機
- 品質モード：Task 2個（ドラフト + 評価）※ 評価は再ドラフト後
- 高品質モード：Task 2個を並列、完了後 Task 1個（評価）、完了後 Task 1個（統合）

```
標準/品質：  ドラフト → [評価] → [再ドラフト] → 最終版
高品質：     ドラフト1 ┐ → 評価 → 統合 → 最終版
             ドラフト2 ┘
```

---

## 7. テスト方法

新規ドラフト生成機能の実装後、以下をテスト：

1. **メモリ隔離確認**
   - `session.drafts_store` に本文が保存される
   - メイン会話トークン数が 15～20% 削減される

2. **メタデータの正確性**
   - JSON メタ抽出が正確に機能
   - `word_count`, `sections` が正しい

3. **並列実行の動作**
   - 高品質モードで 2 ドラフト が同時生成される
   - 総実行時間は逐次実行の 60～70%

4. **メモリクリーンアップ**
   - セッション終了後 `session` オブジェクトが削除される

