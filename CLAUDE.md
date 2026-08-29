# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Vanilla-JS Tetris. No build, no dependencies, no `package.json`, no test suite. Three source files: `index.html`, `style.css`, `game.js`. UI text is Spanish; keep it that way.

## Running

Open `index.html` directly, or serve statically:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

No lint or test commands exist.

## Architecture (`game.js`)

Single file, ~300 lines, module-global mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `animId`). No classes.

- **Board**: `ROWS × COLS` array of ints. `0` = empty; `1–7` = piece type, also index into `COLORS` and `PIECES`.
- **Pieces**: `PIECES[1..7]` are square matrices. Rotation = transpose + row-reverse (`rotateCW`), never mutates the source constant (`randomPiece` deep-copies).
- **Collision**: all geometry checks go through `collide(shape, offsetX, offsetY)` — returns true on out-of-bounds or overlap with a locked cell. Movement, rotation, drops, and game-over detection all call it.
- **Rotation kicks**: `tryRotate` tries x-offsets `[0,-1,1,-2,2]` before rejecting the spin.
- **Game loop**: `loop(ts)` via `requestAnimationFrame`; accumulates `dt` into `dropAccum`, drops one row when `dropAccum >= dropInterval`. `dropInterval` recomputed on level-up: `max(100, 1000 - (level-1)*90)`.
- **Lock cycle**: `lockPiece` = `merge` (write shape into board) → `clearLines` → `spawn`. `spawn` moves `next` into `current`, generates new `next`, calls `endGame` if the new piece already collides.
- **Scoring**: `LINE_SCORES` `[0,100,300,500,800]` × `level`; soft drop +1/row, hard drop +2/cell. Level = `floor(lines/10)+1`.
- **Rendering**: `draw` clears and redraws every frame — grid, locked board, ghost (`ghostY` + alpha 0.2), current piece. `drawNext` renders the preview canvas separately, only on spawn.
- **Input**: single `keydown` listener. `P` toggles pause always; other keys early-return when paused/over. Movement mutates `current.x/y` directly after a `collide` guard.

## Gotchas

- `COLS`, `ROWS`, `BLOCK` in `game.js` must match the `<canvas id="board">` `width`/`height` in `index.html` (`COLS*BLOCK` × `ROWS*BLOCK` = 300×600).
- `overlay` element is shared by PAUSA and GAME OVER states — check `gameOver` to tell them apart.
- No piece bag / no randomizer fairness — pure `Math.random`.
- `init()` is both first-boot and restart (wired to `restart-btn`).
