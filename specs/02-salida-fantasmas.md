# SPEC 02 — Salida escalonada de fantasmas sin re-entrada

> **Status:** Approved
> **Depends on:** SPEC 01
> **Date:** 2026-09-18
> **Objective:** Los 4 fantasmas salen del corral de forma escalonada y no vuelven a entrar por la puerta.

## Scope

**In:**

- Cambiar solo `src/js/game.js`: `createGame()`, `resetPositions()`, `isWall()/canMove()`, `moveGhost()`.
- Añadir `mode: 'pen'|'leaving'|'outside'` y `exitTimer` en frames por fantasma.
- Puerta (`3` en `y=12, x=13-14`) de un solo sentido para fantasmas.
- Mantener `MAZE` pristino, velocidades `0.125/0.1`, `aligned()` con épsilon `1e-3`, orden de scripts en `src/index.html`.

**Out of scope (for future specs):**

- Cambios en `maze.js` (`GHOST_STARTS`), `render.js`, `main.js`, CSS.
- Nueva IA de persecución, modos `scatter/chase`, power pellets / `frightened`.
- Velocidades distintas por fantasma o por zona.

## Data model

```js
// src/js/game.js — fantasma en createGame()
g = {
  x, y, dir: 'up', speed: 0.1, kind, // existentes (SPEC 01)
  mode: 'pen' | 'leaving' | 'outside',
  exitTimer: 0, // frames restantes en 'pen', decrementa en update()
};
// Timers por índice: [0, 120, 240, 360] (~0/2/4/6s a 60fps).
// Corral: x=[11,16], y=[13,15]; puerta: (13,12),(14,12); salida: y=11.
```

Convenciones:

- Coordenadas en celdas, origen arriba-izquierda.
- Decisión de dirección solo cuando `aligned()`.
- Puerta `3` bloquea a Pac-Man siempre.

## Implementation plan

1. En `createGame()` inicializar `mode` y `exitTimer` por índice. Prueba manual: cargar página, sin errores en consola.
2. En `isWall()/canMove()` bloquear puerta `3` a `ghost` con `mode==='outside'`, permitir con `mode==='leaving'`. Prueba manual: Pac-Man no cruza puerta, fantasma en `leaving` sí.
3. En `moveGhost()` bifurcar `pen` (bob vertical y descuento de `exitTimer`), `leaving` (centrar en `x=13|14` y subir a `y=11`) y `outside` (`decideGhost` actual). Prueba manual: fantasma 0 sale inmediato, resto escalonado.
4. En `resetPositions()` restaurar `x,y,mode,exitTimer,dir`. Prueba manual: tras perder vida la cadencia se repite.

## Acceptance criteria

- [ ] Al iniciar `playing`, el fantasma 0 está fuera del corral en <2s.
- [ ] Los 4 fantasmas están en `mode='outside'` (y≈11) en <8s.
- [ ] Durante 30s ningún fantasma `outside` pisa `(13,12)` ni `(14,12)`.
- [ ] Pac-Man no puede entrar al corral por la puerta.
- [ ] Tras perder una vida, timers y posiciones se reinician.
- [ ] Sin errores en consola; `won/lost`, dots y colisiones intactos; UI sigue en español.

## Decisions

- **Sí:** escalonada `[0,120,240,360]` frames. Fidelidad arcade y evita muerte instantánea.
- **No:** salida inmediata de los 4. Dificultad injusta al inicio.
- **Sí:** puerta unidireccional para fantasmas. Evita re-entradas y bucles.
- **No:** re-entrada permitida. Mantenía merodeo interno y el bug reportado.
- **Sí:** bob vertical en `pen` en vez de persecución. Desbloquea laterales `x=12,15` con mínimo cambio.
- **No:** tocar `GHOST_STARTS` o velocidades. Rompería `aligned()` y el tuning.

## What is **not** in this spec

- Power pellets y fantasmas comestibles.
- Alternancia scatter/chase por tiempo.
- Cambios visuales o de sonido.
- Velocidades distintas por fantasma.

Cada uno de esos, si aterriza, va en su propia spec.
