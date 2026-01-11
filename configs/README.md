# MAHO MH400E LinuxCNC Konfiguration - Dateierklärung

Diese Dokumentation erklärt alle Dateien in dieser LinuxCNC-Konfiguration und ihre Funktionen.

---

## ⭐ KRITISCHE KONFIGURATIONSDATEIEN

### MH400e.ini
**Zweck:** Hauptkonfigurationsdatei - enthält ALLE Maschinenparameter  
**Wichtigkeit:** ⭐⭐⭐ KRITISCH - Ohne diese Datei startet LinuxCNC nicht  
**Änderungshäufigkeit:** Gelegentlich (für Tuning, Grenzwerte anpassen)  
**Warnung:** ⚠️ Von PNCconf generiert - kann beim Regenerieren überschrieben werden

**Enthält:**
- Maschinenidentität und Version
- Display-Einstellungen (GUI-Typ, Einheiten, Geschwindigkeiten)
- Bewegungsparameter (Bahnplanung, Zykluszeiten)
- Achsen-/Gelenkkonfiguration (Limits, Geschwindigkeiten, Beschleunigungen)
- HAL-Dateien Ladereihenfolge
- Werkzeugtabellen-Speicherort
- PID-Tuning-Parameter für Servoregelung
- Kinematik-Konfiguration
- MESA 5i25/7i77 Hardware-Einstellungen

### MH400e.hal
**Zweck:** Haupt-Hardware-Abstraktionsschicht - "Verkabelungsplan" in Software  
**Wichtigkeit:** ⭐⭐⭐ KRITISCH  
**Änderungshäufigkeit:** Selten  
**Warnung:** ⚠️ Von PNCconf generiert - kann überschrieben werden

**Funktionen:**
- Lädt Realtime-Komponenten und Treiber (MESA Karten, PID-Regler)
- Verbindet Achsen mit Hardware
- Konfiguriert Encoder-Rückmeldungen
- Verbindet Spindel-Steuerung
- Lädt benutzerdefinierte Gearbox-Komponente
- Definiert Signal-Verbindungen zwischen Komponenten

---

## ✅ BENUTZERDEFINIERTE DATEIEN (Sicher zu ändern)

### custom.hal
**Zweck:** Ihre eigenen HAL-Anpassungen  
**Wichtigkeit:** ⭐⭐ Wichtig für Ihre Zusätze  
**Änderungshäufigkeit:** Nach Bedarf  
**Vorteil:** ✅ Überlebt PNCconf-Regenerierung

Nutzen Sie diese Datei für:
- Zusätzliche Hardware-Verbindungen
- Custom I/O-Konfiguration
- Eigene Signal-Verkabelung
- Zusätzliche Komponenten laden

**Aktuell implementiert:** Schmiermangel-Interlock (NC-Sensor auf IN29)
- Eingang `hm2_5i25.0.7i77.0.0.input-29` wird entprellt (`debounce`, 50 Zyklen) und invertiert (`not`).
- Aktives Signal `lube-low-active` hält den Vorschub an (`motion.feed-inhibit`).
- Ein One-Shot triggert bei Aktivierung eine MDI-Meldung (`M118 Schmierung niedrig – Vorschub angehalten`) über `halui.mdi-command-00`.

### custom_postgui.hal
**Zweck:** HAL-Verbindungen die NACH dem GUI-Start ausgeführt werden  
**Wichtigkeit:** ⭐⭐ Für GUI-bezogene Anpassungen  
**Änderungshäufigkeit:** Nach Bedarf  
**Vorteil:** ✅ Überlebt PNCconf-Regenerierung

Nutzen Sie diese Datei für:
- Verbindungen zu GUI-Elementen (Buttons, LEDs, etc.)
- Verknüpfungen mit PyVCP/GladeVCP Panels
- Anzeigeelemente

### custom_gvcp.hal
**Zweck:** Spezielle Verbindungen für GladeVCP Custom Panels  
**Wichtigkeit:** ⭐ Optional  
**Änderungshäufigkeit:** Bei GladeVCP-Nutzung

### shutdown.hal
**Zweck:** Wird beim LinuxCNC-Herunterfahren ausgeführt  
**Wichtigkeit:** ⭐⭐ Für sichere Abschaltung  
**Änderungshäufigkeit:** Selten

Nutzen Sie diese Datei für:
- Hardware in sicheren Zustand versetzen beim Beenden
- Ausgänge deaktivieren
- Position speichern

---

## 🔧 WERKZEUG- UND STATUSDATEIEN

### tool.tbl
**Zweck:** Werkzeugdatenbank mit Abmessungen  
**Wichtigkeit:** ⭐⭐ Wichtig für korrekte Bearbeitung  
**Änderungshäufigkeit:** ✅ HÄUFIG - Bei jedem Werkzeugwechsel/-vermessung

**Format:**
```
T1 P1 Z123.456 D12.7 ;Kommentar
```
- T = Werkzeugnummer
- P = Platz im Werkzeugwechsler
- Z = Längenkorrektur (mm)
- D = Durchmesser (mm)
- ; = Beschreibung des Werkzeugs

**Aktualisierung:** Messen Sie Werkzeuge und tragen Sie die Werte hier ein

### linuxcnc.var
**Zweck:** Persistente G-Code Variablen  
**Wichtigkeit:** ⭐⭐ Speichert wichtige Daten  
**Änderungshäufigkeit:** ❌ NIEMALS MANUELL BEARBEITEN - automatisch verwaltet

**Speichert:**
- Werkstück-Koordinatensysteme (G54-G59.3)
- Werkzeugoffsets
- Tastergebnisse
- Benutzerdefinierte Variablen

**Backup:** `.var.bak` wird automatisch erstellt

### linuxcnc.var.bak
**Zweck:** Automatisches Backup der `.var` Datei  
**Wichtigkeit:** ⭐ Sicherheitskopie  
**Änderungshäufigkeit:** Automatisch bei jedem LinuxCNC-Start

---

## 🔩 CUSTOM KOMPONENTEN (Fortgeschritten)

### mh400e-linuxcnc-master/ Verzeichnis
Enthält die benutzerdefinierte Gearbox-Steuerung für die MH400E

#### mh400e_gearbox.comp
**Zweck:** Hauptkomponente für automatische Getriebesteuerung  
**Wichtigkeit:** ⭐⭐⭐ KRITISCH für Spindelbetrieb  
**Änderungshäufigkeit:** Sehr selten (nur bei Änderungen der Getriebe-Logik)  
**Typ:** Realtime HAL-Komponente in C

**Funktionen:**
- Automatisches Schalten der Gänge basierend auf Spindeldrehzahl
- Überwacht 12 Positionssensoren (4 pro Getriebestufe)
- Steuert 8 Ausgänge (Motoren, Kupplungen, Richtungsumkehr)
- Implementiert "Twitching" (Mikrobewegungen zum Ausrichten der Zahnräder)
- Verhindert Beschädigung durch falsche Gangwechsel

**Kompilierung:** Wird zu `.so` Datei kompiliert und in HAL geladen

#### mh400e_gearbox_sim.comp
**Zweck:** Simulationsversion für Tests ohne Hardware  
**Wichtigkeit:** ⭐ Nur für Entwicklung/Tests

#### Unterstützende C-Dateien:
- **mh400e_gears.c/.h** - Gangwahl-Logik und Tabellen
- **mh400e_twitch.c/.h** - Zahnrad-Ausrichtungsalgorithmen  
- **mh400e_util.c/.h** - Hilfsfunktionen
- **mh400e_common.h** - Gemeinsame Definitionen

#### Makefile
**Zweck:** Build-Skript zum Kompilieren der Komponenten  
**Nutzung:** `make` im Verzeichnis ausführen

#### README.md
**Zweck:** Englische Dokumentation der Gearbox-Komponente

#### mh400e_gearbox.xml
**Zweck:** Dokumentation für HAL-Komponenten-Viewer

#### LICENSE, COPYING
**Zweck:** Lizenzinformationen (GPL v2)

---

## 📋 KONFIGURATIONSGENERATOR

### MH400e.pncconf
**Zweck:** PNCconf-Konfigurationsdaten (XML-Format)  
**Wichtigkeit:** ⭐⭐ Zum Regenerieren der Konfiguration  
**Änderungshäufigkeit:** Bei Hardware-Änderungen  
**Warnung:** ⚠️ Nur über PNCconf-GUI bearbeiten!

**Enthält:**
- Alle Maschinenparameter in strukturierter Form
- Hardware-Zuordnungen (MESA Karten, I/O)
- Wird verwendet um `.ini` und `.hal` Dateien neu zu generieren

**Nutzung:** 
1. PNCconf starten
2. Diese Datei laden
3. Änderungen vornehmen
4. "Apply" klicken → regeneriert `.ini` und `.hal`

---

## 💾 AUTOMATISCH GESPEICHERTE EINSTELLUNGEN

### autosave.halscope
**Zweck:** HALscope Anzeige-Einstellungen  
**Wichtigkeit:** ⭐ Komfort  
**Änderungshäufigkeit:** Automatisch  
**Status:** ✅ Sicher zu löschen - wird neu erstellt

Speichert:
- Welche Signale angezeigt werden
- Zeitbasis und Trigger-Einstellungen
- Fenstergröße

### halshow.preferences
**Zweck:** HALshow Fenster-Layout und Beobachtungsliste  
**Wichtigkeit:** ⭐ Komfort  
**Änderungshäufigkeit:** Automatisch  
**Status:** ✅ Sicher zu löschen - wird neu erstellt

Speichert:
- Fensterposition und -größe
- Erweiterte/Zusammengeklappte Bereiche
- "Watch"-Liste für häufig betrachtete Signale

---

## 📁 VERZEICHNISSE

### backups/
**Zweck:** Für manuelle Konfigurationsbackups  
**Status:** Aktuell leer  
**Empfehlung:** Nutzen Sie dieses Verzeichnis um wichtige Dateien zu sichern bevor Sie größere Änderungen vornehmen

**Empfohlene Backups:**
```bash
# Backup vor Änderungen erstellen:
cp MH400e.ini backups/MH400e.ini.$(date +%Y%m%d_%H%M%S)
cp MH400e.hal backups/MH400e.hal.$(date +%Y%m%d_%H%M%S)
cp tool.tbl backups/tool.tbl.$(date +%Y%m%d_%H%M%S)
```

### mh400e-linuxcnc-master/doc/
**Zweck:** Zusätzliche Dokumentation für die Gearbox-Komponente  
**Inhalt:** Diagramme, Erklärungen, Timing-Informationen

---

## 📊 ÜBERSICHTSTABELLE

| Datei | Zweck | Wichtigkeit | Ändern? | Häufigkeit |
|-------|-------|-------------|---------|------------|
| MH400e.ini | Maschinenparameter | ⭐⭐⭐ | ⚠️ Vorsicht | Gelegentlich |
| MH400e.hal | Hardware-Verkabelung | ⭐⭐⭐ | ⚠️ Vorsicht | Selten |
| custom.hal | Ihre HAL-Zusätze | ⭐⭐ | ✅ Ja | Nach Bedarf |
| custom_postgui.hal | GUI HAL-Zusätze | ⭐⭐ | ✅ Ja | Nach Bedarf |
| custom_gvcp.hal | GladeVCP Verbindungen | ⭐ | ✅ Ja | Bei Bedarf |
| shutdown.hal | Abschalt-Routine | ⭐⭐ | ✅ Ja | Selten |
| tool.tbl | Werkzeugdaten | ⭐⭐ | ✅ Ja | **Häufig** |
| linuxcnc.var | G-Code Variablen | ⭐⭐ | ❌ Nein | Automatisch |
| MH400e.pncconf | PNCconf Daten | ⭐⭐ | ⚠️ Nur via GUI | Bei HW-Änderung |
| mh400e_gearbox.comp | Getriebe-Komponente | ⭐⭐⭐ | ⚠️ Experten | Sehr selten |
| autosave.halscope | HALscope Einst. | ⭐ | - | Automatisch |
| halshow.preferences | HALshow Einst. | ⭐ | - | Automatisch |

---

## 🔄 DATEI-ABHÄNGIGKEITEN

```
MH400e.ini (Master-Konfiguration)
  │
  ├─→ Lädt: MH400e.hal
  │          ├─→ Lädt MESA Treiber (hm2_pci)
  │          ├─→ Lädt mh400e_gearbox.so
  │          └─→ Verbindet alle Signale
  │
  ├─→ Lädt: custom.hal (Ihre Anpassungen)
  │
  ├─→ Startet: GUI (AXIS oder andere)
  │
  ├─→ Lädt: custom_postgui.hal (nach GUI-Start)
  │
  ├─→ Lädt: custom_gvcp.hal (für Custom Panels)
  │
  ├─→ Beim Beenden: shutdown.hal
  │
  ├─→ Referenziert: tool.tbl (Werkzeugdaten)
  │
  └─→ Referenziert: linuxcnc.var (Persistente Daten)
```

---

## 🛠️ WARTUNGS-CHECKLISTE

### Täglich / Bei jedem Werkzeugwechsel:
- [ ] `tool.tbl` aktualisieren mit neuen Werkzeugmaßen

### Wöchentlich / Monatlich:
- [ ] Backup von `linuxcnc.var` erstellen (Werkstück-Nullpunkte!)
- [ ] Backup von `tool.tbl` erstellen

### Bei Tuning/Optimierung:
- [ ] Änderungen in `MH400e.ini` vornehmen
- [ ] Backup VOR der Änderung erstellen
- [ ] Testen und dokumentieren

### Bei Hardware-Änderungen:
- [ ] PNCconf öffnen (`MH400e.pncconf` laden)
- [ ] Änderungen vornehmen
- [ ] Apply klicken
- [ ] `custom*.hal` Dateien prüfen (werden nicht überschrieben)
- [ ] Testen!

### Bei Software-Updates:
- [ ] Komplettes Backup aller Dateien
- [ ] Besonders: `.var`, `.tbl`, `custom*.hal`

---

## ⚡ SCHNELLREFERENZ: WAS TUN WENN...

### "Ich möchte ein neues Signal/I/O hinzufügen"
→ In `custom.hal` oder `custom_postgui.hal` eintragen

### "Ich möchte Geschwindigkeiten/Beschleunigungen ändern"
→ `MH400e.ini` bearbeiten (oder via PNCconf)

### "Ich habe ein neues Werkzeug"
→ `tool.tbl` aktualisieren

### "Ich möchte die Hardware-Konfiguration ändern"
→ PNCconf öffnen, `MH400e.pncconf` laden, ändern, Apply

### "LinuxCNC startet nicht mehr"
→ Prüfen: `.ini` Datei, HAL Dateien auf Syntax-Fehler  
→ Terminal öffnen: `linuxcnc MH400e.ini` für Fehlermeldungen

### "Ich habe einen Fehler in .ini oder .hal gemacht"
→ Backup wiederherstellen ODER PNCconf neu generieren lassen

### "Werkstück-Nullpunkte sind weg"
→ `linuxcnc.var.bak` nach `linuxcnc.var` kopieren

---

## 📞 TECHNISCHE DETAILS

**Hardware:**
- CNC Controller: MESA 5i25 (PCI FPGA Karte)
- I/O Interface: MESA 7i77 (Servo/Analog Interface)
- Achsen: 3 (X, Y, Z) mit geschlossenem Regelkreis
- Encoder: Direkt an Servomotoren
- Spindel: Mechanisches Getriebe mit automatischer Steuerung

**Mesa I/O Zuordnung (aus MH400e.hal, Klemmen laut 7i77-Layout):**

Digitale Ausgänge (TB8 OUT0..15, Open-Collector):

| Funktion                     | HAL-Pin                       | 7i77-Klemme |
| ---------------------------- | ----------------------------- | ----------- |
| frei                         | hm2_5i25.0.7i77.0.0.output-00 | OUT0 (TB8)  |
| machine-is-on (Relais 19K1)  | hm2_5i25.0.7i77.0.0.output-01 | OUT1 (TB8)  |
| tool-unclamp (M64/M65 P2)    | hm2_5i25.0.7i77.0.0.output-02 | OUT2 (TB8)  |
| frei                         | hm2_5i25.0.7i77.0.0.output-03 | OUT3 (TB8)  |
| spindle-on                   | hm2_5i25.0.7i77.0.0.output-04 | OUT4 (TB8)  |
| spindle-cw                   | hm2_5i25.0.7i77.0.0.output-05 | OUT5 (TB8)  |
| spindle-ccw                  | hm2_5i25.0.7i77.0.0.output-06 | OUT6 (TB8)  |
| coolant-flood                | hm2_5i25.0.7i77.0.0.output-07 | OUT7 (TB8)  |
| set-shaft-motor-lowspeed     | hm2_5i25.0.7i77.0.0.output-08 | OUT8 (TB8)  |
| activate-reducer-motor       | hm2_5i25.0.7i77.0.0.output-09 | OUT9 (TB8)  |
| activate-midrange-motor      | hm2_5i25.0.7i77.0.0.output-10 | OUT10 (TB8) |
| activate-input-stage-motor   | hm2_5i25.0.7i77.0.0.output-11 | OUT11 (TB8) |
| set-reverse-shaft-motor      | hm2_5i25.0.7i77.0.0.output-12 | OUT12 (TB8) |
| set-gear-shift-start         | hm2_5i25.0.7i77.0.0.output-13 | OUT13 (TB8) |
| activate-spindle-twitch-cw   | hm2_5i25.0.7i77.0.0.output-14 | OUT14 (TB8) |
| activate-spindle-twitch-ccw  | hm2_5i25.0.7i77.0.0.output-15 | OUT15 (TB8) |

Digitale Eingänge (TB6 IN0..15, TB7 IN16..31, Sinking):

| Funktion              | HAL-Pin                      | 7i77-Klemme |
| --------------------- | ---------------------------- | ----------- |
| frei                  | hm2_5i25.0.7i77.0.0.input-00 | IN0 (TB6)   |
| spindle-stopped       | hm2_5i25.0.7i77.0.0.input-01 | IN1 (TB6)   |
| frei                  | hm2_5i25.0.7i77.0.0.input-02 | IN2 (TB6)   |
| estop-chain-ok (24V)  | hm2_5i25.0.7i77.0.0.input-03 | IN3 (TB6)   |
| frei                  | hm2_5i25.0.7i77.0.0.input-04 | IN4 (TB6)   |
| frei                  | hm2_5i25.0.7i77.0.0.input-05 | IN5 (TB6)   |
| frei                  | hm2_5i25.0.7i77.0.0.input-06 | IN6 (TB6)   |
| frei                  | hm2_5i25.0.7i77.0.0.input-07 | IN7 (TB6)   |
| frei                  | hm2_5i25.0.7i77.0.0.input-08 | IN8 (TB6)   |
| frei                  | hm2_5i25.0.7i77.0.0.input-09 | IN9 (TB6)   |
| frei                  | hm2_5i25.0.7i77.0.0.input-10 | IN10 (TB6)  |
| frei                  | hm2_5i25.0.7i77.0.0.input-11 | IN11 (TB6)  |
| frei                  | hm2_5i25.0.7i77.0.0.input-12 | IN12 (TB6)  |
| frei                  | hm2_5i25.0.7i77.0.0.input-13 | IN13 (TB6)  |
| frei                  | hm2_5i25.0.7i77.0.0.input-14 | IN14 (TB6)  |
| frei                  | hm2_5i25.0.7i77.0.0.input-15 | IN15 (TB6)  |
| reducer-left          | hm2_5i25.0.7i77.0.0.input-16 | IN16 (TB7)  |
| reducer-right         | hm2_5i25.0.7i77.0.0.input-17 | IN17 (TB7)  |
| reducer-center        | hm2_5i25.0.7i77.0.0.input-18 | IN18 (TB7)  |
| reducer-left-center   | hm2_5i25.0.7i77.0.0.input-19 | IN19 (TB7)  |
| middle-left           | hm2_5i25.0.7i77.0.0.input-20 | IN20 (TB7)  |
| middle-right          | hm2_5i25.0.7i77.0.0.input-21 | IN21 (TB7)  |
| middle-center         | hm2_5i25.0.7i77.0.0.input-22 | IN22 (TB7)  |
| middle-left-center    | hm2_5i25.0.7i77.0.0.input-23 | IN23 (TB7)  |
| input-left            | hm2_5i25.0.7i77.0.0.input-24 | IN24 (TB7)  |
| input-right           | hm2_5i25.0.7i77.0.0.input-25 | IN25 (TB7)  |
| input-center          | hm2_5i25.0.7i77.0.0.input-26 | IN26 (TB7)  |
| input-left-center     | hm2_5i25.0.7i77.0.0.input-27 | IN27 (TB7)  |
| frei                  | hm2_5i25.0.7i77.0.0.input-28 | IN28 (TB7)  |
| frei                  | hm2_5i25.0.7i77.0.0.input-29 | IN29 (TB7)  |
| frei                  | hm2_5i25.0.7i77.0.0.input-30 | IN30 (TB7)  |
| frei                  | hm2_5i25.0.7i77.0.0.input-31 | IN31 (TB7)  |

**E-Stop (Hardware-only):** IN3 (hm2_5i25.0.7i77.0.0.input-03) führt 24V, solange der Not-Aus-Kreis geschlossen ist. Dieser Eingang steuert direkt `iocontrol.0.emc-enable-in`; bei Spannungsverlust (Not-Aus gedrückt) fällt LinuxCNC sofort in E-Stop.

Analoge Ausgänge (TB5 AOUT0..5 + ENA0..5):

| Funktion                 | HAL-Pin                         | 7i77-Klemme               |
| ------------------------ | ------------------------------- | ------------------------- |
| X Antrieb                | hm2_5i25.0.7i77.0.1.analogout0  | AOUT0 / ENA0 (TB5)        |
| Y Antrieb                | hm2_5i25.0.7i77.0.1.analogout1  | AOUT1 / ENA1 (TB5)        |
| Z Antrieb                | hm2_5i25.0.7i77.0.1.analogout2  | AOUT2 / ENA2 (TB5)        |
| Enable alle Analogkanäle | hm2_5i25.0.7i77.0.1.analogena | ENA0..5 (gemeinsame Klemme) |

Encoder (SubD/Stecker pro Achse):

| Funktion    | HAL-Pin                    | 7i77-Anschluss |
| ----------- | -------------------------- | -------------- |
| X Feedback  | hm2_5i25.0.encoder.00.*    | ENC0           |
| Y Feedback  | hm2_5i25.0.encoder.01.*    | ENC1           |
| Z Feedback  | hm2_5i25.0.encoder.02.*    | ENC2           |

**Software:**
- LinuxCNC Version: Wie in `.ini` definiert
- Realtime: PREEMPT-RT oder RTAI
- GUI: AXIS (Standard) oder andere

**Besonderheiten:**
- Custom Gearbox-Komponente für MAHO MH400E Spindel
- Automatisches Gangwechseln
- Überwachung der Getriebeposition
- Twitch-Funktion für präzises Ausrichten

---

## 📚 WEITERE INFORMATIONEN

- HAL-Konzepte: LinuxCNC Documentation → HAL Tutorial
- INI-Datei Referenz: LinuxCNC Documentation → INI Configuration
- PNCconf: LinuxCNC Documentation → Configuration Wizards
- Gearbox-Komponente: `mh400e-linuxcnc-master/README.md`

---

**Letzte Aktualisierung:** 09.01.2026  
**Konfiguration für:** MAHO MH400E Fräsmaschine  
**Erstellt von:** GitHub Copilot
