# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris in vanilla JavaScript + HTML5 Canvas. No dependencies, no build step, no tests, no linter. README.md and UI text are in Spanish; keep new user-facing strings in Spanish.

## Running

Open `index.html` directly in a browser, or serve the folder (e.g. `python3 -m http.server 8000`) and visit `http://localhost:8000`.

## Architecture

Three files, with all logic in `game.js` (a single classic script, `'use strict'`, no modules, global state):

- `index.html` defines the DOM that `game.js` looks up by id at load time (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`). Renaming an id requires updating `game.js`.
- `style.css` handles layout/theme only. The overlay is shown/hidden via the `hidden` class.
- `game.js` is organised around module-level mutable variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, ...) reset by `init()`.

Game flow:
- `init()` → `requestAnimationFrame(loop)`. `loop` accumulates `dt` into `dropAccum` and gravity-drops or locks the piece when it exceeds `dropInterval`, then calls `draw()`.
- Pieces are `{type, shape, x, y}`. `PIECES[type]` is the shape matrix whose non-zero cell value is the index into `COLORS`, so shape, colour and piece id are the same number (index 0 is `null` in both arrays).
- Lock path: `lockPiece()` = `merge()` → `clearLines()` → `spawn()`. `spawn()` promotes `next`, generates a new one, and calls `endGame()` if the spawn position collides.
- `clearLines()` owns scoring, level (`floor(lines/10)+1`) and `dropInterval` (`max(100, 1000 - (level-1)*90)`). Soft drop gives +1/cell and hard drop +2/cell, added directly in `softDrop()`/`hardDrop()`.
- Rotation (`tryRotate`) is clockwise only, with horizontal wall kicks `[0, -1, 1, -2, 2]`; no SRS, no floor kicks.
- Pause/game over both stop the animation loop with `cancelAnimationFrame` and reuse the overlay; `togglePause()` restarts `loop` on resume, and the restart button calls `init()`.
- Piece selection is uniform `Math.random()` (no 7-bag).
