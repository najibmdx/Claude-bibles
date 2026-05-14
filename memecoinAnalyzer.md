# Pump Forensics — Project Bible
_Living document. Updated after every case._
_Last updated: 2026-05-14_

---

## The Core Thesis

Every pump above ~$500K MC on a pump.fun token has a cause. That cause is always one of:
- **Organic narrative** (the name/concept catches fire independently)
- **KOL amplification** (paid or unpaid influencer posts)
- **Coordinated insider operation** (pre-seeded wallets, scheduled pump, planned exit)
- **Bot momentum** (automated trading creates artificial volume that attracts retail)
- **Cross-community spillover** (related token pumps, community migrates)

Most $1M+ pumps are combinations. The forensics job is to find which combination and prove it on-chain.

---

## Operation Typology

### Type 1: Clean KOL Pump
- Dev creates token, pays 1-3 KOLs in USDC/SOL
- KOLs post, retail buys, dev sells into it
- Signature: USDC payments from dev wallet → unknown wallets → Phantom forward → silence
- Typical scale: $200K–$2M ATH
- Example: **PAIN (this case)**

### Type 2: Serial Operator
- Same dev wallet creates 5-10 tokens in rapid succession
- Most fail, 1-2 get KOL treatment
- Cross-token coordination: profits from Token A fund the KOL budget for Token B
- Signature: Multiple CREATE events in dev wallet, several sibling tokens at $0 MC
- Example: **PAIN dev (5dirQCE7) — also running USDUC, ASSDAQ**

### Type 3: Insider Cluster
- Dev pre-seeds 5-20 wallets with tokens before any public activity
- Wallets look unrelated (different ages, different SOL balances) but received token from same source
- All wallets dump in coordinated window
- Signature: Token transfers FROM dev/insider TO fresh wallets days before ATH
- Example: **EgSptShw → 7jkA9z pre-load in this case**

### Type 4: Bot-Manufactured Volume
- Single automated address (fee payer) accounts for 50%+ of tx volume
- Creates illusion of activity, triggers algorithm-driven FOMO buying
- Real buyers pile in, dev exits
- Signature: 1-3 fee payers in 80%+ of sampled transactions
- Example: **FireuLYd taking 80% of PAIN order flow post-ATH**

### Type 5: Dual-Token Operation
- Dev runs two tokens simultaneously, uses one to cross-promote the other
- Allows dev to extract from both simultaneously
- Sophisticated: requires KOL buy-in on both narratives
- Signature: Sibling token with similar launch timing and similar holder patterns
- Example: **PAIN + USDUC running in parallel, USDUC at $7.2M**

### Type 6: Organic
- No USDC payments from dev wallet
- Dev wallet shows no coordinated pre-loads
- Top holders have real transaction history (not burner wallets)
- Tx volume comes from diverse fee payers
- Rare above $1M MC

---

## The KOL Payment Fingerprint

Paid KOL operations leave a consistent on-chain signature:

```
Dev wallet
  └─ USDC transfer to Wallet A  (payment, usually $100-$2000)
       └─ Wallet A forwards to Wallet B via Phantom  (within 5-30 min)
            └─ Wallet B goes silent  (actual KOL's cold wallet)

Timing: payment → post → ATH is typically 15-40 minutes
Amount tiers:
  $50-200: micro-KOL, CT account <10k followers
  $200-500: mid-tier KOL, 10k-50k followers  
  $500-2000: tier-1 KOL for this ecosystem, 50k+ followers
  $2000+: coordinated multi-platform push
```

The Phantom forward is the tell. Real OTC sales don't forward within 6 minutes. The forward is the KOL moving payment from their operational wallet to their actual receiving wallet.

---

## Dev Wallet Behavioral Patterns

### The Extractor (most common)
- Creates token, holds no tokens (all LP)
- Withdraws SOL directly from pump.fun bonding curve as price rises
- Pattern: WITHDRAW → TRANSFER to cashout wallet → repeat
- The withdrawals ARE the dump — they're selling LP position, not tokens

### The Dumper
- Creates token, holds % of supply
- Sells tokens via swap (SWAP transactions, not WITHDRAW)
- Often uses Jupiter or OKX DEX Router for better execution

### The Hybrid (PAIN dev)
- Withdraws from bonding curve (WITHDRAW) AND receives SOL from cashout wallets
- Routes through EgSptShw-type intermediary wallet that converts to USDC
- Full extraction: dev → cashout wallet → OKX swap → USDC → forward out

### Exit Signal
- Dev SOL balance drops to <0.01 SOL
- All USDC converted and forwarded
- No new PUMP_FUN interactions
- = Operation complete, token abandoned

---

## Cashout Infrastructure Patterns

Serial operators don't send funds to exchanges directly. They route through layers:

```
Pump.fun bonding curve
  └─ Dev wallet (extractor)
       └─ Intermediary wallet (EgSptShw type)
            ├─ OKX DEX Router swap → USDC
            └─ CEX deposit or further hops
```

The intermediary wallet is identifiable because:
- Receives SOL from dev wallet repeatedly
- Immediately swaps to USDC or stable
- Has both SOL receipt events and OKX/Jupiter swap events
- Often also holds token (dual role: insider bag + cashout relay)

---

## Tx Volume as Price Proxy

When price data isn't available, tx density per minute ≈ price action:

| Tx/minute | Market state |
|---|---|
| <10 | Dead / no interest |
| 10-100 | Slow accumulation |
| 100-500 | Growing momentum |
| 500-2000 | Active pump phase |
| 2000-5000 | ATH window / peak FOMO |
| 5000+ | Bot-dominated, organic buying peaked |

PAIN peaked at 4,618 tx/min at 04:35 UTC. That's deep into bot-dominated territory — organic retail was overwhelmed by automated trading at that point.

---

## Helius Pagination Notes

- `getSignaturesForAddress` returns max 1000 per call, use `before` param to paginate
- A very hot token (PAIN) generates 200k+ sigs in 90 minutes — you cannot paginate to creation in a reasonable time during peak activity
- Strategy: paginate forward from known creation sig, OR use dev wallet CREATE event timestamp to anchor the start
- Token-2022 blocks `getProgramAccounts` — always use `getTokenLargestAccounts` for holders (top 20 cap)
- DAS `getAsset` gives price_info for any token — useful for quick MC check on sibling tokens

---

## Case Registry

---

### CASE 001 — PAIN (Max Pain)
**Date:** 2026-05-14
**Token:** `2N9BXbRtyZM4YHzj9wvteW1Nuq6B5DzDwh57dDmZpump`
**Dev:** `5dirQCE7Dvn3Jopo4e4ra2fcdY7imDdJ1SYTaGmnYmwt`

**ATH:** ~$2.61M MC
**Current MC:** ~$196K
**Dev extracted:** 363+ SOL (~$54K+)
**Operation type:** Type 1 (KOL Pump) + Type 2 (Serial Operator) + Type 3 (Insider Cluster) + Type 4 (Bot Volume)

**Smoking gun:** $691 USDC payment from dev at 04:11 UTC → forwarded via Phantom at 04:17 → ATH hit at 04:30. 19-minute window from payment to peak.

**KOL Network:**
| Wallet | Payment | Role |
|---|---|---|
| `7jkA9zvNB843...` | $150 USDC + 4.66 SOL | KOL #1, pre-loaded with tokens 05-13 |
| `EgSptShwdAQV...` | $150 USDC + 3.45 SOL + tokens | Core insider / cashout relay |
| `4QfGpNuC5eeM...` | $691 USDC | KOL payment collector |
| `6bZHJzbApPZc...` | $691 USDC (forwarded) | Actual KOL receiving wallet |

**Insider Pre-load:** `EgSptShw` → `7jkA9z`: 75,000 PAIN tokens on 05-13 16:14 (day before pump)

**Cashout route:** Dev → `EgSptShw` → OKX DEX Router → USDC → forward out

**Dev sibling tokens:**
| Token | Symbol | MC at time of analysis |
|---|---|---|
| `CB9dDufT3ZuQ...` | USDUC | $7.2M (also live, dual operation) |
| `7Tx8qTXSakpf...` | ASSDAQ | $290K |
| `2KiHzSXdnenD...` | poop | $0 (failed) |
| `yFtG6tU4mZqi...` | JAIL | $0 (failed) |
| `AzsiFNsZniJX...` | Assem | $0 (failed) |
| `HF5DkVyJDAtS...` | pennystock | $0 (failed) |

**7 Question Answers:**
1. Manipulated? **Yes** — coordinated extraction from day 1
2. Coordinated? **Yes** — multi-wallet pre-load, timed USDC payments
3. Natural? **No**
4. Community-driven? **No**
5. Real-world event? **No** — "max pain" is a financial meme, not a live event
6. Celebrity? **No**
7. KOL/influencer? **Yes** — 3 paid KOLs, $991 total, main KOL received $691

**Verdict:** Professional memecoin operation. Dev is a serial launcher running at least 3 simultaneous tokens. PAIN was the smaller play; USDUC is the primary vehicle. Both used the same cashout infrastructure. Operation complete — dev wallet empty.

---

## Known Wallets Registry

### Dev Wallets (confirmed bad actors)
| Wallet | Alias | Confirmed Tokens | Last Active | Notes |
|---|---|---|---|---|
| `5dirQCE7Dvn3Jopo4e4ra2fcdY7imDdJ1SYTaGmnYmwt` | SerialDev_01 | PAIN, USDUC, ASSDAQ + 4 failed | 2026-05-14 | Professional operator, dual-token strategy |

### Insider / Cashout Wallets
| Wallet | Role | Connected To | Notes |
|---|---|---|---|
| `EgSptShwdAQVgNZ5v4fFaStUQnrig9wDtx9jfQBg7cTp` | Cashout relay + insider | SerialDev_01 | Receives SOL from dev, swaps to USDC via OKX |
| `4QfGpNuC5eeM9ieEp4qvFH49uWm4WqKTU4GQGvKZZyUc` | KOL payment collector | SerialDev_01 | Collects USDC, forwards to actual KOL |
| `6bZHJzbApPZck2gauwxmsEGkz3yHRzLJdkqFnuQ2K7pH` | KOL final destination | 4QfGpNuC5e | Phantom-received $691 USDC |
| `7jkA9zvNB84319exePVn5Cu3kYd4oPF1rW8pobZyDhtX` | KOL #1 | SerialDev_01 + EgSptShw | Pre-loaded with tokens day before pump |

### Bot / Automated Addresses
| Wallet | Type | Seen On | Notes |
|---|---|---|---|
| `FireuLYd4yjJBhXQyBs3Mq6ZpNEjyHPNPG2...` | Trading bot | PAIN | 80% fee payer share in post-ATH window |

---

## Open Questions / Future Research

1. **Who is `6bZHJzbA`?** The final $691 USDC destination — likely a CT/Telegram KOL but unidentified. Cross-referencing with social media is the next step.

2. **USDUC full anatomy.** Same dev, $7.2M MC, still live. Full Layer 1-7 analysis pending.

3. **Is `EgSptShw` the dev's own wallet?** The dual role (insider bag + cashout relay + received SOL from dev) suggests this may be the dev's secondary operational wallet, not a separate KOL.

4. **The Meteora DLMM pool.** Who created it? When? This level of sophistication suggests either the dev has DeFi infra experience, or a third party (market maker?) added LP for a fee.

5. **Scale of the operation.** 363 SOL from PAIN + unknown from USDUC. If USDUC is at $7.2M and follows the same pattern, total extraction could be 1000+ SOL across both tokens.

---

## Methodology Notes

### What makes this different from Spider

Spider answers: **is this token about to dump?**
This project answers: **how was the pump manufactured and who did it?**

Spider is forward-looking risk. This is backward-looking forensics.

They are complementary. Spider catches the pattern live. This project autopsies what Spider caught and builds the institutional knowledge that makes future Spider signals better.

### The Minimum Viable Evidence Standard

To call a pump "coordinated" we need at least TWO of:
- USDC payment from dev wallet within 1 hour of ATH
- Token pre-load to non-dev wallet 12-72 hours before ATH
- Multiple sibling tokens from same dev (serial operator pattern)
- Cashout relay wallet (dev sends SOL to intermediary who swaps to stable)
- Bot dominance (single fee payer >50% of tx volume)

PAIN had all five. Most cases will have 2-3.

### The 19-Minute Rule

In every paid KOL pump analysed so far: payment → Phantom forward → ATH is 15-40 minutes.

The KOL gets paid, moves funds to their real wallet, posts immediately, algorithm/bots front-run the post, retail sees price moving, FOMO in. The speed is the tell. No legitimate OTC transaction settles and triggers a social post in 19 minutes.
