# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

No build/test/lint pipeline exists (no `package.json`, no dependencies). To run the game:

```bash
open index.html        # or just open in a browser directly
python3 -m http.server  # or any static server, then visit localhost
```

## Architecture

Vanilla JS Tetris: `index.html` (DOM/canvas) + `style.css` (dark theme) + `game.js` (~300 lines, all logic). No classes/modules — one global mutable state block (`board, current, next, score, lines, level, paused, gameOver, ...`) reset by `init()`.

- **Board model**: `board` is a `ROWS × COLS` matrix; `0` = empty, `1`-`7` = color index. `PIECES`/`COLORS` are parallel arrays indexed by piece type.
- **Collision** (`collide`) is the single source of truth for movement, rotation, ghost-piece projection, and spawn-collision (which triggers game over).
- **Rotation**: `rotateCW` transposes+reverses the shape matrix; `tryRotate` applies wall-kick offsets `[0, -1, 1, -2, 2]` until one doesn't collide.
- **Game loop**: `requestAnimationFrame`-driven `loop()` accumulates `dt` against `dropInterval`. On drop timeout, `lockPiece()` chains `merge()` → `clearLines()` → `spawn()`.
- **Scoring/level/speed are coupled**: `clearLines()` scores via `LINE_SCORES[cleared] * level`, recomputes `level` from total `lines`, and derives `dropInterval` from `level` (`max(100, 1000 - (level-1)*90)`). Changing one formula affects the others.
- **Rendering**: `draw()` fully redraws the board + ghost piece + current piece on `#board` canvas every frame (no dirty-rect diffing); `drawNext()` separately renders the preview on `#next-canvas`.

See README.md for gameplay features, controls, and tunable constants (`COLS`, `ROWS`, `BLOCK`, etc.).
