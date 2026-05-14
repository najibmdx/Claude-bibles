# MemecoinAnalyzer_Bible.md
_Living document. Updated after every case._
_Last updated: 2026-05-15 (Sessions 1+2 merged — PAIN full analysis + on-chain corrections applied)_

---

## How This Bible Is Structured

**Layer 1 — Universal Mechanics** _(sections 1–8)_
True for every token, every dev, every case. Does not change case to case.
A finding graduates to Layer 1 only when confirmed across 3+ cases.
Single-case observations are hypotheses — marked `[HYPOTHESIS — n=1]`.

**Layer 2 — Case Intelligence** _(sections 9–11)_
What specific tokens showed. What specific devs did. What specific KOL networks looked like.
Every case-specific finding lives here, not in Layer 1.

**The test before writing anything into Layer 1:**
_"Could a future token contradict this?"_
If yes → case-specific or hypothesis. If no → universal.

---

# LAYER 1 — UNIVERSAL MECHANICS

---

## 1. The Core Thesis

Every pump above ~$500K MC on a pump.fun token has a cause. That cause is always one of:
- **Organic narrative** — the name/concept catches fire independently
- **KOL amplification** — paid or unpaid influencer posts
- **Coordinated insider operation** — pre-seeded wallets, scheduled pump, planned exit
- **Bot momentum** — automated trading creates artificial volume that attracts retail
- **Cross-community spillover** — related token pumps, community migrates

Most $1M+ pumps are combinations. The forensics job is to find which combination and prove it on-chain.

---

## 2. Operation Typology

### Type 1: Clean KOL Pump
- Dev creates token, pays 1-3 KOLs in USDC/SOL
- KOLs post, retail buys, dev collects creator fees
- Signature: USDC payments from dev wallet → unknown wallets → Phantom forward → silence
- Typical scale: $200K–$2M ATH
- Example: PAIN (Case 001)

### Type 2: Serial Operator
- Same dev wallet creates 5-10 tokens in rapid succession
- Most fail, 1-2 get KOL treatment
- Cross-token coordination: creator fees from Token A fund KOL budget for Token B
- Signature: Multiple CREATE events in dev wallet, several sibling tokens at $0 MC
- Example: PAIN dev (5dirQCE7Dvn3Jopo4e4ra2fcdY7imDdJ1SYTaGmnYmwt) — also ran USDUC, ASSDAQ

### Type 3: Insider Cluster
- Dev pre-seeds 5-20 wallets with tokens before any public activity
- Wallets look unrelated but received token from same source
- All wallets dump in coordinated window
- Signature: Token transfers FROM dev/insider TO fresh wallets 12-72 hours before ATH
- Example: EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp → 7jkA9zvNB84319exePVn5Cu3kYd4oPF1rW8pobZyDhtX pre-load, PAIN

### Type 4: Bot-Manufactured Volume
- Single automated address (fee payer) accounts for 50%+ of tx volume
- Creates illusion of activity, triggers algorithm-driven FOMO buying
- Signature: 1-3 fee payers in 80%+ of sampled transactions
- Example: FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2vEMd3cxGS — 80% of PAIN order flow at ATH

### Type 5: Dual-Token Operation
- Dev runs two tokens simultaneously, uses one to cross-promote the other
- Sophisticated: requires KOL buy-in on both narratives
- Signature: Sibling token with similar launch timing and holder patterns
- Example: PAIN + USDUC running in parallel

### Type 6: Organic
- No USDC payments from dev wallet
- No coordinated pre-loads
- Top holders have real transaction history
- Tx volume comes from diverse fee payers
- Rare above $1M MC

---

## 3. How pump.fun Creator Fees Work

**This is a universal mechanic. Understanding it is required before analysing any WITHDRAW event.**

pump.fun pays token creators a fee on every trade through the bonding curve. These accumulate on-chain and are claimed via a `CollectCreatorFee` instruction. On-chain this appears as:
- `source: PUMP_FUN`
- `type: WITHDRAW`
- `Program log: Instruction: CollectCreatorFee`
- SOL flowing INTO dev wallet FROM bonding curve program accounts

**This is normal, entitled behaviour under pump.fun's model. It is NOT manipulation, extraction, or LP removal.**

### What an actual rug/LP removal looks like:
- Sudden complete LP removal in a single transaction
- Price goes to near-zero immediately after
- Instruction log shows LP removal, not fee collection
- Very different account delta pattern from fee collection

### The mandatory verification rule:
**Never characterise a PUMP_FUN WITHDRAW event without first checking `getTransaction` with `jsonParsed` encoding and reading `meta.logMessages` for the instruction name.**

`CollectCreatorFee` = normal fee income. Document the amount and move on.
Anything else = investigate further before drawing conclusions.

### Why this matters for case analysis:
A dev collecting large creator fees is not evidence of manipulation — it is evidence of high trading volume, which is itself worth explaining. The manipulation, if present, is in what CAUSED the volume (KOL payments, insider pre-loads, bot amplification) — not in the fee collection itself.

---

## 4. Dev Wallet Behavioral Patterns

### The Fee Collector (most common on pump.fun)
- Creates token, earns SOL via `CollectCreatorFee` as trading volume accrues
- Higher volume = more fees. Incentive to drive volume via KOLs.
- WITHDRAW events = fee collection, NOT a rug
- Verify: always check instruction log on any WITHDRAW before concluding

### The Dumper
- Creates token, holds % of supply
- Sells tokens via swap (SWAP transactions, not WITHDRAW)
- Often uses Jupiter or OKX DEX Router for better execution
- Evidence: large SWAP events from dev wallet involving the token mint

### The Hybrid `[OBSERVED ON PAIN — verify on future cases]`
- Collects creator fees AND pays KOLs to drive more volume
- Funds KOL payments from converted creator fees via USDC
- Flow: trading volume → creator fees (SOL) → DEX swap → USDC → KOL payments → more volume → more fees
- Self-financing loop: the operation pays for itself

### Exit Signal (universal across all types)
- Dev SOL balance drops to <0.01 SOL
- No new PUMP_FUN interactions for 24+ hours
- All USDC converted and forwarded out
- = Operation complete or abandoned

---

## 5. The KOL Payment Fingerprint

Paid KOL operations leave a consistent on-chain signature:

```
Dev wallet
  └─ USDC transfer to Wallet A  (payment)
       └─ Wallet A forwards to Wallet B via Phantom  (within 5-30 min)
            └─ Wallet B goes silent  (actual KOL's cold receiving wallet)
```

**The Phantom forward is the tell.** Legitimate OTC transactions do not forward within 6 minutes. The forward is the KOL separating their social identity from their payment trail.

### Payment Amount Tiers `[HYPOTHESIS — n=1, based on PAIN only]`
```
$50-200:   micro-KOL, CT account <10k followers
$200-500:  mid-tier KOL, 10k-50k followers
$500-2000: tier-1 KOL for this ecosystem, 50k+ followers
$2000+:    coordinated multi-platform push
```
_Treat as hypothesis until confirmed across 5+ cases._

### Payment → ATH Timing `[HYPOTHESIS — n=1, based on PAIN only]`
Payment → Phantom forward → ATH: observed at 19 minutes on PAIN.
Likely range: 15-40 minutes. Needs confirmation across more cases.

### USDC Funding Flow `[OBSERVED ON PAIN — verify on future cases]`
```
pump.fun trading volume
  └─ Creator fees accumulate (CollectCreatorFee)
       └─ Dev wallet receives SOL
            └─ SOL → USDC via DEX router wallet
                 └─ USDC distributed to KOL wallets
                      └─ Phantom forward to actual KOL receiving wallet
```
On PAIN: dev received $991.77 USDC at 03:18 UTC from router wallet `ARu4n5mFdZogZAravu7CcizaojWnS6oqka37gdLT5SZn`, then distributed it as $150 + $150 + $691.

---

## 6. Tx Volume as Price Proxy

When price data is unavailable, tx density per minute ≈ price action:

| Tx/minute | Market state |
|---|---|
| <10 | Dead / no interest |
| 10-100 | Slow accumulation |
| 100-500 | Growing momentum |
| 500-2,000 | Active pump phase |
| 2,000-5,000 | ATH window / peak FOMO |
| 5,000+ | Bot-dominated, organic buying peaked |

_Observed on PAIN: peaked at 4,618 tx/min at 04:35 UTC._

---

## 7. Bot Amplification — Universal Reference

### Bot Types Operating on Solana Memecoins

**Type A: MEV Sandwich Bots**
Pure extractors. No directional view on the token.
Mechanic: detect pending retail tx → insert buy before → let retail execute at worse price → sell immediately after.
Signature: same address as fee payer on majority of transactions, same-block as sandwiched txs.
Impact: guaranteed tax on every retail entry and exit.

**Type B: Momentum Bots**
Directional. Buy into detected signals, sell into FOMO.
Signals: tx density spikes, known KOL wallet movements, large buys from fresh wallets.
Signature: uniform trade sizes, millisecond execution, wallet age hours to days old.
Impact: create the initial price spike retail misreads as organic momentum.

**Type C: Volume Wash Bots**
Trade with themselves to inflate volume metrics.
Signature: same two addresses trading back and forth, volume without proportional price movement.
Impact: inflated Dexscreener volume numbers.

**Type D: Copy Trade Bots**
Mirror tracked profitable wallets automatically within the same block.
Signature: cascade of identical-sized buys in same block as a large buy.
Impact: amplifies whale entries and exits in both directions.

### How Jito MEV Works on Solana

Solana has no traditional public mempool. Jito introduced a private block engine where searcher bots submit transaction bundles to be included in a specific order within a block. Validators running Jito (majority of Solana validators) accept these bundles for tips.

```
Retail tx submitted
    │
    ▼
Jito block engine sees it before chain confirmation
    │
    ▼
Searcher bots bid to sandwich it in the same block
    │
    ▼
Highest bidder: [bot buy] [retail buy] [bot sell] — all same block
    │
    ▼
Bot pays validator tip. Bot keeps spread profit.
Retail gets worse execution. No consent required.
```

### Fee Payer Concentration as Risk Signal

Sample 20 recent token transactions. Calculate top fee payer share.

| Top fee payer % | Meaning | Action |
|---|---|---|
| <20% | Diverse organic flow | Minimal sandwich risk |
| 20-50% | Moderate bot presence | Expect sandwich tax, factor into sizing |
| 50-80% | Heavy bot dominance | Every trade taxed significantly |
| >80% | Pure extraction zone | Do not enter — you are exit liquidity |

### The Bot-KOL Interaction Loop

```
KOL posts (T+0)
    │
    ▼
Momentum bots detect post or initial price move (T+milliseconds)
    │
    ▼
Bots front-run: buy before any human can react (T+seconds)
    │
    ▼
Price moves 10-20% instantly
    │
    ▼
KOL's audience sees both chart moving AND the post (T+1-3 min)
    │
    ▼
Human FOMO buying begins (T+3-15 min)
    │
    ▼
Sandwich bots extract value from every retail buy (ongoing)
    │
    ▼
Dev collects largest creator fee batch near peak (T+19-25 min) [HYPOTHESIS]
    │
    ▼
Momentum bots sell into retail (T+20-30 min)
    │
    ▼
Insider wallets dump token bags (T+30-90 min) [HYPOTHESIS]
    │
    ▼
Price collapses. Retail holds bags.
```

**Peak tx/min = peak fee collection moment = worst possible entry for outsiders.**

---

## 8. The Golden Hour Framework

_Derived from PAIN Case 001. Core logic is universal._
_Specific timing numbers are hypotheses until confirmed on 3+ cases._

### The Core Insight

Price action is the last thing to move. On-chain state moves first — always.

**The question is never "should I buy this?" — it is "where is this operation in its lifecycle?"**

### The Four Variables That Create a Professional Pump `[HYPOTHESIS — n=1]`

```
Good Name × Hot Macro Moment × Paid Distribution Network × Bot Amplification = $1M+ ATH
```

Remove any one variable and the result degrades significantly.

### Operation Lifecycle Stages

```
STAGE 1: LAUNCH (Day 0)
Token created on pump.fun. Dev begins fee collection immediately.
Tx/min: <100. Chart: flat/dead.
On-chain tell: first CollectCreatorFee WITHDRAW events appearing.

STAGE 2: GRIND (Day 1-2) [HYPOTHESIS — timing based on PAIN]
Dev collecting fees as slow organic volume builds.
Tx/min: 100-500. Chart: slow upward grind with pullbacks.
On-chain tell: increasing withdrawal frequency.

STAGE 3: ARMING (12-24 hours before ATH) [HYPOTHESIS — timing based on PAIN]
Insider pre-loads begin. Tokens distributed to KOL wallets
before any USDC payment fires.
Tx/min: 300-600. Chart: still looks like a grind.
On-chain tell: token transfers FROM known insider TO unknown wallets.
This is the loudest pre-ATH signal available.

STAGE 4: GOLDEN HOUR (0-20 min before ATH) [HYPOTHESIS]
USDC payments fire. Phantom forwards within 5-10 minutes.
KOL post goes live. Bots front-run. First explosive candle.
Tx/min: 500 → 2,000+ rapidly.

STAGE 5: ATH WINDOW (ATH ± 10 min)
Dev collects largest single fee batch.
Bot fee payer concentration: 60-80%+.
Tx/min: 2,000-5,000+. Chart: vertical green candle.
Worst risk/reward for any new entry.

STAGE 6: DECAY
Dev SOL balance declining toward zero.
Insider wallets dumping token bags.
Tx/min: declining. Operation winding down.
```

### The Pre-Entry Checklist

All five must be TRUE simultaneously. One false = do not enter.

```
[ ] 1. DEV WALLET SOL BALANCE > 0.1 SOL
        Operation still live. Dev has not fully exited.
        Check: getBalance on dev wallet via Helius RPC.

[ ] 2. USDC PAYMENTS HAVE NOT FIRED YET
        You are pre-catalyst. Maximum available runway.
        Check: scan dev wallet last 20 txs for USDC outflows.
        If USDC fired <20 min ago: narrow window, high risk.
        If USDC fired >40 min ago: likely post-ATH. Do not enter.

[ ] 3. FEE PAYER CONCENTRATION < 50%
        Order flow not yet bot-dominated.
        Check: sample 20 recent token txs, count unique fee payers.

[ ] 4. TX/MIN IN RANGE 100-2,000
        Building but not peaked.
        Check: count signatures in last 60 seconds.

[ ] 5. INSIDER PRE-LOAD DETECTED OR WITHDRAWAL RATE ACCELERATING
        At least one signal the operation is being actively armed.
        Either: token transfer FROM known insider TO unknown wallet, OR
        dev WITHDRAW frequency increased significantly vs prior day.
```

### The Exit Rules

Exit triggers are on-chain and time-based. Never price-based.

**Primary trigger:** First USDC payment detected from dev wallet.
Start 20-minute timer. Begin scaling out. Full exit by timer expiry.

**Secondary trigger:** 20 minutes from entry if USDC still hasn't fired.
If in Stage 3 and no USDC after 20 minutes, reduce position.

**Hard stop:** Dev SOL balance drops below 0.05 SOL at any point.
Exit immediately regardless of price.

**Never hold past:** Any large token dump from a known insider wallet in the registry.

### The PAIN Simulation Baseline `[SINGLE CASE — reference only]`

| Time (UTC) | Event | Price | Action |
|---|---|---|---|
| 05-12 02:00 | Initial analysis — no signals | ~$80K | Watch only |
| 05-12 12:55 | Dev wallet healthy, no USDC fired | $110K | Enter |
| 05-13 16:14 | EgSptShw pre-loads 7jkA9z with 75k tokens | ~$350K | Tighten watch |
| 05-14 04:03 | First USDC payment fires ($150) | ~$800K | Begin exit |
| 05-14 04:11 | Main USDC payment fires ($691) | ~$1.2M | Full exit |
| 05-14 04:30 | ATH | $2.61M | Already out |

Entry $110K → Exit $800K–$1.2M = +627% to +990%.
No price action signal used at any decision point.

---

## Helius API Technical Notes

- `getSignaturesForAddress` returns max 1,000 per call — use `before` param to paginate
- Hot tokens generate 200k+ sigs in 90 minutes — cannot paginate to creation during peak activity
- Strategy: use dev wallet CREATE event timestamp to anchor the start, paginate forward
- Token-2022 blocks `getProgramAccounts` — always use `getTokenLargestAccounts` for holders (top 20 cap)
- DAS `getAsset` gives price_info for any token — quick MC check on sibling tokens
- **Always use `getTransaction` with `jsonParsed` encoding to verify instruction type before drawing conclusions about any WITHDRAW event**

---

# LAYER 2 — CASE INTELLIGENCE

---

## Case Registry

---

### CASE 001 — PAIN (Max Pain)
**Analysis dates:** 2026-05-14 (initial) + 2026-05-15 (on-chain corrections applied)
**Token:** `2N9BXbRtyZM4YHzj9wvteW1Nuq6B5DzDwh57dDmZpump`
**Dev:** `5dirQCE7Dvn3Jopo4e4ra2fcdY7imDdJ1SYTaGmnYmwt`
**Token standard:** Token-2022
**Created on:** pump.fun
**Launch date:** 2026-05-11 14:07 UTC

**ATH:** ~$2.61M MC at 04:30 UTC 2026-05-14
**MC at analysis:** ~$196K
**Dev creator fees collected:** 479 SOL across 73 `CollectCreatorFee` transactions
_(Previously logged as "363+ SOL extracted" — corrected after on-chain verification of sig `46yQACMYrhrvV3DgMUjGnVQw7jBqZjn5wLB32e2Z7UkMs5pnt94Xi2yjMHun1yECoWsqpkB9YKvLHMjbE886hJZv` confirmed `Instruction: CollectCreatorFee`. These are entitled creator fees, not LP removal.)_
**Operation types:** Type 1 + Type 2 + Type 3 + Type 4

**Smoking gun:** $691 USDC payment from dev at 04:11:51 UTC → Phantom forwarded at 04:17:45 UTC → ATH hit at 04:30 UTC. 19-minute window from payment to peak. Confirmed on-chain.

---

**KOL Payment Chain — CONFIRMED ON-CHAIN:**

| Time (UTC) | Amount | From | To | Notes |
|---|---|---|---|---|
| 03:18 | $991.77 USDC | `ARu4n5mFdZogZAravu7CcizaojWnS6oqka37gdLT5SZn` | Dev | DEX router converts creator fees → USDC. This funds the entire KOL budget. |
| 04:03:40 | $150 USDC | Dev | `7jkA9zvNB84319exePVn5Cu3kYd4oPF1rW8pobZyDhtX` | KOL #1 |
| 04:03:40 | $150 USDC | Dev | `EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp` | Insider / KOL #2 |
| 04:11:51 | $691 USDC | Dev | `4QfGpNuC5eeM9ieEp4qvFH49uWm4WqKTU4GQGvKZZyUc` | Main KOL payment collector |
| 04:17:45 | $691 USDC | `4QfGpNuC5eeM9ieEp4qvFH49uWm4WqKTU4GQGvKZZyUc` | `6bZHJzbApPZck2gauwxmsEGkzPGZxxawoNuEFV1HtbDr` | Phantom forward — actual KOL cold wallet |

Total KOL spend: $991 USDC. Self-financing: creator fees → USDC → KOL payments → more volume → more fees.

---

**Insider Pre-load — CONFIRMED ON-CHAIN:**

- From: `EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp`
- To: `7jkA9zvNB84319exePVn5Cu3kYd4oPF1rW8pobZyDhtX`
- Amount: 75,000 PAIN tokens
- Time: 2026-05-13 16:14:39 UTC (13h 45m before ATH)

**Prior relationship — NEW FINDING FROM SESSION 2:**
`EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp` sent `7jkA9zvNB84319exePVn5Cu3kYd4oPF1rW8pobZyDhtX` 20 USDC on 2026-04-25 — six weeks before PAIN. These are recurring counterparties, not a one-off contact.

---

**Holder Concentration:**

| Rank | Tokens | % of supply |
|---|---|---|
| Top 1 | 92.1M | 9.21% |
| Top 2–5 | 72.6M | 7.26% |
| Top 6–10 | 59.2M | 5.92% |
| Top 11–20 | 100.4M | 10.04% |
| **Top 20 total** | **326.4M** | **32.6%** |

All top 10 owner wallets show 0 SOL balance — burner/coordinated accounts.

---

**Bot Activity:**
`FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2vEMd3cxGS` — fee payer on 16 of 20 sampled transactions at peak (80%). Jito-adjacent MEV sandwich infrastructure.

---

**Dev Sibling Tokens — CONFIRMED SAME DEV WALLET:**

| Token | Symbol | MC at PAIN analysis | Notes |
|---|---|---|---|
| `CB9dDufT3ZuQXqqSfa1c5kY935TEreyBw9XJXxHKpump` | USDUC | $7.2M (live) | Also reached $29M after Binance.US listing. Primary vehicle. Case 002. |
| `7Tx8qTXSakpfaSFjdztPGQ9n2uyT1eUkYz7gYxxopump` | ASSDAQ | $290K | Live at analysis |
| `2KiHzSXdnenDoDNsVsjU6VcvgyyDK27iSoZv6TNDpump` | poop | $0 | Failed |
| `yFtG6tU4mZqiUnrBC4gF9eXTdWyzj7ZSKb92wv3bonk` | JAIL | $0 | Failed |
| `AzsiFNsZniJXuB1v6mxYCY9dDVUXm5EC57iDCbV6pump` | Assem | $0 | Failed |
| `HF5DkVyJDAtSPfjrS551251xKw1mRRuhmUCcZcBqbonk` | pennystock | $0 | Failed |
| `7narYDCxG2HnmgJnJigCjES64sMzTL2MVCECXd4Gpump` | Bigotón | $732 | Failed |

7 tokens total. 2 live (PAIN + USDUC). 5 failed. Dev ran 3 simultaneously at PAIN's ATH.

---

**7 Question Answers:**

1. Manipulated? **Yes** — coordinated KOL payments timed to drive trading volume
2. Coordinated? **Yes** — multi-wallet pre-load confirmed, timed USDC payments confirmed, prior counterparty relationship confirmed
3. Natural? **No**
4. Community-driven? **No**
5. Real-world event? **No** — "max pain" is a financial meme, not a live event
6. Celebrity? **No**
7. KOL/influencer? **Yes** — at least 3 paid KOLs, $991 total, main KOL received $691 via Phantom forward

**Verdict:** Professional KOL pump. Dev paid $991 USDC (funded from creator fees) to drive volume. Earned 479 SOL in creator fees as result — self-financing loop. Manipulation is in coordinated KOL payments and insider pre-positioning, not in LP removal. Dev is a serial operator with proven infrastructure. PAIN was the smaller parallel play; USDUC was the primary vehicle.

---

### CASE 002 — USDUC (unstable coin)
**Status:** PENDING
**Token:** `CB9dDufT3ZuQXqqSfa1c5kY935TEreyBw9XJXxHKpump`
**Dev:** `5dirQCE7Dvn3Jopo4e4ra2fcdY7imDdJ1SYTaGmnYmwt` (same as PAIN)
**Key question:** Same dev, same infrastructure — PAIN died at $196K, USDUC reached $29M and got Binance.US listed. What was architecturally different?

---

## Known Wallets Registry

### Dev Wallets

| Wallet | Alias | Confirmed Tokens | Last Active | Notes |
|---|---|---|---|---|
| `5dirQCE7Dvn3Jopo4e4ra2fcdY7imDdJ1SYTaGmnYmwt` | SerialDev_01 | PAIN, USDUC, ASSDAQ + 4 failed | 2026-05-14 | Professional operator. Dual-token strategy. KOL network confirmed. Creator fee model. Self-financing operation. |

### Insider / KOL Network Wallets

| Wallet | Role | Connected To | Notes |
|---|---|---|---|
| `EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp` | Insider + KOL relay | SerialDev_01 | Received $150 USDC from dev. Pre-loaded 7jkA9z with 75k PAIN tokens 13h before ATH. Prior USDC relationship with 7jkA9z in April 2026. May be dev's own secondary wallet. |
| `7jkA9zvNB84319exePVn5Cu3kYd4oPF1rW8pobZyDhtX` | KOL #1 | SerialDev_01 + EgSptShw | Pre-loaded with 75k tokens 13h before ATH. Received $150 USDC from dev. Received 20 USDC from EgSptShw in April 2026. Recurring counterparty. |
| `4QfGpNuC5eeM9ieEp4qvFH49uWm4WqKTU4GQGvKZZyUc` | KOL payment collector | SerialDev_01 | Received $691 USDC at 04:11. Phantom-forwarded to 6bZHJzbA within 6 minutes. Acts as payment relay not final destination. |
| `6bZHJzbApPZck2gauwxmsEGkzPGZxxawoNuEFV1HtbDr` | KOL final destination | 4QfGpNuC5eeM9ieEp4qvFH49uWm4WqKTU4GQGvKZZyUc | Received $691 USDC via Phantom at 04:17. Silent after receipt. Identity unknown — likely CT/Telegram alpha account. |
| `ARu4n5mFdZogZAravu7CcizaojWnS6oqka37gdLT5SZn` | DEX router / USDC conversion | SerialDev_01 | Converted creator fee SOL to $991.77 USDC at 03:18. Funded the entire KOL budget. |

### Bot / Automated Addresses

| Wallet | Type | Seen On | Notes |
|---|---|---|---|
| `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2vEMd3cxGS` | MEV Sandwich Bot (Jito) | PAIN | 80% fee payer share at ATH window. Cross-token presence to be verified on USDUC. |

---

## Open Questions

_Mark RESOLVED when answered with on-chain evidence._

1. **Who is `6bZHJzbApPZck2gauwxmsEGkzPGZxxawoNuEFV1HtbDr`?**
Final $691 USDC destination. Identity unknown. Likely CT/Telegram alpha account with 20-50k followers. Social media cross-reference pending.

2. **Is `EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp` the dev's own secondary wallet?**
Dual role as insider bag holder + received USDC from dev + pre-loaded 7jkA9z + prior April USDC relationship with 7jkA9z. Strongly suggests same entity or very close associate. Requires deeper history investigation.

3. **Who created the Meteora DLMM pool on PAIN and when?**
Meteora DLMM LP detected in token transactions. Suggests either dev has DeFi infrastructure experience or a paid third party (market maker) added LP.

4. **Total creator fee income across PAIN + USDUC for `5dirQCE7Dvn3Jopo4e4ra2fcdY7imDdJ1SYTaGmnYmwt`?**
479 SOL confirmed from PAIN. USDUC reached $29M — total income could be 2,000+ SOL across both tokens.

5. **Does the 19-minute rule hold across different operators?**
One data point (PAIN only). Hypothesis until confirmed on 5+ cases.

6. **What is the April 2026 history of `EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp` and `7jkA9zvNB84319exePVn5Cu3kYd4oPF1rW8pobZyDhtX`?**
20 USDC transaction six weeks before PAIN. Were they working together on a previous token?

7. **Does `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2vEMd3cxGS` appear on USDUC and other tokens?**
If yes, confirms cross-token infrastructure-level bot. To be checked during USDUC analysis.

---

## Research Agenda

### Next: Case 002 — USDUC
Same dev, same infrastructure, escaped pump.fun gravity, reached $29M, got exchange listings.
Primary question: what was architecturally different from PAIN?

### Cases 003–012 — Deliberate Selection

| Case | What to find | Framework question |
|---|---|---|
| 003 | $1M+ pump, different dev | Does pre-load pattern appear without SerialDev_01? |
| 004 | Organic pump, no KOL payments | What does clean organic look like on-chain? |
| 005 | Failed operation — no ATH | What separates failure from success pre-catalyst? |
| 006 | Token where `4QfGpNuC5eeM9ieEp4qvFH49uWm4WqKTU4GQGvKZZyUc` or `6bZHJzbApPZck2gauwxmsEGkzPGZxxawoNuEFV1HtbDr` reappear | KOL network cross-token fingerprinting |
| 007 | Sub-1-hour ATH from launch | Does pre-load pattern compress to match? |
| 008 | 1-2 week slow burn to ATH | Does signal window extend proportionally? |
| 009 | Celebrity-driven pump | How does on-chain differ when catalyst is public? |
| 010 | Second pump after first ATH | What triggers re-pump? Same or different KOL network? |
| 011 | Current live token pre-ATH | First real-time framework test under live conditions |
| 012 | Any token where `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2vEMd3cxGS` reappears | Bot infrastructure cross-token confirmation |
