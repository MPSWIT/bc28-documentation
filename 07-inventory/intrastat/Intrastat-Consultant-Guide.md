# Intrastat-Meldung in Business Central 28

## Ein Leitfaden für Consultants und Anwender

---

## 1. Was ist Intrastat?

**Intrastat** ist das EU-weite System zur statistischen Erfassung des Warenverkehrs zwischen den
Mitgliedstaaten. Seit 1993 müssen Unternehmen in der EU melden, welche Waren sie aus anderen
EU-Ländern beziehen (Eingang/Receipt) und in andere EU-Länder liefern (Versand/Shipment).

Die Meldepflicht betrifft **alle Unternehmen**, die innergemeinschaftliche Warenbewegungen
durchführen und bestimmte **Schwellenwerte** überschreiten. In Österreich liegen diese
Schwellenwerte für 2024 bei:

| Richtung | Jährliche Schwelle |
|----------|--------------------|
| **Eingang** (Intra-EU-Bezüge) | EUR 1.500.000 |
| **Versand** (Intra-EU-Lieferungen) | EUR 1.000.000 |

> **Wichtig:** Auch unterhalb der Schwellenwerte kann eine Meldung sinnvoll sein — viele
> Unternehmen melden freiwillig, um bei Überschreitung keine Nachmeldungen durchführen zu müssen.

---

## 2. Warum ist das für Ihr Unternehmen relevant?

### 2.1 Gesetzliche Verpflichtung

Als Unternehmen in der EU sind Sie gesetzlich verpflichtet, Ihre innergemeinschaftlichen
Warenbewegungen zu melden. Die Nichteinhaltung kann zu **Bußgeldern und Strafzahlungen** führen.

Die Meldung erfolgt monatlich und muss bis zum **10. Arbeitstag des Folgemonats** bei der
zuständigen Statistikbehörde eingereicht werden (in Österreich: Statistik Austria).

### 2.2 Meldepflichtige Vorgänge

Folgende Geschäftsvorfälle sind intrastat-relevant:

✅ **Einkauf von Waren aus anderen EU-Ländern** (z.B. Rohstoffe aus Deutschland, Handelswaren aus
   Italien)

✅ **Verkauf von Waren in andere EU-Länder** (z.B. Fertigprodukte nach Frankreich, Ersatzteile
   nach Tschechien)

✅ **Umlagerungen zwischen eigenen Lagern in verschiedenen EU-Ländern**

✅ **Rücksendungen und Gutschriften** (Korrekturmeldungen)

Nicht meldepflichtig sind:

❌ **Dienstleistungen** (unterliegen einem eigenen Meldeverfahren)

❌ **Drittlandsgeschäfte** (Export/Import mit Nicht-EU-Ländern — diese werden über Zollmeldungen
   erfasst)

❌ **Inlandsgeschäfte** (innerhalb Österreichs)

❌ **Anlagevermögen** (Fix Assets — werden über eigene Meldungen erfasst)

---

## 3. Welche Daten werden gemeldet?

Pro Meldeposition werden folgende Informationen an die Statistikbehörde übermittelt:

| Datenfeld | Beschreibung | Woher in BC? |
|-----------|--------------|--------------|
| **Tarifnummer** | 8-stellige Zolltarifnummer (KN-Code) | Artikelstammdaten |
| **Land** | EU-Partnerland | Belegkopf (Ship-to, Sell-to, Bill-to) |
| **Transaktionstyp** | Art des Geschäfts (z.B. Kauf/Verkauf, Retoure, Lohnveredelung) | Belegkopf oder Setup-Default |
| **Menge** | Bewegte Menge in Grundmengeneinheit | Gebuchte Posten |
| **Gesamtgewicht** | Nettogewicht × Menge | Artikelstammdaten |
| **Statistischer Wert** | Warenwert in EUR (inkl. Nebenkosten) | Berechnet aus gebuchten Kosten/Preisen |
| **Partner-UID** | Umsatzsteuer-ID des Geschäftspartners | Debitor/Kreditor-Stammdaten |
| **Ursprungsland** | Land, in dem die Ware hergestellt wurde | Artikelstammdaten, Serial No., Lot No. |

---

## 4. Der Intrastat-Prozess in Business Central

### 4.1 Übersicht

```
┌─────────────────────────────────────────────────────────────┐
│                    Monatlicher Intrastat-Prozess              │
│                                                               │
│  1. Täglich: Korrekte Stammdaten pflegen                     │
│     • Artikel: Tarifnummer, Nettogewicht, Ursprungsland      │
│     • Kunden/Lieferanten: UID-Nummer                          │
│                                                               │
│  2. Laufend: Geschäftsvorfälle buchen                        │
│     • Einkaufs- und Verkaufsbelege                           │
│     • Intrastat-Felder werden automatisch befüllt            │
│                                                               │
│  3. Monatsende: Intrastat-Meldung erstellen                  │
│     ┌───────────────────────────────────────────────┐        │
│     │  a) Neue Meldung anlegen (Statistikperiode)   │        │
│     │  b) Zeilen abrufen (Get Lines)                │        │
│     │  c) Zeilen prüfen und ggf. ergänzen/korrigieren│       │
│     │  d) Checklist-Validierung durchführen         │        │
│     │  e) Meldung freigeben (Release)               │        │
│     │  f) Export-Datei erzeugen                     │        │
│     │  g) Bei Statistik Austria hochladen           │        │
│     └───────────────────────────────────────────────┘        │
│                                                               │
│  4. Kontrolle: Export-Status und Korrekturmeldungen          │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Schritt-für-Schritt-Anleitung

#### Schritt 1: Einrichtung (einmalig)

Die Intrastat-Einrichtung erreichen Sie über die Suche (**Alt+Q**) → „Intrastat Einrichtung".

| Einstellung | Empfehlung für Österreich |
|-------------|---------------------------|
| Wareneingänge melden | ✅ Ja |
| Warenausgänge melden | ✅ Ja |
| Dateien aufteilen | ✅ Ja (getrennt nach Eingang/Versand) |
| ZIP-Dateien erstellen | ✅ Ja |
| Basis für Ausgänge | Liefer-an-Land (Ship-to Country) |
| MwSt-ID-Format für Kunden | UID-Nummer (VAT Reg. No.) |
| MwSt-ID-Format für Kreditoren | UID-Nummer (VAT Reg. No.) |
| Standard-UID Privatperson | QN999999999999 |
| Standard-UID Dreiecksgeschäft | QV999999999999 |
| Nummernserie | INTR (anlegen falls nicht vorhanden) |
| Transaktionstyp Pflichtfeld | ✅ Ja |

#### Schritt 2: Artikelstammdaten pflegen

Für jeden Artikel, der EU-grenzüberschreitend bewegt wird:

1. Artikelkarte öffnen
2. In den **Intrastat-Inforegister** wechseln
3. Folgende Felder ausfüllen:
   - **Zolltarifnummer** — z.B. `84713000` (tragbare Datenverarbeitungsgeräte)
   - **Nettogewicht** — Gewicht pro Einheit in kg
   - **Ursprungsland** — Herstellungsland (wichtig für Versandmeldung!)
   - **Erg. Mengeneinheit** — falls statistisch gefordert (z.B. Stück, Paar)
   - **Von Intrastat-Meldung ausschließen** = Nein

> **Tipp:** Nutzen Sie die **Konfigurationsvorlagen** für neue Artikel, um Intrastat-Felder
> automatisch vorzubelegen.

#### Schritt 3: Debitoren und Kreditoren pflegen

- **UID-Nummer** im Feld „UST-ID-Nr." auf der Kunden- bzw. Kreditorenkarte eintragen
- Bei **Privatkunden**: Business Central verwendet automatisch den Ersatzwert
  „QN999999999999"
- Bei **Dreiecksgeschäften**: Business Central verwendet automatisch „QV999999999999"

#### Schritt 4: Belege korrekt erfassen

Business Central füllt folgende Felder automatisch auf Einkaufs- und Verkaufsbelegen (Kopf):

| Feld | Herkunft |
|------|----------|
| **Transaktionstyp** | Setup: Default Trans. - Purchase / Default Trans. - Return |
| **Transportmethode** | Manuell oder aus Lieferant/Kunde |
| **Ein-/Ausgangsort** | Manuell |
| **Transaktionsspezifikation** | Manuell |
| **Versandart** | Manuell oder aus Lieferant/Kunde |

> **Wichtig:** Bei Verkaufsgutschriften und Einkaufsgutschriften wird automatisch der
> **Gutschrifts-Transaktionstyp** verwendet (Default Trans. - Return).

#### Schritt 5: Monatliche Meldung erstellen

1. **Meldung anlegen:**
   - Intrastat-Meldungskopf öffnen
   - **Typ**: Einkauf oder Verkauf
   - **Statistikperiode**: Format JJMM (z.B. `2406` für Juni 2024)
   - **Periodizität**: Monat

2. **Zeilen abrufen („Vorschlag erstellen"):**
   - Das System durchsucht alle gebuchten Artikelposten des Monats
   - Einträge mit **grenzüberschreitender Bewegung** werden als Zeilen angelegt
   - **Achtung:** Das Löschen bestehender Zeilen wird nachgefragt — bestätigen Sie nur beim
     ersten Durchlauf!

3. **Zeilen prüfen und ergänzen:**
   - Kontrollieren Sie die automatisch ermittelten Zeilen
   - Prüfen Sie besonders: Tarifnummer, Gewicht, Partner-UID, Ursprungsland
   - Ergänzen Sie manuelle Zeilen (z.B. für nicht über Artikel gebuchte Waren)
   - Nutzen Sie die **Checkliste** zur Validierung

4. **Meldung freigeben:**
   - Status von „Offen" auf „Freigegeben" setzen
   - Danach sind keine Änderungen mehr möglich

5. **Export-Datei erstellen:**
   - Aktion „Exportieren" ausführen
   - Das System erzeugt eine ZIP-Datei mit den formatierten TXT-Dateien
   - Bei österreichischen Meldungen: getrennte Dateien für Eingang (`Receipt-*.txt`) und
     Versand (`Shipment-*.txt`)

6. **Bei Statistik Austria hochladen:**
   - Einloggen auf [www.statistik.at](https://www.statistik.at)
   - Im eQuest-Portal die Dateien hochladen
   - Nach erfolgreicher Übermittlung: **Reported** = Ja setzen

---

## 5. Praxisbeispiele

### 5.1 Beispiel 1: Einkauf von Waren aus Deutschland

**Szenario:** Ihr Unternehmen kauft 100 Stück Elektronikbauteile (Tarifnummer `85423190`) von
einem deutschen Lieferanten (UID: `DE123456789`).

**In BC:**
1. Einkaufsbestellung erfassen
2. Artikel hat Tarifnummer `85423190`, Nettogewicht 0,05 kg
3. Lieferant hat VAT Registration No. `DE123456789`
4. Einkaufslieferung und -rechnung buchen

**Intrastat-Meldung (Juni):**
| Feld | Wert | Quelle |
|------|------|--------|
| Tarifnummer | 85423190 | Artikel |
| Land | DE | Lieferant |
| Transaktionstyp | 11 (Kauf) | Belegkopf |
| Menge | 100 | Artikelposten |
| Gesamtgewicht | 5,0 kg | 100 × 0,05 |
| Statistischer Wert | EUR 2.500,00 | Einkaufspreis |
| Partner-UID | DE123456789 | Lieferant |
| Ursprungsland | DE | Artikel |

### 5.2 Beispiel 2: Verkauf in die Niederlande

**Szenario:** Sie verkaufen 20 Maschinen (Tarifnummer `84798997`, Nettogewicht 45 kg) an einen
Kunden in den Niederlanden (UID: `NL987654321B01`).

**Intrastat-Meldung (Juni):**
| Feld | Wert | Quelle |
|------|------|--------|
| Tarifnummer | 84798997 | Artikel |
| Land | NL | Kunde (Ship-to) |
| Transaktionstyp | 21 (Verkauf) | Belegkopf |
| Menge | 20 | Artikelposten |
| Gesamtgewicht | 900 kg | 20 × 45 |
| Statistischer Wert | EUR 45.000,00 | Verkaufspreis |
| Partner-UID | NL987654321B01 | Kunde |
| Ursprungsland | AT (eigenes Land) → NL | Logik für Versand |

### 5.3 Beispiel 3: Gutschrift/Korrektur

**Szenario:** Ein Kunde in Italien (UID: `IT12345678901`) sendet 3 von 10 gelieferten Maschinen zurück.

1. Sie erstellen eine **Verkaufsgutschrift** über 3 Stück
2. Der Transaktionstyp wird automatisch auf **22 (Gutschrift)** gesetzt
3. In der Intrastat-Meldung erscheint eine **Korrekturzeile** mit negativer Menge (−3)

---

## 6. Häufige Fehler und Lösungen

| Problem | Ursache | Lösung |
|---------|---------|--------|
| Artikel erscheint nicht in der Meldung | EU-Land des Partners nicht erkannt | Prüfen Sie, ob das Land einen Intrastat-Code hat (Länder-Tabelle) |
| Keine Zeilen in der Meldung | Keine grenzüberschreitenden Buchungen im Zeitraum | Zeitraum prüfen, gebuchte Posten kontrollieren |
| Falsches Partnerland | „Shipments Based On" falsch konfiguriert | Setup-Einstellung anpassen (Ship-to / Sell-to / Bill-to) |
| Falscher statistischer Wert | Nebenkosten nicht berücksichtigt | „Amount incl. Item Charges" = Ja beim Zeilenabruf |
| Tarifnummer fehlt | Artikel hat keine Tarifnummer | Artikelstammdaten ergänzen, dann Meldung neu berechnen |
| Partner-UID fehlt | Kunde/Lieferant hat keine VAT-ID | VAT-ID nachtragen oder Default-Wert verwenden |
| Doppelte Zeilen bei erneutem Get Lines | Source Entry No. nicht eindeutig | Bestehende Zeilen vorher löschen oder „Nur neue Einträge" verwenden |
| Meldung lässt sich nicht freigeben | Checklisten-Prüfung schlägt fehl | Fehlende Pflichtfelder ergänzen, dann erneut prüfen |

---

## 7. Checklist-Validierung verstehen

Die Checkliste ist ein konfigurierbares Prüfwerkzeug, das sicherstellt, dass alle gesetzlich
vorgeschriebenen Felder ausgefüllt sind.

In Österreich sind folgende Felder Pflicht:

| Feld | Bedingung |
|------|-----------|
| Zolltarifnummer | Immer |
| Partnerland | Immer |
| Transaktionstyp | Immer |
| Menge | Immer |
| Statistischer Wert | Immer |
| Gesamtgewicht | Immer |
| Ursprungsland | Immer |
| Ergänzende Menge | Nur wenn Artikel eine erg. Einheit hat |
| Partner-UID | Nur bei Versandmeldungen (Shipment) |

> **Hinweis für Consultants:** Die Checkliste ist je Land unterschiedlich und wird über die
> Ländererweiterung (z.B. Intrastat AT) gesteuert. Die Validierung kann als Batch über alle
> Zeilen oder als Warnung/Fehler konfiguriert werden.

---

## 8. Korrekturmeldungen

Wenn Sie nach der Übermittlung feststellen, dass eine Meldung fehlerhaft oder unvollständig war:

### 8.1 In derselben Meldung

- **Meldung wiedereröffnen** (Status: Freigegeben → Offen)
- Zeilen korrigieren oder ergänzen
- Erneut freigeben und exportieren
- **Achtung:** `Corrective Entry` wird automatisch auf Ja gesetzt

### 8.2 In einer neuen Meldung (für frühere Periode)

- Neue Intrastat-Meldung mit derselben Statistikperiode anlegen
- Alte Meldung als **Referenz** angeben (`Corrected Intrastat Report No.`)
- Nur die korrigierten/neuen Zeilen aufnehmen
- Die Statistikbehörde erkennt die Korrektur anhand des `Corrective Entry`-Flags

---

## 9. Intrastat-Setup zusammenfassend konfigurieren

```
Intrastat-Einrichtung — Empfohlene Werte für AT:

┌─────────────────────────────────────────────────────┐
│ Allgemein                                            │
│  ✅ Wareneingänge melden                             │
│  ✅ Warenausgänge melden                             │
│  Ansprechpartner: [Ihr Intrastat-Beauftragter]       │
│  Nummernserie: INTR                                  │
│                                                       │
│ Standardwerte Einkauf                                 │
│  Transaktionstyp Einkauf: 11                         │
│  Transaktionstyp Retoure: 21                         │
│                                                       │
│ Standardwerte Verkauf                                 │
│  Transaktionstyp Verkauf: 11                         │
│  Transaktionstyp Retoure: 21                         │
│                                                       │
│ Export                                                │
│  Dateien aufteilen: ✅ Ja                            │
│  ZIP-Dateien: ✅ Ja                                  │
│  Data Exch. Def. Code - Receipt: INTRA-2022-AT       │
│  Data Exch. Def. Code - Shpt.: INTRA-2022-AT         │
│                                                       │
│ MwSt-IDs                                              │
│  Format Kunden-UID: UID-Nummer                       │
│  Format Kreditoren-UID: UID-Nummer                   │
│                                                       │
│ Ersatzwerte                                           │
│  UID Privatperson: QN999999999999                    │
│  UID Dreiecksgeschäft: QV999999999999                │
│  UID Unbekannt: QV999999999999                       │
│                                                       │
│ Weiteres                                              │
│  Basis für Ausgänge: Liefer-an-Land                  │
│  Herkunftsland bei Artikeln: Artikelursprungsland    │
│  Streckengeschäfte einbeziehen: ✅ Ja                │
└─────────────────────────────────────────────────────┘
```

---

## 10. FAQ — Häufig gestellte Fragen

### F: Wann muss ich eine Intrastat-Meldung abgeben?

Sobald Ihr Unternehmen die **Schwellenwerte** für Wareneingang oder -versand überschreitet.
Maßgeblich sind die Werte des laufenden Jahres. Bei Überschreitung beginnt die Meldepflicht
ab dem Folgemonat.

### F: Ist Intrastat dasselbe wie die Zusammenfassende Meldung (ZM)?

**Nein.** Die ZM erfasst **Dienstleistungen** und meldepflichtige **innergemeinschaftliche
Lieferungen** (nur Verkauf). Intrastat erfasst **Warenbewegungen** in beide Richtungen. Beide
Meldungen sind getrennt einzureichen.

### F: Welche Transaktionstypen gibt es?

Die wichtigsten Codes (vereinfacht):

| Code | Bedeutung | Richtung |
|------|-----------|----------|
| 11 | Kauf / Verkauf | Beide |
| 21 | Retoure / Gutschrift | Beide |
| 22 | Ersatzlieferung | Beide |
| 30 | Umlagerung zwischen Lagern | Beide |
| 41 | Lohnveredelung | Beide |
| 51 | Finanzierungsleasing | Beide |

Eine vollständige Liste finden Sie im Intrastat-Leitfaden Ihrer Statistikbehörde.

### F: Unsere Artikel haben keine Tarifnummer. Was tun?

Die Tarifnummer ist ein **Pflichtfeld** für die Intrastat-Meldung. Sie müssen für jeden
relevanten Artikel die korrekte 8-stellige Zolltarifnummer ermitteln. Nutzen Sie:
- Das **Elektronische Zolltarifsystem** (z.B. TARIC-Datenbank)
- Ihren Zollberater oder Spediteur
- Die Suchfunktion in der BC-Tarifnummerntabelle

### F: Was passiert bei falschen Meldungen?

Fehlerhafte Meldungen müssen korrigiert werden (Korrekturmeldung, siehe Kapitel 8). Bei
vorsätzlichen oder wiederholten Falschmeldungen drohen **Bußgelder** gemäß
Außenhandelsstatistikgesetz.

### F: Kann ich die monatliche Meldung automatisieren?

Ja. Business Central bietet folgende Automatisierungsmöglichkeiten:
- **Aufgabenwarteschlange**: Get Lines und Export als periodischen Job einrichten
- **Standard-Transaktionstypen**: Vorkonfiguration für Belegarten
- **Konfigurationsvorlagen**: Automatische Vorgabewerte für neue Artikel/Kunden/Lieferanten

### F: Müssen Nullmeldungen abgegeben werden?

Ja, auch wenn in einem Monat keine meldepflichtigen Warenbewegungen stattgefunden haben, muss
eine **Fehlanzeige** (Nullmeldung) abgegeben werden. Diese kann direkt im eQuest-Portal
erfolgen.

---

## 11. Weiterführende Ressourcen

- **Statistik Austria — Intrastat**: [www.statistik.at/intrastat](https://www.statistik.at/intrastat)
- **eQuest-Portal**: [equest.statistik.at](https://equest.statistik.at)
- **BC-Lernmaterialien**: Microsoft Learn — „Einrichten der Intrastat-Meldung"
- **Intrastat-Entwickler-Referenz**: Siehe unsere BC28-Dokumentation → [Intrastat Entwickler-Referenz](07-inventory/intrastat/entwickler/)
- **Zolltarif online**: [TARIC-Datenbank der EU](https://ec.europa.eu/taxation_customs/dds2/taric/)

---

<p align="center"><em>Dokument erstellt für die BC28-Dokumentation. Version 1.0, Juni 2024.</em></p>
