# Documentacion Tecnica - Pac-Man Senior Edition Pro Fixes

## 1. Resumen
El proyecto es un juego de Pac-Man implementado en un unico archivo [`pacman.html`](/F:/Pac%20man/pacman.html), con CSS y JavaScript embebidos.

La arquitectura actual se basa en clases:

- `InputController`: centraliza teclado y controles tactiles.
- `SoundManager`: sintetiza efectos de sonido con Web Audio API sin depender de archivos externos.
- `Board`: administra el grid, pellets, super pellets, teletransporte lateral y validacion de celdas caminables.
- `Entity`: base comun para movimiento en grid.
- `Player`: controla animacion, input pendiente y consumo de pellets.
- `Ghost`: base de fantasmas, con estados y logica de movimiento.
- `Blinky`, `Pinky`, `Inky`, `Clyde`: variantes con targeting propio.
- `GameEngine`: coordina HUD, loop, colisiones, vidas, power mode y reinicio.

## 2. Leyenda Del Mapa
El mapa vive en `INITIAL_MAP` y usa numeros para representar cada tile.

- `1`: pared. Bloquea movimiento de Pac-Man y fantasmas.
- `0`: pellet normal. Da puntos y desaparece al consumirse.
- `2`: pasillo vacio o zona sin pellet. Es transitable.
- `3`: posicion inicial de Pac-Man. Solo se usa al inicializar; luego se reemplaza por `2`.
- `4`: super pellet. Da puntos, activa power mode y tambien cuenta como pellet pendiente.

Notas importantes:

- La fila `9` actua como tunel lateral con wrap-around.
- La puerta conceptual de la casa de fantasmas se maneja por logica en `Board.isWalkable(...)`, no con un numero exclusivo en el mapa.

## 3. Mapa Y Reinicio
`Board.reset()` crea una copia limpia de `INITIAL_MAP` en cada partida.

Durante el reset:

- Detecta el tile `3` para guardar el spawn del jugador.
- Convierte ese tile a `2` para que el mapa jugable no tenga una marca especial persistente.
- Cuenta pellets normales (`0`) y super pellets (`4`) en `pelletsLeft`.

Esto evita mutar el mapa base entre reinicios.

## 4. Movimiento
El movimiento es continuo pero alineado a grid:

- Cada entidad se desplaza con velocidad en unidades por segundo.
- `Entity.update(...)` detecta el cruce por el centro del tile para evitar overshoot.
- `Player.onCenterCrossed(...)` intenta aplicar `nextDir` cuando el giro es valido.
- `Ghost.onCenterCrossed(...)` recalcula direccion segun el target del estado actual.

El tunel lateral solo funciona alrededor de la fila central del mapa, mediante `Board.wrapEntityPosition(...)`.

## 5. Sistema De Estados De Fantasmas
La implementacion actual usa estos estados:

- `ScatterMode`: mueve al fantasma hacia su esquina asignada.
- `ChaseMode`: usa la logica de persecucion propia de cada fantasma.
- `ScaredMode`: se activa durante el power mode; reduce velocidad y hace que el fantasma intente alejarse.
- `EatenMode`: se usa cuando Pac-Man come un fantasma; el sprite se reduce a ojos y vuelve a la casa.
- `HouseMode`: mantiene al fantasma dentro de la casa hasta que se cumple su delay de salida.

Observacion:

- En la version actual ya no existe `AngryMode` activo en runtime, aunque quedaron constantes relacionadas con velocidad heredadas de iteraciones anteriores.
- Durante `HouseMode`, los fantasmas se alinean primero con la columna de la puerta (`col 9`) y luego suben hacia la salida. Ese ajuste evita que Clyde quede empujando contra la pared derecha de la casa.

## 6. IA De Cada Fantasma
- `Blinky`: persigue directamente la posicion de Pac-Man.
- `Pinky`: apunta 4 tiles por delante del jugador y conserva el bug clasico al mirar hacia arriba.
- `Inky`: calcula un punto adelantado respecto a Pac-Man, toma el vector desde Blinky y lo duplica.
- `Clyde`: persigue si esta lejos; cuando esta cerca vuelve a su esquina.

Cuando estan en `ScaredMode`, todos cambian a una estrategia comun de huida basada en maximizar distancia al target usado en esa decision.

## 7. Super Pellets Y Power Mode
Hay cuatro super pellets codificados con `4` en el mapa.

Cuando Pac-Man consume uno:

- suma `50` puntos,
- reproduce un sonido mas brillante que el pellet normal,
- activa `powerModeTimeLeft`,
- los fantasmas pasan a `ScaredMode`,
- cambian de sprite,
- bajan su velocidad.

Si Pac-Man toca un fantasma en `ScaredMode`:

- suma `200` puntos,
- el fantasma cambia a `EatenMode`,
- vuelve a la casa para reincorporarse despues.

En el codigo actual `POWER_MODE_DURATION` vale `8` segundos.

## 8. Vidas, Colisiones Y Fin De Juego
El HUD muestra:

- score,
- pellets restantes,
- vidas.

Colisiones:

- jugador vs pellets: resueltas por tile al entrar en la celda.
- jugador vs fantasmas: resueltas por distancia euclidiana entre entidades.

Si Pac-Man choca con un fantasma que no esta en `ScaredMode` ni `EatenMode`:

- pierde una vida,
- se reproduce una secuencia descendente de muerte sintetizada por codigo,
- si quedan vidas, se reposiciona la ronda,
- si no quedan vidas, se muestra `GAME OVER`.

Si `pelletsLeft` llega a `0`, el juego termina en victoria.

## 9. Audio
El juego ahora genera sonido procedural con `SoundManager` y `Web Audio API`.

No hay archivos `.mp3`, `.wav` ni MIDI cargados desde disco. En su lugar:

- cada pellet normal dispara un pulso corto tipo "waka",
- cada super pellet dispara una variante de dos notas,
- la muerte reproduce una secuencia descendente mas larga.

Detalles tecnicos:

- El audio se desbloquea en la primera interaccion del usuario (`keydown`, `pointerdown` o `touchstart`), porque el navegador no permite reproducir sonido antes.
- `playTone(...)` encapsula la creacion de osciladores y una envolvente simple de volumen.
- Se usa un `masterGain` comun para controlar el volumen global.
- Hay un throttle minimo entre pellets para evitar apilar demasiados osciladores si Pac-Man consume varios muy rapido.

## 10. Flujo De Spawns
`GameEngine.createEntities()` construye una vez al jugador y a los cuatro fantasmas.

`GameEngine.resetRoundPositions()`:

- reposiciona al jugador en su spawn,
- reposiciona fantasmas en la casa,
- aplica delays de salida distintos por fantasma,
- limpia el input pendiente,
- reinicia el power mode.

## 11. Detalles A Tener En Cuenta
- Los botones tactiles visibles siguen mostrando `?` en vez de flechas.
- `POWER_MODE_DURATION` esta en `8`, no en `30`.
- `SCARED_GHOST_SPEED`, `HouseMode` y `EatenMode` si estan activos y forman parte real de la logica.
- La documentacion previa que mencionaba `AngryMode` y ausencia de energizers ya no representa el estado actual del juego.
- El audio depende de que el navegador soporte `AudioContext` o `webkitAudioContext`.
