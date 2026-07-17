# 🔋 Marstek Venus E3 – Nulleinspeisung Blueprint
### Home Assistant Blueprint für dynamische Eigenverbrauchsoptimierung

[![HA Blueprint Import](https://img.shields.io/badge/Home%20Assistant-Blueprint%20importieren-41BDF5?style=for-the-badge&logo=home-assistant)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2FHatchetMan111%2Fmarstek-venus-e3-nulleinspeisung-blueprint%2Fmain%2Fblueprints%2Fautomation%2Fmarstek_venus_e3_nulleinspeisung.yaml)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![HA Version](https://img.shields.io/badge/Home%20Assistant-2024.6%2B-blue)](https://www.home-assistant.io/)

---

## 📋 Was macht dieser Blueprint?

Dieser Blueprint regelt den **Marstek Venus E3 Heimspeicher** vollautomatisch, sodass du **keinen Strom ins Netz einspeist** und deinen Eigenverbrauch maximierst.

Er misst kontinuierlich die aktuelle Netzleistung und passt die Lade- und Entladeleistung des Speichers in Echtzeit an – über die Register **Force Mode** und **RS485-Steuermodus** deiner Marstek-Integration.

---

## ✅ Funktionen

| Funktion | Beschreibung |
|---|---|
| ⚡ Nulleinspeisung | Entladung folgt dem Hausverbrauch – kein Strom geht ans Netz |
| ☀️ Überschussladen | Solarüberschuss wird automatisch in die Batterie gespeichert |
| ⚠️ Tiefentladeschutz | Entladung stoppt unter konfigurierbarem Mindest-SoC |
| 🌙 Nacht-Reserve | Hält SoC-Reserve für den Morgen zurück |
| 🔒 Netzladen-Sperre | Batterie wird nicht teuer aus dem Netz geladen |
| 🎯 Totzone/Hysterese | Verhindert ständiges Regeln bei stabiler Lage |
| 🛠️ Diagnose-Logging | Optionale Log-Ausgabe für Fehlersuche |

---

## 🔧 Voraussetzungen

- **Home Assistant** 2024.6 oder neuer
- **Marstek Venus E3** in Home Assistant integriert, z. B. über [Marstek Venus Modbus (ViperRNMC)](https://github.com/ViperRNMC/marstek_venus_modbus)
- **Netzmessgerät** mit HA-Integration, das Netzbezug und Einspeisung misst

> ⚠️ Je nach genutzter Marstek-Integration heißen die Entitäten unterschiedlich und liegen in unterschiedlichen Domains vor (z. B. RS485-Steuermodus als **Switch** bei ViperRNMC, als **Select** bei manchen anderen Integrationen). Prüfe deine Entitäten unter Entwicklertools → Zustände, bevor du das Blueprint konfigurierst.

### Kompatible Netzmessgeräte (Beispiele)

| Gerät | Typ | Hinweis |
|---|---|---|
| Shelly EM / 3EM | WLAN-Stromzähler | Direkt kompatibel (kombinierter Sensor) |
| Eastron SDM630 | RS485-Zähler | Direkt kompatibel (kombinierter Sensor) |
| Holley DTZ541 | Smart Meter | Direkt kompatibel |
| ISKRA MT174 | Smart Meter | Getrennte Sensoren (Bezug + Einspeisung) |
| **Tibber Pulse** | IR-Lesekopf | Getrennte Sensoren – siehe Hinweis unten |
| Discovergy | Cloud-API | Getrennte Sensoren, höhere Latenz |

Das Blueprint unterstützt beide Varianten direkt über den Schalter **"Sensor-Typ"** (Kombiniert / Getrennt) – ein zusätzlicher Template-Sensor ist normalerweise **nicht** mehr nötig.

#### ⚠️ Tibber Pulse – Hinweis

Tibber Pulse liefert zwei separate Sensoren (Bezug + Einspeisung, z. B. `sensor.power_consumption` und `sensor.power_production`). Wähle im Blueprint einfach **Sensor-Typ = Getrennt** und trage beide Sensoren getrennt ein – kein zusätzlicher Template-Sensor erforderlich.

Falls du dennoch einen kombinierten Sensor bevorzugst, kannst du optional folgenden Template-Sensor in deine `configuration.yaml` einfügen:

```yaml
template:
  - sensor:
      - name: "Netzleistung Netto"
        unique_id: "netzleistung_netto_tibber"
        unit_of_measurement: "W"
        device_class: power
        state_class: measurement
        state: >
          {{ states('sensor.power_consumption') | float(0)
           - states('sensor.power_production') | float(0) }}
```

> Ersetze die Entity-IDs durch deine tatsächlichen Tibber-Sensornamen.
> Danach HA neu laden – dann erscheint `sensor.netzleistung_netto` zur Auswahl im Blueprint (Sensor-Typ = Kombiniert).

---

## 🚀 Installation

### Option 1: Ein-Klick-Import (empfohlen)

[![Zu Home Assistant hinzufügen](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2FHatchetMan111%2Fmarstek-venus-e3-nulleinspeisung-blueprint%2Fmain%2Fblueprints%2Fautomation%2Fmarstek_venus_e3_nulleinspeisung.yaml)

### Option 2: Manuell

1. YAML-Datei herunterladen: [`marstek_venus_e3_nulleinspeisung.yaml`](blueprints/automation/marstek_venus_e3_nulleinspeisung.yaml)
2. Datei nach `config/blueprints/automation/` kopieren
3. Home Assistant neu laden: **Entwicklertools → YAML → Blueprints neu laden**
4. **Einstellungen → Automatisierungen → + Erstellen → Aus Blueprint**

> 🔄 **Update-Hinweis:** Ein einmal importiertes Blueprint wird **nicht** automatisch aktualisiert, wenn sich die Datei auf GitHub ändert. Nutze bei per URL importierten Blueprints die Funktion **"Blueprint erneut importieren"** (drei Punkte am Blueprint), um die neueste Version zu ziehen. Bestehende Automatisierungen übernehmen die aktualisierte Logik automatisch, sobald die lokale Blueprint-Datei aktualisiert ist.

---

## ⚙️ Konfiguration

Nach dem Import die Automatisierung anlegen und folgende Werte zuweisen:

### Pflichtfelder

| Feld | Beschreibung |
|---|---|
| Sensor-Typ | Kombiniert (ein Sensor) oder Getrennt (zwei Sensoren) |
| Netzleistung(s-Sensor/en) | Je nach Sensor-Typ ein oder zwei Sensoren |
| Batterie-Ladezustand | SoC-Sensor vom Marstek (z. B. `sensor.batterie_ladezustand`) |
| Ladeleistung einstellen | Number-Entität vom Marstek |
| Entladeleistung einstellen | Number-Entität vom Marstek |
| **Force Mode** | **Select**-Entität vom Marstek (Hauptschalter Laden/Entladen/Standby) |
| **RS485-Steuermodus** | **Switch**-Entität vom Marstek – bei ViperRNMC z. B. `switch.marstek_venus_e_rs485_control_mode` |

### Empfohlene Einstellungen (Standardwerte)

| Parameter | Standard | Empfehlung |
|---|---|---|
| Ziel-Netzleistung | 30 W | 0–50 W (Sicherheitspuffer) |
| Totzone / Deadband | 50 W | 30–60 W |
| Schrittweite Entladen | 100 W | 80–150 W |
| Schrittweite Laden | 80 W | 70–120 W |
| Verzögerung nach Regelschritt | 7 s | 5–10 s |
| Max. Ladeleistung | 2400 W | ≤ 2500 W |
| Max. Entladeleistung | 2400 W | ≤ 2500 W |
| Tiefentladeschutz (Mindest-SoC) | 10 % | 10–15 % |
| Nacht-Reserve | 20 % | 15–30 % |

---

## 🧠 Wie funktioniert die Regellogik?

```
Alle 15 Sekunden / bei Sensor-Änderung:
│
├─ P1: SoC ≤ Mindest-SoC?               → 🛑 Force Mode = Standby, Entladen = 0
├─ P2: Nacht & SoC ≤ Nacht-Reserve?     → 🌙 Force Mode = Standby, Entladen = 0
├─ P3: Abweichung innerhalb Totzone?    → 🎯 Force Mode = Standby (Eigenregelung Marstek)
├─ P4: Netzbezug über Totzone?          → ⚡ Force Mode = Entladen, Leistung +Schrittweite
└─ P5: Einspeisung über Totzone?        → 🔋 Force Mode = Laden, Leistung +Schrittweite
```

Die Prioritäten P1–P5 werden in dieser Reihenfolge geprüft; die erste zutreffende Bedingung gewinnt. Vor jedem Regelzyklus wird zusätzlich der RS485-Steuermodus (Switch) eingeschaltet, da der Marstek sonst alle Befehle ignoriert.

---

## 🗂️ Repo-Struktur

```
📁 marstek-venus-e3-nulleinspeisung-blueprint/
  📄 README.md
  📄 LICENSE
  📁 blueprints/
    📁 automation/
      📄 marstek_venus_e3_nulleinspeisung.yaml
```

---

## 🐛 Fehlerbehebung

**Blueprint-Import schlägt fehl:**
Stelle sicher, dass die Verknüpfung mit my.home-assistant.io aktiv ist: Einstellungen → System → Netzwerk. Alternativ die YAML-Datei manuell nach `config/blueprints/automation/` kopieren (Option 2 oben).

**"Invalid blueprint: expected str … Got None":**
Ein Feld im Blueprint-Kopf (z. B. `author`) ist leer statt mit einem Text oder ganz weggelassen. Entweder einen Textwert eintragen oder die Zeile entfernen.

**RS485-Steuermodus wird nicht gefunden / keine passende Entität:**
Prüfe zuerst die Domain deiner Entität unter Entwicklertools → Zustände. Bei der Integration "Marstek Venus Modbus" (ViperRNMC) ist es ein **Switch**, kein Select – das Blueprint erwartet entsprechend `domain: switch`. Nutzt du eine andere Integration mit Select-Entität, lege einen Hilfs-Switch (Template-Switch) an, der die passende Select-Option setzt.

**Force Mode reagiert nicht / falscher Options-Text:**
Prüfe den exakten Options-Namen unter Entwicklertools → Zustände → deine Force-Mode-Entität → Attribut `options`. Bei ViperRNMC lauten die Werte `None` / `Charge` / `Discharge` (Groß-/Kleinschreibung beachten).

**Regler schwingt (lädt/entlädt ständig abwechselnd):**
Totzone auf 60–80 W erhöhen, Schrittweiten verringern, Verzögerung nach Regelschritt erhöhen (8–10 s).

**Einspeisung trotz aktivem Blueprint:**
Ziel-Netzleistung auf 50 W setzen, Totzone auf 50 W, prüfen ob RS485-Steuermodus (Switch) tatsächlich eingeschaltet ist.

---

## 📊 Changelog

| Version | Datum | Änderung |
|---|---|---|
| 1.0.0 | 2025-04 | Erstveröffentlichung |
| 1.1.0 | 2026-07 | Fix: RS485-Steuermodus als Switch statt Select (ViperRNMC-Integration); Fix: fehlende Verzögerungs-Variable (`delay_after_step` wurde nicht verwendet); `author`-Feld korrigiert; README an tatsächliche Blueprint-Felder angepasst |

---

## 📄 Lizenz

MIT License – Details siehe [LICENSE](LICENSE)

---

## ⚡ Entwickelt von

**TheHatchetMan**

---

*Gefällt dir der Blueprint? ⭐ Star auf GitHub und teile ihn in der HA Community!*
