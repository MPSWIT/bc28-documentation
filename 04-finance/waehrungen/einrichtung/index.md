---
title: "Währungen — Einrichtung & Setup"
---
# 4. Währungen — Einrichtung &amp; Setup

<pre>
4. Finanzwesen
 │
 ├── <a href="{{ '/04-finance/' | relative_url }}">Übersicht</a>
 ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}">Fibu-Einrichtung</a>
 ├── <a href="{{ '/04-finance/waehrungen/' | relative_url }}">Währungen &amp; Wechselkurse</a>
 │    ├─▶ Einrichtung &amp; Setup  ← Sie sind hier
 │    ├── <a href="{{ '/04-finance/waehrungen/verarbeitung/' | relative_url }}">Verarbeitung &amp; Umrechnung</a>
 │    ├── <a href="{{ '/04-finance/waehrungen/anwendung/' | relative_url }}">Anwendung &amp; Beispiele</a>
 │    └── <a href="{{ '/04-finance/waehrungen/entwickler/' | relative_url }}">Entwickler-Referenz</a>
</pre>

---

## Tabelle 4 — Currency (Währungsstamm)

### Felder im Detail

| Feld-Nr. | Name | Typ | Standard | Beschreibung |
|----------|------|-----|----------|-------------|
| 1 | Code | Code[10] | — | Eindeutiger Währungscode (z.B. USD, EUR) |
| 4 | ISO Code | Code[3] | — | ISO-4217-Buchstabencode (z.B. USD, EUR) |
| 5 | ISO Numeric Code | Code[3] | — | ISO-4217-Zahlencode (z.B. 840, 978) |
| 10 | Invoice Rounding Precision | Decimal | 0,01 | Rundungspräzision für Rechnungsbeträge |
| 12 | Invoice Rounding Type | Option | Nearest | Nearest / Up / Down |
| 13 | Amount Rounding Precision | Decimal | 0,01 | Allgemeine Betragsrundung |
| 14 | Unit-Amount Rounding Precision | Decimal | 0,00001 | Rundung für Einzelpreise |
| 15 | Description | Text[30] | — | Bezeichnung (z.B. „US-Dollar") |
| 17 | Amount Decimal Places | Text[5] | '2:2' | Dezimalstellen-Format für Beträge |
| 18 | Unit-Amount Decimal Places | Text[5] | '2:5' | Dezimalstellen-Format für Einheitspreise |
| 44 | Appln. Rounding Precision | Decimal | 0 | Rundung bei Zahlungsausgleich |
| 45 | EMU Currency | Boolean | false | EWU-Währung (Euro-Mitglied)? |
| 46 | Currency Factor | Decimal | — | **System**: Aktueller Umrechnungsfaktor |
| 52 | Max. VAT Difference Allowed | Decimal | — | Max. MwSt-Differenz |
| 53 | VAT Rounding Type | Option | Nearest | Nearest / Up / Down |
| 56 | Symbol | Text[10] | — | Währungssymbol ($, €, £) |
| 166 | Currency Symbol Position | Enum | After | Position des Symbols (vor/nach Betrag) |

### Sachkonten für Kursgewinne/-verluste

| Feld | Name | Verwendung |
|------|------|-----------|
| 6 | Unrealized Gains Acc. | Nicht realisierte Kursgewinne |
| 7 | Realized Gains Acc. | Realisierte Kursgewinne |
| 8 | Unrealized Losses Acc. | Nicht realisierte Kursverluste |
| 9 | Realized Losses Acc. | Realisierte Kursverluste |
| 40 | Realized G/L Gains Account | Realisierte Sachkonto-Kursgewinne |
| 41 | Realized G/L Losses Account | Realisierte Sachkonto-Kursverluste |
| 47 | Residual Gains Account | Rundungsdifferenz-Gewinne |
| 48 | Residual Losses Account | Rundungsdifferenz-Verluste |
| 50 | Conv. LCY Rndg. Debit Acc. | LCY-Umrechnungs-Rundung Soll |
| 51 | Conv. LCY Rndg. Credit Acc. | LCY-Umrechnungs-Rundung Haben |

### FlowFields für Debitoren-/Kreditoren-Analyse

| Feld | Name | CalcFormula |
|------|------|-------------|
| 25 | Customer Balance | sum(Detailed Cust. Ledg. Entry.Amount where Customer Filter, Currency Code) |
| 26 | Customer Outstanding Orders | sum(Sales Line.Outstanding Amount where Bill-to Customer Filter, Currency Code) |
| 30 | Vendor Balance | -sum(Detailed Vendor Ledg. Entry.Amount where Vendor Filter, Currency Code) |
| 31 | Vendor Outstanding Orders | sum(Purchase Line.Outstanding Amount where Pay-to Vendor Filter, Currency Code) |
| 34 | Customer Balance (LCY) | Kundensaldo umgerechnet in Mandantenwährung |
| 35 | Vendor Balance (LCY) | Kreditorensaldo umgerechnet in Mandantenwährung |

### Trigger-Logik

```al
trigger OnInsert()
begin
    TestField(Code);
    "Last Modified Date Time" := CurrentDateTime;
end;

trigger OnModify()
begin
    "Last Date Modified" := Today;
    "Last Modified Date Time" := CurrentDateTime;
end;

trigger OnDelete()
begin
    // Prüfung auf offene Posten in:
    // - Cust. Ledger Entry (Open=true, Currency Code=Code)
    // - Vendor Ledger Entry (Open=true, Currency Code=Code)
    // - Employee Ledger Entry (Open=true, Currency Code=Code)
    // Löschen aller zugehörigen Wechselkurse
    CurrExchRate.SetRange("Currency Code", Code);
    CurrExchRate.DeleteAll();
end;
```

### Währungssymbol-Auflösung

```al
procedure ResolveCurrencySymbol(CurrencyCode: Code[10]): Text[10]
begin
    // Vordefinierte Symbole:
    case CurrencyCode of
        'AUD','BND','CAD','FJD','HKD','MXN','NZD','SBD','SGD','USD':
            exit('$');
        'GBP':  exit('£');
        'DKK','ISK','NOK','SEK':   exit('kr');
        'EUR':  exit('€');
        'CNY','JPY': exit('¥');
    end;
end;
```

---

## Tabelle 330 — Currency Exchange Rate (Wechselkurse)

### Felder

| Feld-Nr. | Name | Typ | Beschreibung |
|----------|------|-----|-------------|
| 1 | Currency Code | Code[10] | FK → Currency |
| 2 | Starting Date | Date | Gültig-ab-Datum |
| 3 | Exchange Rate Amount | Decimal | Betrag in dieser Währung (Standard: 1) |
| 4 | Adjustment Exch. Rate Amount | Decimal | Anpassungskurs (für Kursanpassung) |
| 5 | Relational Currency Code | Code[10] | Bezugswährung (leer = LCY) |
| 6 | Relational Exch. Rate Amount | Decimal | Betrag in Bezugswährung |
| 7 | Fix Exchange Rate Amount | Enum | Welcher Kurs festgehalten wird |
| 8 | Relational Adjmt Exch Rate Amt | Decimal | Anpassungskurs für Bezugswährung |

### Schlüssel

```
Key1 (PK): Currency Code, Starting Date (Clustered)
```

---

## Wechselkursdienste — Automatische Aktualisierung

Business Central kann Wechselkurse über externe Dienste automatisch aktualisieren. Die Codeunit **1281** `Update Currency Exchange Rates` steuert den Prozess.

### Tabelle: Curr. Exch. Rate Update Setup

Jeder Dienst wird als eigener Datensatz konfiguriert:

- **Service Provider** — Name des Anbieters (z.B. „FloatRates", „European Central Bank")
- **Data Exch. Def Code** — Data-Exchange-Definition für das Importformat
- **Enabled** — Aktiv? Ja/Nein
- **Log Web Requests** — Webanfragen protokollieren

### Ablauf der Kursaktualisierung

```
CurrExchRateUpdateSetup.FindSet() (Enabled = true)
 │
 ├── 1. GetCurrencyExchangeData()
 │    ├── WebService-URL aus Service-Definition laden
 │    ├── HttpWebRequestMgt.Initialize(URL)
 │    ├── HttpWebRequestMgt.GetResponse(ResponseInStream)
 │    └── ResponseInStream enthält XML/JSON mit Kursdaten
 │
 ├── 2. CreateDataExchange()
 │    ├── Bei JSON: JsonToXML-Konvertierung
 │    ├── DataExch.InsertRec(SourceName, ResponseInStream, DataExchDefCode)
 │    └── ReadingWritingCodeunit ausführen
 │
 ├── 3. ProcessDataExchange()
 │    ├── Data-Exchange-Definition verarbeitet die Quelldaten
 │    ├── MapCurrencyExchangeRate ordnet Felder zu
 │    └── Schreiben in CurrencyExchangeRate-Tabelle
 │
 └── 4. LogTelemetryWhenExchangeRateUpdated()
      └── Telemetrie: Kategorie = 'AL Exchange Rate Service'
```

### Einrichtung eines Wechselkursdienstes (Schritt-für-Schritt)

1. **Curr. Exch. Rate Update Setup** öffnen
2. Neuen Datensatz anlegen
3. Service Provider wählen (z.B. „ECB")
4. Data Exch. Def Code = `EXCHRATE-ECB` (automatisch eingerichtet)
5. Enabled = Ja
6. **Aktion → Wechselkurse aktualisieren** ausführen
7. Bei Erfolg: Wechselkurse sind in `Currency Exchange Rates` sichtbar

> **Hinweis:** Die Aktualisierung kann auch über die **Aufgabenwarteschlange** automatisiert werden — einfach Codeunit 1281 als periodischen Job einrichten.

---

## Setup in der Fibu-Einrichtung

### Zusätzliche Berichtswährung (ACY)

In der **Fibu-Einrichtung** können Sie eine **Zusätzliche Berichtswährung** festlegen:

1. Fibu-Einrichtung öffnen
2. Feld „Zusätzliche Berichtswährung" = gewünschte Währung (z.B. EUR, wenn LCY = USD)
3. Diese Währung benötigt ebenfalls Wechselkurse

> **Wichtig bei ACY:** Für die ACY müssen alle Realized-GL-Konten (`Realized G/L Gains Account`, `Realized G/L Losses Account`) so konfiguriert sein, dass **Exchange Rate Adjustment = No Adjustment**. BC prüft dies beim Start der Kursanpassung. Ebenso müssen die MwSt-, Steuer- und Bankkonten entsprechend konfiguriert sein.

### Rundungspräzisionen vererben

```al
procedure InitRoundingPrecision()
begin
    GLSetup.Get();
    "Amount Rounding Precision" := GLSetup."Amount Rounding Precision";
    "Unit-Amount Rounding Precision" := GLSetup."Unit-Amount Rounding Precision";
    "Max. VAT Difference Allowed" := GLSetup."Max. VAT Difference Allowed";
    "VAT Rounding Type" := GLSetup."VAT Rounding Type";
    "Invoice Rounding Precision" := GLSetup."Inv. Rounding Precision (LCY)";
    "Invoice Rounding Type" := GLSetup."Inv. Rounding Type (LCY)";
end;
```

---

## Praktische Setup-Konfiguration für EUR (LCY) + USD

```
Währung: EUR (LCY, bereits vorhanden)
 ├── ISO Code: EUR
 ├── Amount Rounding Precision: 0,01
 ├── Unit-Amount Rounding Precision: 0,00001
 └── Symbol: €

Währung: USD
 ├── ISO Code: USD
 ├── Amount Rounding Precision: 0,01
 ├── Unit-Amount Rounding Precision: 0,00001
 ├── Symbol: $
 ├── Unrealized Gains Acc.: 4820 (Kursgewinne nicht realisiert)
 ├── Realized Gains Acc.:   4821 (Kursgewinne realisiert)
 ├── Unrealized Losses Acc.: 4830 (Kursverluste nicht realisiert)
 ├── Realized Losses Acc.:   4831 (Kursverluste realisiert)
 └── Residual Gains/Losses Acc.: 4840 / 4841

Wechselkurseintrag USD:
 ├── Starting Date: 01.06.2024
 ├── Exchange Rate Amount: 1
 ├── Relational Currency Code: EUR
 ├── Relational Exch. Rate Amount: 0,92
 ├── Fix Exchange Rate Amount: Currency
 └── Adjustment Exch. Rate Amount: 1
 └── Relational Adjmt Exch Rate Amt: 0,90
```
