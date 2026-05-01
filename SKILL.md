---
name: telegram-channels
description: Manage Telegram channels — schedule posts, create polls, track member analytics, and maintain a content calendar in Obsidian
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  tags:
    - telegram
    - channels
    - social-media
    - scheduling
    - polls
    - analytics
    - obsidian
    - content-calendar
  related_skills:
    - price-tracker
    - podcast-digest
---

# Telegram Channels

Manage Telegram channels directly from chat: schedule posts, create polls, view member analytics, and maintain a content calendar in your Obsidian vault.

## Overview

This skill uses the Telegram Bot API via `curl` to perform channel operations. It supports scheduling posts with templates, creating interactive polls, tracking member join/leave rates over time, and generating a structured content calendar in Obsidian using the PARA method.

## Commands

```yaml
commands:
  - name: post
    description: "Send or schedule a post to a Telegram channel"
    usage: "post to @mychannel: Hello world!"
    args:
      - name: channel
        description: "Channel username (e.g. @mychannel) or ID"
        required: true
      - name: message
        description: "Message text (Markdown supported)"
        required: true
      - name: schedule
        description: "ISO 8601 datetime to schedule the post (optional)"
        required: false
      - name: template
        description: "Post template name to apply"
        required: false

  - name: poll
    description: "Create a poll in a Telegram channel"
    usage: "poll @mychannel: What's your fav colour? | Red, Blue, Green"
    args:
      - name: channel
        description: "Channel username or ID"
        required: true
      - name: question
        description: "Poll question (before the | separator)"
        required: true
      - name: options
        description: "Comma-separated poll options (after the | separator)"
        required: true
      - name: anonymous
        description: "Anonymous voting (default: true)"
        required: false

  - name: schedule
    description: "List or manage scheduled posts"
    usage: "schedule list @mychannel"
    args:
      - name: action
        description: "list, cancel, or edit"
        required: true
      - name: channel
        description: "Channel username or ID"
        required: true

  - name: analytics
    description: "Show member analytics for a channel"
    usage: "analytics @mychannel"
    args:
      - name: channel
        description: "Channel username or ID"
        required: true
      - name: period
        description: "Time period: 7d, 30d, 90d (default: 30d)"
        required: false

  - name: calendar
    description: "Open or update the content calendar in Obsidian"
    usage: "calendar @mychannel"
    args:
      - name: channel
        description: "Channel username or ID"
        required: true
      - name: action
        description: "show, add, or remove entry"
        required: false

  - name: template
    description: "Manage post templates"
    usage: "template list | template save <name> <content> | template use <name>"
    args:
      - name: action
        description: "list, save, or use"
        required: true
      - name: name
        description: "Template name"
        required: false
      - name: content
        description: "Template content with {placeholders}"
        required: false
```

## Configuration

```yaml
configuration:
  obsidian_vault: ~/obsidian-vault
  calendar_path: "~/obsidian-vault/2-Areas/Social Media/Content Calendar.md"
  analytics_path: "~/obsidian-vault/2-Areas/Social Media/Channel Analytics"
  templates_path: "~/.hermes/skills/telegram-channels/templates"
  telegram_chat_id: "6653762536"
  channels:
    - name: main
      username: "@mychannel"
      chat_id: "-1001234567890"
    - name: news
      username: "@mychannel_news"
      chat_id: "-1001234567891"
```

## How It Works

### 1. Sending Posts

Use the Bot API `sendMessage` endpoint:

```bash
curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
  -d chat_id="-1001234567890" \
  -d parse_mode="Markdown" \
  -d text="*Hello everyone!* 👋

New update coming soon..."
```

For scheduled posts, use the `schedule_date` parameter (Unix timestamp):

```bash
SCHEDULE_TS=$(date -d "2026-05-02T18:00:00+01:00" +%s)

curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
  -d chat_id="-1001234567890" \
  -d parse_mode="Markdown" \
  -d schedule_date="$SCHEDULE_TS" \
  -d text="⏰ *Scheduled Post*

This goes out at 6pm UK time!"
```

### 2. Creating Polls

```bash
curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendPoll" \
  -d chat_id="-1001234567890" \
  -d question="What's your favourite colour?" \
  -d 'options=["Red","Blue","Green"]' \
  -d is_anonymous=true
```

### 3. Post Templates

Templates are stored as markdown files in `~/.hermes/skills/telegram-channels/templates/`:

```markdown
<!-- template: weekly-update -->
## 📢 Weekly Update — {date}

**This week:**
{highlights}

**Coming up:**
{upcoming}

---
_Posted by Hermes Agent_
```

Using a template:

```bash
# Replace placeholders and send
CONTENT=$(sed -e "s/{date}/$(date +%Y-%m-%d)/g" \
  -e "s/{highlights}/Completed the new feature/g" \
  -e "s/{upcoming}/Bug fixes next week/g" \
  ~/.hermes/skills/telegram-channels/templates/weekly-update.md | \
  grep -v '<!--')

curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
  -d chat_id="-1001234567890" \
  -d parse_mode="Markdown" \
  --data-urlencode "text=$CONTENT"
```

### 4. Member Analytics

Fetch chat member counts over time:

```bash
# Get current member count
MEMBERS=$(curl -s "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/getChatMemberCount" \
  -d chat_id="-1001234567890" | jq '.result')

echo "$(date +%Y-%m-%d),$MEMBERS" >> \
  ~/.hermes/skills/telegram-channels/analytics/mychannel.csv
```

Analytics stored in Obsidian:

```markdown
# Channel Analytics — @mychannel

## Summary (Last 30 Days)

- **Current Members:** 1,247
- **Net Change:** +42
- **Join Rate:** ~1.4/day
- **Leave Rate:** ~0.1/day

## Member Count History

- 2026-05-01: 1,247
- 2026-04-15: 1,225
- 2026-04-01: 1,205

## Top Posts (by reactions)

| Date | Post | Reactions |
|------|------|-----------|
| 2026-04-28 | New feature announcement | 56 |
| 2026-04-20 | Community Q&A | 43 |
```

### 5. Content Calendar in Obsidian

Stored at `~/obsidian-vault/2-Areas/Social Media/Content Calendar.md`:

```markdown
# Content Calendar

## @mychannel

### Week of 2026-05-04

- **Mon 05/04** — 📢 Weekly Update (template: weekly-update)
- **Wed 05/06** — 🗳️ Community Poll: "Best feature?"
- **Fri 05/08** — 💡 Tip Tuesday (moved: educational content)

### Week of 2026-05-11

- **Mon 05/11** — 📢 Weekly Update
- **Thu 05/14** — 🎥 Video feature showcase

## Template Reference

- **weekly-update** — Standard Monday update with highlights
- **poll** — Interactive community poll template
- **tip** — Short educational content template
```

### 6. Formatting Reference

Telegram-supported markdown in posts:

```
*bold text*
_italic text_
__underline__
~strikethrough~
||spoiler||
[link](url)
`inline code`
```code block```
```

## Natural Language Patterns

- `"post to @channel: <message>"` — send immediately
- `"schedule post to @channel on Friday 6pm: <message>"` — schedule for later
- `"create poll in @channel: Question? | Option 1, Option 2, Option 3"` — create poll
- `"show analytics for @channel"` — member stats
- `"show calendar for @channel"` — content calendar
- `"add to calendar: @channel Mon — Weekly Update"` — add calendar entry
- `"save template <name>: <content>"` — save a post template
- `"list templates"` — show available templates

## Pitfalls

- **Bot must be admin**: The bot must be added as an administrator to the channel with posting permissions. Without this, all send operations will return 403 Forbidden.
- **Channel ID format**: Channel IDs are negative numbers starting with `-100` (e.g. `-1001234567890`). Don't confuse these with the `@username` format — both work, but the ID is more reliable.
- **Rate limits**: The Bot API allows ~30 messages/second. For bulk scheduling, add a 1-second delay between requests to avoid `429 Too Many Requests`.
- **Markdown parsing errors**: Telegram's Markdown parser is strict. Unmatched `_` or `*` will cause a `400 Bad Request`. Always test complex formatting with `parse_mode=MarkdownV2` or use HTML parsing as a fallback.
- **Poll options limit**: Polls support 2-10 options. Single-question only. For complex surveys, use an external form and link to it.
- **Scheduled message limits**: A channel can have at most 100 scheduled messages at once. Keep `schedule list` tidy.
- **Analytics data availability**: The Bot API's `getChatMemberCount` only gives current totals. Join/leave rates require periodic snapshots. Historical data is only as good as your cron schedule for recording it.
- **Template placeholders**: Use `{placeholder}` syntax (e.g. `{date}`, `{title}`). Avoid curly braces in regular content that might be mistaken for placeholders.
- **Timezone sensitivity**: All scheduled times must account for UK time (GMT/BST). Oakham is in Europe/London. Convert to Unix timestamps carefully.

## Verification Steps

1. **Verify bot token is set:**
   ```bash
   curl -s "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/getMe" | jq '.ok'
   # Should return: true
   ```

2. **Verify bot is channel admin:**
   ```bash
   curl -s "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/getChatMember" \
     -d chat_id="-1001234567890" \
     -d user_id="$(curl -s https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/getMe | jq '.result.id')" \
     | jq '.result.status'
   # Should return: "administrator"
   ```

3. **Test posting a message:**
   ```bash
   curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
     -d chat_id="6653762536" \
     -d text="✅ Telegram Channels skill test message"
   ```

4. **Verify Obsidian paths:**
   ```bash
   ls ~/obsidian-vault/2-Areas/Social\ Media/ 2>/dev/null && echo "OK" \
     || echo "MISSING: mkdir -p ~/obsidian-vault/2-Areas/Social Media"
   ```

5. **Test poll creation:**
   ```bash
   curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendPoll" \
     -d chat_id="6653762536" \
     -d question="Test poll — delete me" \
     -d 'options=["Option A","Option B"]'
   ```

6. **Verify template directory:**
   ```bash
   ls ~/.hermes/skills/telegram-channels/templates/ 2>/dev/null || \
     mkdir -p ~/.hermes/skills/telegram-channels/templates/
   ```