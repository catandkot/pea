<div align="center">

[한국어](README.md) · [简体中文](README_zh.md) · [繁體中文](README_zh-TW.md) · [日本語](README_ja.md) · [Türkçe](README_tr.md) · [Русский](README_ru.md) · [Tiếng Việt](README_vi.md) · [ไทย](README_th.md) · **Deutsch** · [Español](README_es.md) · [Français](README_fr.md) · [Українська](README_uk.md) · [Polski](README_pl.md) · [Português (Brasil)](README_pt-BR.md) · [العربية](README_ar.md) · [فارسی](README_fa.md)

# 🫛 PEA

### Smart-Farm-Agent — liest die Daten am Anbauort, urteilt und handelt.

**Die Sensoren sprechen längst. PEA sorgt dafür, dass man sie versteht.**

[![license](https://img.shields.io/badge/license-MIT-f4c430?style=flat-square)](LICENSE)
[![status](https://img.shields.io/badge/status-concept-5c4ee5?style=flat-square)](#projektstatus)

[Schnellstart](#schnellstart) · [Was PEA tut](#was-pea-tut) · [Arbeitsweise](#arbeitsweise) · [Sicherheitsprinzipien](#sicherheitsprinzipien) · [Roadmap](#roadmap)

</div>

---

PEA (die Erbse) ist ein **Smart-Farm-Agent** für Gewächshäuser und Freilandanbau. Er sammelt Sensorwerte wie Bodenfeuchte, Temperatur und Luftfeuchtigkeit, Globalstrahlung, CO2 und Drain-EC/pH, erklärt *was gerade passiert* und steuert Aktoren wie Bewässerung, Lüftung, Schattierung und Zusatzbelichtung — **innerhalb sicherer Grenzen**.

Jede Entscheidung wird begründet. PEA unterscheidet in seinen Berichten zwischen **beobachteten Werten**, **abgeleiteten Werten** und **noch unbekannten Werten**. „Der Boden ist trocken" ist eine Sensorhistorie, „am Nachmittag ist die Gefahr von Fäulnis hoch" eine Schlussfolgerung, und „ob das Bewässerungsventil tatsächlich geöffnet wurde" bleibt unbekannt, bis ein Durchflussmesser es bestätigt.

Die Arbeit bleibt auf dem Betrieb. Nach außen gehen nur die Daten, die der Nutzer ausdrücklich ausgewählt hat.

## Schnellstart

> **Die folgenden Befehle und Paketnamen sind noch nicht final.** Derzeit gibt es kein `pea`-Paket in npm. Zum Veröffentlichungszeitpunkt wird dieses Dokument auf die tatsächlichen Befehle aktualisiert.

```bash
# 1) install (package not published yet)
npx pea-agents setup

# 2) readiness check — gateway, sensors and actuators
pea doctor

# 3) ask about today's greenhouse
pea analyze ./farm.toml
```

Nach der Verbindung mit dem Agenten genügt es, so zu fragen:

```text
Im Gewächshaus 3 lag die Bewässerungsmenge am Vormittag 12 % über dem Ziel. Woran liegt das?
Wie muss ich den Bewässerungsplan für heute Nachmittag ändern, um eine Drain-Rate von 25 % zu erreichen?
Zeig mir, wie viele Stunden die Blattbenetzung auf Basis der Tiefsttemperatur der letzten Nacht gedauert hat.
```

## Was PEA tut

| Bereich | Was PEA tut | Verwendete Daten |
| --- | --- | --- |
| Wachstumsumgebung | Verlauf und Anomaliebereiche von Temperatur, Luftfeuchtigkeit, VPD und Globalstrahlungssumme im Gewächshaus | Temperatur- und Feuchtesensoren, Pyranometer |
| Bewässerung und Nährlösung | Berechnung der Bewässerungsmenge aus der Globalstrahlungssumme, Alarm bei Abweichung vom Drain-Raten-Ziel | Bodenfeuchte- und Wägesensoren, Drain-EC/pH |
| Lüftung und Klimatisierung | Regelung zur Einhaltung des VPD-Ziels, Vorschläge für Lüftungsklappen- und Ventilatorstufen | Rückmeldung der Klappenöffnung, Außenwetter |
| Schädlings- und Krankheitsalarm | Risikobewertung für Pilzkrankheiten aus Blattbenetzungsdauer und Temperatur-Feuchte-Kombination | Temperatur- und Feuchteverlauf, Blattbenetzungssensoren |
| Anbautagebuch | Automatische Erfassung von Bewässerungs-, Klima- und Arbeitsverlauf, Unterstützung bei der Eingabe von Wachstumserhebungen | Arbeitsprotokolle, Feldaufnahmen |
| Ernteprognose | Schätzung des Erntezeitpunkts anhand der Temperatursumme, Vergleich von Anbauzyklen | Temperaturverlauf, Sortenparameter |

Die Werte und Schwellen in der Tabelle sind **Werte, die für jede Kultur, Sorte und Region neu kalibriert werden müssen**. PEA schlägt nur Standardwerte vor und legt sie nicht als verbindliche Referenzwerte des Betriebs fest.

## Arbeitsweise

Fragt ein Agent PEA über MCP, sammelt PEA am Gateway die Daten und liefert das Ergebnis samt Begründung zurück. Der Agent erklärt damit die Lage oder schlägt eine Steuerung vor. Derselbe Workflow lässt sich unverändert über die CLI im Terminal nutzen.

```text
Agent → MCP → PEA → Gateway → Sensoren
               ← Messwerte + Schlussfolgerungen + Unbekanntes
```

- **Lokal zuerst** — Erfassung und Bewertung werden auf dem Betrieb verarbeitet.
- **Belege zuerst** — Zu jedem Ergebnis werden Rohwerte, Zeitpunkt und Quelle festgehalten.
- **Unbekannt bleibt unbekannt** — Abschnitte ohne Sensor oder mit unterbrochener Verbindung werden nicht durch Schätzungen überbrückt, sondern ausdrücklich benannt.
- **Gleiche Ergebnisse** — CLI und MCP teilen denselben Workflow und denselben Ergebnisvertrag.

## Unterstützte Systeme

| Anbindung | Verfahren | Status |
| --- | --- | --- |
| MQTT-Gateway | Erfassung der Sensor-Topics durch Broker-Abonnement | geplant |
| Modbus RTU / TCP | Abfrage der Regler-Register | geplant |
| LoRaWAN | Freilandsensoren über Gateway | geplant |
| Wetterdaten | Regionale Wetter-API, Außentemperatur, Luftfeuchtigkeit und Niederschlag | geplant |
| Bestehende Anbausysteme | Zusammenführung der Historie über Dateiimport (CSV/JSON) | geplant |

## Sicherheitsprinzipien

Aktoren zu bewegen ist nicht rückgängig zu machen. Deshalb hält sich PEA an Folgendes.

1. **Menschliche Freigabe als Standard** — Tatsächliche Aktionen wie Bewässerung, Lüftung oder Zusatzbelichtung geben zuerst einen Plan aus und holen eine Freigabe ein.
2. **Harte Grenzen** — Die für den Betrieb festgelegten Obergrenzen (maximale Wassermenge pro Vorgang, zulässiger Temperaturbereich usw.) können durch Agentenentscheidungen nicht überschritten werden.
3. **Fail-Safe** — Bei Verbindungsabbruch oder Stromausfall fällt das System in den sicheren Zustand (Bewässerung gestoppt, Lüftung geöffnet usw.).
4. **Keine Automatisierung von Ausbringungen** — Pflanzenschutz- und Düngemittel werden von PEA nicht ausgebracht; PEA schlägt nur Zeitpunkt und Begründung vor.
5. **Vollständige Protokollierung** — Zu jeder Steuerung werden Ziel, Zeitpunkt, Begründung und Ausführungsergebnis festgehalten.

## Roadmap

- **M0 — Konzept und Dokumentation** *(aktuell)*: Produktdefinition, Sicherheitsprinzipien, Entwurf des Ergebnisvertrags
- **M1 — Erfassung**: Gateway-Anbindung, Speicherung der Sensorhistorie, Abfrage der Klimaverläufe
- **M2 — Beratung**: Berater für Bewässerung und Lüftung, Risiko für Schädlinge und Krankheiten, Automatisierung des Anbautagebuchs
- **M3 — Freigabebasierte Steuerung**: Automatische Steuerung mit menschlicher Freigabe und Fail-Safe
- **M4 — Betrieb**: Ernteprognose, Vergleich mehrerer Betriebe, Anbauzyklen-Berichte

## Projektstatus

Dieses Dokument beschreibt das **Produktkonzept von PEA**. Der Code im aktuellen Checkout ist das Reverse-Engineering-Werkzeug (`rea-agents`) und implementiert die oben beschriebenen Funktionen zur Erfassung und Steuerung noch nicht. Befehlsnamen, Liste der Anbindungen und Schwellenwerte werden mit der Implementierung festgelegt. Die übersetzten READMEs richten sich noch nach dem früheren Dokument und werden im Zuge des Rebrandings nach und nach angepasst.

## Mitwirken

Issues und Pull Requests sind willkommen. Besonders hilfreich sind Beobachtungswerte und Fehlerfälle aus der Praxis sowie Kalibrierungswerte je Kultur.

## Lizenz

[MIT](LICENSE)
