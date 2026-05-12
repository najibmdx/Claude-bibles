# SPOTLIGHT — THE BIBLE
_Canonical Reference Document for the Synthesis Layer_
_Last updated: 2026-05-12 | Covers: Spec, Architecture, Signals, Decisions, Validation, Build Sequence, Constraints_

---

## PART 0: WHAT THIS DOCUMENT IS

This is the single source of truth for Spotlight. Every session that touches Spotlight must start here. Nothing in this document is up for debate unless explicitly marked with `[OPEN]`. Locked decisions are marked `[LOCKED]`. Violations of locked decisions require the full session that established them to be re-litigated before changing them.

If a new Claude session is contradicting this document, show it this file and stop.

---

## PART 1: WHAT SPOTLIGHT IS (AND IS NOT)

### What It Is
Spotlight is the **decision synthesis layer** that sits above Spider (on-chain risk) and Beat (social momentum). It reads both tools' outputs, applies context-aware signal weighting, runs a decision matrix, and produces one of four verdicts — each with specific trade parameters attached.

It is a **300-400 line Python HTTP server**. Not a model. Not an agent. Not a merger of Spider and Beat. A deterministic synthesis engine.

**Port:** 8767  
**Endpoints:** `GET /analyze?mint={address}`, `GET /health`  
**Dependencies at runtime:** Spider on 8765, Beat on 8766  
**Response time target:** < 2 seconds  

### What It Does NOT Do
- Predict when a dump will happen
- Guarantee where the bottom is
- Set price targets
- Make buy or sell decisions for the user
- Merge Spider and Beat into one tool
- Use an LLM for any numerical calculation or verdict

### What It DOES Do
- Classify tokens into SKIP / LOTTERY / ACCUMULATE / CONVICTION
- Attach specific trade parameters to each verdict (size %, stop type, time box)
- Detect floor strength within 60 seconds of major drops (when re-run)
- Identify whether continuation conditions are present or absent
- Override to CONVICTION when a known insider wallet is detected
- Override to SKIP when a dead chart or coordinated distribution is detected

---

## PART 2: ARCHITECTURE PRINCIPLES [LOCKED]

These were established across multiple sessions with real capital lost and learned. Do not touch them.

**1. Spotlight is NOT a merger.**
Spider and Beat stay separate permanently. Different data rhythms, different APIs, different failure modes. Merging them means one noisy probabilistic input can override clean deterministic data, and debugging becomes entangled maintenance hell. Spotlight is a third layer that reads their outputs — not a combined tool.

**2. LLMs never touch numbers.**
Mistral (Spider's narrative model) and any AI used in Beat write commentary only. All logic, scoring, weighting, and verdict calculation lives in Python. An LLM that produces "Power Score: 72" or "LOTTERY" is a violation of this principle regardless of how it's framed.

**3. Verdicts, not scores.**
A score (0-100) requires interpretation under pressure. A verdict (LOTTERY) with trade parameters (0.5% size, stop below launch price, 10-minute time box) is an instruction. The HENRY mistake happened because Spider's output required the user to interpret a number under time pressure. That failure mode must not be reproduced.

**4. Temporal context is mandatory.**
"Bundle detected" ≠ "bundle is the current threat." When the bundle exited relative to when organic accumulation started is the signal that matters. A bundle that exited 2 minutes ago with organic wallets already buying is a STALE signal. Treating it as a live threat is the core error that cost the HENRY trade.

**5. Vol/Liq ratio is the primary demand proxy.**
Market cap and price change are lagging indicators. Vol/Liq ratio is real-time demand intensity. 16x+ means people are actively buying relative to available liquidity regardless of what the on-chain cleanliness looks like. It is the first number checked and the one with the most override power.

**6. No absolute language.**
"Pure farm and dump — skip" is too final. It psychologically overrides the user's own read. Every output from Spotlight must be conditional: here is what was detected, here is what it means, here is what to watch. The LOTTERY verdict explicitly preserves risk — it does not pretend the risk doesn't exist.

**7. HENRY at 99K is the benchmark.**
Any Spotlight logic that outputs SKIP on HENRY at 99K MC is wrong. Spider score was LOW. Beat was HIGH. Vol/liq was 16x. Bundles were stale (exited at T+0, organic started T+7). Organic accumulation was active. LunarCrush was spiking. The correct verdict was LOTTERY. If the matrix produces SKIP on HENRY's exact inputs, the weighting is wrong and must be iterated.

**8. Historical validation before live capital.**
No new Spotlight logic touches real trades until it correctly classifies 5+ historical tokens with known outcomes. The 5 test cases defined in Part 7 of this document are the minimum gate.

**9. No database in Phase 1.**
Spotlight Phase 1 is fully stateless. Every verdict is computed independently from scratch. Phase 2 adds a lightweight append-only SQLite log. Phase 3 adds wallet reputation scoring from that log. Nothing from Phase 2 or 3 gets pulled into Phase 1 scope.

**10. Spotlight outputs situational awareness, not prediction.**
The money is in reading the present more accurately and faster than everyone else, not predicting the future. Detection is achievable. Prediction is not. Every feature added to Spotlight must be framed as detection or classification, never prediction.

---

## PART 3: THE STACK CONTEXT

### Spider (Port 8765)
Chrome/Brave extension injecting live analysis panel into trading platforms. Python backend at `C:\iSight\MiniCowScanner\Duck-GooseMerger\WalletScanner\Monkey\Spider\`.

What Spider is good at: bundle detection, holder concentration, LP manipulation detection, operator fingerprinting via pre-launch SOL funding patterns, DEX infrastructure wallet exclusion, PDA account filtering.

What Spider does NOT do: temporal context on bundles, post-dump holder behavior, vol/liq ratio as demand signal, social signals, launchpad context.

Spider's current output (before Spotlight's new on-chain checks are added to the enriched endpoint): Power Score 0-100, bundle count, holder concentration, clean distribution flag, Mistral narrative.

### Beat (Port 8766)
Separate extension focused solely on the social layer. Operational. Fully built.

Beat's output includes: `risk` (CRITICAL/HIGH/MEDIUM/LOW/SAFE), `verdict` (AI archetype), `mention_count`, `twitter_count`, `telegram_count`, `velocity_analysis` (with `overall_momentum`: ACCELERATING/STABLE/DECELERATING/INSUFFICIENT_DATA), `lunarcrush` (alt_rank, galaxy_score, sentiment, classifications), `telegram_quality` (HIGH/MODERATE/LOW/NO_DATA).

Beat's AI model outputs social signals only and never issues trading verdicts. ACTION format is always: `"Social risk [LEVEL] — [detected]. Verify on-chain: [what to check] before acting."` Beat does not need changes to support Spotlight integration.

### Spotlight (Port 8767)
Does not exist yet. This is what gets built.

### File Locations
```
Spider backend:  C:\iSight\MiniCowScanner\Duck-GooseMerger\WalletScanner\Monkey\Spider\
Beat backend:    C:\iSight\MiniCowScanner\Duck-GooseMerger\WalletScanner\Monkey\Beat\backend\
Spotlight:       C:\iSight\MiniCowScanner\Duck-GooseMerger\WalletScanner\Monkey\Spotlight\  (to be created)
```

### Technical Constants
```python
SPOTLIGHT_PORT       = 8767
SPIDER_URL           = "http://localhost:8765"
BEAT_URL             = "http://localhost:8766"
HELIUS_KEY           = "ffb36aa1-17d5-4dcc-94d6-d2f7567cbb68"
MISTRAL_KEY          = "YwWIusyDqIFqSaSPsiYZ2RMfsT9CnCNS"   # narrative only
RPC                  = f"https://mainnet.helius-rpc.com/?api-key={HELIUS_KEY}"

# Known programs
TOKEN_2022           = "TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb"
PUMPSWAP_AMM         = "pAMMBay6oceH9fJKBRHGP5D4bD4sWpmSwMn52FMfXEA"
METEORA_DAMM_V2      = "cpamdpZCGKUy5JxQXB4dcpGPiikHawvSWAd6mEn1sGG"
METEORA_DLMM         = "LBUZKhRxPF3XUpBCjp4YzTKgLLjDo9gmSKy58zFur6a"
RAYDIUM_V4           = "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8"
UXENTO_PROGRAM       = "DDsnwb7dxKSjzTYDFjU8F6rpYNZa1sp7Fmfb2nGDAMEo"

# Spotlight signal thresholds
VOL_LIQ_HOT          = 16      # ≥ this = HOT
VOL_LIQ_ACTIVE       = 8       # ≥ this = ACTIVE
VOL_LIQ_WEAK         = 2       # ≥ this = WEAK
VOL_LIQ_DEAD         = 2       # < this = DEAD (override to SKIP)
BUNDLE_STALE_MINS    = 10      # bundles older than this + organic buying = STALE
HOLDER_CHECK_TOP_N   = 5       # non-LP holders to behavior-check
DUMP_TRIGGER_PCT     = 40      # price drop % in 60 mins triggers floor check
CACHE_TTL_SECONDS    = 300     # 5 minutes
```

---

## PART 4: THE TWO NEW ON-CHAIN CHECKS

These are net-new capabilities that do not exist in Spider or Beat. They live in the Spider backend as part of a new enriched deep scan endpoint, and Spotlight calls them.

### Check 1: Temporal Bundle Analysis

**What exists now:** Spider flags "N bundles detected" as a static, permanent negative. The flag does not change regardless of what happens after.

**What must be added:** Timestamp when bundles exited + timestamp when the first organic wallet accumulation started on this token. Classify the bundle signal as ACTIVE, FRESH, STALE, or CLEARED.

**Classification rules:**
```
ACTIVE   — bundles are still holding tokens (not yet exited)
FRESH    — bundles exited 0-10 minutes ago, no organic accumulation yet started
CLEARED  — bundles exited, time >10 mins, but no clear organic accumulation detected
STALE    — bundles exited, organic accumulation has started (at least one non-bundle wallet accumulating)
```

**Why STALE matters:** On HENRY, bundles exited at T+0. Organic wallets (663Ny72, FMcuUHe) started accumulating at T+7 to T+21. By the time of Spider's scan at T+2 (99K), the bundle threat was already STALE. The risk had been absorbed. Spider's static "bundle detected" treated a historical event as a live threat.

**API cost:** 5-8 additional Helius calls per scan. Acceptable.

**Output fields to add to Spider's enriched endpoint:**
```json
"bundle_status": "ACTIVE" | "FRESH" | "STALE" | "CLEARED" | null,
"bundle_delta_mins": float | null,    // minutes between bundle exit and organic start
"bundle_exit_time": "ISO8601" | null,
"organic_start_time": "ISO8601" | null
```

---

### Check 2: Live Top-5 Holder Behavior

**What exists now:** Spider checks holder concentration as a static snapshot. It does not check what those holders are doing right now.

**What must be added:** For each of the top 5 non-LP holders, pull their last 2 transactions on this specific token and classify their current behavior.

**Classification per wallet:**
```
BUYING   — last transaction was a buy on this token
HOLDING  — no recent transactions on this token
SELLING  — last transaction was a sell on this token
```

**Aggregate classification:**
```
CONVICTION      — 4-5 wallets BUYING or HOLDING, 0-1 SELLING
MIXED_BULLISH   — 3 BUYING/HOLDING, 1-2 SELLING
NEUTRAL         — roughly even split
MIXED_BEARISH   — 3 SELLING, 1-2 BUYING/HOLDING
DISTRIBUTION    — 4-5 wallets SELLING (override to SKIP)
```

**Why this matters:** On HENRY at the 600K dump, 7jsapQi had never sold a single token. DcdzbFztVZ had never sold. FMcuUHe was actively rebuying. The floor was real and readable from on-chain data. Spotlight running this check during a dump would have output FLOOR REAL within 60 seconds.

**API cost:** ~10 additional Helius calls per scan. Acceptable.

**Output fields to add:**
```json
"top_holder_behavior": {
    "behavior_classification": "CONVICTION" | "MIXED_BULLISH" | "NEUTRAL" | "MIXED_BEARISH" | "DISTRIBUTION" | null,
    "buying": int,
    "holding": int,
    "selling": int,
    "wallets_checked": int
}
```

---

## PART 5: INPUT DATA CONTRACTS

Spotlight calls two APIs. These are the exact fields it requires from each. If Spider or Beat changes their output schema and removes required fields, Spotlight must fail gracefully (not crash) and degrade the verdict confidence.

### From Spider — Enriched Deep Scan

**Endpoint:** `GET http://localhost:8765/analyze/deep/enrich?mint={token_address}`

**Required fields:**
```json
{
  "mint": "string — token address",
  "name": "string",
  "symbol": "string",
  "market_cap_usd": "float",

  "vol_liq_ratio": "float",
  "vol_liq_class": "HOT | ACTIVE | WEAK | DEAD | NO POOL",

  "bundle_status": "ACTIVE | FRESH | STALE | CLEARED | null",
  "bundle_delta_mins": "float | null",

  "top_holder_behavior": {
    "behavior_classification": "CONVICTION | MIXED_BULLISH | NEUTRAL | MIXED_BEARISH | DISTRIBUTION | null",
    "buying": "int",
    "holding": "int",
    "selling": "int"
  },

  "known_wallets_detected": {
    "total_count": "int",
    "smart_money_count": "int",
    "insider_count": "int",
    "kol_count": "int",
    "details": "array"
  },

  "buy_pressure": "float — ratio buys/sells",
  "buys_1h": "int",
  "sells_1h": "int"
}
```

**Note:** The `/analyze/deep/enrich` endpoint does not exist yet. It is the enriched version of `/analyze/deep` that includes Check 1 and Check 2 outputs. Building this endpoint is part of the Spider work required before Spotlight can be integrated.

---

### From Beat

**Endpoint:** `GET http://localhost:8766/analyze?mint={token_address}`

**Required fields:**
```json
{
  "ca": "string — token address",
  "token_name": "string",
  "symbol": "string",

  "risk": "CRITICAL | HIGH | MEDIUM | LOW | SAFE",
  "verdict": "string — AI archetype description",

  "mention_count": "int",
  "twitter_count": "int",
  "telegram_count": "int",

  "velocity_analysis": {
    "overall_momentum": "ACCELERATING | STABLE | DECELERATING | INSUFFICIENT_DATA",
    "mentions_velocity": "string",
    "creators_velocity": "string",
    "comparison_window_minutes": "float",
    "data_points_available": "int"
  },

  "lunarcrush": {
    "found": "bool",
    "alt_rank": "int | null",
    "galaxy_score": "int | null",
    "sentiment": "int | null",
    "alt_rank_class": "TOP_10 | TOP_50 | NOTABLE | LOW | UNKNOWN",
    "sentiment_class": "BULLISH | NEUTRAL | BEARISH | UNKNOWN"
  },

  "telegram_quality": {
    "overall_quality": "HIGH | MODERATE | LOW | NO_DATA"
  }
}
```

**Note:** Beat is already operational and already outputs all of these fields. No changes to Beat required to support Spotlight.

---

## PART 6: THE SIGNAL WEIGHTING ENGINE

All calculation is in Python. No exceptions.

### Base Scores

**Spider base score (from bundle status):**
```python
if bundle_status in ("ACTIVE", "FRESH"):
    spider_base = 20   # LOW — active threat
elif bundle_status in ("STALE", "CLEARED"):
    spider_base = 40   # MED — historical threat, reduced weight
else:                   # null — no bundles detected
    spider_base = 50   # MED-HIGH
```

**Beat base score (from risk level):**
```python
if beat_risk in ("CRITICAL", "HIGH"):
    beat_base = 70     # HIGH — strong social signal
elif beat_risk == "MEDIUM":
    beat_base = 50     # MED
else:                   # LOW, SAFE
    beat_base = 20     # LOW — social quiet
```

---

### Spider Signal Modifiers

**Vol/Liq Ratio (applied to spider_adjusted):**
```python
if vol_liq_class == "HOT":        # ≥ 16x
    modifier = +20
elif vol_liq_class == "ACTIVE":   # 8-16x
    modifier = +10
elif vol_liq_class == "WEAK":     # 2-8x
    modifier = 0
elif vol_liq_class == "DEAD":     # < 2x
    modifier = -50  # triggers SKIP override
else:                              # NO POOL
    modifier = -30
```

**Bundle Temporal Status (applied to spider_adjusted):**
```python
if bundle_status == "STALE":      # exited >10 mins ago, organic started
    modifier = +15
elif bundle_status == "CLEARED":  # exited, no organic yet
    modifier = +5
elif bundle_status == "FRESH":    # exited 0-10 mins ago
    modifier = 0
elif bundle_status == "ACTIVE":   # still holding
    modifier = -20
else:                              # no bundles
    modifier = +10
```

**Holder Behavior (applied to spider_adjusted):**
```python
if behavior == "CONVICTION":       # all holding/buying
    modifier = +20
elif behavior == "MIXED_BULLISH":  # 3-4 holding/buying
    modifier = +10
elif behavior == "NEUTRAL":
    modifier = 0
elif behavior == "MIXED_BEARISH":  # 3-4 selling
    modifier = -15
elif behavior == "DISTRIBUTION":   # all 5 selling — triggers SKIP override
    modifier = -30
else:                               # null, no data
    modifier = 0
```

**Known Wallets (applied to spider_adjusted):**
```python
if known["insider_count"] >= 1:
    modifier = +40  # triggers CONVICTION override
elif known["smart_money_count"] >= 3:
    modifier = +25
elif known["smart_money_count"] >= 1:
    modifier = +15
elif known["kol_count"] >= 2:
    modifier = +10
else:
    modifier = 0
```

---

### Beat Signal Modifiers

**Social Velocity (applied to beat_adjusted):**
```python
if momentum == "ACCELERATING":
    modifier = +15
elif momentum == "STABLE":
    modifier = 0
elif momentum == "DECELERATING":
    modifier = -15
else:  # INSUFFICIENT_DATA
    modifier = 0
```

**LunarCrush AltRank (applied to beat_adjusted):**
```python
if lc_found and alt_rank <= 10:
    modifier = +10   # top 10 social momentum globally
elif lc_found and alt_rank <= 50:
    modifier = +5    # top 50 notable
else:
    modifier = 0     # not tracked yet (new token) — no penalty
```

---

### Adjusted Score Calculation

```python
spider_adjusted = spider_base + vol_liq_mod + bundle_mod + holder_mod + known_mod
beat_adjusted   = beat_base   + velocity_mod + lc_mod

# Clamp to 0-100
spider_adjusted = max(0, min(100, spider_adjusted))
beat_adjusted   = max(0, min(100, beat_adjusted))
```

---

### HENRY at 99K — Score Walkthrough (Benchmark Verification)

```
Spider base:       20   (bundles ACTIVE at scan time → FRESH/STALE boundary)
+ vol/liq HOT:    +20   (16.2x)
+ bundle STALE:   +15   (exited 2 mins ago, 663Ny72 already buying)
+ holder MIXED_BULLISH: +10  (2 buying, 2 holding, 1 selling)
+ smart_money ×1: +15   (663Ny72 in database)
= spider_adjusted: 80   → HIGH (but bundles active at exact scan time edge case)

Beat base:         70   (risk: HIGH)
+ ACCELERATING:  +15   (+105% mentions accelerating at scan)
+ AltRank 50:     +5   (notable tier at time of scan)
= beat_adjusted:   90   → HIGH

Matrix: spider_adjusted=80, beat_adjusted=90 → CONVICTION... but wait.

Correction: At exact 03:20 scan, Spider's raw signal shows ACTIVE bundles (they exited
T+0 to T+7, spider scanned at T+2). bundle_status = FRESH, not STALE.

Spider base:       20   (FRESH bundles)
+ vol/liq HOT:    +20
+ bundle FRESH:    +0
+ holder MIXED_BULLISH: +10  (organic wallets already starting to buy)
+ smart_money ×1: +15
= spider_adjusted: 65   → MED

Beat: same as above = 90

Matrix: spider_adjusted=65 (below 70 threshold), beat_adjusted=90 → LOTTERY ✓
```

This confirms the weighting produces LOTTERY on HENRY at 99K. If you change weights and this no longer holds, the weights are wrong.

---

## PART 7: THE DECISION MATRIX

### Hard Overrides (Execute Before Matrix)

These fire before any score comparison. Order matters — check them in this sequence:

**Override 1 — Dead Chart:**
```python
if vol_liq_class == "DEAD" or vol_liq_ratio < 2:
    return verdict("SKIP", "Dead chart — vol/liq below 2x, no organic demand", confidence="HIGH")
```

**Override 2 — Insider Wallet:**
```python
if known_wallets["insider_count"] >= 1:
    return verdict("CONVICTION", "Known insider wallet accumulating", confidence="HIGH")
```

**Override 3 — Coordinated Distribution:**
```python
if holder_behavior == "DISTRIBUTION":
    return verdict("SKIP", "All top holders selling — coordinated dump in progress", confidence="HIGH")
```

**Override 4 — No Liquidity Pool:**
```python
if vol_liq_class == "NO POOL":
    return verdict("SKIP", "No liquidity pool detected — not tradeable", confidence="HIGH")
```

---

### Standard Decision Matrix (After Overrides)

```
spider_adjusted threshold:  < 60 = LOW, ≥ 60 and < 70 = MED, ≥ 70 = HIGH
beat_adjusted threshold:    < 60 = LOW, ≥ 60 and < 70 = MED, ≥ 70 = HIGH
```

| Spider | Beat | Additional condition | Verdict |
|--------|------|----------------------|---------|
| LOW | LOW | any | SKIP |
| LOW | HIGH | vol_liq ≥ 16 AND bundle in (STALE, CLEARED) | LOTTERY |
| LOW | HIGH | vol_liq ≥ 8 AND velocity ACCELERATING | LOTTERY |
| LOW | HIGH | neither condition met | SKIP |
| MED | HIGH | vol_liq ≥ 8 | LOTTERY |
| MED | HIGH | vol_liq < 8 | SKIP |
| HIGH | LOW | any | ACCUMULATE |
| HIGH | MED | any | ACCUMULATE |
| HIGH | HIGH | any | CONVICTION |
| any | any | (override fired) | per override |

---

### Verdict Definitions

**SKIP**
No position. Do not enter. Not a judgment about the token's future — a judgment that the current signal profile does not justify risk.

```json
{
  "action": "NO_POSITION",
  "size_pct": 0,
  "stop": null,
  "time_box_minutes": null,
  "recheck_conditions": []
}
```

**LOTTERY**
High-risk setup with identifiable demand signal. The on-chain picture has problems. The social and demand picture is hot. Small size preserves optionality while limiting max loss.

```json
{
  "action": "SMALL_SPECULATIVE_ENTRY",
  "size_pct": 0.5,
  "stop": "BELOW_LAUNCH_PRICE",
  "time_box_minutes": 10,
  "recheck_conditions": [
    "Vol/Liq drops below 8x → EXIT",
    "Social velocity reverses to DECELERATING → EXIT",
    "Top holders start selling (re-run deep scan) → EXIT",
    "If still holding after 10 mins, re-evaluate — not auto-hold"
  ],
  "notes": "High-risk setup. This is NOT a conviction play. Exit on any trigger."
}
```

**ACCUMULATE**
Clean on-chain setup. Social has not ignited yet. Patient accumulation with room for the thesis to develop. This is not urgent.

```json
{
  "action": "MEDIUM_ACCUMULATION",
  "size_pct": 2.0,
  "stop": "15%_BELOW_ENTRY",
  "time_box_minutes": 60,
  "recheck_conditions": [
    "Wait for social momentum to reach HIGH — upgrade to CONVICTION if it does",
    "Monitor holder behavior — if turns MIXED_BEARISH, reduce size by half",
    "If vol/liq drops below 4x, thesis weakening — exit or tighten stop"
  ],
  "notes": "On-chain is solid. Waiting for social catalyst. Do not chase."
}
```

**CONVICTION**
Both signals aligned, or insider detected. Maximum position. Let it run. Trail the stop.

```json
{
  "action": "FULL_SIZE_ENTRY",
  "size_pct": 5.0,
  "stop_type": "TRAILING_20%",
  "time_box_minutes": null,
  "recheck_conditions": [
    "Trail stop 20% from peak — do not move stop down",
    "If holder behavior turns DISTRIBUTION → immediate exit regardless of stop",
    "If velocity turns DECELERATING + holder behavior MIXED_BEARISH → tighten to 10% trailing"
  ],
  "notes": "Maximum confidence. Let the winner run. The stop manages the exit."
}
```

---

## PART 8: THE FULL OUTPUT SCHEMA

```json
{
  "timestamp": "ISO8601",
  "token_address": "string",
  "token_name": "string",
  "symbol": "string",

  "verdict": {
    "classification": "SKIP | LOTTERY | ACCUMULATE | CONVICTION",
    "confidence": "HIGH | MEDIUM | LOW",
    "reason": "string — one sentence explaining why this verdict"
  },

  "trade_parameters": {
    "action": "string",
    "size_pct": "float",
    "stop": "string | null",
    "time_box_minutes": "int | null",
    "recheck_conditions": ["array of strings"],
    "notes": "string"
  },

  "signal_summary": {
    "spider_base": "LOW | MED | HIGH",
    "spider_adjusted": "int 0-100",
    "beat_base": "LOW | MED | HIGH",
    "beat_adjusted": "int 0-100",

    "vol_liq_ratio": "float",
    "vol_liq_class": "HOT | ACTIVE | WEAK | DEAD | NO POOL",
    "bundle_status": "ACTIVE | FRESH | STALE | CLEARED | null",
    "bundle_delta_mins": "float | null",
    "holder_behavior": "string",
    "known_wallets_count": "int",
    "known_wallets_smart_money": "int",
    "known_wallets_insider": "int",
    "social_velocity": "ACCELERATING | STABLE | DECELERATING | INSUFFICIENT_DATA",
    "lunarcrush_alt_rank": "int | null",
    "telegram_quality": "HIGH | MODERATE | LOW | NO_DATA"
  },

  "key_factors": [
    "array of strings — top 3-5 signals that drove the verdict"
  ],

  "warnings": [
    "array of strings — active risks that remain even with a positive verdict"
  ],

  "override_fired": "string | null — name of override if one triggered, else null"
}
```

---

## PART 9: VALIDATION TEST CASES [REQUIRED GATE]

All 5 tests must pass before Spotlight touches live capital. These are not optional.

### Test 1: HENRY at 99K — Must Output LOTTERY

```
Spider inputs:
  vol_liq_ratio: 16.2, vol_liq_class: HOT
  bundle_status: FRESH, bundle_delta_mins: 2.0
  top_holder_behavior: MIXED_BULLISH (buying:2, holding:2, selling:1)
  known_wallets: smart_money_count=1, insider_count=0

Beat inputs:
  risk: HIGH
  velocity_analysis.overall_momentum: ACCELERATING
  lunarcrush.found: true, alt_rank: 50, sentiment_class: BULLISH
  telegram_quality.overall_quality: MODERATE

Expected:
  verdict.classification: LOTTERY
  trade_parameters.size_pct: 0.5
  trade_parameters.stop: BELOW_LAUNCH_PRICE
  override_fired: null

FAIL condition: outputs SKIP
```

---

### Test 2: Clean Launch, No Social — Must Output ACCUMULATE

```
Spider inputs:
  vol_liq_ratio: 12.0, vol_liq_class: ACTIVE
  bundle_status: null
  top_holder_behavior: CONVICTION (buying:3, holding:2, selling:0)
  known_wallets: smart_money_count=2, insider_count=0

Beat inputs:
  risk: LOW
  velocity_analysis.overall_momentum: INSUFFICIENT_DATA
  lunarcrush.found: false
  telegram_quality.overall_quality: NO_DATA

Expected:
  verdict.classification: ACCUMULATE
  trade_parameters.size_pct: 2.0
  override_fired: null

FAIL condition: outputs SKIP or CONVICTION
```

---

### Test 3: Dead Chart — Must Output SKIP Regardless of Social

```
Spider inputs:
  vol_liq_ratio: 0.8, vol_liq_class: DEAD
  bundle_status: ACTIVE
  top_holder_behavior: NEUTRAL

Beat inputs:
  risk: HIGH
  velocity_analysis.overall_momentum: ACCELERATING
  lunarcrush.found: true, alt_rank: 15

Expected:
  verdict.classification: SKIP
  override_fired: "DEAD_CHART"

FAIL condition: outputs anything except SKIP
```

---

### Test 4: Coordinated Dump — Must Output SKIP Regardless of Other Signals

```
Spider inputs:
  vol_liq_ratio: 9.0, vol_liq_class: ACTIVE
  bundle_status: STALE
  top_holder_behavior: DISTRIBUTION (buying:0, holding:0, selling:5)
  known_wallets: smart_money_count=1

Beat inputs:
  risk: MEDIUM
  velocity_analysis.overall_momentum: STABLE

Expected:
  verdict.classification: SKIP
  override_fired: "DISTRIBUTION"

FAIL condition: outputs anything except SKIP
```

---

### Test 5: Insider Wallet Present — Must Output CONVICTION Regardless of Other Signals

```
Spider inputs:
  vol_liq_ratio: 5.0, vol_liq_class: WEAK
  bundle_status: FRESH
  top_holder_behavior: NEUTRAL
  known_wallets: insider_count=1, smart_money_count=0

Beat inputs:
  risk: MEDIUM
  velocity_analysis.overall_momentum: STABLE

Expected:
  verdict.classification: CONVICTION
  override_fired: "INSIDER_DETECTED"

FAIL condition: outputs anything except CONVICTION
```

---

### Test 6 (Stretch): Both Signals Aligned, Hot Chart — Must Output CONVICTION

```
Spider inputs:
  vol_liq_ratio: 22.0, vol_liq_class: HOT
  bundle_status: CLEARED
  top_holder_behavior: CONVICTION (buying:4, holding:1, selling:0)
  known_wallets: smart_money_count=3

Beat inputs:
  risk: HIGH
  velocity_analysis.overall_momentum: ACCELERATING
  lunarcrush.found: true, alt_rank: 8

Expected:
  verdict.classification: CONVICTION
  trade_parameters.size_pct: 5.0
  override_fired: null

FAIL condition: outputs anything below CONVICTION
```

---

## PART 10: FILE STRUCTURE

```
Spotlight/
├── spotlight.py          # HTTP server, request routing, analyze_token() orchestrator
├── signal_weights.py     # calculate_adjusted_scores() + all modifier functions
├── decision_matrix.py    # apply_decision_matrix() + generate_trade_parameters()
├── api_clients.py        # call_spider(), call_beat(), error handling, timeouts
├── output_formatter.py   # format_output() → final JSON schema
└── config.py             # all thresholds, ports, URLs, constants
```

### Module Responsibilities (Do Not Cross These Lines)

`config.py` — constants only. No logic. No functions. Just values.

`api_clients.py` — HTTP calls to Spider and Beat only. No weighting logic. No decisions. Handles timeouts, retries, and graceful degradation when a backend is unavailable.

`signal_weights.py` — pure functions that take data, return modifier values. No HTTP calls. No decisions. No side effects.

`decision_matrix.py` — pure functions that take adjusted scores and raw data, apply overrides, return verdict + trade parameters. No HTTP calls. No weighting calculations.

`output_formatter.py` — takes verdict, trade params, spider data, beat data; formats into the final JSON schema. No decisions.

`spotlight.py` — orchestrator only. Calls the other modules in sequence. Handles cache. Handles HTTP server. Does not contain weighting or decision logic itself.

---

## PART 11: BUILD SEQUENCE

### Phase 1: Standalone Script (Before Any Integration)

**Goal:** Feed HENRY data manually, get LOTTERY out. Nothing touches Spider or Beat yet.

Step 1 — Create `config.py` with all constants from Part 3.
Step 2 — Create `signal_weights.py` with all modifier functions from Part 6.
Step 3 — Create `decision_matrix.py` with overrides and matrix from Part 7.
Step 4 — Create `output_formatter.py` with schema from Part 8.
Step 5 — Create a test runner script that hardcodes HENRY's inputs and calls the weighting + matrix functions directly (no HTTP).
Step 6 — Run test cases from Part 9. All must pass before proceeding.

**Validation gate:** All 6 test cases pass. Do not proceed to integration until this is done.

---

### Phase 2: Spider Integration (The Enriched Endpoint)

**Goal:** Add the two new on-chain checks to Spider's backend and expose `/analyze/deep/enrich`.

This is work inside Spider's Python backend, not inside Spotlight. Spider must be extended before Spotlight can call real data.

Step 1 — Add Check 1 (Temporal Bundle Analysis) to Spider backend.
Step 2 — Add Check 2 (Live Top-5 Holder Behavior) to Spider backend.
Step 3 — Add vol/liq ratio calculation to Spider backend (if not already present).
Step 4 — Create `/analyze/deep/enrich` endpoint that returns the full schema from Part 5.
Step 5 — Validate enriched endpoint output manually against HENRY CA.

**Validation gate:** Enriched endpoint returns all required fields. bundle_status, bundle_delta_mins, top_holder_behavior all populated.

---

### Phase 3: Full Spotlight Server

**Goal:** Spotlight running as an HTTP server, calling real Spider and Beat data.

Step 1 — Create `api_clients.py` with call_spider() and call_beat().
Step 2 — Create `spotlight.py` HTTP server wiring everything together.
Step 3 — Test `/health` endpoint — must report both Spider and Beat availability.
Step 4 — Test `/analyze` with HENRY CA — must return LOTTERY verdict matching Part 9 Test 1.
Step 5 — Test with a live token to confirm end-to-end latency < 2 seconds.

**Validation gate:** HENRY CA returns LOTTERY. Latency < 2 seconds. No crashes when Spider or Beat is offline.

---

### Phase 4: Extension UI

**Goal:** Third panel in the Chrome extension below Spider's existing panels.

Step 1 — Add Spotlight panel to content.js.
Step 2 — Panel layout: Spider Score | Beat Score | Spotlight Verdict (top row), then trade parameters.
Step 3 — Color coding: SKIP=red, LOTTERY=orange, ACCUMULATE=blue, CONVICTION=green.
Step 4 — Display key_factors list below verdict.
Step 5 — Display warnings list if any.
Step 6 — Live testing on several tokens.

**Validation gate:** Panel renders without breaking Spider's existing panels. Verdict and parameters legible.

---

## PART 12: GRACEFUL DEGRADATION

Spotlight must not crash when Spider or Beat is unavailable. These are the fallback rules:

**If Spider is down:**
```
verdict: SKIP
reason: "Spider unavailable — on-chain data missing, cannot assess risk"
confidence: LOW
notes: "Restart Spider backend before trading"
```

**If Beat is down:**
```
verdict based on Spider data only
beat_adjusted: 50 (treat as MEDIUM — unknown social state)
warnings: ["Beat unavailable — social layer not included in this verdict"]
confidence: LOW
```

**If both are down:**
```
verdict: SKIP
reason: "All data sources unavailable"
```

**If enriched Spider endpoint unavailable (falls back to standard deep scan):**
```
bundle_status: null (treated as no-bundles = +10 to spider_adjusted)
top_holder_behavior: null (no modifier)
warnings: ["Enriched endpoint unavailable — temporal bundle analysis and holder behavior checks skipped"]
confidence: reduced by one tier
```

**If a required field is missing from Spider or Beat response:**
- Treat as the neutral value for that signal (modifier = 0)
- Add to warnings list: "Signal X unavailable — treated as neutral"
- Do not crash

---

## PART 13: WHAT SPOTLIGHT REALISTICALLY DELIVERS

This table exists to prevent scope creep and over-promising. When someone asks Spotlight to do something, check it against this table.

| Question | What Spotlight Actually Delivers | What It Does NOT Deliver |
|---|---|---|
| Should I buy this? | One of 4 verdicts + specific size, stop, and time parameters | A guarantee of profit or a predicted outcome |
| When will it dump? | Early warning within 60 seconds IF top holders begin selling (requires re-scan trigger) | Advance prediction before selling starts |
| Is this the bottom? | Floor strength classification from holder behavior + vol/liq + social (within 60 seconds of re-scan) | Price level where bounce will occur |
| How high will it go? | Whether continuation conditions are present or absent — updated every scan cycle | A price target |
| Is this a rug? | Distribution classification if all top holders are selling simultaneously | Intent or confirmation of rug before it happens |
| Should I hold? | Whether the original verdict conditions still hold on re-scan | A hold recommendation divorced from current state |

---

## PART 14: PHASE 2 PREVIEW (DO NOT BUILD IN PHASE 1)

Phase 2 items are listed here only so they are not forgotten and not scope-crept into Phase 1.

**SQLite outcome log** — every scan logged with: timestamp, token_ca, mc_at_scan, verdict, known_wallets_found, peak_mc_24h (filled hourly by background process). Stored at `Spotlight/data/spotlight_log.db`.

**Wallet database integration** — load 700 curated wallets (KOLs, smart money, insiders, bundlers, bots) from CSV into Python dict at startup. Zero latency lookup. Every holder cross-reference against this dict. Single known smart money wallet flips a SKIP toward LOTTERY or ACCUMULATE.

**Proactive wallet monitoring** — instead of waiting for a token to appear in feed and scanning reactively, watch the 700 known wallets for new first-buy activity. Alert fires the moment a known smart money wallet enters an unknown token. This is the most powerful use of the wallet database.

**KOL wallet → Beat pipeline** — link known KOL wallet addresses to their Twitter accounts. If KOL wallet accumulates → Beat monitors their Twitter. You're positioned before their audience arrives.

These items require Phase 1 to be validated and running. They also require the SQLite data to exist before wallet reputation scoring can be meaningful. Do not rush Phase 2.

---

## PART 15: OPEN QUESTIONS [NOT LOCKED]

These remain unresolved. Do not build against assumptions on these until they are explicitly answered.

1. **Beat interface for velocity rate of change** — Beat currently reports `overall_momentum` as ACCELERATING/STABLE/DECELERATING. Does it also expose the raw rate (e.g., mentions went from 12/min to 28/min = +133%)? If yes, Spotlight can use the raw rate for finer-grained weighting.

2. **Wallet CSV format** — Column names, how categories are tagged (freeform strings vs standardized labels), whether any records include associated Twitter handles.

3. **Retest trigger threshold** — Is 40% price drop in 60 minutes the right threshold to trigger a floor strength re-scan? Or should it be configurable per verdict type (tighter for LOTTERY, wider for CONVICTION)?

4. **LOTTERY size cap** — Is 0.5% portfolio fixed, or should it scale with vol/liq? E.g., 0.3% at 8-12x, 0.5% at 12-16x, 0.8% at 16x+? Higher demand signal = slightly more size.

5. **Launchpad registry** — Uxento tokens inherit community attention. Pump.fun tokens have different behavioral baselines. What other launchpads need priors? What are the baseline prior adjustments per launchpad? This affects the spider_base calculation for tokens with no bundles.

6. **Which 5 historical tokens to use as validation test set beyond HENRY** — Need 4 more: ideally 1 clean launch that worked, 1 clean launch that dumped and died, 1 farm dump that recovered (similar to HENRY), 1 farm dump that did not recover. Actual CAs needed.

7. **Archetype detection** — The SPOTLIGHT_designsv1 doc outlines 5 archetypes (Narrative Attachment, Slow Accumulation, Revival Token, Liquidity Magnet, Community Launch). Phase 1 spec uses a single signal weight set for all. Should archetype detection run before weighting? If yes, when and how does it affect weights?

---

## PART 16: CONSTRAINTS THAT MUST SURVIVE EVERY SESSION

These are the things most likely to be forgotten or violated under time pressure. Read them before every session that writes Spotlight code.

**Do not modify Spider or Beat's core logic.** Spider is excellent at risk detection. Beat is excellent at social forensics. Spotlight adds the synthesis layer. Changes to Spider's existing detection logic are out of scope for Spotlight sessions.

**Do not put AI in the verdict path.** An LLM can generate the `notes` field or a narrative summary. It cannot produce `size_pct`, `verdict.classification`, modifier values, or any numerical output. Those are Python, always.

**Do not add features before validating existing ones.** The test gate in Part 9 is not optional. The pattern of "let's just add one more signal before testing" is how systems grow complex and undebuggable. Build, test, validate, then extend.

**Do not change the output schema without updating this document.** If the output schema drifts from Part 8, the extension UI breaks and future sessions have no contract to work from.

**Do not skip error handling.** Spotlight is downstream of two separate backends. Both can go down. Every API call needs a try/except with a defined fallback behavior per Part 12. A Spotlight that crashes when Spider times out is worse than no Spotlight.

---

_This is the bible. Start every Spotlight session by reading it._
_Update it when locked decisions change. Never let the code drift from this document._
