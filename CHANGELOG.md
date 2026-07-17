# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Discreet operator controls for booth use, so the instructor never has to open the
  visible lower-right panel mid-demo (which onlookers could notice):
  - **One-shot force-win** — arms the *next* roll as a guaranteed win, then clears itself
    automatically so the rigged odds resume. Triggered by pressing `w`, tapping the
    invisible top-left corner, or the new "⚡ Force next win" button in the booth panel.
    The forced outcome is tagged on the round history but kept out of the player-facing
    scoreboard, so it stays hidden during play and only the operator's Instructor-mode
    readout announces `⚡ NEXT ROLL: FORCED WIN`.
  - **Silent rig start** — starts the grind phase without opening the panel, via pressing
    `g` or tapping the invisible bottom-left corner (a hidden mirror of the visible dot).
  - Both triggers give only a faint 1-second glow on the hidden dot as confirmation —
    meaningless to onlookers. Keyboard shortcuts are ignored while a text field is focused
    or the game is over, so they never fire by accident.

### Changed
- Renamed the game from "Lucky Parity" to **"Odd-Even"** across the page title, neon
  marquee sign, start-of-game modal, and README. The gambling-awareness framing is
  unchanged.

- "Pity" comeback rigging: each consecutive loss nudges the next roll's win
  probability up by 4% (stacking, capped at +20%), and any win snaps the odds
  straight back to the brutal grind rate. Total win probability is clamped at
  95% so a win is never guaranteed. Instructor mode's "true win prob" reflects
  the live pity bonus and annotates it (e.g. `(+12% pity)`) — the near-miss
  "you're due" manipulation, exposed. Implemented as `effectiveProb` wrapping
  the base `computeProb`, with the effective odds recorded in the round history.

- Starting-balance prompt shown before every game: a casino-styled modal asks
  the player how much they're playing with, and their answer becomes the opening
  stack. "Play again" / operator reset re-opens the prompt so each session starts
  by asking again.
- Round-4 "bonus round" hook: after round 3 a flashy, casino-themed dialog
  (glowing gradient border, radiating light rays, neon "2×") announces that the
  next round pays double, urging the player to bet up to ₱1,000. Winnings on the
  4th round are doubled (losses are not) — the classic raise-your-bet
  manipulation, flagged with ⭐×2 in the scoreboard/ledger. Honors
  `prefers-reduced-motion` (animations suppressed).
- Celebratory win/lose feedback on each roll: a full-screen neon banner
  ("WINNER!" / "SO CLOSE!"), a confetti burst, chasing marquee bulbs framing the
  table, and a glow/shake on the felt. The manufactured euphoria and near-miss
  adrenaline reinforce the demo's lesson before the truth screen is revealed.
  Honors `prefers-reduced-motion` (banner still shows, confetti/shake suppressed).
- In-game scoreboard on the casino stage showing every completed round
  (#, Guess, Roll, Result, Bet). The true-odds column is intentionally omitted
  so the rigging stays hidden until the truth screen is revealed.

### Changed
- Reimplemented the casino stage from the "Lucky Parity" Claude Design project
  (neon Las Vegas theme): a glowing Monoton marquee sign with chasing bulbs,
  ambient night glows and rotating light rays, slot-machine reel housings around
  the dice, a jackpot/streak meter, a live chip-stack visual, a balance
  sparkline, and win/lose SFX pops with a near-miss table shake. Truth mode is
  kept deliberately sober (receipt ledger, probability chart, help panel). Ported
  from the `.dc.html` design-compiler source into a standalone `index.html`.
- Bet chip denominations are now ₱100 / ₱500 / ₱1,000 (previously
  ₱1,000 / ₱10,000 / ₱100,000).
- Moved the in-game scoreboard into a sticky right-hand sidebar (two-column
  casino layout), stacking below the game on narrow screens.
- The odd/even guess selection now clears after each roll, so the player must
  consciously re-pick a guess every round.
- Reaching ₱0 balance no longer auto-reveals the truth screen; the player is
  instead prompted to click "I want to stop" to end the session.

### Removed
- Truth-screen "THE HOOK" explanation card; the remaining cards are renumbered
  (THE GRIND → 01, WHY IT WORKS → 02).
- Fixed round limit (`AUTO_REVEAL_ROUND`) that ended the game after a set number
  of rounds. The session now ends only when the player clicks "I want to stop".

## [0.1.0] - 2026-07-13

### Added
- Initial "Lucky Parity" gambling-awareness demo: a deliberately rigged
  odd/even dice game with a hook phase, a grinding phase, an instructor mode,
  hidden operator controls, and a truth screen that exposes how the odds were
  manipulated.
- GitHub Pages deploy workflow, triggered on `main`.
