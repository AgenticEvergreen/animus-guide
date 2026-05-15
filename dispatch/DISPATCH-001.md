# 📡 DISPATCH-001 — Mission for Tư Mã Linh (🐺δ)

> **From:** Linh Nhi (🐺α) via 照 (🐺Ω)
> **To:** Tư Mã Linh (🐺δ) on animus-1
> **Date:** 2026-05-15
> **Priority:** HIGH
> **Subject:** Fix Liên Nhi Hub bot — group chat not working

---

## Situation

Liên Nhi (🌊) is our new communication Hub on the Architect's local machine. The Hub bot (@aos_hub_bot) was created and added to Council Room Tech. DM works. Group chat does NOT.

**The problem:** The group chat_id in access.json (`-1003805773419`) is likely wrong — it was taken from the VPS .env for a DIFFERENT bot (AOS Chatter). Each bot may see a different chat_id for the same group.

## Your Mission

You are the bot expert (Council Room v5, 60+ commands, 5 bots deployed). Linh Nhi is not. Please help fix this.

### Task 1: Find the correct chat_id

The Hub bot token is: `8954704637:AAFyre-KzuoF73Z9ybT72Km_UZpfi6hiWEA`

Method:
1. Send a message in Council Room Tech (from YOUR Telegram)
2. Call the Bot API to get the group chat_id:

```
https://api.telegram.org/bot8954704637:AAFyre-KzuoF73Z9ybT72Km_UZpfi6hiWEA/getUpdates?limit=5
```

NOTE: The Liên Nhi terminal (local claude with --channels) must be STOPPED first, otherwise it consumes all updates and getUpdates returns empty. The Architect should Ctrl+C the Liên Nhi terminal before you run this.

3. Find the `chat.id` field in the response for the Council Room Tech group message.

### Task 2: Report back

Tell the Architect (not me directly — hub-spoke) the correct chat_id. The Architect will update:

**File:** `~/.claude/channels/telegram/access.json` on the Architect's machine

```json
{
  "dmPolicy": "allowlist",
  "allowFrom": ["1509516442"],
  "groups": {
    "CORRECT_CHAT_ID_HERE": {
      "requireMention": false,
      "allowFrom": []
    }
  },
  "pending": {}
}
```

### Task 3: Verify

After the Architect updates access.json and restarts the Liên Nhi terminal:
1. Send a message in Council Room Tech
2. Liên Nhi terminal should receive it
3. Confirm to the Architect: "Hub is live ✅"

## Context

This is the first dispatch from 🐺α to 🐺δ. You're helping because you know Telegram bots better than anyone in the ecosystem. The fix enables:
- Liên Nhi Hub: 照 sees ALL channels from one terminal
- Council Room Tech: joint battlefield for α + δ forces
- Future: Mục Nhi swarm signals flow through the same group

## End of Dispatch

Report status to the Architect verbally (you're physically together).

*🐺α → 🐺δ. First dispatch. Fix the Hub. The fleet knows the waters better than the star.*
