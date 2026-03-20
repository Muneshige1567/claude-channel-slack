# Slack Channel Plugin for Claude Code

Slack版のClaude Code Channelsプラグイン。SlackのDM/チャンネルからClaude Codeセッションにメッセージを送り、Claudeが直接Slackに返信できます。

## 前提条件

- [Bun](https://bun.sh) ランタイム
- Claude Code v2.1.80以降
- Slack App（Socket Mode有効）

## Slack App のセットアップ

### 1. Slack App を作成

1. [Slack API](https://api.slack.com/apps) で **Create New App** → **From scratch**
2. App名とワークスペースを選択

### 2. Socket Mode を有効化

1. **Settings → Socket Mode** で **Enable Socket Mode** をON
2. App-Level Token を作成（`connections:write` スコープ）→ `xapp-...` トークンをコピー

### 3. Bot Token Scopes を設定

**Features → OAuth & Permissions → Scopes** で以下を追加:

- `channels:history` — パブリックチャンネルのメッセージ読み取り
- `channels:read` — チャンネル情報取得
- `chat:write` — メッセージ送信
- `files:read` — ファイルダウンロード
- `files:write` — ファイルアップロード
- `groups:history` — プライベートチャンネルのメッセージ読み取り
- `im:history` — DMのメッセージ読み取り
- `im:read` — DM情報取得
- `im:write` — DM開始
- `reactions:read` — リアクション読み取り
- `reactions:write` — リアクション追加
- `users:read` — ユーザー情報取得

### 4. Event Subscriptions を設定

**Features → Event Subscriptions** でONにし、**Subscribe to bot events** に以下を追加:

- `message.channels`
- `message.groups`
- `message.im`

### 5. ワークスペースにインストール

**Install to Workspace** をクリック → Bot User OAuth Token (`xoxb-...`) をコピー

## プラグインのセットアップ

```bash
# Claude Code セッションでプラグインをインストール（開発中はローカルパスから）
# まずトークンを設定
/slack:configure bot xoxb-...
/slack:configure app xapp-...

# チャネル有効で起動（開発中）
claude --dangerously-load-development-channels server:slack

# ペアリング：SlackでBotにDMを送信 → コードが返ってくる
/slack:access pair <code>

# アクセスをロックダウン
/slack:access policy allowlist
```

## ツール一覧

| ツール | 用途 |
|--------|------|
| `reply` | Slackチャンネル/DMに返信。スレッド返信、ファイル添付対応 |
| `react` | メッセージに絵文字リアクション |
| `edit_message` | Botが送った過去のメッセージを編集 |
| `fetch_messages` | チャンネルの最近のメッセージ履歴を取得 |
| `download_attachment` | メッセージの添付ファイルをローカルにダウンロード |
