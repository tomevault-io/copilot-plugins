## pocketclaw

> Wenn der User die PocketClaw-App starten will — immer dieselbe Terminal-Sequenz ausführen


# „App starten“ / „starte die App“

Wenn der User schreibt, die App zu **starten** (z. B. „starte die App“, „App starten“, „run the app“ im Sinne von PocketClaw/Flutter), **nicht nur Befehle nennen**, sondern sie **selbst ausführen** (PowerShell, Workspace-Pfad).

## Standard-Ablauf

1. Arbeitsverzeichnis:

   `app/mobile/flutter_app` (vollständiger Pfad zum Repo: `…\pocketclaw\app\mobile\flutter_app`).

2. **Emulator** nur starten, wenn **kein** Android-Gerät in `flutter devices` erscheint:

   `flutter emulators --launch Pixel_9_Pro`

   (Oder ein anderer vorhandener Emulator-Id aus `flutter emulators` — nicht dieselbe AVD zweimal parallel starten.)

3. App im Debug-Modus auf Android:

   Zuerst `flutter devices` — die **Device-ID** des Emulators (z. B. `emulator-5554`) verwenden:

   `flutter run -d emulator-5554`

   (Die ID `-d android` ist je nach Flutter-Version ungültig; immer die ID aus `flutter devices` nehmen. Wenn nur ein mobiles Gerät verbunden ist, reicht oft auch `flutter run` ohne `-d`.)

Vorher bei Bedarf: `flutter pub get` (nur wenn Abhängigkeiten geändert wurden oder der Build es verlangt).

## Nicht gemeint

- `flutter run -d windows` oder Chrome — **nur** wenn der User **explizit** Desktop/Web will.
- Release-Build — **nur** wenn der User Release/ APK-Build verlangt.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pepler1993-dot) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
