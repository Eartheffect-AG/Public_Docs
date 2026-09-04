# EnergyCalc (energy_tool) — Umsetzung Testbericht, 04.09.2026

Zusammenfassung aller Änderungen aus der Auswertung des Testberichts
`260904_Testbericht_Tool_EnergyCalc.docx` (Feedback von Martin, erfasst
Okt./Nov. 2025). Vollständiges, laufend nachgeführtes Arbeitsdokument mit
allen ~30 Einzelpunkten: `docs/konzept_testbericht_250904.md` im
`energy_tool`-Repo. Dieses Dokument ist die kompakte, für aussenstehende
Leser gedachte Zusammenfassung.

## Infrastruktur

- **Automatisches Deployment** via GitHub Actions eingerichtet
  (`.github/workflows/deploy.yml`): Push auf `main` deployt automatisch auf
  den Server (SSH, `DEPLOY_HOST`/`DEPLOY_USER`/`DEPLOY_SSH_KEY` Secrets),
  inkl. Health-Check nach dem Neustart.

## Sicherheit

- **Initial-Admin-Passwort:** Zufällig generiert statt fest verdrahtetem
  `admin/admin`; wird einmalig beim ersten Start geloggt.
- **Session-Cookie:** `secure`-Flag ergänzt (automatisch aktiv, wenn
  `BASE_URL` mit `https://` beginnt).

## Rollenmodell / Grundsatzentscheide

- Verantwortlichkeiten zwischen Gastroplaner, Lüftungsplaner und
  Kälteplaner geklärt und im Tool abgebildet (siehe ZK/EK-Kennzeichnung
  unten).
- **Beleuchtung** als neues Gewerk beschlossen (Detailkonzept bei
  Umsetzung).
- Projekte bleiben bewusst 1 Planer = 1 Zugriff (kein Multi-User pro
  Projekt).

## Bugfixes

- **Kühlraum 0 m³** ergab fälschlich 156 kWh/a statt 0 (Python-`or`-Falle
  bei explizit eingegebener 0).
- **"None" als sichtbarer Text** in Grössenfeldern, wenn eine Kategorie
  ohne Referenzgrösse existierte.
- **Projektphase** (Konzept/Vorprojekt/Bauprojekt) liess sich bisher gar
  nicht speichern und wurde beim Anzeigen/Berechnen immer auf "Konzept"
  zurückgesetzt — jetzt persistent und korrekt.
- **Grösseneinheit ist jetzt Pflicht**, sobald eine Referenzgrösse gesetzt
  wird (Admin-Kategorien).
- **Zwei Regressionen während der Session selbst eingeführt und noch am
  selben Tag behoben:**
  - `UnboundLocalError` beim Gerät-Hinzufügen für Kühlräume/Lüftung
    (Nebenwirkung eines anderen Fixes).
  - Katalog-Gruppen-Filter zeigte nichts an, weil der Kategorienamen im
    Formular-Feldnamen kodiert war und Sonderzeichen/Umlaute dabei falsch
    dekodiert wurden.

## Neue Features

- **Stabile Geräte-Referenz** `ProjektID_ObjektID_Laufnummer`, frei
  überschreibbar, bleibt beim Umsortieren stabil.
- **Geräte duplizieren** (⧉-Button) und **direkt löschen** (🗑-Button) in
  der Objektansicht.
- **Objekte einem anderen (eigenen) Projekt zuordnen** (verschieben, keine
  Kopie) im Bearbeiten-Formular.
- **Manuell erfasste Geräte** (nicht im Katalog vorhanden, z.B.
  Arbeitssteckdosen 230V/400V als Sammelposition) direkt im Objekt
  erfassbar — kein Umweg über die Admin-Geräteverwaltung mehr nötig.
- **Betriebstage pro Gerät überschreibbar** (z.B. Kühlgerät 365 Tage/Jahr
  statt der Objekt-Standardtage), bleibt stabil bei späteren
  Objekt-Änderungen.
- **Notizfeld pro Geräteinstanz** (Lieferant, Projektstand etc.).
- **ZK/EK-Kennzeichnung** bei Kühlgeräten (zentral/eigenständig gekühlt)
  mit automatischem Hinweis auf approximative Berechnung bei zentraler
  Kühlung.
- **Sortierbare Spalten** in Objekt-Geräteliste, Projekt-Geräteliste und
  Energiebericht (Klick auf Spaltenkopf).
- **Gesamt-Geräteliste** über alle Objekte eines Projekts als In-App-Ansicht
  (ergänzend zum bestehenden Excel-Export).
- **3-Wege-Auswahl** für die Effizienzstufe des ersetzten Geräts (sehr
  alt/Standard/effizient) statt einer einfachen Ja/Nein-Checkbox.
- **Hinweis auf verfügbare EcoGastro-Alternative** (grüner Punkt) bei
  Standardgeräten derselben Kategorie.
- **Referenzgerät sichtbar** ("vs. Hersteller Modell") in der
  Objektansicht.
- **Zusätzlicher Katalog-Filter** nach Unterkategorie (Wärmegeräte,
  Kaffeegeräte, Spülmaschinen, Zusatzgeräte) bei EcoGastro- und
  Standardgeräten, verwaltet über einen neuen Admin-Tab "Katalog-Gruppen".
- **Objekt-Konzept generalisiert:** nicht mehr rein küchenfokussiert
  (neue Bereichstypen Gastraum/Lager/Aussenbereich), Lüftungsbemessung
  bleibt optional und wird nur bei gesetztem Küchentyp aktiv angezeigt.
- Objekttyp **"Kantine" in "Mensa"** umbenannt (inkl. Migration
  bestehender Daten).

## Geklärt, ohne Codeänderung

- Auslastungs-Default war bereits korrekt bei 100 % (ein verwandter,
  echter Bug beim Neuanlegen von Kühlraum-Standardgeräten — Default
  "Hoch/Effizient" statt "Mittel/Standard" — wurde behoben).
- Objekte sind bereits editierbar, Auftraggeber ebenfalls.
- Wärmestrahler/-lampe/-brücke bleiben als drei separate Kategorien
  bestehen, Energiewerte aktuell bewusst identisch, Differenzierung folgt
  später.

## Zurückgestellt

- **Min/Max-Validierung der Gerätegrösse pro Kategorie** — braucht eine
  Schema-Migration und Admin-UI, als eigenes Feature vorgesehen.
- **Prozessbasierter Gerätevergleich** (Version 2, `usage_h*`-Felder) —
  grösseres, separates Feature.

## Offen (braucht Martins Input)

- **Fehlende Spülmaschinen-Werte im Katalog** — kann nicht von aussen
  geprüft werden (kein Zugriff auf die Live-Produktionsdatenbank), Martin
  bittet, selbst unter `/devices?tab=standard` nachzusehen.

---

*Vollständige Nachverfolgung aller ~30 Einzelpunkte mit Status, Tests und
technischen Details: `docs/konzept_testbericht_250904.md` im
`Eartheffect-AG/energy_tool`-Repository (privat).*
