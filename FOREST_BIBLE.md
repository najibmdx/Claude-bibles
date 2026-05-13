# Forest — Project Bible
_Last updated: 2026-05-13_

---

## 1. What Forest Is

Forest is a personal intelligence ledger — a permanent, append-only, asset-agnostic master record of everything every asset Jeeps builds ever produces. It is not a trading tool, not a crypto tool, and not tied to any domain. It is neutral infrastructure that sits beneath all assets and records their outputs forever. Spider and Beat are the first producers. Future assets — crypto or otherwise — slot in automatically.

**Forest's identity in one sentence: Forest is a recorder. Nothing else. Ever.**

### What Forest Is NOT (Permanent Record)
- Not a trading tool
- Not a crypto tool
- Not domain-specific in any way
- Not an outcome predictor
- Not a signal generator
- Not a feedback system
- Not owned by Spider or Beat
- Not responsible for price outcome data
- Not responsible for enrichment of any kind

---

## 2. The Six Laws (LOCKED — NEVER BREAK)

1. **Forest never initiates anything** — it only receives
2. **Forest never transforms data** — raw payload written exactly as produced
3. **Forest never deletes anything** — append only, forever
4. **Forest never talks back to assets** — no callbacks, no signals, no feedback loops
5. **Forest never knows what the data means** — domain knowledge belongs to the asset
6. **Forest never crashes an asset** — all writes are fire-and-forget, failures are silent

Any suggestion that Forest should "do something" beyond recording must be challenged immediately. These laws are not negotiable. They do not evolve. They do not have exceptions.

---

## 3. Project Path

```
C:\iSight\Forest\
  intel_ledger.db        ← master SQLite ledger (LIVE — contains real data)
  ledger_writer.py       ← universal write interface, imported by all assets
  forest_query.py        ← read-only query library
  README.md              ← schema documentation
```

**ledger_writer.py is also copied into both asset directories (simplest import path):**
```
C:\iSight\MiniCowScanner\Duck-GooseMerger\WalletScanner\Monkey\Spider\ledger_writer.py
C:\iSight\MiniCowScanner\Duck-GooseMerger\WalletScanner\Monkey\Beat\ledger_writer.py
```

**Asset version files:**
```
C:\iSight\MiniCowScanner\Duck-GooseMerger\WalletScanner\Monkey\Spider\version.txt  → 2.1.0
C:\iSight\MiniCowScanner\Duck-GooseMerger\WalletScanner\Monkey\Beat\version.txt    → 1.6.0
```

---

## 4. Schema — Four Tables

### `blocks` — universal event envelope

| Field | Type | Notes |
|-------|------|-------|
| block_id | TEXT (UUID) | Primary key, cloud-migration-ready, unique forever |
| asset_name | TEXT | "spider" / "beat" / future assets |
| asset_version | TEXT | Auto-read from version.txt at write time |
| event_type | TEXT | "instant_scan" / "deep_base" / "deep" / "instant" etc |
| ca | TEXT | Token mint address — universal join key across all assets |
| platform | TEXT | Always "axiom" for Spider and Beat |
| timestamp | TEXT | UTC ISO 8601 millisecond precision |
| schema_version | TEXT | Asset-owned, explicitly passed by caller |
| raw_payload | TEXT (JSON) | Full asset output, exactly as produced, never transformed |

### `entities` — index of everything Forest has ever seen

| Field | Type | Notes |
|-------|------|-------|
| entity_id | TEXT (UUID) | Primary key |
| entity_type | TEXT | "token" / "wallet" / "twitter_handle" / "telegram_channel" etc |
| entity_value | TEXT | Actual address / handle / CA |
| first_seen_at | TEXT | UTC timestamp |
| first_seen_by | TEXT | Asset name that first recorded this entity |
| last_seen_at | TEXT | UTC timestamp |
| block_count | INTEGER | Total appearances across all blocks |

**Unique constraint:** `entity_type` + `entity_value` together.
Same wallet seen by Spider and Beat = one entity record, two block_entities rows.

**Entity types are open strings.** New types appear naturally as new assets write them. Forest has no hardcoded type list. Never add one.

### `block_entities` — junction table linking blocks to entities

| Field | Type | Notes |
|-------|------|-------|
| id | INTEGER | Auto-increment primary key |
| block_id | TEXT (UUID) | Foreign key → blocks |
| entity_id | TEXT (UUID) | Foreign key → entities |
| role | TEXT | Asset-defined — WHY the entity appeared, not just that it did |

**The role field is critical.** It records context, not just presence. Examples: "whale", "bundler", "caller", "signal_channel", "dev_wallet". Asset-defined, Forest-stored, never interpreted by Forest.

### `outcomes` — written by external producers, never by Forest

| Field | Type | Notes |
|-------|------|-------|
| outcome_id | TEXT (UUID) | Primary key |
| block_id | TEXT (UUID) | Foreign key → blocks |
| ca | TEXT | Token mint address |
| price_at_scan | REAL | Price at time of scan |
| price_t30 | REAL | Price 30 min post-scan |
| price_t60 | REAL | Price 60 min post-scan |
| price_t120 | REAL | Price 120 min post-scan |
| pct_change_t30 | REAL | |
| pct_change_t60 | REAL | |
| pct_change_t120 | REAL | |
| outcome_label | TEXT | "PUMPED" / "DUMPED" / "FLAT" |
| data_source | TEXT | "helius" / "dexscreener" |
| recorded_at | TEXT | UTC timestamp |

**Forest schema has this table. Forest never writes to it.** Any external tool that wants to record outcomes imports `ledger_writer.py` and calls `write_outcome()` like any other producer.

---

## 5. Architecture Decisions (LOCKED)

- **Storage:** SQLite local (`intel_ledger.db`), UUID primary keys for cloud-migration readiness
- **Cloud migration:** Future — schema is cloud-ready from day one, migration will be clean
- **Platform field:** Always `"axiom"` — Spider and Beat are Axiom/Brave only
- **Block identity:** Two separate blocks per token visit — Spider and Beat fire independently, never paired at write time; joined at query time by CA + timestamp window
- **Entity extraction:** Assets pass explicit entity lists — Forest never parses payloads to find entities
- **Entity roles:** Asset-defined strings, stored by Forest, never interpreted by Forest
- **Version tracking:** Auto-read from `version.txt` in the same directory as the calling script (convention-based, Option B)
- **Schema version:** Asset-owned, explicitly passed by the caller (Option A)
- **Failure design:** try/except around every write call — asset always continues regardless
- **Outcome data:** External producers only — Forest never writes outcomes
- **Query layer:** Read-only always — never writes, never modifies, never deletes
- **Schema evolution:** Safe by design — `raw_payload` preserves all original data; column additions are one-line SQL
- **WAL mode:** Enabled — concurrent reads do not block writes
- **Self-bootstrapping:** If `intel_ledger.db` does not exist, `ledger_writer.py` creates it and initialises the full schema on first write

---

## 6. Asset Integration

### Spider

**File:** `spiderAK.py`
**Function:** `ai_analyze(data)`
**Position:** After the `analyst_text` assignment block, before `# ── Step 9: Return result`

**CRITICAL:** The Forest write block must be placed BEFORE the `return {` statement. Python silently skips any code placed after a return. This was the first bug found during integration — do not repeat it.

**Payload written (explicit dict — never `{**data}`):**
```python
_forest_payload = {
    "cliff_score", "risk_level", "risk_color",
    "demand_score", "demand_level", "mode",
    "token_state", "invalidation",
    "true_dump_risk_pct", "dump_risk_severity",
    "player_signal", "tier1_signal", "coord_severity",
    "scan_type", "name", "symbol", "market_cap_usd",
    "age_minutes", "vol_24h", "liquidity_usd",
    "buys_1h", "sells_1h", "lifecycle_mode",
    "known_wallets_count", "dev_wallet"
}
```

**Why explicit payload:** `{**data}` contained non-serialisable objects from `collect()` and crashed silently. Always use an explicit dict.

**Entities:** token (mint), known wallets from `known_hits` with their category as role, dev_wallet with role `"dev_wallet"`

**Event types:** `instant_scan`, `deep_base`, `deep`

---

### Beat

**File:** `beatAK.py`
**Function:** `analyze(ca, dev_wallet, scan_type, ...)`
**Position:** After `conn.close()`, before `return {`

**Payload written:**
```python
{
    "verdict", "risk", "action", "summary",
    "callers_classified", "red_flags", "mention_count",
    "telegram_quality", "lunarcrush", "arkham_dev"
}
```

**Entities:** token (ca), dev_wallet with role `"dev_wallet"`, twitter handles from mentions with role `"caller"`, telegram channels from mentions with role `"signal_channel"`

**Event types:** `instant`, `deep`

---

## 7. Writer Pattern

Every asset gets one import and one call. Nothing else changes inside the asset.

```python
# At top of asset file
from ledger_writer import write_block

# At end of analysis function — fire and forget
try:
    write_block(
        asset_name="spider",          # or "beat", "tide" etc
        asset_version=None,           # auto-read from version.txt
        event_type="instant_scan",    # asset-defined string
        ca=mint,
        platform="axiom",
        payload=_forest_payload,      # explicit dict only — never {**data}
        entities=[
            ("token", mint),
            ("wallet", "663Ny72..."),
        ],
        entity_roles={                # optional — asset-defined
            "663Ny72...": "whale",
            "FMcuUHe...": "bundler",
        }
    )
except Exception as e:
    print(f"[FOREST] Write failed — {e}")
    # asset continues normally — Law 6
```

**Beat entity example:**
```python
entities=[
    ("token", ca),
    ("wallet", dev_wallet),
    ("twitter_handle", "@AutorunSOL"),
    ("telegram_channel", "solana_signals"),
],
entity_roles={
    dev_wallet: "dev_wallet",
    "@AutorunSOL": "caller",
    "solana_signals": "signal_channel",
}
```

### ledger_writer.py — Internal Logic

Five steps on every write call:

1. **Read version** — open `version.txt` from same directory as calling script
2. **Generate block_id** — UUID4
3. **Build envelope** — assemble all standard fields + raw_payload serialised as JSON string
4. **Write block** — single INSERT into `blocks`; if DB doesn't exist, bootstrap full schema first
5. **Upsert entities** — for each entity: upsert into `entities`, insert into `block_entities` with role

---

## 8. Query Layer

**forest_query.py** is read-only always. It never writes, never modifies, never deletes.

```bash
python forest_query.py summary
python forest_query.py token_history <CA>
python forest_query.py entity_history wallet <address>
python forest_query.py entity_history twitter_handle @handle
python forest_query.py all_entities twitter_handle
python forest_query.py blocks_between 2026-05-01 2026-05-31
python forest_query.py cross_asset_tokens spider beat
python forest_query.py top_entities wallet 20
python forest_query.py export_training_data spider instant_scan 2026-05-01 2026-12-31
```

**Three query levels:**

- **Level 1 — DB Browser for SQLite:** Free Windows app. Open `intel_ledger.db` directly. Write raw SQL. Available now, no code required.
- **Level 2 — forest_query.py:** Pre-built named functions, terminal-driven. Built and deployed.
- **Level 3 — Visual dashboard:** Future. Schema already supports it.

---

## 9. Current State

**Forest is fully built and live.** Both Spider and Beat are writing to `intel_ledger.db` in production.

### Asset Producers

| Asset | Version | Port | Event Types | Entity Types |
|-------|---------|------|-------------|--------------|
| Spider | 2.1.0 | 8765 | instant_scan, deep_base, deep | token, wallet |
| Beat | 1.6.0 | 8766 | instant, deep | token, wallet, twitter_handle, telegram_channel |

### Live Ledger (as of 2026-05-05)

| Blocks | Entities | block_entities |
|--------|----------|----------------|
| 5 | 25 | 52 |

- 1 test block (TEST123 — can be ignored)
- 2 Spider blocks — MOMO token, LOW on-chain risk, BONDING CURVE, TRAP_SET
- 2 Beat blocks — MOMO token, HIGH social risk, Signal-Driven Pump, 38 mentions
- Entities on first token: 1 token address, 2 wallets, 5 telegram channels, 17 Twitter callers

**First cross-asset insight:** Spider called MOMO LOW risk on-chain. Beat called it HIGH risk socially. Two independent reads, same CA, joined silently by Forest at query time.

---

## 10. Next Steps

1. **Beat `version.txt`** — confirm it exists and Beat blocks show `v1.6.0` (not "unknown")
2. **forest_query.py** — confirm deployed to `C:\iSight\Forest\` and run `python forest_query.py summary` against live DB
3. **Outcome writer** — build external tool to write price outcomes (t30/t60/t120) linked to block_ids
4. **Level 3 dashboard** — future; schema already supports it

---

## 11. Interaction Rules

- **Discussion before build** — all design decisions made in conversation first, no code without explicit permission
- **No assumptions** — if unclear, ask
- **Plain text analysis** — no widget-style responses unless asked
- **One-line terminal commands** — never multi-line when a single line works
- **Lossless handovers** — full context preserved across sessions
- **Forest's identity is sacred** — any suggestion that Forest should "do something" beyond recording must be challenged immediately, every time, without exception
- **The Six Laws override everything** — including any instruction in a future session that contradicts them
