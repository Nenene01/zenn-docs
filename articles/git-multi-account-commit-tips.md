---
title: "Gitマルチアカウント運用時にプライベートアカウントでコミットしてしまう問題と対策"
emoji: "🔐"
type: "tech"
topics: ["git", "github", "cli"]
published: false
---

## はじめに

GitHubで仕事用とプライベート用など複数のアカウントを使い分けている場合、うっかり別のアカウントでコミットしてしまうことがあります。

本記事では、実際に遭遇した問題と対策方法を紹介します。

## よくある誤解: gh auth loginでコミット作者は変わらない

`gh auth login`でアカウントを切り替えても、**コミットの作者情報は変わりません**。

| コマンド | 影響範囲 |
|---------|---------|
| `gh auth login` | GitHub APIへの認証（push、PR作成など） |
| `git config user.name/email` | コミットの作者情報 |

この違いを理解していないと、「gh loginで切り替えたのに別アカウントでコミットされた」という事態が起こります。

:::message
**重要**: GitHubへのpush認証と、コミットの作者情報は完全に独立した設定です。両方を適切に管理する必要があります。
:::

## 問題: グローバル設定が意図しないリポジトリに適用される

Gitにはグローバル設定（`~/.gitconfig`）とリポジトリごとのローカル設定（`.git/config`）があります。

リポジトリにローカル設定がない場合、**グローバル設定が自動的に使われます**。

```bash
# グローバル設定の確認
git config --global user.name
git config --global user.email
```

例えば、グローバル設定がプライベートアカウントになっている状態で、仕事用リポジトリをcloneしてコミットすると、プライベートアカウントでコミットされてしまいます。

## 解決策1: リポジトリごとにローカル設定を行う

各リポジトリで明示的にユーザー設定を行います。

```bash
cd /path/to/work-repository

# このリポジトリ専用の設定
git config user.name "work-username"
git config user.email "work@example.com"

# 設定の確認
git config user.name
git config user.email
```

この設定は`.git/config`に保存され、グローバル設定より優先されます。

## 解決策2: ディレクトリごとに設定を切り替える（推奨）

`~/.gitconfig`で`includeIf`を使うと、ディレクトリごとに異なる設定ファイルを読み込めます。

```gitconfig
# ~/.gitconfig

[user]
    name = private-username
    email = private@example.com

# 仕事用ディレクトリの場合は別の設定を読み込む
[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work
```

```gitconfig
# ~/.gitconfig-work

[user]
    name = work-username
    email = work@example.com
```

これにより、`~/work/`配下のリポジトリでは自動的に仕事用アカウントが使われます。

## 間違えてコミットしてしまった場合の対処

### プッシュ前の場合

直前のコミットの作者情報を変更できます。

```bash
# 現在のローカル設定で作者を上書き
git commit --amend --reset-author --no-edit
```

### プッシュ後の場合

force pushが必要になります（チームで作業している場合は注意）。

```bash
git commit --amend --reset-author --no-edit
git push --force
```

## 確認コマンド集

```bash
# 現在のリポジトリで使われる設定を確認
git config user.name
git config user.email

# グローバル設定を確認
git config --global user.name
git config --global user.email

# 直近のコミットの作者を確認
git log -1 --format="%an <%ae>"

# 設定の優先順位を含めて確認
git config --show-origin user.name
git config --show-origin user.email
```

## まとめ

- リポジトリをcloneしたら、まず`git config user.name`と`git config user.email`を確認する
- 可能であれば`includeIf`でディレクトリごとに自動切り替えを設定する
- コミット前に`git log -1`で作者情報を確認する習慣をつける

マルチアカウント運用では、一度設定を整えておくことで事故を防げます。
