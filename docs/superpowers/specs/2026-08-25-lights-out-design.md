# Lights Out Game — Design

**Date:** 2026-08-25
**Scope:** new `lights.html`, one addition to `index.html`

## Game

Classic Lights Out: an N×N grid of lights, some lit at the start. Pressing a cell toggles it and its four orthogonal neighbours. The player wins when every light is off.

- Board size selectable 3×3–7×7 via a slider (default 5×5), following Hanoi's disk-slider pattern. Changing size starts a new game.
- Moves are counted; "par" (the number of presses used to scramble the board) is displayed alongside.

## Puzzle generation

Start from an all-off board and apply K random simulated presses, where K = ceil(N×N / 2), choosing K *distinct* cells (a cell pressed twice cancels out, so distinct cells guarantee the puzzle genuinely needs up to K presses and is always solvable). Par = K.

## Rendering & input

- The board is a CSS grid of `<button>` cells inside the standard `.game-area` layout — no canvas.
- Lit cells: bright green fill with glow. Unlit cells: dark with a faint `#004400` border.
- Click and tap both work natively; keyboard focus works for free.
- The grid sizes with relative units (`aspect-ratio: 1`, width capped) so it works on phones from the start, including the standard `@media (max-width: 800px)` stacking layout used by the other games.

## Scoring & persistence

- Move counter and par shown in the info panel; best score = fewest moves, tracked per board size.
- `localStorage` keys: `lightsOutSettings` ({ gridSize }), `lightsOutBestScores` ({ [gridSize]: fewestMoves }) — same pattern as the other games.
- Buttons: NEW GAME, CLEAR SCORES (with confirm dialog, matching the other games).
- Win message ("✓ SOLVED!" / "✓ NEW BEST!") in the standard result-message style.

## Page & menu

- One self-contained `lights.html`: inline CSS, markup, single script block; green-on-black terminal aesthetic (`#00ff00` on `#0a0a0a`/`#000`, Courier New, glow shadows).
- Third `.game-card` on `index.html` linking to it (icon `▦` or similar).

## Constraints

- No build tooling; file fully self-contained.
- Must be implemented after the mobile-support work (it modifies `index.html`, which the mobile tasks touch) and must include the same responsive treatment from day one.
