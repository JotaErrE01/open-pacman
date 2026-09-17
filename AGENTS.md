# AGENTS.md

Vanilla JS + HTML + CSS Pac-Man clone. No dependencies, no build, no tests, no lint.

## Run

Open `src/index.html` directly in a browser, or serve statically:

```sh
python3 -m http.server -d src 8000
# -> http://localhost:8000
```

Canvas is fixed 560x620 (28 cols x 31 rows x 20px `TILE` in `src/js/render.js`).

## Structure (`src/`)

- `index.html` — loads scripts **in order**: `js/maze.js` → `js/game.js` → `js/render.js` → `js/main.js`
- `js/maze.js` — pristine level data: `MAZE_STR` (31 strings x 28 chars) parsed to `MAZE`, plus `TUNNEL_ROW=14`, `PACMAN_START`, `GHOST_STARTS`
- `js/game.js` — state + rules: `createGame()`, `update(game)`, movement, collisions
- `js/render.js` — canvas drawing: `draw(ctx, game, frame)`
- `js/main.js` — game loop (`requestAnimationFrame`), keyboard, overlay states
- `css/style.css` — fullscreen black layout, `#game-wrap` 560x620

## Conventions that break if ignored

- No ES modules / bundler. Files share state via `window.*` globals (`MAZE`, `createGame`, `update`, `DIRS`, `draw`). Keep the script order in `index.html`; do not convert one file to modules alone.
- Never mutate `MAZE`. `createGame()` copies it to mutable `game.grid`; rendering and dot-eating use `game.grid`.
- Tile codes: `0` empty, `1` wall, `2` dot, `3` pen door. Source chars in `maze.js`: `#` `.` ` ` `-`.
- Collision asymmetry: door (`3`) blocks Pac-Man only (`isWall` in `game.js`); ghosts pass through it.
- Movement is grid-aligned: actors only turn/decide when `aligned()` (epsilon `1e-3`). Speeds: Pac-Man `0.125`, ghost `0.1` cell/frame — tuned to re-align, do not use arbitrary floats.
- Tunnel wrap only applies on `TUNNEL_ROW` (row 14, open maze edges); `wrapTunnel` relies on it.
- `game.state`: `start | playing | won | lost`. `main.js` always draws, only calls `update()` when `playing`. `showOverlay()` rebuilds `#action-btn`, so re-query it after innerHTML replacement (already handled — follow that pattern).
- Ghost `kind`: `hunter` (Manhattan chase) vs `random`. Colors in `render.js` `GHOST_COLORS` cycle by ghost index.
- UI text is Spanish (`GANASTE`/`PERDISTE`, overlay copy). Keep it.

## Spec-driven workflow

- Skills `spec` and `spec-impl` are installed (`.agents/skills/`, see `skills-lock.json`). Load via the `skill` tool before using. This repo has no `specs/` folder yet — the first spec starts at `01-`.
- New feature: run `spec` skill first (`/spec <one-sentence description>`). It clarifies (Phase 2 questions mandatory), then writes `specs/NN-slug.md` in `Draft` state. It writes no code and never auto-implements.
- Implement: only after the user flips the spec state to `Approved`/`Aprobado`. Run `spec-impl` skill (`/spec-impl NN-slug`); it validates the state (anything else = stop), creates/switches to branch `spec-NN-slug` (config `specs/.spec-config.yml`, default `AutoCreateBranch: true`), then implements step-by-step with a pause per step for diff review. Never commit automatically.
- Spec file shape follows `.agents/skills/spec/template.md`: header (`Status/Depends on/Date/Objective` — one sentence), Scope In/Out, data model with real names, numbered runnable steps, boolean acceptance checklist, decisions taken/discarded. States: `Draft → In review → Approved → Implemented` (Spanish equivalents accepted).
