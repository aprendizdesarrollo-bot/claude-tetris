# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris using HTML5 Canvas. No dependencies, no build process, no package manager — just three files: `index.html`, `style.css`, `game.js`.

## Running

Open `index.html` directly in a browser, or serve it with any static server (e.g. `npx serve .`, `python3 -m http.server 8000`). There is no build, lint, or test step — the code runs as-is.

## Architecture

All game logic lives in `game.js` as top-level functions operating on module-scope state (`board`, `current`, `next`, `score`, `lines`, `level`, etc.) — there are no classes or modules.

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: defined in `PIECES` as square matrices. Rotation (`rotateCW`) is a transpose + row reversal, not stored rotation states.
- **Collision** (`collide`): checks a shape against board bounds and already-locked cells.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` until one doesn't collide, else the rotation is discarded.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and drops the piece one row once it exceeds `dropInterval`.
- **Locking** (`lockPiece` → `merge` + `clearLines` + `spawn`): merges the current piece into `board`, clears completed rows (shifting from the bottom up), then spawns the next piece.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 points/row dropped, soft drop 1 point/row.
- **Level/speed**: level increments every 10 lines; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece**: `ghostY()` projects the current piece straight down to its landing row; drawn with `globalAlpha = 0.2`.
- **Game over**: triggered in `spawn()` when a freshly spawned piece immediately collides.

Rendering (`draw`, `drawNext`, `drawBlock`, `drawGrid`) is plain Canvas 2D — no external libraries.

## Tunable constants (in `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell size in px), `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, the `#board` canvas `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`).
