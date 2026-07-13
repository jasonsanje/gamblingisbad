# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Celebratory win/lose feedback on each roll: a full-screen neon banner
  ("WINNER!" / "SO CLOSE!"), a confetti burst, chasing marquee bulbs framing the
  table, and a glow/shake on the felt. The manufactured euphoria and near-miss
  adrenaline reinforce the demo's lesson before the truth screen is revealed.
  Honors `prefers-reduced-motion` (banner still shows, confetti/shake suppressed).
- In-game scoreboard on the casino stage showing every completed round
  (#, Guess, Roll, Result, Bet). The true-odds column is intentionally omitted
  so the rigging stays hidden until the truth screen is revealed.

### Changed
- Moved the in-game scoreboard into a sticky right-hand sidebar (two-column
  casino layout), stacking below the game on narrow screens.
- The odd/even guess selection now clears after each roll, so the player must
  consciously re-pick a guess every round.
- Reaching ₱0 balance no longer auto-reveals the truth screen; the player is
  instead prompted to click "I want to stop" to end the session.

### Removed
- Fixed round limit (`AUTO_REVEAL_ROUND`) that ended the game after a set number
  of rounds. The session now ends only when the player clicks "I want to stop".

## [0.1.0] - 2026-07-13

### Added
- Initial "Lucky Parity" gambling-awareness demo: a deliberately rigged
  odd/even dice game with a hook phase, a grinding phase, an instructor mode,
  hidden operator controls, and a truth screen that exposes how the odds were
  manipulated.
- GitHub Pages deploy workflow, triggered on `main`.
