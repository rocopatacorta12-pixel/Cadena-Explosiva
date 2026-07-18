# HANDOFF — Cadena Explosiva v26 (OPTIMIZADA, QA en 0 fallas)

## Qué es
Juego HTML multijugador online (2-12 jugadores) tipo "pasar la bomba". Single-file (`index.html`), sin frameworks, PeerJS (WebRTC P2P, cloud gratuito). El HOST es la única fuente de verdad: los invitados mandan inputs con nonce y el host retransmite el estado completo. Texto visible 100% en rioplatense (voseo). Mobile-first ~360px. Se publica en GitHub Pages (repo `rocopatacorta12-pixel/Cadena-Explosiva`) + APK debug vía GitHub Actions con Capacitor 6. Rafa trabaja desde la GUI web de GitHub y un celu Android: **entregar SIEMPRE archivos completos, jamás fragmentos ni diffs**.

## Estado actual: v25 FINAL — 6 modos verificados
1. **CLÁSICO** — palabras con consignas, especiales y "al revés".
2. **MECHA COMPARTIDA** — reloj invisible compartido, bonos por acierto.
3. **AHORCADO EXPRÉS** — palabra oculta, 4 opciones de letra con timer, vidas escaladas 4-6.
4. **DUELO DE REFLEJOS** — 27 microjuegos, historial anti-repetición 14.
5. **TRIVIA CON BOMBA** (nuevo, Kimi K3 + cirugía) — pregunta con 3-4 opciones y contador (13/12/11s según tanda); error o timeout = explosión. Pools: `TRV_FACIL` 180 · `TRV_MEDIA` 120 · `TRV_DIFICIL` 100 (400 en total, auditadas: sin duplicadas, opciones únicas, correcta = primer elemento de cada entrada, se baraja al generar y la solución vive solo en el host como `trvSol`). Dificultad creciente cada 2 rondas (r1-2 fácil → r9+ solo difícil). Rotación persistente en localStorage (`cadena_uso_trv_v1`). Los espectadores ven pregunta + opciones en `#trv-info` (render con memo).
6. **EL ORÁCULO MIENTE** (nuevo, Kimi K3 + cirugía) — 5 rondas: voto secreto + justificación anónima (≤60 chars) → las justificaciones se mezclan con 1-3 frases falsas del Oráculo → adivinar autores. Puntos: +2 autor real, +3 falsa detectada, +1 al autor por cada engañado, +2 al más votado. Podio por puntos. Pools: `ORA_PREGUNTAS` 200 · `ORA_FALSAS` 60 (con hueco `{n}` que el host rellena), rotación persistente (`cadena_uso_ora_v1`). Mínimo 3 jugadores: deshabilitado en el lobby **y con guardia host-side en `empezar()`** (fix v25). Etapas avanzan por timer visible (tick campo `o`) o antes si todos respondieron; desconexiones destraban la etapa.

## Auditoría v24 → v25 (entrega de Kimi)
- Diff completo contra la v24: **solo agregados y adaptaciones quirúrgicas**; los 4 modos viejos quedaron intactos (verificado por diff línea a línea y por regresión simulada).
- Arquitectura respetada: host genera y valida todo, inputs con nonce, broadcast de estado completo, render con memos (`_tvKey`, `_tiKey`, `_oraKey`), sin archivos/CDNs nuevos, rutas `dicc.txt` / `img/` intactas, voseo en todos los textos nuevos.

## Bugs encontrados y corregidos en la cirugía v25
1. `TRV_MEDIA`: "¿Quién cantó La voz de los 80…?" atribuía a Los Redondos un tema de Los Prisioneros (Chile). Reemplazada por "¿Qué banda argentina lideraba el Indio Solari?" (misma respuesta y distractores).
2. `TRV_MEDIA`: redacción confusa "¿Qué sangre lleva oxígeno…?" → "¿Qué vasos llevan la sangre con oxígeno…?" (distractor "Los glóbulos" → "Los nervios").
3. `TRV_FACIL`: "La ananá" → "El ananá" (rioplatense).
4. `GAME.empezar()`: faltaba la guardia host-side del Oráculo con <3 conectados (el lobby lo bloquea, pero si alguien se desconectaba con el modo ya elegido, arrancaba igual con 2).

## QA (headless, Node: stub de DOM + timers falsos)
- Unitarios TRV: ~3.600 generaciones (solución en rango y coincidente con la correcta, opciones únicas, segundos 8-20, escala de dificultad, rotación sin repetir hasta agotar la tanda).
- Unitarios ORA: `cantFalsas` en rango 1-3, 500 `fraseFalsa` sin huecos `{n}`, `mezclar` sin pérdidas.
- Validación del host: respuestas de no-portador ignoradas, nonce anti-doble-envío, incorrecta/timeout explota, voto a jugador inexistente / justificación corta / doble voto / mapa incompleto / candidato inválido rechazados, recorte a 60 chars, **puntaje del Oráculo verificado contra cálculo determinista** (incluido bonus del más votado).
- Partidas simuladas: 2-12 jugadores, fallos aleatorios, timeouts, desconexiones (incluido quien tiene la bomba), en los 6 modos; siempre se llega al podio, sin bomba en mano ni timers vivos después del podio. **9 corridas limpias consecutivas, ~22.600 checks por corrida (~204.000 en total), 0 fallas** + corrida final sobre el archivo empaquetado.
- Nota: la Mecha con 7+ jugadores dura mucho por diseño (verificado idéntico en v24, no es regresión).

## Estructura del ZIP (obligatoria)
`index.html` + `dicc.txt` + `img/` + `manifest.json` + íconos en la raíz (Pages) · `www/` con el mismo contenido (APK) · `capacitor.config.json`, `package.json`, `assets/`, `.github/workflows`. **Raíz y `www/` tienen que quedar IDÉNTICOS siempre** (index.html e img/ verificados byte a byte en esta entrega).

## Reglas de trabajo para el próximo chat
1. Archivos COMPLETOS siempre; QA hasta 0 fallas antes de entregar.
2. Todo texto visible en rioplatense (voseo).
3. Host valida todo; invitados solo mandan inputs con nonce.
4. Render ~1/seg con memos: no re-armar DOM si no cambió la clave.
5. Pantallas que entren en ~360px sin scroll.

## v26 — Optimización anti-ralentización en celus gama baja (sin tocar lógica)
Síntoma reportado: en partidas largas (15-20+ min) un celu gama baja empezaba a "transmitir" cada vez más lento. Causa: acumulación de basura + bola de nieve de renders. Cirugías (5, diff total ~57 líneas):
1. **Audio (la fuga principal):** cada sonido creaba Oscillator+Gain conectados al grafo y NUNCA se desconectaban → miles de nodos vivos tras 20 min en WebViews viejos. Ahora `onended → disconnect()`.
2. **Explosión:** el buffer de ruido se creaba nuevo por cada explosión; ahora se genera una vez y se cachea (`AUDIO._noise`).
3. **Coalescing de estados en el invitado (NET.invitadoRecibe):** si llegan varios `state` casi juntos (celu atrasado), se aplican todos pero se PINTA una sola vez por frame (rAF + fallback 250ms para segundo plano). Antes cada mensaje re-armaba todo el DOM: render lento → cola más larga → más lento (la bola de nieve que veía el gama baja). La lógica de "me tocó la bomba" sigue siendo por mensaje.
4. **Memo de la lista del lobby (`UI._lobbyKey`):** se re-armaba el DOM completo del lobby en CADA render, incluso durante la partida. Ahora solo si cambia jugadores/modo/fase.
5. **Memo de `renderPalabraLive` (`UI._plKey`):** una escritura de innerHTML por cada tecla de cualquier jugador; ahora solo si el texto cambió. Ambos memos se limpian en el reset junto a `_tvKey`/`_tiKey`.

QA v26 (headless, Node, DOM/Peer/Audio stubbeados, timers falsos): 18 partidas por corrida en los 6 modos (2-6 jugadores) hasta podio, 9 corridas limpias; fase 2 (×3): maratón de 4 partidas seguidas por modo con intervalos acotados, desconexiones (incluido el portador de la bomba), ráfaga de 30 estados → ≤6 renders (coalescing verificado), memos del lobby y palabra-live invalidándose correcto, nonce anti-doble intacto, y **100% de nodos de audio desconectados** tras todas las corridas. 0 fallas.
