# Crossfade A/B

Videoplayer, der ein Video abspielt und zwei Tonspuren darunterlegt. Während das Video läuft, kann man zwischen Tonspur A und B crossfaden. Das Bild läuft ununterbrochen weiter.

## Im Browser verwenden

`index.html` im Browser öffnen, oder die GitHub-Pages-Adresse dieses Repos aufrufen. Video und zwei Tonspuren per Drag & Drop laden.

**Bedienung:** Mit dem Crossfader zwischen A (Original) und B (neue Version) blenden. Beim Loslassen rastet er an der näheren Seite ein, die Mitte ist nur Übergang. Ein Klick auf A oder B, oder die Tasten A und B, schaltet mit einer kurzen Blende um. Tonspuren aufs Bild ziehen: linke Hälfte = A, rechte Hälfte = B. Ein Video landet immer im Bild. Oder oben auf dem Bild „A laden", „Video laden" oder „B laden" klicken. Play, Fortschritt und Vollbild liegen unten im Bild und blenden sich während der Wiedergabe aus, wenn die Maus ruht. Leertaste = Play/Pause · F = Vollbild

## Teilen mit vorgeladenen Dateien

Nur in der Web-Version (Claude-Artifact): Sind Video, A und B geladen, erscheint oben „Teilen“. Die drei Dateien werden unverändert (Base64, in Stücken von max. 14 MiB) in der Ablage des Artifacts gespeichert, dazu ein Manifest. Der Link ist der Artifact-Link plus `#<Manifest-ID>`; wer ihn öffnet, bekommt Video, A und B automatisch geladen. Dateinamen werden nicht gespeichert. Speicher: 1 GB pro Artifact.

## Als Mac-App starten (Entwicklung)

Voraussetzung: [Node.js](https://nodejs.org) (LTS).

```bash
npm install
npm start
```

## Mac-App bauen

```bash
npm run build:mac
```

Die fertigen Apps liegen danach in `dist/`.

## Wie es funktioniert

- Alles steckt in `index.html` (HTML, CSS und JavaScript in einer Datei). `main.js` ist nur der Electron-Rahmen für die Mac-App.
- Das Video ist die Uhr. Beide Tonspuren werden mit der Web Audio API als Buffer dekodiert und sample-genau gemeinsam gestartet, sodass zwei Versionen desselben Mixes phasengleich bleiben.
- Lautheit angleichen (Schalter, standardmässig an): Die integrierte Lautheit jeder Spur wird nach ITU-R BS.1770 / EBU R128 gemessen (K-Filter, 400-ms-Blöcke, absolutes und relatives Gating). B (Endmix) ist die Referenz und bleibt unverändert, A (Original) wird auf die Lautheit von B gebracht. Würde A dabei über −1 dBFS gehen, wird die gesamte Wiedergabe um genau diesen Betrag leiser. Kein Limiter, keine der beiden Spuren wird bearbeitet.
- Synchron (Schalter, standardmässig an, läuft automatisch, sobald beide Spuren geladen sind): Kreuzkorrelation, erst grob über die Einsätze (10-ms-Raster, FFT, ±10 s), dann fein auf Samples (8 kHz, ±25 ms). Hat das Video einen eigenen Ton, werden beide Spuren am Bild ausgerichtet, sonst B an A. Ohne eindeutige Übereinstimmung bleibt alles unverändert und der Punkt am Schalter grau. Die beiden Schalter sitzen oben rechts.
- Blendkurve automatisch: Sind A und B stark korreliert (zwei Versionen desselben Mixes), wird linear geblendet, sonst Equal Power.
- Bei Play, Pause, Spulen und Drift über 120 ms werden die Spuren mit kurzem Declick neu angesetzt.
- Schalter „Info“ oben rechts (standardmässig aus): blendet unter dem Fader pro Spur Lautheit, Korrektur in dB und Versatz in ms ein. Dateinamen werden nie angezeigt.
- Pro Spur: Lautheit, Korrektur in dB, Versatz in ms nach dem Synchronisieren, Warnung bei abweichender Länge.
- Videoformat: Am sichersten H.264-MP4. ProRes und DNxHD kann Chrome (und damit die Mac-App) nicht darstellen. Der Player zeigt dann eine Meldung im Bild.
- Grenze: Die Audiodateien werden komplett in den Arbeitsspeicher geladen. Für Spots und Clips ist das kein Problem, für Spielfilmlänge in WAV wird es schwer.
