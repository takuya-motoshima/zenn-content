# zenn-content

Zennへの投稿用テック記事を管理するためのプロジェクトです。

## プロジェクト構成

```
.
├── articles/                           # 公開用記事（Zennに投稿される）
│   ├── ec98e0a2e5e53b.md              # Docker入門#1 - Windowsでの環境構築
│   ├── 0028976d074b6f.md              # Docker入門#2 - Hello Worldアプリケーション作成
│   ├── 6041ff6a4ae654.md              # AI駆動で爆速アプリ開発（Cursor + Cline）
│   ├── d2a102316f23fd.md              # Claude Code × Cursor で超高速アプリ開発
│   ├── ec4dd399688d20.md              # スマホで体験しながら学ぶDID/VC
│   └── d88c396a07b5e4.md              # WindowsでTeraTermからWarpに移行した話（執筆中）
├── books/                              # 公開用本（Zennに投稿される）
├── drafts/                             # 記事のドラフト（独自ディレクトリ）
├── images/                             # 画像ファイル
├── .clinerules                         # プロジェクトルール
├── .gitignore
├── package.json
└── README.md
```

## セットアップ

### 1. リポジトリのクローン
```bash
git clone https://github.com/your-username/zenn-content.git
cd zenn-content
```

### 2. 依存関係のインストール
```bash
npm install
```

### 3. Zenn CLIの初期化（初回のみ）
```bash
npx zenn init
```

## 基本的な使い方

### 記事の作成

#### ドラフトから始める場合
```bash
# drafts/ディレクトリで執筆開始
touch drafts/draft-topic-name.md

# 完成後、articles/に移動
mv drafts/draft-topic-name.md articles/topic-name.md
```

#### 直接記事を作成する場合
```bash
# Zenn CLIで新規記事作成
npx zenn new:article

# または手動作成
touch articles/article-slug.md
```

### 本の作成
```bash
# Zenn CLIで新規本作成
npx zenn new:book

# 手動でディレクトリ作成
mkdir books/book-slug
touch books/book-slug/config.yaml
```

### プレビュー
```bash
npx zenn preview
```

## ディレクトリの使い分け

- **articles/**: Zennに公開する記事。`published: true`で投稿される
- **books/**: Zennに公開する本。各章ごとにファイルを分割
- **drafts/**: 執筆中の記事やアイデア。Zennには連携されない

## 関連リンク

- [Zenn公式ドキュメント](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [Zenn CLI](https://github.com/zenn-dev/zenn-editor)
- [Markdown記法](https://zenn.dev/zenn/articles/markdown-guide)