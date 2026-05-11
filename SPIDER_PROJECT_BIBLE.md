# SPIDER — Project Bible
*Last updated: May 2026 | Paste this at the start of any Spider session*

---

## What Spider Is

A Brave/Chrome browser extension + Python backend that analyzes Solana memecoin tokens in real-time and outputs an AI trader's take on dump risk. It runs on top of Bullx, Photon, GMGN, Axiom, and Dexscreener. The user clicks Scan, Spider fetches on-chain data, scores it, and Mistral writes a trader-style commentary. The user makes the final call.

**Not a bot. Not auto-executing. Spider investigates. Jeeps decides.**

---

## File Structure

```
C:\iSight\MiniCowScanner\Duck-GooseMerger\WalletScanner\Monkey\Spider\
├── spider.py          ← Python backend, port 8765
├── content.js         ← Extension frontend, injected into trading platforms
├── background.js      ← Service worker (Brave PNA proxy — PERMANENT, NEVER REMOVE)
├── manifest.json      ← MV3 manifest
├── arkham.py          ← Arkham entity enrichment module (shared with Beat)
└── .env               ← MISTRAL_API_KEY, HELIUS_API_KEY, ARKHAM_API_KEY
```

---

## Architecture — Settled Decisions

### Two-Axis Scoring System
- **Risk Score** (0–100): dump exposure based on supply held by threats
- **Demand Score**: buy/sell pressure ratio from 1h volume
- **MODE badge**: derived from both axes → LOTTERY / ACCUMULATE / DUMP RISK / DEAD etc.

### Scoring Model: DUMP-IMPACT-BASED (never go back to archetype-based)
```
dump_exposure = whale_holdings + coordinated_cluster_holdings
               (with double-count guard if whale is inside operator cluster)
+ farming dump modifier: +15pts
+ operator coordination modifier: +10pts
× MC amplifier (thin liquidity = higher risk multiplier)
```
**Why we changed:** The old archetype model awarded points per threat type regardless of how much supply they held. A whale holding 2% scored the same as one holding 40%. The dump-impact model rewards based on actual selling firepower. Never revert.

### Mistral Prompt Structure: TOKEN INTELLIGENCE REPORT
The prompt gives Mistral raw data across four sections — market state, holder distribution, early activity analysis, pattern recognition — then asks it to respond as an experienced Solana memecoin trader. It outputs trading commentary, NOT a risk algorithm summary.

**Why we changed:** The old prompt gave Mistral pre-digested conclusions. The new prompt gives raw data and lets Mistral do the analysis. Result: sharper, more contextual commentary. Tested on a real post-rug token ($19k MC, collapsed from $3.5M) — output was accurate and trader-calibrated.

### Helius is the Only RPC
Never suggest alternatives. Paid tier. `getTokenLargestAccounts` is the fallback for Token-2022.

### 3-Cycle Scan Then Dormant
Spider scans in cycles, not continuously. No auto-scan. User initiates. Do not suggest changing this.

### Python Owns All Numbers
LLM writes narrative only. Never put scoring logic inside the Mistral prompt. Python calculates; Mistral interprets.

---

## Critical Infrastructure: The Brave Fix

**Background.js is a service worker that proxies all fetch calls from content.js to the Python backend.**

Why it exists: Brave version 147+ enforces Private Network Access policy — content scripts on HTTPS pages cannot reach HTTP localhost directly. This broke Spider after a Brave auto-update in April 2026. The fix: background.js intercepts `chrome.runtime.sendMessage` from content.js and makes the actual fetch to `127.0.0.1:8765`.

**This is a PERMANENT architectural component. Never remove it. Never "simplify" it away. Never suggest going back to direct fetch calls.**

Files involved: `background.js` (new file), `manifest.json` (service_worker entry added), `content.js` (all fetch calls replaced with `chrome.runtime.sendMessage`).

---

## Things We Tried That Failed

| Attempt | Why We Tried It | Why It Failed |
|---|---|---|
| SSL self-signed cert (HTTPS backend) | Bypass PNA by matching protocols | OpenSSL not installed on Windows; Python cryptography lib cert not trusted by Brave without manual trust click |
| Browser flags to disable PNA | Disable the restriction directly | Removed from Brave 147+; no longer available |
| Manifest `private_network_access` permissions | Spec-based whitelisting | Not implemented in Brave at time of attempt |
| Archetype-based scoring | Points per threat type | Didn't reflect actual supply exposure — 2% whale scored same as 40% whale |
| Pre-digested Mistral prompt | Simpler for the LLM | Output was mechanical; missed contextual nuance |
| GLM-5.1 as Mistral replacement | Potentially sharper model | No accessible API confirmed at time of evaluation; Mistral performing well — deferred indefinitely |

---

## Token-2022 / Pre-Migration Behavior

**Pre-migration (bonding curve) tokens:**
- All top accounts are vault addresses → old code filtered them out → crash: "Could not resolve holders"
- Fix: falls back to early buyer wallets from transaction history
- `locked_pct` set to 100% (supply is locked in bonding curve)
- "Clean Distribution" panel suppressed (meaningless pre-migration)
- `pre_migration: true` flag added to payload for UI handling
- Mistral is told the token is pre-migration so commentary is contextually accurate

**Token-2022 program:**
- Program ID: `TokenzQdBNbLqP5VEo7ml8epvgs3ZFZNa4KxK3tpS3`
- Helius blocks `getProgramAccounts` on Token-2022 → returns None
- Always query both programs; fallback to `getTokenLargestAccounts` if both return 0
- `getTokenLargestAccounts` capped at top 20 — by design, not a bug
- Known wallet detection limited to top 20 for Token-2022 tokens

---

## Arkham Integration

`arkham.py` is a shared enrichment module (used by Spider and Beat). It batch-queries wallet addresses against Arkham's entity database.

**What it adds to Spider:**
- Arkham entity hits inject into `known_hits` array → render through existing Known Wallets panel
- `kw_sm_count` and `kw_kol_count` counters update automatically → demand score adjusts
- Parallel execution: Arkham + Mistral run in `ThreadPoolExecutor` → zero added latency
- Graceful fail-open: if Arkham is unavailable or key missing, Spider runs exactly as before

**Honest limitation:** Arkham's Solana coverage is thinner than EVM. Best signal on KOLs with cross-chain presence and exchange-connected wallets.

---

## UI Design System

**Color palette:**
- Base panel: `#0b0f1a`, card backgrounds: `#0d1220`, borders: `#1a2538`
- Green: `#20c97a`, Violet: `#c4b5fd`, Amber: `#f5a623`

**Brightness hierarchy (do not break):**
- Primary (labels, scores, wallet names): `#f0f5ff`
- Secondary (descriptions): `#8aaac8`
- Tertiary (section headers): `#7a9ab8` — intentionally dim

**Demand color semantics (settled — never change meaning):**
- HOT `#E24B4A` = extreme heat = danger
- ACTIVE `#EF9F27` = elevated = caution
- WEAK `#1D9E75` = low concern = safe
- DEAD `#888888` = no activity
- FAST `#E24B4A` = fast bonding curve = danger
- SLOW `#1D9E75` = safe
- STALLED `#888888` = dead

**Merged Demand Card (v2.0+):**
- Replaced the old `sp-demand-wrap` bar + separate Demand signal card
- Demand panel filtered from `data.panels` loop via `p.id !== 'demand'`
- Bar shows buy pressure ratio: `buys1h / (buys1h + sells1h)`
- Sub-cells: Vol / Liq / Buys / Sells

---

## Open Questions / Next Development Areas

- **Post-mortem detection**: When a token has already rugged (e.g., $19k MC after collapsing from $3.5M), the Risk Score correctly shows low (no more supply to dump) but the UI could confuse this with "safe." We decided to stress-test more before adding explicit post-mortem detection. This is open — not forgotten.
- **Skope integration**: Skope (anatomical/forensic) will eventually feed data into Spider's panel alongside Spider's flow-based signals. Architecture not yet designed.
- **Beat integration**: Arkham already integrated. Beat's social signals not yet piped into Spider.

---

## Session Start Checklist

Before any Spider work:
1. Load the `spider-session` skill (rules and constraints for how to work)
2. This bible is already loaded (knowledge context)
3. Ask Jeeps: "Which files do you want to work on?" before touching anything
4. If Jeeps hasn't uploaded current files, ask for them — never assume the files in context are current

---

## What Spider Is NOT

- Not Skope (which does forensic/anatomical analysis — dev identity, insider network, archetype)
- Not Beat (which does social sentiment — Twitter/Telegram/Discord)
- Not a trading bot — it does not execute trades
- Not a continuous monitor — it scans on demand
