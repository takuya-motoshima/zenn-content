## はじめに

最近、相続診断をAIで解決するWebアプリケーションを開発していました。その中で「診断結果のレポートをPDF化して、弁護士事務所や税理士事務所にメール送信する」という要件があり、Node.js + Puppeteer + AWS SESで実装することになりました。

一見シンプルそうな処理ですが、実装中にいくつかの落とし穴にハマり、PDFが壊れて開けない問題に直面しました。この記事では、同じ問題で悩む方のために、実際に動作するコードと共にトラブルシューティングのポイントを解説します。

### この記事で解説すること

- PuppeteerでHTMLからPDFを生成
- 生成したPDFをAWS SESでメール添付送信
- nodemailerを使ったシンプルな実装方法

### ハマったポイント（結論）

1. **Puppeteerの`page.pdf()`は`Uint8Array`を返す** → `Buffer.from()`で変換が必須
2. **nodemailerには`@aws-sdk/client-sesv2`が必要** → レガシーな`client-ses`では動作しない

nodemailerを使えば、MIME構築やBase64エンコードを自動で処理してくれるため、コードが大幅に簡潔になります。

## 環境構築

### 必要なパッケージ

```bash
npm install puppeteer nodemailer @aws-sdk/client-sesv2
```

## 実装コード

以下のコードを `index.js` として保存すれば、すぐに動作確認できます。

```javascript
import puppeteer from 'puppeteer';
import nodemailer from 'nodemailer';
import {SESv2Client, SendEmailCommand} from '@aws-sdk/client-sesv2';

// AWS SES認証情報（実際の値に置き換えてください）
const AWS_SES_REGION = 'ap-northeast-1';  // 東京リージョン
const AWS_SES_ACCESS_KEY_ID = 'AKIA******************';
const AWS_SES_SECRET_ACCESS_KEY = '****************************************';

// メール送信設定（実際の値に置き換えてください）
const FROM_EMAIL = 'sender@example.com';    // 送信元メールアドレス
const TO_EMAIL = 'recipient@example.com';   // 送信先メールアドレス

/**
 * HTMLからPDFを生成
 *
 * ポイント: page.pdf()は Uint8Array を返すため、Buffer.from()で変換が必須
 */
async function generatePdf() {
  let browser;

  try {
    const html = `
      <!DOCTYPE html>
      <html lang="ja">
      <head>
        <meta charset="UTF-8">
        <title>サンプルレポート</title>
        <style>
          body { font-family: sans-serif; padding: 20px; }
          h1 { color: #333; }
        </style>
      </head>
      <body>
        <h1>サンプルレポート</h1>
        <p>これはPuppeteerで生成したPDFのサンプルです。</p>
      </body>
      </html>
    `;

    browser = await puppeteer.launch({
      headless: true,
      args: [
        '--no-sandbox',                    // サンドボックス無効化（Docker/CI環境で必須）
        '--disable-setuid-sandbox',        // setuidサンドボックス無効化（権限関連の問題回避）
        '--disable-dev-shm-usage',         // /dev/shmの使用を無効化（メモリ不足対策）
        '--disable-gpu',                   // GPU無効化（サーバー環境では不要）
        '--no-first-run',                  // 初回起動時の処理をスキップ（高速化）
        '--no-zygote',                     // Zygoteプロセス無効化（メモリ削減）
        '--single-process',                // シングルプロセスモード（メモリ削減、安定性向上）
        '--disable-web-security',          // 同一オリジンポリシー無効化（ローカルHTMLの読み込み許可）
        '--disable-features=VizDisplayCompositor'  // 描画最適化を無効化（安定性優先）
      ]
    });

    const page = await browser.newPage();
    await page.setContent(html, { waitUntil: 'domcontentloaded' });

    const pdfBuffer = await page.pdf({
      format: 'A4',                      // 用紙サイズ（A4, Letter, Legal等）
      printBackground: true,              // 背景色・背景画像を印刷
      margin: {                           // 余白設定
        top: '10mm',
        right: '10mm',
        bottom: '10mm',
        left: '10mm'
      },
      preferCSSPageSize: false,           // CSS @page サイズを優先しない
      displayHeaderFooter: false          // ヘッダー・フッターを非表示
    });

    // ハマりポイント①: Uint8Array → Buffer変換
    // NG: return pdfBuffer;  // Uint8Arrayのまま
    // OK: Buffer.from()で変換してから返す（nodemailerが確実に処理できる）
    return Buffer.from(pdfBuffer);
  } finally {
    if (browser) await browser.close();
  }
}

/**
 * AWS SESでメール送信（PDF添付）
 *
 * ポイント: nodemailerを使えばMIME構築とBase64エンコードが自動化される
 */
async function sendEmail({ from, to, subject, html, attachments }) {
  // AWS SESv2クライアント作成
  const sesClient = new SESv2Client({
    region: AWS_SES_REGION,
    credentials: {
      accessKeyId: AWS_SES_ACCESS_KEY_ID,
      secretAccessKey: AWS_SES_SECRET_ACCESS_KEY,
    }
  });

  // nodemailer transporter作成
  const transporter = nodemailer.createTransport({
    SES: {
      sesClient,
      SendEmailCommand
    }
  });

  // メール送信（添付ファイルは自動でBase64エンコードされる）
  await transporter.sendMail({
    from,
    to,
    subject,
    html,
    attachments: attachments || []
  });

  console.log('メール送信完了');
}

/**
 * メイン処理
 */
async function main() {
  try {
    console.log('PDF生成中...');
    const pdfBuffer = await generatePdf();
    console.log(`PDF生成完了 - サイズ: ${pdfBuffer.length} bytes`);

    console.log('メール送信中...');
    await sendEmail({
      from: FROM_EMAIL,
      to: TO_EMAIL,
      subject: 'レポートを送付します',
      html: '<p>レポートを添付しました。</p>',
      attachments: [
        { filename: 'report.pdf', content: pdfBuffer }
      ]
    });

    console.log('完了');
  } catch (error) {
    console.error('エラー:', error);
  }
}

main();
```

### 実行方法

1. コード内の定数を実際の値に置き換え:
```javascript
// AWS SES認証情報
const AWS_SES_REGION = 'ap-northeast-1';  // あなたのリージョン
const AWS_SES_ACCESS_KEY_ID = 'AKIA******************';  // ← 実際のアクセスキーに置き換え
const AWS_SES_SECRET_ACCESS_KEY = '****************************************';  // ← 実際のシークレットキーに置き換え

// メール送信設定
const FROM_EMAIL = 'sender@example.com';    // ← 送信元メールアドレス（SESで検証済み）
const TO_EMAIL = 'recipient@example.com';   // ← 送信先メールアドレス
```

2. 実行:
```bash
node index.js
```

## 重要ポイント解説

### ハマりポイント①: Uint8Array vs Buffer

```javascript
// NG: Uint8Arrayのままではnodemailerでエラーになる可能性
const pdfBuffer = await page.pdf({ ... });
// pdfBufferはUint8Array

// OK: Buffer.from()で変換してから使う
const pdfBuffer = await page.pdf({ ... });
return Buffer.from(pdfBuffer);  // これで確実にBufferとして扱える
```

### ハマりポイント②: @aws-sdk/client-sesv2が必要

```javascript
// NG: レガシーなclient-sesを使うとエラー
import {SESClient} from '@aws-sdk/client-ses';  // 動作しない

// OK: client-sesv2を使う
import {SESv2Client, SendEmailCommand} from '@aws-sdk/client-sesv2';  // 正しい
```

nodemailerの最新版は`@aws-sdk/client-sesv2`（SES API v2）を要求します。レガシーな`client-ses`を使うと「Using legacy SES configuration」エラーが発生します。

### PDF生成オプション（page.pdf）

実務で使える詳細なオプション設定:

| オプション | 値 | 説明 |
|----------|------|------|
| `format` | `'A4'` | 用紙サイズ（A4, A3, Letter, Legal, Tabloid等） |
| `printBackground` | `true` | 背景色・背景画像を印刷に含める |
| `margin.top` | `'10mm'` | 上余白（mm, cm, in, px指定可） |
| `margin.right` | `'10mm'` | 右余白 |
| `margin.bottom` | `'10mm'` | 下余白 |
| `margin.left` | `'10mm'` | 左余白 |
| `preferCSSPageSize` | `false` | CSS @page サイズを優先しない（format優先） |
| `displayHeaderFooter` | `false` | ヘッダー・フッターを非表示 |

**その他の便利なオプション**:
```javascript
{
  format: 'A4',
  printBackground: true,
  margin: { top: '10mm', right: '10mm', bottom: '10mm', left: '10mm' },
  preferCSSPageSize: false,
  displayHeaderFooter: true,           // ヘッダー・フッター表示
  headerTemplate: '<div style="font-size:10px; text-align:center; width:100%;">ヘッダー</div>',
  footerTemplate: '<div style="font-size:10px; text-align:center; width:100%;"><span class="pageNumber"></span> / <span class="totalPages"></span></div>',
  landscape: false,                    // false=縦向き, true=横向き
  scale: 1                             // 拡大縮小（0.1〜2.0）
}
```

### Puppeteer起動オプション（launch）

本番環境（Docker、EC2、Lambda等）で必要なオプション:

| オプション | 説明 | 必要性 |
|----------|------|-------|
| `--no-sandbox` | Chromeのサンドボックスを無効化 | Docker/CI環境では必須（権限不足エラー回避） |
| `--disable-setuid-sandbox` | setuidサンドボックスを無効化 | 権限関連の問題を回避 |
| `--disable-dev-shm-usage` | `/dev/shm`共有メモリの使用を無効化 | メモリ不足エラー対策（特にDocker） |
| `--disable-gpu` | GPU処理を無効化 | サーバー環境ではGPU不要、安定性向上 |
| `--no-first-run` | 初回起動時の処理をスキップ | 起動時間短縮 |
| `--no-zygote` | Zygoteプロセス生成を無効化 | メモリ使用量削減 |
| `--single-process` | シングルプロセスモードで動作 | メモリ削減、クラッシュ時の復旧が容易 |
| `--disable-web-security` | 同一オリジンポリシーを無効化 | ローカルHTMLファイルの読み込みを許可 |
| `--disable-features=VizDisplayCompositor` | 描画最適化機能を無効化 | 描画バグ回避、安定性優先 |

**推奨環境**: 上記オプションは本番環境（EC2、Docker、Lambda等）で推奨。ローカル開発では`headless: true`のみでも動作します。

## AWS SES認証情報の取得

**取得手順**:

1. **AWS IAMコンソール**にアクセス
2. **SES送信権限を持つIAMユーザー**を作成
3. **アクセスキーを発行**すると、以下のような形式で表示されます:
   - `AccessKeyId`: `AKIA******************`（20文字）
   - `SecretAccessKey`: `****************************************`（40文字）
4. コード内の定数を実際の値に置き換えます:

```javascript
const AWS_SES_REGION = 'ap-northeast-1';  // 東京リージョン
const AWS_SES_ACCESS_KEY_ID = 'AKIA******************';  // ← 実際の値に置き換え
const AWS_SES_SECRET_ACCESS_KEY = '****************************************';  // ← 実際の値に置き換え
```

**セキュリティ上の注意**:
本番環境では、認証情報をコードに直接書かず、環境変数やシークレット管理サービス（AWS Secrets Manager等）を使用してください。

## トラブルシューティング

### 「Using legacy SES configuration」エラー

**症状**: メール送信時に「Using legacy SES configuration, expecting @aws-sdk/client-sesv2」エラー

**原因**: nodemailerの最新版は`@aws-sdk/client-sesv2`（SES API v2）を要求します

**対策**:
```bash
# レガシーなパッケージをアンインストール
npm uninstall @aws-sdk/client-ses

# SESv2をインストール
npm install @aws-sdk/client-sesv2
```

```javascript
// インポートも修正
import {SESv2Client, SendEmailCommand} from '@aws-sdk/client-sesv2';

// transporterの設定
const transporter = nodemailer.createTransport({
  SES: {
    sesClient,
    SendEmailCommand  // このプロパティが重要
  }
});
```

### PDFが壊れて開けない

**症状**: メールに添付されたPDFを開こうとすると「PDFドキュメントを読み込めませんでした」エラー

**原因**: `Uint8Array` → `Buffer`変換忘れ

**対策**:
```javascript
// NG
const pdfBuffer = await page.pdf({ ... });
return pdfBuffer;  // Uint8Arrayのまま

// OK
const pdfBuffer = await page.pdf({ ... });
return Buffer.from(pdfBuffer);  // Buffer に変換
```

nodemailerは内部でBase64エンコードを自動処理してくれますが、Puppeteerが返す`Uint8Array`は`Buffer`に変換しておく方が確実です。

### Puppeteerのメモリ不足

**症状**: `Error: Protocol error (Page.printToPDF): Target closed`

**対策**: Puppeteerの起動オプションを調整（特に`--disable-dev-shm-usage`と`--single-process`が重要）

```javascript
browser = await puppeteer.launch({
  headless: true,
  args: [
    '--no-sandbox',                    // サンドボックス無効化（Docker/CI環境で必須）
    '--disable-setuid-sandbox',        // setuidサンドボックス無効化（権限関連の問題回避）
    '--disable-dev-shm-usage',         // /dev/shmの使用を無効化（メモリ不足対策）重要
    '--disable-gpu',                   // GPU無効化（サーバー環境では不要）
    '--single-process'                 // シングルプロセスモード（メモリ削減）重要
  ]
});
```

Docker環境では、コンテナのメモリ上限を引き上げるか、`--shm-size`オプションで共有メモリを増やす方法もあります：

```bash
docker run --shm-size=2gb your-image
```

## まとめ

- **Puppeteerの`page.pdf()`は`Uint8Array`を返す** → `Buffer.from()`で変換必須
- **nodemailerには`@aws-sdk/client-sesv2`が必要** → レガシーな`client-ses`では動作しない
- **nodemailerを使えばMIME構築とBase64エンコードが自動化** → コードが大幅に簡潔化

nodemailerを使うことで、手動でMIME構築やBase64エンコードを実装する必要がなくなり、コードが約50%削減されます。PDF添付メール送信の実装がずっとシンプルになります。

## 参考資料

- [Nodemailer - SES Transport](https://nodemailer.com/transports/ses/)
- [AWS SDK for JavaScript v3 - SESv2](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/client/sesv2/)
- [Puppeteer API - page.pdf()](https://pptr.dev/api/puppeteer.page.pdf)
