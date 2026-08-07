# ESPR Gewerbliche Geschirrspüler — Zusammenfassung

Stand: 21.07.2026

## Regulatorischer Status

Die EU bereitet zwei zusammengehörige Verordnungen für gewerbliche
Geschirrspüler vor (Rahmen: ESPR, Ecodesign for Sustainable Products
Regulation):

- **Ecodesign (ED)** — Mindestanforderungen für Marktzulassung
- **Energy Labelling (EL)** — Energieeffizienzlabel A–G

Beide liegen aktuell nur als **Entwurf** vor (Stand des Online-Stakeholder-Meetings
vom 23. Juni 2026). Zeitplan laut EU-Kommission / VITO-Vorstudie:

- Feedback-Frist zu den aktuellen Entwürfen: **13. September 2026**
- Öffentliche Konsultation (OPC): Herbst 2026
- Ecodesign-Forum-Treffen: Oktober 2026
- Abschluss der Vorstudie: Dezember 2026

Es handelt sich also um ein laufendes, nicht abgeschlossenes
Gesetzgebungsverfahren — Werte und Fristen im Entwurf können sich noch ändern.

Quelle: [ESPR Preparatory Study – Professional Dishwashers](https://ecodesign-commdishwashers.eu/en)

## Geprüfte Dokumente (4 total)

1. **EL-Annexe** — Label-Design (Annex III), Effizienzklassen A–G (Annex II),
   Produktdatenblatt (Annex V), technische Dokumentation (Annex VI), Werbe-/
   Fernabsatzvorgaben (Annex VII/VIII), Messmethoden (Annex IV), Verifizierung
   (Annex IX)
2. **ED-Annexe** — Ecodesign-Mindestanforderungen (Annex II): Reinigungsleistung,
   Energie-/Wasserverbrauch, Ersatzteile/Reparatur, Stoffinformationen,
   Hygiene; Messmethoden (Annex III), Verifizierung (Annex IV), Benchmarks
   (Annex V)
3. **ED-Artikel** — Verordnungstext Ecodesign (Art. 1–9): Geltungsbereich,
   Konformitätsbewertung, Umgehungsverbot, Review, Inkrafttreten
4. **EL-Artikel** — Verordnungstext Energy Labelling (Art. 1–9): Pflichten von
   Lieferanten, Händlern und Hosting-Plattformen, Review, Inkrafttreten

## Nur zwei Gerätekategorien — keine weitere Differenzierung

Die Verordnung unterscheidet ausschliesslich:

- **under-counter one-tank dishwasher** (Untertisch-Einbecken)
- **hood-type dishwasher** (Haube — schliesst auch Durchschub-Doppelhauben-Typen mit ein)

Es gibt **keine** separate EU-Kategorie für Durchschub- vs. einfache
Haubenspülmaschinen (beide = "hood-type"), und **keine** grössen-/
korbmassabhängigen Klassen. Grösse wird stattdessen implizit über die
Pro-Teller-Normierung (kWh/Teller, l/Teller) berücksichtigt.

## Zentrale Formeln und Referenzwerte (Annex IV, EL-Annexe)

| Grösse | Bedeutung | Formel |
|---|---|---|
| SPEC | Standard Programme Energy Consumption (kWh/Teller) | SPEC = EC / pR |
| REC | Reference Energy Consumption (kWh/Teller) | fest definiert je Gerätetyp (siehe unten) |
| EEI | Energy Efficiency Index (%) | EEI = SPEC / REC · 100 |

**REC (Referenz-Energieverbrauch):**

| Gerätetyp | REC |
|---|---|
| Under-counter one-tank dishwasher | 0,0130 kWh/Teller |
| Hood-type dishwasher | 0,0167 kWh/Teller |

EEI wird anschliessend über Tabelle 1 (Untertisch) bzw. Tabelle 2 (Haube) auf
die Buchstabenklasse A–G abgebildet:

| Klasse | EEI-Bereich Untertisch (Tabelle 1) | EEI-Bereich Haube (Tabelle 2) |
|---|---|---|
| A | ≤ 100 | ≤ 100 |
| B | 100–110 | 100–106 |
| C | 110–120 | 106–112 |
| D | 120–130 | 112–118 |
| E | 130–140 | 118–124 |
| F | 140–150 | 124–130 |
| G | > 150 | > 130 |

## Ecodesign-Mindestanforderungen (Annex II, ED-Annexe)

| Gerätetyp | SPEC max. | SPWC max. |
|---|---|---|
| Under-counter | < 0,021 kWh/Teller | < 0,179 l/Teller |
| Hood-type | < 0,023 kWh/Teller | < 0,169 l/Teller |

Zusätzlich: Reinigungsleistung (xclean) > 90 %, Mindest-Hygieneanforderungen.
SPWC wird in `eu_energielabel.py` aktuell nur dokumentiert, nicht automatisch
geprüft (dafür müsste zusätzlich der Wasserverbrauch pro Gerät ausgewertet
werden).

Ecodesign-Konformität und Label-Klasse sind **unabhängig** voneinander — auch
ein Gerät, das die Mindestanforderung nicht erfüllt (und damit eigentlich
nicht in Verkehr gebracht werden dürfte), bekäme rechnerisch trotzdem eine
Klasse zugeordnet, wenn man die Formel einfach durchrechnet.

## Umgesetztes Werkzeug

`eu_energielabel.py` (im Repo `Marflixx/egs-eartheffect`, Doku in
`docs/konzept_eu_energielabel.md`):

- Liest read-only Geschirrspüler-Geräte aus der EcoGastro Datenbank (`device` ⨝
  `device_listing` ⨝ `measure_category`), erkennt Gerätetyp über den
  Kategorienamen ("haube" → hood_type, "untertisch" → under_counter)
- Prüft Ecodesign-Konformität (SPEC-Grenzwert)
- Berechnet EEI und Label-Klasse (A–G, beide Gerätetypen) mit den oben
  genannten REC-Werten als Default — überschreibbar (`--rec-under-counter`,
  `--rec-hood-type`), falls die EU die Werte in einer späteren Entwurfsfassung
  noch ändert
- Keine Schreiboperation, keine Schema-/Migrationsänderung

## Fazit

Die A–G-Label-Zuordnung ist berechenbar. Die Werte gelten für den
Entwurfsstand vom Stakeholder-Meeting am 23.06.2026 und können sich bis zum
Abschluss der Konsultation (Feedback-Frist 13.09.2026, OPC Herbst 2026) noch
ändern.

## Anhang: Auswertung der EcoGastro Datenbank (Stand 21.07.2026)

Read-only-Abfrage der Produktions-DB (`device` ⨝ `device_listing` ⨝
`measure_category`, Feld `energieverbrauch_pro_teller` = SPEC).
**95 eindeutige Geschirrspüler-Modelle** mit hinterlegtem SPEC-Wert
ausgewertet.

### Ecodesign-Mindestanforderung

- **87 Modelle erfüllen** die Mindestanforderung (62 Haube-, 25
  Untertisch-Modelle)
- **8 Modelle erfüllen sie nicht** (siehe Tabelle unten)

### Label-Klassenverteilung (alle 95 Modelle, mit REC-Werten oben)

| Klasse | Anzahl Modelle |
|---|---|
| A | 7 |
| B | 2 |
| C | 10 |
| D | 23 |
| E | 26 |
| F | 16 |
| G | 11 |

Auffällig: Nur 9 von 95 Modellen (A+B) lägen in den beiden besten Klassen —
die meisten Modelle verteilen sich auf C bis F, auch wenn sie die
Ecodesign-Mindestanforderung locker erfüllen. Das liegt daran, dass REC
(0,0130 / 0,0167 kWh/Teller) deutlich unter den meisten in der Datenbank
vorkommenden SPEC-Werten liegt — REC scheint eher ambitionierte
Referenz-/Zielwerte statt Marktdurchschnitte abzubilden. Vollständige Geräteliste mit Klasse siehe Tabelle unten.

### Modelle, die die Ecodesign-Mindestanforderung NICHT erfüllen

| ID | Hersteller | Modell | Kategorie | Typ | SPEC (kWh/Teller) | EEI | Klasse |
|---|---|---|---|---|---|---|---|
| 416 | Gehrig Group | GTW 3700 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,023 | 137,7 | G |
| 452 | Meiko | M-iClean HL | Haubenspülmaschine 1 Korb, 500x650mm | Haube | 0,027 | 161,7 | G |
| 454 | Meiko | M-iClean HL AirConcept | Haubenspülmaschine 1 Korb, 500x650mm | Haube | 0,026 | 155,7 | G |
| 453 | Meiko | M-iClean HL WWHR | Haubenspülmaschine 1 Korb, 500x650mm | Haube | 0,026 | 155,7 | G |
| 455 | Meiko | M-iClean HL AirConcept/WWHR | Haubenspülmaschine 1 Korb, 500x650mm | Haube | 0,024 | 143,7 | G |
| 590 | Meiko | M-iClean HXL WWHR | Haubenspülmaschine 2 Körbe | Haube | 0,024 | 143,7 | G |
| 383 | Meiko | M-iClean US ComfortAir | Untertischspülmaschinen, 400x400 mm Korb (Gläser) | Untertisch | 0,022 | 169,2 | G |
| 386 | Winterhalter | UC-S Energy | Untertischspülmaschinen, 400x400 mm Korb (Gläser) | Untertisch | 0,023 | 176,9 | G |

Alle 8 nicht-konformen Modelle landen erwartungsgemäss auch in Klasse G
(schlechteste Klasse) — das ist konsistent, da sie den Ecodesign-Grenzwert
überschreiten und REC deutlich darunter liegt.

Auffällig zur Kategorie: 7 der 8 nicht-konformen Modelle stammen aus den
Kategorien „1 Korb, 500x650mm" (4x), „2 Körbe" (1x) oder „400x400 mm Korb
(Gläser)" (2x) — also den grösseren bzw. auf Gläser spezialisierten
Varianten. Die achte Ausnahme (ID 416, Gehrig Group GTW 3700) ist dagegen
eine reguläre „1 Korb, 500x500mm"-Kategorie, in der die meisten konformen
Modelle liegen; mit SPEC = 0,023 kWh/Teller liegt sie genau auf dem
Ecodesign-Grenzwert und ist damit primär ein Grenzfall, kein
Grössen-/Kategorieeffekt.

### Vollständige Geräteliste mit EU-Label-Klasse (alle 95 Modelle)

Sortiert nach Typ, Klasse, Hersteller, Modell.

| ID | Hersteller | Modell | Kategorie | Typ | SPEC (kWh/Teller) | EEI | Klasse | Ecodesign-konform |
|---|---|---|---|---|---|---|---|---|
| 581 | Ackermann | H540 Klima Plus | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,015 | 89,8 | A | ja |
| 410 | Ackermann | H540E Klima Plus | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,015 | 89,8 | A | ja |
| 583 | Bartscher | DS1002 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,014 | 83,8 | A | ja |
| 582 | Bartscher | DS1002-NRG | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,012 | 71,9 | A | ja |
| 584 | Colged | Toptech38-23D NRG | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,014 | 83,8 | A | ja |
| 443 | Krupps | CH150 | Haubenspülmaschine 1 Korb, 500x600mm | Haube | 0,016 | 95,8 | A | ja |
| 408 | Ackermann | H640 Klima Plus | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,017 | 101,8 | B | ja |
| 430 | Krupps | CH100 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,017 | 101,8 | B | ja |
| 422 | Hobart | AMX-10C + SEF-25/048 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,018 | 107,8 | C | ja |
| 424 | Hobart | AMX-10C + SEF-25/048 + SEF-25/045 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,018 | 107,8 | C | ja |
| 719 | Hobart | AMXS-10C + SEF-25/049 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,018 | 107,8 | C | ja |
| 718 | Hobart | AMXS-10C + SEF-25/049 + SEF-25/045 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,018 | 107,8 | C | ja |
| 458 | Hobart | AMXT-10C + SEF-25/054 | Haubenspülmaschine 2 Körbe | Haube | 0,018 | 107,8 | C | ja |
| 460 | Hobart | AMXT-10C + SEF-25/054 + SEF-25/047 | Haubenspülmaschine 2 Körbe | Haube | 0,018 | 107,8 | C | ja |
| 461 | Hobart | AUPT-10C + SEF-25/047 + SEF-25/054 | Haubenspülmaschine 2 Körbe | Haube | 0,018 | 107,8 | C | ja |
| 431 | Krupps | CH110 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,018 | 107,8 | C | ja |
| 407 | Ackermann | H540 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,019 | 113,8 | D | ja |
| 423 | Hobart | AMX-10C | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,019 | 113,8 | D | ja |
| 421 | Hobart | AMX-10C + SEF-25/045 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,019 | 113,8 | D | ja |
| 721 | Hobart | AMXS-10C | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,019 | 113,8 | D | ja |
| 425 | Hobart | AMXX-10C + SEF-25/050 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,019 | 113,8 | D | ja |
| 427 | Hobart | AMXX-10C + SEF-25/050 + SEF-25/045 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,019 | 113,8 | D | ja |
| 723 | Hobart | AMXXS-10C + SEF-25/051 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,019 | 113,8 | D | ja |
| 426 | Hobart | AUP-10C + SEF-25/050 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,019 | 113,8 | D | ja |
| 428 | Hobart | AUP-10C + SEF-25/050 + SEF-25/045 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,019 | 113,8 | D | ja |
| 434 | Meiko | UPster H 500 AirConcept | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,019 | 113,8 | D | ja |
| 406 | Ackermann | H640 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,020 | 119,8 | E | ja |
| 459 | Hobart | AMXT-10C | Haubenspülmaschine 2 Körbe | Haube | 0,020 | 119,8 | E | ja |
| 722 | Hobart | AMXT-10C + SEF-25/044 | Haubenspülmaschine 2 Körbe | Haube | 0,020 | 119,8 | E | ja |
| 457 | Hobart | AMXT-10C + SEF-25/047 | Haubenspülmaschine 2 Körbe | Haube | 0,020 | 119,8 | E | ja |
| 446 | Hobart | AMXXL-10C + SEF-25/052 | Haubenspülmaschine 1 Korb, 500x600mm | Haube | 0,020 | 119,8 | E | ja |
| 447 | Hobart | AMXXL-10C + SEF-25/052 + SEF-25/046 | Haubenspülmaschine 1 Korb, 500x600mm | Haube | 0,020 | 119,8 | E | ja |
| 725 | Hobart | AUPL-10C + SEF-25/043 + SEF-25/046 + SEF-25/052 | Haubenspülmaschine 1 Korb, 500x600mm | Haube | 0,020 | 119,8 | E | ja |
| 449 | Hobart | AUPL-10C + SEF-25/046 + SEF-25/052 | Haubenspülmaschine 1 Korb, 500x600mm | Haube | 0,020 | 119,8 | E | ja |
| 451 | Hobart | AUPL-10C + SEF-25/052 | Haubenspülmaschine 1 Korb, 500x600mm | Haube | 0,020 | 119,8 | E | ja |
| 759 | Hobart | AUPT-10C | Haubenspülmaschine 2 Körbe | Haube | 0,020 | 119,8 | E | ja |
| 462 | Hobart | AUPT-10C + SEF-25/044 | Haubenspülmaschine 2 Körbe | Haube | 0,020 | 119,8 | E | ja |
| 760 | Hobart | AUPT-10C + SEF-25/047 | Haubenspülmaschine 2 Körbe | Haube | 0,020 | 119,8 | E | ja |
| 726 | Hobart | AUPT-10C + SEF-25/047 + SEF-25/044 | Haubenspülmaschine 2 Körbe | Haube | 0,020 | 119,8 | E | ja |
| 463 | Hobart | AUPT-10C + SEF-25/054 | Haubenspülmaschine 2 Körbe | Haube | 0,020 | 119,8 | E | ja |
| 436 | Meiko | M-iClean HM AirConcept | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,020 | 119,8 | E | ja |
| 752 | Winterhalter | PT-L | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,020 | 119,8 | E | ja |
| 441 | Winterhalter | PT-M | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,020 | 119,8 | E | ja |
| 750 | Winterhalter | PT-XL | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,020 | 119,8 | E | ja |
| 445 | Hobart | AMXXL-10C | Haubenspülmaschine 1 Korb, 500x600mm | Haube | 0,021 | 125,7 | F | ja |
| 444 | Hobart | AMXXL-10C + SEF-25/046 | Haubenspülmaschine 1 Korb, 500x600mm | Haube | 0,021 | 125,7 | F | ja |
| 724 | Hobart | AMXXLS-10C | Haubenspülmaschine 1 Korb, 500x600mm | Haube | 0,021 | 125,7 | F | ja |
| 761 | Hobart | AUPL-10C | Haubenspülmaschine 1 Korb, 500x600mm | Haube | 0,021 | 125,7 | F | ja |
| 450 | Hobart | AUPL-10C + SEF-25/046 | Haubenspülmaschine 1 Korb, 500x600mm | Haube | 0,021 | 125,7 | F | ja |
| 448 | Hobart | AUPLS-10C | Haubenspülmaschine 1 Korb, 500x600mm | Haube | 0,021 | 125,7 | F | ja |
| 417 | Hobart | Eco-H604-12B | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,021 | 125,7 | F | ja |
| 429 | Krupps | CH110-BT | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,021 | 125,7 | F | ja |
| 432 | Krupps | EL60E | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,021 | 125,7 | F | ja |
| 433 | Krupps | EL60TH | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,021 | 125,7 | F | ja |
| 437 | Meiko | M-iClean HM AirConcept/WWHR | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,021 | 125,7 | F | ja |
| 435 | Meiko | M-iClean HM WWHR | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,021 | 125,7 | F | ja |
| 467 | Meiko | M-iClean HXL AirConcept/WWHR | Haubenspülmaschine 2 Körbe | Haube | 0,021 | 125,7 | F | ja |
| 753 | Winterhalter | PT-L EnergyPlus | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,021 | 125,7 | F | ja |
| 440 | Winterhalter | PT-M EnergyPlus | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,021 | 125,7 | F | ja |
| 754 | Winterhalter | PT-XL EnergyPlus | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,021 | 125,7 | F | ja |
| 416 | Gehrig Group | GTW 3700 | Haubenspülmaschine 1 Korb, 500x500mm | Haube | 0,023 | 137,7 | G | nein |
| 465 | Gehrig Group | GTW 5700 | Haubenspülmaschine 2 Körbe | Haube | 0,022 | 131,7 | G | ja |
| 452 | Meiko | M-iClean HL | Haubenspülmaschine 1 Korb, 500x650mm | Haube | 0,027 | 161,7 | G | nein |
| 454 | Meiko | M-iClean HL AirConcept | Haubenspülmaschine 1 Korb, 500x650mm | Haube | 0,026 | 155,7 | G | nein |
| 455 | Meiko | M-iClean HL AirConcept/WWHR | Haubenspülmaschine 1 Korb, 500x650mm | Haube | 0,024 | 143,7 | G | nein |
| 453 | Meiko | M-iClean HL WWHR | Haubenspülmaschine 1 Korb, 500x650mm | Haube | 0,026 | 155,7 | G | nein |
| 466 | Meiko | M-iClean HXL AirConcept | Haubenspülmaschine 2 Körbe | Haube | 0,022 | 131,7 | G | ja |
| 590 | Meiko | M-iClean HXL WWHR | Haubenspülmaschine 2 Körbe | Haube | 0,024 | 143,7 | G | nein |
| 388 | Bartscher | TF-517 | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,013 | 100,0 | A | ja |
| 580 | Ackermann | U 540 | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,015 | 115,4 | C | ja |
| 397 | Meiko | M-iClean UM+ ComfortAir | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,015 | 115,4 | C | ja |
| 568 | Bartscher | TopClean 44 | Untertischspülmaschinen, 400x400 mm Korb (Gläser) | Untertisch | 0,016 | 123,1 | D | ja |
| 732 | Hobart | CareS-10C | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,016 | 123,1 | D | ja |
| 391 | Hobart | FP-10C | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,016 | 123,1 | D | ja |
| 731 | Hobart | FPLS-10C | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,016 | 123,1 | D | ja |
| 733 | Hobart | FPS-10C | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,016 | 123,1 | D | ja |
| 572 | Hobart | FX-10C | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,016 | 123,1 | D | ja |
| 735 | Hobart | FXL-10C | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,016 | 123,1 | D | ja |
| 734 | Hobart | FXLS-10C | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,016 | 123,1 | D | ja |
| 736 | Hobart | FXS-10C | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,016 | 123,1 | D | ja |
| 396 | Meiko | M-iClean UM ComfortAir | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,016 | 123,1 | D | ja |
| 755 | Winterhalter | UC-L | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,016 | 123,1 | D | ja |
| 401 | Winterhalter | UC-M | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,016 | 123,1 | D | ja |
| 751 | Winterhalter | UC-XL | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,016 | 123,1 | D | ja |
| 728 | Hobart | Eco+F515-11C | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,017 | 130,8 | E | ja |
| 574 | Hobart | Eco+F515-11D | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,017 | 130,8 | E | ja |
| 727 | Hobart | Eco+F515S-11C | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,017 | 130,8 | E | ja |
| 729 | Hobart | Eco+F515S-11D | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,017 | 130,8 | E | ja |
| 405 | Meiko | M-iClean UL ComfortAir | Untertischspülmaschinen, 500x600 mm Korb | Untertisch | 0,018 | 138,5 | E | ja |
| 757 | Winterhalter | UC-L Energy | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,017 | 130,8 | E | ja |
| 756 | Winterhalter | UC-M Energy | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,017 | 130,8 | E | ja |
| 402 | Winterhalter | UC-XL Energy | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,017 | 130,8 | E | ja |
| 390 | Gehrig Group | GTW 3300 | Untertischspülmaschinen, 500x500 mm Korb | Untertisch | 0,020 | 153,8 | G | ja |
| 383 | Meiko | M-iClean US ComfortAir | Untertischspülmaschinen, 400x400 mm Korb (Gläser) | Untertisch | 0,022 | 169,2 | G | nein |
| 386 | Winterhalter | UC-S Energy | Untertischspülmaschinen, 400x400 mm Korb (Gläser) | Untertisch | 0,023 | 176,9 | G | nein |
