# Migration von Seafile zu kDrive – Anleitung für Teammitglieder

*eartheffect — Umstieg der gemeinsamen Ablage von Seafile Cloud auf Infomaniak kDrive*

Erstellt für: Team eartheffect · Stand: 1. August 2026

## 1. Worum geht es

Wir stellen unsere gemeinsame Dateiablage von Seafile Cloud auf Infomaniak kDrive um. Die Migration der Dateien läuft zentral über einen Sync direkt vom Seafile-Server zu kDrive — ihr müsst dafür **keine Dateien selbst hochladen oder manuell verschieben**.

**Ablauf:** Der Sync läuft am Sonntag. Alle Seafile-Konten wurden bereits am **Samstag, 19:00 Uhr** getrennt — es kann seither nichts mehr mit dem Server synchronisiert werden. Die Migration gilt ab **Montagmorgen** als abgeschlossen, dann steht kDrive bereit.

## 2. Wichtig: Prüft auf Änderungen nach Samstag 19:00 Uhr

Weil eure Seafile-Konten seit Samstag, 19:00 Uhr getrennt sind, wurde **alles, was danach noch lokal bearbeitet oder neu erstellt wurde, nicht mehr auf den Server übertragen** — und ist dadurch auch nicht in der neuen kDrive-Ablage enthalten.

Schaut deshalb in eurem lokalen Seafile-Ordner nach, ob dort Dateien mit einem Änderungsdatum **ab Samstag 19:00 Uhr** liegen (z. B. im Explorer/Finder nach Datum sortieren). Falls ja: diese Dateien müsst ihr am Montag von Hand in den entsprechenden Unterordner der neuen kDrive-Ablage kopieren (Schritt 4 unten, nachdem kDrive eingerichtet ist).

Das betrifft voraussichtlich nur sehr wenige, falls überhaupt Dateien — am Wochenende hat niemand gearbeitet.

## 3. Ab Montagmorgen: Umstieg auf kDrive

1. **Seafile-Desktop-Client deinstallieren oder Ordner-Verknüpfung entfernen.** Die Konten sind ohnehin bereits getrennt.
2. **kDrive-Desktop-App installieren:** [Download-Link kDrive](https://kdrive.infomaniakgroup.com/app/share/100338/2e07e9ef-b679-4058-8363-b8c324e460e2), mit eurem Infomaniak-Konto anmelden. Login: eure Infomaniak-E-Mail-Adresse + dasselbe Kontopasswort, mit dem ihr euch auch bei eurer Infomaniak-Webmail/im Manager anmeldet — es braucht kein separates kDrive-Passwort. (Falls ihr in eurem E-Mail-Programm ein eigenes IMAP/SMTP-Passwort eingerichtet habt: das gilt nur dort, nicht für kDrive.)
3. **Lite Sync aktivieren (Windows/macOS):** Dateien werden lokal nur als Platzhalter angezeigt und erst beim Öffnen heruntergeladen — spart Speicherplatz, da ihr nicht mehr die komplette Ablage lokal vorhalten müsst.
4. Ordner **"Common documents"** zur Synchronisation auswählen.
5. **Falls ihr in Schritt 2 Dateien mit Änderungsdatum ab Sa. 19:00 Uhr gefunden habt:** diese jetzt manuell aus dem alten lokalen Seafile-Ordner in den entsprechenden Unterordner von "Common documents" in kDrive kopieren.
6. **Erst danach den alten lokalen Seafile-Ordner vollständig löschen — und anschliessend zwingend den Papierkorb leeren.** Solange der Papierkorb nicht geleert ist, belegen die gelöschten Dateien weiterhin Speicherplatz. Nur mit geleertem Papierkorb wird der Speicherplatz tatsächlich wieder frei.

## 4. Bei Problemen

Meldet euch bei Martin ([raeber@eartheffect.ch](mailto:raeber@eartheffect.ch)), falls:

- nach dem Umstieg Dateien in kDrive fehlen oder nicht aktuell aussehen,
- ihr unsicher seid, ob eine Datei nach Sa. 19:00 Uhr bearbeitet wurde,
- die kDrive-App sich nicht mit dem Infomaniak-Konto verbinden lässt.
