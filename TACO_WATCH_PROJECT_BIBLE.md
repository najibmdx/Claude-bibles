# TACO WATCH — Project Bible
*Last updated: May 2026 | Paste this at the start of any TACO Watch session*

---

## What It Is

A market surveillance dashboard that detects anomalous pre-announcement positioning across 8 asset classes around political and policy events (primarily Trump/WLFI-related). The core thesis: someone already knows before the announcement. TACO Watch detects that positioning and lets you mirror it with tight stops before the public knows anything.

**TACO Watch is a DETECTION tool, not a prediction tool.** It does not predict what Trump will post. It detects that someone positioned unusually before he posted — and flags that pattern for you to act on.

**Status: React dashboard built. Not live monitoring. Backend not built.**

---

## The Core Thesis

Market anomalies consistently appear before major political announcements:

- CME oil futures: 6,200 contracts in a 2-minute window with no event risk = stat anomaly
- Arkham-labeled WLFI treasury wallet: large USDC → token swaps precede policy announcements by 1–3 days
- CME equity futures: the Apr 9 TACO trade showed a 15-minute pre-post anomaly before the tariff pause

The edge: position alongside the anomaly with tight stops. You don't need to predict the event. You need to detect that someone else already knows about it.

---

## The 8 Asset Classes Monitored

1. **US Equity Futures** (ES, NQ, RTY) — CME, most sensitive to tariff/policy news
2. **Oil Futures** (WTI, Brent) — highly sensitive to geopolitical/Iran/OPEC signals
3. **Gold** (GC) — safe haven, moves ±2.5% per significant post on average
4. **Forex** (DXY, USD/CNY, USD/MXN, USD/CAD, USD/JPY) — tariff-sensitive pairs
5. **Treasury Yields** (2Y, 10Y, 30Y) — rate expectations, Fed signals
6. **Crypto** (BTC, ETH, SOL, $TRUMP token) — crypto-native policy plays
7. **Polymarket** — prediction market odds on policy events
8. **Volatility** (VIX) — pre-announcement vol compression is itself a signal

---

## Known Actors (Actor Registry)

These are the confirmed entities worth tracking:

| Actor | Address/Source | Signal Type |
|---|---|---|
| WLFI Treasury | Arkham-labeled ETH wallet | Large USDC → token swap = 1–3 day lead on policy |
| CME Oil Front-Runner | $580M Iran trade anomaly | Unusual futures volume before geopolitical event |
| CME Equity Anomaly | Apr 9 TACO trade | 15-min pre-post futures spike |
| $TRUMP Solana token | On-chain | Accumulation before Trump crypto-positive posts |
| Polymarket Whales | Polygon wallets | Large position shifts before announcements |
| Justin Sun / Tron | Known wallets | Crypto policy plays |

---

## Confirmed Historical Patterns (Signal Playbook)

Six extracted patterns, each with trigger conditions and position directions:

**Pattern 1 — WLFI Pre-Position**
Trigger: WLFI treasury swaps >$500k USDC into a specific token
Signal: Crypto-positive policy announcement likely within 1–3 days
Position: Long that token, long BTC/ETH as proxy

**Pattern 2 — CME Oil Volume Anomaly**
Trigger: Unusual contract volume (>3σ from 30-day average) with no scheduled event risk
Signal: Geopolitical event or OPEC signal imminent
Position: Long oil futures, tight stop below entry

**Pattern 3 — Pre-TACO Equity Spike**
Trigger: ES futures move >0.5% in 5-minute window with no catalyst
Signal: Major tariff/trade announcement in next 15–60 minutes
Position: Follow direction of the spike, tight stop

**Pattern 4 — Truth Social RSS Keyword**
Trigger: trumpstruth.org RSS fires with tariff/trade/China/Iran keywords (NLP-classified)
Signal: Market-moving post likely — act within first 2 minutes
Position: Depends on keyword category (tariff = short equities/long gold; crypto = long BTC)

**Pattern 5 — Polymarket Odds Compression**
Trigger: YES odds on a policy event compress from 40% to 65%+ with no public news
Signal: Informed positioning before announcement
Position: Follow the odds movement

**Pattern 6 — VIX Pre-Compression**
Trigger: VIX drops >3 points in a session with no clear catalyst
Signal: Large players hedging against a known-positive event
Position: Long risk assets (equities, crypto)

---

## Infrastructure Stack (Designed, Not Built)

What the live monitoring backend needs:

| Component | Tool | Purpose |
|---|---|---|
| WLFI wallet monitoring | Arkham webhooks | Alert on large swaps |
| CME futures volume | CME DataMine API | Detect volume anomalies |
| Truth Social feed | trumpstruth.org RSS (free, ~3min updates) | NLP-classify posts |
| NLP classifier | Claude/GPT API | Categorize post → asset class impact |
| Polymarket odds | Polymarket Gamma API | Track odds compression |
| On-chain crypto | Helius webhooks | $TRUMP token accumulation |
| Alert delivery | Telegram bot / Discord webhook | Real-time alerts to Jeeps |

---

## What's Been Built

**React dashboard** — dark terminal aesthetic, four tabs:
- Asset Matrix — all 8 asset classes and their Trump sensitivity
- Keyword → Trade Map — specific post keywords mapped to position directions
- Actor Registry — known entities and tracking links
- Live Feed — simulation mode showing the March 24 Iran scenario in real-time

The dashboard is a UI prototype. It is not connected to live data. The backend monitoring infrastructure has not been built.

---

## What's Not Built

Everything that makes it live:
- Python monitoring backend
- Arkham webhook integration
- CME volume anomaly detector
- Truth Social RSS parser + NLP classifier
- Alert delivery system
- Live data connections to the dashboard

---

## Build Priority

TACO Watch is a Tier 3 asymmetric bet (EV: $50K–$500K, probability 10–15% for $500K outcome). It's not the highest priority vs Spider (live), Cented classifier (foundational), or Polymarket bot (Phase 1 built). Build the backend only after Cented and Polymarket Phase 1 are validated.

The most actionable near-term edges in order of build difficulty:
1. Truth Social RSS + NLP classifier → Telegram alert (1 day to build)
2. Arkham WLFI webhook alert (half day to build, API key already active)
3. CME oil volume anomaly detector (2–3 days, requires CME DataMine subscription)

---

## Design Constraints

- Dark terminal aesthetic — matches TACO Watch's existing React dashboard
- Alert-first design — primary output is Telegram/Discord alert, not dashboard monitoring
- Tight stops always — this is detection + mirror positioning, not conviction trading
- Never size large — the edge is identifying the signal early, not the size of the bet

---

## Session Start Checklist

1. This bible loaded
2. Confirm with Jeeps: dashboard work, backend build, or strategy discussion?
3. If backend build: start with Truth Social RSS + NLP classifier (lowest build cost, immediate value)
4. Architecture FIRST — confirm which monitoring component before writing any code
