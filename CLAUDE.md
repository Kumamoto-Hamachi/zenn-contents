# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

Zenn CLIを使った技術記事管理リポジトリ。GitHubリポジトリとZennを連携し、記事をMarkdownで管理・公開する。

## コマンド

```bash
# プレビューサーバー起動（localhost:8000）
npx zenn preview

# 新規記事作成（ランダムなslug）
npx zenn new:article

# 新規記事作成（slug指定）
npx zenn new:article --slug 記事のスラッグ

# 新規本作成
npx zenn new:book --slug 本のスラッグ

# Zenn CLIのバージョン確認
npx zenn --version
```

## ディレクトリ構成

- `articles/` - 技術記事（Markdown）
- `books/` - 本（複数章構成のコンテンツ）
- `scraps/` - スクラップ（JSON形式のメモ）

## 記事のフロントマター形式

```yaml
---
title: "記事タイトル"
emoji: "絵文字1文字"
type: "tech"  # tech: 技術記事 / idea: アイデア
topics: ["topic1", "topic2"]  # 最大5つ
published: true  # false: 下書き
---
```

## 参考リンク

- [Zenn CLIガイド](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [Zenn Markdown記法](https://zenn.dev/zenn/articles/markdown-guide)
