# Multiples & Factors Blitz — Design

**Date:** 2026-08-25
**Scope:** new `blitz.html`, one addition to `index.html`

## Game

A 60-second arithmetic sprint. Each round shows a rule — "MULTIPLES OF 7" or "FACTORS OF 84" — above a 4×4 grid of numbers. The player taps every number satisfying the rule:

- Correct tap: cell lights up green and stays lit; +1 to score.
- Wrong tap: cell flashes a warning style; 3 seconds deducted from the clock.
- When all qualifiers in the grid are found, the grid clears and a new rule + grid appears (no time bonus; the sprint clock keeps running).
- When the clock reaches zero, the game ends and the final score (total correct taps) is shown.

## Grid generation

- Pick the rule per round (respecting the rule-mode setting; "mixed" alternates randomly).
- Multiples round: pick target T in the configured range; the qualifier set is multiples of T; distractors are near-misses (multiple ± 1, or multiples of a nearby number).
- Factors round: pick a composite target with at least 4 factors in cell range; qualifiers are its factors; distractors share digits with real factors or are off-by-one.
- Each 16-cell grid contains 4–7 qualifiers, positions shuffled. All values distinct within a grid.

## Settings

- Rule mode: MULTIPLES / FACTORS / MIXED (default MIXED).
- Number range: SMALL (multiple-targets 2–9, factor-targets ≤ 60) / LARGE (multiple-targets 2–19, factor-targets ≤ 144). Default SMALL.

## Scoring & persistence

- Score = correct taps in the 60-second game; countdown displayed prominently (timer text turns brighter/pulses under 10s).
- Best score per settings combination (key `"<mode>_<range>"`), stored in `blitzBestScores`; settings in `blitzSettings`. Same localStorage pattern as the other games.
- Buttons: START (becomes the round in progress; disabled mid-game), RESTART, CLEAR SCORES (confirm dialog).

## Rendering & layout

- DOM/CSS grid of `<button>` cells (like Lights Out) — no canvas. Rule + timer in the info panel and above the grid on mobile.
- One self-contained `blitz.html`: standard `.container`/`.game-area`/`.info-panel`/`.settings` structure, green-on-black terminal aesthetic, the site's `@media (max-width: 800px)` treatment (stacked layout, controls-above-stats compact grid, buttons min-height 44px) from day one.
- Menu card added to `index.html` (icon `÷` or `%`).

## Constraints

- No build tooling; file fully self-contained; vanilla JS.
- Implemented after Lights Out (queue order; both touch `index.html`).
