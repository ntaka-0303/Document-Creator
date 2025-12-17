---
title: "圧縮版 Summary テンプレート"
description: "format_compressed_summary() の出力テンプレート定義"
---

# 圧縮版 Summary テンプレート定義

このドキュメントは、`format_compressed_summary()` 関数が生成する最終的なテンプレート構造を定義します。

---

## テンプレート構造

```markdown
# 圧縮サマリ

## ビジネス背景・課題（100～150 words）

{background_points}

{business_goal}

## 想定ユーザー/読者

{users_list}

## スコープ（WHAT/WHAT NOT）

### 対象（WHAT）

{scope_what_items}

### 対象外（WHAT NOT）

{scope_what_not_items}

## 提案/計画の要点

### ソリューション概要

{solution_summary}

### 主なメリット

{benefits_list}

## 制約・前提条件

{constraints_list}

---

生成日時: {generated_at}
圧縮率: {compression_ratio}%
```

---

## 各セクションの詳細

### セクション 1: ビジネス背景・課題

```markdown
## ビジネス背景・課題（100～150 words）

- {point_1}
- {point_2}
- {point_3}

**ビジネス目標**: {goal_sentence}
```

**ルール**:
- 箇条書き: 3～5 個
- 各ポイント: 20～40 字
- 目標: 1～2 文（30～50 字）

**例**:
```markdown
## ビジネス背景・課題

- 営業データ集計に月 200 時間の手作業が必要
- ヒューマンエラーが月 5～10 件発生
- 営業判断の遅延により機会損失が発生

**ビジネス目標**: AI を活用してデータ集計の手作業を 80% 削減し、6 ヶ月以内に実装完了する。
```

### セクション 2: 想定ユーザー/読者

```markdown
## 想定ユーザー/読者

- **{role_1}**: {description}
- **{role_2}**: {description}
- **{role_3}**: {description}
```

**ルール**:
- ロール別: 3～5 個
- 各説明: 15～25 字
- 太字でロールを強調

**例**:
```markdown
## 想定ユーザー/読者

- **営業部長**: 営業チーム全体の成果を管理・報告
- **営業チーム**: 日々のデータ入力・集計作業
- **IT 部門**: システム保守・技術サポート
```

### セクション 3: スコープ（WHAT/WHAT NOT）

```markdown
## スコープ（WHAT/WHAT NOT）

### 対象（WHAT）

- {what_1}
- {what_2}
- {what_3}
- {what_4}
- {what_5}

### 対象外（WHAT NOT）

- {what_not_1}
- {what_not_2}
- {what_not_3}
```

**ルール**:
- WHAT: 3～5 個
- WHAT NOT: 2～3 個
- 各項目: 15～30 字

**例**:
```markdown
## スコープ（WHAT/WHAT NOT）

### 対象（WHAT）

- 営業データの自動集計・整形
- 週次レポートの自動生成
- 顧客セグメント別の売上分析
- 営業パイプライン可視化

### 対象外（WHAT NOT）

- CRM システムの改修
- 営業プロセスの抜本的変更
- 既存営業システムのインフラ刷新
```

### セクション 4: 提案/計画の要点

```markdown
## 提案/計画の要点

### ソリューション概要

{solution_1_2_sentences}

### 主なメリット

- {benefit_1}
- {benefit_2}
- {benefit_3}
```

**ルール**:
- ソリューション: 1～2 文（40～60 字）
- メリット: 3 個
- 各メリット: 20～40 字（可能であれば数値含む）

**例**:
```markdown
## 提案/計画の要点

### ソリューション概要

AI を活用した営業支援システムを導入し、営業データの自動集計とリアルタイム分析を実現します。

### 主なメリット

- データ集計の手作業削減：50%
- 営業意思決定の速度向上：40% 短縮
- ヒューマンエラー削減：90% 以上
```

### セクション 5: 制約・前提条件

```markdown
## 制約・前提条件

- {constraint_1}
- {constraint_2}
- {constraint_3}
- {constraint_4}
- {constraint_5}
```

**ルール**:
- 項目: 3～5 個
- 各項目: 20～40 字
- 実装・投資に関わる制約を優先

**例**:
```markdown
## 制約・前提条件

- 予算上限：500 万円
- 実装期間：6 ヶ月以内
- 既存営業システムとの連携が必須
- 営業チーム（15 名）への研修実施が必要
- 月次報告時には システム稼働状態を要する
```

---

## フッター情報

```markdown
---

生成日時: 2025-12-17T10:30:45Z
元ファイル: docs/00_inputs/summary.md
圧縮率: 25% （2,500 tokens → 625 tokens）
```

**ルール**:
- 生成日時は ISO 8601 形式
- 圧縮率は元ファイルとの比率

---

## 全体的な制約

### 文字数制限

| セクション | 最小 | 目標 | 最大 |
|---|---|---|---|
| 背景・課題全体 | 80w | 120w | 150w |
| 想定ユーザー | 60w | 90w | 120w |
| スコープ WHAT | 50w | 80w | 120w |
| スコープ WHAT NOT | 30w | 50w | 80w |
| ソリューション | 30w | 50w | 80w |
| メリット | 50w | 80w | 120w |
| 制約 | 60w | 90w | 150w |
| **合計** | **350w** | **560w** | **800w** |

### 行数目安

| セクション | 行数 |
|---|---|
| ビジネス背景・課題 | 6～10 行 |
| 想定ユーザー | 4～6 行 |
| スコープ | 8～12 行 |
| 提案の要点 | 6～10 行 |
| 制約 | 6～10 行 |
| **合計** | **30～50 行** |

---

## 生成関数の Pseudo-code

```python
def format_compressed_summary(data_dict):
    """
    圧縮データからテンプレート形式を生成

    Args:
        data_dict: {
            "background": ["point1", "point2", "point3"],
            "business_goal": "goal sentence",
            "users": [
                {"role": "role1", "desc": "description1"},
                ...
            ],
            "scope_what": ["item1", "item2", ...],
            "scope_what_not": ["item1", "item2", ...],
            "solution": "solution text",
            "benefits": ["benefit1", "benefit2", "benefit3"],
            "constraints": ["constraint1", ..., "constraint5"]
        }

    Returns:
        str: Markdown形式の圧縮版Summary
    """

    # セクション 1: 背景・課題
    section1 = "## ビジネス背景・課題（100～150 words）\n\n"
    for point in data_dict["background"]:
        section1 += f"- {point}\n"
    section1 += f"\n**ビジネス目標**: {data_dict['business_goal']}\n"

    # セクション 2: ユーザー
    section2 = "## 想定ユーザー/読者\n\n"
    for user in data_dict["users"]:
        section2 += f"- **{user['role']}**: {user['desc']}\n"

    # セクション 3: スコープ
    section3 = "## スコープ（WHAT/WHAT NOT）\n\n"
    section3 += "### 対象（WHAT）\n\n"
    for item in data_dict["scope_what"]:
        section3 += f"- {item}\n"
    section3 += "\n### 対象外（WHAT NOT）\n\n"
    for item in data_dict["scope_what_not"]:
        section3 += f"- {item}\n"

    # セクション 4: 提案
    section4 = "## 提案/計画の要点\n\n"
    section4 += "### ソリューション概要\n\n"
    section4 += f"{data_dict['solution']}\n"
    section4 += "\n### 主なメリット\n\n"
    for benefit in data_dict["benefits"]:
        section4 += f"- {benefit}\n"

    # セクション 5: 制約
    section5 = "## 制約・前提条件\n\n"
    for constraint in data_dict["constraints"]:
        section5 += f"- {constraint}\n"

    # フッター
    from datetime import datetime
    now = datetime.utcnow().isoformat() + "Z"
    footer = f"\n---\n\n生成日時: {now}\n"
    footer += f"圧縮率: {data_dict.get('compression_ratio', 'N/A')}%\n"

    # 統合
    result = "# 圧縮サマリ\n\n"
    result += section1 + "\n"
    result += section2 + "\n"
    result += section3 + "\n"
    result += section4 + "\n"
    result += section5
    result += footer

    return result
```

---

## バリデーション

生成後に以下を確認してください：

- [ ] 合計行数が 30～50 行か
- [ ] 合計文字数が 350～800 字か
- [ ] 各セクションが存在するか
- [ ] 箇条書きの数が正しいか
- [ ] 日本語・英語が混在していないか
- [ ] テンプレート構造が正しいか

