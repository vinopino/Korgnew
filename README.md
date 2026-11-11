# VA-1 – Virtual Analog Synthesizer (HTML5)

Ein polyphoner, virtuell-analoger Synthesizer, der vollständig im Browser läuft.
Keine App-Installation, kein Backend, keine Build-Tools – nur statische Dateien.

**Technik:** HTML5 · CSS3 · JavaScript (ohne Framework) · Web Audio API · AudioWorklet · IndexedDB · Web MIDI (optional)

---

## Funktionen

| Bereich | Inhalt |
|---|---|
| Oszillatoren | 2 × Oszillator (Saw, Puls mit PWM, Dreieck, Sinus), bandbegrenzt (PolyBLEP), Oktave/Halbton/Feinverstimmung, Hard-Sync, Sub-Oszillator, Rauschen |
| Unison | 1–4 Stimmen pro Note, Verstimmung, Stereobreite |
| Filter | Zero-Delay-Feedback-SVF: LP24, LP12, HP, BP, Notch – Resonanz, Drive, Hüllkurve, Keytracking, Velocity |
| Hüllkurven | Filter-ADSR und Amp-ADSR (analoges RC-Verhalten), grafische Anzeige |
| LFO | Sinus, Dreieck, Sägezahn, Rechteck, Sample & Hold → Tonhöhe, Filter, PWM, Lautstärke, Retrigger |
| Stimmen | Poly (1–16 Stimmen), Mono, Legato, Glide/Portamento, Pitch-Bend-Bereich |
| Arpeggiator | Auf, Ab, Auf/Ab, Zufall, wie gespielt – 1–4 Oktaven, 1/4 bis 1/32 inkl. Triolen, Gate, Latch, Tempo |
| Effekte | Stereo-Chorus, Ping-Pong-Delay (tempo-synchron), Hall (Faltungshall) |
| Ausgang | Master-Volume, DC-Blocker, Soft-Clipper, Limiter (keine Übersteuerung) |
| Spielen | Touch-Klaviatur mit Mehrfinger-Bedienung und Glissando, Anschlagdynamik (tiefer auf der Taste = lauter), Maus, Computertastatur, Pitch-Bend- und Mod-Rad, HOLD (Sustain) |
| Presets | 13 Werkspresets, eigene Presets (IndexedDB), Export/Import als JSON, letzter Zustand wird automatisch wiederhergestellt |
| MIDI | Web MIDI (Chrome/Edge): Noten, Pitch-Bend, Mod-Rad (CC1), Sustain (CC64), Cutoff (CC74), Resonanz (CC71) |

### Bedienung

- **Audio starten:** Beim ersten Öffnen auf „Audio starten“ tippen (Browser erlauben Audio nur nach einer Benutzeraktion). Der Button **AUDIO** oben startet/stoppt die Engine.
- **Drehregler:** nach oben/unten (oder rechts/links) ziehen · Doppeltipp = Standardwert · Mausrad · Pfeiltasten · Shift = Feineinstellung
- **iPhone:** Funktionsseiten über die Leiste oben umschalten (OSZ, FILTER, HÜLLK., LFO, ARP, FX, PRESETS)
- **iPad / Desktop:** alle Funktionsbereiche gleichzeitig sichtbar
- **Computertastatur:** `A W S E D F T G Z H U J K O L P` = Töne ab C (deutsches Layout) · Taste unten links (`Y`, auf US-Tastatur `Z`) = Oktave tiefer, `X` = Oktave höher · Leertaste = Hold
- **PANIC** beendet sofort alle Töne.

---

## Projektstruktur

```
virtual-analog-synth/
├── index.html              Einstiegspunkt
├── manifest.webmanifest    Web-App-Manifest („Zum Home-Bildschirm“)
├── .nojekyll               für GitHub Pages
├── css/
│   └── style.css           Responsives Layout (iPad / iPhone / Desktop)
├── js/
│   ├── app.js              Oberfläche, Klaviatur, Presets-UI, MIDI
│   ├── synth-engine.js     Parameter-Schema, AudioContext, Worklet-Setup, Fallback
│   ├── oscillator.js       PolyBLEP-Oszillatoren, Rauschen        (DSP)
│   ├── filters.js          ZDF-State-Variable-Filter               (DSP)
│   ├── envelopes.js        ADSR-Hüllkurven                         (DSP)
│   ├── lfo.js              LFO                                     (DSP)
│   ├── arpeggiator.js      samplegenauer Arpeggiator               (DSP)
│   ├── audio-worklet.js    Stimmenverwaltung + AudioWorkletProcessor
│   ├── effects.js          Chorus, Delay, Hall
│   └── presets.js          Werkspresets + IndexedDB-Speicher
└── assets/                 Icons
```

Die DSP-Dateien laufen im **AudioWorklet** (eigener Audio-Thread, latenzarm, keine Aussetzer durch UI-Last).
Ist AudioWorklet nicht verfügbar (sehr alte Browser oder Aufruf über `file://`), schaltet die App automatisch
auf einen **Kompatibilitätsmodus** (ScriptProcessor) mit identischem Klang um.

---

## Lokal starten

AudioWorklet benötigt einen „sicheren Kontext“ (`http://localhost` oder `https://`).
Am einfachsten einen kleinen lokalen Webserver im Projektordner starten:

**Python (macOS/Linux meist vorinstalliert):**
```bash
cd virtual-analog-synth
python3 -m http.server 8000
```
Dann im Browser öffnen: <http://localhost:8000>

**Alternativ mit Node.js:**
```bash
npx serve virtual-analog-synth
```

**Ohne Server:** `index.html` kann auch direkt per Doppelklick geöffnet werden – dann läuft der Kompatibilitätsmodus
(funktioniert, aber mit etwas höherer Latenz).

**Auf iPad/iPhone im lokalen Netz testen:** Server wie oben starten und auf dem Gerät `http://<IP-des-Rechners>:8000` öffnen.
Hinweis: Über eine reine `http://`-IP-Adresse (nicht localhost) gilt die Seite nicht als sicherer Kontext –
die App nutzt dann automatisch den Kompatibilitätsmodus. Für volle AudioWorklet-Leistung auf dem Gerät die Seite per HTTPS hosten (z. B. GitHub Pages).

---

## Veröffentlichen

### GitHub Pages

1. Neues Repository auf GitHub anlegen (z. B. `va-synth`).
2. Den **Inhalt** des Ordners `virtual-analog-synth` hochladen (so dass `index.html` im Hauptverzeichnis liegt):
   ```bash
   cd virtual-analog-synth
   git init
   git add .
   git commit -m "VA-1 Synth"
   git branch -M main
   git remote add origin https://github.com/<BENUTZER>/va-synth.git
   git push -u origin main
   ```
   (Oder auf GitHub: „Add file → Upload files“ und alle Dateien/Ordner hineinziehen.)
3. Im Repository: **Settings → Pages → Build and deployment → Source: „Deploy from a branch“**, Branch `main`, Ordner `/ (root)` → **Save**.
4. Nach ca. 1 Minute erreichbar unter `https://<BENUTZER>.github.io/va-synth/`.

Alle Pfade sind relativ – die App funktioniert daher auch in einem Unterverzeichnis.

### Eigener Webserver

Einfach alle Dateien per FTP/SFTP in ein Verzeichnis des Webservers kopieren. Es ist keine serverseitige
Software nötig. Empfohlen: HTTPS aktivieren (Voraussetzung für AudioWorklet und Web MIDI).
Der Server sollte `.js` als `text/javascript` ausliefern (Standard bei Apache, nginx, GitHub Pages, Netlify).

---

## Hinweise für iPhone & iPad (Safari)

- **Stummschalter:** Ab iOS 17 spielt die App auch bei aktivem Stummschalter (Audio-Session „playback“). Bei älteren
  iOS-Versionen den Stummschalter ggf. deaktivieren.
- **Als App nutzen:** In Safari „Teilen → Zum Home-Bildschirm“ – dann startet der Synth im Vollbild ohne Browserleiste.
- **Unterbrechungen** (Anruf, Siri, App-Wechsel): Nach der Rückkehr einfach eine Taste antippen – die Engine setzt automatisch fort.
- **Web MIDI** wird von Safari nicht unterstützt; alles andere funktioniert vollständig.
- Bei sehr vielen Stimmen mit Unison 4 auf älteren iPhones ggf. die Stimmenanzahl reduzieren.

## Browser-Unterstützung

Safari (iOS/iPadOS 14.5+, macOS), Chrome, Edge, Firefox – jeweils aktuelle Versionen.
Ältere Browser ohne AudioWorklet nutzen automatisch den Kompatibilitätsmodus.
