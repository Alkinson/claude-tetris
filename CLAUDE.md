# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step. Open directly or serve statically:

```powershell
# Windows
start index.html

# Local server (recommended — avoids CORS quirks)
python -m http.server 8000
# then open http://localhost:8000
```

No `package.json`, no bundler, no transpiler.

## Architecture

Three files, no modules:

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600px), sidebar panel, overlay div for PAUSE/GAME OVER.
- **`style.css`** — Dark/retro theme only; no layout logic lives here.
- **`game.js`** — All game logic (~300 lines, `'use strict'`, global state).

### game.js internals

**State** — all mutable state lives in module-level `let` vars: `board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`, `lastTime`.

**Board** — `ROWS × COLS` (20×10) 2D array. Cell value `0` = empty; `1–7` = color index matching `COLORS[]` and `PIECES[]`.

**Pieces** — stored as square matrices in `PIECES[]`. Rotation: `rotateCW()` = transpose + reverse rows. Wall kicks: `tryRotate()` tries offsets `[0, -1, 1, -2, 2]`.

**Game loop** — `requestAnimationFrame`-based. `loop(ts)` accumulates `dropAccum`; fires gravity when `dropAccum ≥ dropInterval`. Each tick ends with `draw()`.

**Lock chain** — `lockPiece() → merge() + clearLines() + spawn()`. If `spawn()` detects immediate collision → `endGame()`.

**Rendering** — `draw()` clears canvas, draws grid, locked board, ghost (alpha 0.2 via `ghostY()`), then current piece. `drawNext()` renders to the separate `#next-canvas`.

### Tunable constants (top of game.js)

| Constant | Default | Note |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | Also update canvas `width`/`height` in `index.html` (`COLS×BLOCK` / `ROWS×BLOCK`) |
| `BLOCK` | 30px | Pixel size per cell |
| `COLORS` | 7 colors | Index 0 = null (empty) |
| `LINE_SCORES` | `[0,100,300,500,800]` | Multiplied by current level |
| `dropInterval` | 1000ms | Initial fall speed; recalculated as `max(100, 1000 − (level−1) × 90)` |
