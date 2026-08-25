# Future Game Ideas

Candidates researched on 2026-08-25 and parked for later. Each fits the site's constraints: single self-contained HTML file, vanilla JS, no assets/server, tap-only input, natural best-score metric for localStorage. (Multiples & Factors Blitz and Code Breaker from the same research round were taken forward to specs instead.)

## Shape Spinner (mental-rotation geometry)

A target blocky polygon is shown alongside 4 candidates: one is a true rotation, the rest are mirrored or altered decoys. Tap the true rotation; streak/accuracy scoring, difficulty scales via rotation angle and subtler decoys. Represent shapes as polyomino coordinate arrays; rotate via coordinate transforms; render on small canvases or SVG. Generation must guarantee exactly one true match. Prior art: Vandenberg–Kuse mental rotation tests.

## Nonogram Mini (picture-logic puzzle)

Classic picross: a 5×5–10×10 grid with run-length clues per row/column; tap to fill/clear cells until the pattern matches every clue. Score on time and mistakes per grid size. Generate a random solution grid (density-biased), derive clues by run-length encoding — always consistent by construction; uniqueness checking optional for a casual game. DOM grid plus clue strips.

## Sequence Recall (Simon-style working memory)

A 2×2/3×3 grid of tiles flashes a growing sequence; tap it back in order. A correct repeat extends the sequence; a mistake ends the round. Score = longest sequence reached; difficulty is self-scaling (optionally speed up playback tempo as rounds grow). Pure DOM/CSS with setTimeout-chained highlights. Prior art: the classic Simon game.

Full research write-ups (fit rationale, implementation sketches, links) were produced in the 2026-08-25 research session; this file records the essentials.
