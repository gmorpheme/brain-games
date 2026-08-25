# Code Breaker — Design

**Date:** 2026-08-25
**Scope:** new `codebreaker.html`, one addition to `index.html`

## Game

Mastermind-style deduction with terminal symbols instead of colours. The game secretly picks a sequence of 4 symbols from a palette of 6 — `◆ ▲ ● ■ ✚ ★` — all rendered in the site's green.

- The player builds a guess by tapping palette symbols, which fill the four guess slots left to right; tapping a filled slot clears it (and shifts later symbols left).
- SUBMIT is enabled only when all 4 slots are filled.
- Feedback per submitted guess: `●` count = right symbol in the right position; `○` count = right symbol, wrong position (standard Mastermind counting: exact matches first, then frequency overlap on the remainder).
- Guess history (guess + feedback) stays on screen, most recent at the top.
- Win: feedback is 4 `●` — "✓ CRACKED IN N!". Lose: 10 guesses exhausted — the secret is revealed.

## Settings

- Duplicates: OFF (secret has 4 distinct symbols, default) / ON (symbols may repeat — harder).

## Scoring & persistence

- Score = guesses used to crack the code; best = fewest, tracked per settings key (`"dup"` / `"nodup"`), stored in `codeBreakerBestScores`; settings in `codeBreakerSettings`. Losses don't record a score.
- Buttons: NEW GAME, CLEAR SCORES (confirm dialog).

## Rendering & layout

- Pure DOM: palette row of 6 symbol buttons, active guess row of 4 slots, SUBMIT button, scrolling history list. No canvas.
- One self-contained `codebreaker.html`: standard structure and green-on-black terminal aesthetic; the site's `@media (max-width: 800px)` treatment (stacked layout, compact controls, 44px touch targets) from day one. Symbols sized generously (≥44px touch targets) on mobile.
- Menu card added to `index.html` (icon `◆` — or `?` to avoid clashing with Hanoi's icon).

## Constraints

- No build tooling; file fully self-contained; vanilla JS.
- Implemented after Lights Out and Multiples & Factors Blitz (queue order; all touch `index.html`).
