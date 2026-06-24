---
title: "Intrastat-Meldung"
---
# 7. Intrastat-Meldung — Überblick & Architektur

Die Intrastat-Meldung ist eine EU-weite Statistikpflicht für den grenzüberschreitenden Warenverkehr. Business Central 28 stellt ein vollständiges Modul zur Erfassung, Verarbeitung, Prüfung und Meldung der Intrastat-Daten bereit.

<pre>
7. Lager &amp; Logistik
 │
 ├── <a href="{{ '/07-inventory/' | relative_url }}">Übersicht</a>
 ├─▶ Intrastat  ← Sie sind hier
 │    ├── <a href="{{ '/07-inventory/intrastat/einrichtung/' | relative_url }}">Einrichtung &amp; Setup</a>
 │    ├── <a href="{{ '/07-inventory/intrastat/verarbeitung/' | relative_url }}">Verarbeitung &amp; Hintergrund</a>
 │    ├── <a href="{{ '/07-inventory/intrastat/export/' | relative_url }}">Export &amp; Hochladen</a>
 │    └── <a href="{{ '/07-inventory/intrastat/entwickler/' | relative_url }}">Entwickler-Referenz</a>
</pre>

---

## 7.1 App-Struktur

Das Intrastat-Modul besteht aus zwei gekoppelten Apps:

| App | ID | ID-Range | Beschreibung |
|-----|----|----------|-------------|
| **Intrastat Core** | `70912191-3c4c-49fc-a1de-bc6ea1ac9da6` | 4810–4840 | Basis-Tabellen, Reports, Codeunits, Setup, Verarbeitung |
| **Intrastat AT** (Beispiel) | `268aefab-94e4-4596-a7a7-dbf4c6785efb` | 11150–11161 | AT-spezifische Data-Exchange-Definition, Checklisten, Feldanpassungen |

Die Intrastat AT-App hat eine Dependency auf Intrastat Core und erweitert Tabellen/Codeunits per `tableextension` und `[EventSubscriber]`.

## 7.2 Objektübersicht (Core)

### Tabellen

| ID | Name | Beschreibung |
|----|------|-------------|
| 4810 | `Intrastat Report Setup` | Zentrale Einrichtung (Singleton) |
| 4811 | `Intrastat Report Header` | Kopf der Intrastat-Meldung |
| 4812 | `Intrastat Report Line` | Zeilen (einzelne Warenbewegungen) |
| 4813 | `Intrastat Report Checklist` | Plausibilitäts-Checkliste |

### Codeunits

| ID | Name | Funktion |
|----|------|----------|
| 4810 | `IntrastatReportManagement` | Kernlogik: Export, Validierung, VAT, Recalc |
| 4812 | `Intrastat Report Doc. Compl.` | Beleg-Vervollständigung: Default-Werte beim Anlegen |
| 4813 | `Intrastat Report Exp. External` | Dummy-Codeunit für Data-Exchange-Definition |
| 4814 | `IntrastatReportUpgrade` | Upgrade-Logik |
| 4815 | `IntrastatReportItemTracking` | Item-Tracking-Daten für Intrastat |

### Reports

| ID | Name | Typ |
|----|------|-----|
| 4810 | `Intrastat Report Get Lines` | ProcessingOnly — holt Zeilen aus Item/Job/FA Ledger Entries |

### Enums (Core)

| ID | Name | Werte |
|----|------|-------|
| 4810 | `Intrastat Report Source Type` | Item Entry, Job Entry, FA Entry |
| 4811 | `Intrastat Report Line Type` | Receipt, Shipment |
| 4812 | `Intrastat Report Status` | Open, Released |
| 4813 | `Intrastat Report Type` | Purchases, Sales |
| — | `Intrastat Report Periodicity` | Month, Quarter, Year |
| — | `Intrastat Report Shpt. Base` | Ship-to Country, Sell-to Country, Bill-to Country |
| — | `Intrastat Report VAT File Fmt` | VAT Reg. No., EU Code + VAT, VAT without EU Code |
| — | `Intrastat Report VAT No. Base` | Document, Sell-to VAT, Bill-to VAT |
| — | `Intr. Rep. Purch. VAT No. Base` | Document, Buy-from VAT, Pay-to VAT |
| — | `Intrastat Report Contact Type` | Contact, Vendor |

## 7.3 Datenquellen & Datenfluss

```
Einkaufsbeleg (Purchase Header)  ──┐
Verkaufsbeleg (Sales Header)     ──┤  Buchen
Umlagerung (Transfer Header)     ──┤  ↓
Servicbeleg (Service Header)     ──┤  Item Ledger Entry
Anlagenbeleg (FA Ledger Entry)   ──┘  Job Ledger Entry
Projektposten (Job Ledger Entry) ────  FA Ledger Entry
                                           │
                    Intrastat Report Get Lines (Report 4810)
                    • Prüft Grenzübertritt (HasCrossedBorder)
                    • Berechnet Beträge (CalculateTotals)
                    • Ermittelt Partner-VAT-ID
                    • Erzeugt Intrastat Report Lines
                                           │
                    Intrastat Report Header (4811) + Lines (4812)
                    • Status-Änderung: Open → Released
                    • Checklist-Validierung
                                           │
                    Export (IntrastatReportManagement)
                    • Data Exchange Definition
                    • Gruppierung gleicher Schlüssel
                    • Transformation (TRIMALL, ROUNDTOINT)
                                           │
                              TXT/ZIP-Datei(en)
```

## 7.4 Wichtige Design-Entscheidungen

1. **Lines sind mengenmäßig gruppiert** (Key Index 5): Gleiche Kombination aus Type, Land, Tarifnummer, Transaktionstyp, Transportmethode, Ursprungsland und Partner-VAT-ID werden in einer Zeile zusammengefasst (via `DataExchFieldGrouping`).

2. **Source Entry No. verhindert Doppelerfassung**: Jede Zeile speichert die Referenz auf den Quell-Eintrag (`Item Ledger Entry`.`Entry No.`, etc.). Bei erneutem `Get Lines` werden bereits vorhandene Zeilen übersprungen.

3. **Negative Menge = Shipment**: `Quantity < 0` → `Type = Shipment`, `Quantity > 0` → `Type = Receipt`.

4. **Partner-VAT-ID mehrstufig**: Erst Dokument-VAT, dann Debitor/Kreditor-VAT, dann Default-Werte (Private Person, 3-Party Trade, Unknown State).

5. **Ländererweiterungen via Event-Subscriber**: AT, DE, etc. abonnieren `OnBeforeInitSetup`, `OnBeforeInitCheckList`, `OnBeforeGetPartnerIDForCountry` und überschreiben die Core-Logik.

6. **Data Exchange Framework**: Der Export verwendet das BC Data Exchange Framework (`Data Exch.`, `Data Exch. Def`, `Data Exch. Mapping`). Die Definition ist als XML-Label im Code hinterlegt und wird beim Setup importiert.
