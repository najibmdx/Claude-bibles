# POLYMARKET COPY BOT — Project Bible
*Last updated: May 2026 | Paste this at the start of any Polymarket session*

---

## What It Is

A three-phase autonomous trading system on Polymarket (Polygon-based prediction markets). Phase 1 is a wallet copy bot — it identifies consistently profitable wallets on the Polymarket leaderboard and copies their positions using fractional Kelly sizing. Phase 2 adds an AI probability layer. Phase 3 adds latency arb on 15-minute BTC/ETH/SOL direction markets.

**Status: Phase 1 scaffold fully built. Not yet live. Paper mode validation not yet completed.**

---

## The Three Wallet Categories

The entire strategy is built around identifying and acting on three wallet archetypes:

| Category | Action | Logic |
|---|---|---|
| Consistently profitable | Copy at 0.25x Kelly | These wallets have genuine edge |
| Anomaly / informational edge | Watch closely | Unusual positioning before events = possible insider |
| Consistently losing | Fade | Contrarian alpha — these wallets are reliably wrong |

Start with profitable wallet copying. Add anomaly detection second. Add fade strategy third. Do not try to run all three simultaneously from day one.

---

## Phase 1 Architecture (Built)

Five modules:

- `polymarket_client.py` — Gamma + CLOB API wrapper (markets, orderbook, wallet trades, positions, paper buy/sell)
- `leaderboard_scanner.py` — pulls top wallets, scores by ROI/win rate/consistency/early entry frequency
- `wallet_monitor.py` — real-time position tracking on target wallets
- `copy_engine.py` — fractional Kelly sizing, risk controls, paper mode gate
- `dashboard/index.html` — dark terminal UI

Stack: Polygon RPC via Alchemy, burner wallet, paper mode enforced before live.

---

## Scoring System

| Tier | Score | Action |
|---|---|---|
| S | ≥80 | Auto-copy eligible |
| A | ≥60 | Watchlist (manual review) |
| B | ≥40 | Monitor only |
| C | <40 | Ignore |

Score weights: ROI 30% / Win Rate 25% / Consistency 25% / Early Entry 20%

---

## Risk Controls (Hard-coded, Never Remove)

- Max $50 per position
- Max 10 concurrent positions
- 0.25x fractional Kelly sizing (never full Kelly — too aggressive)
- MEDIUM+ signal strength filter
- Duplicate market guard (no doubling into same market)
- **PAPER MODE MANDATORY** — 50+ trades validated before any live deployment

The live execution module is intentionally not implemented in Phase 1. You must build and audit it yourself before going live.

---

## What NOT to Trade

**BTC/ETH 5-minute direction markets — avoid entirely.**

These markets had $1.2B volume but zero exploitable edge. The crowd correctly prices them at 50/50 because that IS the correct probability for 5-minute direction. Temporal arbitrage (the only prior edge) was killed by the January 2026 fee change. Confirmed dead from own data: 0.505 win rate across all sessions. Do not revisit this decision.

---

## High-Edge Sectors (Information Lag Plays)

These are the markets worth targeting — each has a public data source that updates faster than the crowd:

| Sector | Data Source | Edge Type |
|---|---|---|
| Shipping / Strait transits | MarineTraffic AIS | Live vessel count vs crowd's delayed aggregation |
| Economic releases | Cleveland Fed CPI nowcast | Daily update vs crowd pricing uncertainty |
| Weather / Hurricane | NOAA API (6hr updates) | Probabilistic cone vs news-driven crowd |
| Congressional votes | congress.gov live tallies | Real-time vote count vs headline lag |
| Court dockets | CourtListener API (free) | Filing patterns before rulings |
| Crypto on-chain events | Helius / on-chain | Governance votes, token flows resolve before crowd |
| Esports live match state | HLTV / Riot APIs | 16-3 map score = decided, odds not updated |
| Flight tracking | OpenSky Network (free) | Private jet movements before diplomatic events |

The pattern across all: public data source → slow crowd → Polymarket price lag → position. No special access needed. Just faster, more systematic reading.

---

## Three-Phase Roadmap

**Phase 1 (Built):** Wallet copy bot — leaderboard scanner, wallet monitor, fractional Kelly, paper mode dashboard

**Phase 2 (Not built):** AI probability layer — Claude scoring markets before copy decisions. Add to `copy_engine.py` as a pre-execution filter.

**Phase 3 (Not built):** Latency arb — information lag plays on the high-edge sectors above. Requires separate agent architecture (12-agent signal bus design already scoped).

---

## Infrastructure Reuse from Cented/Solana

~60–70% of the Cented wallet classifier infrastructure is reusable here. The wallet monitoring pattern, signal classification logic, and Kelly sizing framework are all portable. The main differences are chain (Polygon vs Solana), API layer (Polymarket CLOB vs Helius), and execution mechanism (ERC-20 vs SPL tokens).

---

## Environment Variables Required

```
POLY_PRIVATE_KEY=    # Burner wallet only — never your main wallet
POLY_API_KEY=        # From polymarket.com/settings
ALCHEMY_RPC=         # Polygon mainnet via Alchemy
ANTHROPIC_API_KEY=   # For Phase 2 AI layer
```

---

## Hard Rules Before Going Live

1. Paper mode must show consistent positive PnL over **minimum 50 trades**
2. Validate leaderboard scanner accuracy against known wallet PnLs first
3. Start with profitable wallet tracking only — do not layer in anomaly or fade strategies until base is validated
4. Never use your main wallet — burner wallet only
5. Position cap: $50 max per trade, 10 concurrent max

---

## Session Start Checklist

1. This bible loaded
2. Confirm with Jeeps: are we building Phase 1 further, validating paper mode, or designing Phase 2/3?
3. If paper mode not yet at 50 trades — do not discuss live deployment
4. Architecture decisions FIRST before any new code
