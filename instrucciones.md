# Notas De Implementacion

## Estructura Actual
El proyecto esta resuelto en un unico HTML con estas piezas principales:

```js
class InputController {}
class SoundManager {}
class Board {}
class Entity {}
class Player extends Entity {}
class GhostMode {}
class ChaseMode extends GhostMode {}
class ScatterMode extends GhostMode {}
class ScaredMode extends GhostMode {}
class EatenMode extends GhostMode {}
class HouseMode extends GhostMode {}
class Ghost extends Entity {}
class Blinky extends Ghost {}
class Pinky extends Ghost {}
class Inky extends Ghost {}
class Clyde extends Ghost {}
class GameEngine {}
```

## Semantica Del Mapa
`INITIAL_MAP` usa esta codificacion:

- `1`: pared
- `0`: pellet normal
- `2`: pasillo vacio
- `3`: spawn inicial de Pac-Man
- `4`: super pellet

La fila `9` se usa como tunel lateral.

## Reglas Implementadas
- El mapa base no se muta directamente.
- El jugador gira en centros de tile.
- Los fantasmas usan estados para salir de la casa, perseguir, dispersarse, asustarse y volver al spawn.
- Mientras estan en `HouseMode`, se alinean con la puerta de la casa antes de subir para salir.
- Los super pellets activan invencibilidad temporal para Pac-Man.
- Los sonidos se generan por codigo con `Web Audio API`; no hay archivos de audio externos.
- Los fantasmas comidos vuelven a la casa.
- Hay sistema de vidas y respawn por ronda.

## Observaciones
- La duracion actual del power mode es `8` segundos.
- La puerta de la casa de fantasmas no esta representada por un numero del mapa; se controla por logica.
- Los controles tactiles existen, pero visualmente sus botones hoy muestran `?`.
- El audio se habilita recien con la primera interaccion del usuario por restricciones del navegador.
