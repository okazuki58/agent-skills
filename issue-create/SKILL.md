---
name: issue-create
description: |
  雑な要件や思考メモからGitHub Issueを生成するスキル。
  「Issueを作成して」「Issueを作って」「Issue起票して」「これをIssueにして」などのリクエストで発動。
  テキスト入力や添付ファイルから要件を抽出し、構造化されたIssueを作成する。
  記載されていない仕様は推測せず、不明点はユーザーに質問する。
---

# Issue生成

雑な要件や思考メモからGitHub Issueを生成する。

## 振る舞い

1. 入力内容を確認（テキスト入力または添付ファイル）
2. 内容を整理し、Issue用の構造に再構成
3. **記載されていない仕様は推測しない**
4. 未確定事項がある場合は、AskUserQuestionで確認
5. `gh issue create` で Issue を作成

## 出力構造

Issue本文のテンプレートは [references/output-structure.md](references/output-structure.md) を参照。

## 制約

- コードを書かない
- 技術選定・実装方針を書かない
- 提供された情報に書かれていない内容は推測しない

## 前提条件

- GitHub CLI (`gh`) がインストールされ、認証済み
- 現在のディレクトリがGitリポジトリ
- リポジトリに対してIssue作成権限がある
