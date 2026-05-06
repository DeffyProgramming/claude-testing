# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the games

No build step. Open any `.html` file directly in a browser (`start tictactoe.html` or `start tower-defense.html` on Windows). All logic, styles, and markup are self-contained in each file.

## Architecture

Every game is a **single HTML file** — one `<style>` block, static HTML, and one `<script>` block. No frameworks, no modules, no external dependencies.

### tower-defense.html

The script is organized into labeled sections in this order:

1. **Constants** — canvas dimensions, `CELL` size (40px), `SPAWN_INTERVAL`
2. **`TOWER_DEFS` / `ENEMY_DEFS`** — plain objects keyed by type string; all game balance lives here
3. **`WAVES`** — 50-entry array of `[{type, count}]` groups; edit this to tune difficulty
4. **`PATH_WAYPOINTS`** — axis-aligned pixel coords; `buildPathCells()` derives the blocked grid cells from them at init
5. **`state`** / `freshState()` — single mutable object holding all runtime state (phase, lives, gold, arrays of towers/enemies/projectiles/effects, spawn queue)
6. **Update functions** — `updateSpawner`, `updateEnemies`, `updateTowers`, `updateProjectiles`, `updateEffects`; each takes `dt` in milliseconds
7. **`render()`** — draws everything to canvas in a fixed layer order; called every frame regardless of phase
8. **`gameLoop(timestamp)`** — `requestAnimationFrame` loop; only runs update functions when `state.phase === 'wave'`
9. **UI / input handlers** — DOM event listeners wired at module level
10. **`initGame()`** — resets state and starts the loop

**Delta-time normalization:** All pixel movement uses `speed * (dt / 16.67)` to stay frame-rate independent. `dt` is capped at 100ms to prevent teleportation on tab blur.

**Damage pipeline:** `dealDamage(baseDmg, towerType, enemy)` applies armor/boss multipliers. Armored enemies take 40% from `arrow`/`sniper`. Boss enemies take 35% from everything. Both flags live in `ENEMY_DEFS`.

**Chain tower** does not fire a projectile — it resolves hits instantly in `fireChain()` and pushes a `lightning` visual effect. All other towers push to `state.projectiles` which home on `targetEnemy` by reference.

**Adding a tower type:** Add an entry to `TOWER_DEFS`. If `chainCount > 0`, `updateTowers` routes to `fireChain`; if `splash > 0`, `applyHit` does AoE; `slowFactor > 0` applies slow on hit. No other code needs changing.

**Adding an enemy type:** Add an entry to `ENEMY_DEFS` and reference the key in `WAVES`. Set `armored: true` to halve arrow/sniper damage, `slowImmune: true` to block slow, `boss: true` for the 35% damage cap.

### tictactoe.html

Minimal DOM-based game. `board` is a flat 9-element array. `applyMove` handles both human and computer turns. Hard AI uses `minimax` with depth scoring (`10 - depth` for wins) so it prefers faster wins. Easy AI is random.

## Shared conventions

- **Color palette:** background `#1a1a2e` / `#0d0d0d`, accent red `#e94560`, font `'Courier New', monospace`
- **Overlay pattern:** win/lose screens are a fixed-position `<div id="overlay">` injected with `innerHTML` at game-end, not canvas-drawn
- **Stats display:** DOM `<span>` elements updated imperatively by `updateStatsDisplay()` — not reactive
- **No classes:** state is plain objects; behavior is free functions
