# HANDOFF — Cadena Explosiva v28

Juego P2P de fiesta para celulares (PeerJS, un solo `index.html`, sin backend propio): una bomba pasa de mano en mano y explota en la cara del que falla. **7 modos**, host autoritario, espectadores en vivo, podio + revancha.

## Modos actuales

| # | Modo | Clave | Mín. jug. | Idea |
|---|------|-------|-----------|------|
| 1 | 💣 Clásico | `clasico` | 2 | Palabra según sílaba/regla antes de que explote (reloj oculto) |
| 2 | 🕯️ Mecha Compartida | `mecha` | 2 | Cada acierto suma segundos al reloj visible |
| 3 | 🔤 Ahorcado Exprés | `ahorcado` | 2 | Adiviná la próxima letra entre 4 opciones |
| 4 | ⚡ Duelo de Reflejos | `reflejos` | 2 | 27 microjuegos relámpago |
| 5 | ❓ Trivia con Bomba | `trivia` | 2 | 4 opciones, dificultad creciente (pools 180/120/100) |
| 6 | 🧠 A Ver Si Te Acordás | `memoria` | 2 | **NUEVO v27** — Simon con bomba + 2 giros únicos |
| 7 | 🔮 El Oráculo Miente | `oraculo` | 3 (guardia lobby+host) | Social: voto secreto, excusas, desenmascarar |

## El modo nuevo: `memoria` (v27)

**Flujo del turno (host):** `arrancarMemoria()` → etapa `ver` (la cadena se ilumina sola, la ven TODOS con el mismo timing) → si `traic`, etapa `mezclar` (los paneles cambian de lugar con animación) → etapa `toca` (el portador repite; cada toque se valida y se broadcastea como evento `memtoque` para que todos lo vean en vivo).

- **Cadena** (`GAME.memSeq`): empieza en 3 (`CFG.MEM_LEN_INI`), crece **+1 por acierto** (`memExtender`), vuelve a 3 tras explosión (`memNuevaSeq` en `explota`).
- 👻 **Color Señuelo** (desde len ≥ `MEM_SEN_DESDE` = 4): se elige un color fantasma, se inyecta una aparición extra en lo mostrado (`state.mem.ver`) y se anuncia. El objetivo (`memObjetivo`) = la cadena **sin NINGUNA aparición de ese color** (filtrado por color, no por posición; se garantizan ≥ 2 toques válidos vía `cands`).
- 🔀 **Paneles Traicioneros** (desde len ≥ `MEM_TRAIC_DESDE` = 6): tras mostrar, `state.mem.orden` se baraja (garantizado ≠ identidad) y todos ven la animación `memHop`.
- **Reloj:** `turnoSeg = MEM_SEG_BASE (5) + ceil(len * MEM_SEG_PASO (1,5))`; intervalo `memInt` con `broadcastTick` (mismo patrón que trivia).
- **Validación:** `intentarMemoria(deId, a, nonce)` — guardas: modo/fase/portador/etapa `toca`/nonce/regex `[0-5]`; mal → `memtoque{ok:false}` + `explota()`; bien → `memtoque{ok:true}` + progreso; completa → `pasadas++`, evento `paso`, cadena +1, `pasarBombaA(sig)`.

## Arquitectura (qué toca cada capa para un modo nuevo)

1. **CFG:** constantes del modo (tuning en un solo lugar).
2. **Estado público** `GAME.state.*` (viaja por `state` full): `mem: { etapa, ver, sen, orden, len, progreso, traic }` + `turnoSeg`.
3. **Estado privado del host:** `memSeq`, `memObjetivo`, `memIdx`, timers (`memInt`, `_memT`, `_memT2`) — limpiados todos en `limpiarTimer`.
4. **Red (NET):** entrada `kind:"mem"` en `hostRecibe` (con guarda `bombaEn === conn.peer`); eventos salientes `memtoque`/`paso`/`invalida`/`explosion`; en `invitadoRecibe` el evento actualiza `state.mem.progreso` y flashea el panel.
5. **UI:** `renderMem()` (memo por `_memKey = etapa|ver|sen|orden|bombaEn|ronda` — la animación NO se reinicia en re-renders), `renderMemInfo()` (memo por texto), `memAnimarVer()` (mismo timing en todos los celus, tonos por color), `memAnimarMezcla()`, `memElegir()` (flash local + nonce + `enviarInput`), `memFlashExterno()`. Compactación `#app.mem-turno` (patrón copiado de `trv-turno`). `pintarTurnoCount` incluye memoria. `reset()` limpia todo lo del modo.
6. **Puntos de integración** (buscarlos para futuros modos): `prepararRonda`, `continuarTurno`, `pasarBombaA`, `empezar`, `explota`, `chequearFin`, `volverAlLobby`, `UI.reset`, `MODOS_UI`, `NOMBRES`, tarjeta del lobby, CSS compacto.

## Invariantes que NO hay que romper

- **El host valida todo:** ningún invitado decide aciertos, explosiones ni turnos. Toda entrada lleva nonce anti-doble (`ultimoNonce`).
- `limpiarTimer` debe limpiar TODOS los timers del modo (intervalos y timeouts) — el QA verifica que queden en `null` en cada podio.
- **Memo de render:** nunca re-armar DOM que corre animación si la clave no cambió (el render corre 1/seg por los ticks).
- **Regla de oro mobile:** en el turno del jugador, TODO lo interactivo entra sin scroll (clases `*-turno` + `max-height: 620px`).
- Los eventos efímeros (`memtoque`, `invalida`, `paso`…) **no van en el state full**; van aparte y cada cliente los refleja localmente.
- **No tocar** el bloque `--vhreal`/`visualViewport` ni el CSS `trv-turno`.

## Auditoría de esta entrega (v27 vs v26.1)

- **Archivo completo:** 4831 líneas / 299.785 bytes, sin cortes ni "resto igual". `node --check` OK.
- **Solo 13 funciones compartidas cambiaron**, todas con hooks puramente aditivos del modo memoria: `chequearFin`, `continuarTurno`, `empezar`, `explota`, `hostRecibe`, `invitadoRecibe`, `limpiarTimer`, `pasarBombaA`, `pintarTurnoCount`, `prepararRonda`, `render`, `reset`, `volverAlLobby`. Ningún cambio de lógica en los 6 modos existentes.
- **Host-autoridad verificada:** los invitados solo mandan `{kind:"mem", a, nonce}` vía `enviarInput`; `hostRecibe` guarda `bombaEn === conn.peer`; toda la generación y validación vive en el host.
- **CSS crítico intacto:** las 21 reglas `trv-turno` y los 5 usos de `--vhreal` de v26.1 se conservan tal cual; el bloque `visualViewport` es idéntico.
- **Optimización verificada:** `renderMem` memoizado (`_memKey`), animación solo cuando cambia la clave; timers de animación en `_memTimers` limpiados por `_memLimpiarTimers`; `AUDIO.tono` desconecta nodos WebAudio en `onended` (patrón v26); `memInt`/`_memT`/`_memT2` en `limpiarTimer` y `reset`.
- **Layout compacto:** `#app.mem-turno` oculta ronda/regla-box/quién-tiene/usadas/vidas y (en `@media max-height:620px`) la bomba; pads con `clamp` de alto. A 620px el turno ocupa ~240px verticales: entra sin scroll.

## QA de esta versión (harness propio, headless en Node)

Harness con reloj virtual (`setTimeout`/`setInterval` falsos), DOM stub, Peer stub y bots que juegan **solo con información pública** (reconstruyen el objetivo filtrando el fantasma de `state.mem.ver` — valida que la regla sea justa para un humano).

**3 corridas completas × 7 escenarios — 415/448/468 checks, 0 fallas:**

- **S1:** 2 jugadores, partida entera mezclando aciertos y errores; crecimiento exacto +1 de la cadena; señuelo desde len 4 (objetivo sin el fantasma, fantasma visible en `ver`, ≥2 toques válidos, bot público reconstruye el objetivo); traicioneros desde len 6; fórmula del reloj; podio; **revancha** re-arranca memoria y **"otro modo"** vuelve al lobby limpio.
- **S2:** 5 jugadores con timeouts (pierde vida, cadena vuelve a 3), toques erróneos y **desconexión del portador en `ver` y en `toca`** — la partida siempre siguió y llegó al podio.
- **S3:** 12 jugadores; **nonce duplicado rechazado**, toque **fuera de fase** ignorado, toque de **no-portador** ignorado, inputs inválidos (`"9"`, `"banana"`, `null`) ignorados sin explotar; podio.
- **S4 (regresión):** clásico, mecha, ahorcado, reflejos y trivia arrancan y llegan al podio solo por timeouts, sin timers de modo colgados.
- **S5:** render de portador y espectador en las 3 etapas (`ver`, `mezclar`, `toca`) sin excepciones.
- **S6:** semántica de la entrada de red `kind:"mem"`: el no-portador no avanza progreso, el portador valida; lobby limpio al salir.
- **S7:** caída del **host** con memoria activa (lado invitado): `hostSeCayo` no lanza excepciones.
- **En todos los podios:** `memInt`, `_memT` y `_memT2` en `null` (sin timers colgados).

Nota menor (aceptada, no es fuga): `memFlashExterno`/`memFlashLocal` usan `setTimeout` de 200–520 ms no registrados en `_memTimers`; solo togglean una clase sobre un nodo capturado y no se acumulan.

## Cambio de assets (v28)

- **Los 34 avatares de `img/animales/` fueron reemplazados** por el set nuevo estilo sticker (144×144, fondo sólido). La **mariposa salió** del set y entró el **tiburón** en su lugar (34 animales igual).
- **Reordenamiento por color:** los avatares con fondos parecidos (azules marinos: tiburón/ballena/delfín/pato/pingüino/lobo/lechuza/pulpo/elefante; verdes: vaca/caballo/oso/panda/mono/rana/koala/trex/tortuga/zorro; cálidos: gallina/tigre/cerdo/gato/erizo/hámster/jirafa/león; morados/grises: cebra/unicornio/conejo/ratón/águila/dragón/perro) se intercalaron en round-robin para que nunca queden dos paletas iguales juntas — ni horizontal ni verticalmente en una grilla de 6 columnas. El orden vive solo en `CFG.EMOJIS` (un solo lugar); los archivos se renombraron `NN_nombre.png` acorde. El **zorro es el avatar #1** (default de perfil nuevo).
- El avatar no se persiste en `localStorage`, así que el renombrado no rompe perfiles guardados; el default (`CFG.EMOJIS[0]`) sigue siendo válido.

## QA extra de v28

- `CFG.EMOJIS`: 34 entradas únicas, todas con archivo existente en `img/animales/` y `www/img/animales/` (verificado programáticamente).
- Regla de separación de color verificada por script: sin vecinos de la misma paleta ni horizontales ni verticales (grilla ×6).
- `node --check` OK sobre el script completo; `index.html` raíz idéntico a `www/index.html`.
- Suite completa de 7 escenarios re-corrida ×3 sobre el build v28: 435/448/501 checks, **0 fallas**.

## Qué NO se tocó

Clásico, Mecha Compartida, Ahorcado, Reflejos, Trivia, Oráculo, protocolo PeerJS, podio, premios, revancha, salas, sonidos generales, diccionario (`dicc.txt`), workflow de GitHub Actions, íconos, manifest, capacitor.

## Cómo probarlo en 30 segundos

1. Abrí `index.html` (con internet para el CDN de PeerJS), creá sala y entrá con otro celu/pestaña.
2. Elegí 🧠 **A VER SI TE ACORDÁS** y arrancá.
3. Turnos 1–2: cadena de 3, sin giros. Desde la cadena 4 aparece el 👻 **fantasma**; desde la 6, los 🔀 **paneles se mueven**. Mirá cómo el otro celu ve la secuencia y tus toques en vivo.

## Candidatos para v28 (ideas, sin implementar)

- 3.er giro opcional de memoria: eco invertido (repetir al revés en rondas altas) o paso solo-audio.
- Temporizador de mezcla configurable por dificultad; skin de paneles (formas además de color, para daltonismo).
- Sonido de "mezcla" propio y vibración diferenciada al completar la cadena.
