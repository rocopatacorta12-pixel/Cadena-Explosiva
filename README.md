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
