# MemecoinAnalyzer_Bible.md

_Living document. Updated after every case._
_Last updated: 2026-05-17 (Session 8+9 complete — GAPLA Case 006 + Cryptocrat Case 007 + rapidlaunch.io platform forensics + Type 2C upgraded to n=2 + DsCJ5si confirmed across 3 cases)_

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

## 0. The Foundational Principle

**Every token is guilty until proven innocent.**

Assume manipulation as the default. The analysis exists to disprove it — not to find it.

- "Organic" is never a conclusion. It is a working hypothesis that requires active disproof.
- No KOL payments found ≠ organic. Bundle check, insider pre-loads, and bot volume must all independently clear.
- Absence of one manipulation type does not imply absence of others.
- Every statement in a case file must be traceable to a specific RPC call and a specific result. If it cannot be traced, it does not appear.

---

## 0.1 Layer 0 — Bundle Check (Run Before Everything Else)

**Mandatory before Layer 1. No analysis proceeds until this clears.**

A bundled launch = dev pre-allocates a large portion of supply to a controlled wallet in the creation transaction. That wallet then dumps into retail buyers during the pump.

**How to check:**

1. Get the first 20 transactions on the token address in chronological order (`getSignaturesForAddress` reversed)
2. Fetch the creation transaction (`getTransaction` with `jsonParsed`)
3. Check `postTokenBalances` — who received tokens and how many?
4. Flag if: dev is fee payer AND receiver is a fresh wallet AND >5% of supply received in block 0

**Bundle confirmed → map the receiver wallet's full sell timeline before proceeding to any other layer.**

**If bundle is confirmed, the operation is manipulated. Classification does not change to organic because no KOL payments were found.**

_Example: BABYHANDS Case 004 — 84.5% of supply sent to fresh wallet `HB1LvHc5Z6zhUCRjXNBdf4HL3ZYWRRkqMAmuK6Sua3Qo` in creation block. Entire position dumped in 20 min 18 sec._

### Bundle false-positive guard `[HYPOTHESIS — n=1, YAE Case 005]`

Not every >5% token balance in the creation transaction is a bundle wallet.

Before confirming a bundle:

1. Check whether the >5% receiver is:
   - the pump.fun bonding curve / pair / pool address,
   - the dev wallet itself,
   - a fresh externally controlled wallet,
   - or an unresolved token account owner.
2. A bundle requires a **fresh or controlled receiver wallet** receiving a large allocation in block 0.
3. If the dev wallet itself receives >5% and immediately sells it, classify separately as **creator self-allocation dump**, not Type 2B bundle unless a separate controlled wallet is proven.

YAE example:
- Creation tx showed pump/pool address holding ~90.355177% and dev holding ~9.644957%.
- The >5% non-dev receiver was pump/pool context, not a proven fresh controlled wallet.
- Therefore bundle status was `NOT FOUND` under the existing Bible definition.
- However, dev self-allocation dump was confirmed.

### rapidlaunch.io tokens require a modified bundle check `[HYPOTHESIS — n=1, GAPLA Case 006]`

rapidlaunch.io uses the `FLASHX8DrLbgeR8FcfNV1F5krxYcYMUdBkrP1EPBtxB9` program to seed a pump AMM pool directly at creation. There is no bonding curve phase.

Key differences from standard pump.fun that affect forensics:

1. **Pool creation tx is pruned within ~8 hours.** `getTransaction` returns NULL for rapidlaunch genesis txs after a few hours. The standard bundle check cannot be run post-hoc for rapidlaunch tokens.
2. **DAS `creators` array is empty.** Use `ClaimCashback` instruction in dev wallet history to confirm creator identity.
3. **No `CollectCreatorFee`.** The equivalent is `ClaimCashback`, which yields negligible SOL (confirmed GAPLA: 0.002379 SOL on $202K ATH volume).
4. **Pre-launch insider allocations are possible before pool live.** Tokens can be transferred to insider wallets during or immediately after pool creation (before the pool is publicly visible). Check `getSignaturesForAddress` on the token mint from the earliest slot backwards.
5. **StreamFlow vesting as insider allocation mechanism.** Pre-launch wallets may immediately lock allocations in StreamFlow (`strmRqUCoQUgGUan5YhzUZa6KqdzwX5L6FpUxfmKg5m`) vesting contracts. The vested tokens appear as a self-owned token account with a single creation tx.

**rapidlaunch.io identification markers:**
- Metadata URI at `metadata.rapidlaunch.io/m/[id]`
- DAS `createdOn` field is `null`
- Pool seeded at creation (no bonding curve accumulation)
- `FLASHX8DrLbgeR8FcfNV1F5krxYcYMUdBkrP1EPBtxB9` in tx logs

---

## 1. The Core Thesis

Every pump above ~$500K MC on a pump.fun token has a cause. That cause is always one of:

- **Organic narrative** — the name/concept catches fire independently
- **KOL amplification** — paid or unpaid influencer posts
- **Coordinated insider operation** — pre-seeded wallets, scheduled pump, planned exit
- **Bot momentum** — automated trading creates artificial volume that attracts retail
- **Bundle launch** — dev controls large % of supply at creation, dumps into retail
- **Cross-community spillover** — related token pumps, community migrates

Most $1M+ pumps are combinations. The forensics job is to find which combination and prove it on-chain.

---

## 2. Operation Typology

### Type 1: Clean KOL Pump

- Dev creates token, pays 1–3 KOLs in USDC/SOL
- KOLs post, retail buys, dev collects creator fees
- Signature: USDC payments from dev wallet → unknown wallets → Phantom forward → silence
- Typical scale: $200K–$2M ATH
- Example: PAIN (Case 001)

### Type 2: Serial Operator

- Same dev wallet creates 5–10 tokens in rapid succession
- Most fail, 1–2 get KOL treatment or bundle structure
- Cross-token coordination: creator fees from Token A fund operation on Token B
- Signature: Multiple CREATE events in dev wallet, several sibling tokens at $0 MC
- Example: PAIN dev (`5dirQCE7Dvn3Jopo4e4ra2fcdY7imDdJ1SYTaGmnYmwt`) — USDUC, ASSDAQ, 4 failed
- Example: BABYHANDS dev (`7pKUwa3MZjcw3vFbW1UDAuAKEQtz8T8n82SCRXPkz4dU`) — 8 tokens, 7 failed
- Example: YAE dev (`HiSo5kykqDPs3EG14Fk9QY4B5RvkuEs8oJTiqPX3EDAn`) — 92+ sampled `CreateV2` launches
- Example: Cryptocrat dev (`9pkJqJLtefuna3GLtKZyF5DBWwEGUdxZUSGCmf5jZhA9`) — 88+ sampled CREATE events

### Type 2B: Serial Operator + Bundle `[confirmed Case 004]`

- Serial operator who uses bundle launch instead of KOL payments for distribution
- Lower operating cost than Type 1 (no USDC KOL spend), lower ceiling ($36K ATH vs millions)
- Supply control at launch replaces paid distribution
- Example: BABYHANDS Case 004

### Type 2C Candidate: Serial Operator + Creator Self-Allocation Dump `[HYPOTHESIS — n=2, YAE Case 005 + Cryptocrat Case 007]`

A serial operator launches a token, receives a meaningful creator-side token allocation during the creation flow, and sells the full allocation almost immediately.

This differs from Type 2B:

| Type | Supply control mechanism | Dump wallet | YAE fit? | Cryptocrat fit? |
|---|---|---|---|---|
| Type 2B Serial Operator + Bundle | Fresh/controlled wallet receives large block-0 allocation | Bundle receiver wallet | No | No |
| Type 2C Candidate | Dev wallet receives creator-side allocation and dumps directly | Dev wallet | Yes | Yes |

YAE evidence:
- Dev: `HiSo5kykqDPs3EG14Fk9QY4B5RvkuEs8oJTiqPX3EDAn`
- 92 sampled `CreateV2` launches in dev history.
- Dev received `96,449,438.144093 YAE`, equal to `9.6449567%` of supply, in creation tx.
- Dev sold full allocation 2 seconds later.
- Sale tx native delta: `+4.443323273 SOL`.

Cryptocrat evidence:
- Dev: `9pkJqJLtefuna3GLtKZyF5DBWwEGUdxZUSGCmf5jZhA9`
- 88+ sampled CREATE events in dev history.
- Creation tx `4sB4zRGwhaGmxeitzZYradaQFQQsAgHkHVUQH9moj3Q8pWTk34U5MubXmn2WCe4PdiKhg77NY9ayuUKwn2JtKCAS`
- Dev received `96,449,438.144093 Cryptocrat tokens`, equal to `9.645%` of supply.
- Dev sold full allocation across five `SellV2` txs within 9 seconds for `+8.742681479 SOL`.

Classification rule:
- If dev self-allocation >5% occurs and is sold within seconds/minutes, classify as **Serial Operator + Creator Self-Allocation Dump candidate**.
- Do not classify as bundle unless a separate fresh/controlled receiver wallet is proven.
- Do not classify as organic even if KOL payments are not found.
- **Do not graduate to confirmed Type 2C until 3+ cases confirm.** Currently n=2.

### Type 2 on rapidlaunch.io `[HYPOTHESIS — n=1, GAPLA Case 006]`

Serial operators may use rapidlaunch.io in addition to or instead of pump.fun. The insider allocation mechanism differs: instead of dev receiving tokens in the creation tx and dumping (Type 2C), an associated wallet receives tokens pre-pool-live and locks them in StreamFlow vesting. This appears team/investor-distribution-like rather than immediate extraction. No dev self-allocation dump was found on GAPLA — the Type 2C pattern appears to be pump.fun specific so far.

### Type 3: Insider Cluster

- Dev pre-seeds 5–20 wallets with tokens before any public activity
- Signature: Token transfers FROM dev/insider TO fresh wallets 12–72 hours before ATH
- Example: EgSptShw → 7jkA9z pre-load (PAIN)
- Example: GAPLA — `BAr5csYtpWoNpwhUjixX7ZPHXkUciFZzjBp9uNxZXJPh` received 2.77% supply via StreamFlow vesting 1h 54m before pool live

### Type 4: Bot-Manufactured Volume

- Single automated address (fee payer) accounts for 50%+ of tx volume
- Signature: 1–3 fee payers in 80%+ of sampled transactions
- Example: FireuLYd (PAIN, 80% at ATH)

### Type 5: Dual-Token Operation

- Dev runs two tokens simultaneously, cross-promotes
- Example: PAIN + USDUC

### Type 6: Organic

- No USDC payments, no coordinated pre-loads, no bundle
- Top holders have real tx history, diverse fee payers
- Rare above $1M MC
- Example: USDUC (Case 002), HANTA bonding curve phase (Case 003)

### Type 7: Dormant Narrative Revival `[confirmed Case 003]`

- Token launched months/years ago, sat dormant
- Real-world news event reactivates it
- Dev cashes out via CEX during bonding curve phase
- Post-graduation pump driven by professional traders responding to platform verification
- Example: HANTA (Case 003) — launched July 2024, dormant 22 months, revived by Hantavirus outbreak May 2026

---

## 3. How pump.fun Creator Fees Work

**Mandatory understanding before analysing any WITHDRAW event.**

pump.fun pays token creators a fee on every trade through the bonding curve. Claimed via `CollectCreatorFee` instruction. On-chain:

- `source: PUMP_FUN`
- `type: WITHDRAW`
- `Program log: Instruction: CollectCreatorFee`
- SOL flowing INTO dev wallet FROM bonding curve

**This is normal. It is NOT manipulation, extraction, or LP removal.**

### The mandatory verification rule

**Never characterise a PUMP_FUN WITHDRAW without first checking `getTransaction` with `jsonParsed` encoding and reading `meta.logMessages` for the instruction name.**

`CollectCreatorFee` = normal fee income.
Anything else = investigate before concluding.

### Graduation mechanics `[confirmed Case 003]`

When bonding curve fills (~$69K), emits `MigrateBondingCurveCreator` + `MigratePoolCoinCreator`. After graduation:
- Dev earns NO more creator fees
- Token trades on AMM at open market price
- The real pump often happens POST-graduation

### rapidlaunch.io variant `[HYPOTHESIS — n=1, GAPLA Case 006]`

rapidlaunch.io does NOT use `CollectCreatorFee`. The dev is the registered creator via a `ClaimCashback` instruction sourced from a dedicated per-token cashback escrow account. Yield is negligible compared to pump.fun (confirmed 0.002379 SOL on $202K ATH volume on GAPLA). The cashback escrow address links the dev to the specific token and is the primary on-chain proof of creator identity when the pool creation tx is pruned.

---

## 4. Dev Wallet Behavioral Patterns

### The Fee Collector
Creates token, earns SOL via `CollectCreatorFee`. Higher volume = more fees. Incentive to drive volume.

### The Dumper
Creates token, holds % of supply. Sells via SWAP transactions. Evidence: large SWAP events from dev wallet involving the token mint.

### The Bundle Operator `[confirmed Case 004]`
Pre-allocates large % of supply (>50%) to a controlled wallet in the creation transaction. That wallet dumps into retail during the pump. Dev simultaneously collects creator fees. Full exit in hours via fresh cashout wallet.
- Signature: dev as fee payer on creation tx, large % to fresh wallet in block 0, fresh wallet sells to zero quickly, dev balance goes to 0.
- Lower overhead than KOL pump — no USDC payments required.
- Cashout wallet typically has <5 lifetime txs, created same day.

### The Creator Self-Allocation Dumper `[HYPOTHESIS — n=2, YAE Case 005 + Cryptocrat Case 007]`

Creates token, receives a meaningful allocation during the creation transaction, and sells the allocation immediately through token sale / swap path.

Signature:
- Dev is creation transaction fee payer.
- Creation logs include `CreateV2` and buy/mint path.
- `postTokenBalances` show dev receives >5% supply.
- A near-immediate `SellV2` or swap removes dev token balance to zero.
- Dev receives SOL/native delta from sale.
- No separate bundle wallet is required.

YAE example:
- Received: `96,449,438.144093 YAE` at `2026-05-16 07:18:37 UTC`.
- Sold: full amount at `2026-05-16 07:18:39 UTC`.
- Time to zero: 2 seconds.
- Final dev YAE balance: 0 / token account closed.

Cryptocrat example:
- Received: `96,449,438.144093 Cryptocrat` at creation.
- Sold: full amount across five `SellV2` txs within 9 seconds.
- Sale delta: `+8.742681479 SOL`.
- Final dev Cryptocrat balance: 0.

Doctrinal implication:
- This is manipulation/extraction evidence, but it is **not the same mechanism** as a fresh-wallet bundle.

### The rapidlaunch.io Creator `[HYPOTHESIS — n=1, GAPLA Case 006]`

Creates token via rapidlaunch.io, which seeds a pump AMM pool directly. Dev revenue comes via `ClaimCashback` rather than `CollectCreatorFee`. Creator identity provable only via the cashback escrow relationship. Pool creation tx is pruned from RPC within hours.

Pre-launch insider allocations to a coordinated third party may occur at/around creation via StreamFlow vesting. This is distinct from a bundle (the tokens are locked, not immediately sold) but is structurally manipulative.

Evidence signature:
- `ClaimCashback` instruction in dev wallet tx history with source = per-token cashback escrow
- DAS metadata URI matches `metadata.rapidlaunch.io/m/[id]`
- Dev has no direct token account for the launched token

### The Hybrid `[OBSERVED PAIN — verify future cases]`
Collects creator fees AND pays KOLs. Creator fees fund KOL payments. Self-financing loop.

### The CEX Exit Dev `[confirmed Case 003]`
Collects creator fees during bonding curve. Routes SOL → relay wallets → USDC → CEX deposit. No ongoing stake post-graduation.

### Exit Signal (universal)
- Dev SOL balance drops to <0.01 SOL
- No new PUMP_FUN interactions for 24+ hours
- All USDC converted and forwarded out

---

## 5. The KOL Payment Fingerprint

```
Dev wallet
  └─ USDC transfer to Wallet A  (payment)
       └─ Wallet A forwards to Wallet B via Phantom  (within 5-30 min)
            └─ Wallet B goes silent  (actual KOL's cold receiving wallet)
```

**The Phantom forward is the tell.** Legitimate OTC does not forward within 6 minutes.

### Payment Amount Tiers `[HYPOTHESIS — n=1, PAIN only]`

```
$50-200:   micro-KOL, <10k followers
$200-500:  mid-tier, 10k-50k followers
$500-2000: tier-1 ecosystem KOL, 50k+ followers
$2000+:    coordinated multi-platform push
```

### Distinguishing KOL payments from liquidation `[confirmed Case 003]`

Not every SOL → USDC flow is a KOL payment:
- **KOL payment**: USDC ends at wallet that goes silent, has CT/Telegram identity, Phantom-forwards within minutes
- **Liquidation**: USDC ends at high-frequency aggregator (GMGN, LiquidMesh, Wintermute) or CEX deposit

Always trace 2+ hops AND verify terminal wallet activity before concluding KOL payment.

---

## 6. Tx Volume as Price Proxy

| Tx/minute   | Market state           |
|-------------|------------------------|
| <10         | Dead / no interest     |
| 10-100      | Slow accumulation      |
| 100-500     | Growing momentum       |
| 500-2,000   | Active pump phase      |
| 2,000-5,000 | ATH window / peak FOMO |
| 5,000+      | Bot-dominated          |

---

## 7. Bot Amplification

### Bot Types

**Type A: MEV Sandwich Bots** — Pure extractors. Same fee payer on majority of txs. Guaranteed tax on every retail entry/exit.

**Type B: Momentum Bots** — Directional. Buy into detected signals, sell into FOMO.

**Type C: Volume Wash Bots** — Trade with themselves to inflate volume metrics.

**Type D: Copy Trade Bots** — Mirror tracked profitable wallets within same block.

**Type E: Indiscriminate Launch Snipers `[confirmed Case 004]`** — Created fresh (days old), 1000+ txs in 14 hrs, non-stop PUMP_FUN SWAPs. Snipe every new launch without selectivity. Not connected to any specific dev.
- Example: `5brv79eFZ2rGprXNvqgVJBkBptkkw8GJX1XydJyZLyAr` (BABYHANDS, 138 SOL, created 2026-05-15)
- Example: `AVMpb2bXMdNec6ykkEvf7krff7nXMZuKvadPELPNA9M6` (BABYHANDS, created 2026-05-15)

### Fee Payer Concentration Risk Table

| Top fee payer % | Meaning               | Action                          |
|-----------------|----------------------|---------------------------------|
| <20%            | Diverse organic flow  | Minimal sandwich risk           |
| 20-50%          | Moderate bot presence | Factor into sizing              |
| 50-80%          | Heavy bot dominance   | Every trade taxed significantly |
| >80%            | Pure extraction zone  | Do not enter                    |

### Fee Payer Concentration Shifts By Phase `[confirmed Case 003]`

- Bonding curve: typically <20% — organic retail discovery
- Post-graduation ATH window: 20% — professional traders, some bots
- Re-pump phase: can spike to 50-80% as MEV bots dominate

### Known bots can coexist with live demand `[HYPOTHESIS — n=1, YAE Case 005]`

Bot presence does not automatically kill momentum. In YAE, `DsCJ5siuJTPQtQa3A9N69azGZaWtUPzi9VPp2G9Jfpx9` appeared as a recurring top fee payer (16.25% in one sample, 33/220 in another) while the token continued to expand from $1M to $2M+ MC. Bot activity is a risk factor to price into position sizing, not an automatic exit trigger when live demand remains strong.

---

## 8. The Golden Hour Framework

### Operation Lifecycle Stages

```
STAGE 1: LAUNCH (Day 0)
Token created. Dev begins fee collection immediately.
Tx/min: <100.

STAGE 2: GRIND (Day 1-2) [HYPOTHESIS]
Slow organic volume builds.
Tx/min: 100-500.

STAGE 3: ARMING (12-24 hours before ATH) [HYPOTHESIS]
Insider pre-loads begin (Type 3) OR bundle wallet positioned (Type 2B).
Tx/min: 300-600.

STAGE 4: GOLDEN HOUR (0-20 min before ATH) [HYPOTHESIS]
USDC payments fire (Type 1) OR bundle wallet begins selling (Type 2B).
Tx/min: 500 → 2,000+.

STAGE 5: ATH WINDOW
Dev collects largest fee batch. Bot concentration 60-80%+.
Tx/min: 2,000-5,000+.

STAGE 6: DECAY
Dev SOL declining to zero. Insider wallets dumping.
```

### Dormant Narrative Revival Lifecycle `[confirmed Case 003]`

```
STAGE 0: DORMANCY (months to years)
Near-zero trading. No catalyst.

STAGE 1: CATALYST (news event)
CT discovers token. Bonding curve buying begins.
OUTSIDER WINDOW: MAXIMUM

STAGE 2: GRADUATION (~$69K MC)
Dev exits via CEX. Token on AMM.

STAGE 3: PLATFORM VERIFICATION
Moonshot/Dexscreener verification. Professional traders respond.
OUTSIDER WINDOW: CLOSING FAST above $3M

STAGE 4: WHALE ATTACK
100+ SOL buyer(s) hit AMM in rapid succession. This IS the ATH.
OUTSIDER WINDOW: CLOSED

STAGE 5: DECAY + RE-PUMP ATTEMPTS
Fee payer concentration spikes to 50-80%. MEV-dominated.
OUTSIDER WINDOW: TRAP
```

### The Pre-Entry Checklist

All five must be TRUE simultaneously:

```
[ ] 1. DEV WALLET SOL BALANCE > 0.1 SOL
[ ] 2. USDC PAYMENTS HAVE NOT FIRED YET
[ ] 3. FEE PAYER CONCENTRATION < 50%
[ ] 4. TX/MIN IN RANGE 100-2,000
[ ] 5. INSIDER PRE-LOAD DETECTED OR WITHDRAWAL RATE ACCELERATING
```

### Additional Check — Bundle Detection `[confirmed Case 004]`

```
[ ] BUNDLE CHECK (run before all other checks):
    Fetch first 20 txs on token address chronologically.
    On creation tx: is dev the fee payer?
    On creation tx postTokenBalances: did any fresh wallet receive >5% supply?
    → YES to both: BUNDLED LAUNCH. Do not enter. Dev controls supply exit.
    → NO: check if dev wallet itself received >5% (self-allocation dump candidate).
    → NEITHER: continue to standard checklist.
    → rapidlaunch.io token (metadata.rapidlaunch.io URI): creation tx is pruned.
      Check for StreamFlow vesting + pre-launch insider allocations instead.
```

### Forensic Cleanliness vs Live Tradeability `[HYPOTHESIS — n=1, YAE Case 005]`

Forensic cleanliness and tradeability are different questions.

| Question | Purpose | YAE answer |
|---|---|---|
| Is the token clean? | Truth classification | No |
| Is the dev/operator safe? | Operator risk | No |
| Is the move organic? | Demand-source classification | No / not proven |
| Is there still a live momentum window? | Tradeability classification | Yes, during YAE run |
| Is it safe to hold full size late? | Position risk classification | No |

> Dirty does not automatically mean untradeable. Dirty means untrusted. Tradeability depends on whether live demand, liquidity growth, holder breadth, and order flow are still absorbing the dirt.

Suggested tradeability labels:

| Label | Meaning |
|---|---|
| `CLEAN TRADE` | Forensic checks and live flow both supportive |
| `DIRTY BUT TRADEABLE` | Manipulation/dirt confirmed, but live demand still absorbs supply |
| `PROFIT-PROTECTION ONLY` | Position already profitable; no new entry/add logic; monitor exit triggers |
| `TRAP` | Late entry risk too high; bot/holder/dev risk dominates |
| `DEAD / EXIT` | Demand gone, sellers dominate, or bot extraction overwhelms flow |

YAE observed path: `DIRTY BUT TRADEABLE` → `PROFIT-PROTECTION ONLY` → `BOT-ASSISTED ATH EXPANSION WITH HOLDER ROTATION`

Mark as `[HYPOTHESIS — n=1]` until repeated across more live case studies.

### Dirty-but-tradeable decays to TRAP quickly after ATH failure `[HYPOTHESIS — n=2, YAE + Cryptocrat]`

Dirty-but-tradeable is time-limited. If the ATH fails to continue and liquidity shrinks while bot/fee-payer concentration remains high, downgrade to `TRAP / PROFIT-PROTECTION ONLY` even if some known holders remain.

Cryptocrat observed path: dirty token attracted real demand + KOL-holder wallets entered → ATH expanded to ~$787K → ATH failed → liquidity shrank from ~$65.9K to ~$51.7K → top-holder rotation → automation-heavy flow → `TRAP`.

Candidate triggers to downgrade from `DIRTY BUT TRADEABLE` to `TRAP`:
- Failed reclaim after ATH wick
- Liquidity falls while bot concentration holds high
- Major holders reduce together
- Known bot share spikes while price stalls
- Prior breakout zone lost and not reclaimed

### The Exit Rules

**Primary trigger (KOL pump):** First USDC payment detected from dev wallet. Start 20-minute timer. Full exit by expiry.

**Bundle pump exit:** No USDC to watch for. Monitor bundle wallet balance. When bundle wallet approaches zero, dev is almost done extracting. Do not enter after bundle wallet has begun selling.

**Hard stop:** Dev SOL balance drops below 0.05 SOL at any point. Exit immediately.

**Dormant token exit:** When a 100+ SOL wallet starts buying in rapid succession, begin scaling out within 10 minutes.

### Dirty Momentum Profit-Protection Rule `[HYPOTHESIS — n=1, YAE Case 005]`

When a token is forensically dirty but live flow remains strong, the correct state label after a large unrealized gain is not "safe hold." It is `PROFIT-PROTECTION ONLY`.

YAE signals that moved the trade into profit-protection mode:

| Signal | YAE observation | Meaning |
|---|---|---|
| Prior forensic dirt | Dev self-allocation dump + serial operator | Cannot trust token structure |
| Strong live demand | Buyers kept absorbing sells | Pump remained alive |
| Liquidity expansion | Liquidity rose from ~$95K to ~$127K during run | Not dead illiquid wick |
| Bot participation | `DsCJ5siuJTPQtQa3A9N69azGZaWtUPzi9VPp2G9Jfpx9` repeatedly appeared as top fee payer | Flow not clean |
| Holder rotation | Some top holders reduced while new buyers absorbed | Momentum alive but fragile |
| Near/above ATH | $1M–$2M+ expansion zone | Late-stage volatility risk |

Candidate warning signs to reduce/exit dirty momentum:

- Failed reclaim after ATH wick
- Price stops making new highs while volume remains high
- 5m buy/sell flow flips sell-heavy for repeated refreshes
- Top holders reduce together
- Known bot share spikes while price stalls
- Liquidity drops while sells rise
- Demand score cools while market cap remains elevated
- Prior breakout zone is lost and not quickly reclaimed

At +40%, +65%, +70%, and +127% in YAE, the analysis shifted away from entry and toward protecting profit / runner logic. Treat as case-specific model, not universal law yet.

---

## Helius API Technical Notes

- `getSignaturesForAddress` max 1,000 per call — use `before` param to paginate
- Token-2022 blocks `getProgramAccounts` — use `getTokenLargestAccounts` for holders (top 20 cap)
- DAS `getAsset` gives price_info for quick MC check
- Always use `getTransaction` with `jsonParsed` to verify instruction type before concluding on any WITHDRAW
- `postTokenBalances` in transaction data shows who received tokens — critical for bundle detection
- **rapidlaunch.io pool creation txs are pruned from `getTransaction` within ~8 hours of creation.** For forensics, must be captured live. Post-hoc analysis limited to sig confirmation via `getSignaturesForAddress`.
- `ClaimCashback` instruction (rapidlaunch.io) = equivalent of `CollectCreatorFee` for creator identification. Source is a per-token cashback escrow, not PUMP_FUN directly.
- StreamFlow metadata accounts (`strmRqUCoQUgGUan5YhzUZa6KqdzwX5L6FpUxfmKg5m` owner) carry vesting schedule data. Can be decoded via `getAccountInfo` base64 + manual struct parsing.
- Self-owned token accounts (where owner = the account pubkey itself) indicate StreamFlow escrow accounts or other program-controlled vaults. Do not classify as insider wallets without checking the tx that created them.

## Arkham API Technical Notes

- Base URL: `https://api.arkm.com`
- Auth: `API-Key` header
- Key endpoint: `/intelligence/address/{address}/all` — entity label, Arkham label
- Key endpoint: `/transfers?address={address}&limit=N`
- Run Arkham on ALL key wallets before finalising verdict
- MEXC hot wallet (`ASTyfSima4LLAdDgoFGkgqoKowG1LZFDr9fAQrg7iaJZ`) appears as large SOL holder — always check Arkham before calling it a whale

---

# LAYER 2 — CASE INTELLIGENCE

---

## Case Registry

---

### CASE 001 — PAIN (Max Pain)

**Token:** `2N9BXbRtyZM4YHzj9wvteW1Nuq6B5DzDwh57dDmZpump`
**Dev:** `5dirQCE7Dvn3Jopo4e4ra2fcdY7imDdJ1SYTaGmnYmwt`
**Launch:** 2026-05-11 14:07 UTC | **ATH:** ~$2.61M | **Operation:** Type 1 + 2 + 3 + 4

**Smoking gun:** $691 USDC payment at 04:11:51 → Phantom forward at 04:17:45 → ATH at 04:30. 19-minute window.

**KOL payments:** $991 USDC total (funded from 479 SOL creator fees)
- $150 → `7jkA9zvNB84319exePVn5Cu3kYd4oPF1rW8pobZyDhtX`
- $150 → `EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp`
- $691 → `4QfGpNuC5eeM9ieEp4qvFH49uWm4WqKTU4GQGvKZZyUc` → Phantom to `6bZHJzbApPZck2gauwxmsEGkzPGZxxawoNuEFV1HtbDr`

**Insider pre-load:** `EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp` → `7jkA9zvNB84319exePVn5Cu3kYd4oPF1rW8pobZyDhtX` 75,000 tokens at 13h 45m before ATH.

**Bot:** `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2vEMd3cxGS` — 80% fee payer at ATH.

---

### CASE 002 — USDUC (unstable coin)

**Status:** PENDING
**Token:** `CB9dDufT3ZuQXqqSfa1c5kY935TEreyBw9XJXxHKpump`
**Dev:** `5dirQCE7Dvn3Jopo4e4ra2fcdY7imDdJ1SYTaGmnYmwt` (same as PAIN)
**Key question:** PAIN died at $196K, USDUC reached $29M + Binance.US listing. What was architecturally different?

---

### CASE 003 — HANTA (Hantavirus)

**Token:** `2tXpgu2DLTsPUf9zFmuZmA4xrYxXKBTpVq9wAM7hzs9y`
**Dev:** `5P7SSkFxrjk9rEUZJ6BLeELLW5B9bWykvzFFMqSrzHzS` (alias: "fuckjerry" on pump.fun)
**Launch:** 2024-07-26 | **ATH:** ~$17.8M | **Operation:** Type 7 (Dormant Narrative Revival)

**Catalyst:** Real-world Hantavirus outbreak aboard MV Hondius cruise ship (WHO confirmed May 4 2026). CT discovered dormant HANTA token May 6.

**Smoking gun:** `7ofDdxo7mMzfK25ZbrMt8ErvnvSLUNXp2Xn2jHesryxK` deploying ~740 SOL in rapid succession caused the ATH. Uses PumpSpecialRouter/PumpSplitRouter. Polymarket: "alphajj".

**Registry hit:** `2fg5QD1eD7rzNNCsvnhmXFm5hqNgwTTG8p7kQ6f3rx6f` = Cupsey3 (@Cupseyy). Bought 11.4M HANTA for 485 SOL. Entered 40 min before whale attack.

**KOL payments:** NONE. Dev cashout was liquidation via CEX (Coinbase deposit `H1xmLoZSxjA9akJCbV4PrFkRyNFx59LpwBp5E3Uor21Z`, Arkham confirmed).

**Verdict:** Organic Type 7. No manipulation infrastructure. Professional traders (Cupsey3 + `7ofDdxo7mMzfK25ZbrMt8ErvnvSLUNXp2Xn2jHesryxK`) captured most upside via Moonshot verification signal.

---

### CASE 004 — BABYHANDS

**Analysis date:** 2026-05-16
**Token:** `FnEJrLqJCBiAWPmLRekRSwsYpiL36Js2NEj7Dfwfpump`
**Dev:** `7pKUwa3MZjcw3vFbW1UDAuAKEQtz8T8n82SCRXPkz4dU`
**Token standard:** Token-2022
**Created on:** pump.fun
**Launch:** 2026-05-16 03:00:38 UTC

**ATH:** ~$36,300 MC
**Dev creator fees:** 0.734 SOL total (two `CollectCreatorFee` withdrawals — both verified)
**USDC profit-taken:** $473.59 USDC (swap into SOL)
**Cashout:** 11.60 SOL → `EVNJsBsTM373wdJFqGj9iiYvRiv8yViCjE5zBr4afW9W` at 05:12:58 UTC
**Dev exit:** 0.0000 SOL — fully exited within 2.5 hours of creation
**Operation type:** Type 2 + Bundle (Type 2B)

**Smoking gun:**
Tx `F7ZEn12eQctG5oXk537uS27wPTR3f8EaYwapuY7pboN4uArVtt2PHq1vMdyzy5LFGrTX352rkXWRzuLyym3Qb5W` at 2026-05-16 03:00:38 UTC — dev `7pKUwa3MZjcw3vFbW1UDAuAKEQtz8T8n82SCRXPkz4dU` is fee payer, `845,076,438 tokens (84.5% of total supply)` sent to fresh wallet `HB1LvHc5Z6zhUCRjXNBdf4HL3ZYWRRkqMAmuK6Sua3Qo` in the creation block. `HB1LvHc5Z6zhUCRjXNBdf4HL3ZYWRRkqMAmuK6Sua3Qo` sold the entire position to zero in 20 minutes 18 seconds (03:00:38–03:20:56 UTC), cycling 3,457M tokens total via automated sandwich selling.

**Bundle wallet sell timeline:**
- Received: 845,076,438 tokens at 03:00:38 UTC
- Sold: continuously 03:00:38 → 03:20:56 UTC (20 min 18 sec)
- Method: automated buy/sell cycling (3,457M tokens cycled = 3x total supply)
- Final balance: 0 tokens at 03:20:56 UTC

**KOL payments:** NONE. Zero USDC outflows to unknown recipients. Zero hits against 939-wallet KOL registry.

**First-block snipers** (independent, not connected to dev):
- `5brv79eFZ2rGprXNvqgVJBkBptkkw8GJX1XydJyZLyAr` — 100M tokens, 138 SOL, indiscriminate sniper bot (created 2026-05-15, snipes every pump.fun launch)
- `AVMpb2bXMdNec6ykkEvf7krff7nXMZuKvadPELPNA9M6` — 48M tokens, bot pattern (created 2026-05-15)
- `FTXgeRxnr2ZU1dMKjLfEF6i7j3MXrAVkj317oRFtSFjK` — 32.8M tokens, sniper (created 2026-05-08)
- `Ag9JF87A6mvaPyNJybPGbPaL4km64CwCGHHt1f8PHHZh` — 55M tokens, not a bot (created 2026-04-29, 57 txs)
- `7gEnRjDud56BVmBQMWLTticJoouzrpnyaAnN5A3EEqiy` — 28.8M tokens
- `6UtEUmnXxGNUtRGy2bVGhmvKTuqeVYJkP4ACoHFeps1a` — 24.2M tokens (created 2026-05-14)
- `9N4taaPKAvJ7UHpu6GM3sjJUFbsZTgpCdMHnraVUvoSU` — 15.7M tokens, aged wallet (created 2025-09-18)

**Co-operator:** `J87UST7jVkLWxf5irqTJTSTYPoevXy3jKrFNw8Pr665f`
- Own token creator (SpaceXTesla/SEXT `8nJXsbasRYRaQusn6ujazBzA5HuCVxXnohFXb1pdpump`, unindexed token `97VeY7aBstbS5pVMFzpCNw48R3E8KYn5McCGP1DzTCVN`)
- Mar 24: sent 4.066 SOL TO dev. May 14: received 4.172 SOL back. May 15: sent 6.357 SOL to relay `6JM1poX1S25FW75qSFekJDK9g4kj3Z3C2UNFedzFcKCL`
- Never held BABYHANDS tokens. Capital counterparty, not operational participant.

**Dev sibling tokens (all confirmed via CREATE txs):**

| Date | Token | Symbol | MC |
|------|-------|--------|-----|
| 2025-10-20 | `CdKWL6dEYwSpBtiNiyqh318og3fi24ayNAJ3E6bepump` | airdrop | $0 |
| 2026-03-24 | `41MeTHbgux4WBuNcTujbRiePxegs3roTLSf2yNVipump` | IBR | $0 |
| 2026-03-26 | `HKBXLqo4J2ahxCccvP5Kx6aQM6Wt78iV9iwcZhknpump` | LONGCOIN | $0 |
| 2026-04-01 | `87XtTLCzg3h3kwgQLR6uBJsJji9r8ocDjPj4Y93Fpump` | IPO | $0 |
| 2026-04-17 | `D15VNw9WPsBzBgGqcMqydj1UDBNGLH5QzDYwPeHapump` | Astrojak | $0 |
| 2026-04-21 | `7gBgWeb92czboTXR2xfjf7JBSdQFWtkK3JJS48pPpump` | TREECOIN | $0 |
| 2026-05-14 | `ArykdF21UkKSCyvQods5HmkTDenCcn9eijAc7KwUpump` | UNDERDOG | $0 |
| **2026-05-16** | `FnEJrLqJCBiAWPmLRekRSwsYpiL36Js2NEj7Dfwfpump` | **BABYHANDS** | **$3,988 (post-ATH)** |

**7 Question Answers:**

1. Manipulated? **Yes** — bundle pre-allocation + controlled dump into retail
2. Coordinated? **Partial** — `J87UST7jVkLWxf5irqTJTSTYPoevXy3jKrFNw8Pr665f` capital flows confirmed, SOL relay infrastructure, fresh cashout wallet
3. Natural? **No**
4. Community-driven? **Unknown** — demand-side catalyst not verifiable (social data unavailable)
5. Real-world event? **Unknown** — "baby hands" = Trump political reference; plausible but unconfirmed
6. Celebrity? **No evidence**
7. KOL-driven? **No** — zero USDC payments, zero registry hits

---

### CASE 005 — YAE

**Analysis date:** 2026-05-16
**Token:** `GbuHq6UTNUp8CaFsoPgUhxpPBnkVXGepDHgFhN2kpump`
**Dev:** `HiSo5kykqDPs3EG14Fk9QY4B5RvkuEs8oJTiqPX3EDAn`
**Launch:** 2026-05-16 07:18:37 UTC
**Operation:** Type 2 Serial Operator + creator self-allocation dump `[HYPOTHESIS — n=2 candidate Type 2C, confirmed alongside Cryptocrat Case 007]`
**Bundle:** Not found under current Bible definition
**Organic:** No / not proven
**Manipulated:** Yes
**KOL-driven:** Unknown — no confirmed KOL payment
**Bot-assisted:** Partial / confirmed live bot participation via known bot recurrence

**Smoking gun:**
Creation tx `3PuPCoEazqZrtZ3EiGHVZoNGAAfkHiTmp42gCsTaxQQLzA7MDrUCSaZKVgM2kRyGP5TDcjmYJheVJAjUerVCQAQE` showed dev receiving `96,449,438.144093 YAE`, equal to `9.6449567%` of supply. Dev sold the full allocation 2 seconds later in tx `2pP19B...`, with native delta `+4.443323273 SOL`.

**Creation transaction details:**
- Fee payer: `HiSo5kykqDPs3EG14Fk9QY4B5RvkuEs8oJTiqPX3EDAn`
- Logs/instructions: `CreateV2`, `InitializeMint2`, `MintTo`, `BuyExactQuoteInV2`
- Supply: `999,998,661.828355 YAE`
- Pump/pool context address: `Ffx6MPyFcuJ5sYKmWixtNWvfNnWhByTNU3HnmSh2DiSD` held `903,550,561.855907 YAE` = `90.355177%`
- Dev held `96,449,438.144093 YAE` = `9.644957%`

**Dev sell timeline:**
- Received allocation: 2026-05-16 07:18:37 UTC
- Sold full allocation: 2026-05-16 07:18:39 UTC
- Time to zero: 2 seconds
- Final dev token balance: 0 / token account closed

**Serial operator evidence:**
- At least 92 sampled `CreateV2` transactions found in dev history.
- True wallet genesis unknown — fetched history did not reach definitive genesis.
- Funding source unknown.

**KOL / payment chain:**
- No confirmed KOL payment.
- Dev sent `3.399973486 SOL` to `21wG4F3ZR8gwGC47CkpD6ySBUgH9AABtYMBWFiYdTTgv` via `SingleSolPayment` at 2026-05-16 07:23:32 UTC.
- `21wG4F3ZR8gwGC47CkpD6ySBUgH9AABtYMBWFiYdTTgv` forwarded `6.796268645 SOL` to `5tzFkiKscXHK5ZXCGbXZxdw7gTjjD1mBwuoFbhUvuAi9` at 07:30:00 UTC.
- Recipient was not silent; 1,000 txs fetched, newest activity at 08:46:54 UTC.
- Classification: unclassified payment/service/relay — not KOL-confirmed.

**Holder concentration at analysis:**
- Top 10: `24.11%`
- Top 20: `35.91%`
- Later snapshot (during live run): top 10 improved to `19.55%`, top 20 to `30.47%` — broadening while price rose.

**Known smart-money / registry hits:**
- `Bf4zji6S979QySiGNjPJ2VMZ5i2SRVtAzfx8QUBScJm6` — `runitbackghost`, Arkham `@runitbackghost`. Held ~20.47M YAE during analysis; position unchanged during live run.
- `A8Z1ejQGk45EJibBPJviWnM3UvwKSuYun53nSCkWKM52` — `lspfi`, Arkham `@lspfinance`. Top holder at analysis, later reduced.

**Bot / fee-payer concentration:**
- Pump.fun early sample: top fee payer `DXxGpA...` at 12%.
- Pumpswap latest sample: `BX7h4...` at 28%, `Ak6ba...` at 13%, `BaXmv...` at 11%, `AxGt...` at 9%.
- Mint recent sample: `BX7h4...` at 37.5%, `AxGt...` at 37.5%, `BaXmv...` at 18.3%.
- `DsCJ5siuJTPQtQa3A9N69azGZaWtUPzi9VPp2G9Jfpx9`: 26/160 txs = 16.25% in one live sample; 33/220 pool tx sample in later order-flow check. **Known Bible bot — recurrence confirmed (also HANTA).**
- Top fee payers `BX7h4...`, `AxGt...`, `BaXmv...`, `DXxGp...`, `Ak6ba...` not yet bot-confirmed — remain open candidates.

**Live flow observations:**
- Token continued into $1M–$2M+ MC zone despite dirty launch.
- At ~$2.16M current MC / ~$2.22M ATH live snapshot: liquidity ~$127K, global fees 114.1 SOL, Spider demand 80 HOT, pool health 28x HOT, top-holder behavior MIXED_BULLISH.
- Holder concentration improved while price rose — real demand absorbing seller rotation.
- Interpretation: real narrative demand + broadening holder base + bot-assisted momentum + smart-money rotation.
- Tradeability label during run: `DIRTY BUT TRADEABLE` → `PROFIT-PROTECTION ONLY`.

**7 Question Answers:**

1. Manipulated? **Yes** — dev sold full 9.6449567% self-allocation 2 seconds after creation.
2. Coordinated? **Unknown** — SOL payment chain exists, but coordinated actor relationship not proven.
3. Natural / organic? **No** — dev dump and serial-operator evidence prevent organic classification.
4. Community-driven? **Unknown** — social/community data unavailable beyond chart/link context.
5. Real-world event? **Unknown** — Dexscreener linked a Livedoor article, but catalyst impact not on-chain confirmed.
6. Celebrity-driven? **Unknown** — no verified celebrity driver found.
7. KOL / influencer-driven? **Unknown** — no confirmed KOL payment; known smart-money holders exist but paid promotion not proven.

---

### CASE 006 — GAPLA (Federated States of Gapla)

**Analysis date:** 2026-05-17
**Token:** `3y3X6Bk6zpZpaFJ1KYJ7b33HbFM7ip5TqQPYLWJhpump`
**Pool:** `2FxX1awKLyAaiWZSa4pL9bYcLzmEtie6JpuoCu8HTzRo`
**Dev:** `HiSo5kykqDPs3EG14Fk9QY4B5RvkuEs8oJTiqPX3EDAn` (SerialDev_02 — same as YAE Case 005)
**Launch platform:** rapidlaunch.io (NOT pump.fun)
**Pool genesis:** slot 420085766 = 2026-05-16 08:44:48 UTC (tx pruned)
**ATH:** ~$202K MC (chart-sourced)
**ATH window:** ~14:00–15:00 UTC 2026-05-16
**Current MC at analysis:** ~$44.6K (decay)
**Operation:** Type 2 Serial Operator + Type 3 Insider Cluster (pre-launch StreamFlow vesting)
**Creator fees collected:** 0.002379 SOL via ClaimCashback (negligible)

**Smoking gun:**
`3HGJbUSi9MRY8TyBb7ZpNFy7fCKa4qP5Tx3XGebQwZSqd5Nxfg9J3dRaeYMNNYjtb9DTzasyfJ3J5Lx9qwYGErdX` at 2026-05-16 06:50:42 UTC — `BAr5csYtpWoNpwhUjixX7ZPHXkUciFZzjBp9uNxZXJPh` already held 27,722,546 GAPLA (2.77% of supply) 1 hour 54 minutes before the pool went live. This wallet StreamFlow-vested 27,584,623 GAPLA to `DkzgUjvAd5G5ru7wvcacPKs67rqnzUuWGEQBfLu3CWkX`. Stream metadata: `5YAbng5uWXnnkgSYJfsJUagYqaFfcUv8Lv68arxTMf7H`. StreamFlow fee: 137,923 GAPLA to `5SEpbdjFK5FxwTvfsGMXVQTD2v4M2c5tyRTxhdsPkgDw`.

**Creator identity proof:**
Dev called `ClaimCashback` at 13:58:30 UTC, collecting +0.002379 SOL from escrow `FLDf1A1NxF5tnQNvRsSrpPKHEzLEQAB2ZmaoVfxJLqox`. Tx: `3EYV93HTYVXGeLz1qUPZavQU9k6ZrXfhTeKekVWHyrgTMxXgYv9WLzmQaW1hhY3udmRYsaQBZo1VakiP5WURdyqA`.

**Dev activity post-launch:**
- 11:06 UTC: Failed FLASHX relaunch (`Yto7FjEMhD6GPYnGW6PMm3gQgaE1h13vVyriaxAuTHjaENoesq8NWjSt1gansvwyumfiihPqAUoKkEufDz8uhdP`) — needed 40,414 SOL, failed
- 11:55 UTC: Failed FLASHX relaunch (`45AUZhip6ibouGGmcEccjFTQarDqxeqqkaDUfEk61unRhgc8UsgwCSZ9VmwUMQS8iaZgLc91HgHXYrNPtkaqSdhe`) — needed 186 SOL, failed
- Dev has NO GAPLA token account — never held GAPLA directly
- Dev SOL at analysis: 5.555 SOL (not fully exited)

**Social narrative:** Real accounts — `@digitalmanul` (Twitter: `https://x.com/digitalmanul/status/2055512653628645585`) and `@gaplaofficial` (TikTok). "Federated States of Gapla" is a pre-existing internet meme/fictional nation concept. KOL vs organic: UNKNOWN.

**KOL payments:** NONE confirmed. Dev has 0 USDC. No USDC outflow chain detected.

**Key platform differences (rapidlaunch.io vs pump.fun):**
- No bonding curve — pool funded at creation and live immediately
- Creation tx pruned from RPC within hours
- DAS creators empty, update authority renounced
- `ClaimCashback` ≠ `CollectCreatorFee` — yields negligible SOL
- FLASHX deploy wallet `GH5sng7gR67zNj1HqEXYT5sa4y82x4o43Ycsrqmb1MNy` is service-level intermediary

**7 Question Answers:**

1. Manipulated? **Yes** — pre-launch 2.77% insider allocation via StreamFlow; serial operator dev
2. Coordinated? **Partial** — StreamFlow allocation confirmed; BAr5c-to-dev link not confirmed on-chain
3. Natural / organic? **No**
4. Community-driven? **Unknown** — real social presence exists
5. Real-world event? **Unknown** — fictional nation meme; trigger event not identified
6. Celebrity-driven? **Unknown** — `@digitalmanul` identity unknown
7. KOL-driven? **Unknown** — no USDC payment chain; social accounts pre-exist launch

---

### CASE 007 — CRYPTOCRAT

**Analysis date:** 2026-05-17
**Token:** `HacxkKvctUzgryZBi7g27RULqm98tH4se39dPeQepump`
**Dev:** `9pkJqJLtefuna3GLtKZyF5DBWwEGUdxZUSGCmf5jZhA9`
**Pair / chart:** `BhBDxSh74dUgVbDZ78866o9UiPfjcX6n7zBSGPC97ttz`
**Operation:** Type 2C Candidate — Serial Operator + Creator Self-Allocation Dump + Bot-assisted distribution + KOL-holder participation (payment unconfirmed)

**Smoking gun:**
Creation tx `4sB4zRGwhaGmxeitzZYradaQFQQsAgHkHVUQH9moj3Q8pWTk34U5MubXmn2WCe4PdiKhg77NY9ayuUKwn2JtKCAS` — dev `9pkJqJLtefuna3GLtKZyF5DBWwEGUdxZUSGCmf5jZhA9` received `96,449,438.144093` tokens = `9.645%` of supply. Dev sold full allocation across five `SellV2` txs **within 9 seconds** for `+8.742681479 SOL`. Final dev token balance: 0.

**Serial operator evidence:** 88+ sampled CREATE events in dev history.

**Bot activity:** `DsCJ5siuJTPQtQa3A9N69azGZaWtUPzi9VPp2G9Jfpx9` dominant fee payer:
- `305 / 1000` txs = 30.5% in one dump-window sample
- `132 / 1000` txs = 13.2% in a later status sample
- **This is now 3rd confirmed case for this bot (HANTA, YAE, Cryptocrat)**

**Chart path:**
- Initial active re-pump around $595K
- ATH expanded to approximately $787K
- Failed ATH continuation
- Later MC near $330K
- Liquidity fell from ~$65.9K to ~$51.7K

**KOL-holder wallets (participation confirmed, payment chain unconfirmed):**
- `9yYya3F5EJoLnBNKW6z4bZvyQytMXzDcpU5D6yYr4jqL` — Loopierr / Arkham `"xix111" on Pump.fun`. First buy 2026-05-16T15:00:08Z; held 14,838,760.098066 tokens.
- `LeenseyyUU3ccdBPCFCrrZ8oKU2B3T2uToGGZ7eVABY` — Leens. First buy 2026-05-16T15:01:38Z; later dropped out of top 10.
- `CAPn1yH4oSywsxGU456jfgTrSSUidf9jgeAnHceNUJdw` — cap. First buy 2026-05-16T15:08:16Z; later dropped out of top 10.
- `215nhcAHjQQGgwpQSJQ7zR26etbjjtVdW74NLzwEgQjP` — OGandtV2 / Arkham `binladen.sol`. Bought 7,830,119.246591 tokens for 45.46207408 SOL; held 15,097,769.637342 tokens.
- `6S8GezkxYUfZy9JPtYnanbcZTMB87Wjt1qx3c6ELajKC` — Nyhrox / Arkham `"nyrra." on Pump.fun`. Deployed ~105.918 SOL; later reduced and dropped out of top 10.
- `GdEZQDsaNdoy2vQev9qbJjmTfPh2Br8UvGh3bhpyUENv` — Arkham `@f4vel`. First buy 2026-05-16T15:15:56Z.

**Tradeability path:** `DIRTY BUT TRADEABLE` (early demand real) → `TRAP / PROFIT-PROTECTION ONLY` (ATH failed, liquidity shrank, top-holder rotation, automation-heavy flow)

**7 Question Answers:**

1. Manipulated? **Yes** — dev sold full 9.645% self-allocation within 9 seconds of creation
2. Coordinated? **Unknown** — KOL-holder participation without confirmed payment chain
3. Natural / organic? **No**
4. Community-driven? **Unknown**
5. Real-world event? **Unknown**
6. Celebrity-driven? **Unknown**
7. KOL-driven? **Unknown** — named CT wallets entered but no USDC payment proven

---

## Known Wallets Registry

### Dev Wallets

| Wallet | Alias | Confirmed Tokens | Notes |
|--------|-------|-----------------|-------|
| `5dirQCE7Dvn3Jopo4e4ra2fcdY7imDdJ1SYTaGmnYmwt` | SerialDev_01 | PAIN, USDUC, ASSDAQ + 4 failed | Professional KOL pump operator. Self-financing creator fee model. |
| `5P7SSkFxrjk9rEUZJ6BLeELLW5B9bWykvzFFMqSrzHzS` | fuckjerry (pump.fun) | HANTA, TERMAI, GMECUBE | CEX exit dev. Arkham confirmed. 3 tokens, 2 failed. |
| `7pKUwa3MZjcw3vFbW1UDAuAKEQtz8T8n82SCRXPkz4dU` | BundleDev_01 | BABYHANDS + 7 failed | Serial operator. Bundle launch model. 8 tokens (Oct 2025–May 2026), 7 at $0. Fully exited BABYHANDS in 2.5 hrs. |
| `HiSo5kykqDPs3EG14Fk9QY4B5RvkuEs8oJTiqPX3EDAn` | SerialDev_02 / SelfAllocationDev_01 | YAE (pump.fun) + GAPLA (rapidlaunch.io) + 92 sampled `CreateV2` launches | YAE: received 9.6449567% supply, sold 2 seconds later. GAPLA: rapidlaunch.io launch; no direct self-allocation; pre-launch StreamFlow allocation to associate. Uses BOTH pump.fun and rapidlaunch.io. Funding source/genesis unknown. |
| `9pkJqJLtefuna3GLtKZyF5DBWwEGUdxZUSGCmf5jZhA9` | SerialDev_03 / SelfAllocationDev_02 | Cryptocrat + 88 sampled CREATE events | Received 96,449,438.144093 Cryptocrat tokens (9.645% supply) in CreateV2 and sold full allocation within 9 seconds for +8.742681479 SOL. High-risk serial operator. |

### Insider / KOL Network Wallets

| Wallet | Role | Connected To | Notes |
|--------|------|--------------|-------|
| `EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp` | Insider + KOL relay | SerialDev_01 | Received $150 USDC. Pre-loaded `7jkA9zvNB84319exePVn5Cu3kYd4oPF1rW8pobZyDhtX` with 75k PAIN tokens 13h 45m before ATH. |
| `7jkA9zvNB84319exePVn5Cu3kYd4oPF1rW8pobZyDhtX` | KOL #1 | SerialDev_01 | Pre-loaded 75k tokens 13h before ATH. $150 USDC from dev. |
| `4QfGpNuC5eeM9ieEp4qvFH49uWm4WqKTU4GQGvKZZyUc` | KOL payment collector | SerialDev_01 | Received $691 USDC. Phantom-forwarded within 6 min. |
| `6bZHJzbApPZck2gauwxmsEGkzPGZxxawoNuEFV1HtbDr` | KOL final destination | `4QfGpNuC5eeM9ieEp4qvFH49uWm4WqKTU4GQGvKZZyUc` | Received $691 via Phantom. Silent after. Identity unknown. |
| `ARu4n5mFdZogZAravu7CcizaojWnS6oqka37gdLT5SZn` | DEX router / USDC conversion | SerialDev_01 | Converted 479 SOL creator fees → $991.77 USDC. |
| `2fg5QD1eD7rzNNCsvnhmXFm5hqNgwTTG8p7kQ6f3rx6f` | KOL trader / known CT | HANTA Case 003 | Cupsey3 (@Cupseyy). Bought 11.4M HANTA for 485 SOL. |
| `7ofDdxo7mMzfK25ZbrMt8ErvnvSLUNXp2Xn2jHesryxK` | ATH whale — professional trader | HANTA Case 003 | ~740 SOL into HANTA. PumpSpecialRouter. Polymarket: "alphajj". |
| `J87UST7jVkLWxf5irqTJTSTYPoevXy3jKrFNw8Pr665f` | Co-operator dev | BABYHANDS Case 004 | Own token creator. Bilateral SOL flows with BundleDev_01. Never held BABYHANDS. |

### Pre-Launch Insider / Allocation Wallets

| Wallet | Role | Connected To | Notes |
|--------|------|--------------|-------|
| `BAr5csYtpWoNpwhUjixX7ZPHXkUciFZzjBp9uNxZXJPh` | Pre-launch insider / team allocation recipient | GAPLA Case 006 / SerialDev_02 | Received 27,722,546 GAPLA (2.77% supply) at/during pool creation (pruned tx). StreamFlow-vested 27,584,623 GAPLA to `DkzgUjvAd5G5ru7wvcacPKs67rqnzUuWGEQBfLu3CWkX` at 06:50:42 UTC — 1h 54m before pool live. 69 SOL balance. Active since 2026-05-14. No shared txs with dev. Arkham: no entity. Identity: UNKNOWN. |
| `DkzgUjvAd5G5ru7wvcacPKs67rqnzUuWGEQBfLu3CWkX` | StreamFlow vesting escrow | GAPLA Case 006 | Holds 27,584,623 GAPLA (2.76% supply) in locked StreamFlow vesting. Self-owned token account pattern. Stream metadata: `5YAbng5uWXnnkgSYJfsJUagYqaFfcUv8Lv68arxTMf7H`. Vesting terms: UNKNOWN. |

### KOL-Holder Wallets (Participation Confirmed, Payment Unproven)

| Wallet | Identity | Connected To | Notes |
|--------|----------|--------------|-------|
| `9yYya3F5EJoLnBNKW6z4bZvyQytMXzDcpU5D6yYr4jqL` | Loopierr / Arkham `"xix111" on Pump.fun` | Cryptocrat Case 007 | First buy 2026-05-16T15:00:08Z; held 14,838,760.098066 tokens. |
| `LeenseyyUU3ccdBPCFCrrZ8oKU2B3T2uToGGZ7eVABY` | Leens | Cryptocrat Case 007 | First buy 2026-05-16T15:01:38Z; later dropped out of top 10. |
| `CAPn1yH4oSywsxGU456jfgTrSSUidf9jgeAnHceNUJdw` | cap | Cryptocrat Case 007 | First buy 2026-05-16T15:08:16Z; later dropped out of top 10. |
| `215nhcAHjQQGgwpQSJQ7zR26etbjjtVdW74NLzwEgQjP` | OGandtV2 / Arkham `binladen.sol` | Cryptocrat Case 007 | Bought 7,830,119.246591 tokens for 45.46207408 SOL; held 15,097,769.637342 tokens. |
| `6S8GezkxYUfZy9JPtYnanbcZTMB87Wjt1qx3c6ELajKC` | Nyhrox / Arkham `"nyrra." on Pump.fun` | Cryptocrat Case 007 | Deployed ~105.918 SOL; later reduced and dropped out of top 10. |
| `GdEZQDsaNdoy2vQev9qbJjmTfPh2Br8UvGh3bhpyUENv` | Arkham `@f4vel` | Cryptocrat Case 007 | First buy 2026-05-16T15:15:56Z. |

### Unclassified Payment / Relay Wallets

| Wallet | Role | Connected To | Notes |
|--------|------|--------------|-------|
| `21wG4F3ZR8gwGC47CkpD6ySBUgH9AABtYMBWFiYdTTgv` | Unclassified payment / service / relay candidate | YAE / `HiSo5kykqDPs3EG14Fk9QY4B5RvkuEs8oJTiqPX3EDAn` | Received 3.399973486 SOL via `SingleSolPayment` shortly after YAE launch. Forwarded 6.796268645 SOL to `5tzFkiKscXHK5ZXCGbXZxdw7gTjjD1mBwuoFbhUvuAi9`. Not silent. No registry hit. Do **not** label as KOL without further evidence. |
| `5tzFkiKscXHK5ZXCGbXZxdw7gTjjD1mBwuoFbhUvuAi9` | Downstream recipient | YAE payment trace | Received 6.796268645 SOL from `21wG4F3ZR8gwGC47CkpD6ySBUgH9AABtYMBWFiYdTTgv`. Classification unknown. |

### Bundle Infrastructure Wallets

| Wallet | Role | Connected To | Notes |
|--------|------|--------------|-------|
| `HB1LvHc5Z6zhUCRjXNBdf4HL3ZYWRRkqMAmuK6Sua3Qo` | Bundle receiver | BundleDev_01 / BABYHANDS | Received 845M tokens (84.5% supply) at creation. Dumped to zero in 20 min 18 sec. Fresh wallet, automated. |
| `EVNJsBsTM373wdJFqGj9iiYvRiv8yViCjE5zBr4afW9W` | Dev cashout | BundleDev_01 / BABYHANDS | Received 11.60 SOL at 05:12:58 UTC. 3 txs total. Created same day. |
| `6JM1poX1S25FW75qSFekJDK9g4kj3Z3C2UNFedzFcKCL` | SOL relay/staging | BundleDev_01 / BABYHANDS | Repeatedly moves SOL between dev and `J87UST7jVkLWxf5irqTJTSTYPoevXy3jKrFNw8Pr665f`. 0 SOL. Fully drained. |
| `D7DDkxgSd8ySgEv971AA2vgFAk4btgw9anChz7jvkRZM` | Burner funder | BundleDev_01 / BABYHANDS | Funded relay 2.94 SOL. 2 txs. Created solely for this. |

### rapidlaunch.io Infrastructure Wallets

| Wallet | Role | Notes |
|--------|------|-------|
| `GH5sng7gR67zNj1HqEXYT5sa4y82x4o43Ycsrqmb1MNy` | rapidlaunch.io deploy wallet | 0 SOL; 16 total txs; used as fee payer for FLASHX failed launch attempts connected to SerialDev_02. Ephemeral — funded by `Fupr9EcwV3cAfy2VM7uL7C3vQ8Y38LxiWz7dW2qUUqpE` at 05:59 UTC. Pattern: funded → attempt → fails → drained. |
| `FLDf1A1NxF5tnQNvRsSrpPKHEzLEQAB2ZmaoVfxJLqox` | rapidlaunch cashback escrow (GAPLA-specific) | Source of dev's ClaimCashback +0.002379 SOL. Per-token escrow — one per rapidlaunch creation. |
| `FLASHX8DrLbgeR8FcfNV1F5krxYcYMUdBkrP1EPBtxB9` | rapidlaunch.io program | Core program for token creation on rapidlaunch.io. |

### HANTA Cashout Chain Wallets

| Wallet | Role | Notes |
|--------|------|-------|
| `FiST32C9csQPCTt8SZ9KYa9zGfcfpipKx7eYqXFBFYad` | SOL→USDC relay | 12 SOL → 1,072 USDC → `GxjcMTWWDcKx5sxnHkNfPtuTSfV9cLgVkw3G7KvMnvBW`. Liquidation. |
| `325f2VGzRT6U1NS94nBxqXTvh6xYsLyxcDCYPXCidWXN` | SOL→USDC relay | 20 SOL → 1,790 USDC → `GxjcMTWWDcKx5sxnHkNfPtuTSfV9cLgVkw3G7KvMnvBW`. Liquidation. |
| `83a9F4EvaDaQcZxnTKWP5DpyMAfhmfVkpqeaB6rGsJ3w` | SOL→USDC relay | 8.58 SOL → 766 USDC → `GxjcMTWWDcKx5sxnHkNfPtuTSfV9cLgVkw3G7KvMnvBW`. |
| `Bhtaq7F9z3WFEXsVVGpnSwQsKdTnZKne3o5vgbBa7yWb` | SOL→USDC relay | 10.6 SOL → 947 USDC → `GxjcMTWWDcKx5sxnHkNfPtuTSfV9cLgVkw3G7KvMnvBW`. |
| `GxjcMTWWDcKx5sxnHkNfPtuTSfV9cLgVkw3G7KvMnvBW` | Multi-chain trading aggregator | Received all HANTA USDC. GMGN, LiquidMesh, Wintermute counterparty. NOT a KOL desk. |
| `XEfwn8pCxSZyp7iMqUCyEnm7eRPS6qqcHnKRvyXQEEE` | Trading capital | Arkham: scooterxbt.sol. Received 14 SOL from fuckjerry May 6. |
| `AmidQKtyerkVPGTWjzcig3sjsHwNwHPzrZdN2muMrRCd` | SOL aggregator | 68+ SOL from fuckjerry → Coinbase deposit. |
| `H1xmLoZSxjA9akJCbV4PrFkRyNFx59LpwBp5E3Uor21Z` | CEX deposit | Arkham: Coinbase Deposit. Dev cashing out. |

### Smart-Money / Observed Holders

| Wallet | Role | Connected To | Notes |
|--------|------|--------------|-------|
| `Bf4zji6S979QySiGNjPJ2VMZ5i2SRVtAzfx8QUBScJm6` | Smart-money holder | YAE Case 005 | Registry: `runitbackghost`; Arkham: `@runitbackghost`. Held ~20.47M YAE; position unchanged during live run. |
| `A8Z1ejQGk45EJibBPJviWnM3UvwKSuYun53nSCkWKM52` | Smart-money holder | YAE Case 005 | Registry: `lspfi`; Arkham: `@lspfinance`. Top holder at analysis, later reduced. |
| `4iD83DYUbfcLx1A6V1D5uHbpZdQaZnYYXHywq6NNG4rs` | Unknown whale holder | GAPLA Case 006 | 131.32 SOL; 20,089,093 GAPLA (2.01% supply). Active since 2026-05-13. Still active post-ATH. Arkham: no entity. Entry timing unknown. |

### Case-Only Watchlist (Cryptocrat — Pending Recurrence Proof)

| Wallet | Role in Cryptocrat | Evidence |
|--------|-------------------|---------|
| `7FixNQS6bZ9QR7KAyRXhosvUkXn26z7dTsSqy1TzQdi7` | Major holder / overhang | Held 45,043,053.479012 tokens (4.504% supply); actively bought and sold in late samples |
| `FcfJmRr8wYRmE1rdrLeqvnzzJvXambDZxsjbjKyv6mCM` | Large late seller | Sold 9,712,405.027849 tokens in tx `9w1gNKhJx3NknvgtVShZL1cjVjHpLzNfK8wEszNVv14VMdey3YQrXBzDmsvvW8dYczMuRDpShNEF3TfRFQWhYN2` |
| `CoYrhm486gwjfGAN45bJehimAidrUUHNUJpbUxBatNGL` | Large late seller / routed flow | Sold 1,643,190.207053 tokens in tx `2RLSPidYxH9hjN4LLYv8vK53NUe76uWWwo3eCbRt5Y5jwASeSEdL5Zq5rR5m5Qxspf5CRAz7MMjaEK5ZpTWkFD2b` |
| `5XRAYV5x2DFjx1UtFKeKwWU6pAfznr7zN2J1oUgopEis` | Late seller | Sold 382,063.323891 tokens in tx `3Wv77BKxJXFtzLvS13E9rBvus5jMKHFXXLzF3ULviXCmHec3puAncaQXSgYzzkVqa1eSPeXVV4heHmrZGxqj3rqU` |

### Bot / Automated Addresses

| Wallet | Type | Seen On | Notes |
|--------|------|---------|-------|
| `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2vEMd3cxGS` | MEV Sandwich Bot (Jito) | PAIN | 80% fee payer at ATH. |
| `DsCJ5siuJTPQtQa3A9N69azGZaWtUPzi9VPp2G9Jfpx9` | MEV / automated flow bot **`[CONFIRMED ACROSS CASES: HANTA, YAE, Cryptocrat]`** | HANTA re-pump; YAE live ATH expansion; Cryptocrat dump window | HANTA: 55% fee payer during re-pump. YAE: 16.25% and 13.2% in live samples. Cryptocrat: 30.5% in dump-window sample, 13.2% in later sample. Recurrence across 3 cases confirmed. Do not treat as automatic exit trigger when live demand remains strong — but high concentration is significant risk signal. |
| `roUteHjDohtkatXTb79PJ99bbxkTipgo3GJ4EJZ1YpB` | Copy-trade bundler | HANTA early AMM | 92 tx/min. Bundled buy orders. |
| `5brv79eFZ2rGprXNvqgVJBkBptkkw8GJX1XydJyZLyAr` | Indiscriminate Launch Sniper | BABYHANDS | Created 2026-05-15. 1000+ txs/14 hrs. Non-stop PUMP_FUN SWAPs. Snipes every launch. |
| `AVMpb2bXMdNec6ykkEvf7krff7nXMZuKvadPELPNA9M6` | Indiscriminate Launch Sniper | BABYHANDS | Created 2026-05-15. Same bot pattern. |
| `FTXgeRxnr2ZU1dMKjLfEF6i7j3MXrAVkj317oRFtSFjK` | Launch Sniper | BABYHANDS | Created 2026-05-08. 1000+ txs in 8 days. |
| `zeroDbresTUpHKau1vP4jXrkqtN8Dmh8yuEqyn6GMYh` | OnChainArbMultiPath (NA247a7) | GAPLA | Failed arb at slot 420092070 (09:26 UTC). Part of NA247a7YE9S3p9CdKmMyETx8TTwbSdVbVYHHxpnHTUV arb program. |

### Bot Infrastructure Candidates — Not Yet Confirmed

The following showed fee-payer concentration in samples but have no recurrence proof across cases. Do not label as bots until confirmed.

| Wallet | Seen On | Evidence |
|--------|---------|---------|
| `BX7h4...` | YAE | 28–37.5% fee payer in pumpswap/mint samples. Full address needed. |
| `AxGt...` | YAE | 9–37.5% fee payer. Full address needed. |
| `BaXmv...` | YAE | 11–18.3% fee payer. Full address needed. |
| `DXxGp...` | YAE | 12% fee payer in early pump.fun sample. Full address needed. |
| `Ak6ba...` | YAE | 13% fee payer. Full address needed. |
| `A7FMMgue4aZmPLLoutVtbC7gJcyqkHybUieiaDg9aaVE` | Cryptocrat | Top fee payer: 85/1000 in one sample, 63/1000 in later sample |
| `3DStwn3emHr9F6XAY5GrKafMpoPxEDaf43R92tvK7A7q` | Cryptocrat | Top fee payer: 72/1000 |
| `4r33xEKAD2cNMrC9NyJy8nb4XmruUKebZ6LZZm65PVUZ` | Cryptocrat | Top fee payer: 65/1000, later 67/1000 |
| `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2eqhTP9RHV` | Cryptocrat | Top fee payer: 64/1000 — NOTE: different address from confirmed `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2vEMd3cxGS` (PAIN). Verify before merging. |
| `CoPyPPWdm8SirumaAEe8S68nSJpHYhguN2L5i2YGxLB` | Cryptocrat; GAPLA (arb infrastructure) | Cryptocrat: 61/1000, later 75/1000. GAPLA: seen in arb tx at slot 420111276. |
| `66666y3RmL6gNyttNKXNhF5awxAWbBdGSfpKuaTLrjRH` | Cryptocrat; GAPLA | Cryptocrat: 56/1000, later 39/1000. GAPLA: arb infrastructure slot 420111276. |
| `8TPWakvWw4xQbk7uAYdNjZiDKKHgv9GE5GebzsbtUaHr` | Cryptocrat | Top fee payer: 78/1000 |
| `zeronaXJsbvPZFmzytV4RPXWBHQGMFcRCvoYaDNPiRL` | Cryptocrat | Top fee payer: 52/1000 |
| `AXmnRBrNtYYyyo82cLBBhnWJ7o1iqNLZbuEVpDB3V666` | Cryptocrat | Top fee payer: 42/1000; also appeared in routed churn |
| `327677XqTEwYxo8kaQxAJUWvUvzpVoSjiMXMc8u4wQS6` | Cryptocrat | Top fee payer: 33/1000 |

_Note: Full addresses must be recovered before these entries can be promoted to the main bot registry. YAE partial addresses (`BX7h4...` etc.) still need full recovery._

### Exchange Hot Wallets (Frequently Misidentified)

| Wallet | Identity | Notes |
|--------|----------|-------|
| `ASTyfSima4LLAdDgoFGkgqoKowG1LZFDr9fAQrg7iaJZ` | MEXC Hot Wallet (Arkham confirmed) | Appeared as "whale" in HANTA — corrected. Exchange flow only. |

---

## Open Questions

1. **Who is `6bZHJzbApPZck2gauwxmsEGkzPGZxxawoNuEFV1HtbDr`?** Final $691 USDC destination (PAIN). Unknown CT/Telegram identity.

2. **Is `EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp` the dev's own secondary wallet?**

3. **Who created the Meteora DLMM pool on PAIN?**

4. **Total creator fee income across PAIN + USDUC for `5dirQCE7Dvn3Jopo4e4ra2fcdY7imDdJ1SYTaGmnYmwt`?** Likely 2,000+ SOL.

5. **Does the 19-minute payment → ATH rule hold across operators?** [HYPOTHESIS — n=1]

6. **April 2026 history of `EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp` and `7jkA9zvNB84319exePVn5Cu3kYd4oPF1rW8pobZyDhtX`?** Prior token collaboration?

7. **Does `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2vEMd3cxGS` appear on USDUC?** Cross-token bot confirmation pending.

8. **Who is `7ofDdxo7mMzfK25ZbrMt8ErvnvSLUNXp2Xn2jHesryxK`?** Polymarket: "alphajj". No Arkham entity.

9. **What is the relationship between scooterxbt.sol and fuckjerry?**

10. **Did Cupsey3 (@Cupseyy) post about HANTA publicly?**

11. **What CT post first surfaced HANTA on May 6?**

12. **What triggered demand-side retail volume on BABYHANDS beyond first-block snipers?** (LunarCrush subscription needed)

13. **Are sniper bots `5brv79eFZ2rGprXNvqgVJBkBptkkw8GJX1XydJyZLyAr`, `AVMpb2bXMdNec6ykkEvf7krff7nXMZuKvadPELPNA9M6`, `FTXgeRxnr2ZU1dMKjLfEF6i7j3MXrAVkj317oRFtSFjK` connected to each other or same entity?**

14. **Does `7pKUwa3MZjcw3vFbW1UDAuAKEQtz8T8n82SCRXPkz4dU` dev reuse bundle wallet pattern on prior sibling tokens?**

15. **Does the creator self-allocation then instant sell pattern recur across `HiSo5kykqDPs3EG14Fk9QY4B5RvkuEs8oJTiqPX3EDAn` launches?** (PARTIAL ANSWER — Case 006: NO self-allocation dump on GAPLA/rapidlaunch.io. Pattern appears pump.fun specific for this dev. Remain open for pump.fun launches only.)

16. **Was `21wG4F3ZR8gwGC47CkpD6ySBUgH9AABtYMBWFiYdTTgv` a service/payment processor, relay wallet, or promotion recipient?** (Case 005 open)

17. **Did `A8Z1ejQGk45EJibBPJviWnM3UvwKSuYun53nSCkWKM52`, `Bf4zji6S979QySiGNjPJ2VMZ5i2SRVtAzfx8QUBScJm6` enter before or after the main YAE pump?** (Case 005 open)

18. **Which wallet caused the largest upward move on YAE?** (Case 005 open)

19. **Are `BX7h4...`, `AxGt...`, `BaXmv...`, `DXxGp...` automation infrastructure across other live pumps?** Full addresses needed first. (Case 005 open)

20. **What social/narrative trigger drove YAE demand?** (Case 005 open)

21. **Why did demand keep absorbing sell pressure on YAE despite confirmed dev dirt?** (Case 005 open)

22. **Who is `BAr5csYtpWoNpwhUjixX7ZPHXkUciFZzjBp9uNxZXJPh`?** Pre-launch GAPLA insider — team member, investor, or service? On-chain link to dev not found (0 shared txs). (Case 006 open)

23. **What are the StreamFlow vesting terms for `DkzgUjvAd5G5ru7wvcacPKs67rqnzUuWGEQBfLu3CWkX`?** Cliff, duration, unlock rate for 27,584,623 GAPLA. Stream metadata: `5YAbng5uWXnnkgSYJfsJUagYqaFfcUv8Lv68arxTMf7H`. (Case 006 open)

24. **Who is `@digitalmanul`?** Twitter linked in GAPLA metadata — known CT figure? Did they post about GAPLA before or after launch? (Case 006 open)

25. **Who is `4iD83DYUbfcLx1A6V1D5uHbpZdQaZnYYXHywq6NNG4rs`?** Whale with 131 SOL and 2.01% GAPLA supply. Entry timing and ATH driver status unknown. (Case 006 open)

26. **What were the failed FLASHX relaunches at 11:06 and 11:55 UTC targeting?** Dev failed needing 40,414 SOL and 186 SOL — different targets. Second token? Parameter change? (Case 006 open)

27. **How many of SerialDev_02's 92+ sampled launches are rapidlaunch.io vs pump.fun?** (Case 006 open)

28. **What drove the 3.5-hour gap between GAPLA pool launch (08:44 UTC) and main pump (~13:00 UTC)?** (Case 006 open)

29. **Is `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2eqhTP9RHV` (Cryptocrat) the same entity as `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2vEMd3cxGS` (PAIN)?** Addresses differ at last characters — must verify. (Case 007 open)

30. **Does `9pkJqJLtefuna3GLtKZyF5DBWwEGUdxZUSGCmf5jZhA9` (SerialDev_03) repeat the 9–10% self-allocation dump across its other launches?** (Case 007 open)

31. **Is `7FixNQS6bZ9QR7KAyRXhosvUkXn26z7dTsSqy1TzQdi7` a recurring post-ATH overhang wallet across other tokens?** (Case 007 open)

32. **Are `A7FMMgue4aZmPLLoutVtbC7gJcyqkHybUieiaDg9aaVE`, `4r33xEKAD2cNMrC9NyJy8nb4XmruUKebZ6LZZm65PVUZ`, `CoPyPPWdm8SirumaAEe8S68nSJpHYhguN2L5i2YGxLB`, and `AXmnRBrNtYYyyo82cLBBhnWJ7o1iqNLZbuEVpDB3V666` reusable automation infrastructure across tokens?** (Case 007 open)

33. **Does KOL-holder participation without proven payment commonly appear in Type 2C dirty launches that still make ATH expansion?** (Case 007 open)

34. **What separates YAE-style dirty-but-tradeable continuation from Cryptocrat-style post-ATH bleed?** (Case 007 open)

---

## Research Agenda

| Case | Status | What to find | Framework question |
|------|--------|-------------|-------------------|
| 006 | **COMPLETE** | GAPLA by SerialDev_02 | Does creator self-allocation dump recur on rapidlaunch.io? **NO** — different mechanism. rapidlaunch uses pre-launch StreamFlow allocation. |
| 007 | **COMPLETE** | Cryptocrat by SerialDev_03 | Is Type 2C real or YAE-only? **YES, n=2 confirmed** — same 9.645% dump pattern on a different dev. |
| 008 | Open | Another live dirty-but-tradeable pump | Can `DIRTY BUT TRADEABLE` be detected earlier before major run? |
| 009 | **RESOLVED** | Token where `DsCJ5siuJTPQtQa3A9N69azGZaWtUPzi9VPp2G9Jfpx9` appears again | **CONFIRMED across 3 cases (HANTA, YAE, Cryptocrat).** Bot is real infrastructure, not noise. |
| 010 | Open | Token where `BX7h4...` / `AxGt...` / `BaXmv...` recur | Confirm or reject YAE fee-payer candidates as bot infrastructure (full addresses still needed) |
| 011 | Open | Clean non-dormant organic pump | Establish the control sample against dirty-but-tradeable cases |
| 012 | Open | Failed dirty launch with dev self-allocation dump | What separates YAE-style survival from immediate death? |
| 013 | Open | Token where `4QfGpNuC5eeM9ieEp4qvFH49uWm4WqKTU4GQGvKZZyUc` or `6bZHJzbApPZck2gauwxmsEGkzPGZxxawoNuEFV1HtbDr` reappear | KOL network cross-token fingerprinting |
| 014 | Open | Any token where `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2vEMd3cxGS` or `DsCJ5siuJTPQtQa3A9N69azGZaWtUPzi9VPp2G9Jfpx9` reappear | Bot infrastructure cross-token confirmation |
| 015 | Open | Any token where `7ofDdxo7mMzfK25ZbrMt8ErvnvSLUNXp2Xn2jHesryxK` appears | Confirm professional trader recurrence |
| 016 | Open | Current live token pre-ATH | First real-time framework test |
| 017 | Open | Decode `DkzgUjvAd5G5ru7wvcacPKs67rqnzUuWGEQBfLu3CWkX` StreamFlow schedule | When do 27.58M GAPLA unlock? Is a dump incoming? |
| 018 | Open | Identify `BAr5csYtpWoNpwhUjixX7ZPHXkUciFZzjBp9uNxZXJPh` — trace to dev or independent | Is the rapidlaunch pre-launch allocation a recurring team model for SerialDev_02? |
| 019 | Open | Scan SerialDev_02 for FLASHX program interactions | How many rapidlaunch tokens did this dev create? |
| 020 | Open | Another rapidlaunch.io token (any dev) | Does the StreamFlow pre-launch allocation pattern recur? Confirm or reject as rapidlaunch platform norm. |
| 021 | Open | Third token with Type 2C pattern | Graduate Type 2C from `[HYPOTHESIS — n=2]` to `[CONFIRMED]` |
| 022 | Open | Token where Loopierr / Leens / cap / OGandtV2 / Nyhrox / @f4vel appear again | KOL-holder cross-token fingerprinting (no payment chain yet proven — track recurrence first) |
