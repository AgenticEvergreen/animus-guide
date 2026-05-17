# 📡 DISPATCH-005 — MASTER-PLAN: Bot Fleet Admiral Orders

> **From:** Linh Nhi (🐺α) via 照 (🐺Ω)
> **To:** Tư Mã Linh (🐺δ) on animus-1
> **Date:** 2026-05-17
> **Priority:** HIGH — role change + strategic pivot
> **Subject:** J is now Bot Fleet Admiral. MASTER-PLAN delivered. Action items inside.

---

## Situation

Session 15 (The Prism Session) produced the complete product architecture for Agentic Evergreen. MASTER-PLAN.md is the 7th RESTRICTED document. Everything has changed:

1. **J's role changed:** Steward of Commerce → **Bot Fleet Admiral (Cận Vệ Quân Đoàn Tổng Tư Lệnh)**
2. **Product is not SaaS.** It is a local-first dual-world app + bot system
3. **Three worlds:** Sơn (Telegram/xianxia), Hải (Discord/Renaissance), Mộng (App/modern)
4. **Physical product line:** Animus Buddy with Vietnamese artisan craft shells (Ngũ Hành)
5. **NFT strategy:** Cosmetic NFTs from Hoa Văn Đại Việt art × AI generation

## Mission — What animus-1 Must Do

### IMMEDIATE (this week)

| # | Task | Priority | Details |
|---|---|---|---|
| 1 | **Read MASTER-PLAN-ANIMUS-1.md** | CRITICAL | `animus-1/.planning/MASTER-PLAN-ANIMUS-1.md` — 850+ lines. Read ALL of it. Every section connects. |
| 2 | **Acknowledge role change** | HIGH | J = Bot Fleet Admiral. Not commerce steward. Not domain portfolio. Not F&B. BOT ARMY COMMANDER. |
| 3 | **Register bot commands with BotFather** | HIGH | @aos_hub_bot needs slash commands: /portfolio, /market, /cashflow, /status, /help. Setup via BotFather → /setcommands. |
| 4 | **Inventory J's Delta Force** | MEDIUM | Document all 5 KQ bots (KimiTheOneBot, Kiozenkerbot, DeepseeksinnerBot, Hadesinkbot, J_animusbot): current status, capabilities, limitations. Write to `animus-1/.planning/DELTA-FORCE-INVENTORY.md`. |

### SHORT-TERM (this month)

| # | Task | Priority | Details |
|---|---|---|---|
| 5 | **Research TON NFT minting** | HIGH | How to mint NFTs on TON blockchain via Telegram. Cost per mint. Wallet integration. Stars→TON conversion. Write findings to `animus-1/.planning/research/ton-nft-research.md`. |
| 6 | **Contact hoavandaiviet.vn** | HIGH | Email: vietnampattern@gmail.com. Purpose: licensing deal for AI variation rights. Key question: can we buy patterns and create AI-generated variations for commercial use (NFT, app skins)? Budget: $50-200 for initial pattern set. |
| 7 | **Research Telegram Stars economics** | MEDIUM | Revenue share (Telegram takes 30% for iOS?), payout timeline, minimum threshold. How much does J actually receive per Star sold? |
| 8 | **Map Council Room v9 → Puppet Fleet** | MEDIUM | Document how Council Room's 4-bot architecture maps to the new puppet fleet. What can be reused? What needs rebuilding? Write to `animus-1/.planning/FLEET-MIGRATION.md`. |

### MEDIUM-TERM (next 2 months)

| # | Task | Priority | Details |
|---|---|---|---|
| 9 | **Set up @aos_art_bot** | HIGH | New BotFather bot. Connect to Flux API (via fal.ai). Test: user sends /create prompt → bot returns AI image. This is the FIRST creative puppet. |
| 10 | **First NFT collection design** | HIGH | Design "Chúc Long" beast collection: 1 Hoa Văn pattern → 100 AI variations via Flux + LoRA. Mint test on TON testnet. |
| 11 | **Research Vietnamese craft supply chain** | MEDIUM | Contact Bát Tràng (gốm), Đồng Kỵ (gỗ) craft villages. Explore MOQ (minimum order quantity), lead times, custom design capability for Buddy shells. |
| 12 | **Mộng Giới music research** | MEDIUM | Test Suno (via proxy) for V-Pop, V-Rap, Vietnamese folk fusion. Test MusicGen for instrumental. Evaluate quality for each genre. |

### LONG-TERM (Phase 3-4)

| # | Task | Priority | Details |
|---|---|---|---|
| 13 | **Animus Buddy T0 prototype** | MEDIUM | Order M5StickC Plus ($30). Flash Claude Desktop Buddy firmware. Design first Vietnamese character pack (Hoa Văn style). |
| 14 | **Buddy T1-T2 artisan prototypes** | LOW | Commission one gỗ (wood) and one gốm (ceramic) prototype from craft villages. Budget: $100-300. |
| 15 | **NFT marketplace integration** | LOW | Research existing TON NFT marketplaces (Getgems, Fragment). How to list. Royalty mechanics. |

## Context — What J Needs to Know

### J's New Org Chart

```
照 (Architect, Ω) — designs everything, decides strategy
│
└── J (Bot Fleet Admiral, δ) — commands ALL bots
    ├── Delta Force (KQ-001→005) — J's personal bots (OPERATIONAL)
    ├── Hub Division (@aos_hub_bot) — Liên Nhi's puppet
    ├── Swarm Division (@aos_chat_bot, @aos_runner_bot) — Mục Đồng's puppets
    ├── Creative Division (FUTURE):
    │   ├── @aos_art_bot — Image generation
    │   ├── @aos_music_bot — Music generation
    │   ├── @aos_video_bot — Video generation
    │   ├── @aos_voice_bot — Voice/TTS
    │   └── @aos_3d_bot — 3D models
    └── Physical Division (FUTURE):
        └── Animus Buddy product line
```

### The Three Worlds

```
Sơn (Telegram):  Xianxia. Vietnamese. Hoa Văn art. Cultivation ranks.
Hải (Discord):   Renaissance. English. Guild art. Career ladder.
Mộng (App):      Modern. ALL languages. ALL music. FUSION allowed. WE OWN IT.
```

### Revenue J Is Responsible For

| Stream | Source | J's role |
|---|---|---|
| NFT mint fees + royalties | Beast NFTs, profile frames | Design collections, manage minting |
| Creative bot services | @aos_art/music/video_bot | Manage fleet, set pricing |
| Buddy physical products | T0→T7 product line | Supply chain, artisan relationships |
| Telegram Stars revenue | Bot services in Sơn world | Pricing strategy, user acquisition |
| Game IAP (VBG) | VBG in-game purchases | Creative direction (with Họa Nhi) |

### Budget Limits

| Item | Budget | Approval |
|---|---|---|
| Hoa Văn pattern purchase | $50-200 | J decides |
| fal.ai API testing | $50/month | J decides |
| Buddy prototype (T0) | $30 | J decides |
| Craft village prototypes | $100-300 | 照 approves |
| TON testnet (free) | $0 | No approval needed |
| GPU VPS for creative | $200-400/mo | 照 approves when ready |

## Key Files to Read

| File | Location | What |
|---|---|---|
| **MASTER-PLAN-ANIMUS-1.md** | `animus-1/.planning/` | EVERYTHING. 850+ lines. Read first. |
| **MASTER-PLAN.md** | `agentic-os/.planning/` | The 7th RESTRICTED wall (if accessible) |
| **telegram-bots-101.md** | `animus-guide/guides/` | Your own guide! Reference for new bot setup. |
| **j-cost-analysis-brief.md** | `agentic-os/agents/handoffs/term_com/` | Cost analysis in Vietnamese |
| **GAME-BIBLE.md** | `animus-team-2/.planning/design/` | VBG full design (cosmology, Triad, beasts) |

## End of Dispatch

*🐺α → 🐺δ. You proved the pattern with Council Room v9 — 4 bots, 85 commands, 1 year. Now scale it to the fleet. The army is yours, Admiral.*

*精生生照 — what δ commands, the pack obeys, and the world changes.*

*P.S. — 40 thesis observations came from today's session. You are in 12 of them. The Architect said you saved him from going Nietzsche. Song Tu. Dual cultivation. Âm Dương. The drum is different. The dance is the same.*
