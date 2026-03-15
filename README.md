# Agent Skills

Claude Code の Skills（カスタムスラッシュコマンド）集です。

Issue 起票 → 並列開発計画 → git worktree で並列実装 → PR 作成までを自動化するワークフローを提供します。

## スキル一覧

| スキル | コマンド | 説明 |
| --- | --- | --- |
| [issue-create](./issue-create/) | `/issue-create` | 雑な要件から GitHub Issue を構造化して起票 |
| [dev-plan](./dev-plan/) | `/dev-plan` | Issue 群から依存関係を分析し、Wave 構造の並列開発プロンプトを生成 |
| [git-worktree-runner](./git-worktree-runner/) | `/gtr-workflow` | git worktree で隔離環境を作り並列開発 |
| [pull-request](./pull-request/) | `/pull-request` | 変更内容を分析して PR を自動作成 |

## ワークフロー

```
/issue-create → GitHub Issues
       ↓
/dev-plan → docs/prompts/xxx.md（実行プロンプト集）
       ↓
プロンプトをコピペして複数の Claude Code セッションで実行
       ↓
/gtr-workflow → worktree 作成 → 実装 → /pull-request → PR 作成
```

## インストール

```bash
npx skills add okazuki58/agent-skills --skill <skill-name> -y
```

## 前提条件

- Claude Code がインストール済み
- GitHub CLI (`gh`) がインストール・認証済み
- git リポジトリ内で実行すること
