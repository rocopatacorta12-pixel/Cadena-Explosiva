# 💣 Cadena Explosiva

Juego multijugador online (2-8 jugadores) de pasar la bomba con palabras. P2P vía PeerJS, sin servidor propio.

## Modos de juego (el anfitrión elige en el lobby)
- **💣 Clásico:** la bomba explota al azar en cada mano (7-20s ocultos). Nadie sabe cuándo.
- **🕯️ Mecha Compartida:** una mecha larga oculta (escala con la cantidad de jugadores, 3-5 vueltas al azar) se consume entre todos. Cada uno ve los segundos de su turno, que se achican a medida que la mecha baja. Turno vencido, palabras inválidas, palabras largas y una deriva oculta mueven la mecha al azar: imposible de predecir.

## Jugar online (GitHub Pages)
1. Settings → Pages → Branch: `main` → carpeta `/ (root)` → Save.
2. Entrá a `https://TU-USUARIO.github.io/NOMBRE-DEL-REPO/`
3. Uno crea sala, comparte el código de 4 letras, los demás se unen.

## Descargar el APK
1. Al subir los archivos, la pestaña **Actions** compila el APK automáticamente (tarda ~5-8 min).
2. Actions → el workflow "Build APK" más reciente → abajo en **Artifacts** → descargá `cadena-explosiva-apk`.
3. Descomprimí el zip, pasá `app-debug.apk` al celu e instalalo (permitir orígenes desconocidos).

También podés relanzar la compilación a mano: Actions → Build APK → Run workflow.

## Estructura
- `index.html` — el juego (para GitHub Pages)
- `www/index.html` — copia que usa Capacitor para el APK
- `capacitor.config.json`, `package.json` — config de Capacitor
- `.github/workflows/build-apk.yml` — compilación automática del APK

⚠️ Si editás el juego, actualizá **los dos** index.html (raíz y `www/`).


## v1.3
- **Diccionario real ES + EN** (~900.000 palabras): solo valen palabras que existen, con o sin tilde ("árbol" = "arbol"). Reglas de nombres propios/marcas/lugares no usan diccionario.
- **Rondas aleatorias**: cada ronda arranca en un jugador al azar (nunca el mismo dos veces seguidas) y recorre a todos; el contador de ronda ahora cuenta ciclos completos.
- **Doble o nada escalable**: cada pase sube el nivel (más letras mínimas, letra obligatoria, letra inicial).
- **Teclado**: al escribir, la pantalla pasa a modo compacto y la regla se ve siempre (también aparece arriba del casillero).
- **Fix revancha**: reinicio limpio y sincronización de pantalla a prueba de fallos; el que tiene la bomba siempre ve la barra para escribir.
- **Casillero siempre vacío** al arrancar cada turno.
- **Ícono de app** (clásico morado) para APK y para instalar desde el navegador (manifest + apple-touch-icon). El workflow genera los íconos Android automáticamente.


## v1.4
- **Teclado propio dentro del juego**: sin autocorrector ni sugerencias del celu (igual para todos), la pantalla no se mueve nunca y la consigna queda siempre a la vista. En PC se puede seguir tipeando con el teclado físico.
- **Consignas coherentes**: cada categoría (frutas, animales, instrumentos, países, oficios, etc.) valida contra su propia lista de palabras; las rimas se validan por terminación real + diccionario. Se eliminaron las consignas invalidables (marcas, nombres propios, personajes).
- **Pool de ~880 consignas** con rotación persistente: el celu del anfitrión recuerda cuáles ya salieron (aunque se cierre la app) y no repite hasta agotar el pool (~100+ partidas).
- **Rondas especiales con anuncio**: aparecen a pantalla completa para todos con cuenta regresiva de 4 segundos, y recién ahí la bomba sale a un jugador al azar. Nadie arranca en desventaja.
- **Palabras bloqueadas toda la partida**: lo que se dijo en cualquier ronda no se puede repetir hasta que alguien gane, aunque la consigna sea otra. Se muestran arriba con el total.
- **Hasta 12 jugadores** por sala.
- **Aviso de anfitrión**: la app explica que conviene que cree la sala el celu más potente y con mejor internet.
