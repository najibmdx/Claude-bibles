# MemecoinAnalyzer_Bible.md
_Living document. Updated after every case._
_Last updated: 2026-05-15 (Session 5 complete — HANTA full analysis + Arkham corrections applied)_

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
- Example: USDUC (Case 002), HANTA bonding curve phase (Case 003)

### Type 7: Dormant Narrative Revival `[NEW — confirmed Case 003]`
- Token launched months/years ago, sat dormant
- Real-world news event reactivates it
- Dev cashes out via CEX during bonding curve phase
- Post-graduation pump driven by professional traders responding to platform verification (e.g. Moonshot)
- Signature: Token age >3 months, all creator fees collected pre-graduation, Coinbase/CEX deposit as terminal cash-out
- Example: HANTA (Case 003) — launched July 2024, dormant 22 months, revived by cruise ship Hantavirus outbreak May 2026

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

### Graduation mechanics `[confirmed Case 003]`:
When a bonding curve fills to the graduation threshold (~$69K on old pump.fun), it emits `MigrateBondingCurveCreator` + `MigratePoolCoinCreator` instructions and migrates liquidity to an AMM pool. After graduation:
- Dev earns NO more creator fees
- Token trades on AMM (Raydium, pump.fun AMM) at open market price
- The real pump often happens POST-graduation on the AMM — dev has no fee stake in this phase

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

### The CEX Exit Dev `[NEW — confirmed Case 003]`
- Collects creator fees during bonding curve phase
- Routes SOL through relay wallets → USDC → trading aggregator
- Terminal destination = Coinbase/Binance deposit address (verified via Arkham)
- Has NO ongoing stake post-graduation — token pumps without them
- Signature: all creator fees collected pre-graduation, Arkham shows CEX deposit as final hop
- Dev alias: may have a pump.fun username (e.g. "fuckjerry") findable via Arkham label

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

### Distinguishing KOL payments from liquidation `[NEW — confirmed Case 003]`
Not every SOL → USDC flow from a dev wallet is a KOL payment. HANTA showed a dev using relay wallets → USDC → trading aggregator that had NO downstream KOL recipients. Key differentiators:
- **KOL payment**: USDC ends at a wallet that goes silent, has CT/Telegram identity, and/or Phantom-forwards within minutes
- **Liquidation**: USDC ends at a high-frequency aggregator (GMGN, LiquidMesh, Wintermute counterparty) or directly at a CEX deposit address
- Always trace 2+ hops AND verify terminal wallet activity before concluding KOL payment

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
_Observed on HANTA: peaked at 3,300+ tx/min across multiple ATH clusters over 36 hours._

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

### Fee Payer Concentration Shifts By Phase `[NEW — confirmed Case 003]`
A token's fee payer concentration is NOT static. It changes dramatically across lifecycle phases:
- **Bonding curve phase**: typically low (<20%) — organic retail discovery
- **Post-graduation ATH window**: moderate (20%) — professional traders, some bots
- **Re-pump phase**: can spike to 50-80% as MEV bots dominate declining liquidity
- Always sample fee payers at the CURRENT phase, not historical data

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

### Dormant Narrative Revival Lifecycle `[NEW — confirmed Case 003]`

Different from the standard lifecycle above. Applies to tokens dormant >3 months.

```
STAGE 0: DORMANCY (months to years)
Token on bonding curve. Near-zero trading. No catalyst.
MC: ~$0. Tx/min: <1.
On-chain tell: flat line on chart, no WITHDRAW events.

STAGE 1: CATALYST (news event / viral post)
Real-world event matches token name/narrative.
CT discovers the token exists. Bonding curve buying begins.
MC: $4K → $69K. Tx/min: 10-500 over hours.
On-chain tell: first WITHDRAW events appearing after dormancy.
OUTSIDER WINDOW: MAXIMUM — bonding curve, no MEV risk, cheap entry.

STAGE 2: GRADUATION (~$69K MC)
MigrateBondingCurveCreator + MigratePoolCoinCreator instructions.
Dev has collected max creator fees. Exits via CEX.
Token now on AMM with open price discovery.
OUTSIDER WINDOW: GOOD — post-graduation quiet period before whales arrive.

STAGE 3: PLATFORM VERIFICATION
Moonshot/Dexscreener/Birdeye verification event.
Professional traders (Cupsey-tier, 7ofDdxo7-tier) respond within minutes.
MC: $69K → $1M+ rapidly.
Fee payer concentration: 20% (moderate — professionals, not bots yet).
OUTSIDER WINDOW: CLOSING FAST above $3M MC.

STAGE 4: WHALE ATTACK
100+ SOL buyer(s) hit the AMM in rapid succession.
This causes the ATH — the whale IS the pump.
MC: $1M → ATH ($17.8M in HANTA's case).
On-chain tell: multiple 100+ SOL buys in single wallet within 10 minutes.
OUTSIDER WINDOW: CLOSED — you are buying into the whale's position.

STAGE 5: DECAY + RE-PUMP ATTEMPTS
Fee payer concentration spikes to 50-80% as MEV bots dominate.
Re-pump attempts by exchange flow (MEXC hot wallet appears as "whale").
Declining MACD, lower highs, bot-dominated order flow.
OUTSIDER WINDOW: TRAP.
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

### Additional Check for Dormant Narrative Tokens `[NEW — confirmed Case 003]`

```
[ ] DORMANT TOKEN ADDITIONAL CHECKS:
    Token age >3 months? → Apply dormant playbook, not standard playbook.

    Is token still on bonding curve?
    → YES: BEST entry zone. No MEV risk. Buy directly on pump.fun.
    → NO: Continue to next checks.

    Days since graduation?
    → <2 days: still in early AMM phase. Platform verification may not have hit yet.
    → 2-7 days: likely post-verification. Check if whale attack has happened.

    Has a 100+ SOL whale entered yet?
    → Check: any single wallet spending 100+ SOL in a single tx or cluster of txs?
    → If yes: whale attack window open or past. Assess MC vs ATH.

    Fee payer concentration at current moment?
    → >40%: re-pump phase. MEV-dominated. Do not enter.
    → <25%: still in professional accumulation phase. Window open.

    Is "re-pump whale" actually a CEX hot wallet?
    → Check Arkham label on large buyers. MEXC/Binance hot wallets = exchange flow, not a coordinated actor.
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

**Dormant token exit rule `[NEW]`:** For dormant narrative tokens, exit trigger is the WHALE ATTACK detection, not USDC payment (dev has no fee stake post-graduation). When a 100+ SOL wallet starts buying in rapid succession, begin scaling out within 10 minutes.

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

### The HANTA Simulation Baseline `[SINGLE CASE — dormant narrative reference]`

| Time (UTC) | Event | MC | Outsider Action |
|---|---|---|---|
| May 2 2026 | WHO confirms MV Hondius Hantavirus outbreak | — | Monitor narrative |
| May 6 07:08 | First creator fee collected — bonding curve activating | ~$10K | **ENTER** — bonding curve, max runway, no MEV |
| May 6 23:42 | Bonding curve graduates — `MigrateBondingCurveCreator` | ~$69K | Still hold — graduation = AMM opens |
| May 7 22:08 | Cupsey3 (@Cupseyy) starts accumulating | ~$500K | Watch fee payer conc. (still 20%) |
| May 7 22:48 | 7ofDdxo7 deploys 740 SOL in 11 minutes | $1M–$5M | **BEGIN EXIT** — whale attack detected |
| May 8 ATH | ATH $17.8M MC | $17.8M | Already out |

Entry ~$10K bonding curve → Exit $1M+ = **100x+**
No price action signal needed. Dev fee collection start = entry trigger. Whale attack = exit trigger.

---

## Helius API Technical Notes

- `getSignaturesForAddress` returns max 1,000 per call — use `before` param to paginate
- Hot tokens generate 200k+ sigs in 90 minutes — cannot paginate to creation during peak activity
- Strategy: use dev wallet CREATE event timestamp to anchor the start, paginate forward
- Token-2022 blocks `getProgramAccounts` — always use `getTokenLargestAccounts` for holders (top 20 cap)
- DAS `getAsset` gives price_info for any token — quick MC check on sibling tokens
- **Always use `getTransaction` with `jsonParsed` encoding to verify instruction type before drawing conclusions about any WITHDRAW event**
- Standard SPL tokens (not Token-2022) = old pump.fun program (`6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P`) — bonding curve PDA derivable via seeds `["bonding-curve", mint_pubkey]`
- For tokens with 1M+ signatures, pagination to creation is impractical — use Arkham or bonding curve PDA's own tx history (much smaller) to find creation tx and dev identity
- `MigrateBondingCurveCreator` instruction in bonding curve PDA tx history = graduation event timestamp

## Arkham API Technical Notes `[NEW]`

- Base URL: `https://api.arkm.com`
- Auth: `API-Key` header
- Key endpoint: `/intelligence/address/{address}/all` — returns entity label, Arkham label, chain
- Key endpoint: `/transfers?address={address}&limit=N` — returns counterparty entity connections
- Solana addresses return `arkhamEntity` (e.g. MEXC, @Cupseyy) and `arkhamLabel` (e.g. "fuckjerry on Pump.fun", "Coinbase Deposit", "scooterxbt.sol")
- Run Arkham on ALL key wallets before finalising verdict — CEX deposits and exchange hot wallets are frequently misclassified as "co-operators" or "whales" without it
- MEXC hot wallet (`ASTyfSima4LLAdDgoFGkgqoKowG1LZFDr9fAQrg7iaJZ`) appears as a large SOL holder on many Solana tokens — always check Arkham before calling it a whale

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

### CASE 003 — HANTA (Hantavirus)
**Analysis dates:** 2026-05-15 (full analysis — all 7 layers + Arkham + wallets.csv cross-reference)
**Token:** `2tXpgu2DLTsPUf9zFmuZmA4xrYxXKBTpVq9wAM7hzs9y`
**Dev:** `5P7SSkFxrjk9rEUZJ6BLeELLW5B9bWykvzFFMqSrzHzS`
**Dev alias:** "fuckjerry" on Pump.fun (confirmed via Arkham label)
**Token standard:** Standard SPL (NOT Token-2022) — old pump.fun program
**Created on:** pump.fun
**Launch date:** 2024-07-26 06:18:21 UTC (dormant for ~22 months before pump)

**ATH:** ~$17.8M MC (~May 8 2026, post-graduation AMM)
**MC at analysis:** ~$2.59M (in decay, re-pump attempt failing)
**Dev creator fees collected:** 168.15 SOL across 32 `CollectCreatorFee` transactions (all on May 6 2026, during bonding curve fill)
**Graduation:** 2026-05-06 23:42:39 UTC — `MigrateBondingCurveCreator` + `MigratePoolCoinCreator`. 20.94 SOL migrated to AMM pool.
**Operation type:** Type 7 (Dormant Narrative Revival) — organic catalyst, professional trader amplification, no KOL payment chain

**Catalyst:** Real-world Hantavirus outbreak aboard MV Hondius cruise ship (April–May 2026). WHO confirmed outbreak May 4. CT discovered the dormant HANTA token May 6. Moonshot platform verified the token on May 7, triggering professional trader inflows.

**Smoking gun:** `7ofDdxo7mMzfK25ZbrMt8ErvnvSLUNXp2Xn2jHesryxK` deploying 567 SOL across 5 transactions in 11 minutes (May 7 22:48–22:59 UTC) + 175 SOL on May 9 = ~740 SOL total. This wallet caused the ATH. Uses `PumpSpecialRouter` and `PumpSplitRouter` — purpose-built pump.fun execution infrastructure. Has Polymarket identity "alphajj". Multi-chain operator (Solana, Base, Optimism, Avalanche). No Arkham entity label — unidentified individual.

---

**KOL Payment Chain:** NONE CONFIRMED. Exhaustive dev wallet scan found zero USDC payments to identifiable KOL wallets. The relay wallet structure (SOL → USDC → GxjcMTWWDcKx) was confirmed as **liquidation infrastructure, not KOL payment infrastructure** — GxjcMTWWDcKx is a multi-chain trading aggregator (GMGN, Wintermute, LiquidMesh counterparty), not a KOL payment desk.

**Cash-out Route (confirmed via Arkham):**
```
Dev (5P7SSk) collects 168 SOL in creator fees (May 6)
  └─ FiST32C9csQPCTt8SZ9KYa9zGfcfpipKx7eYqXFBFYad (relay, 12 SOL → 1,072 USDC → GxjcMTWWDcKx)
  └─ 325f2VGzRT6U1NS94nBxqXTvh6xYsLyxcDCYPXCidWXN (relay, 20 SOL → 1,790 USDC → GxjcMTWWDcKx)
  └─ 83a9F4EvaDaQcZxnTKWP5DpyMAfhmfVkpqeaB6rGsJ3w (relay, 8.58 SOL → 766 USDC → GxjcMTWWDcKx)
  └─ Bhtaq7F9z3WFEXsVVGpnSwQsKdTnZKne3o5vgbBa7yWb (relay, 10.6 SOL → 947 USDC → GxjcMTWWDcKx)
  └─ XEfwn8pCxSZyp7iMqUCyEnm7eRPS6qqcHnKRvyXQEEE → scooterxbt.sol (14 SOL as trading capital)
  └─ AmidQKtyerkVPGTWjzcig3sjsHwNwHPzrZdN2muMrRCd (68+ SOL aggregated)
       └─ H1xmLoZSxjA9akJCbV4PrFkRyNFx59LpwBp5E3Uor21Z = Coinbase Deposit (Arkham confirmed)
          Dev cashing out to CEX. NOT a co-operator.
```

**Registry cross-reference hit:** `2fg5QD1eD7rzNNCsvnhmXFm5hqNgwTTG8p7kQ6f3rx6f` = **Cupsey3 (@Cupseyy)** in wallets.csv (kolscan category). Bought **11.4M HANTA for 485 SOL** between May 7–10. First buy at **22:08 UTC May 7** — 40 minutes before 7ofDdxo7's whale attack at 22:48. Still holds 10,009,079 HANTA (currently underwater -237 SOL unrealised). Arkham confirms identity = `@Cupseyy` on Twitter. No post about HANTA found — entry was a Moonshot verification signal response, not a self-called trade.

---

**Holder Concentration:**

| Range | Tokens | % |
|---|---|---|
| #1 | 41.5M | 4.15% |
| #2–5 | 71.6M | 7.16% |
| #6–10 | 55.6M | 5.56% |
| #11–20 | 84.5M | 8.45% |
| **Top 20** | **253.3M** | **25.3%** |

Less concentrated than PAIN (32.6%). Holders with significant SOL balances (non-burner). No coordinated zero-balance accounts in top 20.

---

**Fee Payer Concentration by Phase:**

| Phase | Top fee payer % | Unique payers | Interpretation |
|---|---|---|---|
| Post-graduation AMM open (May 7 16:24) | 20% | 12/20 | Professional traders, moderate bot activity |
| Whale attack window (May 7 22:48) | 20% | 13/20 | Same — still manageable |
| Re-pump phase (May 15 current) | 55% | 4/20 | MEV dominated — `DsCJ5siuJTPQtQa3A9N69azGZaWtUPzi9VPp2G9Jfpx9` at 1,935 tx/min |

---

**Bot Activity:**
`DsCJ5siuJTPQtQa3A9N69azGZaWtUPzi9VPp2G9Jfpx9` — 55% fee payer share during re-pump phase. 1,935 tx/min. Pure Jito MEV sandwich infrastructure. Do not enter when this wallet dominates.

---

**Sibling Tokens:**

| Token | Symbol | MC | Notes |
|---|---|---|---|
| `BhEUbSckBF9LoittiCce2XSnRkkWikeGyJRC4n8Gpump` | TERMAI | $0 | Failed Oct 2024 |
| `F9ZczdW7m1CCZibyMUi6cZj2veYcd3MEhG75TfUEpump` | GMECUBE | $0 | Failed Oct 2024 |

Both failed. Dev (fuckjerry) is not a serial operator — 3 total tokens, 1 succeeded dramatically.

---

**7 Question Answers:**

1. Manipulated? **No** — no KOL payments, no coordinated pre-loads, no engineered pump
2. Coordinated? **No** — professional traders (Cupsey3, 7ofDdxo7) acted independently in response to same Moonshot signal
3. Natural? **Yes** — organic narrative discovery via real-world news event
4. Community-driven? **Yes** — CT discovery of dormant token matching viral news narrative
5. Real-world event? **Yes** — Hantavirus outbreak aboard MV Hondius cruise ship (April–May 2026), WHO confirmed May 4
6. Celebrity? **No**
7. KOL/influencer? **Organic only** — Cupsey3 (@Cupseyy) held a position but no paid post found; 7ofDdxo7 is an unidentified professional trader

**Verdict:** Type 7 — Dormant Narrative Revival. Token sat dormant 22 months. Real-world Hantavirus outbreak triggered organic CT discovery. Dev cashed out 168 SOL to Coinbase during bonding curve phase, then graduated. Post-graduation ATH ($17.8M) was driven by Moonshot verification + professional traders (Cupsey3 + 7ofDdxo7) responding to the signal. Re-pump attempt on May 13–15 driven by retail + MEXC exchange flow (not a whale). The token is fundamentally clean — no manipulation infrastructure — but professional traders with purpose-built tools captured most of the available upside.

**HANTA vs PAIN contrast:** PAIN = closed engineering loop (dev → KOL → retail → fees → repeat). HANTA = open organic event (news → CT → Moonshot → professionals → retail). The on-chain fingerprints are opposite: PAIN has concentrated holders, bot dominance at ATH, USDC payment chain, Phantom forwards. HANTA has distributed holders, moderate bot presence at ATH, no payment chain, CEX exit.

---

**Outsider Playbook (HANTA-specific):**

| Phase | Entry MC | Upside to ATH | Risk | Verdict |
|---|---|---|---|---|
| Bonding curve (news breaks May 6) | $4K–$69K | 258x–4,450x | Very low (no MEV on bonding curve) | **BEST ENTRY** |
| Post-graduation quiet (May 7 00:00–16:00) | ~$69K | 258x | Low-medium | **GOOD ENTRY** |
| Post-verification early (May 7 16:00–22:00) | ~$200K–$500K | 35x–89x | Medium | **ACCEPTABLE** |
| Whale attack window (May 7 22:48+) | $1M–$17.8M | <18x | High | **AVOID** |
| Decay (May 8–May 13) | N/A | N/A | N/A | **EXIT ONLY** |
| Re-pump (May 13–15) | $2M–$8M | <4x | Severe (55% bot) | **TRAP** |

---

## Known Wallets Registry

### Dev Wallets

| Wallet | Alias | Confirmed Tokens | Last Active | Notes |
|---|---|---|---|---|
| `5dirQCE7Dvn3Jopo4e4ra2fcdY7imDdJ1SYTaGmnYmwt` | SerialDev_01 | PAIN, USDUC, ASSDAQ + 4 failed | 2026-05-14 | Professional operator. Dual-token strategy. KOL network confirmed. Creator fee model. Self-financing operation. |
| `5P7SSkFxrjk9rEUZJ6BLeELLW5B9bWykvzFFMqSrzHzS` | fuckjerry (pump.fun) | HANTA, TERMAI, GMECUBE | 2026-05-13 | Arkham label: "fuckjerry" on Pump.fun. CEX exit dev. Created HANTA July 2024, collected 168 SOL fees during May 6 bonding curve fill, exited via Coinbase. No KOL network. 3 total tokens, 2 failed. |

### Insider / KOL Network Wallets

| Wallet | Role | Connected To | Notes |
|---|---|---|---|
| `EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp` | Insider + KOL relay | SerialDev_01 | Received $150 USDC from dev. Pre-loaded 7jkA9z with 75k PAIN tokens 13h before ATH. Prior USDC relationship with 7jkA9z in April 2026. May be dev's own secondary wallet. |
| `7jkA9zvNB84319exePVn5Cu3kYd4oPF1rW8pobZyDhtX` | KOL #1 | SerialDev_01 + EgSptShw | Pre-loaded with 75k tokens 13h before ATH. Received $150 USDC from dev. Received 20 USDC from EgSptShw in April 2026. Recurring counterparty. |
| `4QfGpNuC5eeM9ieEp4qvFH49uWm4WqKTU4GQGvKZZyUc` | KOL payment collector | SerialDev_01 | Received $691 USDC at 04:11. Phantom-forwarded to 6bZHJzbA within 6 minutes. Acts as payment relay not final destination. |
| `6bZHJzbApPZck2gauwxmsEGkzPGZxxawoNuEFV1HtbDr` | KOL final destination | 4QfGpNuC5eeM9ieEp4qvFH49uWm4WqKTU4GQGvKZZyUc | Received $691 USDC via Phantom at 04:17. Silent after receipt. Identity unknown — likely CT/Telegram alpha account. |
| `ARu4n5mFdZogZAravu7CcizaojWnS6oqka37gdLT5SZn` | DEX router / USDC conversion | SerialDev_01 | Converted creator fee SOL to $991.77 USDC at 03:18. Funded the entire KOL budget. |
| `2fg5QD1eD7rzNNCsvnhmXFm5hqNgwTTG8p7kQ6f3rx6f` | KOL trader / known CT | HANTA Case 003 | **Cupsey3 (@Cupseyy)** — in wallets.csv (kolscan). Bought 11.4M HANTA for 485 SOL. First buy May 7 22:08 UTC, 40 min before 7ofDdxo7 whale attack. Still holding 10M HANTA (currently -237 SOL unrealised). Arkham confirmed identity = @Cupseyy. Entry was Moonshot signal response. No paid post found. |
| `7ofDdxo7mMzfK25ZbrMt8ErvnvSLUNXp2Xn2jHesryxK` | ATH whale — professional trader | HANTA Case 003 | Deployed ~740 SOL into HANTA post-graduation. First buys May 7 22:48 UTC. CAUSED the $17.8M ATH. Uses PumpSpecialRouter/PumpSplitRouter — purpose-built pump.fun tools. Multi-chain (Solana/Base/Optimism/Avalanche). Polymarket identity: "alphajj". Counterparties: Wintermute, Hyperliquid, GMX, Binance, Bitget. No Arkham entity label — unidentified individual. |

### HANTA Cashout Chain Wallets

| Wallet | Role | Connected To | Notes |
|---|---|---|---|
| `FiST32C9csQPCTt8SZ9KYa9zGfcfpipKx7eYqXFBFYad` | SOL→USDC relay | fuckjerry / Case 003 | Received 12 SOL from dev May 6. Swapped to 1,072 USDC, forwarded to GxjcMTWWDcKx within 3 seconds. Now empty. |
| `325f2VGzRT6U1NS94nBxqXTvh6xYsLyxcDCYPXCidWXN` | SOL→USDC relay | fuckjerry / Case 003 | Received 20 SOL from dev May 6. Swapped to 1,790 USDC, forwarded to GxjcMTWWDcKx. Now empty. |
| `83a9F4EvaDaQcZxnTKWP5DpyMAfhmfVkpqeaB6rGsJ3w` | SOL→USDC relay | fuckjerry / Case 003 | Received 8.58 SOL from dev May 6. Swapped to 766 USDC → GxjcMTWWDcKx. Now empty. |
| `Bhtaq7F9z3WFEXsVVGpnSwQsKdTnZKne3o5vgbBa7yWb` | SOL→USDC relay | fuckjerry / Case 003 | Received 10.6 SOL from dev May 6. Swapped to 947 USDC → GxjcMTWWDcKx. Now empty. |
| `GxjcMTWWDcKx5sxnHkNfPtuTSfV9cLgVkw3G7KvMnvBW` | Multi-chain trading aggregator | fuckjerry cashout chain | Received all USDC from HANTA relay wallets. Counterparties: GMGN, LiquidMesh, Wintermute, MEXC, Binance. NOT a KOL payment desk — this is liquidation infrastructure. |
| `XEfwn8pCxSZyp7iMqUCyEnm7eRPS6qqcHnKRvyXQEEE` | Trading capital recipient | fuckjerry / Case 003 | Arkham label: **scooterxbt.sol**. Received 14 SOL from dev May 6. Used as pump.fun sniper bot capital. Buying random tokens on Solana. Not a KOL. Relationship with dev unclear. |
| `AmidQKtyerkVPGTWjzcig3sjsHwNwHPzrZdN2muMrRCd` | SOL aggregator | fuckjerry / Case 003 | Received 68+ SOL from dev May 6. Distributed to DEX aggregator (44GW6aFire) and Coinbase deposit (H1xmLoZS). No Arkham entity label. |
| `H1xmLoZSxjA9akJCbV4PrFkRyNFx59LpwBp5E3Uor21Z` | CEX deposit | fuckjerry / Case 003 | Arkham label: **Coinbase Deposit**. Received 45 SOL from Amid chain. Dev cashing out to CEX. Previously misclassified as "co-operator" — Arkham corrected. |

### Bot / Automated Addresses

| Wallet | Type | Seen On | Notes |
|---|---|---|---|
| `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2vEMd3cxGS` | MEV Sandwich Bot (Jito) | PAIN | 80% fee payer share at ATH window. Cross-token presence to be verified on USDUC. |
| `DsCJ5siuJTPQtQa3A9N69azGZaWtUPzi9VPp2G9Jfpx9` | MEV Sandwich Bot (Jito) | HANTA re-pump | 55% fee payer share at re-pump phase. 1,935 tx/min. Do not enter any token where this wallet exceeds 40% fee payer share. |
| `roUteHjDohtkatXTb79PJ99bbxkTipgo3GJ4EJZ1YpB` | Copy-trade bundler | HANTA (early AMM) | 92 tx/min. Executes bundled buy orders for multiple wallets simultaneously. Appeared in early HANTA AMM transactions. |

### Exchange Hot Wallets (Frequently Misidentified)

| Wallet | Identity | Notes |
|---|---|---|
| `ASTyfSima4LLAdDgoFGkgqoKowG1LZFDr9fAQrg7iaJZ` | MEXC Hot Wallet | Arkham confirmed. Appeared as "31,870 SOL re-pump whale" in HANTA analysis — corrected. This is exchange flow, not a coordinated actor. Do not interpret MEXC hot wallet activity as individual whale behaviour. |

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

8. **Who is `7ofDdxo7mMzfK25ZbrMt8ErvnvSLUNXp2Xn2jHesryxK` (HANTA ATH whale)?**
Polymarket identity "alphajj". Multi-chain operator. Unidentified individual. Uses purpose-built pump.fun router infrastructure. No Arkham entity label. Social media cross-reference (Twitter/Telegram) needed to close.

9. **What is the relationship between `scooterxbt.sol` (`XEfwn8pCxSZyp7iMqUCyEnm7eRPS6qqcHnKRvyXQEEE`) and fuckjerry (HANTA dev)?**
Dev sent 14 SOL to a named CT trader. Could be a friend, a trading arrangement, or a soft amplification deal. Requires checking scooterxbt's CT posts around May 6-8 for any HANTA mention.

10. **Did Cupsey3 (@Cupseyy) post about HANTA publicly?**
No post found in web search. But they hold a 10M HANTA bag currently underwater. If they posted after entry, it may have contributed to the re-pump attempt. Check @Cupseyy Twitter for HANTA posts between May 7–15.

11. **What triggered the HANTA bonding curve to start filling on May 6 specifically?**
The WHO confirmed outbreak on May 4. CT was drawing COVID comparisons by May 6. The specific CT post or account that first surfaced the HANTA token to the trading community is unidentified. Snopes/Newsweek articles confirm `@__Mujeres` post on May 7 went viral — was there an earlier crypto-specific post on May 6?

---

## Research Agenda

### Next: Case 002 — USDUC
Same dev, same infrastructure, escaped pump.fun gravity, reached $29M, got exchange listings.
Primary question: what was architecturally different from PAIN?

### Cases 004–012 — Deliberate Selection

| Case | What to find | Framework question |
|---|---|---|
| 004 | $1M+ pump, different dev | Does pre-load pattern appear without SerialDev_01? |
| 005 | Organic pump, no KOL payments (non-dormant) | What does fresh organic look like vs HANTA's revival organic? |
| 006 | Failed operation — no ATH | What separates failure from success pre-catalyst? |
| 007 | Token where `4QfGpNuC5eeM9ieEp4qvFH49uWm4WqKTU4GQGvKZZyUc` or `6bZHJzbApPZck2gauwxmsEGkzPGZxxawoNuEFV1HtbDr` reappear | KOL network cross-token fingerprinting |
| 008 | Sub-1-hour ATH from launch | Does pre-load pattern compress to match? |
| 009 | 1-2 week slow burn to ATH | Does signal window extend proportionally? |
| 010 | Celebrity-driven pump | How does on-chain differ when catalyst is public? |
| 011 | Second pump after first ATH | What triggers re-pump? Same or different KOL network? |
| 012 | Current live token pre-ATH | First real-time framework test under live conditions |
| 013 | Any token where `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2vEMd3cxGS` or `DsCJ5siuJTPQtQa3A9N69azGZaWtUPzi9VPp2G9Jfpx9` reappear | Bot infrastructure cross-token confirmation |
| 014 | Any token where `7ofDdxo7mMzfK25ZbrMt8ErvnvSLUNXp2Xn2jHesryxK` (alphajj) appears | Confirm professional trader recurrence — builds the "whale network" registry |
