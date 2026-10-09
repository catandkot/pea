<div align="center">

[한국어](README.md) · [简体中文](README_zh.md) · [繁體中文](README_zh-TW.md) · [日本語](README_ja.md) · [Türkçe](README_tr.md) · [Русский](README_ru.md) · [Tiếng Việt](README_vi.md) · [ไทย](README_th.md) · [Deutsch](README_de.md) · [Español](README_es.md) · [Français](README_fr.md) · [Українська](README_uk.md) · **Polski** · [Português (Brasil)](README_pt-BR.md) · [العربية](README_ar.md) · [فارسی](README_fa.md)

# 🫛 PEA

### Agent inteligentnego gospodarstwa — odczytuje dane z pola uprawnego, ocenia je i działa.

**Czujniki już mówią. PEA sprawia, że ich głos staje się zrozumiały.**

[![license](https://img.shields.io/badge/license-MIT-f4c430?style=flat-square)](LICENSE)
[![status](https://img.shields.io/badge/status-concept-5c4ee5?style=flat-square)](#status-projektu)

[Szybki start](#szybki-start) · [Co robi PEA](#co-robi-pea) · [Jak to działa](#jak-to-działa) · [Zasady bezpieczeństwa](#zasady-bezpieczeństwa) · [Plan rozwoju](#plan-rozwoju)

</div>

---

PEA (groch) to **agent inteligentnego gospodarstwa** dla szklarni i upraw polowych. Zbiera odczyty czujników, takie jak wilgotność gleby, temperatura i wilgotność powietrza, nasłonecznienie, CO2 oraz EC/pH w roztworze odpływowym, wyjaśnia *co dzieje się teraz*, a także steruje elementami wykonawczymi — nawadnianiem, wentylacją, cieniowaniem i doświetlaniem — **w bezpiecznych granicach**.

Każda decyzja ma uzasadnienie. PEA rozróżnia w raportach **wartości zaobserwowane**, **wartości wywnioskowane** i **wartości jeszcze nieznane**. „Gleba jest sucha" to historia odczytów z czujnika, „po południu ryzyko więdnięcia jest wysokie" to wniosek, a „czy zawór nawadniający rzeczywiście się otworzył" pozostaje nieznane, dopóki nie potwierdzi tego przepływomierz.

Praca odbywa się na terenie gospodarstwa. Na zewnątrz trafiają wyłącznie dane, które użytkownik wyraźnie wybrał.

## Szybki start

> **Poniższe polecenia i nazwy pakietów nie są jeszcze ostateczne.** Pakiet `pea` nie jest obecnie opublikowany w npm. W momencie publikacji zaktualizujemy ten dokument o rzeczywiste polecenia.

```bash
# 1) install (package not published yet)
npx pea-agents setup

# 2) readiness check — gateway, sensors and actuators
pea doctor

# 3) ask about today's greenhouse
pea analyze ./farm.toml
```

Po podłączeniu agenta wystarczy zapytać w ten sposób.

```text
W szklarni nr 3 poranne nawadnianie było o 12% większe od celu. Jaka jest przyczyna?
Jak zmienić dzisiejszy harmonogram nawadniania po południu, aby współczynnik odpływu wyniósł 25%?
Pokaż, ile godzin utrzymywała się wilgoć na liściach przy najniższej temperaturze z ostatniej nocy.
```

## Co robi PEA

| Obszar | Co robi PEA | Wykorzystywane dane |
| --- | --- | --- |
| Warunki wzrostu | Przebieg i wskazanie odcinków odbiegających od normy dla temperatury i wilgotności powietrza w szklarni, VPD oraz sumy nasłonecznienia | Czujniki temperatury i wilgotności, pyranometr |
| Nawadnianie i pożywka | Obliczanie dawki nawadniania na podstawie sumy nasłonecznienia, alarm przy odchyleniu od docelowego współczynnika odpływu | Czujniki wilgotności i masy gleby, EC/pH odpływu |
| Wentylacja i klimatyzacja | Sterowanie utrzymujące docelowe VPD, propozycje stopni otwarcia okien i wentylatorów | Informacja zwrotna o otwarciu okien, pogoda zewnętrzna |
| Alarmy chorób i szkodników | Ocena ryzyka chorób grzybowych na podstawie czasu utrzymywania się wilgoci na liściach oraz kombinacji temperatury i wilgotności | Historia temperatury i wilgotności, czujniki wilgoci na liściach |
| Dziennik uprawy | Automatyczne zapisywanie historii nawadniania, warunków i prac, pomoc w wprowadzaniu wyników pomiarów wzrostu | Rejestry prac, zdjęcia z pola |
| Prognoza zbiorów | Szacowanie terminu zbioru na podstawie sumy temperatur, porównywanie sezonów uprawy | Historia temperatury, parametry odmiany |

Wartości i progi w tabeli to **wartości wymagające ponownej kalibracji dla każdej uprawy, odmiany i regionu**. PEA jedynie proponuje wartości domyślne i nie uznaje ich za obowiązujące normy danego gospodarstwa.

## Jak to działa

Gdy agent zada pytanie do PEA przez MCP, PEA zbiera dane z bramki i zwraca wynik wraz z uzasadnieniem. Na tej podstawie agent opisuje sytuację lub proponuje sposób sterowania. Ten sam tok pracy jest dostępny bez zmian w CLI w terminalu.

```text
Agent → MCP → PEA → bramka → czujniki
               ← odczyty + wnioski + niewiadome
```

- **Najpierw lokalnie** — zbieranie danych i ocena odbywają się na terenie gospodarstwa.
- **Najpierw uzasadnienie** — wraz z wnioskiem zapisywane są wartości źródłowe, czas i pochodzenie danych.
- **Nieznane pozostaje nieznane** — odcinki bez czujnika lub z przerwanym połączeniem nie są wypełniane szacunkami, lecz wskazywane wprost.
- **Identyczne wyniki** — CLI i MCP korzystają z tego samego toku pracy i tego samego kontraktu wyników.

## Obsługiwane systemy

| Integracja | Sposób | Status |
| --- | --- | --- |
| Brama MQTT | Zbieranie tematów czujników przez subskrypcję brokera | planowane |
| Modbus RTU / TCP | Odpytywanie rejestrów sterownika | planowane |
| LoRaWAN | Czujniki polowe przez bramkę | planowane |
| Dane meteorologiczne | Regionalne API pogodowe, temperatura, wilgotność i opady na zewnątrz | planowane |
| Istniejące systemy zarządzania uprawą | Scalanie historii przez import plików (CSV/JSON) | planowane |

## Zasady bezpieczeństwa

Wprawienie w ruch elementów wykonawczych jest nieodwracalne. Dlatego PEA przestrzega następujących zasad.

1. **Domyślna zgoda człowieka** — rzeczywiste działania, takie jak nawadnianie, wentylacja czy doświetlanie, najpierw przedstawiają plan i wymagają zatwierdzenia.
2. **Twarde ograniczenia** — ustalonych dla gospodarstwa wartości granicznych (maksymalna jednorazowa dawka nawadniania, dopuszczalny zakres temperatur itp.) nie może przekroczyć decyzja agenta.
3. **Fail-safe** — przy utracie połączenia lub zaniku zasilania system przechodzi w stan bezpieczny (zatrzymanie nawadniania, otwarcie wentylacji itp.).
4. **Brak automatyzacji oprysków** — PEA nie wykonuje oprysków środkami ochrony roślin ani nawozami; proponuje jedynie termin i uzasadnienie.
5. **Pełna rejestracja** — każde sterowanie pozostawia zapis celu, czasu, uzasadnienia i wyniku wykonania.

## Plan rozwoju

- **M0 — koncepcja i dokumentacja** *(obecnie)*: definicja produktu, zasady bezpieczeństwa, projekt kontraktu wyników
- **M1 — zbieranie danych**: integracja z bramką, zapisywanie historii czujników, przeglądanie przebiegu warunków
- **M2 — doradztwo**: doradca nawadniania i wentylacji, ryzyko chorób i szkodników, automatyzacja dziennika uprawy
- **M3 — sterowanie za zgodą**: automatyczne sterowanie za zgodą człowieka i fail-safe
- **M4 — eksploatacja**: prognoza zbiorów, porównywanie wielu gospodarstw, raporty sezonu uprawy

## Status projektu

Ten dokument opisuje **koncepcję produktu PEA**. Kod znajdujący się obecnie w repozytorium to narzędzie do inżynierii wstecznej (`rea-agents`), które nie implementuje jeszcze opisanych powyżej funkcji zbierania danych i sterowania. Nazwy poleceń, lista integracji i progi zostaną ustalone wraz z implementacją. Przetłumaczone pliki README opierają się jeszcze na wcześniejszym dokumencie i będą sukcesywnie dostosowywane do nowej marki.

## Współtworzenie

Zgłoszenia i pull requesty są mile widziane. Szczególnie cenne są obserwacje i przypadki niepowodzeń zebrane w praktyce oraz wartości kalibracyjne dla poszczególnych upraw.

## Licencja

[MIT](LICENSE)
