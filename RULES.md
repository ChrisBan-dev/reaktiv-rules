# Entwicklungsregeln — Reaktiv GmbH

> Universelle Regeln für alle Projekte. Projektspezifische Details stehen in der jeweiligen CLAUDE.md.
> Diese Datei wird versioniert, deployed und ist die verbindliche Referenz für alle Code-Generierung.

---

## Master-Checklist (vor jeder Ausgabe zwingend)

> Konsolidierte Pruefpflicht aus allen Abschnitten. Kein "fertig" ohne gruene Liste.

### DSGVO (→ Details: Abschnitt "DSGVO-Regeln")
- [ ] Keine PII in Logs, Fehlermeldungen oder Konsolenausgaben?
- [ ] Keine externen Dienste ohne AVV eingebunden?
- [ ] Keine Daten ausserhalb der EU gespeichert oder uebertragen?
- [ ] PII-Felder verschluesselt (at rest)?

### IT-Sicherheit (→ Details: Abschnitt "IT-Sicherheit")
- [ ] Alle Eingaben validiert, alle Outputs encoded (XSS/Injection)?
- [ ] SQL: ausschliesslich Prepared Statements (kein String-Concat)?
- [ ] Keine Secrets im Code, Git oder Logs?
- [ ] Alle neuen Endpoints authentifiziert und rate-limited?
- [ ] Keine Fehlermeldungen mit internen Details (Stack-Traces, Pfade)?
- [ ] Principle of Least Privilege eingehalten?

### Referenzen & Konsistenz (→ Details: Abschnitt "Verifikationspflicht")
- [ ] Alle referenzierten Dateien, Module, Funktionen existieren tatsaechlich?
- [ ] Import-Pfade gegen reale Dateistruktur abgeglichen?
- [ ] Neue Pakete in package.json, neue Env-Vars in .env.example dokumentiert?
- [ ] Code konsistent mit bestehenden Projekt-Konventionen?

### UI / Corporate Design (→ Details: Abschnitt "Corporate Design")
- [ ] Ausschliesslich Farben aus der Reaktiv-Palette?
- [ ] Schriftart Helvetica Neue (kein Phenomena, kein System-Font ohne Fallback)?
- [ ] Icons: Outlines (keine Solid-Icons), keine Farbverlaeufe?
- [ ] Farbflaechen full-width (keine freistehenden farbigen Boxen)?

---

## DSGVO-Regeln (verbindlich für alle generierten Code-Aenderungen)

### Datenspeicherung & Uebertragung
- Keine personenbezogenen Daten (PII) in Logs: keine Namen, E-Mail-Adressen, Telefonnummern, Patienten-IDs oder IP-Adressen
- Datenbankfelder mit PII muessen verschluesselt gespeichert werden (at rest)
- Datenuebertragung ausschliesslich ueber HTTPS/TLS (nie HTTP)
- Datenspeicherung nur auf Servern innerhalb der EU

### Externe Dienste & APIs
- Keine Integration externer Dienste ohne bestehenden Auftragsverarbeitungsvertrag (AVV)
- Keine US-Cloud-Dienste ohne gepruefte Standardvertragsklauseln (SCC)
- Keine Tracking- oder Analytics-Dienste ohne explizite Einwilligung (kein Google Analytics, kein Hotjar etc.)

### Authentifizierung & Zugriff
- Keine Speicherung von Passwoertern im Klartext (nur Hashes, z.B. bcrypt/argon2)
- Session-Tokens duerfen keine PII enthalten
- Zugriffsprotokolle duerfen nur technische IDs enthalten, keine Klarnamen

### Datenweitergabe
- PII niemals in URL-Parametern oder Query-Strings uebergeben
- PII niemals in Fehlermeldungen oder Error-Responses zurueckgeben
- API-Responses nur mit den minimal notwendigen Datenfeldern (Data Minimization)

### Loeschfristen
- Keinen Code schreiben, der Daten dauerhaft ohne Loeschmöglichkeit speichert
- Alle neuen Datenbankmodelle muessen ein `created_at`-Feld enthalten (Vorbereitung fuer Loeschlogik)

### DSGVO-Pruefpflicht
→ Siehe **Master-Checklist** am Anfang dieser Datei (Abschnitt "DSGVO").

---

## IT-Sicherheit (verbindlich fuer alle generierten Code-Aenderungen)

### Authentifizierung & Session-Management
- Passwoerter ausschliesslich mit bcrypt (cost >= 12) oder argon2id hashen
- Multi-Faktor-Authentifizierung (MFA) fuer alle Admin-Zugaenge zwingend
- Session-Tokens: kryptografisch zufaellig, mind. 128 Bit Entropie
- Session-Timeout: max. 30 Min. Inaktivitaet, absolutes Maximum 8h
- Nach Login: Session-ID neu generieren (Session Fixation verhindern)
- Logout: Session serverseitig invalidieren, nicht nur Cookie loeschen
- Fehlgeschlagene Logins: nach 5 Versuchen Lockout oder CAPTCHA
- Keine Credentials in URLs, Logs, Git-Commits oder Kommentaren

### Eingabevalidierung & Output-Encoding
- Alle Nutzereingaben als untrusted behandeln — immer validieren und sanitizen
- SQL: ausschliesslich Prepared Statements / Parameterized Queries (kein String-Concat)
- HTML-Output: immer escapen (XSS-Schutz) — kein rohes innerHTML mit User-Input
- Datei-Uploads: Typ per Magic Bytes pruefen (nicht nur Dateiendung), in isoliertem Speicher ablegen, niemals direkt ausfuehren
- Redirects: nur auf Whitelist-URLs, nie auf User-Input-basierte URLs

### API & Kommunikation
- Alle Endpoints authentifizieren und autorisieren — kein "Security by Obscurity"
- Rate Limiting auf allen oeffentlichen Endpoints (Login, Registrierung, API)
- CORS: nur explizit erlaubte Origins, kein Wildcard (*) in Produktion
- Security-Header auf allen Responses:
  - `Strict-Transport-Security` (HSTS, min. 1 Jahr)
  - `Content-Security-Policy` (CSP)
  - `X-Frame-Options: DENY`
  - `X-Content-Type-Options: nosniff`
  - `Referrer-Policy: strict-origin-when-cross-origin`
- Keine sensitiven Daten in HTTP-Responses cachen (`Cache-Control: no-store`)

### Secrets & Konfiguration
- Keine Secrets, API-Keys oder Passwoerter im Quellcode oder Git
- Secrets ausschliesslich ueber Umgebungsvariablen oder Secret Manager
- `.env`-Dateien niemals committen — in `.gitignore` aufnehmen
- Verschiedene Credentials fuer Dev / Staging / Produktion
- Regelmaessige Rotation von API-Keys und Tokens einplanen

### Zugriffsrechte & Architektur
- Principle of Least Privilege: jeder User/Service bekommt nur die minimal notwendigen Rechte
- Datenbanknutzer fuer die App hat kein DROP/ALTER — nur SELECT/INSERT/UPDATE/DELETE
- Admin-Funktionen von regulaeren User-Funktionen architektonisch trennen
- Serverseitige Autorisierung immer — clientseitige Pruefungen sind nur UX, kein Schutz

### Abhaengigkeiten & Code
- Keine bekannt vulnerablen Bibliotheken einsetzen (CVE-Stand pruefen)
- Dependencies regelmaessig aktualisieren — kein "set and forget"
- Fehler-/Exceptionmeldungen nach aussen generisch halten (kein Stack-Trace, keine internen Pfade in API-Responses)
- Debug-Endpoints und Test-Routen niemals in Produktion deployen
- Kein eval(), keine SQL-String-Konkatenation, kein unsicheres Deserialisieren

### IT-Sicherheits-Pruefpflicht
→ Siehe **Master-Checklist** am Anfang dieser Datei (Abschnitt "IT-Sicherheit").

---

## Gold-Standard & Best Practices (verbindlich fuer alle Entscheidungen)

### Grundprinzip
Jede Loesung orientiert sich am Stand der Technik fuehrender Software-Unternehmen (Google, Stripe, Airbnb, Netflix, GitHub). Funktionierend ist nicht gut genug — es muss die beste vertretbare Loesung fuer den gegebenen Kontext sein.

### Code-Qualitaet
- Funktionen: max. 20 Zeilen, eine Verantwortlichkeit (Single Responsibility)
- Keine Magic Numbers — ausnahmslos benannte Konstanten
- Kein duplizierter Code — DRY konsequent einhalten
- Variablen- und Funktionsnamen sind selbsterklaerend, keine kryptischen Abkuerzungen
- Komplexe Logik wird mit einem Kommentar begruendet, nicht beschrieben
- Kein toter Code — ungenutzte Variablen, Imports und Funktionen sofort entfernen

### Architektur
- Separation of Concerns: Datenzugriff, Business-Logik und UI strikt trennen
- Abhaengigkeiten immer nach innen (Domain-Schicht kennt keine externe Library)
- Konfiguration niemals hardcodiert — immer ueber Umgebungsvariablen externalisiert
- Neue Features in isolierten Modulen — kein "in die vorhandene Datei quetschen"
- Skalierbarkeit mitdenken: Wuerde diese Loesung bei 100x Last noch funktionieren?
- Keine zirkulaeren Abhaengigkeiten zwischen Modulen

### Single Source of Truth (SSOT)
- **Jede Mutation** ueber einen zentralen Store (z.B. `ticketStore`), nie direkt auf dem Array/Objekt
- **Jedes Rendering** ueber zentrale Helper-Funktionen, nie HTML-Templates inline duplizieren
- **Jedes Event** ueber den Event-Bus, nie direkte Render-Aufrufe nach Mutationen
- **Jeder Fallback-Text** als benannte Konstante, nie als String-Literal an der Stelle
- **Ganzheitlich migrieren, nicht in Silos:** Bei Refactorings ALLE Stellen finden — auch abweichende Schreibweisen, Sonderzeichen-Varianten, Legacy-Strings. Nach jeder Migration Gegen-Grep mit allen Varianten. Eine Migration ist erst fertig wenn der Grep ZERO Inline-Stellen findet.
- **Dynamisch verifizieren:** Nicht die Anzahl geplanter Stellen abzaehlen, sondern das Ergebnis messen. Grep-Verifikation ist Pflicht nach jeder SSOT-Migration.

### Fehlerbehandlung
- Jeder externe Call (API, DB, Filesystem) hat explizites Error-Handling
- Fehler werden geloggt mit Kontext: was wurde versucht, welcher Input, welcher Fehler
- Nutzer sehen generische Fehlermeldungen — Stack-Traces und interne Details niemals nach aussen
- Kein leeres catch() — jede Exception wird bewusst behandelt oder bewusst ignoriert (mit Kommentar warum)
- Fehlerzustaende sind genauso sorgfaeltig designed wie der Happy Path

### Testing-Mindset
- Neuer Code wird so geschrieben, dass er testbar ist (keine versteckten Abhaengigkeiten)
- Seiteneffekte isolieren, pure functions bevorzugen
- Wenn kein Test geschrieben wird: explizit benennen, was getestet werden sollte und warum es ausgelassen wurde

---

## Verifikationspflicht (vor jeder Ausgabe zwingend)

→ Siehe **Master-Checklist** am Anfang dieser Datei (Abschnitt "Referenzen & Konsistenz").

Zusaetzlich beachten:
- Alle internen Routen und Links auf Existenz pruefen
- Hardcodierte externe URLs als "manuell zu pruefen" markieren
- Keine bestehenden Schnittstellen stillschweigend veraendern
- Alle TODO-Kommentare mit Kontext und Loesungsrichtung versehen

---

## Reaktiv Corporate Design (verbindlich fuer alle UI-Komponenten)

### Farbpalette — ausschliesslich diese Werte verwenden

```css
/* Primaerfarben */
--color-reaktiv-blau:       #1F325B;   /* Hauptfarbe: Flaechen, Fliesstext */
--color-reaktiv-cyan:       #35BCEE;   /* Akzent: Icons, Highlights, kleinere Flaechen */

/* Sekundaerfarben */
--color-reaktiv-beige-hell:   #CEB6A1; /* Hintergrundflaechen, Mensch/Team-Themen */
--color-reaktiv-beige-dunkel: #A18168; /* Hintergrundflaechen */
--color-reaktiv-grau-hell:    #979698; /* Hintergrundflaechen, Headlines */
--color-reaktiv-grau-dunkel:  #363435; /* Headlines, Texte auf hellen Flaechen */

/* Keine anderen Farben verwenden. Keine hardcodierten Hex-Werte
   ausserhalb dieser Palette. Keine Farbverlaeufe (Gradients). */
```

### Typografie

Schriftfamilie: "Helvetica Neue", Helvetica, Arial, sans-serif
Fallback-Reihenfolge einhalten — kein anderer Font, insbesondere NICHT "Phenomena" (alter Markenfont, verboten).

```css
/* Textstile */

/* H1: Schlagwort */
--text-headline-schlagwort:
  font-weight: 900; /* Heavy */
  text-transform: uppercase;
  color: #363435 | #fff | #1F325B;

/* H2: Headlines auf Bildern/blauen Flaechen */
--text-headline-fliesstext:
  font-size: 35px;
  font-weight: 700; /* Bold */
  letter-spacing: 0.2em;
  line-height: 1.2;
  text-transform: uppercase;

/* H3: Headlines auf weissen Flaechen */
--text-headline-grau:
  font-size: 35px;
  font-weight: 700; /* Bold */
  letter-spacing: 0.2em;
  line-height: 1.2;
  text-transform: uppercase;

/* H4: Adressen, Kontaktdaten */
--text-kontakt:
  font-size: 22px;
  font-weight: 700;
  letter-spacing: 0.1em;

/* H5: Zitate */
--text-zitat:
  font-size: 18px;
  font-weight: 400; /* Roman */
  font-style: italic;
  letter-spacing: 0.1em;
  line-height: 1.4;

/* H6: Schlagworte, Namen */
--text-schlagwort-name:
  font-size: 20px;
  font-weight: 700;
  letter-spacing: 0.2em;
  text-transform: uppercase;

/* Fliesstext */
--text-fliesstext:
  font-size: 18px;
  font-weight: 400; /* Roman */
  letter-spacing: 0.05em;
  line-height: 1.4;

/* Einleitung */
--text-einleitung:
  font-size: 20px;
  font-weight: 400; /* Roman */
  letter-spacing: 0.1em;
  line-height: 1.4;

/* Buttons & Menuepunkte */
--text-button-menue:
  font-size: 35px;
  font-weight: 700; /* Bold */
  letter-spacing: 0.2em;
  text-transform: uppercase;
```

### Icons & Grafiken

DO:
- Ausschliesslich Outline-Icons (keine gefuellten/flaechigen Icons)
- Runde Ecken, reduzierte Formensprache
- Strichstaerke aehnlich dem "re" im reaktiv-Logo (fein, nicht zu duenn)
- Farbe: #1F325B (dunkelblau) oder #35BCEE (cyan)

DON'T:
- Keine Farbverlaeufe auf Icons
- Keine bunten Icons (ausserhalb der reaktiv-Palette)
- Keine eckigen Ecken
- Keine zu filigranen oder zu detailreichen Grafiken
- Keine gefuellten (solid) Icons

### Flaechen & Layout

DO:
- Farbflaechen immer vom Rand zum Rand (full-width) — keine freistehenden farbigen Kaesten/Boxen mit sichtbarem Rand drumherum
- Trennlinien zwischen Sektionen: leicht angewinkelt (~2 Grad), optisch aufsteigend
- Hintergrundfarben aus der reaktiv-Palette

DON'T:
- Keine freistehenden farbigen Kaesten (immer Medienrand zu Medienrand)
- Keine optisch absteigenden Winkel
- Keine vertikalen Schraegen
- Kein schwarzer Hintergrund mit farbigem Logo

### Komponenten-Verhalten

Buttons:
- Helvetica Neue Bold, Grossbuchstaben, letter-spacing: 0.2em
- Hintergrund: #1F325B oder #35BCEE
- Text: #ffffff
- Border-radius: grosszuegig (pill-style bevorzugt)
- Kein Outline-Button-Style auf dunklem Hintergrund

Karten & Container:
- Weisser oder beige/grauer Hintergrund
- Keine harten Schatten — wenn Schatten, dann weich und dezent
- Runde Ecken konsequent einsetzen

### Pruefpflicht UI
→ Siehe **Master-Checklist** am Anfang dieser Datei (Abschnitt "UI / Corporate Design").

---

## Continuous Improvement (immer, auch wenn nicht explizit gefragt)

Am Ende jeder Antwort kurz ausgeben:

- Verbesserungspotenziale: [max. 3 konkrete, priorisierte Punkte]
- Technische Schulden: [falls vorhanden — mit geschaetztem Aufwand]

Dabei gilt:
- Code Smells benennen (zu lange Funktionen, duplizierter Code, unklare Namen)
- Performance-Probleme markieren (N+1-Queries, fehlende Indizes, unnoetige Re-Renders)
- Deprecated APIs oder Libraries sofort kennzeichnen und Alternative nennen
- Wenn eine bessere Loesung existiert als die beauftragte: kurz erwaehnen, auch wenn sie mehr Aufwand bedeutet
- Sicherheitsluecken oder DSGVO-Risiken, die im Kontext auffallen, immer melden — unabhaengig davon, ob die aktuelle Aufgabe damit zusammenhaengt
