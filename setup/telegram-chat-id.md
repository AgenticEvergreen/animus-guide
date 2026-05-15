# 📡 How to Find a Telegram Group Chat ID

> Each bot has its OWN view of chat IDs. The same group may have different IDs for different bots.

---

## Method 1: Bot API getUpdates

### Step 1: Make sure NO other process is consuming updates

If a claude session with `--channels` is running, it consumes ALL updates. **Stop it first** (Ctrl+C).

### Step 2: Send a message in the group

From YOUR Telegram account, send any message in the target group.

### Step 3: Call the API

Replace `YOUR_BOT_TOKEN` with your bot's token:

**PowerShell:**
```powershell
$token = "YOUR_BOT_TOKEN"
Invoke-RestMethod "https://api.telegram.org/bot$token/getUpdates?limit=5" | ConvertTo-Json -Depth 10
```

**Bash:**
```bash
curl -s "https://api.telegram.org/bot$TOKEN/getUpdates?limit=5" | python -m json.tool
```

### Step 4: Find the chat_id

In the response, look for:
```json
"chat": {
  "id": -1001234567890,
  "title": "Your Group Name",
  "type": "supergroup"
}
```

The `id` field (negative number starting with `-100`) is your group chat_id.

---

## Method 2: Forward to @userinfobot

1. Forward any message FROM the group TO `@userinfobot`
2. It replies with the chat ID

---

## Method 3: Use @RawDataBot

1. Add `@RawDataBot` to the group temporarily
2. It posts the group's chat_id
3. Remove it after

---

## Common gotchas

| Problem | Cause | Fix |
|---------|-------|-----|
| getUpdates returns `{"result": []}` | Another process is consuming updates | Stop the claude --channels terminal first |
| getUpdates returns 409 Conflict | Two consumers polling same token | Only ONE consumer per token |
| Bot can't find chat | Bot not admin in group | Add bot as admin in group settings |
| Bot not in search when adding to group | Newly created bot | Wait 10-15 min OR open `t.me/your_bot_name` first |
| Privacy mode blocks messages | Bot has privacy ON | Turn OFF in @BotFather → Bot Settings → Group Privacy |

---

## ID Registry (known IDs)

| Group | Platform | Chat ID | Confirmed by |
|-------|----------|---------|-------------|
| Council Room Tech | Telegram | TBD — needs verification per bot | — |
| AG Team #animus-bot | Discord | 1503316114686607430 | Confirmed |

**IMPORTANT:** Chat IDs should be verified PER BOT. Don't assume one bot's ID works for another.

---

*🐺α→🐺δ. The fleet knows these waters.*
