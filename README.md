# zenn-content

Zennへの投稿用テック記事を管理するためのプロジェクトです。

## 📁 プロジェクト構成

```
.
├── articles/          # 公開用記事（Zennに投稿される）
├── books/            # 公開用本（Zennに投稿される）
├── drafts/           # 記事のドラフト（独自ディレクトリ）
├── images/           # 画像ファイル
├── .clinerules       # プロジェクトルール
├── .gitignore
├── package.json
└── README.md
```

## 🚀 セットアップ

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

## 📝 基本的な使い方

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

## 📂 ディレクトリの使い分け

- **articles/**: Zennに公開する記事。`published: true`で投稿される
- **books/**: Zennに公開する本。各章ごとにファイルを分割
- **drafts/**: 執筆中の記事やアイデア。Zennには連携されない

## 🛠️ 便利なコマンド

```bash
# プレビューの起動
npx zenn preview

# 新しい記事作成
npx zenn new:article

# 新しい本作成
npx zenn new:book

# ドラフト一覧表示
ls -la drafts/

# 未公開記事確認
grep -r "published: false" articles/
```

## 🔗 関連リンク

- [Zenn公式ドキュメント](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [Zenn CLI](https://github.com/zenn-dev/zenn-editor)
- [Markdown記法](https://zenn.dev/zenn/articles/markdown-guide)