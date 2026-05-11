# CENTED CLASSIFIER — Project Bible
*Last updated: May 2026 | Paste this at the start of any Cented session*

---

## What It Is

A real-time wallet classifier that monitors a single KOL wallet (Cented/@Cented7, address `CyaE1VxvBrahnPWkqm5VsdCvyS2QmNht2UFrKJHga54o`) and fires alerts based on their on-chain behavior patterns. The goal is to follow Cented's genuine runner trades and avoid their averaging-down losses.

**Status: Architecture fully designed and validated against 503 real trades. Not yet built.**
**Build sequence: Alert system first → live observation mode → small-size copy execution.**

---

## The Core IP — What Cented's Edge Actually Is

From 503-trade dataset analysis (5 days, Day 4 anomaly: 209 SOL, 167 trades, 62% win rate):

**Genuine edge: Runner scaling** — small probe entry, rapid adds into rising price action within 10–30 seconds.

**The signal:**
- **RUNNER CONFIRMED** fires when second add arrives within ~15 seconds of first add at a higher price than the first add
- This is the follow signal. Fire FOLLOW alert immediately.

**The loss pattern (do NOT follow):**
- All worst losses (14–18 adds, -12 SOL, -6 SOL) showed AVERAGING DOWN pattern
- Adds coming slow, price falling between adds
- AVERAGING DOWN fires on this pattern — stand down regardless of how many subsequent adds come in

---

## 5-State Classifier

| State | Trigger | Action |
|---|---|---|
| **RUNNER CONFIRMED** | Second add within ~15s at higher price | FOLLOW immediately |
| **POSSIBLE RUNNER** | Second add within 15–45s, price flat or slightly up | WATCH, await third add |
| **AVERAGING DOWN** | Second add at lower price than first | STAND DOWN — do not follow |
| **WAIT** | Second add after 60+ seconds | Momentum slowing — hold |
| **ABANDONED** | No second add within 120 seconds | Probe discarded — stand down |

---

## Exit Detection (Protects Open Positions)

Once following Cented in a position:

| Signal | Trigger | Action |
|---|---|---|
| **FIRST EXIT** | First sell on a token you're holding | Move stop loss up (not panic sell) |
| **ACCELERATING EXIT** | Second sell within 60s of first | Exit now |
| **MAJOR EXIT** | Sell amount >50% of total deployed | Exit immediately if not already |
| **LOSS CUT** | Price dropping between his sell tranches | Exit immediately, don't wait |

---

## Technical Architecture

**RPC:** Helius paid tier (websocket subscription on Cented's wallet address)

**Critical insight — RPC efficiency:**
- Subscribe to ONE wallet address
- Only trigger additional RPC calls when Cented's wallet touches a NEW token mint
- This means ~10–30 RPC calls per session — not thousands, not all of Solana
- Per-new-mint calls: `getTokenLargestAccounts` or DAS API for holder concentration (~500ms latency)

**Why holder concentration matters:** This is the most important insider flag. If Cented buys a token where 3 wallets hold 80% of supply, that's an insider setup. Do not follow.

**Four data calls per new token:**
1. Helius websocket → Cented's wallet transactions (real-time feed)
2. Token balance check before entry (has he been holding this before his "entry"?)
3. Token metadata (launch time, holder distribution)
4. `getTokenLargestAccounts` or DAS API (concentration check — possible insider flag)

---

## Market Context Layer (Critical for Calibration)

**Day 4 was an outlier** — 209 SOL in one session because market conditions produced genuine runners, not just Cented's skill.

The classifier must track: **how many tokens did 3x+ in the last 30 minutes?**
- Hot market (many tokens running) → calibrate aggression UP
- Cold market (nothing running) → calibrate DOWN, many more AVERAGING DOWN losses expected

Without this layer, the classifier will over-fire on dead market days.

---

## Known Wallet Relationships (Cented Network)

From Cented + Cupsey network analysis (34 wallets mapped, 6 tiers):

Key insight: **BA36AnwpT** is a front-running wallet that enters BEFORE Cented announces — this is a pre-signal, not a copy-signal. If BA36AnwpT enters a token before Cented tweets, that's the highest-confidence buy signal in the network.

Wallet `T5/EbXDFLFkLoGT3d8RjhXiVbMknniaNUbjd2uzw7DtZMHE` — shared pass-through connecting Cented and Cupsey simultaneously. Structural link between the two KOLs.

The full 34-wallet CSV has been produced. Ask to load it if needed.

---

## Dead Period

Cented showed a **dead period post-March 24, 2026** — significantly reduced activity. Monitor whether this persists or represents a style change.

---

## Build Order (Confirmed)

1. **Rules-on-paper validation** — backtest the 5-state classifier against the 503 trades manually
2. **Helius websocket integration** — live feed, alert output only
3. **Live observation mode** — watch classifier fire on real trades, validate signals
4. **Market context layer** — add the 3x+ token counter
5. **Small-size copy execution** — only after live observation validates accuracy

**Never skip to execution before observation is validated.**

---

## Relationship to Other Projects

- **Polymarket copy bot** — same architecture pattern (wallet monitoring → classification → Kelly sizing → paper mode → live). ~60–70% infrastructure reusable.
- **Skope** — the 952-wallet DB overlaps with the Cented network map. Cented and network wallets are candidates for the Skope DB.
- **Spider** — separate concern. Spider analyzes tokens; Cented classifier analyzes wallet behavior.

---

## Session Start Checklist

1. This bible is loaded
2. Confirm with Jeeps: are we validating rules, building the feed, or building execution?
3. Never jump to execution layer without confirmation that observation mode has been validated
