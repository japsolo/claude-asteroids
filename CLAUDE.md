# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Clone of the classic arcade game Asteroids, built with pure HTML5 canvas and vanilla JavaScript (ES6+). No dependencies, no bundler, no build step, no tests — the entire game lives in one file, `game.js`.

## Running

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

There is no build, lint, or test command — there is nothing to compile and no test suite.

## Architecture

Everything is in `game.js`, organized into sections (marked by `── Section ──` comments) in this order:

1. **Input** — a global `keys`/`justPressed` map populated by `keydown`/`keyup` listeners. `pressed(code)` consumes a one-shot "just pressed" event (used for firing/restart so holding the key doesn't repeat).
2. **Entity classes** — `Bullet`, `Asteroid`, `Ship`, `Particle`. Each has `update(dt)` and `draw()`, and sets `this.dead = true` when it should be removed. `Asteroid.split()` returns the two smaller asteroids spawned when one is destroyed (sizes 3→2→1, see `RADII`/`SPEEDS`/`POINTS` arrays indexed by size).
3. **Game state** — module-level `let` variables (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`, `deadTimer`) rather than a state object/class. `state` is one of `'playing' | 'dead' | 'gameover'`.
4. **`update(dt)`** — branches on `state` first (gameover/dead have their own short-circuited logic), then otherwise: reads input, updates all entities, filters dead ones out of arrays, does collision detection (bullet↔asteroid, ship↔asteroid), and advances to `nextLevel()` when all asteroids are cleared.
5. **`draw()`** — clears the canvas and draws particles → asteroids → bullets → ship → HUD, in that order (back to front).
6. **Main loop** — `requestAnimationFrame` loop computing `dt` in seconds, clamped to 0.05 to avoid large jumps after tab-switching.

Key conventions to preserve when editing:

- The play field is toroidal: `wrap(v, max)` wraps both position axes, so movement/spawn logic must account for edge wraparound (see how `Bullet`, `Asteroid`, and `Ship` all wrap `x`/`y` in `update`).
- Entities never remove themselves from arrays directly; they set `dead = true` and the owning array is filtered in `update()`.
- New entity types should follow the same `update(dt)`/`draw()`/`dead` pattern used by existing classes so they compose with the existing loop and filtering logic.
- All game constants (speeds, radii, points, thrust, drag, cooldowns) are defined near their relevant class/section rather than centralized — keep new tunables close to the code that uses them, consistent with the current style.
