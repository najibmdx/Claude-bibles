# THE FIRM — Bible
*Last updated: May 2026 | Load this for any strategy, hiring, structure, or firm-level discussion*

---

## What The Firm Is

A crypto-native quantitative trading firm based in Singapore, with the long-term ambition of reaching institutional scale modeled on Wintermute and Jane Street. It operates across proprietary trading (Solana MEV, HFT, memecoin microstructure, prediction markets), intelligence tool development, and adjacent ventures.

**Founder:** Jeeps — PhD-level background in mathematics and physics. Singapore-based.

---

## The Dual-Flywheel Architecture

This is the firm's strategic engine. Two flywheels that fund each other:

```
Flywheel 1 (Prop Trading)          Flywheel 2 (Exchange)
─────────────────────────          ────────────────────────
Solana MEV                    →    Hyperliquid-style
HFT / Memecoin microstructure →    Solana-native
Prediction markets            →    Perpetuals exchange
Market making                 →    On-chain order books
        ↓                               ↓
Trading profits fund          →    Exchange fees dwarf
infrastructure build               prop arm revenue
```

**The logic:** Prop trading generates the capital and proves the edge. The exchange product generates fee revenue independent of alpha — that's the $10B+ path. Neither flywheel works alone at scale.

**Critical warning:** The $10B outcome is only achievable if the exchange achieves Hyperliquid-level product-market fit by Year 3–4. Without the exchange flywheel, you plateau at $200–500M as a pure prop shop — still excellent, but not transformational.

---

## The Four Flywheels (Detailed)

1. **Trading profits** fund infrastructure build
2. **Infrastructure** generates proprietary data and edge
3. **Edge** attracts institutional capital (fund structure)
4. **AUM** funds the exchange layer (perps) which generates fee revenue independent of alpha

Each flywheel must be validated before funding the next. Never skip stages.

---

## Proof of Concept Threshold

**Sharpe ratio >2.5 on total capital in Year 1** is the only metric that matters before anything else.

- Sharpe >3 = edge is real, risk management working, scale aggressively
- Sharpe 1–2 = making money but edge is thin, find the leak before scaling
- Sharpe <1 = taking too much risk relative to returns, stop and diagnose

This single number, sustained above 2 for 24 months, is what gives you credibility to attract the best talent, best counterparties, and external capital if you ever want it. **Revenue is not the proof of concept. Sharpe is.**

---

## MEV Expansion Sequencing

Build in this order. Do not skip ahead:

1. **Solana first** — home turf, existing infrastructure, known microstructure
2. **BSC second** — high volume, fat spreads, less competitive than EVM chains
3. **Arbitrum / Base third** — EVM ecosystem, more competition but large markets
4. **Sui fourth** — emerging, early mover advantage window closing

---

## Hiring Philosophy (Non-Negotiable)

**Hire physicists and mathematicians. Not finance veterans.**

The Renaissance Technologies model: hire for raw intelligence in adjacent hard sciences. The people who unlocked Renaissance's equity markets were speech recognition researchers from IBM — not traders.

**Hire profile:**
- PhD or strong MSc in mathematics, physics, statistics, or computer science
- Academic crossovers preferred over industry veterans
- Must be able to build what they design — no pure theorists
- Early hires get profit participation points (not salary-first), vesting over 4 years

**Why not finance veterans:**
- Finance veterans carry institutional baggage and consensus thinking
- They optimize for their own career, not the firm's edge
- The edge comes from seeing markets as a physics problem, not a finance problem

**Incentive structure:**
- Profit participation > salary for senior talent in early stages
- Points-based model for early hires — stake in the upside vehicle, not the operating company
- Everyone invested in the same fund they work on (Renaissance model)

---

## Legal and Fund Structure

**Jurisdiction:** Singapore — MAS regulated. This is the legitimacy layer.

**Entity structure:**
```
[For-Profit Fund Manager] ← MAS regulated, full control
        ↓ yield distribution
[Non-Profit Distribution Entity] ← pass-through only
        ↓ quarterly distributions
[Singapore Mosques / MUIS] ← beneficiary, not stakeholder
```

**The firewall principle:** MUIS is a beneficiary, not a stakeholder. They receive quarterly distributions. They do not have visibility into the fund's operations, strategies, or positions. MAS regulation is the legitimacy — MUIS oversight is not required and must not be granted. "You are a beneficiary, not a stakeholder. Beneficiaries receive, they don't govern."

**Fund structure:**
- VCC (Variable Capital Company) — cleanest structure for multi-strategy funds in Singapore
- Management fee 1–2% covers operations; carry 15–20% is where wealth is made
- LP/GP separation creates liability firewall and tax efficiency
- Side pockets for illiquid positions (on-chain assets, early-stage tokens)
- Halal investment screening required for the mosque distribution structure

**Security:** Two independent audits minimum before any exchange mainnet launch. Bug bounty from day one. A single exploit before trust is built is existential — not optional.

---

## Models We Study

**Renaissance Technologies** — hire adjacent hard scientists, one model everyone works on together, incentive alignment (employees invested in same fund they run). 39% annual returns after fees, zero losing years in 34 attempts.

**Jane Street** — market maker as information processor, incremental expansion into neighboring markets, culture as the secret weapon. Started with ADRs on AMEX, expanded systematically to ETFs, then everything else.

**Wintermute** — crypto-native prop desk, institutional market making, systematic MEV. The operational model for Year 1–2.

**Hyperliquid** — exchange product with community-first token launch, on-chain order books, fee revenue that dwarfs prop arm. The exchange flywheel target for Year 3–4.

---

## What Would Kill This

Be brutally honest about these risks:

1. **Exchange security** — A single exploit before trust is built is existential. Non-negotiable: two independent audits before mainnet.
2. **Regulatory sequencing** — Don't repeat CZ's mistake. Structure first, build second. MAS compliance is the foundation.
3. **Hiring the wrong first five** — Early hires set culture permanently. One finance veteran who optimizes for personal career over firm edge can corrupt the culture before it forms.
4. **Skipping flywheel validation** — Don't build the exchange before prop trading generates consistent Sharpe >2.5. Don't raise external capital before proof of concept. Sequence matters.
5. **Execution velocity** — The binding constraint is not intelligence or ideas. It is focus depth and execution speed. Every unfinished project is a tax on the firm's compounding.

---

## Current Project Portfolio (May 2026)

**Active / Live:**
- Spider — memecoin dump risk analyzer (live)

**In build:**
- Skope — forensic token anatomy (approaching build phase)

**Architected, not built:**
- Cented classifier — KOL wallet tracker (architecture validated against 503 trades)
- Polymarket copy bot — Phase 1 scaffold built, paper mode not yet validated
- Beat — social sentiment agent (concept stage)
- Tide — order flow / liquidity monitor (concept stage)

**Built, not live:**
- TACO Watch — market surveillance dashboard (React UI built, backend not built)

---

## Session Rules for Firm-Level Discussions

- No hype framing. Evidence-based reasoning only.
- Never present unbuilt projects as revenue-generating.
- Strategy and architecture first. Execution only after explicit approval.
- Brutal honesty preferred. No hedged advice.
- The firm vision is 10–15 years. Don't optimize individual decisions for short-term optics.
