# CLAUDE.md

このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイダンスを提供します。

## プロジェクト概要

Zenn.dev へ技術記事を投稿するためのコンテンツ管理リポジトリです。Zenn CLI を使って記事のプレビューと管理を行い、執筆中のコンテンツは独自の `drafts/` ディレクトリで管理します。

## アーキテクチャ

### ディレクトリ構造
- **articles/**: Zenn に公開される記事（front matter で `published: true` が必要）
- **books/**: Zenn に公開される本（複数章形式）
- **drafts/**: 執筆中のコンテンツ（Zenn に同期されない独自ディレクトリ）
- **images/**: 記事で参照する画像ファイル

### コンテンツフロー
1. `drafts/` ディレクトリで初稿を執筆
2. 完成した記事を `articles/` ディレクトリに移動
3. 最終確認のため `published: false` に設定
4. 公開する際に `published: true` に変更

## よく使うコマンド

### 記事管理
```bash
# ローカルでプレビュー表示（http://localhost:8000 で起動）
npx zenn preview

# Zenn CLI で新規記事作成
npx zenn new:article

# Zenn CLI で新規本作成
npx zenn new:book

# ドラフト一覧表示
ls -la drafts/

# 未公開記事確認
grep -r "published: false" articles/
```

## 記事フォーマット

### Front Matter 構造（articles/）
```yaml
---
title: "記事タイトル"
emoji: "📝"
type: "tech"  # "tech" または "idea"
topics: ["javascript", "react", "frontend"]  # 最大5つまで
published: true  # true または false
publication_name: "publication_name"  # オプション
---
```

### ドラフト用 Front Matter（drafts/）
```yaml
---
title: "ドラフトタイトル"
target: "article"  # "article" または "book"
status: "writing"  # "planning", "writing", "reviewing", "ready"
topics: ["javascript"]
notes: "執筆メモやアイデア"
---
```

### 本の構造（books/）
各本には以下が必要:
- `books/{slug}/config.yaml` - 本の設定
- `books/{slug}/chapter-{number}.md` - 各章のファイル

## コンテンツガイドライン

### ファイル命名規則
- 記事: `{slug}.md`（12〜50文字の英数字とハイフン）
- 本: `{slug}/chapter-{number}.md`
- ドラフト: 自由形式、推奨は `draft-{topic}-{date}.md`

### 執筆基準
- 日本の技術コミュニティ向けに日本語で執筆
- 記事は最低1000文字以上
- 見出し階層を適切に保つ（H2→H3→H4）
- コードブロックには必ず言語指定（```javascript、```typescript など）
- 実行可能なサンプルコードを含める
- 画像には必ず代替テキストを設定

### 画像管理
- 画像は記事と同じディレクトリまたは `images/` ディレクトリに配置
- ファイル名は記事の slug と関連付ける
- わかりやすいファイル名を使用
- 公開コンテンツでは絶対 URL を使用（GitHub の raw URL が便利）

## 品質チェックリスト

記事公開前に以下を確認:
- [ ] タイトルが具体的で検索しやすい
- [ ] 適切な emoji を選択
- [ ] topics 配列が関連技術を正確に表現（最大5つ）
- [ ] すべてのコードブロックに言語指定
- [ ] 見出し階層が適切
- [ ] 誤字脱字がない
- [ ] すべてのリンクが正常に動作
- [ ] スマートフォンでの読みやすさを確認

## プロジェクトルール（.clinerules）

リポジトリの `.clinerules` に詳細なコンテンツガイドラインがあります:
- ドラフトのワークフロー状態管理（planning → writing → reviewing → ready）
- 機密情報、個人情報、API キーの記載を厳禁
- SEO を意識したキーワード選択
- 読者のフィードバックに基づいた定期的なコンテンツの見直しと更新
- 関連記事間の内部リンク活用

## 重要な注意事項

- **機密情報は絶対にコミットしない**: API キー、トークン、認証情報は `.env` またはセキュアな保管場所へ
- **ドラフトはローカルのみ**: `drafts/` 内のコンテンツは Zenn に同期されない
- **画像ホスティング**: 公開記事では絶対 URL を使用（例: GitHub の raw URL）
- **Zenn CLI バージョン**: 現在 `zenn-cli@^0.2.3` を使用
