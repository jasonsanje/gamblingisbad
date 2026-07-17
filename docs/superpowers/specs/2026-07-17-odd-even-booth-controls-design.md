# Odd-Even — Rename & Discreet Booth Controls

**Date:** 2026-07-17
**Status:** Approved design, pending implementation
**Scope:** `index.html`, `README.md`, `CHANGELOG.md` (single-page, no dependencies)

## Context

This is a gambling-**awareness** demo for classroom/booth use. Players guess odd/even on
two dice; the game secretly rigs the odds to teach how "fair-looking" gambling apps stack
the deck against players. The rig is driven by hidden operator ("Booth") controls and an
Instructor mode that exposes the true odds.

The operator currently starts the rig by clicking a small dot in the lower-right corner,
which opens a **visible** panel — an onlooker at a booth can see the operator reaching for
it. This spec adds discreet, no-panel triggers for two operator actions and renames the
game.

## Goals

1. Rename the game from "Lucky Parity" to **"Odd-Even"** in all user-visible surfaces.
2. Give the operator a one-shot "force this roll to win" control the audience won't notice.
3. Let the operator start the rig without opening the visible lower-right panel.

## Non-Goals

- No change to the underlying rigging math (`computeProb` / `pityBonus` / `effectiveProb`).
- No persistent "always win" mode — force-win is deliberately one-shot (see Decisions).
- No backend, build step, or new dependencies.

## Decisions (confirmed with user)

- **Triggers:** BOTH hidden keyboard shortcuts AND invisible corner tap zones, so the
  control works on keyboard booths and touchscreen booths alike.
- **Force-win behavior:** ONE-SHOT — each activation guarantees only the *next* roll, then
  reverts to the rigged odds. Pressing it every round yields an effective 100% streak while
  leaving per-round control in the operator's hands.

## Design

### 1. Rename Lucky Parity → Odd-Even

User-visible occurrences to change:

| Location | Current | New |
|---|---|---|
| `<title>` (~line 6) | `Lucky Parity — A Rigged Game, On Purpose` | `Odd-Even — A Rigged Game, On Purpose` |
| Neon marquee `<h1>` (~line 433) | `LUCKY PARITY` | `ODD-EVEN` |
| Start-modal heading (~line 403) | `Lucky Parity` | `Odd-Even` |
| `README.md` title/body | `Lucky Parity` | `Odd-Even` |

The "rigged on purpose / awareness demo" framing is retained. Internal identifiers,
class names, and the `.dc.html` design-project provenance notes are left unchanged.

### 2. One-shot force-win ("100% win" control)

- Add `state.forceNextWin = false` to the initial state and to `resetGame()`.
- In `resolveRound()` (~line 897), replace:
  ```js
  const win = Math.random() < prob;
  ```
  with:
  ```js
  const forced = state.forceNextWin;
  const win = forced ? true : (Math.random() < prob);
  state.forceNextWin = false; // one-shot: always consume
  ```
- Record `forced` on the history entry (`forced: forced`) so the truth screen can, if
  desired, distinguish operator-forced wins. Like `trueProb`, this field is **omitted from
  the player-facing scoreboard**, so it stays hidden during play.

**Arming triggers (all call one `armForceWin()` helper):**
- Keyboard: press **`w`**.
- Tap: an invisible fixed zone in the **top-left** corner (~48×48px, transparent,
  `z-index` above the stage, `aria-hidden`).
- Visible booth-panel button (see §4).

`armForceWin()` sets `state.forceNextWin = true`, gives discreet feedback (§5), and is a
no-op when `state.over` is true.

### 3. Silent "start rigging" trigger

- Reuse the existing `startRiggingNow()` (sets `state.grindStartRound = state.round`,
  refreshes locks/status). It already does the right thing; today it is only reachable via
  the visible panel button.
- Add triggers that call it **without** toggling the panel open:
  - Keyboard: press **`g`** (grind).
  - Tap: an invisible fixed zone in the **bottom-left** corner (mirror of the visible
    lower-right dot, but transparent).
- The lower-right dot and its panel remain unchanged for setup/inspection; the operator
  simply never needs to open it mid-demo.

### 4. Booth panel additions (visible path)

- Add a **"⚡ Force next win"** button to `#operatorPanel`, wired to `armForceWin()`.
- Add an armed-state line that reads e.g. `Force-win: ARMED (next roll)` when
  `state.forceNextWin` is true, else `Force-win: off`. Updated by the existing
  `updateOperatorStatus()`.

### 5. Discreet feedback (must not tip off players)

- On any trigger, apply a faint ~1s glow/pulse to the hidden lower-right dot
  (`#operatorDot`) via a temporary CSS class. To an onlooker this is meaningless.
- **Instructor mode** (already exists) is the operator's real confirmation channel:
  `updateInstructorPanel()` gains a line such as `⚡ NEXT ROLL: FORCED WIN` while armed, and
  the existing phase/true-odds readout already reflects a rig start.
- No banner, sound, or scoreboard change fires on the operator triggers.

### 6. Safety guards

- A single `keydown` listener on `document` handles `w` and `g`. It returns early when:
  - the event target is an `<input>`/`<textarea>` (so typing in the balance field is safe),
    or a modifier key (ctrl/meta/alt) is held;
  - `state.over` is true.
- Tap zones are `pointer-events:auto` only on themselves; they do not block the game UI
  because they sit in otherwise-empty corners and are small.

### 7. Changelog

Add entries under `## [Unreleased]` → `### Added` and `### Changed` describing the rename
and the two discreet operator controls, in the established Keep a Changelog voice.

## Testing / Verification

Manual, in-browser (no test harness exists in this single-file project):

1. **Rename:** page title, neon sign, and start modal all read "Odd-Even"; README updated.
2. **Force-win via key:** start a game, press `w`, roll → result is a win; the *following*
   roll (no re-press) resolves by normal rigged odds.
3. **Force-win via tap:** same, tapping the top-left corner.
4. **Force-win via panel button:** same, and the armed line shows/clears correctly.
5. **Silent rig via key/tap:** press `g` (or tap bottom-left) → operator status/instructor
   readout show grind phase, bet limit lifts, panel never opened.
6. **Guards:** focus the balance input, type a value containing `w`/`g` → no arming/rig
   fires; after cash-out (`state.over`), keys/taps do nothing.
7. **Discretion:** with Instructor mode OFF, triggering produces only the faint dot glow —
   no banner, no scoreboard change.

## Rollout

GitHub Flow: implement on `feature/odd-even-booth-controls`, open a PR to `main`, update
`CHANGELOG.md` in the same PR.
