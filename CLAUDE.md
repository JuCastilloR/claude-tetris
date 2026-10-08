# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris in vanilla JavaScript + HTML5 Canvas. No dependencies, no build step, no package manager, no tests, no linter. The README and UI text are in Spanish.

## Running

Open `index.html` directly in a browser, or serve the folder (e.g. `python -m http.server 8000`). Edit and reload; there is nothing to compile.

## Architecture

Three files: `index.html` (DOM: `#board` 300x600 canvas, `#next-canvas` 120x120, HUD spans, `#overlay`), `style.css`, and `game.js` (all logic, one script, strict mode, global state).

Key points in `game.js` that span multiple functions:

- **Global mutable state** (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, ...) is declared on one line and reset in `init()`. Restart calls `init()` again, which cancels the previous animation frame.
- **Board encoding**: `board[row][col]` is `0` for empty or a piece type `1..7`. That same integer indexes both `PIECES` (shape matrices) and `COLORS`, so adding/reordering pieces means updating both arrays in sync (and `randomPiece`'s hard-coded `7`).
- **Game loop**: `loop(ts)` via `requestAnimationFrame` accumulates `dropAccum` and does gravity once it passes `dropInterval`, then calls `draw()`. Pause/game over stop the loop with `cancelAnimationFrame`; unpausing restarts it by calling `loop` directly.
- **Piece lifecycle**: `lockPiece()` = `merge()` → `clearLines()` → `spawn()`. `spawn()` promotes `next`, generates a new `next`, and calls `endGame()` if the new piece collides. Both gravity (`loop`) and soft/hard drop funnel into `lockPiece()`.
- **Collision**: `collide(shape, ox, oy)` treats `ny < 0` as free (pieces may sit above the top) and out-of-bounds sides/bottom as solid. Rotation (`tryRotate`) is clockwise only, with simple horizontal kicks `[0,-1,1,-2,2]`.
- **Scoring/speed** live in `clearLines()`: `LINE_SCORES[cleared] * level`, level = `floor(lines/10)+1`, `dropInterval = max(100, 1000 - (level-1)*90)`. Soft drop gives +1/cell, hard drop +2/cell.
- **Input**: a single `keydown` listener (arrows, `X` rotate, `Space` hard drop, `P` pause). It returns early when the event target is `#name-input`.
- **Start screen / records**: the game no longer starts on load; the script ends with `showStart()` (overlay with the records table and a "Jugar" button that calls `init()`). Top 5 `{name, score, lines, combo, date}` plus historical `bestCombo`/`maxLines` live in `localStorage` key `tetris-records` (all access wrapped in try/catch). `endGame()` updates the historical stats and, if the score qualifies, shows `#name-form`; `submitRecord()` saves it. `combo`/`maxCombo` are tracked in `clearLines()` (consecutive locking pieces that clear lines; reset otherwise). The shared overlay uses `showOverlay(title, scoreText, withRecords)`.
