# BEAT — Project Bible
*Last updated: May 2026 | Paste this at the start of any Beat session*

---

## What Beat Is

Beat is a **social sentiment intelligence agent** for Solana memecoins. It scrapes Twitter, Telegram, and Discord, tracks KOL activity, runs NLP sentiment scoring, and surfaces pre-trending signals — tokens gaining social momentum before they hit Dexscreener's trending page.

**Beat is NOT the Polymarket wallet tracker.** That is a separate project entirely. Beat's domain is social/off-chain intelligence.

**Status: Concept/early design. No production code written. Architecture mostly settled, data source decisions partially open.**

---

## What Beat Does (Confirmed Scope)

**Data sources:**
- Twitter/X — KOL posts, ticker mentions, sentiment
- Telegram — group chatter, KOL channel monitoring
- Discord — server activity around specific tokens

**Processing:**
- NLP sentiment scoring (bullish/bearish/neutral per token)
- KOL tracking — who is posting what, when
- Mention velocity — rate of new mentions (not just count)
- Pre-trending detection — surface tokens gaining social heat before Dexscreener trending
- Coordinated shill detection — multiple KOLs posting the same token in a tight window (red flag vs organic signal)

**Output:**
- Token sentiment scores
- KOL activity alerts ("12 KOLs mentioned $WIF in last hour")
- Early trend signals (pre-Dexscreener)
- LLM briefing fed into verdict: BULLISH / BEARISH / NEUTRAL
- `arkham_dev` field — dev wallet entity identity from Arkham

---

## Arkham Integration (Designed, Not Yet Built)

Beat's Arkham integration is narrower than Spider's. Beat doesn't analyze on-chain wallet clusters — it analyzes social reputation. But one on-chain address Beat always has is the **dev wallet** from pump.fun metadata (`pf_meta.get("creator")`).

**The Arkham enrichment flow:**
```
analyze() → dev_wallet resolved → arkham_enrich([dev_wallet]) →
arkham_dev dict → injected into build_briefing() →
LLM reads: "DEV WALLET IDENTITY (Arkham): Jump Trading (trading_firm) — confidence 0.94"
→ Verdict factors in known institutional actor
```

**New Beat payload field:**
```python
"arkham_dev": {
    "entity_name": "Jump Trading",
    "entity_type": "trading_firm",
    "label": "Market Maker",
    "confidence": 0.94,
    "found": True,
}
```

**DB schema addition:** `arkham_dev_entity TEXT DEFAULT ''` column in `tokens` table.

**Important constraint:** Beat's callers are Twitter/Telegram handles, not on-chain addresses. Arkham can't help with social handles — only the dev wallet is on-chain. Don't try to extend Arkham enrichment to social handles.

---

## Data Source Decision — Open Problem

Twitter/social scraping is Beat's hardest infrastructure problem. The options, ranked:

| Option | Cost | Reliability | Notes |
|---|---|---|---|
| Twitter API (official) | $100/month minimum | High | Most reliable but expensive |
| Nitter (unofficial scraper) | Free | Medium | Breaks when Twitter changes internals |
| Playwright/Puppeteer (headless browser) | Free + compute | Medium | Better than Nitter, no vision model needed |
| LunarCrush MCP (already connected) | Included in plan | High | Covers social metrics, may reduce need for raw scraping |

**Key decision not yet made:** Does LunarCrush (already connected as an MCP) cover enough of Beat's social signal needs, or does Beat need its own raw scraping layer?

**Recommendation when building:** Evaluate LunarCrush MCP capabilities first before building custom scrapers. LunarCrush already tracks social volume, sentiment, and KOL activity for crypto tokens. If it covers 70%+ of Beat's requirements, it avoids the scraping infrastructure entirely.

**Telegram:** No official scraping API. Options are Telethon (unofficial Python library) or monitoring specific public channel webhooks. This is the hardest data source to make reliable.

**Discord:** Webhooks work for monitoring specific servers. Lower priority than Twitter/Telegram.

---

## Signal Architecture (Designed)

Beat's output feeds into a broader signal bus — not just for human consumption. The architecture intent is:

```
Beat (social signals)
    ↓
Signal Bus
    ↓
Spider (cross-reference: is this token also showing dump risk?)
Cented Classifier (is Cented buying something Beat flagged?)
Execution Layer (future)
```

Beat running alone is valuable. Beat running alongside Spider and the Cented classifier is where the real edge emerges — social momentum + on-chain confirmation + KOL wallet behavior converging on the same token.

---

## KOL Tracking Database

Beat needs a KOL database — a list of tracked accounts with their Twitter handles, Telegram channels, historical accuracy, and influence tier. This overlaps with:

- The 952-wallet Skope database (KOLs with known on-chain addresses)
- The Cented network map (Cented, Cupsey, and related wallets)

These should be unified into a single KOL master database accessible by Beat (social handles), Skope (on-chain addresses), and the Cented classifier (wallet behavior). Architecture decision for future session: how does this shared KOL DB get structured and maintained?

---

## Beat as Autonomous Agent (Future State)

Current concept is Beat as an on-demand tool. The stated long-term direction (April 2026 discussion) is Beat as a **24/7 autonomous agent**:

```
CURRENT: User triggers → Beat fetches → shows sentiment
FUTURE: Beat runs continuously → detects social spikes → fires alert to Telegram/Discord
        → feeds signal bus → no human trigger needed
```

The agent version doesn't need to be v1. Build the on-demand version first, validate signal quality, then automate.

---

## What Beat Is NOT

- Not the Polymarket wallet tracker (separate project entirely)
- Not Tide (order flow / liquidity monitoring — different domain)
- Not Spider (Spider analyzes on-chain dump risk — Beat is social/off-chain)
- Not Skope (Skope does forensic anatomy — dev identity, insider network, archetypes)

---

## Project Location

Lives in the `Tide-Skope-Beat` project folder alongside Tide. Beat and Tide are related (both feed the same signal bus) but separate concerns — social signal vs order flow signal.

---

## Things We Discussed But Deferred

**n8n deployment:** Discussed migrating the full agent suite to n8n (low-code automation platform) once fully built. Pros: visual workflow management, built-in API integrations, scheduling. Cons: performance ceiling for latency-sensitive operations, less control than raw Python. Decision deferred — build in Python first, consider n8n only for orchestration layer after the signal logic is validated.

**Playwright/Claw for scraping:** Discussed using browser automation agents (Claw-type systems) for Beat's Twitter scraping. Verdict: Playwright is better than Claw for this use case — no vision model overhead, faster, more reliable. But evaluate LunarCrush MCP first before building any scraper.

---

## Open Architecture Questions (Resolve at Build Phase Start)

1. Does LunarCrush MCP satisfy Beat's social data needs, or is custom scraping required?
2. What is the KOL database format and how is it shared with Skope?
3. Is Beat v1 on-demand (user triggers) or autonomous (runs on loop)?
4. What is the alert delivery mechanism — Telegram bot? Discord webhook? Dashboard?
5. How does Beat's sentiment score combine with Spider's dump risk score into a unified signal?

---

## Session Start Checklist

1. This bible is loaded
2. Confirm with Jeeps: architecture discussion or build phase?
3. If build phase: confirm LunarCrush MCP was evaluated first
4. Architecture decisions FIRST — data source choice must be settled before any scraping code is written
