# Mobile Support Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make all three pages (`index.html`, `angles.html`, `hanoi.html`) usable on phones and small screens, including tap-to-move for Towers of Hanoi on touchscreens.

**Architecture:** Pure static HTML — each page is one self-contained file with inline CSS and a single `<script>` block. Changes are: media-query CSS for stacking layouts, a resize handler per game that sets the canvas's internal resolution from its container width and redraws, and a touch-only tap-to-move input path for Hanoi. Desktop mouse drag in Hanoi is untouched.

**Tech Stack:** Vanilla HTML/CSS/JS, 2D canvas. No build tooling, no test framework. Verification is done in a real browser via the Playwright MCP tools (`mcp__playwright__*`, loaded via ToolSearch).

**Spec:** `docs/superpowers/specs/2026-08-25-mobile-support-design.md`

## Global Constraints

- No build tooling; each HTML file stays fully self-contained (inline CSS + one script block).
- Green-on-black terminal aesthetic unchanged (`#00ff00` on `#0a0a0a`/`#000`, `Courier New`, glow shadows).
- Responsive breakpoint: `@media (max-width: 800px)`.
- Canvas internal resolution is capped at current sizes: Hanoi 700×400 (7:4 aspect), Angles 500×500 (square).
- Hanoi desktop drag-and-drop behaviour must not change.
- Touch targets on small screens: buttons min-height 44px.

## Verification Setup (used by every task)

There is no test framework. Each task is verified in a live browser:

1. Serve the site (once, leave running): `python3 -m http.server 8765 --directory /Users/greg/dev/brain-games` (run in background).
2. Load the Playwright MCP tools via ToolSearch: `select:mcp__playwright__browser_navigate,mcp__playwright__browser_resize,mcp__playwright__browser_take_screenshot,mcp__playwright__browser_evaluate,mcp__playwright__browser_click,mcp__playwright__browser_snapshot`.
3. "Phone viewport" means `browser_resize` to 390×844. "Desktop viewport" means 1280×900.
4. "No horizontal overflow" check (run with `browser_evaluate`):

```js
() => {
  const d = document.documentElement;
  return { scrollWidth: d.scrollWidth, innerWidth: window.innerWidth, ok: d.scrollWidth <= window.innerWidth };
}
```

Expected: `ok: true`.

---

### Task 1: Responsive menu page (`index.html`)

**Files:**
- Modify: `index.html` (add media query at end of `<style>` block, before `</style>`)

**Interfaces:**
- Consumes: nothing
- Produces: nothing (self-contained page change)

- [ ] **Step 1: Add media query**

In `index.html`, immediately before the closing `</style>` tag, add:

```css
        @media (max-width: 800px) {
            body {
                padding: 10px;
            }

            .container {
                padding: 25px 15px;
            }

            h1 {
                font-size: 1.8em;
                letter-spacing: 1px;
            }

            .subtitle {
                font-size: 1em;
                margin-bottom: 30px;
            }

            .game-card {
                padding: 20px;
            }

            .game-card h2 {
                font-size: 1.4em;
            }

            .game-card p {
                font-size: 1em;
            }

            .game-icon {
                font-size: 2em;
            }
        }
```

- [ ] **Step 2: Verify at phone viewport**

Navigate to `http://localhost:8765/index.html`, resize to 390×844, run the no-horizontal-overflow check (expect `ok: true`), and take a screenshot: heading fits on one line or wraps cleanly, both game cards visible and readable.

- [ ] **Step 3: Verify desktop unchanged**

Resize to 1280×900, take a screenshot: layout identical to before (centred 800px container, large heading).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Make menu page responsive on small screens"
```

---

### Task 2: Responsive layout CSS for angles game (`angles.html`)

**Files:**
- Modify: `angles.html` (`.canvas-container` rule ~line 48; add media query before `</style>`)

**Interfaces:**
- Consumes: nothing
- Produces: stacked single-column layout below 800px that Task 3's canvas resizing relies on (`.canvas-container` becomes full-width with no min-width).

- [ ] **Step 1: Remove the min-width**

In `angles.html`, in the `.canvas-container` rule, delete the line:

```css
            min-width: 500px;
```

- [ ] **Step 2: Add media query**

Immediately before the closing `</style>` tag, add:

```css
        @media (max-width: 800px) {
            body {
                padding: 10px;
            }

            .container {
                padding: 15px;
            }

            h1 {
                font-size: 1.6em;
                letter-spacing: 1px;
                margin-bottom: 20px;
            }

            .game-area {
                flex-direction: column;
                gap: 25px;
            }

            .info-panel {
                flex: none;
            }

            #resultMessage {
                font-size: 1.6em !important;
                height: 50px;
                line-height: 50px;
                letter-spacing: 3px;
                margin-top: 15px;
            }

            button {
                min-height: 44px;
            }

            input[type="checkbox"] {
                width: 28px;
                height: 28px;
            }
        }
```

(The `!important` on `#resultMessage` is required because the element carries an inline `font-size` style.)

- [ ] **Step 3: Verify at phone viewport**

Navigate to `http://localhost:8765/angles.html`, resize to 390×844. Run the no-horizontal-overflow check — expect `ok: true`. Note: the canvas is still 500px wide internally until Task 3, so the overflow check may fail on the canvas itself; what must be verified here is that `.game-area` stacks vertically (canvas above info panel) and the info panel/settings are full-width. Screenshot to confirm stacking.

- [ ] **Step 4: Verify desktop unchanged**

Resize to 1280×900, screenshot: side-by-side layout as before.

- [ ] **Step 5: Commit**

```bash
git add angles.html
git commit -m "Stack angles game layout on small screens"
```

---

### Task 3: Responsive canvas for angles game (`angles.html`)

**Files:**
- Modify: `angles.html` script block (top-of-script constants ~lines 265-269; initialisation section ~line 364; nothing else)

**Interfaces:**
- Consumes: stacked layout from Task 2 (`.canvas-container` width drives canvas size)
- Produces: `resizeCanvas()` — sets canvas resolution, recomputes `centerX`/`centerY`/`radius`, redraws. All existing draw code keeps using the same three variable names.

- [ ] **Step 1: Make geometry variables mutable and add resize handler**

Replace:

```js
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const centerX = canvas.width / 2;
        const centerY = canvas.height / 2;
        const radius = 180;
```

with:

```js
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const canvasContainer = canvas.parentElement;
        let centerX = canvas.width / 2;
        let centerY = canvas.height / 2;
        let radius = 180;

        function resizeCanvas() {
            const size = Math.min(500, canvasContainer.clientWidth);
            canvas.width = size;
            canvas.height = size;
            centerX = size / 2;
            centerY = size / 2;
            radius = size * 0.36;
            draw();
        }

        window.addEventListener('resize', resizeCanvas);
        window.addEventListener('orientationchange', resizeCanvas);
```

(0.36 preserves the original proportions: 180/500.)

- [ ] **Step 2: Call resizeCanvas on load**

At the bottom of the script, replace:

```js
        // Initial draw
        draw();
```

with:

```js
        // Initial size + draw
        resizeCanvas();
```

- [ ] **Step 3: Verify at phone viewport**

Navigate to `http://localhost:8765/angles.html`, resize to 390×844. Run the no-horizontal-overflow check — expect `ok: true` now. Verify via `browser_evaluate`:

```js
() => {
  const c = document.getElementById('gameCanvas');
  return { w: c.width, h: c.height, square: c.width === c.height, fits: c.getBoundingClientRect().right <= window.innerWidth };
}
```

Expected: `w` < 500, `square: true`, `fits: true`. Click START, wait ~2s, click STOP — game plays, line rotates within the visible circle, result message fits.

- [ ] **Step 4: Verify desktop unchanged**

Resize to 1280×900, reload. Same evaluate: expect `w: 500, h: 500`. Play one round — behaviour identical to before.

- [ ] **Step 5: Commit**

```bash
git add angles.html
git commit -m "Resize angles canvas to fit small screens"
```

---

### Task 4: Responsive layout CSS for Hanoi (`hanoi.html`)

**Files:**
- Modify: `hanoi.html` (`.canvas-container` rule ~line 48; add media query before `</style>`)

**Interfaces:**
- Consumes: nothing
- Produces: stacked single-column layout below 800px that Task 5's canvas resizing relies on.

- [ ] **Step 1: Remove the min-width**

In `hanoi.html`, in the `.canvas-container` rule, delete the line:

```css
            min-width: 500px;
```

- [ ] **Step 2: Add media query**

Immediately before the closing `</style>` tag, add:

```css
        @media (max-width: 800px) {
            body {
                padding: 10px;
            }

            .container {
                padding: 15px;
            }

            h1 {
                font-size: 1.6em;
                letter-spacing: 1px;
                margin-bottom: 20px;
            }

            .game-area {
                flex-direction: column;
                gap: 25px;
            }

            .info-panel {
                flex: none;
            }

            #resultMessage {
                font-size: 1.6em !important;
                height: 50px !important;
                line-height: 50px !important;
                letter-spacing: 3px !important;
                margin-top: 15px !important;
            }

            button {
                min-height: 44px;
            }
        }
```

(The `!important`s are required because Hanoi's `#resultMessage` carries all of these as inline styles.)

- [ ] **Step 3: Verify at phone viewport**

Navigate to `http://localhost:8765/hanoi.html`, resize to 390×844. Screenshot: `.game-area` stacked (canvas above info panel), settings full-width. As in Task 2, the canvas itself is still 700px until Task 5 — stacking is what's being verified.

- [ ] **Step 4: Verify desktop unchanged**

Resize to 1280×900, screenshot: side-by-side layout as before.

- [ ] **Step 5: Commit**

```bash
git add hanoi.html
git commit -m "Stack Hanoi layout on small screens"
```

---

### Task 5: Responsive canvas for Hanoi (`hanoi.html`)

**Files:**
- Modify: `hanoi.html` script block (top-of-script setup; `getRodBaseY`, `getDiskWidth`, `getRodFromX`, `draw`, `drawDisk`; the `// Initialize` section at the bottom)

**Interfaces:**
- Consumes: stacked layout from Task 4
- Produces: `scale()` — returns `canvas.width / 700`; used to scale all horizontal/vertical pixel constants. `resizeCanvas()` — sets canvas resolution (width capped at 700, height = width × 4/7) and redraws. Task 6 relies on `getRodFromX(x)` continuing to work at any canvas size.

- [ ] **Step 1: Add scale helper and resize handler**

Immediately after `const ctx = canvas.getContext('2d');`, add:

```js
        const canvasContainer = canvas.parentElement;

        function scale() {
            return canvas.width / 700;
        }

        function resizeCanvas() {
            const w = Math.min(700, canvasContainer.clientWidth);
            canvas.width = w;
            canvas.height = Math.round(w * 400 / 700);
            draw();
        }

        window.addEventListener('resize', resizeCanvas);
        window.addEventListener('orientationchange', resizeCanvas);
```

- [ ] **Step 2: Scale the geometry functions**

Replace `getRodBaseY` with:

```js
        function getRodBaseY() {
            return canvas.height - 50 * scale();
        }
```

Replace the body of `getDiskWidth` with:

```js
        function getDiskWidth(size) {
            const range = MAX_DISK_WIDTH - MIN_DISK_WIDTH;
            const step = range / diskCount;
            return (MIN_DISK_WIDTH + (size * step)) * scale();
        }
```

In `getRodFromX`, replace the condition `if (Math.abs(x - rodX) < BASE_WIDTH / 2) {` with:

```js
                if (Math.abs(x - rodX) < (BASE_WIDTH / 2) * scale()) {
```

- [ ] **Step 3: Scale the draw code**

In `draw()`:

- At the top of the function, after `const baseY = getRodBaseY();`, add: `const s = scale();`
- Base: `ctx.fillRect(x - BASE_WIDTH / 2, baseY, BASE_WIDTH, BASE_HEIGHT);` → `ctx.fillRect(x - (BASE_WIDTH * s) / 2, baseY, BASE_WIDTH * s, BASE_HEIGHT);`
- Rod: `ctx.fillRect(x - ROD_WIDTH / 2, baseY - ROD_HEIGHT, ROD_WIDTH, ROD_HEIGHT);` → `ctx.fillRect(x - ROD_WIDTH / 2, baseY - ROD_HEIGHT * s, ROD_WIDTH, ROD_HEIGHT * s);`
- Rod label font: `ctx.font = '16px "Courier New", monospace';` → `ctx.font = `${Math.max(11, Math.round(16 * s))}px "Courier New", monospace`;`
- Label position: `ctx.fillText(`ROD ${i + 1}`, x, baseY + BASE_HEIGHT + 25);` → `ctx.fillText(`ROD ${i + 1}`, x, baseY + BASE_HEIGHT + Math.max(18, 25 * s));`
- Disk stacking: `const y = baseY - diskIndex * DISK_HEIGHT;` → `const y = baseY - diskIndex * DISK_HEIGHT * s;`

In `drawDisk()`, add `const s = scale();` as the first line, and replace both rect calls:

```js
            ctx.fillRect(x - width / 2, y - DISK_HEIGHT * s, width, DISK_HEIGHT * s);
```

```js
            ctx.strokeRect(x - width / 2, y - DISK_HEIGHT * s, width, DISK_HEIGHT * s);
```

(`width` is already scaled — it comes from `getDiskWidth`.)

- [ ] **Step 4: Call resizeCanvas on load**

At the bottom of the script, replace:

```js
        // Initialize
        loadSettings();
        initGame();
```

with:

```js
        // Initialize
        loadSettings();
        initGame();
        resizeCanvas();
```

- [ ] **Step 5: Verify at phone viewport**

Navigate to `http://localhost:8765/hanoi.html`, resize to 390×844. Run the no-horizontal-overflow check — expect `ok: true`. Verify canvas size via `browser_evaluate`: `() => { const c = document.getElementById('gameCanvas'); return { w: c.width, h: c.height }; }` — expect `w` < 700 and `h ≈ w * 4/7`. Set the disk slider to 8 and screenshot: all 8 disks visible on rod 1, distinguishable widths, rod labels legible, three rods evenly spaced.

- [ ] **Step 6: Verify desktop drag still works**

Resize to 1280×900, reload. Expect canvas 700×400. Using Playwright, drag the top disk from rod 1 to rod 3 (mouse down over rod 1 at canvas centre-height, move to rod 3, mouse up — rod centres are at x = canvas-left + 175/350/525). Expect: MOVES counter = 1, disk now on rod 3.

- [ ] **Step 7: Commit**

```bash
git add hanoi.html
git commit -m "Resize Hanoi canvas to fit small screens"
```

---

### Task 6: Tap-to-move touch input for Hanoi (`hanoi.html`)

**Files:**
- Modify: `hanoi.html` script block (game state ~line 271; `initGame`; disk loop inside `draw`; new `touchstart` listener next to the mouse handlers)

**Interfaces:**
- Consumes: `getRodFromX(x)`, `isValidMove(diskSize, targetRod)`, `showWarning(message)`, `checkWin()`, `updateDisplay()`, `draw()`, `scale()` — all existing (Task 5 for `scale`).
- Produces: nothing consumed later.

- [ ] **Step 1: Add selection state**

In the `// Game state` block, after `let dragSourceRod = null;`, add:

```js
        let selectedRod = null;
```

In `initGame()`, after `dragSourceRod = null;`, add:

```js
            selectedRod = null;
```

- [ ] **Step 2: Draw the selected disk lifted**

In `draw()`, inside the disk loop, immediately after the existing "Skip if this is the disk being dragged" block, add:

```js
                    // Draw the tap-selected top disk lifted above its rod
                    if (selectedRod === rodIndex && diskIndex === rod.length - 1) {
                        const liftedWidth = getDiskWidth(diskSize);
                        drawDisk(getRodX(rodIndex), baseY - ROD_HEIGHT * s - 15, liftedWidth, diskSize, true);
                        continue;
                    }
```

(`s` is in scope from Task 5's Step 3. Passing `true` reuses the drag glow styling.)

- [ ] **Step 3: Add the touchstart handler**

After the `canvas.addEventListener('mouseleave', ...)` block, add:

```js
        // Touch: tap a rod to pick up its top disk, tap another rod to drop it
        canvas.addEventListener('touchstart', (e) => {
            e.preventDefault();
            if (gameWon) return;

            const rect = canvas.getBoundingClientRect();
            const x = e.changedTouches[0].clientX - rect.left;
            const rodIndex = getRodFromX(x);
            if (rodIndex === -1) return;

            if (selectedRod === null) {
                if (rods[rodIndex].length > 0) {
                    selectedRod = rodIndex;
                    draw();
                }
            } else if (rodIndex === selectedRod) {
                // Tapping the same rod cancels without counting a move
                selectedRod = null;
                draw();
            } else {
                const diskSize = rods[selectedRod][rods[selectedRod].length - 1];
                if (isValidMove(diskSize, rodIndex)) {
                    rods[selectedRod].pop();
                    rods[rodIndex].push(diskSize);
                    moves++;
                    selectedRod = null;
                    updateDisplay();
                    draw();
                    checkWin();
                } else {
                    showWarning('⚠ INVALID MOVE!');
                    selectedRod = null;
                    draw();
                }
            }
        }, { passive: false });
```

Notes: `preventDefault()` stops the tap from scrolling/zooming AND suppresses the browser's synthesised `mousedown`/`mouseup`, so the drag path never fires for touches. `{ passive: false }` is required for `preventDefault()` to be honoured.

- [ ] **Step 4: Verify tap-to-move at phone viewport**

Navigate to `http://localhost:8765/hanoi.html`, resize to 390×844. Synthesise taps with `browser_evaluate`:

```js
() => {
  window.tap = (rodIndex) => {
    const c = document.getElementById('gameCanvas');
    const r = c.getBoundingClientRect();
    const x = r.left + (c.width / 4) * (rodIndex + 1);
    const y = r.top + c.height / 2;
    const t = new Touch({ identifier: Date.now(), target: c, clientX: x, clientY: y });
    c.dispatchEvent(new TouchEvent('touchstart', { touches: [t], changedTouches: [t], bubbles: true, cancelable: true }));
  };
  return 'tap helper installed';
}
```

Then verify each behaviour (read `#moves` text and screenshot between steps):

1. `tap(0)` → screenshot shows top disk lifted and glowing above rod 1.
2. `tap(0)` again → disk back in place, MOVES still 0 (cancel).
3. `tap(0)` then `tap(2)` → smallest disk on rod 3, MOVES = 1.
4. `tap(0)` then `tap(2)` → INVALID MOVE warning appears (bigger disk onto smaller), MOVES still 1, disk stays on rod 1.
5. With 3 disks, play the 7-move optimal solution via taps: (0→2, 0→1, 2→1, 0→2, 1→0, 1→2, 0→2). Expect "✓ PERFECT!" and MOVES = 7.

- [ ] **Step 5: Verify desktop drag unaffected**

Resize to 1280×900, reload, repeat Task 5 Step 6's drag check — drag still works, MOVES counts correctly.

- [ ] **Step 6: Commit**

```bash
git add hanoi.html
git commit -m "Add tap-to-move touch input to Hanoi"
```

---

### Task 8: Surface controls above stats on small screens (user-reported bug)

**Files:**
- Modify: `angles.html` (inside the existing `@media (max-width: 800px)` block)
- Modify: `hanoi.html` (inside the existing `@media (max-width: 800px)` block)

**Interfaces:**
- Consumes: the 800px media queries added by Tasks 2 and 4
- Produces: on small screens, `.controls` renders above the `.info-item` stats inside `.info-panel`.

**Bug (user-reported, reproduced):** at 390×844 the stacked layout keeps desktop document order — the five stat boxes sit between the canvas and the buttons, pushing the primary controls ~200px below the fold (angles START button measured at y=1041 in an 844px viewport). The game cannot be played without scrolling. Fix verified by live CSS injection: with the rules below, START lands at y=561–632, fully visible.

- [ ] **Step 1: Add ordering rules to angles.html**

In `angles.html`, inside the existing `@media (max-width: 800px)` block, after the `.info-panel { flex: none; }` rule, add:

```css
            .info-panel {
                display: flex;
                flex-direction: column;
            }

            .controls {
                order: -1;
                margin-bottom: 20px;
            }
```

(Keep the existing `.info-panel { flex: none; }` rule; the new declarations can be merged into it or added as a second rule — merged is cleaner: `flex: none; display: flex; flex-direction: column;`.)

- [ ] **Step 2: Add the same rules to hanoi.html**

Same edit in `hanoi.html`'s `@media (max-width: 800px)` block: `.info-panel` gains `display: flex; flex-direction: column;` and a `.controls { order: -1; margin-bottom: 20px; }` rule is added.

- [ ] **Step 3: Verify at phone viewport**

At 390×844 for BOTH `angles.html` and `hanoi.html`: via `browser_evaluate`, the primary button's bounding rect (`#startButton` for angles, `#newGameButton` for Hanoi) must satisfy `bottom <= window.innerHeight` without scrolling; buttons render above the stat boxes; stats remain below and reachable by scrolling. Screenshot each.

- [ ] **Step 4: Verify desktop unchanged**

At 1280×900, both games: side-by-side layout with stats above buttons in the info panel, exactly as before (the new rules are inside the media query and must not affect desktop).

- [ ] **Step 5: Commit**

```bash
git add angles.html hanoi.html
git commit -m "Show controls above stats on small screens"
```

---

### Task 9: Compact mobile controls (user-reported)

**Files:**
- Modify: `angles.html` (inside the existing `@media (max-width: 800px)` block only)
- Modify: `hanoi.html` (same)

**Interfaces:**
- Consumes: the 800px media queries from Tasks 2/4/8
- Produces: on small screens, the buttons form a compact grid so canvas + message area + all buttons are fully visible without scrolling even in a 390×700 viewport. The `#resultMessage` block is deliberately KEPT — it is where round results display; do not remove, collapse, or overlay it.

**User feedback (live testing):** only the START/STOP button is visible without scrolling on a real phone (browser chrome shrinks the 844px viewport). The buttons need to be more compact; the message area below the canvas stays.

- [ ] **Step 1: Compact the angles controls**

In `angles.html`'s `@media (max-width: 800px)` block:

Change the `.game-area` rule's `gap: 25px` to `gap: 10px` (keep `flex-direction: column`).

Replace the existing `.controls { order: -1; margin-bottom: 20px; }` rule and the `button { min-height: 44px; }` rule with:

```css
            .controls {
                order: -1;
                margin-bottom: 15px;
                display: grid;
                grid-template-columns: 1fr 1fr;
                gap: 10px;
            }

            button {
                min-height: 44px;
                padding: 10px 8px;
                font-size: 0.9em;
                letter-spacing: 1px;
            }

            .start-button {
                grid-column: 1 / -1;
                font-size: 1.2em;
                padding: 12px;
            }
```

(START spans both columns; NEXT ROUND + RESTART GAME share the second row; CLEAR SCORES sits in the third row's first cell.)

Leave the existing `#resultMessage` rule exactly as it is.

- [ ] **Step 2: Same rework in hanoi.html**

In `hanoi.html`'s `@media (max-width: 800px)` block, make the equivalent edits: `.game-area` gap to `10px`; replace the `.controls` and `button` rules with the same grid/compact rules (NEW GAME carries `.start-button` and spans both columns; RESTART and CLEAR SCORES share the second row). Leave `#resultMessage` untouched.

- [ ] **Step 3: Verify on a short phone viewport**

At **390×700** (not just 390×844), for BOTH games: via `browser_evaluate`, every button in `.controls` must have `getBoundingClientRect().bottom <= window.innerHeight` without scrolling. Play one angles round (START → STOP): the result message appears in its block below the canvas as before. Screenshot each game.

- [ ] **Step 4: Verify desktop unchanged**

At 1280×900, both games: result message still renders in its own block below the canvas, buttons full-size in a column — identical to before.

- [ ] **Step 5: Commit**

```bash
git add angles.html hanoi.html
git commit -m "Compact mobile controls and overlay result message"
```

---

### Task 7: Full verification pass

**Files:** none modified (fix-forward if issues found)

**Interfaces:** consumes everything above.

- [ ] **Step 1: Phone pass (390×844)**

For each of `index.html`, `angles.html`, `hanoi.html`: navigate, run the no-horizontal-overflow check (`ok: true`), screenshot, and confirm all controls are reachable and legible. The primary button (`#startButton` / `#newGameButton`) must be fully inside the viewport without scrolling (Task 8's rule). Play one full round of the angles game and one 3-disk Hanoi solve via taps.

- [ ] **Step 2: Narrow-phone spot check (320×568)**

Load both games at 320px width: no horizontal overflow, canvases fit.

- [ ] **Step 3: Desktop regression pass (1280×900)**

All three pages look and behave as before: side-by-side layouts, full-size canvases, Hanoi drag works, angles game plays.

- [ ] **Step 4: Fix anything found, then final commit if changes were made**

Any issue found: fix in the relevant file, re-verify, commit with a message describing the fix.
