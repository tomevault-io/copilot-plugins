## reel

> Dieser Ordner enthält eine fertige, lauffähige Pipeline, die aus einem Thema automatisch

# MorgenAI-Reels — Projektanweisungen für Claude Code

Dieser Ordner enthält eine fertige, lauffähige Pipeline, die aus einem Thema automatisch
ein 9:16-Reel im MorgenAI-Stil baut (Vertonung → wortgenaue Untertitel → HeyGen-Avatar →
Endcard → finaler Schnitt). Details zum Stil, Ausspracheregeln etc. stehen in
[`STYLE_GUIDE.md`](STYLE_GUIDE.md) — die gilt verbindlich für jeden geschriebenen Skripttext.

## Dateien

- `generate_reel.py` — das Automatisierungsskript (CLI, siehe unten)
- `config.json` — API-Keys (ElevenLabs, HeyGen, Pexels) + Style-Settings (Stimme, Avatar,
  Farben, Standard-CTA). **Nie den Inhalt der Keys in Chat-Antworten ausgeben.**
- `fonts/` — Inter Bold & Regular (Untertitel-/Endcard-Schrift)
- `news_<datum>_images.json` — optionale Sidecar-Datei neben dem Skripttext mit den
  Bild-Cutaway-Momenten (siehe Abschnitt "Bild-Cutaways" unten)
- `assets/cta_final.mp3` — fest bestätigte CTA-Aufnahme, wird automatisch wiederverwendet,
  wenn der CTA-Text dem `default_cta_text` in `config.json` entspricht
- `STYLE_GUIDE.md` — Referenz für Ton, Struktur, Ausspracheregeln, bekannte Grenzen
- `output/reel_<timestamp>/` — Arbeitsordner je Lauf (Rohdateien, Untertitel-PNGs etc.)

## Trigger: Nutzer schreibt "reel"

Wenn der Nutzer in diesem Projekt eine Nachricht schickt, die im Kern "reel" enthält oder
klar signalisiert, dass ein neues Reel entstehen soll (z. B. "reel", "neues reel", "reel
machen"), dann **nicht sofort loslegen**, sondern:

1. Rückfragen: "Willst du ein neues Reel erstellen?"
2. Bei Zustimmung: nach dem **Thema** fragen ("Was ist das Thema?").
   - Der Nutzer kann in derselben oder einer folgenden Nachricht optional auch einen
     anderen **Avatar-Look** nennen (z. B. "im Office stehend" statt dem Standard-Look,
     siehe Abschnitt "Avatar-Look ändern" unten). Falls nichts erwähnt wird: Standard-Look
     aus `config.json` verwenden, nichts nachfragen.
3. Sobald das Thema feststeht:
   a. Haupttext (**ohne** CTA-Satz) nach den Regeln aus `STYLE_GUIDE.md` schreiben:
      - Hook als erster Satz, energisch, Fakten/Body danach
      - ca. 500–600 Zeichen (≈ 30–35 Sek. Sprechzeit)
      - Ausspracheregeln beachten: `Morgen AI.` (Großschreibung + Punkt), `Punkt D E`
        (kein Punkt zwischen den Buchstaben), keine Apostroph-Kontraktionen (`hat es`
        statt `hat's`)
   b. 2–3 **Bild-Cutaway-Momente** auswählen (siehe Abschnitt "Bild-Cutaways" unten für
      Details/Regeln) — nur wenn `pexels_api_key` in `config.json` gesetzt ist, sonst
      diesen Schritt auslassen.
   c. Text **und** die geplanten Bild-Cutaways (Phrase + kurze Beschreibung, was zu sehen
      sein wird) zusammen im Chat zeigen, damit der Nutzer beides absegnen oder
      korrigieren kann, bevor gerendert wird (Rendering dauert 3–6 Min. wegen HeyGen —
      sich unnötige Wiederholungen sparen).
   d. Text in eine neue Datei `news_<YYYY-MM-DD>.txt` im Projektordner speichern
      (bei mehreren Reels am selben Tag: Thema als Suffix, z. B.
      `news_2026-08-04_deepseek.txt`). Falls Bild-Cutaways bestätigt wurden, zusätzlich
      als `news_<YYYY-MM-DD>_images.json` im gleichen Ordner speichern (Format siehe
      unten).
   e. Ausführen (im Projektordner):
      ```
      python3 generate_reel.py --script news_<datum>.txt --out reel_<datum>.mp4
      ```
      Die Bildmomente-JSON wird automatisch mitgenommen, wenn sie unter
      `news_<datum>_images.json` neben dem Skript liegt — kein extra Flag nötig. Falls
      ein anderer Avatar-Look gewünscht wurde, `--avatar-id <id>` anhängen (siehe unten,
      wie die ID ermittelt wird).
   f. Nach Abschluss: Pfad zum fertigen Video mitteilen und die Datei mit `SendUserFile`
      zeigen.

Ohne den Trigger "reel" (z. B. bei reinen Fragen zur Pipeline, Debugging, Code-Änderungen)
ganz normal wie sonst auch arbeiten — die Rückfrage ist nur für den "neues Reel starten"-Fall.

## Avatar-Look ändern

Standard ist `avatar_id` aus `config.json` (aktuell `Brandon_Lobby_Sitting_Front_public`).
HeyGen bietet für "Brandon" mehrere Looks/Posen mit jeweils eigener ID (z. B. "Brandon
Office Standing Front" — siehe HeyGen-Dashboard unter Avatare → Brandon).

So wird ein anderer Look für **ein einzelnes** Reel verwendet (ohne den Standard in
`config.json` dauerhaft zu ändern):

1. Verfügbare Looks abfragen: `heygen_list_avatars()` in `generate_reel.py` ruft
   `GET https://api.heygen.com/v2/avatars` ab und liefert `avatar_id` + `avatar_name` für
   alle Looks des Accounts. Bei Bedarf per Python-Einzeiler aufrufen und nach "Brandon" +
   Stichwort (z. B. "office", "standing") filtern.
2. Passende `avatar_id` in den Pipeline-Aufruf geben:
   ```
   python3 generate_reel.py --script news_<datum>.txt --out reel_<datum>.mp4 --avatar-id <ID>
   ```
3. Kein eindeutiger Treffer per Name gefunden? Den Nutzer bitten, die ID direkt aus dem
   HeyGen-Dashboard zu kopieren (Avatar-Karte → "Use in video" zeigt den Look, ID per API
   wie oben nachschlagen).

Soll ein Look zum neuen **Standard** werden (nicht nur für ein Reel), stattdessen
`avatar_id` in `config.json` dauerhaft ersetzen.

## Bild-Cutaways

Statt die ganze Zeit den Avatar zu zeigen, kann das Reel an 2–3 kurzen Stellen (2–4 Sek.)
stattdessen ein thematisch passendes Stock-Foto einblenden (mit sanftem Zoom), während
Stimme und Untertitel normal weiterlaufen. Bildquelle ist ausschließlich die kostenlose
Pexels-API — keine KI-generierten Bilder (siehe Begründung unten).

**Voraussetzung:** `pexels_api_key` in `config.json` muss gesetzt sein. Ist das Feld leer,
läuft die Pipeline einfach ohne Bild-Cutaways durch (kein Fehler).

**Beim Schreiben eines neuen Reel-Skripts wählt Claude selbst 2–3 Bildmomente aus** und
legt sie als `news_<datum>_images.json` neben der Skripttextdatei ab:

```json
[
  {"phrase": "riesige Rechenzentren", "query": "data center server room"},
  {"phrase": "auf dem Smartphone tippt", "query": "person using smartphone technology"}
]
```

Regeln dafür:
- `phrase` muss **exakt** (Groß-/Kleinschreibung, Satzzeichen) als Teilstück im
  Haupttext vorkommen — daraus wird der Zeit-Bereich im fertigen Audio abgeleitet
  (`resolve_image_moments()` in `generate_reel.py`). Passt die Phrase nicht exakt, wird
  dieser Moment beim Rendern einfach übersprungen.
- `query` ist ein kurzer **englischer** Suchbegriff (bessere Pexels-Trefferquote als
  Deutsch).
- Bildmomente sind **generische, thematisch passende Szenen** (Serverraum,
  Chip-Nahaufnahme, Code auf Bildschirm, Roboter, Person mit Smartphone) — **keine**
  konkreten Firmenlogos/Markenzeichen. Weder Pexels (Lizenzrecht) noch KI-Generierung
  (verzerrt echte Logos oft falsch) eignen sich dafür. Braucht ein Reel wirklich mal ein
  exaktes Logo/Screenshot, liefert der Nutzer das Bild manuell — das ist kein Fall für
  diese automatische Auswahl.
- Findet Pexels für einen Suchbegriff kein Foto, wird dieser eine Cutaway beim Rendern
  automatisch übersprungen (kein KI-Bild als Fallback, keine kaputte Pipeline).

Manuelles Überschreiben/Testen: `--images <pfad.json>` (Default: `<script>_images.json`),
komplett deaktivieren: `--no-images`.

## Bekannte Grenzen (siehe auch STYLE_GUIDE.md)

- HeyGen-Endpoints `v2/video/generate` / `v1/video_status.get` sind "Legacy", Abschaltung
  laut HeyGen zum 31. Oktober 2026 angekündigt — danach ggf. Migration auf `POST /v3/videos`
  nötig.
- ElevenLabs-Quota kann aufgebraucht sein (`quota_exceeded`-Fehler) — im ElevenLabs-Dashboard
  prüfen.
- Untertitel-Position/-Größe ist in `render_caption_image()` / Variable `y_pos` in
  `generate_reel()` fest einprogrammiert.

---
> Source: [Mvrkke/Reel](https://github.com/Mvrkke/Reel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
