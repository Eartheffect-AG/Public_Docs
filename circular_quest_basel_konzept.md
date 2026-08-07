# Circular Quest Basel — Arbeitskonzept

Stand: 03.08.2026

**Zweck dieses Dokuments:** lebendes Arbeitsdokument für Circular Quest
Basel — eine eigene, selbst gehostete Ablösung von Actionbound. Fasst
Konzept, aktuellen technischen Stand, Betrieb und offene Punkte an einem
Ort zusammen, damit künftige Anpassungen und Entscheidungen direkt hier
nachgetragen werden können, statt über mehrere Repos/Docs verstreut zu
bleiben. Quelle der Wahrheit für den *Code* bleibt trotzdem das Repo
[`Marflixx/Circular-Quest-Basel`](https://github.com/Marflixx/Circular-Quest-Basel)
(`README.md`, `docs/ARCHITECTURE.md`, `docs/DEPLOYMENT.md`, `backend/README.md`)
— dieses Dokument ist die verdichtete Gesamtsicht für Planung und Diskussion.

---

## 1. Ausgangslage & Ziel

Der Circular Quest Basel ist eine geführte Entdeckungstour durch Basel zum
Thema zirkuläres Bauen: App, physisches Erkundungsheft und reale Lernorte
(Franck Areal, Pförtnerhaus, Kreislaufhaus, K024, Elys, Weinlager,
Lysbüchel/LysP8, Bau 92, Warteck Areal, …) greifen ineinander. Rund zehn
Posten, durchgehendes didaktisches Gerüst sind die „drei Hebel des
zirkulären Bauens": Gebäude länger nutzen, Materialien länger nutzen,
kreislauffähig planen.

Bisher lief die App-Seite über **Actionbound**, eine Drittanbieter-Plattform
für standortbasierte Rallyes. Inhalte wurden redaktionell in Word-Tabellen
erarbeitet (Spalten „Art des Inhalts", „Inhalt", „P" für Punkte, „Notizen")
und manuell im Actionbound-Editor nachgebaut.

**Ziel dieses Projekts:** Actionbound durch eine eigene Applikation
ersetzen — volle Datenhoheit, keine wiederkehrenden Lizenzkosten, frei
gestaltbare Inhaltstypen (z. B. der Gebäude-Pass als eigene Komponente statt
Behelfslösung), und eine Plattform, die künftig auch für weitere
Circular-Quest-Touren in anderen Städten wiederverwendet werden kann.

**Vorgehen:** stufenweise Migration — MVP mit Pilot-Posten (Franck Areal),
danach schrittweiser Vollausbau aller Posten und Ablösung von Actionbound.

---

## 2. Aktueller Stand (03.08.2026)

Live unter **https://go.circularquest.ch** (vorherige Domain
`basel.circularquest.ch` leitet per 301 dorthin um). Umgesetzt ist deutlich
mehr als das ursprüngliche MVP:

- **Datenmodell** vollständig (Quest, Posten, Media, ContentBlock, Question,
  ParticipantSession, AnswerLog) inkl. Multi-Tenancy (mehrere Quests
  parallel möglich, siehe Abschnitt 4).
- **Admin-Oberfläche als vollwertiges Redaktions-CMS**: eigene
  Navigationsstruktur, Quest-Arbeitsbereich, Nutzer-Einladungsflow mit
  Rollen, E-Mail-Konfiguration, automatisches Backup — siehe Abschnitt 5.
- **REST-API** für die PWA: Session starten/pausieren, Posten per GPS oder
  QR freischalten, alle fünf Fragetypen serverseitig bewertet, korrekte
  Antworten werden nie an den Client ausgeliefert (siehe Abschnitt 6).
- **PWA-Frontend** (Vanilla JS, kein Build-Schritt) deckt den vollständigen
  Ablauf funktional ab — bewusst noch ohne UI/UX-Feinschliff.
- **Pilot-Content** (Franck-Areal-Intro + Posten 1) über ein Seed-Command
  geladen; GPS-Koordinaten dort noch ein grober, vor Ort zu verifizierender
  Platzhalter.
- **CI/CD**: GitHub Actions prüft jeden Push (Django-Checks, Migrations-
  Konsistenz, vollständige Testsuite gegen Postgres) — siehe Abschnitt 7.
- **Automatisches Backup** (Datenbank-Dump + Medien, lokal + optional
  WebDAV) — siehe Abschnitt 5.4.

**Noch offen:** generisches Content-Migrationsskript für die restlichen
Posten (aktuell nur der eine Pilot-Posten per Seed-Command erfasst), echter
Foto-Upload (Foto-Challenges zeigen aktuell nur eine lokale Browser-Vorschau),
UI/UX-Feinschliff des PWA-Frontends, Bezahlinterface (bewusst erst später,
siehe Abschnitt 10), Vor-Ort-Pilottest mit echten Teilnehmenden.

---

## 3. Warum eine eigene Applikation? (Actionbound vs. Eigenentwicklung)

| Kriterium | Actionbound | Eigene Applikation |
|---|---|---|
| Kosten | Wiederkehrende Lizenz-/Teilnehmerkontingente | Einmaliger Entwicklungsaufwand + Hosting (mitlaufend auf bestehendem Eartheffect-VPS) |
| Individualisierung | Design/Fragetypen vorgegeben | Vollständig frei gestaltbar (z. B. eigener Gebäude-Pass) |
| Datenhoheit/Datenschutz | Teilnehmerdaten beim Anbieter | Daten bleiben auf eigenem Server in der Schweiz |
| Skalierung auf weitere Touren | Zusatzkosten pro Bound/Kontingent | Eine Plattform, beliebig viele Quests ohne Zusatzlizenz (siehe Abschnitt 4) |
| Auswertung/Reporting | Eingeschränkt auf Actionbound-Statistiken | Frei definierbar (z. B. für Förderberichte) |
| Wartungsaufwand | Gering, Anbieter betreibt die Plattform | Eigene Verantwortung für Betrieb, Updates, Sicherheit |

---

## 4. Architektur & Datenmodell

### 4.1 Grundentscheid: eine Codebasis, viele Quests

Die Plattform ist **ein** Django-Projekt, das **mehrere** Quests (Touren)
gleichzeitig bedienen kann — nicht eine separate Installation pro
Stadt/Areal. Circular Quest Basel ist heute die einzige `Quest`-Instanz;
eine zweite Tour (andere Stadt, anderes Areal, andere Sprache) ist
ausschliesslich neue Daten, kein neuer Code, kein neues Deployment.

Mandantenfähigkeit läuft **"multi-tenant by row"**: jede Tabelle, die zu
einer Tour gehört, trägt (direkt oder über `Posten`) einen Fremdschlüssel
auf `Quest`. Ein Deployment, ein Backup, eine Codebasis.

### 4.2 Technologie-Stack

- **Backend:** Python 3.12 / Django 5.2 LTS, Django REST Framework für die API.
- **Datenbank:** SQLite in Produktion (`backend/db.sqlite3` auf dem VPS,
  passt zum Server-Muster der übrigen Eartheffect-Apps dort — kein Docker,
  keine gemeinsame DB); Postgres 16 nur in der CI-Pipeline (Abschnitt 7) und
  optional im dokumentierten Docker-Alternativweg. `DATABASE_URL` (via
  `django-environ`) steuert das zentral in `config/settings.py`.
- **Static Files:** Whitenoise mit `CompressedManifestStaticFilesStorage`
  (gehashte Dateinamen, korrektes Cache-Busting pro Deploy).
- **App-Server:** Gunicorn (`--workers 3`, siehe `deploy/circularquest.service`).
- **Frontend:** Progressive Web App, Vanilla JS ohne Build-Schritt
  (`frontend/static/quest/app.js` + `style.css`), erreichbar unter
  `/q/<quest-slug>/`. Standort/Kamera über Standard-Browser-APIs.
- **Karten:** OpenStreetMap/Leaflet für die Routen-Karte im Admin.
- **Hosting:** Eartheffect-VPS (venv + systemd + nginx + Certbot), siehe
  Abschnitt 8.

### 4.3 Kernentitäten

| Modell | Zweck |
|---|---|
| `Quest` | Eine Tour, z. B. „Circular Quest Basel". Name, Sprache, Stadt, Branding-Farbe, Logo. Tenancy-Grenze. |
| `Posten` | Ein Standort: GPS-Koordinaten, Geofence-Radius, QR-Fallback-Code, Gebäude-Pass-Felder (Baujahr, früher/heute, warum spannend, zirkulärer Schwerpunkt). |
| `Media` | Bild-, Video- oder Audiodatei, gehört explizit zu einer Quest. |
| `ContentBlock` | Ein Element eines Postens — 1:1 auf eine Zeile der ursprünglichen Word-Tabelle gemappt (`block_type` = „Art des Inhalts", `body` = „Inhalt", `points` = „P", `notes` = „Notizen"). Typen: Information, Video, Audio, Bild/Plakat, Gebäude-Pass, Frage, Foto-Challenge, Beobachtungsaufgabe, Erkundungsheft-Verweis, Rätsel, Übergang/Navigation. |
| `Question` | 1:1 an einen `ContentBlock` vom Typ „Frage" gehängt. Fünf Typen (MC, Schätzfrage, Richtig/Falsch, Sortierfrage, Zuordnungsfrage), typspezifische Daten als JSON in `options` statt fünf eigener Tabellen. |
| `ParticipantSession` | Ein Teilnehmer-/Team-Durchlauf, kein Login — anonymer `session_key` (UUID) für Fortschritt/Pause/Fortsetzen. |
| `AnswerLog` | Protokoll: welche Antwort auf welche Frage, mit wie vielen Punkten. `total_points` wird live aus diesem Protokoll aggregiert, keine eigene Punktestand-Tabelle (kein Risiko des Auseinanderlaufens). |
| `EmailSettings` | Singleton-Konfigseite für System-Mails (SMTP-Server/Port/User/Passwort/TLS, Absender). Ohne aktive Konfiguration Fallback auf `.env`. |
| `BackupSettings` | Singleton-Konfigseite fürs automatische Backup (siehe 5.4). |

### 4.4 Geolocation & QR-Fallback

`Posten.latitude`/`longitude`/`radius_meters` definieren eine Geofence
(Haversine-Distanzberechnung, `quest/geo.py`). Die API prüft beim
Freischalten, ob die Gerätekoordinaten innerhalb des Radius liegen.
`Posten.qr_fallback_code` ist der manuelle Ersatzweg für Orte mit
unzuverlässigem GPS-Signal (Innenhöfe, dicke Industriemauern) — ein Scan
des vor Ort ohnehin vorhandenen Plakats schaltet den Posten alternativ frei.

### 4.5 Bewertungslogik

`quest/grading.py` bewertet serverseitig alle fünf Fragetypen; der Client
entscheidet nie selbst, ob eine Antwort richtig war. `public_options()`
filtert die korrekten Antworten heraus, bevor eine Frage an die App
ausgeliefert wird (getestet in `test_api.py::test_posten_detail_hides_correct_answers`).
Geo- und Grading-Logik sind bewusst als eigenständige, ungebundene Module
gehalten, unabhängig von Django-Views testbar.

### 4.6 Bewusst (noch) nicht Teil des Datenmodells

- **Bezahlinterface** — sinnvoll erst in einer späteren Phase (Abschnitt 10).
  Das Modell steht dem nicht im Weg: ein künftiges `Order`/`Payment`-Modell
  würde sich an `Quest`/`ParticipantSession` anhängen, ohne Bestehendes zu ändern.
- **Mehrsprachige Inhalte pro Content-Block** — `Quest.language` ist
  vorbereitet, echte Mehrsprachigkeit (z. B. via `django-modeltranslation`)
  bewusst nicht Teil des aktuellen Stands.
- **Generisches Content-Migrationsskript** — der Pilot-Content ist direkt
  als Seed-Command erfasst; ein Skript, das beliebige Word-Tabellen liest
  (`python-docx`, Mapping „Art des Inhalts" → `block_type`), ist für die
  restlichen Posten noch zu bauen.

---

## 5. Admin-Oberfläche (Redaktions-CMS)

### 5.1 Navigationsstruktur

Die Sidebar hat genau zwei Rubriken:

1. **Quests** — jede bestehende Quest direkt als eigene, anklickbare Zeile
   (kein Umweg über eine Zwischenseite; `/admin/quest/` zeigt dieselbe
   Übersicht wie die Modell-Changelist selbst, ohne Redirect). Posten,
   Content-Blöcke, Fragen, Medienobjekte, Sessions und Antwortprotokolle
   erscheinen bewusst **nicht** als eigene, quest-übergreifende
   Menüpunkte — sonst würde man z. B. in einer flachen „alle Posten aller
   Quests"-Liste landen, ohne Kontext, wozu ein Eintrag gehört. Erreichbar
   bleiben sie trotzdem jederzeit über ihre eigene URL (z. B. für Lesezeichen).
2. **Einstellungen** — installationsweite Dinge ohne Quest-Bezug, fester
   Reihenfolge: Benutzer, Gruppen, E-Mail-Konfiguration, Backup.

### 5.2 Quest-Arbeitsbereich

Eine Quest öffnen zeigt ihren Namen als Titel und zwei Teile:

- **Arbeitsbereich** — Status (aktiv/inaktiv) mit Umschalt-Knopf direkt
  hier, plus Links zu allem, was zur Quest gehört: N Posten (darin pro Zeile
  „N Blöcke"-Link zu den Content-Blöcken, im Posten-Formular auch inline
  editierbar, bei Fragen zusätzlich das verknüpfte Question-Objekt), N
  Medienobjekte, N Teilnehmer-Sessions, N Antwortprotokolle, Routen-Karte,
  „App öffnen" (nur bei aktiven Quests).
- **Konfiguration** — Stammdaten (Name, Slug, Tagline, Beschreibung,
  Sprache, Stadt, Branding-Farbe, Logo) standardmässig nur **angezeigt**;
  erst „Anpassen" schaltet auf editierbare Felder um.

### 5.3 Nutzer einladen & Rollen

Neue Admin-/Redaktionsnutzer werden per E-Mail-Einladung angelegt (Admin →
Benutzer → „Nutzer einladen", erfordert `auth.add_user`). Zwei Rollen:

- **Redaktion** — darf Quests/Posten/Content-Blöcke/Fragen/Medien anlegen
  und bearbeiten; kein Löschen von Quests/Posten, kein Zugriff auf
  Teilnehmerdaten oder Nutzerverwaltung. Rechte definiert in
  `quest/management/commands/setup_groups.py`, bei jedem Deploy automatisch
  aktualisiert (`manage.py setup_groups`, idempotent).
- **Vollzugriff** — vollwertiger Superuser.

Der neue Nutzer erhält einen Link, um selbst ein Passwort zu setzen (7 Tage
gültig, Djangos eigener Token-Mechanismus) — kein Passwort wird per Mail
verschickt.

### 5.4 E-Mail-Konfiguration & Backup (Singleton-Konfigseiten)

Zwei Admin-Seiten folgen demselben Muster: **eine** Konfigurationszeile
statt einer Liste, per `get_solo()`-Klassenmethode geholt (`.first()`, sonst
neu anlegen — bewusst **kein** `get_or_create(pk=1)`, siehe Abschnitt 7.2).

- **E-Mail-Konfiguration**: SMTP-Server/Port/Benutzer/Passwort/TLS/SSL,
  Absender, „Aktiv"-Schalter, „Test-Mail senden" direkt auf der Seite.
  Passwort wird nie im Klartext angezeigt. Ohne aktive Konfiguration
  Fallback auf `DJANGO_EMAIL_*` in der `.env`.
- **Backup**: automatisches, komprimiertes Backup (Datenbank-Dump via
  Djangos `dumpdata` + alle Mediendateien) nach konfigurierbarem Zeitplan
  (täglich/wöchentlich/monatlich, Uhrzeit, Aufbewahrungsfrist in Tagen),
  optional zusätzlich auf ein WebDAV-Ziel (z. B. Infomaniak kDrive)
  hochgeladen. „Jetzt sichern" und „WebDAV-Verbindung testen" direkt auf
  der Seite, plus Liste der lokalen Backups mit Download-Link. Läuft als
  Hintergrund-Thread pro Gunicorn-Worker (`quest/backup.py`,
  `start_scheduler()` in `config/wsgi.py`), mit Dateisperre
  (`fcntl.flock`), damit die drei Worker sich nicht gegenseitig überlappen.
  `dumpdata` statt `pg_dump`/Rohdatei bewusst gewählt: funktioniert
  identisch auf SQLite (Produktion) und Postgres (CI), kein zusätzliches
  Systembinary nötig.

  **Hinweis zum Repo-Stand:** `docs/DEPLOYMENT.md` im Code-Repo beschreibt an
  dieser Stelle noch den *alten* Plan (Cronjob nach EGS-Muster) — der ist
  durch dieses eingebaute Backup-Feature überholt und sollte bei
  Gelegenheit im Code-Repo aktualisiert werden.

---

## 6. API (für die PWA)

Keine Server-Side-Rendering-Logik für Inhalte — die PWA spricht ausschliesslich mit dieser JSON-API:

| Endpoint | Zweck |
|---|---|
| `GET /api/quests/<slug>/` | Quest-Infos + Liste der Posten (ohne Inhalte) |
| `GET /api/posten/<id>/` | Vollständiger Inhalt eines Postens (Fragen ohne Lösungen) |
| `POST /api/sessions/` | Neue Teilnehmer-Session starten (`quest_slug`, optional `team_name`) |
| `GET/PATCH /api/sessions/<session_key>/` | Session abrufen bzw. pausieren/fortsetzen/umbenennen |
| `POST /api/sessions/<session_key>/unlock/` | Posten freischalten via GPS (`lat`,`lng`) oder QR-Fallback (`qr_code`) |
| `POST /api/sessions/<session_key>/answer/` | Antwort abgeben, serverseitig bewertet, `total_points` zurückgegeben |

Keine Authentifizierung nötig — Teilnehmende sind ausschliesslich über den
unrateraren `session_key` (UUID) identifiziert, analog zu Actionbounds
„kein Account nötig"-Erlebnis.

---

## 7. Qualitätssicherung: Tests & CI/CD

### 7.1 Testsuite

87 Tests über sieben Dateien (`quest/tests.py`, `test_api.py`, `test_admin.py`,
`test_admin_nav.py`, `test_backup.py`, `test_email_settings.py`,
`test_invite.py`) — Datenmodell/Multi-Tenancy, vollständiger API-Ablauf
(Session, GPS-/QR-Unlock, alle fünf Fragetypen inkl. Punktevergabe, dass
korrekte Antworten nie über die API preisgegeben werden), Admin-Navigation,
Backup-Logik, E-Mail-Konfiguration, Einladungsflow. Lokal ausführen:
`python manage.py test`.

### 7.2 GitHub Actions (`.github/workflows/ci.yml`)

Läuft bei jedem Push/PR auf `main`: Checkout, Python 3.12, ein Postgres-16-
Service-Container, dann `manage.py check`, `manage.py makemigrations
--check --dry-run`, `manage.py test` — alles gegen Postgres, nicht gegen
die lokale SQLite-Entwicklungsdatenbank, damit Postgres-spezifisches
Verhalten (siehe unten) auch wirklich vor dem Deploy auffällt.

**Wichtige, bereits gemachte Erfahrung (Konvention für neue Singleton-Modelle):**
Ein früherer CI-Fehler entstand durch `EmailSettings.objects.get_or_create(pk=1)`.
Unter Postgres wachsen Auto-increment-Sequenzen auch über zurückgerollte
Test-Transaktionen hinweg weiter (Sequenzen sind nicht transaktional) — die
fest verdrahtete `pk=1`-Annahme legte dadurch fälschlich eine zweite Zeile
an, statt die tatsächlich existierende (z. B. `pk=5`) zu finden. Unter
SQLite lokal blieb der Fehler unsichtbar, weil Rowids dort nach Rollback
oft wieder bei 1 beginnen. **Fix-Pattern, verbindlich für jedes künftige
Singleton-Admin-Modell:** eine `get_solo()`-Klassenmethode
(`cls.objects.first()`, sonst `cls.objects.create()`) statt jeder
`pk=1`-Annahme — bereits so umgesetzt bei `EmailSettings` und `BackupSettings`.

### 7.3 Weitere Konventionen (aus wiederholten Bugs gelernt)

- **Django-Template-Kommentare** (`{# ... #}`) dürfen **nicht** mehrzeilig
  sein — der Tokenizer bricht dann ab und der rohe Kommentartext landet im
  gerenderten HTML. Mehrzeilige Kommentare müssen `{% comment %}...{% endcomment %}` verwenden.
- **CSS-Kaskade bei Admin-Templates:** Djangos eigene Templates (z. B.
  `admin/change_form.html`) erweitern unser `admin/base_site.html` und
  überschreiben denselben `{% block extrastyle %}` via `{{ block.super }}`
  — Djangos eigenes `forms.css` landet dadurch im HTML immer *nach*
  `admin_branding.css`. Bei gleicher Spezifität gewinnt die später
  geladene Regel; eigene Überschreibungen von Django-Admin-Defaults (z. B.
  Fieldset-Farben) brauchen deshalb `!important`, nicht nur höhere
  CSS-Spezifität.
- **Geteilte CSS-Variablen:** Django nutzt dieselbe `--header-bg`-Variable
  sowohl für die Kopfleiste als auch für jede Fieldset-Überschrift auf
  jeder Formularseite (`admin/css/forms.css`). Eine Branding-Farbe für die
  Kopfleiste braucht daher eine eigene, spezifischere Regel für
  Fieldset-Überschriften, sonst wird ungewollt jedes mehrteilige Formular
  im ganzen Admin mitgefärbt.

---

## 8. Betrieb & Deployment

### 8.1 Zielserver

Läuft auf demselben Eartheffect-VPS wie EGS/EnergyCheck/Piazza Circulaire
(`ubuntu@179.237.69.217`, siehe `Marflixx/egs-eartheffect`). Alle Apps dort
folgen demselben Muster: Python-venv + systemd-Service + eine gemeinsame
nginx-Instanz mit einem Server-Block pro Domain + Certbot. Kein Docker,
keine gemeinsame Datenbank — jede App unabhängig über ihr eigenes
systemd-Unit und ihre eigene SQLite-Datei.

- **Domain:** `go.circularquest.ch` (primär). `basel.circularquest.ch`
  leitet per 301 dorthin um (Domain-Umzug Juli 2026, bereits verteilte
  Links/QR-Codes/Poster funktionieren weiter).
- **App-Verzeichnis:** `/var/www/circularquest/circularquest/backend`
- **systemd-Service:** `circularquest`, Port `8010` (lokal, hinter nginx)

### 8.2 Update-Einzeiler (laufender Betrieb)

```bash
ssh -i ~/.ssh/egs-eartheffect.pem ubuntu@179.237.69.217 "cd /var/www/circularquest/circularquest && git pull origin main && cd backend && venv/bin/pip install -r requirements.txt -q && venv/bin/python manage.py migrate --noinput && venv/bin/python manage.py setup_groups && venv/bin/python manage.py collectstatic --noinput && sudo systemctl restart circularquest && git log -1 --oneline"
```

`setup_groups` läuft idempotent bei jedem Update mit, damit
Redaktionsrechte automatisch aktuell bleiben.

### 8.3 Alternativweg (dokumentiert, aktuell nicht im Einsatz)

Für den Fall eines Umzugs auf einen eigenen, dedizierten Server: Docker
Compose (Web + Postgres) + eigene nginx-Instanz + Let's-Encrypt-Overlay,
vorbereitet in `backend/Dockerfile`, `docker-compose.yml`,
`docker-compose.prod.yml`, `scripts/deploy.sh`. Produktions-Sicherheits-
einstellungen in `config/settings.py` greifen automatisch, sobald
`DJANGO_DEBUG=False` gesetzt ist.

### 8.4 SSH-Grundsatz

Server-Zugriffe erfolgen ausschliesslich über fertige, direkt einfügbare
SSH-Einzeiler (Schlüssel `~/.ssh/egs-eartheffect.pem`) — Martin führt diese
selbst aus, kein direkter Serverzugriff von aussen.

---

## 9. Inhaltlicher Aufbau: Content-Migration aus den Word-Unterlagen

Der technische Unterbau steht; der Schwerpunkt verschiebt sich jetzt auf
den **inhaltlichen Aufbau** — die realen Posten-Inhalte aus den
Actionbound-Word-Unterlagen ins System zu bringen — sowie auf **gezielte
Funktionsanpassungen**, die sich dabei als nötig zeigen.

### 9.1 Quelldokumente

- **`Inhalte_Actionbound_Basel_Juli 2026.docx`** (Stand Juli 2026,
  **aktuelle Fassung**) — die vollständige Redaktionstabelle (1 Tabelle,
  285 Zeilen, Spalten „Art des Inhalts" / „Inhalt" / „P" / „Notizen"),
  gegliedert in POSTEN- bzw. CLUSTER-Abschnitte. Ersetzt die Juni-Fassung
  unten — die Redaktion hat seither deutlich weitergearbeitet (siehe 9.2).
  Als Markdown direkt auf GitHub lesbar (automatisch erzeugter Mirror, die
  docx bleibt die Quelle der Wahrheit für die Redaktion):
  [`docs/content/inhalte_actionbound_basel_juli_2026.md`](https://github.com/Marflixx/Circular-Quest-Basel/blob/main/docs/content/inhalte_actionbound_basel_juli_2026.md)
  im Code-Repo.
- **`Inhalte_Actionbound_Basel_Juni 2026.docx`** (Vorgängerversion,
  Stand Juni 2026, 303 Zeilen) — nur noch als Referenz für den
  Fortschrittsvergleich relevant, nicht mehr die massgebende Quelle.
- **`Beispielposten_App Format.docx`** — ein Auszug der Juni-Tabelle (die
  ersten 79 Zeilen: Posten 0 + Franck-Areal-Intro + Beginn Posten 1). Das
  ist genau der Ausschnitt, der bereits als `seed_demo_quest` ins System
  migriert wurde.

### 9.2 Inventar: Stand pro Posten (Juli-Fassung)

Gegenüber der Juni-Fassung hat die Redaktion deutlich nachgeliefert: Elys,
Weinlager und Lysbüchel/LysP8 waren im Juni-Dokument nur leere
Platzhalter-Überschriften — jetzt vollständig ausgearbeitet. Warteck Areal
war 72 leere Zeilen lang reserviert — jetzt ein kompakter, fertiger Posten.
Dazu drei komplett neue Abschnitte: der Fahrt-Impuls St. Johann–Wettstein,
ein zusätzlicher Posten „Bravo Ricky" und ein Outro/Abschluss. Ausserdem
wurden Elys und Weinlager in der Nummerierung vertauscht (neu: Weinlager =
Posten 6, Elys = Posten 7) und Lysbüchel in zwei eigenständige Abschnitte
aufgeteilt (Posten 8 „LysP8" + eigener „Circular Spot: Lysbüchel Süd").

| Abschnitt | Blöcke (Juli) | Änderung ggü. Juni | Im System (`seed_demo_quest`) |
|---|---|---|---|
| Posten 0 – Einführung & Begrüssung | 4 | unverändert | ✅ migriert |
| Franck Areal – Intro | 9 | unverändert | ✅ migriert |
| Posten 1 – Warum müssen wir anders bauen? | 33 | +1 Block | ⚠️ nur 3 von 33 Blöcken |
| Posten 2 – Wie können wir anders bauen? | 5 | unverändert | ❌ nicht migriert |
| Hebel 1/2/3 (Gebäude/Materialien/Planung) | 4+4+3 | unverändert | ❌ nicht migriert |
| Franck Areal – Hebel-Zusammenfassung | 7 | unverändert | ❌ nicht migriert |
| Posten 3 – Pförtnerhaus | 12 | unverändert | ❌ nicht migriert |
| Posten 4 – Kreislaufhaus | 18 | unverändert | ❌ nicht migriert |
| Posten 5 – Unterdessen K024 | 10 | unverändert | ❌ nicht migriert |
| Circular Spot 1 – Fischergalgen | 7 | unverändert | ❌ nicht migriert |
| Circular Spot 2 – Novartis Campus | 6 | +2 Blöcke | ❌ nicht migriert |
| Audio-Impuls unterwegs – Voltahalle | 3 | unverändert | ❌ nicht migriert |
| Übergang (Wegstück) | 2 | unverändert | ❌ nicht migriert |
| **Posten 6 – Weinlager** (vorher Posten 7, leer) | 13 | ✅ **neu ausgearbeitet** | ❌ nicht migriert |
| **Posten 7 – Elys** (vorher Posten 6, leer) | 17 | ✅ **neu ausgearbeitet** | ❌ nicht migriert |
| **Posten 8 – LysP8** (vorher Teil von „Lysbüchel/LysP8", leer) | 14 | ✅ **neu ausgearbeitet** | ❌ nicht migriert |
| **Circular Spot: Lysbüchel Süd** (neu abgespalten) | 5 | ✅ **neu ausgearbeitet** | ❌ nicht migriert |
| **Fahrt-Impuls: St. Johann–Wettstein** | 5 | ✅ **komplett neuer Abschnitt** | ❌ nicht migriert |
| Posten 9 – Bau 92 (Roche, „Version Selina") | 34 | +3 Blöcke | ❌ nicht migriert |
| **Posten 10 – Warteck Areal** (vorher 72 leere Zeilen) | 11 | ✅ **neu ausgearbeitet** (kompakt statt Platzhalter) | ❌ nicht migriert |
| **Posten 11 – Bravo Ricky** | 10 | ✅ **komplett neuer Posten** | ❌ nicht migriert |
| **Outro: Abschluss** | 5 | ✅ **komplett neuer Abschnitt** | ❌ nicht migriert |

Cluster 1/2/3 bleiben reine Gliederungs-Überschriften ohne eigenen Inhalt —
das ist beabsichtigt, sie gruppieren nur die jeweils folgenden Posten.

**Kurzfassung:** die Redaktion hat den Content-Aufbau seit der Juni-Fassung
im Alleingang praktisch abgeschlossen — **alle** vorgesehenen Posten
inklusive der drei neuen Abschnitte sind jetzt redaktionell fertig
ausgearbeitet. Es gibt keine leeren Platzhalter-Posten mehr. Der
Engpass liegt jetzt eindeutig auf der technischen Seite: von rund 260
inhaltlichen Blöcken sind erst 11 (Posten 0 + Franck-Intro + 3 von 33
Blöcken aus Posten 1) tatsächlich im System. Der „inhaltliche Aufbau" ist
damit primär noch ein **Migrations-**, nicht mehr ein Redaktionsprojekt.

### 9.3 Content-Typ-Mapping: Word → Datenmodell

Die rohen „Art des Inhalts"-Bezeichnungen im Word sind uneinheitlicher
(Tippfehler, Synonyme, Emoji-Präfixe) als die bereits im System vorhandenen
`ContentBlock.BlockType`-Werte. Ein Migrationsskript braucht dafür eine
feste Mapping-Tabelle (aktualisiert anhand der Juli-Fassung, inkl. neu
hinzugekommener Varianten aus Weinlager/Elys/LysP8/Bravo Ricky/Outro):

| Wert im Word | Ziel im System | Bemerkung |
|---|---|---|
| Information / Informationen / Information (Exkurs/Vertiefung/Abschluss/Cliffhanger) | `INFO` | Varianten alle auf `INFO`, Klammerzusatz landet in `notes` |
| MC-Frage / Quiz – Single Choice | `QUESTION` + `Question.MULTIPLE_CHOICE` | Single-Choice = MC mit genau einer `is_correct: true`-Option, kein neuer Typ nötig |
| Übergang | `TRANSITION` | — |
| Challenge / Aufgabe | `PHOTO_CHALLENGE` | Fallunterscheidung: Foto vs. reine Vor-Ort-Aufgabe |
| Booklet | `BOOKLET_REFERENCE` | — |
| Bild / Bild / Grafik / Titelbild | `IMAGE` | „Titelbild" ggf. mit eigenem `title`-Flag statt neuem Typ |
| Beobachtungsaufgabe / Beobachten / Beobachtung | `OBSERVATION` | Schreibvarianten vereinheitlichen |
| Gamification | `RIDDLE` | — |
| Video / Video (optional) / Information / Video (optional) | `VIDEO` | „(optional)" als Redaktionshinweis, nicht als Datenfeld nötig |
| Plakat / Plakat Mockup / Plakat/Gebäude-Pass / Gebäude-Pass (App und Plakat) / Gebäudepass | `BUILDING_PASS` | Nutzt die bestehenden Gebäude-Pass-Felder auf `Posten`; „Plakat Mockup" ist teils nur eine Produktionsnotiz ohne Inhaltstext (z. B. QR-Code-Platzierung) |
| Schätzfrage | `QUESTION` + `Question.ESTIMATE` | — |
| Sortierfrage | `QUESTION` + `Question.SORT` | — |
| Richtig-Falsch | `QUESTION` + `Question.TRUE_FALSE` | — |
| Zuordnungsfrage | `QUESTION` + `Question.MATCH` | — |
| Fussweg / Fahrt / 🎧 Audio / Audio (2–4 Min.) | `TRANSITION` / `AUDIO` | „Fussweg" (zu Fuss) und „Fahrt" (ÖV) haben oft eine Zeitangabe im Text — aktuell nur Freitext, keine strukturierte Minuten-Angabe |
| Reflexionsfrage / Transferfrage (Berufsbezug) | **kein passender Typ** | Offene Frage ohne „richtig/falsch", teils bewusst mit Berufsbezug (Zielgruppe u. a. Zeichner:innen EFZ) — neuer `BlockType` oder als `INFO` mit Freitext-Charakter behandeln (Entscheidung nötig, siehe 9.4) |
| Austausch (zu zweit) | **kein passender Typ** | Paarweise Reflexionsaktivität ohne Bewertung, z. B. während einer Fahrt — am nächsten an `OBSERVATION`, aber nicht wirklich deckungsgleich (siehe 9.4) |
| Fun Fact / Fun Fact (optional) / 💡 Schon gewusst? | **kein passender Typ** | Aktuell nächstliegend `INFO`, verliert aber die "Fun Fact"-Kennzeichnung fürs UI (eigenes Badge/Icon wäre denkbar) |
| Hinweis (optional, unbewertet) | `INFO` | Braucht keine Modelländerung — `notes` = "optional, unbewertet" reicht, da Punkte ohnehin nur bei `QUESTION`-Blöcken vergeben werden |
| Link | **kein passender Typ** | Externer Link (z. B. zu Roche-Website) — bisher kein `BlockType` dafür vorgesehen |

### 9.4 Vorschläge für Funktionsanpassungen (zur Diskussion, noch nicht umgesetzt)

Aus dem Mapping ergeben sich konkrete, aber überschaubare Erweiterungen —
keine davon verändert das Grundmodell, alle sind zusätzliche
`BlockType`-Werte bzw. kleine Feldergänzungen:

- **Neuer `BlockType.LINK`** für externe Weiterverweise (z. B. „Mehr über
  Roche und Bau 92 erfahren").
- **Neuer `BlockType.REFLECTION_QUESTION`** für offene Reflexions- und
  Transferfragen (inkl. der berufsbezogenen Varianten bei Bau 92/Bravo
  Ricky), oder bewusste Entscheidung, diese als `INFO` mit einer
  speziellen Kennzeichnung zu führen (keine automatische Bewertung nötig,
  da es keine „richtige" Antwort gibt).
- **„Austausch (zu zweit)" klären:** eigener `BlockType` (z. B.
  `PAIR_EXCHANGE`) oder Spezialfall von `REFLECTION_QUESTION`/`OBSERVATION`
  — kommt bislang nur einmal vor (Fahrt-Impuls), Entscheidung kann warten,
  bis klar ist, ob dieses Format öfter wiederverwendet wird.
- **„Fun Fact" als eigenes, leichtes Label** auf bestehenden `INFO`-Blöcken
  (z. B. ein `is_fun_fact`-Flag statt neuem Typ), damit die App dafür ein
  eigenes Icon/Badge zeigen kann, ohne die Content-Type-Logik aufzublähen.
- **`TRANSITION` um eine optionale strukturierte Zeit-/Verkehrsmittel-Angabe
  ergänzen** (z. B. `duration_minutes`, `mode` = zu Fuss/ÖV), statt die
  Angabe nur im Fliesstext zu führen — betrifft jetzt sowohl „Fussweg" als
  auch den neuen „Fahrt"-Typ (Tram/ÖV) und wäre die Grundlage für eine
  spätere Routen-/Zeitplanungsanzeige in der App.
- **Optionales „Cluster"/Gruppierungs-Feld auf `Posten`** (rein editorial,
  z. B. `cluster_label`), um Stationen wie Weinlager/Elys/LysP8 (Cluster 2)
  bzw. Bau 92/Warteck/Bravo Ricky (Cluster 3) als zusammengehörige Gruppe
  darzustellen — im Word-Dokument bereits als Gliederung angelegt, im
  Datenmodell aber noch nicht abgebildet.
- **Outro/Abschluss als eigener Quest-Baustein:** bisher endet eine Quest
  einfach beim letzten Posten; jetzt gibt es redaktionell einen expliziten
  Abschluss-Posten mit Rückblick-Reflexionsfrage und optionalem
  Outro-Video — technisch reicht dafür vermutlich ein ganz normaler
  `Posten` als letzter in der Reihenfolge, ohne Modelländerung.

Diese Punkte sind Vorschläge, keine Entscheidungen — vor der Umsetzung mit
der Redaktion abstimmen, welche davon tatsächlich gebraucht werden.

### 9.5 Empfohlenes Vorgehen

1. Mapping-Tabelle (9.3) und offene Punkte (9.4) mit der Redaktion
   besprechen und verbindlich festlegen.
2. Nötige Modellerweiterungen umsetzen (kleine Migration, siehe 9.4).
3. Generisches Migrationsskript bauen (`python-docx`, liest die Tabelle
   zeilenweise, erkennt POSTEN-Header über die „alle vier Zellen gleich"-
   Struktur, wendet das Mapping an) — ersetzt das bisherige, rein manuelle
   `seed_demo_quest`-Vorgehen. Wichtig: robust gegenüber weiteren
   Änderungen der Redaktion machen (Juni → Juli hat bereits Nummerierung,
   Abschnittsgrenzen und Spaltenwerte verändert).
4. Alle redaktionell fertigen Posten migrieren (nach aktuellem Stand:
   praktisch der gesamte Quest, Posten 1 Rest bis Outro) — reine
   Migrationsarbeit, keine zusätzliche Textarbeit mehr nötig.
5. GPS-Koordinaten und QR-Fallback-Codes für alle Posten vor Ort erheben
   (aktuell nur für den Franck-Areal-Pilotposten vorhanden, und auch dort
   laut `backend/README.md` noch ein grober Platzhalter) — das ist jetzt
   der grössere verbleibende Vor-Ort-Aufwand, nicht mehr die Textredaktion.
6. Medien (Fotos, Videos, Audio-Impulse) einsammeln — im Word-Dokument
   grösstenteils nur als Beschreibung/Platzhalter vermerkt („geplant/
   anzufragen", „wird separat aufgegleist").

---

## 10. Roadmap / Projektphasen

| Phase | Inhalt | Stand |
|---|---|---|
| Phase 0 – Grundlagen | Server, Domain, Repository, Basis-Stack | ✅ erledigt |
| Phase 1 – MVP | Datenmodell, Admin, PWA-Grundgerüst, GPS-Check, ein Posten end-to-end (Franck Areal) | ✅ erledigt, weit überschritten (CMS, Invite-Flow, Backup zusätzlich) |
| Phase 2 – Vollständige Inhalte | Alle Fragetypen (✅), Migrationsskript für Word-Inhalte (offen), alle Posten befüllt (offen), Erkundungsheft-Verweise | teilweise |
| Phase 3 – Pilot vor Ort | Test mit echten Teilnehmenden, GPS-Feinschliff, QR-Fallback, Fehlerkorrektur | offen |
| Phase 4 – Rollout | Launch, Auswertungs-Dashboard, schrittweise Ablösung von Actionbound | offen |
| Phase 5 – Skalierung (optional) | Weitere Städte/Areale, Mehrsprachigkeit, weitere Touren auf derselben Plattform | offen, Datenmodell ist bereits dafür vorbereitet |

**Nächste konkrete Schritte (Vorschlag):**

- Inhaltlicher Aufbau gemäss Abschnitt 9 (Mapping abstimmen,
  Migrationsskript bauen, redaktionell fertigen Content migrieren, fehlende
  Inhalte für Elys/Weinlager/Lysbüchel/Warteck erarbeiten).
- Echten Foto-Upload-Endpunkt für Foto-Challenges ergänzen.
- GPS-Koordinaten des Franck Areal vor Ort verifizieren (aktuell Platzhalter).
- UI/UX-Feinschliff des PWA-Frontends.
- `docs/DEPLOYMENT.md` im Code-Repo bezüglich Backup aktualisieren (siehe
  Hinweis in Abschnitt 5.4).

---

## 11. Risiken & offene Entscheidungspunkte

- **Eigenverantwortung für den Betrieb:** Wartung, Updates und Sicherheit
  liegen vollständig beim eigenen Team statt bei einem SaaS-Anbieter.
- **Design/UX-Aufwand:** Actionbound liefert eine fertige, getestete
  Oberfläche; hier muss das Interface selbst gestaltet und getestet werden.
- **GPS-Genauigkeit:** in dicht bebauten Arealen/Innenhöfen ungenau — der
  QR-Fallback ist deshalb fester Bestandteil, nicht Nebensache.
- **Übergangsphase:** Parallelbetrieb mit Actionbound während der Migration
  empfohlen, um den laufenden Tourbetrieb nicht zu gefährden.
- **Langfristiger Unterhalt:** wer betreut die Plattform nach dem Launch
  inhaltlich und technisch?

---

## Änderungsprotokoll

| Datum | Änderung |
|---|---|
| 03.08.2026 | Dokument neu angelegt, konsolidiert aus `README.md`, `docs/ARCHITECTURE.md`, `docs/DEPLOYMENT.md` und `backend/README.md` des Code-Repos plus aktuellem Stand (Backup-Feature, CI-Fix, Admin-Branding). |
| 03.08.2026 | Abschnitt 9 „Inhaltlicher Aufbau" ergänzt: Auswertung der Quelldokumente `Inhalte_Actionbound_Basel_Juni 2026.docx` und `Beispielposten_App Format.docx` (Inventar pro Posten, Content-Typ-Mapping, Vorschläge für Funktionsanpassungen, empfohlenes Vorgehen). Roadmap/Risiken-Abschnitte entsprechend zu 10/11 verschoben. |
| 03.08.2026 | Abschnitt 9 auf Basis von `Inhalte_Actionbound_Basel_Juli 2026.docx` aktualisiert: Redaktion hat Elys, Weinlager, LysP8, Warteck Areal fertig ausgearbeitet und drei neue Abschnitte ergänzt (Fahrt-Impuls St. Johann–Wettstein, Posten 11 „Bravo Ricky", Outro/Abschluss) — Content ist jetzt praktisch vollständig, Engpass liegt bei der technischen Migration. Mapping- und Funktionsanpassungs-Tabellen entsprechend erweitert. |

*(Neue Einträge hier ergänzen, wenn sich Konzept, Architektur oder Betrieb ändern.)*
