# Persona-Kammer als Android-APK

Dieser Ordner ist ein fertiges **Capacitor**-Projekt. Es packt die App in eine echte
Android-App (WebView) – **ohne Server, offline nutzbar**. Die App spricht dein Ollama/TTS
direkt über die im Menü eingetragene Adresse an (auch `http://` im LAN ist erlaubt).

Die kompilierte `.apk` kann ich dir nicht direkt liefern (dafür braucht es das Android-SDK).
Du erzeugst sie mit **einem Befehl lokal** – oder komplett automatisch per **GitHub Actions**,
ohne irgendetwas zu installieren.

---

## Weg 1 — Vollautomatisch per GitHub Actions (kein lokales Setup)

1. Diesen Ordner (`android-apk/`) als **eigenes GitHub-Repo** hochladen
   (die Datei `.github/workflows/build-apk.yml` ist schon dabei).
2. Push auf `main` startet den Build automatisch – oder im Reiter **Actions**
   den Workflow „Build Android APK" manuell über **Run workflow** starten.
3. Nach ~3–5 Min unter dem Lauf bei **Artifacts** die
   `persona-kammer-debug-apk` herunterladen → enthält `app-debug.apk`.

Das ist der einfachste Weg für dich, weil kein Android Studio/SDK nötig ist.

---

## Weg 2 — Lokal bauen (ein Befehl)

Voraussetzungen: **Node 18+**, **JDK 17**, **Android SDK** (Android Studio oder
`sdkmanager`), Umgebungsvariable `ANDROID_HOME` gesetzt.

```bash
cd android-apk
npm install
npx cap add android          # legt das native android/-Projekt an (einmalig)
npx cap sync android
cd android
./gradlew assembleDebug
```
Fertige Datei:
```
android/app/build/outputs/apk/debug/app-debug.apk
```

Optional das Launcher-Icon aus `assets/icon.png` erzeugen (nach `cap add android`):
```bash
npx capacitor-assets generate --android \
  --iconBackgroundColor '#140d2b' --iconBackgroundColorDark '#140d2b'
```

---

## APK aufs Handy bringen & installieren

1. `app-debug.apk` aufs Android-Gerät übertragen (USB, Cloud, Tailscale-Freigabe …).
2. Datei antippen → falls gefragt, **„Installieren unbekannter Apps"** für den
   verwendeten Datei-/Browser-App erlauben → installieren.
3. App öffnen, im Menü **Einstellungen** dein Modell + (optional) TTS-Server eintragen.

Hinweis: Die Debug-APK ist ohne Play-Store-Signatur, aber ganz normal installierbar.
Für eine signierte Release-APK (`assembleRelease`) brauchst du zusätzlich einen Keystore
– nur nötig, wenn du sie im Play Store veröffentlichen willst.

---

## App anpassen
- **App-Name / Paket-ID:** in `capacitor.config.json` (`appName`, `appId`).
  `appId` ist die Java-Paket-ID (Kleinbuchstaben, Punkte, keine Bindestriche),
  aktuell `de.larskammer.persona`.
- **Neue App-Version einspielen:** `pwa/index.html` ist die Quelle. Nach Änderungen einfach
  die aktuelle `index.html` nach `android-apk/www/index.html` kopieren, dann neu bauen
  (bzw. erneut pushen).
- **Ollama/TTS über http im LAN:** funktioniert, weil in `capacitor.config.json`
  `allowMixedContent: true` gesetzt ist. Trotzdem muss Ollama mit
  `OLLAMA_ORIGINS=*` laufen.
