# Konzept: EcoGastro-Zertifizierung für energieeffiziente Küchen

Stand: 2026-07-19 — Erstkonzept zur Diskussion. Fachkonzept (Prozesse, Rollen,
Datenflüsse, Kriterien-Mapping); technische Architektur/Datenmodell folgt in
einem separaten Schritt, sobald das Vorgehen hier steht.

Grundlage: `Faktenblatt Kriterien EcoGastro Küchen-Zertifizierung.pdf`,
`20251114_Grundlagenmodul_Prozesse_Skript.pdf`,
`20251126_Fachmodul_Geräte_Energie_Skript_final.pdf`,
`20251201_Fachmodul energetische Küchenplanung T1_Skript_final.pdf`, sowie
die bestehenden Applikationen `energy_tool` (Energietool,
energy.ecogastro.org) und `energycheck` (check.ecogastro.org).

---

## 1. Ausgangslage

EcoGastro zertifiziert heute bereits Geräte (Förderprogramm) und plant, Gastro-
planer über einen dreistufigen Lehrgang auszubilden. Neu soll die **ganze
Küche** eines Gastronomiebetriebs zertifiziert werden — nicht nur einzelne
Geräte. Der Weg dahin läuft über zwei Teilzertifikate:

- **Teilzertifikat 1 (Technik/Konzept, 30 Kriterien):** wird geprüft, sobald
  frühestens 3 Monate vor der Inbetriebsetzung (IBS) klar ist, wie die Küche
  geplant und gebaut wird bzw. wurde. Grundlage ist primär die Planung des
  Gastroplaners mit dem Energietool, verifiziert durch ein Abnahme-Audit vor
  Ort.
- **Teilzertifikat 2 (Betrieb/Handling, 18 Kriterien):** wird frühestens 6
  Monate nach IBS geprüft und alle 2 Jahre wiederholt (Re-Zertifizierung).
  Grundlage ist primär EnergyCheck (Self-Assessment von Mitarbeitenden und
  Betriebsleitung), ergänzt durch ein schlankes Audit.

Als IBS-Datum gilt der Tag des offiziellen Gästeempfangs. Beide Teilzertifikate
zusammen ergeben das **Vollzertifikat «EcoGastro-zertifizierte Küche»**
(energieeffizient gebaut *und* betrieben). Mit nur Teilzertifikat 1 besteht ein
**Basis-Zertifikat**.

Zwei Applikationen decken heute schon wesentliche Teile davon ab, ohne dass sie
bisher als Zertifizierungs-Werkzeuge verknüpft sind:

| App | Rolle heute | Rolle in der Zertifizierung |
|---|---|---|
| **Energietool** (`energy_tool`, FastAPI/SQLite, energy.ecogastro.org) | Planungstool für Gastroplaner: Geräteauswahl, Effizienzvergleich, Energiebedarfsberechnung, VA102-01-Lüftungsbemessung | Liefert die Datenbasis und Nachweise für einen grossen Teil der 30 Kriterien aus Teilzertifikat 1 |
| **EnergyCheck** (`energycheck`, FastAPI/SQLite, check.ecogastro.org) | Self-Assessment-Tool: Mitarbeitende und Leitung beantworten Fragen zum Umgang mit Küchengeräten, System berechnet Sparpotenzial und erstellt einen PDF-Report | Deckt inhaltlich fast deckungsgleich die 18 Kriterien aus Teilzertifikat 2 ab («bedarfsgerechter und effizienter Umgang mit …») |

Was fehlt, ist die **dritte Applikation**, die den Zertifizierungsprozess selbst
führt: Kriterien-Tracking pro Betrieb, Verknüpfung der Nachweise aus
Energietool und EnergyCheck, Audit-Abwicklung (intern und extern), Zertifikats-
Ausstellung/-Verwaltung sowie das Register der zertifizierten Gastroplaner.
Dazu kommt eine Erweiterung des Energietools als Ablage-, Informations- und
Austauschplattform für die Planer.

---

## 2. Prozess im Überblick

```
Planung (Gastroplaner, Energietool)
        │
        ▼
Bau der Küche
        │
        ▼
Inbetriebsetzung (IBS) = Tag des offiziellen Gästeempfangs
        │
   ├─ ab 3 Mt. vor IBS möglich: Abnahme-Audit vor Ort
        │
        ▼
Teilzertifikat 1 (Technik/Konzept) — BASIS-ZERTIFIKAT
        │
        │  ~6 Monate Betrieb
        ▼
EnergyCheck-Durchlauf (Mitarbeitende + Leitung)
        │
   ab 6 Mt. nach IBS möglich: Betriebs-Audit (schlank)
        │
        ▼
Teilzertifikat 2 (Betrieb/Handling) — VOLLZERTIFIKAT
        │
        │  alle 2 Jahre
        ▼
Re-Zertifizierung (nur Teilzertifikat 2: neuer EnergyCheck-Durchlauf + Kurz-Audit)
```

Zusätzliche Voraussetzungen, die parallel dazu erfüllt sein müssen (siehe
Faktenblatt/Grundlagenmodul, Anhang «Bedingungen»):

- Der Betrieb muss die Zulassungsbedingungen erfüllen (mind. 1 Koch EFZ oder
  Systemgastronom EFZ fest angestellt, gewerbliche Küche mit mind. 2
  Grossgeräten, ≥ 50 Mahlzeiten/Tag, ≥ 5 Öffnungstage/Woche, bei
  Hauptküche+Nebenküchen muss die Hauptküche mitzertifiziert werden, die
  Zertifizierung gilt für alle Gastrobereiche).
- Die Planung muss durch einen **zertifizierten Gastroplaner** erfolgt sein
  (Kriterium 25) — das ist der Anknüpfungspunkt zum Zertifizierungslehrgang.
- Für die Re-Zertifizierung gilt: letztes (Re-)Zertifikat nicht älter als 2
  Jahre, Zielvereinbarungs-/EVA-Pflichten müssen erfüllt sein, bei grossen
  betrieblichen Veränderungen ist keine reine Re-Zertifizierung mehr möglich,
  sondern eine (Teil-)Neuzertifizierung.

---

## 3. Rollen

| Rolle | Wer | Aufgaben |
|---|---|---|
| **Gastroplaner** | extern, EcoGastro-zertifiziert | Plant die Küche im Energietool, stellt die Kriterien-relevanten Daten/Nachweise bereit, bespricht Gerätevergleich mit Bauherrschaft |
| **Betrieb / Bauherrschaft** | Gastronomiebetrieb bzw. Investor | Zielobjekt der Zertifizierung, beauftragt Gastroplaner, nimmt an EnergyCheck teil (Kriterien 31–48), sieht eigenen Zertifizierungsstatus |
| **Auditor** | intern (Eartheffect) **oder** extern, geschult | Führt das Abnahme-Audit (Teilzertifikat 1) und Betriebs-Audit (Teilzertifikat 2) vor Ort durch, dokumentiert Feststellungen, gibt Empfehlung zuhanden EcoGastro-Admin |
| **EcoGastro-Admin** (Eartheffect) | intern | Verwaltet Zertifizierungsfälle, prüft/genehmigt Audits, stellt Zertifikate aus, verwaltet Auditoren- und Planer-Register, Reminder für Re-Zertifizierung |
| **Gerätehersteller** | extern | Liefert Geräteschulung (Kriterium 23), liefert Messdaten für neue Geräte im Energietool-Katalog (bestehender Prozess, unverändert) |

Da externe Auditoren vorgesehen sind, braucht die neue App eine **Auditoren-
Verwaltung**: Profil, ein einfaches Qualifikations-Flag (die eigentliche
Schulung/Qualifizierung läuft **ausserhalb der App** als interner
EcoGastro-Prozess — die App bildet nur das Ergebnis ab), Zuteilung zu Fällen
(z. B. nach Region), eingeschränkte Sicht (ein Auditor sieht nur die ihm
zugewiesenen Fälle, nicht alle Betriebe), und eine Nachvollziehbarkeit, wer
welche Feststellung gemacht hat (Vier-Augen-Prinzip: Auditor dokumentiert,
EcoGastro-Admin gibt frei). **Strikte Rollentrennung:** ein zertifizierter
Gastroplaner kann nicht gleichzeitig als Auditor registriert sein — die App
verhindert das aktiv (z. B. bei Registrierung geblockt oder Warnung bei
Doppelzuweisung).

---

## 4. Kriterien-Mapping: welche Quelle deckt was ab?

Ziel ist maximale Automatisierung: wo immer möglich, wird ein Kriterium direkt
aus bestehenden Daten (Energietool-Projekt, EnergyCheck-Ergebnis) hergeleitet,
statt es ein zweites Mal manuell zu erfassen. Wo das (noch) nicht geht, braucht
es entweder eine gezielte Erweiterung der Quell-App um ein zusätzliches Feld,
oder es bleibt eine manuelle Bestätigung/ein Beleg-Upload/eine
Audit-Feststellung.

Legende: 🟢 automatisch aus bestehenden Daten ableitbar · 🟡 automatisch
ableitbar nach kleiner Erweiterung der Quell-App · 🔴 manuelle Bestätigung,
Beleg-Upload oder Audit-Feststellung nötig

### Teilzertifikat 1 — Technik/Konzept (30 Kriterien, Handlungsfeld in Klammern)

| # | Kriterium | Quelle | Status |
|---|---|---|---|
| 1 | Geprüfte Geräte: EcoGastro-zertifiziert (Kochen) | Energietool: Gerätekatalog kennzeichnet EcoGastro-Geräte bereits (`eff_class` „Sehr hoch") | 🟢 |
| 2 | Geprüfte Geräte: weitere Label (Energystar, Topten etc.) (Kochen) | Energietool-Katalog kennt aktuell keine Drittlabel | 🟡 (Katalogfeld ergänzen) |
| 3 | Gerätevergleich Planer liegt vor (Kochen) | Energietool: Vergleichsfunktion alt/neu-Standard/neu-effizient je Gerät | 🟢 |
| 4 | Gerätevergleich mit Bauherrschaft besprochen (Kochen) | Prozessschritt, kein Systemwert | 🔴 (Bestätigung + optional Protokoll-Upload) |
| 5 | Zentral geschaltete Arbeitssteckdosen (Kochen) | Entschieden: keine Energietool-Erweiterung, manuell im Zertifizierungsfall, tief priorisiert | 🔴 (bewusst, siehe 4a) |
| 6 | Geräte/Kraftanschlüsse zentral/zonenhaft geschaltet (Kochen) | wie Kriterium 5 | 🔴 (bewusst) |
| 7 | WRG Kühlanlagen genutzt (Kälte/Kühlen) | Entschieden: keine Energietool-Erweiterung, manuell im Zertifizierungsfall | 🔴 (bewusst) |
| 8 | Dimensionierung Kühl-/TK-Räume angemessen (Kälte/Kühlen) | Energietool liefert automatische Soll/Ist-Einschätzung, Auditor bestätigt/korrigiert vor Ort | 🟡 (Vorprüfung + Audit, siehe 4a) |
| 9 | Verhältnis autonome/zentrale Kühlgeräte angemessen (Kälte/Kühlen) | Energietool erfasst pro Kühlgerät zentral/steckerfertig | 🟢 (Kennzahl), Schwellenwert für „angemessen" nötig |
| 10 | Anteil offener Kühl-/TK-Geräte (Kälte/Kühlen) | Energietool erfasst Gerätetyp offen/geschlossen | 🟢 |
| 11 | Zugang TK-Räume nur via Kühlräume (Kälte/Kühlen) | bauliche Anordnung, nicht im Energietool | 🔴 (Audit/Planbestätigung) |
| 12 | Gleichzeitigkeitsfaktor Lüftung berechnet (Lüftung) | Energietool VA102-01-Modul berechnet dies bereits | 🟢 |
| 13 | Latente/sensible Wärmelasten berechnet (Lüftung) | Energietool VA102-01-Modul (Gastraum/Nebenräume) | 🟢 |
| 14 | Effizientes Steuersystem Küche (Lüftung) | Korrigiert: Energietool erfasst die Bedarfsführung der Lüftungssteuerung bereits | 🟢 |
| 15 | Effizientes Steuersystem Gastraum (Lüftung) | wie Kriterium 14 | 🟢 |
| 16 | WRG Küche vorhanden (Lüftung) | Entschieden: keine Energietool-Erweiterung, manuell im Zertifizierungsfall | 🔴 (bewusst) |
| 17 | WRG Gastbereich vorhanden (Lüftung) | Entschieden: keine Energietool-Erweiterung, manuell im Zertifizierungsfall | 🔴 (bewusst) |
| 18 | Effiziente Beleuchtung (Beleuchtung) | Entschieden: Bestätigung/Beleg des Lichtplaners im Zertifizierungsfall | 🔴 (bewusst) |
| 19 | Warmwasseranschluss Geschirrreinigung (Geschirr-Reinigung) | Entschieden: manuell im Zertifizierungsfall, unabhängig vom Energietool-Katalog | 🔴 (bewusst) |
| 20 | Warmwasseranschluss Kochgeräte (Kochen) | wie Kriterium 19 | 🔴 (bewusst) |
| 21 | Effizienz übriger diverser Verbraucher (Diverse Verbraucher) | Entschieden: weggelassen, keine digitale Abbildung | 🔴 (bewusst, kein System-Feld) |
| 22 | WRG aus Abwasser vorhanden (Diverse Verbraucher) | Entschieden: weggelassen, keine digitale Abbildung | 🔴 (bewusst, kein System-Feld) |
| 23 | Geräteschulung Hersteller erfolgt (Kochen) | Entschieden: Selbstdeklaration per Checkbox, kein Beleg-Upload | 🔴 (bewusst, ohne Beleg) |
| 24 | Anteil effizienter Kaffeegeräte (Kaffee) | Entschieden: tief priorisiert, vorerst keine Prüfung/Katalog-Erweiterung | 🔴 (bewusst, zurückgestellt) |
| 25 | Betrieb durch zertifizierten Gastroplaner geplant (Strategisch) | neues Gastroplaner-Register + Projektzuordnung im Energietool | 🟢 (sobald Register existiert) |
| 26 | Verantwortliche Person Energie definiert (Strategisch) | Stammdatum, in neuer App zu erfassen | 🔴 |
| 27 | Energiebedarf separat erfasst (Strategisch) | Entschieden: weggelassen, keine digitale Abbildung | 🔴 (bewusst, kein System-Feld) |
| 28 | Weitere Bereiche Teil der Zertifizierung (Strategisch) | Scope-Definition, Stammdatenfeld in neuer App | 🔴 (Stammdatum, kein Audit nötig) |
| 29 | Weitere Gewerke nebst Strom Teil der Zertifizierung (Strategisch) | Entschieden: weggelassen, keine digitale Abbildung | 🔴 (bewusst, kein System-Feld) |
| 30 | Kochsystem und Einfluss auf Energieverbrauch (Strategisch) | Energietool erfasst Küchentyp/Kochsystem bereits | 🟢 |

*(Ursprüngliche Einschätzung vor dem Review — siehe Abschnitt 4a für die
tatsächlichen Entscheidungen und die aktuelle Bilanz nach Durchsprache mit
Martin.)*

### Teilzertifikat 2 — Betrieb/Handling (18 Kriterien, frühestens 6 Mt. nach IBS)

| # | Kriterium | Quelle | Status |
|---|---|---|---|
| 31 | Regelmässige Energieberatung Technik (Strategisch) | Entschieden: bestehende EnergyCheck-Frage `mgmt_energieaudit` genügt 1:1 | 🟢 |
| 32 | Regelmässige Energieberatung Handhabung (Strategisch) | Entschieden: weggelassen, keine digitale Abbildung | 🔴 (bewusst, kein System-Feld) |
| 33 | Regelmässige Schulung Management (Strategisch) | Entschieden: bestehende Frage `mgmt_schulung` wird trotz Unschärfe als Näherung verwendet | 🟢 (Näherung, keine neue Frage) |
| 34 | EcoGastro.Train alle 2 Jahre (Kochen) | Geklärt: „EcoGastro.Train" = EnergyCheck-Durchlauf selbst; automatisch erfüllt durch den generellen 2-Jahres-Zyklus | 🟢 (keine separate Reminder-Logik nötig) |
| 35 | Nutzerdaten vernetzter Küchengeräte erfasst (Kochen) | Entschieden: weggelassen, keine digitale Abbildung | 🔴 (bewusst, kein System-Feld) |
| 36 | Umgang Küchengeräte (Kochen) | **EnergyCheck**: Assessment-Score Kombidämpfer/Fritteuse/Salamander/Griddle/Tellerwärmer/Pastakocher | 🟢 |
| 37 | Umgang Küchen-Zusatzgeräte (Kochen) | Entschieden: weggelassen als eigenständiges Kriterium | 🔴 (bewusst, kein System-Feld) |
| 38 | Umgang Kältegeräte (Kälte/Kühlen) | Entschieden: EnergyCheck-Score Kategorie `pluskuehl` (klare Trennung zu Kriterium 43) | 🟢 |
| 39 | Umgang Geschirr-Reinigungsgeräte (Geschirr-Reinigung) | EnergyCheck-Score Kategorie `spuelmaschine` | 🟢 |
| 40 | Umgang Beleuchtung (Beleuchtung) | Entschieden: neue EnergyCheck-Kategorie „Beleuchtung" wird ergänzt | 🟡 (sobald Kategorie existiert) |
| 41 | Umgang Kaffeegeräte (Kaffee) | Verifiziert & bestätigt: EnergyCheck-Score Kategorie `kaffee` existiert bereits | 🟢 |
| 42 | Umgang diverse Verbraucher (Diverse Verbraucher) | Entschieden: keine eigene Kategorie, über `management`-Kategorie als Näherung mitabgedeckt | 🟡 (Näherung) |
| 43 | Umgang Kühl-/TK-Geräte (Kälte/Kühlen) | Entschieden: EnergyCheck-Score Kategorie `minuskuehl` (ex „TK-Raum", klare Trennung zu Kriterium 38) | 🟢 |
| 44 | Pflege/Reinigung Kühl-/TK-Anlagen (Kälte/Kühlen) | Verifiziert: `minuskuehl` hat bereits „Vereisung & Kondensatoren reinigen"; `pluskuehl` fehlt analoge Frage, wird ergänzt | 🟡 (teilweise 🟢, Lücke bei pluskuehl) |
| 45 | Schaltzeiten Küche (Lüftung) | Entschieden: bestehende `lueftung`-Frage wird in 3 Fragen nach Zone aufgeteilt | 🟢 (nach Aufteilung) |
| 46 | Schaltzeiten Gastraum (Lüftung) | wie Kriterium 45 | 🟢 (nach Aufteilung) |
| 47 | Schaltzeiten Nebenräume (Lüftung) | wie Kriterium 45 | 🟢 (nach Aufteilung) |
| 48 | Pflege/Reinigung Lüftungsanlagen (Lüftung) | Verifiziert & bestätigt: bestehende Frage „Fettfilter monatlich reinigen" genügt 1:1 | 🟢 |

*(Ursprüngliche Einschätzung vor dem Review — siehe Abschnitt 4a für die
tatsächlichen Entscheidungen und die aktuelle Bilanz nach Durchsprache mit
Martin.)*

---

## 4a. Prüflogik je Kriterium — Entscheidungen aus dem Review

Dieser Abschnitt wird laufend im Kriterium-für-Kriterium-Review mit Martin
ergänzt. Er hält fest, *wie* ein Kriterium konkret geprüft wird (Schwellenwert,
Nachweisform), nicht nur *woher* die Daten kommen (das steht in Abschnitt 4).

**Teilzertifikat 1**

- **Kriterium 1** (EcoGastro-zertifizierte Geräte, Kochen): Erfüllt ab einem
  **Mindestanteil** der Hauptgeräte, nicht schon ab einem einzelnen Gerät.
  Offen: konkreter Schwellenwert (%) und Definition „Hauptgerät" (vermutlich
  Anlehnung an die im Energietool bereits geführten Gerätekategorien/
  VA102-Klassen) — später festzulegen.
- **Kriterium 2** (weitere Label, Kochen): **Keine Katalog-Erweiterung** im
  Energietool. Wird als manuelle Angabe im Zertifizierungsfall erfasst
  (Freitext + Beleg-Upload durch den Planer).
- **Kriterium 3** (Gerätevergleich liegt vor, Kochen): Muss für **alle
  Hauptgeräte** im Projekt vorliegen, nicht nur für die energieintensivsten.
- **Kriterium 4** (Gerätevergleich mit Bauherrschaft besprochen, Kochen):
  **Selbstdeklaration per Checkbox** durch den Planer, kein Beleg-Upload
  nötig.
- **Kriterien 5 + 6** (zentral geschaltete Arbeitssteckdosen / Kraftanschlüsse
  zonenhaft geschaltet, Kochen): **keine** Erweiterung des Energietools.
  Manuelle Erfassung im Zertifizierungsfall, **tief priorisiert** (kein
  MVP-Kandidat). Damit bleiben beide Kriterien dauerhaft 🔴 statt 🟡.
- **Kriterium 7** (WRG Kühlanlagen genutzt, Kälte/Kühlen): ebenfalls
  **keine** Energietool-Erweiterung, manuelle Erfassung im
  Zertifizierungsfall. Bleibt dauerhaft 🔴 statt 🟡.
- **Kriterium 8** (Dimensionierung Kühl-/TK-Räume angemessen, Kälte/Kühlen):
  **zweistufig** — das Energietool liefert automatisch eine
  Soll/Ist-Einschätzung anhand der normativen Grössenklassen (Warnung bei
  Abweichung), der Auditor bestätigt oder korrigiert das Ergebnis vor Ort.
  Damit bleibt der Status 🟡, aber mit definiertem Zusammenspiel
  Vorprüfung/Audit statt offener Frage.
- **Kriterien 9 + 10** (Verhältnis autonome/zentrale Kühlgeräte / Anteil
  offener Kühl-/TK-Geräte, Kälte/Kühlen): **kein fixer Schwellenwert im
  System.** Das Energietool zeigt automatisch die berechnete Kennzahl (%) an,
  die Beurteilung „angemessen" trifft der Auditor im Projekt-Kontext. Damit
  bleiben beide 🟢 für die Kennzahl-Bereitstellung, die Erfüllungs-Bewertung
  selbst ist aber ein Auditor-Ermessensentscheid, keine reine Systemlogik.
- **Kriterium 11** (Zugang TK-Räume nur via Kühlräume, Kälte/Kühlen): Planer
  lädt vorab einen Grundriss-/Situationsplan hoch, der die Anordnung zeigt;
  der Auditor verifiziert vor Ort nur noch stichprobenartig statt die ganze
  Prüfung neu zu machen.
- **Kriterium 12** (Gleichzeitigkeitsfaktor Lüftung berechnet, Lüftung):
  bestätigt vollautomatisch über das bestehende VA102-01-Modul, keine
  Änderung nötig.
- **Kriterium 13** (latente/sensible Wärmelasten berechnet, Lüftung):
  ebenfalls bestätigt vollautomatisch über das bestehende VA102-01-Modul.
- **Kriterien 14 + 15** (effizientes Steuersystem Küche/Gastraum, Lüftung):
  **Korrektur gegenüber Erstfassung** — das Energietool erfasst die
  Bedarfsführung der Lüftungssteuerung bereits. Beide Kriterien sind damit
  🟢 automatisch auswertbar, keine Erweiterung nötig.
- **Kriterium 16** (WRG Küche vorhanden, Lüftung): **keine**
  Energietool-Erweiterung, manuelle Erfassung im Zertifizierungsfall.
- **Kriterium 17** (WRG Gastbereiche vorhanden, Lüftung): gleich wie
  Kriterium 16, manuell im Zertifizierungsfall.
- **Kriterium 18** (effiziente Beleuchtung, Beleuchtung): **Bestätigung des
  Lichtplaners** wird als Beleg im Zertifizierungsfall hochgeladen — kein
  Ausbau des Energietools zu einem eigenen Beleuchtungs-Modul.
- **Kriterien 19 + 20** (Warmwasseranschluss Geschirr-Reinigung/Kochgeräte,
  Geschirr-Reinigung/Kochen): **manuelle Erfassung im Zertifizierungsfall**,
  unabhängig vom Energietool-Gerätekatalog (keine Katalog-Prüfung/-Erweiterung
  vorgesehen).
- **Kriterien 21 + 22** (Effizienz diverser Verbraucher / WRG aus Abwasser,
  Diverse Verbraucher): **weggelassen** — keine digitale Abbildung, kein
  Checklisten-Feld und keine Beleg-Vorgabe in der neuen App. Bleiben reine
  Ermessensfrage des Auditors vor Ort (Freitext-Notiz im Audit-Bericht,
  kein strukturiertes Kriterium).
- **Kriterium 23** (Geräteschulung Hersteller erfolgt, Kochen):
  **Selbstdeklaration per Checkbox**, kein Beleg-Upload nötig.
- **Kriterium 24** (Anteil effizienter Kaffeegeräte, Kaffee): **tief
  priorisiert / vorerst nicht geprüft**, da Kaffeegeräte aktuell kein
  EcoGastro-Förderschwerpunkt sind. Keine Energietool-Katalog-Erweiterung im
  MVP.
- **Kriterium 25** (Betrieb durch zertifizierten Gastroplaner geplant,
  Strategisch): bestätigt vollautomatisch, sobald das Gastroplaner-Register
  (Abschnitt 6) existiert und mit der Projekt-Planer-Zuordnung im Energietool
  verknüpft ist.
- **Kriterium 26** (Verantwortliche Person Energie definiert, Strategisch):
  Stammdatenfeld (Name/Rolle) im Zertifizierungsfall der neuen App.
- **Kriterium 27** (Energiebedarf separat erfasst/Messkonzept, Strategisch):
  **weggelassen** — keine digitale Abbildung, bleibt reine
  Audit-Ermessensfrage.
- **Kriterium 28** (weitere Bereiche nebst Küche Teil der Zertifizierung,
  Strategisch): Stammdatenfeld (Scope-Definition) im Zertifizierungsfall,
  wird bei Falleröffnung festgelegt.
- **Kriterium 29** (weitere Gewerke nebst Strom Teil der Zertifizierung,
  Strategisch): **weggelassen** — keine digitale Abbildung, bleibt reine
  Audit-Ermessensfrage (anders als Kriterium 28, das als Stammdatum geführt
  wird).
- **Kriterium 30** (Kochsystem und Einfluss auf Energieverbrauch,
  Strategisch): bestätigt vollautomatisch aus dem im Energietool bereits
  erfassten Küchentyp/Kochsystem, als generierter Reporttext.

**Teilzertifikat 1 — Bilanz nach Review:** von 30 Kriterien sind **10 🟢**
vollautomatisch (1, 3, 12, 13, 14, 15, 25, 30, plus 9/10 als Kennzahl mit
Auditor-Bewertung), **2 🟡** mit definiertem Zusammenspiel
Vorprüfung/Audit (8) bzw. Beleg-Vorprüfung (11), **18 🔴** bewusst als
manuelle Erfassung/Selbstdeklaration/Audit-Ermessen entschieden — davon 3
komplett weggelassen (21, 22, 27, 29). Wichtigste Erkenntnis aus dem Review:
Martin hat sich mehrfach bewusst **gegen** eine Energietool-Erweiterung
entschieden, auch wo eine automatisierbar gewesen wäre (5, 6, 7, 16, 17, 19,
20, 24) — der Fokus liegt auf schnellem MVP statt maximaler Automatisierung
um jeden Preis.

**Teilzertifikat 2**

- **Kriterium 31** (regelmässige Energieberatung Technik, Strategisch): die
  bestehende EnergyCheck-Leitungsfrage `mgmt_energieaudit` genügt 1:1 als
  Nachweis, keine Anpassung nötig.
- **Kriterium 32** (regelmässige Energieberatung Handhabung, Strategisch):
  **weggelassen** — keine neue EnergyCheck-Frage, keine digitale Abbildung,
  bleibt reine Audit-Ermessensfrage.
- **Kriterium 33** (regelmässige Schulung Management, Strategisch): die
  bestehende EnergyCheck-Frage `mgmt_schulung` wird **trotz Unschärfe**
  (zielt eigentlich auf Mitarbeitende, nicht aufs Management) als Näherung
  verwendet — keine neue, präzisere Frage im MVP.
- **Kriterium 34** (EcoGastro.Train alle 2 Jahre, Kochen): **wichtige
  Klärung** — „EcoGastro.Train" ist der alte Name, heute entspricht das dem
  **EnergyCheck-Durchlauf selbst**. Das Kriterium ist damit kein
  eigenständiges Reminder-Thema mehr, sondern automatisch erfüllt, sobald im
  Rahmen der Re-Zertifizierung ohnehin ein neuer EnergyCheck-Durchlauf
  stattgefunden hat (🟢, deckungsgleich mit dem generellen 2-Jahres-Zyklus
  aus Abschnitt 5 — keine separate Reminder-Logik nötig).
- **Kriterium 35** (Nutzerdaten vernetzter Küchengeräte erfasst, Kochen):
  **weggelassen** — keine digitale Abbildung im MVP, bleibt reine
  Audit-Ermessensfrage.
- **Kriterium 36** (Umgang Küchengeräte, Kochen): bestätigt vollautomatisch
  aus dem EnergyCheck-Gesamtscore über die Hauptgerätekategorien.
- **Kriterium 37** (Umgang Küchen-Zusatzgeräte, Kochen): **weggelassen** als
  eigenständiges Kriterium — keine eigene EnergyCheck-Kategorie, keine
  digitale Abbildung.
- **Kriterium 38 vs. 43** (Umgang Kältegeräte / Umgang Kühl-/TK-Geräte,
  Kälte/Kühlen): klare 1:1-Trennung — Kriterium 38 nutzt den EnergyCheck-Score
  der Kategorie `pluskuehl`, Kriterium 43 den Score der Kategorie
  `minuskuehl`. Beide damit 🟢, sauber getrennt statt einem kombinierten
  Kälte-Score.
- **Kriterium 39** (Umgang Geschirr-Reinigungsgeräte, Geschirr-Reinigung):
  bestätigt vollautomatisch aus dem EnergyCheck-Score der Kategorie
  `spuelmaschine`.
- **Kriterium 40** (Umgang Beleuchtung, Beleuchtung): **neue
  EnergyCheck-Kategorie** wird ergänzt (analog `management`/`lueftung`, 2–3
  Fragen) — damit wird aus diesem Kriterium 🟡 statt dauerhaft 🔴, sobald die
  Kategorie existiert.
- **Kriterium 41** (Umgang Kaffeegeräte, Kaffee): bestätigt vollautomatisch
  aus dem bestehenden EnergyCheck-Score der Kategorie `kaffee`.
- **Kriterium 42** (Umgang diverse Verbraucher, Diverse Verbraucher): **keine
  eigene Kategorie**, wird über die bestehende `management`-Kategorie
  (Abschaltplan, Geräte-Alter) als Näherung mitabgedeckt.
- **Kriterium 44** (Pflege/Reinigung Kühl-/TK-Anlagen, Kälte/Kühlen):
  verifiziert — für `minuskuehl` existiert bereits die Frage „Vereisung &
  Kondensatoren reinigen" (🟢). Für `pluskuehl` fehlt eine analoge Frage
  bisher; wird ergänzt (🟡 bis dahin).
- **Kriterien 45–47** (Schaltzeiten Küche/Gastraum/Nebenräume, Lüftung): die
  bestehende `lueftung`-Frage „Betriebszeiten & Stufen anpassen" wird **in
  drei separate Fragen** (Küche/Gastraum/Nebenraum) aufgeteilt — damit werden
  alle drei Kriterien 🟢 statt einer gemeinsamen Näherung.
- **Kriterium 48** (Pflege/Reinigung Lüftungsanlagen, Lüftung): bestätigt
  vollautomatisch, bestehende Frage „Fettfilter monatlich reinigen" genügt
  1:1.

**Teilzertifikat 2 — Bilanz nach Review:** von 18 Kriterien sind **13 🟢**
(31, 33, 34, 36, 38, 39, 41, 43, 45, 46, 47, 48, plus 44 zur Hälfte) direkt
aus bestehenden EnergyCheck-Scores ableitbar oder durch geklärte
Näherungen/Klarstellungen (33, 34) erledigt — deutlich mehr als in der
Erstfassung, weil EnergyCheck schon heute viel mehr abdeckt als angenommen
(`kaffee`, `lueftung`, `management` als bereits existierende Kategorien).
**2 🟡** brauchen noch kleine Ergänzungen im Fragenkatalog (40 neue
Beleuchtungs-Kategorie, 44 fehlende Pluskühlung-Reinigungsfrage). **3 🔴**
sind bewusst weggelassen (32, 35, 37, 42 teilweise über `management`
mitabgedeckt). Insgesamt liegt die Automatisierungsquote in Teilzertifikat 2
nach dem Review klar höher als ursprünglich geschätzt.

---

## 4b. Quantitative Bewertung und Zertifizierungsschwelle

Bisher war jedes Kriterium ein reines Ja/Nein. Damit müsste ein Betrieb
praktisch alle gescorten Kriterien erfüllen, um zertifiziert zu werden — das
ist weder realistisch noch berücksichtigt, dass ein Kriterium in der Küche
(z. B. Kälteanlage) energetisch viel mehr wiegt als eines bei der
Kaffeezubereitung. Vorschlag daher: ein gewichtetes Punktemodell mit
Mindestschwelle und drei Leistungsstufen, entschieden mit Martin wie folgt.

### Punktemodell pro Kriterium

- **Binär** (die meisten Kriterien): volle Punktzahl bei „erfüllt", 0 bei
  „nicht erfüllt".
- **Kriterium 1 (Skala, geklärt)**: volle Punktzahl ab **90 % derjenigen
  Gerätekategorien im Projekt, für die überhaupt ein EcoGastro-zertifiziertes
  Gerät existiert** (aktuell: Kombidämpfer, Multifunktionsgeräte,
  Spülmaschinen, Fritteusen, Warmhaltewagen/Bain-Marie, Pizzaöfen, Griddle —
  siehe Grundlagenmodul). Kategorien ohne zertifiziertes Marktangebot zählen
  **nicht** in den Nenner — das Kriterium bestraft also nicht, wenn es für
  eine benötigte Gerätekategorie schlicht kein EcoGastro-Gerät gibt.
  Unterhalb 90 % linear abgestuft (z. B. 45 % erfüllte Kategorien → 50 % der
  Punkte).
- **Kriterien 9/10 (Auditor-Ermessen mit Abstufung, geklärt)**: kein starrer
  Systemwert. Der Auditor vergibt **0/1/2 Punkte** je nach Einschätzung
  (nicht angemessen / teilweise angemessen / angemessen) und muss dazu eine
  **kurze Begründung** im Audit-Bericht erfassen — Pflichtfeld, damit die
  Bewertung nachvollziehbar bleibt und nicht als reine Zahl ohne Kontext im
  System landet.
- **Weggelassene Kriterien** (21, 22, 27, 29 in Teilzertifikat 1; 32, 35, 37
  in Teilzertifikat 2): zählen **nicht** in Zähler oder Nenner des Scores.
  Sie bleiben aber als **Veto-Kriterien** beim Auditor — eine gravierende
  Feststellung dort (Freitext-Notiz im Audit-Bericht) kann die Zertifizierung
  trotz ausreichendem Score blockieren, ohne dass sie den Score selbst
  beeinflusst.

### K.O.-Kriterien (zwingend, unabhängig vom Score)

- Die generellen Zulassungsbedingungen aus Abschnitt 2 (Koch EFZ/Systemgastronom
  EFZ, ≥ 2 Grossgeräte, ≥ 50 Mahlzeiten/Tag, ≥ 5 Öffnungstage/Woche etc.) —
  das sind ohnehin Vorbedingungen, keine Scoring-Kriterien.
- **Kriterium 25** (Betrieb durch zertifizierten Gastroplaner geplant): muss
  erfüllt sein, unabhängig vom sonst erreichten Score — kein Punktbeitrag,
  sondern reines Gate. Fehlt es, gibt es kein Zertifikat, egal wie gut der
  Rest bewertet ist.

### Gewichtung nach Handlungsfeld (Energierelevanz)

Drei Gewichtsstufen, angelehnt an den energetischen Anteil des jeweiligen
Handlungsfelds (Kochen macht laut Grundlagenmodul rund ein Drittel des
gesamten Stromverbrauchs aus, Kälteanlagen laufen 24/7 durch):

| Handlungsfeld | Gewicht | Begründung |
|---|---|---|
| Kochen | hoch (3) | grösster Einzelposten, ~1/3 des Stromverbrauchs |
| Kälte/Kühlen | hoch (3) | Dauerlast 24/7, zweitgrösster Posten |
| Lüftung | mittel (2) | signifikant, stark objektabhängig |
| Geschirr-Reinigung | mittel (2) | warmwasserlastig, spürbarer Anteil |
| Strategisch/Organisatorisch | mittel (2) | kein direkter kWh-Hebel, sichert aber Nachhaltigkeit der übrigen Massnahmen |
| Beleuchtung | tief (1) | kleinerer Anteil ggü. Kochen/Kälte |
| Kaffee | tief (1) | Nischenverbraucher |
| Diverse Verbraucher | tief (1) | Rest-Kategorie, meist ohnehin weggelassen |

### Berechnung

```
Score (%) = Σ (erreichte Punkte × Gewicht) / Σ (max. Punkte × Gewicht)
```

— nur über die gescorten Kriterien (weggelassene und K.O.-Kriterien zählen
nicht mit), getrennt berechnet für Teilzertifikat 1 und Teilzertifikat 2.

**Konkret nach heutigem Stand des Reviews:**

- **Teilzertifikat 1**: 30 Kriterien − 4 weggelassen (21, 22, 27, 29) − 1
  K.O. (25) = **25 gescorte Kriterien**, gewichtete Maximalpunktzahl **61**
  (Kochen 8 Kriterien × 3 = 24, Kälte/Kühlen 5 × 3 = 15, Lüftung 6 × 2 = 12,
  Geschirr-Reinigung 1 × 2 = 2, Strategisch 3 × 2 = 6, Beleuchtung 1 × 1 = 1,
  Kaffee 1 × 1 = 1).
- **Teilzertifikat 2**: 18 Kriterien − 3 weggelassen (32, 35, 37) = **15
  gescorte Kriterien**, gewichtete Maximalpunktzahl **32** (Kälte/Kühlen 3 ×
  3 = 9, Kochen 2 × 3 = 6, Lüftung 4 × 2 = 8, Strategisch 2 × 2 = 4,
  Geschirr-Reinigung 1 × 2 = 2, Beleuchtung 1 × 1 = 1, Kaffee 1 × 1 = 1,
  Diverse Verbraucher 1 × 1 = 1).

### Schwelle und Leistungsstufen

Mindestschwelle für den Erhalt eines Teilzertifikats: **60 %** des
gewichteten Scores (plus zwingende Erfüllung aller K.O.-Kriterien). Darüber
hinaus drei Stufen, um Betrieben einen Anreiz über das Minimum hinaus zu
geben:

| Stufe | Score-Bereich |
|---|---|
| Kein Zertifikat | < 60 % |
| Basis | 60–74 % |
| Silber | 75–89 % |
| Gold | ≥ 90 % |

Die Stufe wird **pro Teilzertifikat separat** ausgewiesen. Im Vollzertifikat
(beide Teilzertifikate erfüllt) werden beide Stufen sichtbar gemacht, z. B.
„EcoGastro-zertifizierte Küche — Planung: Gold, Betrieb: Silber". Das ergänzt
die bestehende Basis-/Vollzertifikat-Unterscheidung aus Abschnitt 1 (die
bleibt als grobe Einordnung bestehen), verfeinert sie aber um eine sichtbare
Qualitätsstufe — mit direktem Mehrwert für die in Abschnitt 8 offene Frage
nach der öffentlichen Sichtbarkeit/Marketingwert des Zertifikats.

---

## 5. Die neue Zertifizierungs-App

Arbeitstitel: **EcoGastro.Cert** (Platzhalter). Analog zu Energietool und
EnergyCheck als eigenständige FastAPI/SQLite-Applikation auf demselben Server
denkbar — das ist aber eine Architekturfrage für den nächsten Schritt, hier
nur die fachlichen Bausteine.

**Kernobjekte (fachlich, nicht als Datenmodell):**

- **Zertifizierungsfall**: ein Betrieb/Projekt auf seinem Weg durch die
  beiden Teilzertifikate. Verweist auf das zugehörige Energietool-Projekt
  (für Teilzertifikat 1) und den zugehörigen EnergyCheck-Betrieb (für
  Teilzertifikat 2).
- **Kriterien-Checkliste**: die 48 Kriterien mit Status je Fall (erfüllt /
  nicht erfüllt / nicht anwendbar), Quelle (automatisch/manuell/Audit),
  Nachweis (Verweis auf Energietool-Bericht, EnergyCheck-Report oder
  hochgeladenes Dokument).
- **Audit**: ein Termin/Durchlauf (Abnahme-Audit oder Betriebs-Audit),
  zugewiesener Auditor, Vor-Ort-Checkliste, Fotos, Feststellungen,
  Empfehlung, Freigabe durch EcoGastro-Admin.
- **Zertifikat**: ausgestelltes Dokument (Basis- oder Vollzertifikat) mit
  Gültigkeitsdauer, Leistungsstufe je Teilzertifikat (Basis/Silber/Gold,
  siehe Abschnitt 4b), PDF, öffentlich verifizierbar (z. B. QR-Code/Link
  analog zum bestehenden EnergyCheck-Report-Token-Muster).
- **Gastroplaner-Register**: zertifizierte Planer, Lehrgangs-/Modulstatus,
  Erfahrungsmodul-Nachweise (siehe Abschnitt 6), Verknüpfung zum
  Energietool-Login.
- **Auditoren-Register**: interne und externe Auditoren, Qualifikations-Flag,
  Zuteilungsregion, zugewiesene Fälle — mit erzwungener Trennung von der
  Planer-Rolle (siehe Abschnitt 3).
- **Öffentliches Verzeichnis (Betriebe)**: durchsuchbare, öffentliche Liste
  zertifizierter Betriebe mit Stufe (Basis/Silber/Gold je Teilzertifikat) und
  Gültigkeitsdatum — Marketingwert für EcoGastro und die Betriebe, separat
  vom internen Zertifizierungsfall.
- **Öffentliches Verzeichnis (Planer)**: analog, aber bewusst schlank — nur
  Firma, Name und Jahr der Zertifizierung, keine Projekt-/Kundenreferenzen
  ohne separates Opt-in (siehe Abschnitt 5a).
- **Zahlung**: Audit-Gebühr pro Zertifizierungsfall, analog zum bestehenden
  Stripe-Kauf-/Rabattcode-System in EnergyCheck (Details zu Höhe/Struktur
  folgen mit der technischen Architektur).

**Automatisierte Datenflüsse (Ziel):**

- Sobald ein Gastroplaner im Energietool ein Projekt als „bereit zur
  Zertifizierung" markiert, wird im Hintergrund ein Zertifizierungsfall
  angelegt bzw. aktualisiert; die 🟢/🟡-Kriterien aus Abschnitt 4 werden
  automatisch vorausgefüllt.
- 6 Monate nach IBS wird automatisch ein EnergyCheck-Durchlauf für den Betrieb
  angestossen (Reminder-Mail, analog zur bestehenden `automation.py`-Logik in
  EnergyCheck); nach Abschluss werden die 🟢/🟡-Kriterien aus Teilzertifikat 2
  automatisch aus den EnergyCheck-Scores übernommen.
- Der Auditor sieht bei Terminvereinbarung bereits die vorausgefüllte
  Checkliste und muss vor Ort nur noch die 🔴-Kriterien sowie Stichproben der
  🟢/🟡-Kriterien prüfen — das hält das Audit schlank.
- Zwei Jahre nach der letzten Zertifizierung/Re-Zertifizierung löst das System
  automatisch eine Erinnerung an Betrieb und Admin aus (analog EcoGastro.Train-
  Reminder, Kriterium 34).
- Das System vergleicht laufend die Betriebs-/Gerätedaten aus Energietool und
  EnergyCheck mit dem Stand der letzten (Re-)Zertifizierung; bei wesentlicher
  Abweichung (Gerätebestand, Kapazität, Öffnungstage — Toleranz noch zu
  definieren) wird automatisch eine Warnung an den EcoGastro-Admin ausgelöst,
  der über (Teil-)Neuzertifizierung statt einfacher Re-Zertifizierung
  entscheidet.

**Schlankes Audit-Vorgehen (Vorschlag):**

1. **Vorbereitung**: Auditor öffnet den Fall, sieht vorausgefüllte Checkliste
   samt Nachweisen aus Energietool/EnergyCheck.
2. **Vor-Ort-Begehung**: nur die verbleibenden 🔴-Kriterien plus eine kurze
   Stichprobe der automatisch übernommenen Werte (Plausibilitätscheck, keine
   Neuberechnung). Erfassung mobil (Tablet/Smartphone), mit Foto-Beleg wo
   sinnvoll (z. B. Kühlraum-Zugang, Wärmerückgewinnung, Beleuchtung).
3. **Abschluss**: Auditor gibt Empfehlung ab (bestanden/nicht bestanden/mit
   Auflagen), EcoGastro-Admin prüft und gibt frei.
4. **Zertifikat**: bei Freigabe automatische Zertifikatserstellung (PDF +
   öffentlicher Verifizierungslink), Gültigkeitsfrist wird gesetzt.

---

## 5a. Marketing & öffentlicher Auftritt (EcoGastroCert)

Die neue Zertifizierungs-App ist nicht nur internes Verwaltungswerkzeug für
Eartheffect, sondern soll aktiv Bauherrschaften — insbesondere die öffentliche
Hand — dafür gewinnen, ihre Gastroküchen zertifizieren zu lassen. Das
verändert den Charakter der App: sie bekommt einen eigenständigen,
öffentlichkeitswirksamen Frontend-Teil unter eigener Marke, **EcoGastroCert**,
als Subdomain (`cert.ecogastro.org`), analog zu `energy.ecogastro.org`/`check.ecogastro.org`, getrennt von den technischen
Marken Energietool/EnergyCheck — analog zu Minergie als etabliertem Vorbild
für ein glaubwürdiges, unabhängiges Gebäude-/Prozesslabel.

### Zielgruppen und Nutzenversprechen

| Zielgruppe | Nutzenversprechen |
|---|---|
| Private Bauherrschaften (Hotels, Gastro-Ketten, Investoren) | tiefere Betriebskosten, Zukunftssicherheit bei steigenden Energiepreisen, Imagegewinn bei Gästen und ESG-Reporting institutioneller Investoren |
| Öffentliche Bauherrschaften (Gemeinden, Kantone, Spitäler, Schulen, Alters-/Pflegeheime) | Nachweis gegenüber Politik/Bevölkerung für eigene Netto-Null-/Energiestrategien, geringeres Planungsrisiko durch klare Systemgrenzen |
| Vergabestellen/Beschaffung | fertiges, rechtlich sauber eingeordnetes Kriterium (siehe Vergabe-Kit unten), das nicht selbst entwickelt werden muss |
| Architekten/Fachplaner (HLKKSE) | Multiplikatoren, Profilierung mit Nachhaltigkeitskompetenz |
| Gastronomiebetriebe (Endnutzer) | Reputation bei Gästen, analog Bio-Label/Green Globe |

**Kernbotschaft:** „Die einzige Zertifizierung, die Planung *und* Betrieb
prüft" — Abgrenzung zu reinen Geräte-Labels. Getragen von EnergieSchweiz und
dem Verband Schweizer Gastroplaner als Glaubwürdigkeits-Anker.

### Rechtlicher Hebel bei öffentlichen Ausschreibungen

Das revidierte Bundesgesetz über das öffentliche Beschaffungswesen (BöB)
verankert Nachhaltigkeit als Grundsatz (Art. 21) und ruft Vergabestellen dazu
auf, vermehrt auf Qualität/Nachhaltigkeit ausgerichtete **Zuschlagskriterien**
zu verwenden, statt nur nach dem günstigsten Preis zu vergeben. Wichtig:
Nachhaltigkeitsanforderungen sollten als **Zuschlagskriterium** (Bonuspunkte)
formuliert werden, nicht als technische Spezifikation mit Ausschlusswirkung
— sonst droht Anfechtung wegen Wettbewerbsbeschränkung. **Minergie-ECO** ist
der etablierte Präzedenzfall: Kantone/Gemeinden referenzieren das Label
direkt in ihren Vergabeunterlagen. Dieses Muster soll EcoGastroCert
übernehmen — siehe das separate **Vergabe-Kit** (Musterausschreibungstext +
Argumentarium für Beschaffungsstellen, eigenes Dokument).

### Landingpage-Struktur (öffentliche Routen, fachlich)

- **`/`** — Hero mit den drei Zielgruppen-Kacheln (private/öffentliche
  Bauherrschaft, Gastronomiebetriebe), Ablaufprozess-Grafik (Planung → Bau →
  Audit → Zertifikat → Betrieb → Re-Zertifizierung), Trust-Kennzahlen
  („X zertifizierte Küchen, Y zertifizierte Planer, Z kWh eingespart"),
  Corporate-Design-Bezug zu EnergieSchweiz/Verband Schweizer Gastroplaner.
- **`/kuechen`** — öffentliches, durchsuchbares Verzeichnis zertifizierter
  Betriebe mit Stufe (Basis/Silber/Gold) und Gültigkeitsdatum (siehe
  Abschnitt 5).
- **`/planer`** (neu) — öffentliches Verzeichnis zertifizierter Gastroplaner.
  Umfang bewusst schlank gehalten aus Diskretionsgründen: **Firma, Name,
  Jahr der Zertifizierung** — keine Projekt-/Kundenreferenzen ohne separates
  Opt-in.
- **`/oeffentliche-hand`** (neu) — dedizierter Bereich für Vergabestellen:
  Download des Vergabe-Kits, Kurzargumentarium, Kontaktformular für Beratung.
- **`/zertifikat/{id}`** — öffentliche, teilbare Verifizierungsseite pro
  Zertifikat mit Badge-Embed-Code für die Website des Betriebs (analog zum
  EnergyCheck-Report-Token-Muster).
- **News/Case-Studies-Bereich** — Kurzporträts zertifizierter Küchen
  (Vorher/Nachher, kWh/Jahr gespart, O-Ton Betreiber), sobald erste Fälle
  vorliegen.

Admin-seitig braucht es ein einfaches CMS für Landingpage-Texte und Case
Studies, damit Eartheffect Inhalte ohne Entwicklerhilfe pflegen kann —
analog zum bestehenden Mailtemplate-/Settings-Muster in EnergyCheck.

**Priorisierung:** Landingpage und öffentliche Verzeichnisse werden **in
Abschnitt 9 nach vorne gezogen** — Marketing/Kundenakquise ist strategisch
zentral und soll nicht erst nach Abschluss des internen
Zertifizierungsprozesses kommen.

---

## 6. Gastroplaner-Zertifizierung (Anknüpfungspunkt zu Kriterium 25)

Der dreistufige Lehrgang ist bereits definiert (Grundlagenmodul 1 Tag,
Fachmodul Geräte & Energie 1 Tag, Fachmodul Küchenplanung 2 Tage mit
Abschlussprüfung). Damit Kriterium 25 („Betrieb wurde durch zertifizierten
Gastroplaner geplant") automatisch geprüft werden kann, muss die neue App den
Zertifizierungsstatus der Planer selbst führen:

- Zulassungsbedingungen zum Lehrgang (Unabhängigkeit von Geräte-/Anlagen-
  Anbietern) als Stammdatum.
- Abschluss aller Module inkl. Abschlussprüfung Fachmodul Küchenplanung →
  Zertifizierungsstatus „aktiv".
- Erhalt der Zertifizierung: Teilnahme an einem halbtägigen Erfahrungsmodul im
  ersten Jahr, danach mindestens 3 Teilnahmen innerhalb von 5 Jahren — das
  braucht eine Reminder-/Fristenlogik, ähnlich wie bei der 2-Jahres-Re-
  Zertifizierung der Betriebe.
- Verknüpfung mit dem Energietool-Login des Planers, damit ein Projekt
  automatisch dem richtigen (zertifizierten oder nicht-zertifizierten)
  Planer zugeordnet ist.

---

## 7. Erweiterung des Energietools: Ablage-, Informations- und Austauschplattform für Planer

Da die Gastroplaner bereits im Energietool eingeloggt sind (Rolle
„Konsulent"), liegt es nahe, diese Plattform dort zu integrieren statt in der
neuen Zertifizierungs-App — die Zertifizierungs-App bleibt Auditoren/Admin-
lastig, das Energietool bleibt der Ort, an dem Planer arbeiten. Es gibt hier
bereits einen ersten Baustein: die kürzlich gebaute Anleitungsseite
`/anleitung` mit Grundablauf, FAQ etc.

Vorgeschlagene Bausteine (Ausbau der bestehenden Planer-Sicht):

- **Ablage**: Schulungsunterlagen (die drei Skripte), Normen-Referenzen
  (SWKI VA102-01, SNG 482500), Kriterienliste als Nachschlagewerk, aktuelle
  Massnahmenkataloge — versioniert, mit Änderungsdatum.
- **Information**: Ankündigungen (z. B. neue Geräte im Katalog, Änderungen an
  Kriterien oder Normen), FAQ (Ausbau des bestehenden `/anleitung`-Bereichs).
- **Austausch**: einfaches Forum oder zumindest ein Kommentar-/Frage-Kanal
  pro Thema — Umfang hängt davon ab, wie viel Moderationsaufwand EcoGastro
  leisten will; als Einstieg reicht evtl. auch nur eine moderierte FAQ ohne
  offenes Forum.
- **Eigener Zertifizierungsstatus**: Planer sieht auf einen Blick seinen
  Lehrgangs-/Zertifizierungsstatus und die Frist fürs nächste
  Erfahrungsmodul (Datenquelle: Gastroplaner-Register aus Abschnitt 6).
- **Eigene zertifizierte Projekte**: Übersicht, in welchem Status sich die
  eigenen Projekte im Zertifizierungsprozess befinden (Datenquelle:
  Zertifizierungsfall aus der neuen App).

---

## 8. Offene Fragen (grösstenteils im Review geklärt)

Die meisten der ursprünglich offenen Punkte wurden im Kriterien- und
Konzept-Review mit Martin geklärt; nur Kriterium 24 (Kaffeegeräte) bleibt
bewusst offen zurückgestellt. Der Abschnitt bleibt als Entscheidungs-Log
stehen.

- **Auditoren-Schulung — geklärt**: die Qualifizierung externer Auditoren ist
  ein rein interner Prozess bei EcoGastro/Eartheffect, **nicht Teil des
  Tools**. Die neue App führt nur ein Auditoren-Register mit einem
  Qualifikations-Flag (qualifiziert: ja/nein, manuell durch den Admin
  gesetzt) — kein Schulungs-/Prüfungsmodul in der App selbst.
- **Planer als Auditor — geklärt**: **strikte Trennung**. Ein zertifizierter
  Gastroplaner kann nicht gleichzeitig als Auditor auftreten, auch nicht bei
  fremden Projekten — beide Rollen sind im Register gegenseitig
  ausschliessend.
- **Verbleibende kleine Erweiterungen — geklärt, bleiben früh im MVP**
  (siehe Abschnitt 4a im Detail): EnergyCheck-Kategorie „Beleuchtung"
  (Kriterium 40), fehlende Reinigungsfrage bei `pluskuehl` (Kriterium 44),
  Aufteilung der Lüftungs-Schaltzeiten-Frage nach Zone (Kriterien 45–47).
  Bestätigt: alle drei werden früh priorisiert, nicht auf „nach dem MVP"
  verschoben.
- **Kaffeegeräte im Energietool** (Teilzertifikat 1, Kriterium 24): bewusst
  zurückgestellt (siehe 4a) — bleibt aber im Hinterkopf, falls Kaffee künftig
  doch EcoGastro-Förderschwerpunkt wird.
- **Öffentliche Sichtbarkeit des Zertifikats — geklärt**: Es soll ein
  **öffentliches Verzeichnis/Suche** zertifizierter Betriebe geben (nicht nur
  ein Verifizierungslink pro Einzelzertifikat), inkl. der neuen Basis-/
  Silber-/Gold-Stufe aus Abschnitt 4b und Gültigkeitsdatum. Das wird als
  neues Kernobjekt „öffentliches Verzeichnis" in Abschnitt 5 ergänzt.
- **Bezahlmodell — geklärt**: Die Zertifizierung erhält eine **Audit-Gebühr**,
  analog zum bestehenden Stripe-Kauf-/Rabattcode-System in EnergyCheck.
  Details (Höhe, ob pro Teilzertifikat oder pauschal, Rabattcode-Logik
  analog EnergyCheck) sind ein Folgeschritt, sobald die App-Architektur
  ansteht.
- **„Grosse Veränderungen" — geklärt**: die Erkennung erfolgt **automatisch**
  über einen Abgleich der Betriebs-/Gerätedaten zwischen Energietool und
  EnergyCheck seit der letzten (Re-)Zertifizierung (z. B. wesentliche
  Änderungen im Gerätebestand, Kapazität, Öffnungstage). Bei Abweichung über
  einer noch zu definierenden Toleranz wird automatisch eine Warnung an den
  EcoGastro-Admin ausgelöst, der dann entscheidet, ob eine (Teil-)
  Neuzertifizierung statt einer einfachen Re-Zertifizierung nötig ist —
  keine reine Selbstmeldepflicht des Betriebs mehr.

---

## 9. Vorschlag für die Priorisierung (MVP zuerst)

**Aktualisiert:** Marketing/Kundenakquise (Landingpage, öffentliche
Verzeichnisse) wurde bewusst nach vorne gezogen — das ist der stärkste Hebel,
um Bauherrschaften und insbesondere die öffentliche Hand für
EcoGastroCert zu gewinnen, und soll nicht erst nach Abschluss des internen
Zertifizierungsprozesses kommen.

1. Gastroplaner-Register + Verknüpfung Energietool-Projekt → Fall (Basis für
   Kriterium 25 und generell für die Fallanlage).
2. Kriterien-Checkliste mit den bereits 🟢 verfügbaren automatischen Werten
   aus Energietool und EnergyCheck (schnellster Mehrwert, kein Mehraufwand in
   den Quell-Apps nötig).
3. Audit-Modul (Checkliste, Foto, Freigabe) für die 🔴-Kriterien.
4. Zertifikats-Ausstellung (PDF + Verifizierungslink) und Fristen-/
   Reminder-Engine (Re-Zertifizierung, Erfahrungsmodul, EcoGastro.Train).
5. **Landingpage + öffentliche Verzeichnisse (Betriebe und Planer, Abschnitt
   5a)** — vorgezogen, da zentraler Marketing-/Akquise-Hebel, insbesondere
   für die öffentliche Hand.
6. **Vergabe-Kit für Beschaffungsstellen** (Musterausschreibungstext +
   Argumentarium, eigenes Dokument) — direkt einsatzbereit sobald die
   Landingpage steht.
7. Die drei kleinen, bestätigt priorisierten EnergyCheck-Erweiterungen
   (Beleuchtungs-Kategorie, Pluskühl-Reinigungsfrage, Lüftungs-Schaltzeiten
   nach Zone) sowie die übrigen 🟡-Kriterien aus Abschnitt 4a.
8. Auditoren-Verwaltung für externe Auditoren (Register, Rollentrennung zu
   Gastroplanern).
9. Ablage-/Info-/Austauschplattform im Energietool.
10. Zahlungsmodul (Audit-Gebühr) — eher spätere Ausbaustufe, sobald der
    Kern-Zertifizierungsprozess läuft.
11. Automatischer Abgleich „grosse Veränderungen" (Change-Detection zwischen
    Energietool/EnergyCheck und letzter Zertifizierung) — sinnvoll erst,
    sobald mehrere Re-Zertifizierungszyklen echte Vergleichsdaten liefern.

---

## 10. Nächste Schritte

**Stand:** Das Fachkonzept ist inhaltlich abgeschlossen — Prozess, Rollen,
alle 48 Kriterien Kriterium-für-Kriterium geklärt (Abschnitt 4a), das
quantitative Bewertungsmodell mit Schwellen und Stufen (Abschnitt 4b), sowie
Marketing/Landingpage/Vergabe-Kit (Abschnitt 5a). Die ursprünglich hier
notierten Punkte (Rückmeldung zu Abschnitt 8, Verifikation der 🟡-Annahmen)
sind erledigt.

Offen sind noch, in empfohlener Reihenfolge:

1. **Technisches Architekturkonzept — erledigt**: siehe
   `docs/architektur_ecogastrocert.md` (nur im privaten `energy_tool`-Repo,
   nicht im öffentlichen `Public_Docs`, da Serverdetails/SSH-Pfade enthalten
   sind). Tech-Stack-Entscheid, Datenmodell, Schnittstellen zu Energietool/
   EnergyCheck, Deployment.
2. **Kleine, bereits spezifizierte Erweiterungen vorziehen**: die drei
   bestätigten EnergyCheck-Ergänzungen (Beleuchtungs-Kategorie,
   Pluskühl-Reinigungsfrage, Lüftungs-Schaltzeiten nach Zone) und die
   Energietool-Erweiterungen für Kriterium 1 (Tracking EcoGastro-Anteil je
   Gerätekategorie) sind unabhängig von der neuen App umsetzbar und könnten
   parallel schon begonnen werden.
3. **Skalierungsgrenzwerte/Rundungsregeln** für das Punktemodell aus
   Abschnitt 4b technisch spezifizieren (z. B. exakte Rundung bei der
   linearen Skalierung von Kriterium 1).
4. **Externe Validierung**: das Konzept mit EnergieSchweiz und dem Verband
   Schweizer Gastroplaner als benannte Partner gegenspiegeln, bevor die
   Umsetzung beginnt — insbesondere die Kriterien-Gewichtung (Abschnitt 4b)
   und das Vergabe-Kit (rechtliche Formulierung).
5. **Domain/Branding EcoGastroCert** registrieren und Corporate Design
   (Logo, Farben) festlegen, bevor die Landingpage gebaut wird.
