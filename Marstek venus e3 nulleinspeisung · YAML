# 🔋 Marstek Venus E3 – Nulleinspeisung Blueprint
### Home Assistant Blueprint für dynamische Eigenverbrauchsoptimierung

[![HA Blueprint Import](https://img.shields.io/badge/Home%20Assistant-Blueprint%20importieren-41BDF5?style=for-the-badge&logo=home-assistant)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2FHatchetMan111%2Fmarstek-venus-e3-nulleinspeisung-blueprint%2Fmain%2Fblueprints%2Fautomation%2Fmarstek_venus_e3_nulleinspeisung.yaml)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![HA Version](https://img.shields.io/badge/Home%20Assistant-2024.6%2B-blue)](https://www.home-assistant.io/)
[![Developed by](https://img.shields.io/badge/Entwickelt%20von-LichtValleyApps.de-FFD234?style=flat)](https://lichtvalleyapps.de)

---

## 📋 Was macht dieser Blueprint?

Dieser Blueprint regelt den **Marstek Venus E3 Heimspeicher** vollautomatisch, sodass du **keinen Strom ins Netz einspeist** und deinen Eigenverbrauch maximierst.

Er misst kontinuierlich die aktuelle Netzleistung und passt die Lade- und Entladeleistung des Speichers in Echtzeit an.

---

## ✅ Funktionen

| Funktion | Beschreibung |
|---|---|
| ⚡ Nulleinspeisung | Entladung folgt dem Hausverbrauch – kein Strom geht ans Netz |
| ☀️ Überschussladen | Solarüberschuss wird automatisch in die Batterie gespeichert |
| ⚠️ Tiefentladeschutz | Entladung stoppt unter konfigurierbarem Mindest-SoC |
| 🌙 Nacht-Reserve | Hält SoC-Reserve für den Morgen zurück |
| 🔒 Netzladen-Sperre | Batterie wird nicht teuer aus dem Netz geladen |
| 🎯 Totzone/Hysterese | Verhindert ständiges Reglern bei stabiler Lage |
| 🛠️ Diagnose-Logging | Optionale Log-Ausgabe für Fehlersuche |

---

## 🔧 Voraussetzungen

- **Home Assistant** 2024.6 oder neuer
- **Marstek Venus E3** in Home Assistant integriert (Modbus/RS485-Integration)
- **Netzmessgerät** mit HA-Integration, das Netzbezug und Einspeisung misst

### Kompatible Netzmessgeräte (Beispiele)

| Gerät | Typ | Hinweis |
|---|---|---|
| Shelly EM / 3EM | WLAN-Stromzähler | Direkt kompatibel |
| Eastron SDM630 | RS485-Zähler | Direkt kompatibel |
| Holley DTZ541 | Smart Meter | Direkt kompatibel |
| ISKRA MT174 | Smart Meter | Direkt kompatibel |
| **Tibber Pulse** | IR-Lesekopf | ⚠️ Siehe Hinweis unten |
| Discovergy | Cloud-API | Funktioniert, höhere Latenz |

#### ⚠️ Tibber Pulse – Wichtiger Hinweis

Tibber Pulse liefert oft **zwei separate Sensoren** (Bezug + Einspeisung). Du brauchst einen kombinierten Sensor. Füge diesen Template-Sensor in deine `configuration.yaml` ein:

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
> Danach HA neu laden – dann erscheint `sensor.netzleistung_netto` zur Auswahl im Blueprint.

---

## 🚀 Installation

### Option 1: Ein-Klick-Import (empfohlen)

[![Zu Home Assistant hinzufügen](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2FHatchetMan111%2Fmarstek-venus-e3-nulleinspeisung-blueprint%2Fmain%2Fblueprints%2Fautomation%2Fmarstek_venus_e3_nulleinspeisung.yaml)

### Option 2: Manuell

1. YAML-Datei herunterladen: [`marstek_venus_e3_nulleinspeisung.yaml`](blueprints/automation/marstek_venus_e3_nulleinspeisung.yaml)
2. Datei nach `config/blueprints/automation/lichtvalleyapps/` kopieren
3. Home Assistant neu laden: **Entwicklertools → YAML neu laden → Blueprints**
4. **Einstellungen → Automatisierungen → + Erstellen → Aus Blueprint**

---

## ⚙️ Konfiguration

Nach dem Import die Automatisierung anlegen und folgende Werte zuweisen:

### Pflichtfelder

| Feld | Beschreibung |
|---|---|
| Netzleistung Sensor | Dein Stromzähler-Sensor (positiv = Bezug, negativ = Einspeisung) |
| Batterie-Ladezustand | `sensor.batterie_ladezustand` vom Marstek |
| Ladeleistung einstellen | `number.ladeleistung_einstellen` vom Marstek |
| Entladeleistung einstellen | `number.entladeleistung_einstellen` vom Marstek |
| Modus Eigenverbrauch | `switch.modus_eigenverbrauch` vom Marstek |
| RS485-Steuermodus | `select.rs485_steuermodus` vom Marstek |

### Empfohlene Einstellungen (Standardwerte)

| Parameter | Standard | Empfehlung |
|---|---|---|
| Ziel-Netzleistung | 30 W | 0–50 W (Sicherheitspuffer) |
| Totzone | 40 W | 30–60 W |
| Regelungsverstärkung | 1.1 | 1.0–1.2 |
| Max. Ladeleistung | 2400 W | ≤ 2500 W |
| Max. Entladeleistung | 2400 W | ≤ 2500 W |
| Tiefentladeschutz | 10 % | 10–15 % |
| Nacht-Reserve | 20 % | 15–30 % |

---

## 🧠 Wie funktioniert die Regellogik?

```
Alle 20 Sekunden / bei Sensor-Änderung:
│
├─ SoC ≤ Mindest-SoC?          → 🛑 Entladen stopp, 100 W Laden
├─ Nacht & SoC ≤ Nacht-Reserve? → 🌙 Alles pausieren
├─ Netz > Ziel + Totzone?       → ⚡ Entladeleistung erhöhen
├─ Netz < Ziel - Totzone?       → 🔋 Ladeleistung erhöhen
└─ In der Totzone?              → 🎯 Sanfte 30%-Feinjustierung
```

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
Stelle sicher, dass du `my.home-assistant.io` in deiner HA-Instanz aktiviert hast:  
Einstellungen → System → Home Assistant Cloud → Lokale Instanz-URL

**RS485-Steuermodus funktioniert nicht:**
Prüfe den exakten Options-Namen in deiner Marstek-Integration unter  
Entwicklertools → Zustände → nach `rs485` suchen.

**Regler schwingt (lädt/entlädt ständig abwechselnd):**
Totzone auf 60–80 W erhöhen, Verstärkung auf 0.9 reduzieren.

**Einspeisung trotz aktivem Blueprint:**
Ziel-Netzleistung auf 50 W setzen, Totzone auf 50 W.

---

## 📊 Changelog

| Version | Datum | Änderung |
|---|---|---|
| 1.0.0 | 2025-04 | Erstveröffentlichung |

---

## 📄 Lizenz

MIT License – Details siehe [LICENSE](LICENSE)

---

## ⚡ Entwickelt von

**[LichtValleyApps.de](https://lichtvalleyapps.de)**  
SmartHome-Integration & Eigenverbrauchsoptimierung  
Für Privathäuser, Höfe und Kleinunternehmen

> 💡 *LichtValleyApps führt keine Elektroinstallationsarbeiten durch –  
> ausschließlich SmartHome-Integration zur Eigenverbrauchsoptimierung.*

---

*Gefällt dir der Blueprint? ⭐ Star auf GitHub und teile ihn in der HA Community!*
