# MemecoinAnalyzer_Bible.md

_Living document. Updated after every case._
_Last updated: 2026-05-16 (Session 6 complete — BABYHANDS Case 004 + Layer 0 bundle check added)_

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

_Example: BABYHANDS Case 004 — 84.5% of supply sent to fresh wallet HB1LvHc5 in creation block. Entire position dumped in 20 min 18 sec._

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

### Type 2B: Serial Operator + Bundle `[NEW — confirmed Case 004]`

- Serial operator who uses bundle launch instead of KOL payments for distribution
- Lower operating cost than Type 1 (no USDC KOL spend), lower ceiling ($36K ATH vs millions)
- Supply control at launch replaces paid distribution
- Example: BABYHANDS Case 004

### Type 3: Insider Cluster

- Dev pre-seeds 5–20 wallets with tokens before any public activity
- Signature: Token transfers FROM dev/insider TO fresh wallets 12–72 hours before ATH
- Example: EgSptShw → 7jkA9z pre-load (PAIN)

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

---

## 4. Dev Wallet Behavioral Patterns

### The Fee Collector
Creates token, earns SOL via `CollectCreatorFee`. Higher volume = more fees. Incentive to drive volume.

### The Dumper
Creates token, holds % of supply. Sells via SWAP transactions. Evidence: large SWAP events from dev wallet involving the token mint.

### The Bundle Operator `[NEW — confirmed Case 004]`
Pre-allocates large % of supply (>50%) to a controlled wallet in the creation transaction. That wallet dumps into retail during the pump. Dev simultaneously collects creator fees. Full exit in hours via fresh cashout wallet.
- Signature: dev as fee payer on creation tx, large % to fresh wallet in block 0, fresh wallet sells to zero quickly, dev balance goes to 0.
- Lower overhead than KOL pump — no USDC payments required.
- Cashout wallet typically has <5 lifetime txs, created same day.

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

### Additional Check — Bundle Detection `[NEW — confirmed Case 004]`

```
[ ] BUNDLE CHECK (run before all other checks):
    Fetch first 20 txs on token address chronologically.
    On creation tx: is dev the fee payer?
    On creation tx postTokenBalances: did any fresh wallet receive >5% supply?
    → YES to both: BUNDLED LAUNCH. Do not enter. Dev controls supply exit.
    → NO: continue to standard checklist.
```

### The Exit Rules

**Primary trigger (KOL pump):** First USDC payment detected from dev wallet. Start 20-minute timer. Full exit by expiry.

**Bundle pump exit:** No USDC to watch for. Monitor bundle wallet balance. When bundle wallet approaches zero, dev is almost done extracting. Do not enter after bundle wallet has begun selling.

**Hard stop:** Dev SOL balance drops below 0.05 SOL at any point. Exit immediately.

**Dormant token exit:** When a 100+ SOL wallet starts buying in rapid succession, begin scaling out within 10 minutes.

---

## Helius API Technical Notes

- `getSignaturesForAddress` max 1,000 per call — use `before` param to paginate
- Token-2022 blocks `getProgramAccounts` — use `getTokenLargestAccounts` for holders (top 20 cap)
- DAS `getAsset` gives price_info for quick MC check
- Always use `getTransaction` with `jsonParsed` to verify instruction type before concluding on any WITHDRAW
- `postTokenBalances` in transaction data shows who received tokens — critical for bundle detection

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

**Insider pre-load:** `EgSptShw` → `7jkA9z` 75,000 tokens at 13h 45m before ATH.

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

**Verdict:** Organic Type 7. No manipulation infrastructure. Professional traders (Cupsey3 + 7ofDdxo7) captured most upside via Moonshot verification signal.

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
Tx `F7ZEn12eQctG5oXk537uS27wPTR3f8EaYwapuY7pboN4uArVtt2PHq1vMdyzy5LFGrTX352rkXWRzuLyym3Qb5W` at 2026-05-16 03:00:38 UTC — dev `7pKUwa3MZjcw3vFbW1UDAuAKEQtz8T8n82SCRXPkz4dU` is fee payer, `845,076,438 tokens (84.5% of total supply)` sent to fresh wallet `HB1LvHc5Z6zhUCRjXNBdf4HL3ZYWRRkqMAmuK6Sua3Qo` in the creation block. HB1LvHc5 sold the entire position to zero in 20 minutes 18 seconds (03:00:38–03:20:56 UTC), cycling 3,457M tokens total via automated sandwich selling.

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
- Mar 24: sent 4.066 SOL TO dev. May 14: received 4.172 SOL back. May 15: sent 6.357 SOL to relay `6JM1poX`
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
2. Coordinated? **Partial** — J87UST7 capital flows confirmed, SOL relay infrastructure, fresh cashout wallet
3. Natural? **No**
4. Community-driven? **Unknown** — demand-side catalyst not verifiable (social data unavailable)
5. Real-world event? **Unknown** — "baby hands" = Trump political reference; plausible but unconfirmed
6. Celebrity? **No evidence**
7. KOL-driven? **No** — zero USDC payments, zero registry hits

**Open questions from Case 004:**
- What triggered demand-side retail volume beyond first-block snipers? (LunarCrush paid subscription needed)
- Are sniper bots (`5brv79`, `AVMpb2bX`, `FTXgeRx`) connected to each other or same entity?
- Does `7pKUwa3` dev reuse the same bundle wallet pattern on prior sibling tokens?

---

## Known Wallets Registry

### Dev Wallets

| Wallet | Alias | Confirmed Tokens | Notes |
|--------|-------|-----------------|-------|
| `5dirQCE7Dvn3Jopo4e4ra2fcdY7imDdJ1SYTaGmnYmwt` | SerialDev_01 | PAIN, USDUC, ASSDAQ + 4 failed | Professional KOL pump operator. Self-financing creator fee model. |
| `5P7SSkFxrjk9rEUZJ6BLeELLW5B9bWykvzFFMqSrzHzS` | fuckjerry (pump.fun) | HANTA, TERMAI, GMECUBE | CEX exit dev. Arkham confirmed. 3 tokens, 2 failed. |
| `7pKUwa3MZjcw3vFbW1UDAuAKEQtz8T8n82SCRXPkz4dU` | BundleDev_01 | BABYHANDS + 7 failed | Serial operator. Bundle launch model. 8 tokens (Oct 2025–May 2026), 7 at $0. Fully exited BABYHANDS in 2.5 hrs. |

### Insider / KOL Network Wallets

| Wallet | Role | Connected To | Notes |
|--------|------|--------------|-------|
| `EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp` | Insider + KOL relay | SerialDev_01 | Received $150 USDC. Pre-loaded 7jkA9z with 75k PAIN tokens 13h 45m before ATH. |
| `7jkA9zvNB84319exePVn5Cu3kYd4oPF1rW8pobZyDhtX` | KOL #1 | SerialDev_01 | Pre-loaded 75k tokens 13h before ATH. $150 USDC from dev. |
| `4QfGpNuC5eeM9ieEp4qvFH49uWm4WqKTU4GQGvKZZyUc` | KOL payment collector | SerialDev_01 | Received $691 USDC. Phantom-forwarded within 6 min. |
| `6bZHJzbApPZck2gauwxmsEGkzPGZxxawoNuEFV1HtbDr` | KOL final destination | 4QfGpNuC | Received $691 via Phantom. Silent after. Identity unknown. |
| `ARu4n5mFdZogZAravu7CcizaojWnS6oqka37gdLT5SZn` | DEX router / USDC conversion | SerialDev_01 | Converted 479 SOL creator fees → $991.77 USDC. |
| `2fg5QD1eD7rzNNCsvnhmXFm5hqNgwTTG8p7kQ6f3rx6f` | KOL trader / known CT | HANTA Case 003 | Cupsey3 (@Cupseyy). Bought 11.4M HANTA for 485 SOL. |
| `7ofDdxo7mMzfK25ZbrMt8ErvnvSLUNXp2Xn2jHesryxK` | ATH whale — professional trader | HANTA Case 003 | ~740 SOL into HANTA. PumpSpecialRouter. Polymarket: "alphajj". |
| `J87UST7jVkLWxf5irqTJTSTYPoevXy3jKrFNw8Pr665f` | Co-operator dev | BABYHANDS Case 004 | Own token creator. Bilateral SOL flows with BundleDev_01. Never held BABYHANDS. |

### Bundle Infrastructure Wallets

| Wallet | Role | Connected To | Notes |
|--------|------|--------------|-------|
| `HB1LvHc5Z6zhUCRjXNBdf4HL3ZYWRRkqMAmuK6Sua3Qo` | Bundle receiver | BundleDev_01 / BABYHANDS | Received 845M tokens (84.5% supply) at creation. Dumped to zero in 20 min 18 sec. Fresh wallet, automated. |
| `EVNJsBsTM373wdJFqGj9iiYvRiv8yViCjE5zBr4afW9W` | Dev cashout | BundleDev_01 / BABYHANDS | Received 11.60 SOL at 05:12:58 UTC. 3 txs total. Created same day. |
| `6JM1poX1S25FW75qSFekJDK9g4kj3Z3C2UNFedzFcKCL` | SOL relay/staging | BundleDev_01 / BABYHANDS | Repeatedly moves SOL between dev and J87. 0 SOL. Fully drained. |
| `D7DDkxgSd8ySgEv971AA2vgFAk4btgw9anChz7jvkRZM` | Burner funder | BundleDev_01 / BABYHANDS | Funded relay 2.94 SOL. 2 txs. Created solely for this. |

### HANTA Cashout Chain Wallets

| Wallet | Role | Notes |
|--------|------|-------|
| `FiST32C9csQPCTt8SZ9KYa9zGfcfpipKx7eYqXFBFYad` | SOL→USDC relay | 12 SOL → 1,072 USDC → GxjcMTWW. Liquidation. |
| `325f2VGzRT6U1NS94nBxqXTvh6xYsLyxcDCYPXCidWXN` | SOL→USDC relay | 20 SOL → 1,790 USDC → GxjcMTWW. Liquidation. |
| `83a9F4EvaDaQcZxnTKWP5DpyMAfhmfVkpqeaB6rGsJ3w` | SOL→USDC relay | 8.58 SOL → 766 USDC → GxjcMTWW. |
| `Bhtaq7F9z3WFEXsVVGpnSwQsKdTnZKne3o5vgbBa7yWb` | SOL→USDC relay | 10.6 SOL → 947 USDC → GxjcMTWW. |
| `GxjcMTWWDcKx5sxnHkNfPtuTSfV9cLgVkw3G7KvMnvBW` | Multi-chain trading aggregator | Received all HANTA USDC. GMGN, LiquidMesh, Wintermute counterparty. NOT a KOL desk. |
| `XEfwn8pCxSZyp7iMqUCyEnm7eRPS6qqcHnKRvyXQEEE` | Trading capital | Arkham: scooterxbt.sol. Received 14 SOL from fuckjerry May 6. |
| `AmidQKtyerkVPGTWjzcig3sjsHwNwHPzrZdN2muMrRCd` | SOL aggregator | 68+ SOL from fuckjerry → Coinbase deposit. |
| `H1xmLoZSxjA9akJCbV4PrFkRyNFx59LpwBp5E3Uor21Z` | CEX deposit | Arkham: Coinbase Deposit. Dev cashing out. |

### Bot / Automated Addresses

| Wallet | Type | Seen On | Notes |
|--------|------|---------|-------|
| `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2vEMd3cxGS` | MEV Sandwich Bot (Jito) | PAIN | 80% fee payer at ATH. |
| `DsCJ5siuJTPQtQa3A9N69azGZaWtUPzi9VPp2G9Jfpx9` | MEV Sandwich Bot (Jito) | HANTA re-pump | 55% fee payer. 1,935 tx/min. Do not enter when >40%. |
| `roUteHjDohtkatXTb79PJ99bbxkTipgo3GJ4EJZ1YpB` | Copy-trade bundler | HANTA early AMM | 92 tx/min. Bundled buy orders. |
| `5brv79eFZ2rGprXNvqgVJBkBptkkw8GJX1XydJyZLyAr` | Indiscriminate Launch Sniper | BABYHANDS | Created 2026-05-15. 1000+ txs/14 hrs. Non-stop PUMP_FUN SWAPs. Snipes every launch. |
| `AVMpb2bXMdNec6ykkEvf7krff7nXMZuKvadPELPNA9M6` | Indiscriminate Launch Sniper | BABYHANDS | Created 2026-05-15. Same bot pattern. |
| `FTXgeRxnr2ZU1dMKjLfEF6i7j3MXrAVkj317oRFtSFjK` | Launch Sniper | BABYHANDS | Created 2026-05-08. 1000+ txs in 8 days. |

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

6. **April 2026 history of `EgSptShw` and `7jkA9z`?** Prior token collaboration?

7. **Does `FireuLYd` appear on USDUC?** Cross-token bot confirmation pending.

8. **Who is `7ofDdxo7mMzfK25ZbrMt8ErvnvSLUNXp2Xn2jHesryxK`?** Polymarket: "alphajj". No Arkham entity.

9. **What is the relationship between scooterxbt.sol and fuckjerry?**

10. **Did Cupsey3 (@Cupseyy) post about HANTA publicly?**

11. **What CT post first surfaced HANTA on May 6?**

12. **What triggered demand-side retail volume on BABYHANDS beyond first-block snipers?** (LunarCrush subscription needed)

13. **Are sniper bots `5brv79`, `AVMpb2bX`, `FTXgeRx` connected to each other or same entity?**

14. **Does `7pKUwa3` dev reuse bundle wallet pattern on prior sibling tokens?**

---

## Research Agenda

### Next: Case 005

BABYHANDS confirmed a new archetype (Type 2B bundle operator). Next deliberate selections:

| Case | What to find | Framework question |
|------|-------------|-------------------|
| 005 | Token where bundle wallet appears on multiple launches by same dev | Does `7pKUwa3` reuse bundle infrastructure? |
| 006 | Fresh organic pump, no KOL payments, no bundle | What does clean non-dormant organic look like? |
| 007 | Failed operation | What separates failure from success pre-catalyst? |
| 008 | Token where `4QfGpNuC` or `6bZHJzbA` reappear | KOL network cross-token fingerprinting |
| 009 | Sub-1-hour ATH from launch | Does pre-load/bundle pattern compress? |
| 010 | Current live token pre-ATH | First real-time framework test |
| 011 | Any token where `FireuLYd` or `DsCJ5` reappear | Bot infrastructure cross-token confirmation |
| 012 | Any token where `7ofDdxo7` appears | Confirm professional trader recurrence |
