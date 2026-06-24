---
title: "Währungen & Wechselkurse"
---
# 4. Finanzwesen — Währungen &amp; Wechselkurse

<pre>
4. Finanzwesen
 │
 ├── <a href="{{ '/04-finance/' | relative_url }}">Übersicht</a>
 ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}">Fibu-Einrichtung</a>
 ├─▶ Währungen &amp; Wechselkurse  ← Sie sind hier
 │    ├── <a href="{{ '/04-finance/waehrungen/einrichtung/' | relative_url }}">Einrichtung &amp; Setup</a>
 │    ├── <a href="{{ '/04-finance/waehrungen/verarbeitung/' | relative_url }}">Verarbeitung &amp; Umrechnung</a>
 │    ├── <a href="{{ '/04-finance/waehrungen/anwendung/' | relative_url }}">Anwendung &amp; Beispiele</a>
 │    └── <a href="{{ '/04-finance/waehrungen/entwickler/' | relative_url }}">Entwickler-Referenz</a>
</pre>

---

## 4.X.1 Architektur — Das Währungssystem in BC28

Das Währungssystem in Business Central ist in **Microsoft.Finance.Currency** (BaseApp) implementiert und besteht aus drei Hauptkomponenten:

| Komponente | Dateien | Funktion |
|------------|---------|----------|
| **Währungsstamm** | `Currency.Table.al` (Tabelle 4), `Currencies.Page.al` | Währungsdefinition, Rundungspräzision, Sachkonten-Zuordnung |
| **Wechselkursverwaltung** | `CurrencyExchangeRate.Table.al` (Tabelle 330), `CurrencyExchangeRates.Page.al` | Tägliche Wechselkurse mit relationalem Modell |
| **Kursaktualisierung** | `UpdateCurrencyExchangeRates.Codeunit.al` (1281), `CurrExchRateUpdateSetup.Table.al`, `SetUpCurrExchRateService.Codeunit.al` | Automatischer Abruf von Wechselkursen über Webservices |
| **Kursanpassung** | `ExchRateAdjmtProcess.Codeunit.al` (699), `ExchRateAdjustment.Report.al`, `ExchRateAdjmtReg.Table.al`, `ExchRateAdjmtLedgEntry.Table.al` | Periodische Neubewertung von Fremdwährungsposten |
| **Erweiterte Währungen** | `AdditionalCurrencyManagement.Codeunit.al`, `GLCurrencyRevaluation.Report.al` | Zusätzliche Berichtswährung (ACY), Sachkontenumwertung |

### Namensraum

```al
namespace Microsoft.Finance.Currency;
```

### Objektübersicht

| ID | Typ | Name | Beschreibung |
|----|-----|------|-------------|
| 4 | Table | Currency | Währungsstammdaten |
| 330 | Table | Currency Exchange Rate | Wechselkurse |
| — | Table | Curr. Exch. Rate Update Setup | Kursaktualisierungs-Setup |
| — | Table | Exch. Rate Adjmt. Register | Kursanpassungs-Register |
| — | Table | Exch. Rate Adjmt. Ledg. Entry | Kursanpassungs-Posten |
| — | Table | Exch. Rate Adjmt. Parameters | Kursanpassungs-Parameter |
| 699 | Codeunit | Exch. Rate Adjmt. Process | Hauptlogik Kursanpassung |
| 1281 | Codeunit | Update Currency Exchange Rates | Kursaktualisierung |
| — | Codeunit | Additional Currency Management | Zusatzwährungs-Umrechnung |
| — | Codeunit | Map Currency Exchange Rate | Mapping Kursdaten von Services |
| — | Report | Exch. Rate Adjustment | Kursanpassung ausführen |
| — | Report | GL Currency Revaluation | Sachkonto-Neubewertung |

---

## 4.X.2 Schlüsselkonzepte

### Relationale Wechselkurse

BC verwendet ein einzigartiges Modell für Wechselkurse: **jeder Wechselkurs ist eine Relation zwischen zwei Währungen**.

Ein Eintrag in Tabelle 330 sieht so aus:

| Feld | Beispiel | Bedeutung |
|------|----------|-----------|
| Currency Code | `USD` | Währung, für die der Kurs gilt |
| Starting Date | `01.06.2024` | Gültig ab |
| Exchange Rate Amount | `1` | 1 Einheit der Currency... |
| Relational Currency Code | `EUR` | ...in Bezug auf diese Währung |
| Relational Exch. Rate Amount | `0,92` | ...entspricht 0,92 EUR |
| Adjustment Exch. Rate Amount | `1` | Kurs für Anpassungsbewertung |
| Relational Adjmt Exch Rate Amt | `0,90` | Relationaler Anpassungskurs |

> **Bedeutung:** `1 USD = 0,92 EUR` ab dem 01.06.2024, aber für die Kursanpassung wird `1 USD = 0,90 EUR` verwendet.

### Fix Exchange Rate Amount (Enum)

Steuert, welcher Kurs bei Cross-Rate-Berechnungen fixiert wird:

| Wert | Bedeutung |
|------|-----------|
| **Relational Currency** | Relationaler Währungsbetrag ist fix |
| **Currency** | Währungsbetrag ist fix |
| **Both** | Beide Beträge werden neu berechnet |

### Standard-Wechselkurs ohne Relational Currency

Wenn `Relational Currency Code` leer ist, wird die **Mandantenwährung (LCY)** implizit als Bezugswährung verwendet:

```
Exchange Rate Amount = 1
Relational Exch. Rate Amount = 1 / Faktor
```

Beispiel: `1 USD = 0,92 EUR`, LCY = EUR:
- `Exchange Rate Amount` = `1`
- `Relational Exch. Rate Amount` ≈ `1,08696` (1 / 0,92)

### Währungsfaktor

Der **Currency Factor** (Feld 46 in Tabelle 4) wird von BC automatisch berechnet:

```
Currency Factor = Exchange Rate Amount / Relational Exch. Rate Amount
```

Bedeutet: **1 LCY = ? Fremdwährung**. Der Faktor wird im `ExchRateAdjmtProcess` gespeichert und in allen Umrechnungsfunktionen verwendet.

---

## 4.X.3 Datenfluss — Vom Beleg zum Buchungsbetrag

```
Belegerfassung (z.B. Verkaufsrechnung in USD)
 │
 ├── Benutzer gibt Fremdwährungsbetrag ein
 │
 ├── BC ermittelt Wechselkurs für Belegdatum
 │    └── CurrencyExchangeRate.FindCurrency(Date, CurrencyCode)
 │         ├── Sucht letzten Kurs ≤ Belegdatum
 │         └── Cache-Mechanismus: CurrencyExchRate2[1], CurrencyCode2[1], Date2[1]
 │
 ├── Umrechnung: ExchangeAmtFCYToLCY(Date, USD, $1000, Factor)
 │    ├── Case 1: Keine Relational Currency → $1000 / Factor
 │    ├── Case 2: Relational Currency = '' und Fix Exchange Rate = Both
 │    │           → ($1000 / ExchangeRateAmt) × RelExchRateAmt
 │    ├── Case 3: Relational Currency vorhanden (z.B. EUR)
 │    │           → Dreiecksumrechnung über EUR
 │    │           → Faktor = (ExchRateUSD × ExchRateEUR) / (RelExchRateUSD × RelExchRateEUR)
 │    └── Ergebnis: Betrag in LCY
 │
 ├── Buchung im Fibu-Journal
 │    ├── Soll/Haben in LCY (umgerechnet)
 │    ├── Fremdwährungsbetrag gespeichert
 │    └── Wechselkurs in Buchungszeile dokumentiert
 │
 └── Periodische Kursanpassung (Monatsende)
      ├── ExchRateAdjmtProcess.Codeunit.al (699)
      ├── Anpassung für: Debitoren, Kreditoren, Bank, Sachkonten, MwSt
      └── Ergebnis: Kursdifferenz als Fibu-Buchung
```
