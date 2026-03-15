# git-worktree-runner

git worktree を使って、複数の Claude Code セッションで安全に並列開発するためのワークフロースキル。

## 使い方

```
> /gtr-workflow で develop ブランチから feature/auth ブランチを作成して作業して。
```

Claude Code が以下を自動実行します：

1. develop を最新化
2. `git gtr new feature/auth` で worktree を作成
3. `.gtrconfig` に従って環境をセットアップ（npm install 等）
4. worktree 内で作業開始

## 並列開発のイメージ

```
ターミナル 1: /gtr-workflow → worktree-1 (feature/api)
ターミナル 2: /gtr-workflow → worktree-2 (feature/ui)
ターミナル 3: /gtr-workflow → worktree-3 (feature/tests)
```

各 worktree は独立したディレクトリなので、ファイル競合なく同時作業できます。

## .gtrconfig

プロジェクトルートに `.gtrconfig` を置くと、worktree 作成時の初期化を自動化できます：

```ini
[copy]
include = **/.env*
include = **/CLAUDE.md

[hooks]
postCreate = npm install
```

## 前提条件

- [git-worktree-runner (gtr)](https://github.com/coderabbitai/git-worktree-runner) がインストール済み
- git リポジトリ内で実行すること
