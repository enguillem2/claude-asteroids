# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas + vanilla JS (ES6+). No bundler, no dependencies, no tests, no lint config. The README and code comments are in Spanish; keep that convention.

## Running

Open `index.html` directly in a browser, or serve statically (`npx serve .`, then http://localhost:3000). There is no build step.

## Architecture

All game logic lives in a single file, `game.js` (loaded by `index.html`), organized in sections:

- **Input**: `keys` (held) and `justPressed` (edge-triggered). Use `pressed(code)` for one-shot actions — it consumes the flag when read.
- **Entities**: classes `Bullet`, `Asteroid`, `Ship`, `Particle`, each with `update(dt)` / `draw()`; removal is by a `dead` flag, filtered out each frame in `update`.
- **Game state**: module-level globals (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`) and a `state` machine: `'playing' | 'dead' | 'gameover'`. `update(dt)` branches per state; `'dead'` is a respawn timer during which asteroids/particles keep moving.
- **Loop**: `requestAnimationFrame` loop with `dt` in seconds, clamped to 0.05.

Key conventions:
- Fixed logical canvas of `W`×`H` (800×600); space is toroidal — use `wrap(v, max)` for positions.
- Asteroid sizes are 1–3 (small→large); `RADII`, `SPEEDS`, `POINTS` are arrays indexed by size (index 0 unused). Asteroids split via `a.split()` into the next size down.
- Collisions are simple circle checks with `dist()`; ship vs asteroid uses `a.radius * 0.82` and is skipped while `ship.invincible > 0`.
- Power-ups and the "shooting star" were removed from the game (see git history) even though the README still mentions them. The one exception is the triple shot: `TriplePickup` always drops (on the first asteroid of size <= `TRIPLE_DROP_MAX_SIZE` destroyed) once per level (`tripleDropped`, reset in `nextLevel()`) when an asteroid is destroyed; picking it up sets `ship.tripleTimer` for `TRIPLE_DURATION` seconds, and dying clears it.
