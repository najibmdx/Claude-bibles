# SKOPE — Project Bible
*Last updated: May 2026 | Paste this at the start of any Skope session*

---

## What Skope Is

A browser extension (same host as Spider) that performs **forensic/anatomical analysis** on Solana memecoin tokens. While Spider asks "is it dumping right now?", Skope asks "what IS this token and who built it?"

Skope tells you the token's identity. Spider tells you its behavior. They are used simultaneously on the same token. That's the design.

**Status: Approaching build phase. Architecture decisions still open. No code written yet.**

---

## The Core Distinction (Never Re-Raise the Overlap Question)

This was stress-tested and resolved in April 2026. Spider and Skope are NOT duplicates.

| Dimension | Spider | Skope |
|---|---|---|
| Core question | Is it dumping? | What is it? |
| Intelligence type | Momentum and exit pressure | Anatomy and identity |
| Data source | Price/flow signals from token page | Dev wallet, holders, traders → proprietary DB |
| Output | Dump risk score + trader commentary | Archetype + insider recognition |
| Temporal focus | Live, right now | Structural (with live monitoring layer) |
| Proprietary edge | Mistral prompt calibration | 952-wallet cross-reference universe |

The weapon stack: **Skope tells you what you're holding. Spider tells you whether to hold it.**

---

## Proprietary Edge

**952-wallet cross-reference universe** — this is Skope's core IP. A proprietary database of known wallets (operators, cabals, KOLs, insider networks, serial deployers) that Skope cross-references against every token's holder list and transaction graph.

This database is NOT shared with Spider. Spider uses Arkham for entity enrichment; Skope uses this proprietary DB for archetype classification.

---

## Skope's Output Archetypes

Known archetypes from the design phase:
- **CABAL RUN** — coordinated insider group, pre-loaded wallets
- **EXIT TRAP** — designed to look organic, built for exit liquidity
- **FARM OP** — farming rewards/airdrops, not a genuine community token
- *(Others TBD during build phase)*

---

## Project Location

Lives in the `Tide-Skope-Beat` project folder. Skope is a separate concern from Spider (different folder, different codebase, different team context when scaled).

---

## Architecture Decisions — Open vs Settled

**Settled:**
- Skope is a browser extension (same extension runtime as Spider, possibly same manifest)
- Primary data source: proprietary wallet DB + on-chain graph traversal
- Output: archetype verdict + known wallet recognition + dev identity
- Skope and Spider are used simultaneously — not sequential
- 952-wallet DB is Skope's exclusive asset

**Still Open (decide at build phase start):**
- Does Skope share the same Python backend port as Spider, or run on its own port?
- Does Skope share `background.js` with Spider (one extension) or is it a separate extension?
- What is the dev identity resolution pipeline? (On-chain graph traversal → known DB → Arkham fallback?)
- How are new wallets added to the 952-wallet DB? Manual curation? Automated detection?
- What's the minimum viable Skope v1 — archetype only, or archetype + dev identity + live monitoring?

---

## The One Risk to Watch

Skope's Layer 2 includes **live monitoring of known wallets** (dev sell activity, insider position changes). Spider also watches live token dynamics. If Skope's live monitoring expands into generic flow analysis, they start producing duplicate signals (e.g., both flagging "dev is selling").

**Guard rail:** Skope's live monitoring must remain anatomical — watching *known specific wallets* only. Spider handles aggregate flow. Keep the boundary clean.

---

## Relationship to Other Projects

- **Tide** — order flow/liquidity monitoring. Lives in same `Tide-Skope-Beat` folder. Stated design intent: Skope + Tide used together as a combined intelligence layer.
- **Beat** — social sentiment (Twitter/Telegram/Discord). Same folder. Feeds KOL context into Skope's wallet recognition layer (future integration, not built).
- **Arkham** — Skope may use Arkham as a fallback for wallets not in the proprietary 952 DB. Architecture not confirmed.

---

## What To Do at Build Phase Start

Before writing any code:
1. Resolve the open architecture decisions above (especially: one extension or two? one backend or two?)
2. Define Skope v1 scope precisely — archetype verdict only? Or archetype + dev identity in v1?
3. Define the 952-wallet DB format and query interface
4. Confirm the extension injection points (which trading platforms, same as Spider or expanded?)
5. Design the panel UI separately from Spider's panel — Skope needs its own visual identity

---

## Session Start Checklist

Before any Skope work:
1. This bible is loaded
2. Spider bible is also loaded (understand the boundary)
3. Confirm with Jeeps: "Are we designing architecture or building code today?"
4. Architecture decisions FIRST — no code until open questions above are resolved
