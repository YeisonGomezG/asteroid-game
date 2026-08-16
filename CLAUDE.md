# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A clone of the classic arcade game **Asteroids**, built with plain HTML5 Canvas and vanilla JavaScript (ES6+). No framework, no bundler, no dependencies, no package manager — the entire game logic lives in a single file, `game.js`.

## Running the game

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

There is no build step, no test suite, and no linter configured in this repo.

## Architecture (`game.js`)

Everything runs in one `requestAnimationFrame` loop (`loop` → `update(dt)` then `draw()`) operating on a fixed 800×600 canvas (`W`, `H`). Key structure, top to bottom:

- **Input**: raw key state tracked in `keys` (held) and `justPressed` (edge-triggered, consumed via `pressed(code)`).
- **Utils**: `wrap(v, max)` implements toroidal (screen-wrapping) space — used by every moving entity's `update()`. `dist`, `rand`, `randInt` are shared helpers.
- **Entity classes** (`Bullet`, `Asteroid`, `Ship`, `Particle`): each owns its own `update(dt)` and `draw()`, and is expected to set `this.dead = true` when it should be removed. Arrays of entities are pruned each frame with `.filter(x => !x.dead)`.
- **Asteroid sizing**: size is an integer 3 (large) → 1 (small); `RADII`, `SPEEDS`, `POINTS` are parallel arrays indexed by size. `Asteroid.split()` produces two smaller asteroids (or none at size 1).
- **Global mutable state** (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`) lives at module scope rather than in a class/store. `state` is one of `'playing' | 'dead' | 'gameover'` and gates most of `update()`.
- **Game flow**: `initGame()` resets everything and starts level 1; `nextLevel()` bumps `level` and spawns more asteroids (`3 + level`); `killShip()` handles life loss and transitions `state` to `'dead'` (brief pause, ship respawns invincible) or `'gameover'`.
- **Collision detection** is brute-force O(n·m) distance checks (bullets×asteroids, ship×asteroids) — fine at this entity count, but the first thing to reconsider if adding many more objects.

When adding a new entity type, follow the existing pattern: a class with `update(dt)`/`draw()`/`dead`, pushed into its own array, filtered each frame.
