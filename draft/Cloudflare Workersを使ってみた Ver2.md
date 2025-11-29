## 背景

Cloudflare Workersは、サーバーレス環境で高速かつセキュアにコードを実行できるプラットフォームです。特に個人開発やPoC（Proof of Concept）での利用に適しており、UIも見やすく、設定もシンプルな点が魅力です。本記事では、前回の内容を踏まえつつ、より実践的な手順と改善点を解説します。

## 事前準備

- Cloudflareアカウントの作成
- Cloudflare Workersの使用権限確保
- 必要なNPMモジュールのインストール（前回と同様）
- Slack Appのインストールと設定（APIトークン取得）

### Slackアプリの準備

SlackのAPIを利用するために、Slack Appを作成し必要なボットトークンと権限を設定してください。これにより、後述の環境変数設定やAPI呼び出しが可能になります。

## 手順

### 1. プロジェクトのセットアップ

前回と同様に、`npm`や`npx`を使ってプロジェクトの雛形を作成します。

```bash
npm init -y
npm install @cloudflare/workers-types
```

### 2. コード例と型定義の修正

APIレスポンスの型定義を正確にします。次のようにインターフェースを調整してください。

```typescript
interface ApiResponse {
  choices?: Array<{
    message?: {
      content?: string;
    };
  }>;
}
```

そして、`json()`の結果に型アノテーションを付けて、安全にアクセスします。

```typescript
const data: ApiResponse = await response.json();

// 安全にアクセス
const content = data.choices?.[0]?.message?.content ?? '';
```

### 3. 環境変数の設定

ローカルで動作確認を行う場合は`.env`ファイルに環境変数を設定してください。

```bash
# .env
SLACK_BOT_TOKEN=your-slack-bot-token
```

Cloudflare上で動作させる場合は、「Workers KV」や「Secrets」を利用してシークレットとテキストを分けて設定しましょう。この場合、設定にダブルクォートは不要です。

### 4. Slack Appのインストール操作

SlackのAPIを呼び出すために、必ずSlack Appのインストールと適切な権限付与を行います。環境変数に設定したトークンをコード内で利用してください。

```typescript
const token = SLACK_BOT_TOKEN; // 環境変数から取得
// APIリクエスト例
fetch('https://slack.com/api/chat.postMessage', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    channel: 'your-channel-id',
    text: 'Hello from Cloudflare Workers!',
  }),
});
```

### 5. デプロイと動作確認

Cloudflare Wokers CLI（`wrangler`など）を使ってデプロイします。操作は前回同様です。

```bash
wrangler publish
```

必要に応じてAPI呼び出しやレスポンス処理を調整してください。

## まとめ

Cloudflare Workersは、UIが見やすく設定もシンプルなため、個人開発やPoCに最適です。今回は型安全性の向上や環境変数の便利な設定方法も紹介しました。今後はよりセキュアで効率的なAPI連携を目指し、実践的な開発を継続してください。