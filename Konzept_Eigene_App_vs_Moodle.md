# Kritisches Konzept: Eigene App vs. Moodle

*Projekt „Future Perfect Next Level" — Plattformentscheid für das neue BNE-Lernmodul*

Erstellt für: Future Perfect / eartheffect · Stand: 30. Juli 2026

## 1. Fragestellung

Könnte das neue, zusammengeführte BNE-Lernmodul statt auf Moodle auf einer eigenen, selbst entwickelten App realisiert werden? Dieses Konzept prüft die Frage bewusst kritisch: Es geht nicht darum, eine Eigenentwicklung schönzureden, sondern die realen Nachteile offenzulegen und zu prüfen, ob und wie sie sich entschärfen lassen. Am Schluss steht eine begründete Empfehlung.

## 2. Ausgangslage: Was Moodle heute für das Projekt leistet

Die bestehenden sechs Kurse laufen bereits auf Moodle, und ein Teil der Zielgruppe der geplanten Lehrpersonenbefragung sind explizit Moodle-Verantwortliche an den Schulen. Moodle ist zudem bei vielen Schweizer Berufsfachschulen bereits im produktiven Einsatz und institutionell freigegeben (IT-Sicherheitsprüfung, Datenschutz, Beschaffung sind i. d. R. bereits erfolgt). Das Train-the-Trainer-Format des Projekts ist ausdrücklich auch auf Moodle-Kompetenzen ausgerichtet. Eine Abkehr von Moodle betrifft also nicht nur eine technische, sondern auch eine organisatorische und vertrauensbezogene Dimension.

## 3. Mögliche Vorteile einer Eigenentwicklung

- Volle Kontrolle über die Nutzererfahrung: z. B. ein natives, interaktives Mindmap- oder Reflexionstool statt eines Verweises auf ein externes Video plus manuellen Upload.
- Schlankere, fokussierte Oberfläche ohne den administrativen "Ballast" eines allgemeinen LMS — potenziell ansprechender für jugendliche Berufslernende.
- Eigene, feingranulare Lernstandsdaten für die Wirkungsmessung des Projekts (wichtig gegenüber Fördergebern).
- Leichter Raum für innovative, KI-gestützte oder adaptive Lernelemente, die in Moodle nur mit Zusatzaufwand (Plugins, Wartung) umsetzbar wären.
- Eigene Marke und Wiedererkennbarkeit für „Future Perfect", unabhängig vom Erscheinungsbild einzelner Schul-Moodle-Instanzen.

## 4. Kritische Nachteile und wie sie sich entschärfen lassen

| Nachteil | Warum kritisch | Mögliche Gegenmassnahme |
|---|---|---|
| Entwicklungs- und Unterhaltskosten | Eine eigene App erfordert kontinuierliche Weiterentwicklung, Hosting, Bugfixing und Sicherheitsupdates über Jahre – ein Aufwand, den ein kleines Projektteam ohne eigene Software-Abteilung kaum dauerhaft stemmen kann. | Keine vollständige LMS-Kernfunktionalität (Login, Rollen, Notenbuch) selbst bauen. Stattdessen nur klar abgegrenzte interaktive Bausteine entwickeln und diese per LTI 1.3 oder iFrame in die bestehenden Moodle-Instanzen der Schulen einbetten. |
| Institutionelle Freigabe an jeder Schule | Jede Schule/jeder Kanton hat eigene IT-Sicherheits- und Beschaffungsprozesse für neue Cloud-Tools. Eine unbekannte, neue App müsste diesen Prozess bei potenziell Dutzenden Schulen einzeln durchlaufen – Moodle hat diesen Prozess bei den meisten Schulen bereits hinter sich. | Hybrid-/Embed-Ansatz: Die Schule bleibt bei ihrer bereits freigegebenen Moodle-Instanz; nur ein eingebettetes Zusatzmodul ist neu, wodurch der Freigabeaufwand pro Schule deutlich kleiner bleibt. |
| Datenschutz (nDSG) und Hosting | Eine eigene App, die Personendaten von Lernenden verarbeitet, braucht eine eigene Datenschutz-Folgenabschätzung, Auftragsverarbeitungsverträge, ein Verarbeitungsverzeichnis und jährliche Schulungen – zusätzlicher administrativer Aufwand für ein kleines Team. | Hosting bei einem etablierten Schweizer Cloud-Anbieter (z. B. demselben, der bereits für die Mail-Infrastruktur genutzt wird), Datenminimierung (keine Klarnamen, pseudonyme IDs), einmalige Folgenabschätzung mit wiederverwendbarer Vorlage. |
| Barrierefreiheit (eCH-0059 / WCAG 2.1 AA) | Öffentlich bzw. institutionell genutzte Bildungsangebote in der Schweiz sollen WCAG 2.1 auf Stufe AA erfüllen. Moodle hat diese Arbeit weitgehend bereits geleistet; eine neue App beginnt bei null. | Accessibility von Beginn an ins Design-System einbauen (semantisches HTML, Tastaturbedienung, Kontrastprüfung), auf geprüfte Component-Bibliotheken setzen statt komplett eigenes UI, externes Accessibility-Audit vor dem Launch einplanen. |
| Interoperabilität / Lock-in-Risiko | Inhalte einer eigenen App sind an diese App gebunden. Ohne Exportformat sind sie verloren, falls Projekt oder Finanzierung enden oder eine Schule wechseln möchte. | Von Anfang an eine Exportfunktion (SCORM- oder xAPI-Paket) einplanen, damit Inhalte im Ernstfall in Moodle oder ein anderes LMS migriert werden können – eine explizite „Exit-Strategie". |
| Schulungsaufwand / Change Management | Eine komplett neue App bedeutet zusätzliches Onboarding für Lehrpersonen und Lernende – zusätzlich zum ohnehin geplanten Train-the-Trainer-Format zu den BNE-Inhalten. | Durch den Embed-Ansatz bleiben Lehrpersonen im gewohnten Moodle-Kurskontext; die neue Funktionalität wird als zusätzlicher Baustein eingeführt, nicht als Systemwechsel. |
| Funktionsverlust gegenüber einem reifen LMS | Moodle bietet einen reifen Funktionsumfang (Notenbuch, Rollen-/Rechtekonzept, Backup/Restore, Foren, Quiz-Engine, Plugin-Ökosystem, mehrsprachige Oberfläche). Das müsste in einer Eigenentwicklung erst nachgebaut werden – mit entsprechendem Reiferisiko in einer ersten Version. | Nur dort selbst entwickeln, wo Moodle nachweislich limitiert ist (z. B. spezifische interaktive Reflexions- oder KI-Tools); alles Etablierte (Kursstruktur, Bewertung, Rollen) bei Moodle belassen. |
| Nachhaltigkeit nach Projektende | Stiftungsfinanzierungen laufen typischerweise über 2–3 Jahre. Bei einer Eigenentwicklung droht danach ein unterhaltsloses, absterbendes System – Moodle-Inhalte dagegen können von der Schule selbst weitergeführt werden. | Eigenes Modul unter offener Lizenz veröffentlichen, damit Schulen oder eine Community es auch nach Projektende warten können; zusätzlich durch die Exportfunktion (siehe oben) abgesichert. |

## 5. Entscheidungsmatrix

Grobe qualitative Einschätzung entlang der wichtigsten Kriterien (++ sehr gut, + gut, o neutral, - schwach, -- sehr schwach):

| Kriterium | Moodle (Status quo) | Vollständige Eigenentwicklung | Hybrid (Empfehlung) |
|---|---|---|---|
| Entwicklungsaufwand | ++ | -- | o |
| Institutionelle Akzeptanz an Schulen | ++ | -- | + |
| Datenschutz-/Compliance-Aufwand | + | -- | o |
| Barrierefreiheit | + | - | o |
| Interoperabilität / kein Lock-in | ++ | -- | + |
| Individualisierbarkeit der UX | - | ++ | + |
| Innovationspotenzial (z. B. KI-Elemente) | o | ++ | + |
| Nachhaltigkeit ohne Projektteam | ++ | -- | + |
| Time-to-Market | ++ | -- | o |

## 6. Empfehlung: Hybrid-Ansatz statt Systemwechsel

Aus der Gegenüberstellung ergibt sich klar: Eine vollständige Eigenentwicklung als Ersatz für Moodle wäre für ein Projekt dieser Grösse mit hoher Wahrscheinlichkeit überdimensioniert und riskant – insbesondere wegen der institutionellen Freigabeprozesse an den Schulen und der fehlenden Kapazität für dauerhaften Unterhalt. Empfohlen wird stattdessen ein Hybrid-Ansatz:

- Moodle bleibt das tragende System für Kursstruktur, Rollen, Bewertung, Datenschutz-Grundlagen und Barrierefreiheit.
- Eigene Entwicklung beschränkt sich auf klar abgegrenzte, interaktive Zusatzmodule (z. B. ein digitales Mindmap-/Reflexionstool), die über LTI 1.3 oder eingebettet per iFrame in Moodle-Kurse eingebunden werden.
- Eine Exportfunktion (SCORM/xAPI) sichert die Inhalte unabhängig von der Lebensdauer des eigenen Zusatzmoduls.
- Vor einer grösseren Investition wird ein kleiner Pilot mit 1–2 Partnerschulen empfohlen, um die institutionelle Machbarkeit (IT-Freigabe, Datenschutz-Prüfung) frühzeitig zu testen, bevor in die volle Entwicklung investiert wird.

## 7. Grobe Aufwandseinschätzung (qualitativ)

Ohne verbindliche Kostenschätzung (dafür bräuchte es ein technisches Angebot), aber zur groben Einordnung des Aufwands:

| Phase | Inhalt | Grobe Aufwandsgrösse |
|---|---|---|
| Discovery & Pilot | Anforderungen an ein konkretes Zusatzmodul klären, IT-/Datenschutz-Machbarkeit mit 1–2 Partnerschulen testen | Klein |
| MVP eines Moduls | Ein einzelnes interaktives Zusatzmodul entwickeln und via LTI/iFrame in Moodle einbetten | Mittel |
| Rollout auf weitere Schulen | Ausrollen auf zusätzliche Schulen, inkl. deren individueller IT-Freigabe | Mittel bis Gross (abhängig von Schulanzahl) |
| Laufender Unterhalt | Sicherheitsupdates, Hosting, Bugfixing, Weiterentwicklung | Laufend, nicht einmalig – muss budgetiert werden |

## 8. Fazit

*Eine eigene App kann didaktisch und markentechnisch reizvoll sein, löst aber kein Problem, das aktuell drängend wäre – während sie mehrere neue, ernstzunehmende Probleme schafft (Institutionelle Akzeptanz, Datenschutz, Barrierefreiheit, Nachhaltigkeit nach Projektende). Der Hybrid-Ansatz nutzt die Stärken von Moodle (institutionell verankert, datenschutzkonform, barrierefrei, interoperabel) und ermöglicht gleichzeitig gezielte Innovation dort, wo Moodle tatsächlich an Grenzen stösst.*

## Quellen

- eCH-0059 Accessibility Standard V3.0 — WCAG 2.1 Konformitätsstufe AA für Schweizer E-Government- und Bildungsangebote (ech.ch).
- Neues Schweizerisches Datenschutzgesetz (nDSG), in Kraft seit 1.9.2023 — Anforderungen an Auftragsbearbeitung, Verarbeitungsverzeichnis und Datentransfer ins Ausland.
- SWITCH edu-ID — föderierte Login-Lösung für den Schweizer Bildungsbereich, primär im Hochschulumfeld etabliert (switch.ch/de/edu-id).
