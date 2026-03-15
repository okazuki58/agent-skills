# dev-plan

GitHub Issue 群から依存関係を分析し、Claude Code 並列開発用の実行プロンプト集を自動生成するスキル。

## 使い方

単体の Issue でも、複数の Issue でも使えます。

```
# 単体のIssue
> /dev-plan #465

# 複数のIssue（範囲指定）
> /dev-plan #467-469

# 複数のIssue（カンマ区切り）
> /dev-plan #2, #3, #4

# 複数のIssue（スペース区切り）
> /dev-plan 2 3 4

# 混合
> /dev-plan #2, #4-8, #13
```

## 処理の流れ

1. 各 Issue の内容を `gh issue view` で取得
2. 依存関係を分析し、Wave 構造（実行順序）を提案
3. ユーザーに確認・修正してもらう
4. 各 Issue の実行プロンプトを生成
5. `docs/prompts/xxx.md` に出力

## 出力例

```markdown
# PDF見積書出力 — Claude Code 実行プロンプト

## 実行順序
Wave 1（直列）: #467 → #468 → #469

---

### #467 Project → Client 紐づけ機能

/gtr-workflow で develop ブランチから feature/project-client-linking
ブランチを作成して作業して。

まず gh issue view 467 でIssue内容を読んで。
...
```

このプロンプトを Claude Code にコピペするだけで、実装 → テスト → PR 作成が自動で走ります。

## Wave 構造の判断基準

| パターン | 判断 |
| --- | --- |
| DB スキーマ変更 | 最初の Wave |
| UI 基盤（ナビ、レイアウト） | 早い Wave |
| 独立した画面・機能 | 並列可能（同じ Wave） |
| 明示的な依存関係 | 後の Wave |

## 前提条件

- GitHub CLI (`gh`) がインストール・認証済み
- 対象の Issue が作成済みであること
