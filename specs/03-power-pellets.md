# SPEC 03 — Power pellets en las 4 esquinas con modo frightened clásico

> **Status:** Implemented
> **Depends on:** SPEC 01, SPEC 02
> **Date:** 2026-09-18
> **Objective:** Añadir 4 power pellets en las esquinas que activan un modo frightened donde los fantasmas huyen, son comestibles en cadena y luego vuelven a la normalidad.

## Scope

**In:**

- 4 pellets en esquinas de `src/js/maze.js` con tile nuevo `4` (char fuente `o`), reemplazando dots existentes.
- Estado `frightTimer` + cadena `ghostChain` en `src/js/game.js`; comer pellet da 50 pts y activa 420 frames (~7s a 60fps).
- Durante `frightened`: fantasmas en `outside` usan movimiento aleatorio a velocidad lenta `0.05`, son comestibles (200/400/800/1600 en cadena); la colisión no quita vida.
- Fantasma comido: respawn en el corral en `mode='pen'` con `exitTimer=120`, re-sale con la lógica de SPEC 02.
- Re-comer pellet con frightened activo: reinicia `frightTimer` a 420 y la cadena a 200.
- Visual en `src/js/render.js`: pellet grande (radio ~5.5) con parpadeo por `frame`; fantasma azul `#2121de` y blanco `#ffffff` parpadeando los últimos 120 frames.
- Pellets cuentan para `dotsRemaining`/`won`; los ya comidos no reaparecen tras perder vida; al perder vida el `frightTimer` se cancela y la cadena se reinicia.

**Out of scope (for future specs):**

- Niveles múltiples o aumento de dificultad por nivel.
- Sonido o música del modo frightened.
- Puntuación flotante / textos de puntos sobre el fantasma comido.
- Modo `scatter/chase` por temporizador.

## Data model

```js
// src/js/maze.js
// MAZE_STR: sustituir '.' por 'o' en (1,1), (26,1), (1,29), (26,29).
// parseTile: 'o' → 4. Códigos: 0 vacío, 1 muro, 2 dot, 3 puerta, 4 pellet.
const POWER_PELLETS = [{x:1,y:1},{x:26,y:1},{x:1,y:29},{x:26,y:29}];

// src/js/game.js — en createGame()
game = {
  // ...existente (SPEC 01/02),
  frightTimer: 0,   // frames restantes de frightened, decrementa en update()
  ghostChain: 0,    // índice 0..3 dentro del frightened actual → 200<<chain
};
// fantasma comido: g.mode='pen', g.exitTimer=120, g.x,g.y = celda libre del corral.
```

Convenciones:

- Coordenadas en celdas, origen arriba-izquierda.
- `frightened = frightTimer > 0`.
- Decisión de dirección solo cuando `aligned()` con épsilon `1e-3`.
- Velocidades: Pac-Man `0.125`, fantasma normal `0.1`, fantasma frightened `0.05` (1/20 celda/frame, re-alinea cada 20 frames).
- Orden de scripts en `src/index.html` no cambia; `MAZE` sigue pristino.

## Implementation plan

1. En `src/js/maze.js`: cambiar 4 `.` por `o` en las esquinas y mapear `o → 4` en `parseTile`; exponer `POWER_PELLETS`. Prueba manual: cargar página, 4 celdas con valor `4`.
2. En `src/js/game.js` `createGame()`: contar `dotsRemaining` como `v===2 || v===4`; añadir `frightTimer: 0, ghostChain: 0`. Prueba manual: `dotsRemaining` inicial = dots anteriores + 4.
3. En `movePacman()`: al pisar `4`, poner celda a `0`, `score += 50`, `dotsRemaining--`, `frightTimer=420`, `ghostChain=0`. Prueba manual: comer esquina suma 50 y los fantasmas se vuelven azules.
4. En `update()`: decrementar `frightTimer` cada frame; en `decideGhost()` usar elección aleatoria si `frightened`; en `moveGhost()` usar `speed 0.05` si `frightened` y `mode==='outside'`. Prueba manual: durante ~7s los fantasmas deambulan lento.
5. En `update()` colisiones: si `frightened` y colisión con fantasma en `outside`/`leaving` → `score += 200<<ghostChain` (cap 1600), `ghostChain++`, respawn del fantasma en corral con `exitTimer=120`; si no hay `frightened`, regla actual de perder vida. Prueba manual: comer 2 fantasmas seguidos da 200 + 400.
6. En `resetPositions()`: cancelar `frightTimer=0`, `ghostChain=0`, no restaurar pellets comidos. Prueba manual: perder vida quita el azul inmediatamente.
7. En `src/js/render.js`: `drawDots()` dibuja `4` como círculo grande con parpadeo (`frame`); `draw()` pinta fantasma de azul/blanco según `game.frightTimer` (blanco intermitente si `<120`). Prueba manual: se ven 4 cocos grandes y el aviso blanco al final.

## Acceptance criteria

- [ ] La página carga sin errores con 4 pellets visibles en las esquinas.
- [ ] Comer un pellet suma exactamente 50 y pone `frightTimer` a 420.
- [ ] Durante frightened la colisión con fantasma no resta vidas.
- [ ] Comer fantasmas en el mismo frightened suma 200/400/800/1600 en orden.
- [ ] Comer un segundo pellet reinicia el timer a 420 y la cadena al primer valor (200).
- [ ] El fantasma comido reaparece en el corral y vuelve a salir en <3s.
- [ ] Tras ~7s los fantasmas recuperan color e IA de SPEC 01 sin errores.
- [ ] Los últimos ~2s los fantasmas parpadean en blanco.
- [ ] Comer todos los dots + los 4 pellets lleva a `won`; los pellets comidos no reaparecen tras perder vida.
- [ ] UI sigue en español y el orden de scripts no cambia.

## Decisions

- **Sí:** esquinas extremas `(1,1),(26,1),(1,29),(26,29)` reemplazando dots. Mínimo cambio al mapa, simetría garantizada.
- **No:** coordenadas clásicas interiores (p. ej. fila 3/23). Exigiría revalidar pasillos del mapa actual.
- **Sí:** tile nuevo `4` con char `o`. No rompe los códigos `0-3` existentes.
- **Sí:** 420 frames (~7s) con aviso blanco final de 120 frames. Fidelidad arcade con aviso legible.
- **No:** duración fija sin aviso. El jugador no sabría cuándo termina.
- **Sí:** cadena clásica 200/400/800/1600 con reinicio al nuevo pellet. Comportamiento arcade esperado.
- **No:** 200 plano por fantasma. Menos arcade y menos incentivo de riesgo.
- **Sí:** fantasma frightened a `0.05` (mitad). Lento pero re-alinea el grid (1/20).
- **No:** mantener `0.1` en frightened. No daría ventaja real a Pac-Man.
- **Sí:** fantasma comido al corral con `exitTimer=120`. Reutiliza SPEC 02 sin ramas nuevas.
- **Sí:** pellets cuentan para victoria y no se restauran; `frightTimer` se cancela al perder vida. Regla simple y predecible.

## What is **not** in this spec

- Niveles o dificultad progresiva.
- Sonidos del modo frightened.
- Textos flotantes de puntos.
- Alternancia scatter/chase por tiempo.

Cada uno de esos, si aterriza, va en su propia spec.
