# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

An Asteroids clone built with plain HTML5 Canvas and ES6+ JavaScript. It has no framework, bundler, package.json, dependencies, tests or linter. All source lives in `claude-asteroids-main/`: `index.html` (an 800×600 `<canvas id="canvas">` that loads `game.js`), `game.js` (all game logic) and `favicon.svg`.

Code comments, UI strings (`NIVEL`, `PUNTAJE`, `ESPACIO PARA REINICIAR`) and the README are in Spanish. Keep new comments and in-game text in Spanish.

## Running

Open `claude-asteroids-main/index.html` in a browser, or serve the folder:

```bash
cd claude-asteroids-main
npx serve .   # then visit http://localhost:3000
```

There is no build step. Edits to `game.js` take effect when you reload the page.

## Architecture (`game.js`)

`game.js` is a single script with no modules. Everything is a global at script scope. Sections are separated by `// ── Name ──` banner comments, in this order: Input, Utils, Bullet, Asteroid, Ship, Particles, game state, Update, Draw, main loop.

- **Main loop**: `loop(ts)` runs on `requestAnimationFrame`. It computes `dt` in seconds, capped at 0.05, then calls `update(dt)` and `draw()`. All motion is in px/s scaled by `dt`, so new movement code must use `dt` too.
- **Entities**: `Ship`, `Asteroid`, `Bullet` and `Particle` are classes with `update(dt)` and `draw()` methods. They draw directly to the global `ctx`. Removal works with a `dead` flag, and the global arrays are re-filtered each frame (`bullets = bullets.filter(b => !b.dead)`).
- **Toroidal space**: positions wrap with `wrap(v, max)` on the constants `W` and `H`. Particles are the exception and do not wrap.
- **State machine**: the global `state` is `'playing'`, `'dead'` or `'gameover'`.
  - `killShip()` decrements `lives`. It moves to `'dead'`, with `deadTimer` set to 2s before respawn, or to `'gameover'`.
  - In `'gameover'`, pressing Space calls `initGame()`.
  - `nextLevel()` runs when `asteroids` is empty and spawns `3 + level` large asteroids.
- **Asteroid sizes** are integers 3 (large), 2 (medium) and 1 (small), used as indexes into the parallel arrays `RADII`, `SPEEDS` and `POINTS`. Index 0 is unused. `split()` returns two asteroids of `size - 1`.
- **Input**: `keys[code]` holds held keys and is used for rotation and thrust. `pressed(code)` is a one-shot edge trigger that consumes `justPressed[code]`; use it for actions like shooting or restarting.
- **Collisions** use circle distance via `dist(a, b)`. Ship-vs-asteroid uses `a.radius * 0.82` for forgiveness, and is skipped while `ship.invincible > 0`. The ship gets 3s of invincibility after `reset()` and blinks during it.

## README vs. code

The README mentions power-ups and special asteroid types such as a "shooting star" (estrella fugaz). These are **not implemented** in `game.js`. Treat them as planned features, not existing code.
