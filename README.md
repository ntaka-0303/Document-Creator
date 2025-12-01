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
│   ├── agents/                    # Claude Agentの定義
│   │   ├── doc-drafter-*.md       # 各種ドラフターエージェント
│   │   └── doc-evaluator.md       # 評価エージェント
│   └── commands/
│       └── create_document.md     # ドキュメント作成コマンド（オーケストレーション機能含む）
│
└── docs/
    ├── 00_inputs/                 # 入力情報（要件、背景など）
    ├── 01_proposal/               # 提案書関連
    │   ├── definitions/
    │   │   ├── template.md        # 提案書のテンプレート
    │   │   └── evaluation.md      # 提案書の評価基準
    │   └── output/                # 生成されたドラフト・最終版
    │       ├── draft/             # ドラフトファイル格納ディレクトリ
    │       └── <filename>_<YYYYMMDD>.md  # 最終版ファイル
    │
    ├── 02_plan/                   # 計画書関連
    │   ├── definitions/
    │   │   ├── template.md        # 計画書のテンプレート
    │   │   └── evaluation.md      # 計画書の評価基準
    │   └── output/                # 生成されたドラフト・最終版
    │       ├── draft/             # ドラフトファイル格納ディレクトリ
    │       └── <filename>_<YYYYMMDD>.md  # 最終版ファイル
    │
    ├── 03_requirement/            # 要件定義書関連
    │   ├── definitions/
    │   │   ├── template.md        # 要件定義書のテンプレート
    │   │   └── evaluation.md      # 要件定義書の評価基準
    │   └── output/                # 生成されたドラフト・最終版
    │       ├── draft/             # ドラフトファイル格納ディレクトリ
    │       └── <filename>_<YYYYMMDD>.md  # 最終版ファイル
    │
    ├── 04_to-be_workflow/         # To-Beワークフロー関連
    │   ├── definitions/
    │   │   ├── template.md        # To-Beワークフローのテンプレート
    │   │   └── evaluation.md      # To-Beワークフローの評価基準
    │   └── output/                # 生成されたドラフト・最終版
    │       ├── draft/             # ドラフトファイル格納ディレクトリ
    │       └── <filename>_<YYYYMMDD>.md  # 最終版ファイル
    │
    └── 05_spec/                   # 仕様書関連
        ├── definitions/
        │   ├── template.md        # 仕様書のテンプレート
        │   └── evaluation.md      # 仕様書の評価基準
        └── output/                # 生成されたドラフト・最終版
            ├── draft/             # ドラフトファイル格納ディレクトリ
            └── <filename>_<YYYYMMDD>.md  # 最終版ファイル
```

## エージェント構成

### コマンド
- **/create_document**: ドキュメント生成プロセス全体を管理（オーケストレーション機能を含む）

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
      - エージェントは以下をインプットとして使用します：
     - `docs/00_inputs/` 配下のすべての `.md` ファイル
     - `docs/<doc_type>/output/` 配下の最終版ファイル（`draft/` 配下を除く）
   - 過去に生成されたドキュメントがある場合、それらも参考情報として活用されます

2. **ドキュメント生成の実行**

   以下の2つの方法で実行できます：

   - **ドキュメント種別を明示的に指定する場合**（推奨）
     必ず `/create_document` コマンドを使用してください。
     ```
     /create_document spec 機能仕様書を作成してください
     ```
     ※ 先頭にドキュメント種別（`spec`, `proposal` 等）を指定すると、自動判定を行わずにその種別で確定します。

   - **ドキュメント種別を自動判定させる場合**
     チャットで直接指示を出すか、コマンドの種別指定を省略します。
     ```
     機能仕様書を作成してください
     ```
     ```
     /create_document 新規サービスの提案資料を作りたいです
     ```
     → システムが文脈から最適なドキュメント種別を自動判定します。

3. **スタイルの指定**（オプション）

   特定のスタイルを希望する場合は、以下のようなキーワードを使用：
   - 「ストーリーで」「読み物として」→ narrative
   - 「経営層向け」「インパクト」→ impact
   - 「簡潔に」「短く」→ concise
   - 「論理構成を重視」→ logical

### 利用可能なドキュメントタイプ

| doc_type | 説明 | コマンド指定例 |
|----------|------|--------|
| `proposal` | 提案書 | `/create_document proposal ...` |
| `plan` | 計画書 | `/create_document plan ...` |
| `requirement` | 要件定義書 | `/create_document requirement ...` |
| `to-be_workflow` | To-Beワークフロー | `/create_document to-be_workflow ...` |
| `spec` | 機能仕様書 | `/create_document spec ...` |

### 生成されるファイル

ドキュメント生成後、以下のファイルが `docs/<doc_type>/output/` に保存されます：

- `draft/draft-1.md`, `draft/draft-2.md`, ... : 各スタイルで生成されたドラフト（draftサブディレクトリ内）
- `<テンプレート名>_<YYYYMMDD>.md` : 最終的に採用されたドキュメント（例: 機能仕様書_20251127.md）

## ワークフロー

`/create_document` コマンドが以下のプロセスを自動的に実行します：

1. **依頼受付**: ユーザーからの依頼内容を把握
2. **doc_type決定**: 明示的指定または自動判定
3. **インプット収集**: `docs/00_inputs/` と `docs/<doc_type>/output/`（`draft/` 除く）からインプット情報を収集
4. **ドラフター選択**: ドキュメントタイプと要件に応じて適切なドラフターを選択（通常2〜3個）
5. **ドラフト生成**: 各ドラフターが独立してドラフトを生成
6. **評価**: 評価エージェントが各ドラフトをスコアリング
7. **判定**:
   - 合格案がある場合 → 最適な案を最終版として保存
   - すべて不合格の場合 → フィードバックをもとに再生成（最大2〜3回）
8. **最終出力**: 最終版を `docs/<doc_type>/output/<テンプレート名>_<YYYYMMDD>.md` に保存

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
