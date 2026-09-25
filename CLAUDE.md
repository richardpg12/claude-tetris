# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla-JS Tetris rendered with HTML5 Canvas. No build step, no dependencies, no
package.json, no tests, no linter. Three source files: `index.html`, `style.css`,
`game.js`. UI strings are in Spanish.

## Running

Open `index.html` directly in a browser, or serve statically:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

## Architecture (`game.js`)

The whole game is one IIFE-free module-scoped script; `init()` runs on load and on
the restart button. Key pieces of the design:

- **Board model**: `board` is a `ROWS`×`COLS` array of ints. `0` = empty; `1`–`7`
  index into `COLORS` and `PIECES`, identifying which tetromino filled the cell.
- **Pieces**: `PIECES[type]` is a square matrix. Rotation (`rotateCW`) is
  transpose + row-reverse and produces a new matrix each call. `current` holds
  `{ type, shape, x, y }`; `shape` is mutated in place on rotation.
- **Wall kicks**: `tryRotate` tests horizontal offsets `[0, -1, 1, -2, 2]` against
  the rotated shape and applies the first that doesn't `collide`. This is not full
  SRS — it's a simplified kick table.
- **Collision**: `collide(shape, ox, oy)` is the single gatekeeper for every move,
  rotation, drop, ghost projection, and the game-over check in `spawn`.
- **Game loop**: `loop(ts)` via `requestAnimationFrame`. Accumulates `dt` into
  `dropAccum`; when it exceeds `dropInterval` the piece drops one row or locks.
  `dropInterval` is recomputed in `clearLines` as `max(100, 1000 - (level-1)*90)`.
- **Locking flow**: `lockPiece` → `merge` (stamp shape into `board`) → `clearLines`
  → `spawn` (promote `next` to `current`, roll a new `next`, check game over).
- **Rendering**: `draw()` clears and repaints every frame — grid, board, ghost
  (`ghostY` + alpha 0.2), then the active piece. `drawNext` paints the preview
  canvas and is called only from `spawn`. HUD (`updateHUD`) is DOM text, updated
  on scoring events and after each keydown, not per frame.
- **State**: all game state is top-level `let` bindings reset by `init()`. Pause
  and game-over share the `#overlay` element and gate the keydown handler.

## Coupling to watch for

`COLS`, `ROWS`, `BLOCK` in `game.js` must stay in sync with the `<canvas id="board">`
`width`/`height` in `index.html` (`width = COLS*BLOCK`, `height = ROWS*BLOCK`). The
next-piece preview assumes a 4×4 area at 30px (`NB` in `drawNext`, `120` canvas).
