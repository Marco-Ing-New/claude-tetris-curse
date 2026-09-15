# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Classic Tetris implemented in vanilla JavaScript with HTML5 Canvas and CSS. No dependencies, no build tool, no package.json — just three files: `index.html`, `style.css`, `game.js`.

## Running / testing

There is no build step and no test suite. To run the game, open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000`. Changes to `game.js`/`style.css`/`index.html` take effect on browser reload — no compilation needed.

## Architecture

All game logic lives in `game.js` (single file, no modules). Key pieces:

- **Board model**: `board` is a `ROWS × COLS` matrix where each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: defined in `PIECES` as square matrices. Rotation (`rotateCW`) is a transpose + row-reverse, not a lookup table of rotation states.
- **Collision** (`collide`): checks board bounds and existing locked cells for a shape at a given offset.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until a non-colliding position is found, else the rotation is discarded.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and advances the piece one row when it exceeds `dropInterval`.
- **Line clearing** (`clearLines`): scans bottom-to-top, splices full rows out and unshifts empty rows at the top.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by current `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row.
- **Level/speed**: level increments every 10 lines; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at `globalAlpha = 0.2`.

Global mutable game state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, timing vars) lives in module-level `let` bindings and is reset by `init()`.

### Control flow

```
init() → createBoard(), spawn first pieces, requestAnimationFrame(loop)
loop(ts) → accumulate dt → drop piece or lockPiece() when dropInterval exceeded → draw() → re-schedule
lockPiece() → merge() into board → clearLines() → spawn() next piece
spawn() → if the new piece immediately collides, endGame()
keydown handler → move / tryRotate / softDrop / hardDrop / togglePause (ignored while paused/gameOver, except P)
```

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell pixel size), `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).

## File responsibilities

- `index.html` — DOM structure: main board canvas (300×600), side panel (score/lines/level/next-piece preview/controls), pause/game-over overlay. `lang="es"`; UI copy (labels, overlay text, restart button) is in Spanish — keep new user-facing strings consistent with that.
- `style.css` — dark/retro arcade visual theme.
- `game.js` — all game logic and rendering (~300 lines, no modules/classes).
