# Mobile Support for Brain Games — Design

**Date:** 2026-08-25
**Scope:** `index.html`, `angles.html`, `hanoi.html`

## Problem

The site is unusable on phones and small screens:

- Both games use fixed-size canvases (Hanoi 700×400, Angles 500×500) inside containers with `min-width: 500px`, and side-by-side flex layouts that never stack, causing horizontal overflow.
- Towers of Hanoi handles only mouse events (`mousedown`/`mousemove`/`mouseup`), so it cannot be played on a touchscreen at all.
- The angles game is button-driven so it works on touch, but does not fit the screen.

## Decisions

- **Touch input for Hanoi:** tap-to-move (tap a rod to pick up its top disk, tap another to drop). Desktop keeps the existing drag-and-drop unchanged.
- **Canvas sizing:** resize the canvas's internal resolution to fit its container and redraw, rather than CSS-scaling a fixed-resolution canvas. Keeps rendering crisp and hit targets accurate.

## Design

### 1. Layout (all three pages)

- Remove `min-width: 500px` from `.canvas-container`.
- Add a media query at ~800px that stacks `.game-area` vertically (canvas above the info/controls panel) and reduces container padding, heading sizes, and letter-spacing so nothing overflows.
- `index.html` needs only padding/font-size reductions; its game cards already stack.

### 2. Responsive canvas (both games)

On load, window resize, and orientation change, set the canvas's internal pixel size from the container's actual width — capped at the current sizes (700×400 Hanoi, 500×500 Angles), preserving aspect ratio — then redraw.

- **Hanoi:** rod positions and base Y already derive from `canvas.width`/`height`. Scale disk widths proportionally so 7 disks still fit on narrow layouts.
- **Angles:** replace the fixed `centerX`/`centerY`/`radius = 180` constants with values recomputed from the canvas size (radius ≈ 36% of canvas width).

### 3. Hanoi tap-to-move (touch only)

- First tap on a rod picks up its top disk, drawn highlighted/lifted.
- Tap on another rod attempts the drop through the existing `isValidMove`/move-counting/win logic; invalid drops show the existing warning.
- Tapping the same rod again cancels the pick-up without counting a move (matches existing same-rod drag behaviour).
- Touch handlers call `preventDefault()` so canvas taps don't scroll or zoom the page.
- Desktop mouse drag path is untouched.

### 4. Small-screen ergonomics

Buttons and checkboxes get larger touch targets (min-height ~44px) on small screens.

### 5. Testing

Verify with Playwright at a phone viewport (390×844): all pages fit with no horizontal scroll; Hanoi is playable via tap events. Re-verify desktop drag still works at full size.

## Constraints

No build tooling; each file stays self-contained; green-on-black terminal aesthetic unchanged.
