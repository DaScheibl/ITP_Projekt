# Praxis-Dashboard (PHP + MySQL)

## Start
```bash
php -S localhost:8000
```
Dann `http://localhost:8000` öffnen.

## Datenbankanbindung
Die Anwendung nutzt MySQL via PDO mit folgender Verbindung:

```php
new PDO('mysql:host=bszw.ddns.net;dbname=wit12a_ITP_StiefScheibl;charset=utf8', 'wit12a', 'geheim');
```

Beim ersten Start werden die Tabellen automatisch angelegt.

## Funktionen
- Login für Ärzte.
- Wenn Arzt nicht existiert: optional direkt speichern.
- Dashboard mit Tabs:
  - Offene Patienten abhaken (nur `erledigt=1`, kein Delete in DB)
  - Patient inkl. Leistung + Kostenträger anlegen
  - Leistungen verwalten
  - Suche erledigte/offene Patienten + Transfer (nur solange nicht erledigt)
