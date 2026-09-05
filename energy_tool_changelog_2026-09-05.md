# Changelog: Testbericht-Umsetzung EnergyCalc — 05.09.2026

Saubere, chronologisch nachvollziehbare Zusammenfassung aller Änderungen.
Ausgangspunkt war die Auswertung des Testberichts
`260904_Testbericht_Tool_EnergyCalc.docx`; im Verlauf kamen einige zusätzlich
gewünschte Funktionen dazu. Für die vollständige Nachverfolgung jedes
einzelnen der ~30 ursprünglichen Testbericht-Punkte (inkl. Diskussion,
Tests und Alternativen) siehe `docs/konzept_testbericht_250904.md` — dieses
Dokument hier ist die kompakte Übersicht "was wurde geändert und warum".

Alle Punkte wurden lokal End-to-End getestet, bevor sie committet und über
die neue CI-Pipeline automatisch deployt wurden.

---

## 1. Infrastruktur

- **Automatisches Deployment** eingerichtet: Push auf `main` deployt
  automatisch auf den Server (`.github/workflows/deploy.yml`, GitHub
  Actions, SSH-basiert, mit Health-Check nach dem Neustart).

## 2. Sicherheit

- **Initial-Admin-Passwort:** Zufällig generiert statt fest verdrahtetem
  `admin/admin`, wird einmalig beim ersten Start geloggt.
- **Session-Cookie:** `secure`-Flag ergänzt (aktiv, sobald `BASE_URL` mit
  `https://` beginnt).

## 3. Grundsatzentscheide (Rollenmodell)

Sechs offene Grundsatzfragen aus dem Testbericht geklärt (Details:
Konzeptdokument Block A):
- Systemgrenze/Messvergleich: methodische Anmerkung, kein Feature.
- Projekte bleiben 1 Planer = 1 Zugriff.
- Firmenwechsel eines Planers: entspricht bereits dem Ist-Zustand.
- **Beleuchtung** als neues Gewerk aufgenommen und umgesetzt (siehe Abschnitt 5).
- Gastroplaner darf Lüftung/Kälte weiterhin berechnen, mit Pflicht-Disclaimer.
- Bei Zentralkühlung erfasst der Gastroplaner nur EK-Angaben, klar markiert
  (→ siehe ZK/EK-Kennzeichnung unten).

## 4. Bugfixes

| Bug | Ursache | Fix |
|---|---|---|
| Kühlraum mit 0 m³ ergab 156 kWh/a statt 0 | Python `or`-Falle: `0 or 1.0` wird zu `1.0` | `is not None`-Check |
| "None" als sichtbarer Text bei fehlender Referenzgrösse | Jinja-Bedingung prüfte nur ob Kategorie existiert, nicht ob der Wert gesetzt ist | Bedingung ergänzt |
| Projektphase liess sich nie speichern, immer "Konzept" | Kein Formularfeld + Query-Parameter-Default überschrieb den DB-Wert immer | Feld ergänzt, Default-Logik korrigiert |
| Grösseneinheit optional trotz gesetzter Referenzgrösse | Keine Validierung | Server-seitige Pflichtprüfung ergänzt |
| Neue Kühlraum-Standardgeräte defaulteten auf "Hoch/Effizient" | Falscher Default im Admin-Formular | Auf "Mittel/Standard" korrigiert |

**Drei Regressionen, jeweils noch am selben Tag behoben:**
- `UnboundLocalError` beim Gerät-Hinzufügen für Kühlräume/Lüftung (Nebenwirkung
  des EcoGastro-Alternative-Hinweises, Variable nur bedingt initialisiert).
- Katalog-Gruppen-Filter zeigte nichts an, weil der Kategoriename im
  Formular-**Feldnamen** kodiert war statt im Wert — Sonderzeichen/Umlaute
  wurden dabei inkonsistent dekodiert.
- Derselbe Bugtyp (Variable nur bedingt initialisiert) trat beim Bau der
  Beleuchtungs-Funktion erneut auf und legte kurzzeitig alle 6 Gerätetabs lahm.

## 5. Neue Funktionen

**Beleuchtung als neues Gewerk**
- Neuer Tab **💡 Beleuchtung**, positioniert zwischen Kühlräume und Lüftung.
- Berechnung: `Fläche (m²) × spezifische Anschlussleistung (W/m²) ×
  Betriebsstunden/Tag × Betriebstage/Jahr` — nach **SIA 387/4** (aktuelle
  Norm, hat die ältere SIA 380/4 abgelöst) i.V.m. **SIA 2024**
  (Standard-Nutzungsbedingungen für Energie- und Gebäudetechnik).
- W/m² und Betriebsstunden/Tag sind bewusst **freie Eingabefelder** für den
  Planer, kein automatisch nachgeschlagener Normwert — die konkreten
  SIA-2024-Tabellenwerte für die Raumnutzung "Küche/Gastronomie" liegen
  hinter der kostenpflichtigen Norm und wurden hier nicht erfunden.
- Neue Admin-Einstellungsseite für Vorschlagswerte (aktuell leer), die beim
  Erfassen eines Beleuchtungs-Geräts als Default vorausgefüllt werden, aber
  pro Gerät überschreibbar bleiben.
- Noch kein Vergleichsgerät-/Einsparungsmodell für Beleuchtung definiert —
  vorerst reine Verbrauchsberechnung ohne ausgewiesene Einsparung.

**Geräte-Referenz & IDs**
- Stabile Geräte-Referenz `Projekt-ID_Objekt-ID_Geräte-Nr.`, sichtbar in der
  Objektansicht, bleibt beim Umsortieren stabil.
- **Projekt-ID** und **Objekt-ID** sind eigene, alphanumerische Felder
  (nicht mehr die reine Datenbank-Nummer), automatisch vergeben, im
  jeweiligen Bearbeiten-Formular frei anpassbar (z.B. "KUNDE-2026-A").
- In der Geräte-Bearbeiten-Ansicht ist **nur die Geräte-Nr.** (letzter Teil
  der Referenz) editierbar — Projekt- und Objekt-Teil sind fix und kommen
  immer aus Projekt/Objekt.

**Objekte & Projekte**
- Objekte lassen sich einem anderen (eigenen) Projekt zuordnen (verschieben,
  keine Kopie).
- Objekttyp "Kantine" → "Mensa" umbenannt; Objekt-Konzept generalisiert
  (neue Bereichstypen Gastraum/Lager/Aussenbereich, nicht mehr nur Küche).
- Lüftungsbemessung nur aktiv/anwählbar, wenn ein Küchentyp gesetzt ist.
- **Checkbox pro Objekt** "geht in den Gesamtreport des Projekts ein"
  (Default: ja) — betrifft Projekt-Report, Excel-Export, Gesamt-Geräteliste.

**Geräte**
- Geräte direkt duplizieren (⧉) und löschen (🗑) je Zeile in der Objektansicht.
- **Geräte, die nicht im Katalog sind**, lassen sich direkt im Objekt
  erfassen (inkl. Vorschlägen "Arbeitssteckdose 230V/400V" als
  Sammelposition für unbekannte Verbraucher).
- Betriebstage pro Gerät überschreibbar (z.B. Kühlgerät 365 Tage/Jahr),
  bleibt stabil bei späteren Änderungen der Objekt-Betriebstage.
- Freitext-Notiz je Geräteinstanz (Lieferant, Projektstand etc.).
- **ZK/EK-Kennzeichnung** bei Kühlgeräten (zentral/eigenständig gekühlt) mit
  automatischem Disclaimer bei zentraler Kühlung.
- 3-Wege-Auswahl für die Effizienzstufe des ersetzten Geräts (sehr
  alt/Standard/effizient) statt einer einfachen Checkbox.
- Referenzgerät sichtbar ("vs. Hersteller Modell") in der Objektansicht.
- Grüner Hinweispunkt bei Standardgeräten mit verfügbarer EcoGastro-Alternative.

**Reports & Listen**
- Sortierbare Spalten in Objekt-Geräteliste, Projekt-Geräteliste und
  Energiebericht.
- Gesamt-Geräteliste über alle Objekte eines Projekts als In-App-Ansicht
  (ergänzend zum bestehenden Excel-Export).
- Zurück-Link im Energiebericht (korrekt ausgeblendet bei externen
  Kunden-Share-Links).
- CHF/Jahr-Bedeutung geklärt und als Tooltip/Fussnote dokumentiert (reine
  Stromkosten-Einsparung, ohne Gerätemehrpreis).

**Gerätekatalog**
- Zusätzlicher Filter nach Unterkategorie (Wärmegeräte, Kaffeegeräte,
  Spülmaschinen, Zusatzgeräte u.a.) bei EcoGastro- und Standardgeräten.
- **Unterkategorien admin-erweiterbar:** neuer Bereich "Unterkategorien
  verwalten" im Admin-Tab "🗂️ Katalog-Gruppen" — neue Unterkategorien
  anlegen oder bestehende löschen.
- Kategorie-Dropdown schränkt sich automatisch auf die der gewählten
  Unterkategorie zugeordneten Kategorien ein (bei Wechsel wird die
  Kategorie-Auswahl zurückgesetzt, um widersprüchliche Filter zu vermeiden).

**Kursanleitung**
- Die Planer-Anleitung im Tool ("So funktioniert's") wurde vollständig um
  alle oben genannten neuen Funktionen ergänzt.

## 6. Korrekturen im Verlauf (auf Rückmeldung angepasst)

Zwei Punkte wurden zunächst anders gebaut, als gewünscht, und danach korrigiert:

- **Gerätevergleich:** Erster Versuch erlaubte die freie Wahl eines
  beliebigen Vergleichsgeräts über alle Kategorien hinweg. Auf Rückmeldung
  zurückgesetzt: Das Vergleichsgerät bleibt automatisch das Standardgerät
  derselben Kategorie. Stattdessen war die 3-Wege-Effizienzstufe gemeint
  (siehe oben).
- **Geräte-Referenz-Editierbarkeit:** Erster Versuch erlaubte, die ganze
  Referenz frei zu überschreiben. Auf Rückmeldung angepasst: Nur die
  Geräte-Nr. ist editierbar, Projekt-/Objekt-Teil sind fix.

## 7. Geklärt, ohne Codeänderung

- Auslastungs-Default war bereits korrekt bei 100 %.
- Objekte und Auftraggeber sind bereits editierbar.
- Wärmestrahler/-lampe/-brücke bleiben drei separate Kategorien, Werte
  aktuell bewusst identisch (Differenzierung folgt später).
- Zu viele Standardmodelle bei Spülmaschinen: aktueller Stand passt so.

## 8. Zurückgestellt

- Min/Max-Validierung der Gerätegrösse pro Kategorie (braucht Schema-
  Migration + Admin-UI).
- Prozessbasierter Gerätevergleich (Version 2, `usage_h*`-Felder).

## 9. Offen (braucht Martins Input)

- Fehlende Spülmaschinen-Werte im Katalog — kein Zugriff auf die Live-
  Produktionsdatenbank von hier aus möglich.
- Reale SIA-2024-Tabellenwerte (W/m², Betriebsstunden/Tag) für die
  Beleuchtungsberechnung, falls Zugriff auf die Norm besteht.

---

*Detaillierte Nachverfolgung mit allen Tests, Diskussionen und Alternativen:
`docs/konzept_testbericht_250904.md`. Schema-Änderungen: `docs/schema_raw.sql`
(bei Bedarf neu exportieren). Technische Gesamtdokumentation:
`docs/technische_dokumentation.md`.*
