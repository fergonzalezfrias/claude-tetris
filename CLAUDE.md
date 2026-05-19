# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Tetris clásico en JavaScript vanilla + HTML5 Canvas. Sin build, sin deps, sin tests.

## Ejecutar

`open index.html` o servidor estático: `python3 -m http.server 8000`.
No hay lint, build ni test runner — verificación es manual en navegador.

## Arquitectura (game.js, ~300 líneas, único módulo)

- Estado global mutable en variables sueltas (`board`, `current`, `next`,
  `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`...).
  No hay clases ni módulos: las funciones operan sobre estas globales.
- `init()` (re)inicializa todo el estado y arranca el loop; lo invoca también
  el botón Reiniciar. Es el único punto de reset.
- Game loop: `loop(ts)` con `requestAnimationFrame`; acumula `dt` en
  `dropAccum` y baja la pieza al superar `dropInterval`.
- `board`: matriz `ROWS×COLS`; cada celda es `0` o índice 1–7 (= color en
  `COLORS` y tipo en `PIECES`). Ese índice es la clave compartida entre forma,
  color y tablero.
- Rotación: `rotateCW` (transpuesta + reverso) + `tryRotate` con wall kicks
  `[0,-1,1,-2,2]`. `collide` es la única función de colisión (la usan
  movimiento, rotación, ghost, spawn y game over).
- Ciclo de pieza: `spawn()` → si `collide` al aparecer → `endGame()`;
  `lockPiece()` = `merge()` + `clearLines()` + `spawn()`.

## Invariante crítico

`COLS·BLOCK` y `ROWS·BLOCK` deben coincidir con `width`/`height` del
`<canvas id="board">` en index.html (hoy 10·30=300, 20·30=600). Cambiar
`COLS`/`ROWS`/`BLOCK` en game.js sin actualizar index.html rompe el render.
El panel `NEXT` asume rejilla 4×4 (`drawNext`).
