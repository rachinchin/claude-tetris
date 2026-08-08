# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris clásico implementado en JavaScript vanilla puro, HTML5 Canvas y CSS. Sin dependencias, sin `package.json`, sin bundler ni transpilador. Todo el código vive en 3 archivos: `index.html`, `style.css`, `game.js` (~300 líneas).

## Comandos

No hay build, lint ni tests — no existe toolchain de npm. Para ejecutar el juego:

```bash
# Opción 1: abrir directamente
start index.html       # Windows

# Opción 2: servidor estático local (recomendado para evitar restricciones de file://)
python3 -m http.server 8000
npx serve .
```

Luego abrir `http://localhost:8000`.

## Arquitectura

Toda la lógica del juego está en `game.js`, un único script sin módulos que se ejecuta al cargar la página (`init()` al final del archivo).

- **Tablero**: matriz `ROWS × COLS` (20×10) donde cada celda es `0` (vacía) o índice de color 1–7 (`board[y][x]`).
- **Piezas** (`PIECES`): matrices cuadradas fijas para las 7 piezas estándar (I, O, T, S, Z, J, L). Rotación por transposición + reverso de filas (`rotateCW`), no hay tablas de rotación SRS.
- **Colisión** (`collide(shape, ox, oy)`): única función que valida límites del tablero y solapamiento con bloques fijados; usada tanto por movimiento como por rotación y ghost piece.
- **Wall kicks** (`tryRotate`): tras rotar, prueba desplazamientos `[0, -1, 1, -2, 2]` columnas hasta encontrar una posición válida.
- **Game loop** (`loop`, vía `requestAnimationFrame`): acumula delta time en `dropAccum`; cuando supera `dropInterval` baja la pieza una fila o llama a `lockPiece()`.
- **Fijado de pieza** (`lockPiece` → `merge` + `clearLines` + `spawn`): al colisionar hacia abajo, la pieza se funde en `board`, se limpian líneas completas y se genera la siguiente.
- **Puntuación/nivel**: `LINE_SCORES = [0,100,300,500,800]` multiplicado por `level`; nivel sube cada 10 líneas (`Math.floor(lines/10)+1`); velocidad de caída = `max(100, 1000 - (level-1)*90)` ms.
- **Estado global**: variables sueltas a nivel de módulo (`board, current, next, score, lines, level, paused, gameOver, ...`), sin clases ni closures — todo mutado directamente por las funciones.
- **Renderizado**: `draw()` limpia y redibuja todo el canvas cada frame (grid + tablero fijado + ghost piece con `globalAlpha=0.2` + pieza actual); `drawNext()` dibuja en un segundo canvas (`next-canvas`) usando el mismo `drawBlock`.
- **Input**: un único listener `keydown` global que ignora input si `paused || gameOver` (excepto `KeyP`, que siempre alterna pausa).

Si se cambian `COLS`, `ROWS` o `BLOCK` en `game.js`, hay que ajustar también `width`/`height` del `<canvas id="board">` en `index.html` para que coincidan (`COLS×BLOCK`, `ROWS×BLOCK`).
