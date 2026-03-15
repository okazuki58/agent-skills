---
name: dev-plan
description: |
  GitHub Issue群からClaude Code並列開発用の実行プロンプト集（docs/prompts/xxx.md）を自動生成するスキル。
  「開発計画を作成して」「実行プロンプトを生成して」「dev-planを作って」「プロンプト集を作って」
  「並列開発の計画を立てて」などのリクエストで発動。
  Issue番号リストを受け取り、依存関係のWave構造を対話で決定し、
  /gtr-workflow → 実装 → /e2e-test → /pull-request の一連の指示を各Issueごとに生成する。
---

# Dev Plan — 実行プロンプト集ジェネレーター

GitHub Issueから Claude Code 並列開発用の実行プロンプト集を生成する。

## ワークフロー

### Step 1: Issue番号リストの取得

引数からIssue番号を抽出する。以下のいずれの形式も受け付ける:

- 範囲指定: `#3-10` → #3, #4, #5, #6, #7, #8, #9, #10
- カンマ区切り: `#2, #3, #4, #5`
- スペース区切り: `2 3 4 5`
- 混合: `#2, #4-8, #13` → #2, #4, #5, #6, #7, #8, #13

`#` プレフィックスは有無どちらでもOK。

引数がない場合は AskUserQuestion で確認:
```
「対象のIssue番号を教えてください（例: #3-10, #2,#4,#5）」
```

### Step 2: Issue内容の取得

各Issueの内容を `gh issue view N` で取得し、以下を抽出:
- タイトル
- 本文（補足ドキュメント参照、既存コード参考の記述を含む）
- 完了条件（DoD）の有無

### Step 3: ブランチ名の自動生成

Issueタイトルから `feature/xxx` 形式のブランチ名を自動生成する。

ルール:
- 日本語タイトルの場合は英訳してケバブケースに変換
- 例: 「データモデル変更」→ `feature/data-model`
- 例: 「ベンダー見積依頼画面」→ `feature/vendor-quotes`

### Step 4: ベースブランチの確認

AskUserQuestion で確認:
```
options: ["develop", "main", "その他"]
```

### Step 5: Wave構造の決定

1. Issue内容から依存関係を分析し、Wave構造を提案
2. AskUserQuestion で確認・修正

提案フォーマット:
```
Wave 1（直列）: #2 → #3
Wave 2（並列）: #4, #5, #6
Wave 3（並列）: #7, #8
```

判断基準:
- DBスキーマ変更 → 最初のWave（他が依存する可能性が高い）
- UI基盤（ナビゲーション、レイアウト）→ 早いWave
- 独立した画面 → 並列可能
- 明示的な依存（「〜の画面上に追加」等）→ 後のWave

### Step 6: プロンプト生成

各Issueに対し、[references/prompt-template.md](references/prompt-template.md) のテンプレートに従ってプロンプトを生成。

Issue本文から以下を自動抽出してテンプレートに埋め込む:
- **補足ドキュメント**: `docs/SPEC-*.md` や `docs/*.md` への参照
- **既存コード参考**: `app/`, `components/` 等への参照
- 該当する記述がなければ省略

### Step 7: ファイル出力

AskUserQuestion でファイル名を確認:
```
「出力ファイル名を教えてください」
default: "docs/prompts/{プロジェクト名}.md"
```

出力形式は [references/output-example.md](references/output-example.md) を参照。

## 制約

- コードを書かない（プロンプトファイルの生成のみ）
- Issue本文に書かれていない補足情報は推測しない
- Wave構造は必ずユーザーに確認してから確定する
- `gh` CLI が認証済みであることが前提
