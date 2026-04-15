# index.php – Logik & Struktur (Detaillierte Erklärung)

Diese Datei erklärt **nur** die `index.php` und beschreibt, wie Request-Handling, Datenfluss und UI-Struktur im Projekt funktionieren.

---

## 1) Grundidee der `index.php`

`index.php` ist der zentrale Einstiegspunkt der Anwendung und übernimmt:

1. Session-Start und Laden der DB-Helfer.
2. Verarbeitung aller Formular-Aktionen (POST).
3. Laden aller Daten für die Ansicht (GET).
4. Rendern von Login-Ansicht oder Dashboard-Ansicht.

Damit ist die Datei gleichzeitig **Controller + View**.

---

## 2) Initialisierung

Am Anfang der Datei passiert:

- `session_start()`
- `require_once 'config.php'`
- `$pdo = db()` für DB-Zugriff
- Vorbereiten von Arrays für `errors` und `messages`
- Bestimmen des aktiven Reiters über `$_GET['tab']`

Diese Werte steuern den gesamten Ablauf und die spätere Anzeige.

---

## 3) Authentifizierung (Login/Registrierung)

### Aktion `login`
- Liest `username` und `password`.
- Prüft, ob der Arzt in Tabelle `arzt` existiert.
- Wenn Passwort korrekt:
  - Session wird gesetzt: `$_SESSION['arzt'] = ['id', 'name']`
  - Redirect auf `index.php`
- Wenn Arzt nicht existiert:
  - Es wird ein Registrierungs-Prompt vorbereitet (`$pendingRegistration`).

### Aktion `register_doctor`
- Erstellt den Arzt in der Tabelle `arzt`.
- Danach wird eine Erfolgsmeldung gezeigt und ein normaler Login erwartet.

> Wichtig: Passwortspeicherung ist aktuell noch Klartext (wie in der aktuellen Projektlogik).

---

## 4) Aktionen im eingeloggten Zustand

Sobald ein Arzt in der Session existiert, verarbeitet `index.php` zusätzlich folgende Aktionen:

### `add_service`
- Fügt eine neue Leistung in Tabelle `leistung` ein.
- Verhindert ungültige Eingaben (leer/negativer Preis).

### `add_patient`
Transaktionaler Ablauf:
1. Ort suchen (PLZ + Ort), ggf. neu erzeugen.
2. Prüfen, ob Patient mit gleichen Stammdaten bereits existiert.
3. Falls nicht vorhanden: neuen Patienten anlegen.
4. Leistungseintrag in `patient_leistung` anlegen (`erledigt = 0`).
5. Commit / Rollback bei Fehler.

Damit ist sichergestellt, dass **ein Patient mehrere Leistungen** bekommen kann.

### `mark_done`
- Markiert alle offenen Leistungen (`patient_leistung.erledigt = 0`) des aktuellen Arztes für den gewählten Patienten als erledigt.

### `transfer_patient` / `transfer_group`
- Verschiebt offene Leistungen auf einen anderen Arzt (`patient_leistung.arzt_id`).
- Nur offene Leistungen werden übertragen.

---

## 5) Umschalten zwischen Login und Dashboard

Nach der Aktionsverarbeitung prüft die Datei:

- **Nicht eingeloggt** -> Rendert Login-Seite.
- **Eingeloggt** -> Lädt Dashboard-Daten und rendert die Reiter.

Das sorgt für klare Trennung ohne zweite Entry-Datei.

---

## 6) Datenladen für Dashboard

Vor dem HTML-Teil werden die benötigten Daten geladen:

- Ärzteliste (`arzt`) für Transfer-Select.
- Leistungsliste (`leistung`) für Patient-Anlage.
- Offene Patienten (über `EXISTS` in `patient_leistung` mit `erledigt = 0`).
- Suchergebnisse für `Suche & Transfer`.
- Leistungsdetails je Patient inkl. Status, Preis, Kostenträger und behandelndem Arzt.
- Rechnungsdaten:
  - Alle Patienten (im Reiter „Rechnung (Alle Patienten)“)
  - Einzelpatient (innerhalb „Suche & Transfer“)

---

## 7) Aufbau der UI-Reiter

Im Dashboard werden vier Tabs gerendert:

1. **Offene Patienten**
   - Liste offener Patienten
   - Button: „Erledigt abhaken"

2. **Patient hinzufügen**
   - Formular für Stammdaten
   - Auswahl Leistung + Kostenträger
   - Extra-Form für neue Leistung

3. **Suche & Transfer**
   - Namenssuche
   - Patientenkarte mit Leistungsdetails
   - Transfer offener Leistungen
   - Button „Rechnung Patient“
   - Einzelrechnung mit Summen

4. **Rechnung (Alle Patienten)**
   - Gesamtliste aller Leistungen des eingeloggten Arztes
   - Summen getrennt nach Selbstzahler/Krankenkasse + Gesamt

Der aktive Reiter wird über `?tab=...` gesteuert.

---

## 8) Rechnungslogik in der `index.php`

### Gesamtrechnung
- Trigger: `tab=rechnung`
- Query: alle Leistungen des aktuellen Arztes aus `patient_leistung` + Join auf Patient/Leistung
- Ausgabe als Tabelle mit Summen:
  - Selbstzahler
  - Krankenkasse
  - Gesamtbetrag

### Einzelrechnung
- Trigger: `invoice_patient_id=<id>` im Tab `suche`
- Query: alle Leistungen des Patienten inkl. Arztname
- Ausgabe ebenfalls mit getrennten Summen nach Kostenträger

---

## 9) Fehler- und Statusmeldungen

`$errors` und `$messages` werden während der Aktionsermittlung befüllt und zentral im UI angezeigt.
So bekommt der Nutzer direktes Feedback bei:
- Validierungsfehlern
- DB-Fehlern
- Erfolgreichen Aktionen (Speichern, Transfer, Erledigt)

---

## 10) Warum die Struktur aktuell so ist

Die Datei ist absichtlich kompakt gehalten (ein Einstiegspunkt), um schnell iterieren zu können.
Für größere Projekte wäre als nächster Schritt sinnvoll:

- Trennung in Controller/Service/View
- eigene Klassen für Repository/DB-Zugriff
- saubere Routing-Struktur
- Tests pro Action

---

## 11) Kurz-Zusammenfassung

`index.php` steuert das komplette Verhalten der App:

- Authentifizierung
- Patienten-/Leistungs-Workflows
- Erledigt-Status pro Leistung
- Transfer
- Einzel- und Sammelrechnung
- Rendering aller Reiter

Damit ist sie der funktionale Kern der Anwendung.
