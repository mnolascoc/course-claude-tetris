# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris implementation using HTML5 Canvas. No build process, no dependencies, no package.json — just static files served or opened directly.

## Running

```bash
start index.html        # Windows: open directly in browser
python3 -m http.server 8000   # or: npx serve .   /   php -S localhost:8000
```

There are no tests, linters, or build/compile steps in this repo.

## Architecture

Three files, no modules: `index.html` → `style.css` → `game.js`. All game logic lives in `game.js` as top-level functions operating on module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`).

Key mechanics to know before editing `game.js`:

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or `1–7` (a color index identifying which piece locked there). `COLORS[]` and `PIECES[]` are parallel arrays indexed by piece type.
- **Pieces as matrices**: each piece is a square matrix (not an offset table). Rotation is computed generically via `rotateCW` (transpose + row reversal), not via precomputed rotation states.
- **Collision (`collide`)** checks board bounds and already-locked cells; it's reused for movement, rotation, and ghost-piece projection — any change to piece/board representation must keep this signature compatible.
- **Wall kicks (`tryRotate`)**: after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until one doesn't collide, else the rotation is discarded.
- **Game loop (`loop`)**: driven by `requestAnimationFrame`, accumulates elapsed time in `dropAccum` and advances the piece one row once `dropAccum >= dropInterval`; `animId` is tracked so pause/game-over/restart can cancel and restart the frame loop cleanly.
- **Line clearing / scoring**: `clearLines` removes full rows bottom-up and unshifts empty rows at the top; scoring uses the classic table `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`. Level increases every 10 lines, and `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Ghost piece**: `ghostY` projects the current piece straight down until it would collide, and is called both for drawing (alpha 0.2) and for hard-drop placement — keep these two call sites consistent if changing drop logic.
- **Rendering** is immediate-mode: `draw()` clears and redraws the whole board canvas every frame (grid, locked cells, ghost, current piece); the next-piece preview has its own canvas/context (`nextCanvas`/`nextCtx`) and is redrawn only on `spawn()` via `drawNext()`.

Tunable constants at the top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, the `<canvas id="board">` `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`).
