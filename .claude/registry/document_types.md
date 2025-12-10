# ドキュメント種別レジストリ

## ドキュメント種別一覧

| 番号 | doc_type | 表示名 | ディレクトリ | デフォルトスタイル |
|------|----------|--------|--------------|-------------------|
| 1 | proposal | 提案書 | 01_proposal | structured, narrative |
| 2 | plan | 計画書 | 02_plan | structured, logical |
| 3 | requirement | 要件定義書 | 03_requirement | structured, logical |
| 4 | to-be_workflow | To-Beワークフロー | 04_to-be_workflow | structured, narrative |
| 5 | spec | 機能仕様書 | 05_spec | structured, logical |

## パス規則

- テンプレート: `docs/<ディレクトリ>/definitions/template.md`
- 評価基準: `docs/<ディレクトリ>/definitions/evaluation.md`
- 最終版出力: `docs/<ディレクトリ>/output/<filename>_<YYYYMMDD>.md`

## 新規追加時

1. 上記テーブルに行を追加
2. `docs/<番号>_<doc_type>/definitions/` に `template.md`（FILE_NAME コメント必須）と `evaluation.md` を作成
3. `docs/<番号>_<doc_type>/output/` ディレクトリを作成
