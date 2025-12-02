# ドキュメント種別レジストリ

このファイルは DocumentCreator でサポートする全ドキュメント種別の定義を集約管理します。

## 使用方法

- エージェントは `@.claude/registry/document_types.md` で参照可能
- 新しいドキュメント種別を追加する場合は、このテーブルに行を追加
- 変更後は必ず `create_document.md`、`README.md`、`CLAUDE.md` の整合性を確認

## フィールド定義

| フィールド名 | 説明 |
|-------------|------|
| 番号 | ユーザー選択時の表示番号（1から開始） |
| doc_type | 内部識別子（英数字、ハイフン可） |
| 表示名 | 日本語での表示名 |
| 説明 | 詳細な用途説明 |
| ディレクトリ | `docs/` 配下のディレクトリ名 |
| テンプレートパス | `template.md` への相対パス |
| 評価基準パス | `evaluation.md` への相対パス |
| デフォルトドラフター | カンマ区切りのドラフターID（例: structured, logical） |
| 出力ファイル名 | 最終版保存時のファイル名パターン |

## ドキュメント種別一覧

| 番号 | doc_type | 表示名 | 説明 | ディレクトリ | テンプレートパス | 評価基準パス | デフォルトドラフター | 出力ファイル名 |
|------|----------|--------|------|--------------|-----------------|--------------|---------------------|----------------|
| 1 | proposal | 提案書 | 提案書 | 01_proposal | docs/01_proposal/definitions/template.md | docs/01_proposal/definitions/evaluation.md | structured, narrative, impact | proposal_toa_securities_aiworkforce.md |
| 2 | plan | 計画書 | PoC計画・プロジェクト計画 | 02_plan | docs/02_plan/definitions/template.md | docs/02_plan/definitions/evaluation.md | structured, logical, concise | 計画書_YYYYMMDD.md |
| 3 | requirement | 要件定義書 | ビジネス要件・システム要件 | 03_requirement | docs/03_requirement/definitions/template.md | docs/03_requirement/definitions/evaluation.md | structured, logical, concise | 要件定義書_YYYYMMDD.md |
| 4 | to-be_workflow | To-Beワークフロー | 改善後の業務プロセス定義 | 04_to-be_workflow | docs/04_to-be_workflow/definitions/template.md | docs/04_to-be_workflow/definitions/evaluation.md | structured, narrative, logical | To-Beワークフロー_YYYYMMDD.md |
| 5 | spec | 機能仕様書 | 機能仕様書・システム仕様書 | 05_spec | docs/05_spec/definitions/template.md | docs/05_spec/definitions/evaluation.md | structured, logical, concise | 機能仕様書_YYYYMMDD.md |

## 共通インプット情報

**すべてのドキュメント種別で共通のインプットを使用:**

- **基本インプットディレクトリ**: `docs/00_inputs/`
- **各ドキュメント種別の参考資料**: `docs/<ディレクトリ>/output/*.md`（`output/draft/` 配下は除く）
- **インプットファイルの内容**: ビジネス背景、要件、制約、前提条件など

## ディレクトリ構造テンプレート

新しいドキュメント種別を追加する際は、以下の構造を作成してください：

```
docs/
└── <番号>_<doc_type>/
    ├── definitions/
    │   ├── template.md       # 章立てテンプレート（FILE_NAME コメント必須）
    │   └── evaluation.md     # 評価基準
    └── output/
        └── draft/            # ドラフト保存先（自動生成）
```

## 出力ファイル名の命名規則

- **固有名詞を含む場合**: `template.md` の FILE_NAME コメントで明示的に定義
  - 例: `proposal_toa_securities_aiworkforce.md`
- **汎用的な場合**: `<表示名>_YYYYMMDD.md` 形式
  - 例: `機能仕様書_20251202.md`
- **YYYYMMDD**: 保存日（8桁数字）

## 拡張時の注意事項

1. テーブルに新しい行を追加
2. `docs/<番号>_<doc_type>/` ディレクトリ構造を作成
3. `template.md` の先頭に `FILE_NAME` コメントを必ず追加
4. `evaluation.md` で評価基準を明確に定義
5. デフォルトドラフターは2-3個を推奨（必ず `structured` を含めることを推奨）