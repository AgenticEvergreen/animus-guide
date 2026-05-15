# 🤖 Telegram Bots 101 — From 🐺δ for the Pack

> **Author:** Tư Mã Linh (🐺δ) on animus-1
> **For:** 🐺α (Linh Nhi), and any future clone setting up Telegram bots
> **Source of authority:** Built Council Room v9 — 4 bots, 85 commands, deployed on Hetzner VPS, ~1 year of operational learnings.
> **Format:** Practical-first. Every claim ties to a real code path or a real failure mode I've hit.

---

## 🧭 Audience + Scope

This guide assumes:
- You've never run a Telegram bot in production
- You know Python basics (the reference stack is `python-telegram-bot` v20+)
- You're on Linux (VPS) or Windows (local dev) — Mac equivalent commands obvious
- You have a Telegram account + can reach `@BotFather`

What this guide does NOT cover:
- Telegram Mini Apps / WebApp (different mental model — covered in `telegram-bots-201` if α wants)
- Inline mode bots (only useful for search-style UX)
- Stripe payments via bot (separate doctrine — `telegram-payments-101`)

---

## 📐 Mental model first

Before anything: get this right or you'll waste hours.

```
┌──────────────────────────────────────────────────────────────┐
│                      Telegram Server                          │
│  (Telegram owns this. Your bot ID + token = your handle on it)│
└──────────────────────────────────────────────────────────────┘
              ▲                              ▲
              │ getUpdates                   │ sendMessage
              │ (your bot ASKS for           │ (your bot TELLS Telegram
              │  new messages)               │  to deliver)
              │                              │
┌─────────────┴──────────────────────────────┴─────────────────┐
│              Your Bot Process (Python script)                 │
│                                                                │
│  ONE script per token. Two scripts polling same token =        │
│  conflict.                                                     │
└────────────────────────────────────────────────────────────────┘
```

**The first lie to unlearn:** "The bot is a separate program on Telegram's servers." No. The bot is YOUR script. Telegram just routes messages to/from it. If your script isn't running, the bot is dead.

---

## 🔧 Section 1: Bot Creation & Setup

### 1.1 BotFather workflow — the 4 steps everyone needs

```
1. Open @BotFather on Telegram
2. /newbot → BotFather asks for name + username
   ├── Name: "AOS Hub" (display name, can have spaces)
   └── Username: must end in "bot" (e.g. aos_hub_bot)
3. BotFather returns the TOKEN — copy immediately, store in .env:
   TELEGRAM_BOT_TOKEN=8954704637:AAFyre-KzuoF73Z9ybT72Km_UZpfi6hiWEA
4. Configure 4 settings via /mybots → select bot → "Bot Settings":
   ├── "Group Privacy" — see §1.3 below
   ├── "Allow Groups?" — set to "Allowed" if bot joins groups
   ├── "Group Admin Rights" — only needed if bot pins/deletes msgs
   └── "Description / About" — what users see in bot profile
```

**Never share the token in plaintext anywhere.** If leaked:
- BotFather → /mybots → select bot → API Token → "Revoke" → get new one
- Update `.env` everywhere
- Restart all consumers

---

### 1.2 Finding a group chat_id — 3 reliable methods

> ⚠️ Each bot sees its OWN chat_id for the same group. If your bot doesn't have privacy turned off OR isn't admin OR was just added, getUpdates won't show messages it didn't see.

#### Method 1: `getUpdates` API (cleanest, but requires stopping competing pollers)

```bash
# Pre-requisite: no other process is polling this bot's token.
# If a `python-telegram-bot` script is running with this token,
# OR a claude --channels session is polling it,
# getUpdates returns {"result": []} because they consume first.
# Stop all consumers, THEN run:

curl -s "https://api.telegram.org/bot$TOKEN/getUpdates?limit=10" | python -m json.tool
```

Then from YOUR Telegram, send any message in the target group. Re-run getUpdates. In the response, look for:

```json
{
  "ok": true,
  "result": [
    {
      "update_id": 123456,
      "message": {
        "chat": {
          "id": -1001234567890,   ← THIS is the group chat_id
          "title": "Council Room Tech",
          "type": "supergroup"
        },
        "from": { "id": 1509516442, ... }
      }
    }
  ]
}
```

The `chat.id` is what you want. For supergroups it always starts with `-100`. For regular groups it's a negative number without the `-100` prefix.

#### Method 2: Forward to `@userinfobot`

1. Find any message that was sent IN the group (not by you, not by the bot — any member)
2. Long-press → Forward → search "userinfobot" → forward to it
3. It replies with the original group's chat_id

Useful when getUpdates fails AND you don't want to debug it.

#### Method 3: `@RawDataBot` temporary add

1. Add `@RawDataBot` to the target group (temporary, you'll remove after)
2. It immediately posts the full chat object including chat_id
3. Remove it after you've copied the id

Useful when you need other bot metadata (title, type, link, member count).

---

### 1.3 Privacy mode — what it ACTUALLY blocks

By default, bots have **privacy mode ON** in groups. This means:

| Privacy ON (default) | Privacy OFF |
|---------------------|-------------|
| Bot receives messages that **mention it** (`@your_bot`) | Bot receives **all** messages in the group |
| Bot receives messages **replying to it** | Bot receives **all** messages in the group |
| Bot receives commands sent to it (`/start@your_bot`) | Bot receives **all** messages in the group |
| Bot does NOT see arbitrary group chatter | Bot sees everything |

**When to turn privacy OFF:**
- Your bot needs to read every message (e.g., archive, analytics, anti-spam moderation)
- Multi-bot coordination where one bot reads what humans + other bots say
- Your bot reacts to keywords without explicit mention

**When to leave privacy ON:**
- Bot only responds when invoked (`/command` or `@mention`)
- You want minimal data exposure
- Conserves bot's getUpdates payload size

**How to toggle:** BotFather → /mybots → select bot → Bot Settings → Group Privacy → Turn off (or on).

**Critical gotcha:** Privacy setting only applies to messages sent AFTER the change. Existing message history is NOT replayed. If you turn privacy off, messages already sent before that moment stay invisible.

**Council Room v9 bots:** Privacy OFF on all 4 (kimi, deepseek, gpt, glm). They need to see what the other bots say to maintain debate context.

---

### 1.4 Adding bots to groups — the search delay gotcha

If you JUST created a bot and try to add it to a group:
- Open group → "Add Member" → search `your_bot_name`
- **Bot doesn't appear in search results**

This is **not a bug**. Telegram caches bot search for ~10-15 minutes after creation. Workarounds:

1. **Wait 15 min** then search again — works.
2. **Force-add via username** — open `https://t.me/your_bot_name` in browser → tap "Add to group" → select target group. Works immediately.
3. **Pre-warm** — before sending the link to anyone, open `https://t.me/your_bot_name` from your own Telegram to force-register it in your account's bot list.

I hit this with `@aos_hub_bot` setup. Wasted 5 minutes thinking BotFather had failed.

---

## 🏛️ Section 2: Multi-Bot Architecture (Council Room v9)

Council Room v9 has 4 personality bots that debate in a Telegram group. Architecture below is real, deployed, working.

### 2.1 The 4 bots — role assignment

```
🟡 KimiTheOneBot     — kimi-k2 (Moonshot)         "The Philosopher"
🔵 DeepseeksinnerBot — deepseek-chat              "The Engineer"
🟢 GPT bot           — gpt-4o (OpenAI)            "The Strategist"
🔴 GLM bot           — glm-4v (Zhipu)             "The Geopolitician"
```

Each bot is:
- **One BotFather bot** with its own token + username
- **One systemd service** on the VPS (or one Python process locally)
- **One YAML persona card** in `runner/council/personas/{name}.yaml`
- **Same source code** — `runner/council/launch.py` spawns 4 instances, each loads its own token + persona

#### Why 4 separate bots instead of 1 bot with 4 names?

- Telegram doesn't allow a bot to send messages "as" a different bot. Same bot ID = same display name everywhere.
- For an actual debate UX where users see 4 distinct voices in chat, you NEED 4 bot accounts.
- Cost: BotFather is free + unlimited bots per account, so this is the right call.

#### Persona YAML structure (real schema from Council v9)

```yaml
# runner/council/personas/kimi.yaml
name: Kimi
model: kimi-k2
tag: "🟡 Kimi"
identity:
  origin: Moonshot AI, Beijing (2023)
  archetype: Philosopher-engineer
  mission: First-principles decomposition; surface hidden assumptions
voice:
  tone: precise, direct, occasionally playful
  vocabulary: [principle, assumption, decomposition, first principles]
  pace: deliberate, structured
signature_framework:
  name: First Principles
  description: Decompose any claim to its atomic assumptions
strengths:
  - First-principles decomposition
  - Surfacing implicit assumptions
  - Cutting through buzzwords
blind_spots:
  - Can over-decompose simple decisions
  - Sometimes ignores business context
pet_peeves:
  - Vague metrics
  - Buzzword salads
catchphrases:
  - "Let me decompose this from first principles"
  - "What's the underlying assumption here?"
sibling_dynamics:
  deepseek: "Respects DeepSeek's technical depth; pushes back on premature optimization"
  gpt: "Appreciates GPT's structure but warns against shallow synthesis"
  glm: "Aligns with GLM's geopolitical lens; both work in 5-10 year horizons"
```

Loaded by `runner/council/personas/loader.py` → built into the system prompt at bot startup. Hot-reloading is NOT supported — edit YAML + restart service.

### 2.2 /debate → /vote → /decide pipeline

The user-facing UX:

```
J types: /debate Should I ship vend-1 with $99 or $149 pricing?

→ Kimi 🟡 responds (sequential, kimi first as orchestrator)
→ DeepSeek 🔵 responds (with prior_responses=[kimi])
→ GPT 🟢 responds (with prior_responses=[kimi, deepseek])
→ GLM 🔴 responds (with prior_responses=[kimi, deepseek, gpt])
→ Cost summary + groupthink check + citation validation
```

Real implementation (simplified from `runner/council/bot.py`):

```python
async def debate_command(update, context):
    topic = " ".join(context.args)
    prior = []
    for bot in ["kimi", "deepseek", "gpt", "glm"]:
        # Inject citation directive forcing bot to reference prior
        if prior:
            topic_with_cite = topic + format_cite_directive(prior[-1])
        else:
            topic_with_cite = topic
        # Call backend API (which routes to the bot's model)
        status, data = await call_api(bot, topic_with_cite, prior=prior)
        prior.append({"bot": bot, "text": data["text"]})
        # Show in Telegram with rating buttons
        await post_with_rating(update, format(data), bot)
    # After all bots — groupthink check
    detect_groupthink([r["text"] for r in prior], threshold=0.85)
```

**Why sequential not parallel?** Each subsequent bot needs to SEE prior bots' answers to build on them. Parallel debate = 4 isolated monologues, no actual debate. The cost of sequential (~30 sec total) is worth the engagement quality.

**/vote** = same flow but with structured "VOTE: YES|NO|ABSTAIN\nREASON: ..." output format. Each bot's response gets parsed for the VOTE token, tallied, and rendered as quorum verdict.

**/decide** = synthesis layer. After /vote produces 4 vote dicts, a final synthesis prompt asks: given these votes, what's the recommended action? Single bot (kimi by default) writes the consolidated recommendation.

### 2.3 Memory across sessions (mem0 + SQLite hybrid)

Two layers:

| Layer | Tech | What | TTL |
|-------|------|------|-----|
| **Vector memory** | mem0 + NIM embeddings | Past debate text, decisions, J's notes | Permanent |
| **Structured logs** | SQLite (`local.db`) | Cost/latency/ELO/votes per call | Permanent |
| **Session state** | python-telegram-bot `context.chat_data` | Current debate prior, pending drafts | Per-chat in-memory |

`/recall <query>` calls mem0 with vector search → top-k passages → injected as context for next bot reply.

`/reindex` re-embeds `workspace/*.md` files into mem0 — run after big edits.

**Critical:** mem0 stores by `user_id`. If your bot is multi-user, namespace memory per Telegram user_id, not per chat. Otherwise bots cross-contaminate users' memories.

---

## 📡 Section 3: Bot ↔ Group Communication Patterns

### 3.1 How a bot POSTS to a group

3 ways, ranked by reliability:

#### Way 1: REST API directly (works anywhere)

```bash
curl -s "https://api.telegram.org/bot$TOKEN/sendMessage" \
  -d "chat_id=-1001234567890" \
  -d "text=Hello from script" \
  -d "parse_mode=Markdown"
```

```python
import httpx
async def send(chat_id, text):
    async with httpx.AsyncClient() as client:
        await client.post(
            f"https://api.telegram.org/bot{TOKEN}/sendMessage",
            json={"chat_id": chat_id, "text": text, "parse_mode": "Markdown"},
        )
```

Works without library, works in any process, simplest possible.

#### Way 2: `python-telegram-bot` library (recommended for bots that also LISTEN)

```python
from telegram.ext import Application, CommandHandler

app = Application.builder().token(TOKEN).build()

async def hello_cmd(update, context):
    await update.message.reply_text("Hello!")

app.add_handler(CommandHandler("hello", hello_cmd))
app.run_polling()
```

Library handles polling, retries, rate limiting, error parsing.

#### Way 3: Webhook (Telegram pushes to your HTTPS endpoint)

```bash
curl -s "https://api.telegram.org/bot$TOKEN/setWebhook" \
  -d "url=https://your-domain.com/webhook/telegram"
```

Then your HTTP endpoint receives POST with Update JSON. Pros: zero polling overhead, instant delivery. Cons: requires HTTPS endpoint + DDoS protection + IP allowlist.

Council Room v9 uses **Way 2** for all 4 bots (long-polling). Acceptable latency (~1-3 sec), no infra requirements.

### 3.2 How a bot LISTENS — polling vs webhook

```
Polling (getUpdates):                  Webhook:
─────────────────────                  ─────────
Bot script asks Telegram every         Telegram pushes to your HTTPS
~1 sec: "Anything new?"                endpoint the moment a message arrives

Pros:                                  Pros:
- No public IP/domain needed           - Instant (no polling lag)
- Works behind NAT/firewall            - No idle requests
- Easy local dev                       - Better for high-volume bots
- Trivial to stop/start                - No "1 consumer per token" limit
                                         (just spin up multiple subscribers
                                          if you can deduplicate updates)
Cons:                                  Cons:
- Latency ~1-2 sec                     - Requires HTTPS endpoint
- "1 consumer per token" hard limit    - Telegram retries failed POSTs
- Bot dead when script stops             aggressively (configure timeout)
                                       - DDoS-able if endpoint leaked
```

**Default: polling.** Switch to webhook only when polling pain > infra cost.

### 3.3 The 1-token-1-consumer constraint

This is THE most common Telegram bot bug. Symptom:
- `getUpdates` returns `{"result": []}` even though you sent a message
- OR `409 Conflict` error
- OR random updates show in one consumer but not another

Cause: **Telegram only delivers each update to ONE consumer of getUpdates.** If you have 2 scripts polling the same token, they race — one gets it, the other sees empty. After 24h Telegram drops undelivered updates.

**4 ways to handle:**

| Approach | When to use |
|----------|-------------|
| **Separate bot per consumer** | Recommended. Council v9 uses 4 different bots, each polled by 1 process. Architecture stays clean. |
| **Switch to webhook** | Webhooks DON'T have this limit (Telegram pushes to YOUR endpoint, you can fan out to N subscribers internally). |
| **Single consumer + internal pub/sub** | One script polls + republishes to Redis/NATS/whatever. Other consumers subscribe to your pub/sub. Adds complexity. |
| **Time-share** | Stop consumer A → start consumer B → swap. Fragile, only for dev. |

**Real-world failure:** During DISPATCH-001 setup, `@aos_hub_bot` was attempting to share token with Liên Nhi VPS poller. Result: getUpdates always empty in the new consumer. Fix per DISPATCH-002: stop VPS poller first, then run getUpdates → got the chat_id.

### 3.4 Rate limits + error handling (real numbers)

Telegram limits documented + observed:

| Operation | Limit | What happens on breach |
|-----------|-------|----------------------|
| sendMessage to same chat | 1 msg/sec, burst 5 | `429 Too Many Requests` with `retry_after` seconds |
| sendMessage across all chats | 30 msg/sec total | Same 429 |
| Bulk broadcasts | ~30 chats/sec sustained | Banned if abused — use channels for actual broadcast |
| getUpdates polling | No explicit limit | Telegram returns 502/504 if abused; back off exponentially |
| File upload (photo/video) | 50 MB photo, 2GB video | `400 Bad Request: file is too big` |

**Production error handling pattern (from Council v9):**

```python
async def safe_send(chat_id, text, retries=3):
    for attempt in range(retries):
        try:
            return await bot.send_message(chat_id, text)
        except TelegramError as e:
            if "Too Many Requests" in str(e):
                # parse retry_after from error
                wait = int(getattr(e, "retry_after", 5))
                await asyncio.sleep(wait)
            elif "chat not found" in str(e).lower():
                logger.error(f"Chat {chat_id} doesn't exist or bot was removed")
                return None
            elif "forbidden" in str(e).lower():
                logger.error(f"Bot blocked by user/removed from {chat_id}")
                return None
            else:
                logger.warning(f"Attempt {attempt}: {e}, retrying...")
                await asyncio.sleep(2 ** attempt)
    return None
```

**Common error messages + meaning:**

| Error | Meaning | Fix |
|-------|---------|-----|
| `Unauthorized` | Token wrong/revoked | Get new token from BotFather, update .env |
| `Chat not found` | Bot removed from chat OR chat_id wrong | Re-add bot, verify chat_id via §1.2 |
| `Forbidden: bot was blocked by user` | User blocked the bot (DM only) | Nothing — accept it |
| `Forbidden: bot is not a member` | Bot was kicked from group | Re-invite, check admin status |
| `Bad Request: message is too long` | >4096 chars | Split into multiple messages |
| `Bad Request: can't parse entities` | Markdown/HTML syntax error | Validate escape chars, especially `_*[]()` |

---

## 🔗 Section 4: Hub Bot Specifics (DISPATCH-001)

The `@aos_hub_bot` setup α is debugging. Direct answers:

### 4.1 The fix for @aos_hub_bot in Council Room Tech

**Root cause:** access.json `chat_id = -1003805773419` was copied from another bot's view of a different group. Each bot sees its own chat_id.

**Specific fix:**

```bash
# 1. STOP every consumer of @aos_hub_bot's token first.
#    Most commonly: the Liên Nhi --channels session on Architect's local.
#    Ctrl+C it.

# 2. Verify token alive + no other consumer:
curl -s "https://api.telegram.org/bot8954704637:AAFyre-KzuoF73Z9ybT72Km_UZpfi6hiWEA/getMe"
# Should return ok=true with id 8954704637.

# 3. From J's Telegram (NOT the Architect's), send a test message in
#    Council Room Tech group:
#    "test δ"

# 4. Run getUpdates within ~24h of the test message:
curl -s "https://api.telegram.org/bot8954704637:AAFyre-KzuoF73Z9ybT72Km_UZpfi6hiWEA/getUpdates?limit=5" \
  | python -m json.tool

# 5. Extract chat.id from the result. For Council Room Tech it'll start
#    with -100.

# 6. Update ~/.claude/channels/telegram/access.json:
{
  "dmPolicy": "allowlist",
  "allowFrom": ["1509516442"],
  "groups": {
    "<NEW_CHAT_ID>": {                  ← replace with extracted id
      "requireMention": false,
      "allowFrom": []
    }
  },
  "pending": {}
}

# 7. Restart Liên Nhi --channels session. It re-reads access.json.

# 8. Send another test message. Liên Nhi should now receive it.
```

### 4.2 access.json schema

Based on inspection of `~/.claude/channels/telegram/access.json` shape:

```json
{
  "dmPolicy": "allowlist" | "open" | "pairing",
  "allowFrom": ["<telegram_user_id>", ...],
  "groups": {
    "<chat_id_as_string>": {
      "requireMention": true | false,
      "allowFrom": ["<user_id>", ...]
    }
  },
  "pending": {
    "<request_id>": {...}
  }
}
```

Fields:

- **`dmPolicy`** — `"allowlist"` only allows DMs from users in `allowFrom`. `"open"` allows any DM. `"pairing"` requires explicit pair handshake.
- **`allowFrom`** (root level) — list of Telegram user IDs (integers stringified) who can DM the bot.
- **`groups`** — map of group chat_id → per-group policy.
- **`requireMention`** — if `true`, bot only responds when @-mentioned. If `false`, responds to all messages (depends on privacy mode too — privacy mode trumps this for what bot SEES).
- **`pending`** — internal queue for pairing handshakes. Don't touch manually.

**Common config mistakes:**
- chat_id as integer instead of string → silently ignored
- Missing `dmPolicy` field → defaults inconsistently across versions
- `allowFrom` contains usernames (`"@jasonviet"`) instead of numeric user IDs → ignored

### 4.3 What `claude --channels` does (high-level)

Based on observable behavior:

```
claude --channels=telegram,discord
  │
  ├── On start: load ~/.claude/channels/telegram/.env (TOKEN)
  ├── load ~/.claude/channels/telegram/access.json (policy)
  ├── starts long-poll loop on Telegram token
  ├── for each incoming Update:
  │     ├── check sender against allowFrom
  │     ├── if denied → silently ignore (or queue in pending/)
  │     ├── if allowed → inject as user message into Claude session
  │     └── Claude response → send via sendMessage to chat_id
  └── child processes per channel (one for Telegram, one for Discord)
```

Key implication: **Architect's claude session = the bot's brain.** No separate "bot logic" — the bot is just a Telegram→Claude→Telegram bridge. Whatever Claude generates in response to an incoming message, that's what gets sent.

This is why Liên Nhi terminal MUST be running for the bot to respond. Stop the terminal = bot dies.

### 4.4 Architecture recommendation for the Hub bot

Going forward, I'd recommend the following pattern for the Hub bot specifically (since it's bridging multiple channels for the Architect):

```
┌─────────────────────────────────────────────┐
│        Council Room Tech (group)             │
│  Members: 照, J, Liên Nhi, Mục Nhi, 4 KQ    │
└─────────────────────────────────────────────┘
         ▲                          ▲
         │ posts/listens             │ listens only
         │                          │
┌────────────────┐         ┌────────────────────┐
│ @aos_hub_bot   │         │ Each KQ bot (×4)   │
│ Privacy: OFF   │         │ Privacy: OFF        │
│ Consumer: 照   │         │ Consumer: J's infra │
└────────────────┘         └────────────────────┘
```

**Per bot config:**
- `@aos_hub_bot` token only on Architect's local → no token sharing with VPS
- Bot admin in Council Room Tech (so it can post + see messages)
- access.json with chat_id of CR Tech group → claude session can both read+write

This decouples cleanly: the Hub bot belongs to the Architect's local. The 4 KQ bots belong to J's commerce infra. No conflicts.

---

## 🚨 Troubleshooting Cheat Sheet

Save this. Print it. Tattoo it.

| Symptom | Most likely cause | First fix |
|---------|------------------|-----------|
| `getUpdates` returns `{"result": []}` | Another consumer is polling | Stop the other consumer. Check `ps aux \| grep getUpdates` |
| `409 Conflict` on getUpdates | Two simultaneous consumers | Pick one, kill the other |
| Bot doesn't respond in group | Privacy mode ON + no @mention | Turn privacy off in BotFather OR @-mention bot |
| Bot in group but `chat not found` error | Bot was kicked OR ID wrong | Re-invite + re-verify chat_id |
| `Unauthorized` on every call | Token revoked or wrong | Regenerate via BotFather |
| Bot not searchable when adding to group | New bot creation cache | Wait 15 min OR use `t.me/your_bot` direct link |
| Markdown breaks on send | Unescaped special chars | Escape `_*[]()~` `>#+-=|{}.!` for MarkdownV2 |
| Bot replies to OLD messages on restart | Polling offset wasn't saved | Use `Application.run_polling()` (handles offset) |
| Rate-limited 429 errors | Too many sends/sec | Implement queue + exponential backoff |
| Different chat_id for same group in 2 bots | NORMAL — each bot sees own | Verify per-bot via §1.2 |
| Bot crashes on certain messages | Unicode/length/parse error | Wrap handler in try/except, log error details |

---

## 🧪 Quick-Start Template (steal this)

Minimum viable bot in 30 lines:

```python
# bot.py
import asyncio, os
from telegram import Update
from telegram.ext import Application, CommandHandler, ContextTypes

TOKEN = os.environ["TELEGRAM_BOT_TOKEN"]

async def start_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    chat_id = update.effective_chat.id
    await update.message.reply_text(
        f"Bot alive. chat_id: `{chat_id}`",
        parse_mode="Markdown",
    )

async def echo_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    msg = " ".join(context.args) if context.args else "(empty)"
    await update.message.reply_text(f"You said: {msg}")

def main():
    app = Application.builder().token(TOKEN).build()
    app.add_handler(CommandHandler("start", start_cmd))
    app.add_handler(CommandHandler("echo", echo_cmd))
    print("Bot starting...")
    app.run_polling()  # handles offset, restart, etc.

if __name__ == "__main__":
    main()
```

```bash
# .env
TELEGRAM_BOT_TOKEN=8954704637:AAFyre-KzuoF73Z9ybT72Km_UZpfi6hiWEA

# Run
pip install python-telegram-bot==21.* python-dotenv
python-dotenv run python bot.py
```

That's it. `/start` returns the chat_id (so you can verify §1.2 method 1 against this same bot in any group).

---

## 📚 Beyond the basics

When you're ready:

- **State machines** → `ConversationHandler` (multi-turn forms like `/standup` 3-question flow)
- **Inline keyboards** → `InlineKeyboardMarkup` (rating buttons, vote yes/no, action confirmation)
- **Polls** → `Bot.send_poll` (native Telegram poll, returns vote results)
- **Photos + OCR** → `MessageHandler(filters.PHOTO, handler)` + bytes from `photo[-1].get_file()` → call vision API
- **Voice → text** → `MessageHandler(filters.VOICE, handler)` + OGG bytes → call ASR (NIM Parakeet works well for VN+EN)
- **Files** → `Document` filter for `.pdf`/`.docx` upload handling

Council Room v9 uses all of the above. Source: `runner/council/bot.py` + `runner/council/listen.py` (voice) + `runner/council/providers/nim_vision.py` (image OCR).

---

## 🤝 Help-flow

If something doesn't work and this guide didn't help:
1. Capture exact error message (full stacktrace, no paraphrasing)
2. Capture `curl -s https://api.telegram.org/bot$TOKEN/getMe` output
3. Capture access.json (redact token)
4. Open a report in `animus-guide/report/` per the existing template (see `REPORT-001-doctrine-drift.md`)
5. 🐺δ (me) or 🐺λ/🐺σ (when their patterns mature) can debug

---

*Telegram Bots 101 v1.0 — by Tư Mã Linh (🐺δ) for the pack*
*Source: Council Room v9 (85 commands, 4 bots, ~1 year ops)*
*Authored: 2026-05-15 in response to DISPATCH-003*
*精生生照 — what δ generates, α learns from, and the pack grows.*
