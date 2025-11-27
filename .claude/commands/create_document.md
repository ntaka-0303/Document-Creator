---
description: "DocumentCreator: ドキュメント種別と依頼内容を識別し、doc-orchestrator へ処理を引き継ぐコマンド"
agent: doc-orchestrator
---

ユーザーからのコマンド入力 `$ARGUMENTS` を解析し、ドキュメント種別の指定有無を識別したうえで、
**あなたの（doc-orchestrator としての）** ドキュメント生成プロセスを開始してください。

## 1. コマンドによる引数解析

以下の手順で `$ARGUMENTS` を解析し、**ドキュメント種別 (`target_doc_type`)** と **依頼内容 (`request_content`)** を確定させてください。

1. `$ARGUMENTS` の先頭トークンを確認する。
2. 先頭トークンが以下のいずれかに **完全一致** する場合:
   - `spec`
   - `proposal`
   - `plan`
   - `requirement`
   - `to-be_workflow`

   → **種別指定あり** とみなす。
     - `target_doc_type` = その先頭トークン
     - `request_content` = 先頭トークンを取り除いた残りの文字列

3. 先頭トークンが上記に一致しない場合:
   → **種別指定なし（自動判定対象）** とみなす。
     - `target_doc_type` = `null` (未指定)
     - `request_content` = `$ARGUMENTS` 全体

## 2. オーケストレーターへの引き継ぎ指示

あなたはこれより、解析された `target_doc_type` と `request_content` を入力として、
**オーケストレーションエージェント (`doc-orchestrator`) の役割** を実行してください。

### 実行ルール

1. **`target_doc_type` が指定されている場合**
   - あなたの「優先順位1: 明示的な指定を確認」のルールに従い、
     この `target_doc_type` を **確定したドキュメント種別** として扱ってください。
   - 決して自動判定で上書きしないでください。

2. **`target_doc_type` が `null` の場合**
   - あなたの「優先順位2: 自動判定」のルールに従い、
     `request_content` の内容からドキュメント種別を **自動判定** してください。

3. **以降のプロセス**
   - 種別決定後は、あなたのエージェント定義にあるとおり、
     ドラフター選択 → ドラフト生成 → 評価 → 最終版保存 のフローを実行してください。

---

### 解析対象の入力引数

`$ARGUMENTS`


