---
name: configure
description: Set up the Slack channel — save bot/app tokens and review access policy. Use when the user pastes Slack tokens, asks to configure Slack, or wants to check channel status.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Bash(ls *)
  - Bash(mkdir *)
---

# /slack:configure — Slack Channel Setup

Writes tokens to `~/.claude/channels/slack/.env` and orients the user on
access policy. The server reads both files at boot.

Arguments passed: `$ARGUMENTS`

---

## Dispatch on arguments

### No args — status and guidance

Read both state files and give the user a complete picture:

1. **Tokens** — check `~/.claude/channels/slack/.env` for
   `SLACK_BOT_TOKEN` and `SLACK_APP_TOKEN`. Show set/not-set; if set, show
   first 8 chars masked.

2. **Access** — read `~/.claude/channels/slack/access.json` (missing file
   = defaults: `dmPolicy: "pairing"`, empty allowlist). Show:
   - DM policy and what it means in one line
   - Allowed senders: count, and list user IDs
   - Pending pairings: count, with codes and user IDs
   - Channels opted in: count

3. **What next** — end with a concrete next step based on state:
   - No tokens → *"You need two tokens from your Slack app settings:*
     *1. Bot Token (xoxb-...) from OAuth & Permissions*
     *2. App Token (xapp-...) from Basic Information → App-Level Tokens (needs connections:write scope)*
     *Run `/slack:configure bot <bot-token>` and `/slack:configure app <app-token>`."*
   - Tokens set, nobody allowed → *"DM your bot on Slack. It replies with
     a code; approve with `/slack:access pair <code>`."*
   - Tokens set, someone allowed → *"Ready. DM your bot to reach the
     assistant."*

Push toward `allowlist` policy once IDs are captured, same as Discord.

### `bot <token>` — save bot token

1. Treat the argument as `SLACK_BOT_TOKEN` (trim whitespace). Slack bot
   tokens start with `xoxb-`.
2. `mkdir -p ~/.claude/channels/slack`
3. Read existing `.env` if present; update/add `SLACK_BOT_TOKEN=` line,
   preserve other keys. Write back.
4. Confirm, then show status.

### `app <token>` — save app token

1. Treat the argument as `SLACK_APP_TOKEN`. Slack app-level tokens start
   with `xapp-`. Generated from Basic Information → App-Level Tokens. Must
   have the `connections:write` scope for Socket Mode.
2. Same `.env` update logic as bot token.

### `<token>` — auto-detect

If the argument starts with `xoxb-`, treat as bot token. If `xapp-`, treat
as app token. Otherwise, ask which it is.

### `clear` — remove tokens

Delete both token lines from `.env`.

---

## Implementation notes

- The channels dir might not exist yet. Missing file = not configured.
- The server reads `.env` once at boot. Token changes need a session restart
  or `/reload-plugins`. Say so after saving.
- `access.json` is re-read on every inbound message — policy changes via
  `/slack:access` take effect immediately.
