# DocumentCreator

Claude Agent SDKを活用した、AIによるドキュメント自動生成システムです。複数のスタイルでドラフトを生成し、評価・改善を繰り返すことで、高品質なドキュメントを作成します。

## 概要

DocumentCreatorは、以下の機能を提供します：

- **複数スタイルでのドラフト生成**: 構造化、ナラティブ、簡潔、論理的、インパクト重視など、異なるスタイルで複数のドラフトを生成
- **自動評価**: 生成されたドラフトを定義された評価基準に基づいて自動評価
- **反復改善**: 評価結果に基づき、合格基準を満たすまで改善を繰り返し
- **柔軟なドキュメントタイプ**: 提案書、計画書、要件定義書、To-Beワークフロー、仕様書など、複数のドキュメントタイプに対応

## プロジェクト構造

```
DocumentCreator/
├── .claude/
│   └── agents/                    # Claude Agentの定義
│       ├── doc-orchestrator.md    # オーケストレーションエージェント
│       ├── doc-drafter-*.md       # 各種ドラフターエージェント
│       └── doc-evaluator.md       # 評価エージェント
│
└── docs/
    ├── 00_inputs/                 # 入力情報（要件、背景など）
    ├── 01_proposal/               # 提案書関連
    │   ├── definitions/
    │   │   ├── template.md        # 提案書のテンプレート
    │   │   └── evaluation.md      # 提案書の評価基準
    │   └── output/                # 生成されたドラフト・最終版
    │
    ├── 02_plan/                   # 計画書関連
    │   ├── definitions/
    │   │   ├── template.md        # 計画書のテンプレート
    │   │   └── evaluation.md      # 計画書の評価基準
    │   └── output/                # 生成されたドラフト・最終版
    │
    ├── 03_requirement/            # 要件定義書関連
    │   ├── definitions/
    │   │   ├── template.md        # 要件定義書のテンプレート
    │   │   └── evaluation.md      # 要件定義書の評価基準
    │   └── output/                # 生成されたドラフト・最終版
    │
    ├── 04_to-be_workflow/         # To-Beワークフロー関連
    │   ├── definitions/
    │   │   ├── template.md        # To-Beワークフローのテンプレート
    │   │   └── evaluation.md      # To-Beワークフローの評価基準
    │   └── output/                # 生成されたドラフト・最終版
    │
    └── 05_spec/                   # 仕様書関連
        ├── definitions/
        │   ├── template.md        # 仕様書のテンプレート
        │   └── evaluation.md      # 仕様書の評価基準
        └── output/                # 生成されたドラフト・最終版
```

## エージェント構成

### オーケストレーター
- **doc-orchestrator**: ドキュメント生成プロセス全体を管理

### ドラフター（5種類のスタイル）
- **doc-drafter-structured**: 構造化・網羅性重視
- **doc-drafter-narrative**: ストーリー性・可読性重視
- **doc-drafter-impact**: 経営層向け・インパクト重視
- **doc-drafter-concise**: 簡潔・事実ベース
- **doc-drafter-logical**: 論理構成（WHY/WHAT/HOW）重視

### 評価器
- **doc-evaluator**: 生成されたドラフトを評価基準に基づいて評価

## 使い方

### 基本的な使用方法

1. **入力情報の準備**
   - `docs/00_inputs/` ディレクトリに、要件や背景情報を記載したMarkdownファイルを配置

2. **ドキュメント種別の指定**

   以下の2つの方法でドキュメント種別を指定できます：

   - **明示的に指定**（推奨）
     ```
     doc_type=spec で機能仕様書を作成してください
     ```
     ```
     proposal で提案書を作成してください
     ```

   - **自動判定**
     ```
     機能仕様書を作成してください
     ```
     → システムが自動的に `spec` と判定します

3. **スタイルの指定**（オプション）

   特定のスタイルを希望する場合は、以下のようなキーワードを使用：
   - 「ストーリーで」「読み物として」→ narrative
   - 「経営層向け」「インパクト」→ impact
   - 「簡潔に」「短く」→ concise
   - 「論理構成を重視」→ logical

### 利用可能なドキュメントタイプ

| doc_type | 説明 | 使用例 |
|----------|------|--------|
| `proposal` | 提案書 / PoC計画書 | `proposal で作成して` |
| `plan` | 計画書 | `plan で作成して` |
| `requirement` | 要件定義書 | `requirement で作成して` |
| `to-be_workflow` | To-Beワークフロー | `to-be_workflow で作成して` |
| `spec` | 機能仕様書 | `doc_type=spec で作成して` |

### 生成されるファイル

ドキュメント生成後、以下のファイルが `docs/<doc_type>/output/` に保存されます：

- `draft-1.md`, `draft-2.md`, ... : 各スタイルで生成されたドラフト
- `final.md` : 最終的に採用されたドキュメント

## ワークフロー

1. **依頼受付**: ユーザーからの依頼内容を把握
2. **doc_type決定**: 明示的指定または自動判定
3. **ドラフター選択**: ドキュメントタイプと要件に応じて適切なドラフターを選択（通常2〜3個）
4. **ドラフト生成**: 各ドラフターが独立してドラフトを生成
5. **評価**: 評価エージェントが各ドラフトをスコアリング
6. **判定**:
   - 合格案がある場合 → 最適な案を最終版として保存
   - すべて不合格の場合 → フィードバックをもとに再生成（最大2〜3回）
7. **最終出力**: 最終版を `docs/<doc_type>/output/final.md` に保存

## デフォルトのドラフター組み合わせ

### 仕様書（spec）
- `doc-drafter-structured`
- `doc-drafter-logical`
- `doc-drafter-concise`（オプション）

### 提案書（proposal）
- `doc-drafter-structured`
- `doc-drafter-narrative`
- `doc-drafter-impact`

## 技術スタック

- **Claude Agent SDK**: マルチエージェントシステムの実装基盤
- **Claude Code**: エージェントの実行環境

## ライセンス

このプロジェクトはMITライセンスの下で公開されています。

## 今後の拡張予定

- 新しいドキュメントタイプの追加
- カスタムテンプレートのサポート
- 評価基準のカスタマイズ機能
- 出力フォーマットの多様化（PDF、HTML等）
