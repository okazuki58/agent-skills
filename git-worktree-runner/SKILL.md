---
name: gtr-workflow
description: Git Worktree Runner (gtr) workflow guidance for parallel development with Claude Code. Use when the user asks about git worktrees, parallel AI development, running multiple Claude sessions, managing worktrees with gtr, or setting up isolated development environments for feature work.
---

# gtr (Git Worktree Runner) Workflow

`git gtr` is a CLI tool that wraps `git worktree` with quality-of-life features for parallel AI-assisted development.

## Core Commands

```bash
git gtr new <branch>           # Create worktree
git gtr editor <branch>        # Open in editor (cursor/vscode/zed)
git gtr ai <branch>            # Start AI tool (claude/aider/codex)
git gtr go <branch>            # Print path (use: cd "$(git gtr go feature)")
git gtr run <branch> <cmd>     # Run command in worktree
git gtr rm <branch>            # Remove worktree
git gtr list                   # List all worktrees
git gtr copy <branch> -- <pattern>  # Copy files to worktree
```

Special ID `1` references the main repo: `git gtr go 1`, `git gtr ai 1`

## Installation

```bash
git clone https://github.com/coderabbitai/git-worktree-runner.git
cd git-worktree-runner
sudo ln -s "$(pwd)/bin/git-gtr" /usr/local/bin/git-gtr
```

## Initial Setup (per repository)

```bash
cd ~/your-project
git gtr config set gtr.editor.default cursor
git gtr config set gtr.ai.default claude
```

## ブランチの起点に注意（重要）

`git gtr new <branch>` はデフォルトで `gtr.defaultBranch`（通常は `develop` や `main`）からブランチを作成する。
**featureブランチから派生させたい場合は必ず `--from-current` または `--from` を指定すること。**

```bash
# 現在のブランチから作成（最もよく使う）
git gtr new feature/child-branch --from-current

# 特定のブランチから作成
git gtr new feature/child-branch --from feature/parent-branch

# NG: featureブランチにいてもデフォルトではdevelopから作成されてしまう
git gtr new feature/child-branch  # → developベースになる！
```

## Parallel Development Workflow

### Single Feature with Claude Code

```bash
git gtr new feature-auth
git gtr ai feature-auth
# Claude Code runs in isolated worktree
# Main repo remains untouched
git gtr rm feature-auth  # Cleanup when done
```

### Multiple Parallel Claude Sessions

```bash
# Terminal 1: API work
git gtr new feature-api
git gtr ai feature-api

# Terminal 2: UI work
git gtr new feature-ui
git gtr ai feature-ui

# Terminal 3: Tests
git gtr new feature-tests
git gtr ai feature-tests
```

Each session has isolated context - no stashing, no branch switching.

### Same Branch, Multiple Worktrees

Use `--force` with `--name` for parallel work on the same feature:

```bash
git gtr new feature-auth
git gtr new feature-auth --force --name backend
git gtr new feature-auth --force --name frontend
# Creates: feature-auth/, feature-auth-backend/, feature-auth-frontend/
# All on same branch - coordinate commits carefully
```

### PR Review While Working

```bash
# Working on feature
git gtr new my-feature
git gtr ai my-feature

# Need to review PR - create separate worktree
git gtr new review-pr-123 --from origin/pr-branch
git gtr editor review-pr-123
# Review without interrupting Claude session
git gtr rm review-pr-123
```

## Configuration

### Team-Shared Config (.gtrconfig)

Create `.gtrconfig` in repo root:

```ini
[copy]
include = **/.env.example
include = **/CLAUDE.md
exclude = **/.env

[copy]
includeDirs = node_modules
excludeDirs = node_modules/.cache

[hooks]
postCreate = npm install

[defaults]
editor = vscode
ai = claude
```

### Copy Files to Worktrees

```bash
# Add patterns to config
git gtr config add gtr.copy.include "**/.env.example"
git gtr config add gtr.copy.include "**/CLAUDE.md"

# Copy node_modules to avoid npm install
git gtr config add gtr.copy.includeDirs "node_modules"
```

### Post-Create Hooks

```bash
git gtr config add gtr.hook.postCreate "npm install"
git gtr config add gtr.hook.postCreate "cp .env.example .env"
```

## Branch Name Mapping

Slashes and special characters become hyphens:
- `feature/user-auth` → folder `feature-user-auth`
- `fix/bug#123` → folder `fix-bug-123`

## Worktree作成前の確認

### 1. `.gtrconfig` の確認
worktree作成前に、プロジェクトルートに `.gtrconfig` が存在するか確認する。
なければ以下の内容で作成し、コミットする:

```ini
[copy]
include = **/.env*
include = **/CLAUDE.md

[hooks]
postCreate = npm install
```

これにより、worktree作成時に自動で `npm install` が実行される。

### 2. ベースブランチの確認
現在のブランチが `develop` / `main` 以外（featureブランチ等）の場合、
worktree作成時に **必ず `--from-current`** を付けること。
付け忘れると `develop` ベースになり、featureブランチの変更が含まれない。

```bash
# featureブランチから派生する場合
git gtr new feature/child --from-current
```

## Best Practices

1. **One worktree per task** - Keep Claude sessions focused
2. **Copy CLAUDE.md** - Include project context in each worktree
3. **Use hooks** - Automate dependency installation
4. **Clean up regularly** - `git gtr rm` when PRs merge
5. **Use `git gtr list`** - Track active worktrees

## Troubleshooting

```bash
git gtr doctor          # Health check
git gtr adapter         # List available editors/AI tools
git gtr clean           # Remove stale worktrees
git gtr clean --merged  # Remove worktrees for merged PRs (requires gh CLI)
```