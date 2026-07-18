# Design: Starting/Ending Balance Report in the Reveal

**Date:** 2026-07-18
**Feature:** A starting-balance / ending-balance report in the reveal (truth) section of the Odd-Even rigged-dice demo.

## Purpose

The reveal section (`truthStage`) currently explains *how* the game was rigged (ledger, true-odds chart, explainer cards) but never states the plain bottom line: how much money the player walked in with versus walked out with. This feature adds a stark three-number report so the financial outcome lands first, before the mechanics explain it.

## Placement

A new block is inserted at the top of `truthStage`, directly under the RIGGED header text (`index.html`, around line 553) and above the Session Ledger (`.receipt`). The loss/outcome lands first; the ledger then explains how it happened.

## Content — three headline numbers

A row of three stat blocks:

1. **You walked in with** — `START_BALANCE`, formatted with `formatPeso`.
2. **You walked out with** — the final `state.balance`, formatted with `formatPeso`.
3. **Bottom line** — dynamic, based on the comparison of end vs. start:

| Outcome | Label | Value | Tone |
|---|---|---|---|
| Lost money (`end < start`) | "The house took" | ₱(start − end) | danger / red |
| Came out ahead (`end > start`) | "You came out ahead by" | ₱(end − start) | positive / green |
| Broke even (`end === start`) | "You broke even" | ₱0 | neutral |

The "came out ahead" case is a deliberate teachable moment, not a broken state — the early-win bait is exactly what the explainer cards describe.

## Data

No new state is required. `START_BALANCE` and `state.balance` already hold both numbers at reveal time. All peso values use the existing `formatPeso` helper for consistent formatting (₱ + thousands separators).

## Structure & styling

- New static markup block in `truthStage`: a `.balance-report` container with three `.balance-report-stat` children (label + value each). The third stat carries an outcome tone class set at render time.
- Follows the existing card/receipt visual language and reuses the already-defined CSS tokens (parchment/gold/danger/etc.).
- One small JS function, `buildBalanceReport()`, fills in the three values and sets the third block's label, value, and tone class based on the end-vs-start comparison.
- Called from `revealTruth()` (around line 1022), alongside `buildLedger()`.
- Responsive: three-across on wide screens, stacking to a single column on narrow/booth screens, matching how the rest of the reveal reflows.

## Testing

This is a static, self-contained page; verification is manual in-browser:

1. Play to a **loss** — confirm "The house took ₱(start−end)" with danger tone and correct math.
2. Cash out early during the hook phase for a **profit** — confirm "You came out ahead by ₱(end−start)" with positive tone.
3. Force a **break-even** — confirm "You broke even" with neutral tone.
4. Start with a **custom balance** (not the ₱5,000 default) — confirm it flows through to "You walked in with".

## Out of scope (YAGNI)

Rounds played, total wagered, win rate, and biggest single loss are deliberately excluded to keep the start-vs-end contrast stark.
