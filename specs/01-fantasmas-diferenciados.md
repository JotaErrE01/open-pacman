# SPEC 01 — Cuatro fantasmas con personalidades clásicas diferenciadas

> **Status:** Approved
> **Depends on:** Ninguna
> **Date:** 2026-09-17
> **Objective:** Dotar al juego de 4 fantasmas con comportamientos distintos de estilo arcade clásico, incluyendo uno agresivo que persigue directamente a Pac-Man.

## Scope

**In:**

- Ampliar `GHOST_STARTS` en `src/js/maze.js` de 2 a 4 fantasmas, todos dentro de la jaula, libres desde el frame 1.
- Implementar 4 `kind` en `src/js/game.js`: `aggressive`, `ambusher`, `erratic`, `shy`, con targeting clásico exacto sobre `decideGhost`.
- Mantener reglas actuales: misma velocidad `0.1` celda/frame, sin giro de 180º salvo callejón, decisión solo con `aligned()`, fantasmas atraviesan puerta `3`.
- Color fijo por rol en `src/js/render.js`: rojo `aggressive`, rosa `ambusher`, cian `erratic`, naranja `shy`.

**Out of scope (for future specs):**

- Modos `scatter/chase` alternados por temporizador.
- Power pellets / modo `frightened` / comerse fantasmas.
- Salida escalonada de la jaula con retardos o contadores.
- Velocidades distintas por fantasma.

## Data model

```js
// src/js/maze.js
const GHOST_STARTS = [
  { x: 12, y: 13, kind: 'aggressive' },
  { x: 13, y: 13, kind: 'ambusher' },
  { x: 14, y: 13, kind: 'erratic' },
  { x: 15, y: 13, kind: 'shy' },
];
// Nota: validar en implementación que las 4 celdas son transitables
// (valor 0). Si alguna es pared, ajustar dentro del pen (filas 13-14).

// src/js/game.js — fantasma en createGame()
ghosts: [{ x, y, dir: 'up', speed: 0.1, kind: 'aggressive' /* | 'ambusher' | 'erratic' | 'shy' */ }]

// Targets (celda objetivo, origen arriba-izquierda):
// aggressive: { px, py } de Pac-Man redondeado.
// ambusher: Pac-Man + 4 * DIRS[pacman.dir].
// erratic: pivot = Pac-Man + 2 * DIRS[pacman.dir]; target = 2*pivot - pos(aggressive).
// shy: si dist(fantasma, Pac-Man) > 8 → como aggressive; si no → esquina { x: 1, y: 29 }.
```

Convenciones:

- Coordenadas en celdas, origen arriba-izquierda.
- Decisión de dirección solo cuando `aligned()` con épsilon `1e-3`.
- Selección por distancia Manhattan, excluyendo `OPPOSITE[dir]` salvo callejón.

## Implementation plan

1. Ampliar `GHOST_STARTS` a 4 entradas con los `kind` nuevos en `src/js/maze.js`. Prueba manual: cargar página, contar 4 fantasmas en la jaula.
2. Extender `decideGhost` en `src/js/game.js` con `targetFor(kind)` y selección Manhattan. Prueba manual: el `aggressive` recorta distancia a Pac-Man en pasillo recto.
3. Implementar `ambusher` (4 tiles por delante) y `shy` (persigue a >8, huye a `{1,29}` si no). Prueba manual: con Pac-Man quieto mirando, el `ambusher` se posiciona por delante.
4. Implementar `erratic` usando la posición del `aggressive` como referencia. Prueba manual: su giro cambia cuando el `aggressive` se mueve, no solo cuando Pac-Man lo hace.
5. Fijar color por `kind` en `src/js/render.js` (mapa `kind → color`, no índice). Prueba manual: cada rol conserva su color tras perder una vida (`resetPositions`).

## Acceptance criteria

- [ ] La página carga sin errores en consola con 4 fantasmas visibles.
- [ ] Existe exactamente un fantasma `aggressive` de color rojo que reduce su distancia Manhattan a Pac-Man cuando tiene camino libre.
- [ ] El fantasma `ambusher` rosa apunta a 4 celdas por delante de la dirección de Pac-Man.
- [ ] El fantasma `erratic` cian calcula su objetivo a partir del `aggressive` (doble del vector desde el agresivo al punto 2-delante).
- [ ] El fantasma `shy` naranja persigue a más de 8 celdas y se dirige a la esquina `{1,29}` a 8 o menos.
- [ ] Los 4 arrancan dentro de la jaula y se mueven desde el inicio sin quedarse atascados en la puerta.
- [ ] Tras colisión con pérdida de vida, los 4 conservan `kind` y color al reaparecer.
- [ ] Textos de UI siguen en español y el orden de scripts en `index.html` no cambia.

## Decisions

- **Sí:** targeting clásico exacto (4 delante / 2+doble vector / 8 de corte). Da variedad real con lógica Manhattan ya existente.
- **No:** versión simplificada de targeting. Se descartó porque no diferenciaría lo suficiente los 4 roles.
- **Sí:** misma velocidad `0.1` para los 4. Conserva el re-alineado del grid y el balance actual.
- **No:** velocidad por fantasma. Cambiaría la dificultad y exigiría re-ajustar `aligned()`.
- **Sí:** todos libres a la vez. Evita temporizadores de salida en esta spec.
- **No:** salida escalonada. Va a otra spec si se quiere fidelidad total.
- **Sí:** `kind` descriptivos (`aggressive`, `ambusher`, `erratic`, `shy`). Autoexplicativos frente a `blinky/pinky/inky/clyde`.
- **No:** modos scatter/frightened/power pellets. Otra spec si aterrizan.
- **Sí:** color fijo por `kind`. El jugador distingue comportamientos de un vistazo.

## What is **not** in this spec

- Alternancia scatter/chase por tiempo.
- Power pellets y fantasmas comestibles.
- Salida escalonada de la jaula.
- Velocidades o sonidos distintos por fantasma.

Cada uno de esos, si aterriza, va en su propia spec.
