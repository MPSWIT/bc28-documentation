---
title: "Konsolidierung, Abgrenzungen & Intercompany"
---
# 4. Finanzwesen — Konsolidierung, Abgrenzungen & Intercompany

> 📄 **Zurück zur [Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }})**
> 📋 Konzernkonsolidierung, periodengerechte Abgrenzungen (Deferrals), und automatische IC-Partner-Transaktionen.

```
4. Finanzwesen
 │
 ├── [Fibu-Einrichtung (Tab. 98)]({{ '/04-finance/fibu-einrichtung/' | relative_url }})
 ├── [Kontenplan & Buchungsgruppen]({{ '/04-finance/kontenplan-buchungsgruppen/' | relative_url }})
 ├── [MwSt-System]({{ '/04-finance/mwst-system/' | relative_url }})
 ├── [Journale, Debitoren/Kreditoren]({{ '/04-finance/journale-debitoren-kreditoren/' | relative_url }})
 ├── [Bank, Anlagen & Währung]({{ '/04-finance/bank-anlagen-waehrung/' | relative_url }})
 ├── [Berichte, Budget & Analyse]({{ '/04-finance/berichte-analyse-budget/' | relative_url }})
 ├─▶ Konsolidierung, Abgrenzungen & IC  ← Sie sind hier
 ├── [Querschnitt — Fibu-Relevanz aller Module]({{ '/04-finance/querschnitt/' | relative_url }})
 └── [Entwickler-Referenz]({{ '/04-finance/entwickler/' | relative_url }})
```

---

## 4.19 Konsolidierung

> **Namensraum:** `Microsoft.Finance.Consolidation`
> **Kern-Tabellen:** `ConsolidationSetup.Table.al` (96), `BusinessUnit.Table.al` (97), `ConsolidationAccount.Table.al` (95)

Konsolidierung fasst mehrere Mandanten (Business Units) in einem Konsolidierungsmandanten zusammen — essenziell für Konzerne.

**Beispiel 1 — Holding mit 3 Tochtergesellschaften konsolidieren:**
> Die Holding (DE) konsolidiert ihre Töchter AT, CH, NL monatlich.
> ➜ Jede Tochter als `Business Unit` anlegen, Konsolidierungskonten (`Consolidation Account`) aus dem Quellmandanten zuweisen.
> **Ergebnis:** Der Bericht `Consolidate` überträgt die Salden aller Töchter in den Konsolidierungsmandanten. Die Konzernbilanz und -GuV entstehen automatisch.

**Beispiel 2 — Währungsumrechnung für die Konsolidierung:**
> Die CH-Tochter bucht in CHF, die Konzernmutter in EUR.
> ➜ In der `Consolidation Setup` wird der CHF-zu-EUR-Wechselkurs hinterlegt. Die Konsolidierung rechnet alle CHF-Posten zu Stichtagskurs um.
> **Ergebnis:** Konzernabschluss in EUR. Währungskursdifferenzen werden auf das `Currency Translation Adjustment`-Konto gebucht.

**Beispiel 3 — Intercompany-Eliminierungen:**
> Die DE-Holding hat Forderungen von 100.000 € gegen die AT-Tochter im Soll. Die AT-Tochter hat Verbindlichkeiten von 100.000 € gegen die DE-Holding im Haben.
> ➜ `G/L Consolidation Eliminations`-Bericht bucht automatisch eine Eliminierungszeile: Soll Verbindlichkeiten 100.000 € / Haben Forderungen 100.000 €.
> **Ergebnis:** Konzernbilanz ist frei von Intercompany-Salden.

**Querverweis:** → [Kap. 4.14 Intercompany](konsolidierung-abgrenzung-ic#421-intercompany-ic-partner)

---

## 4.20 Abgrenzungen (Deferrals)

> **Namensraum:** `Microsoft.Finance.Deferral`
> **Kern-Tabellen:** `DeferralHeader.Table.al` (1700), `DeferralLine.Table.al` (1701)

Abgrenzungen verteilen Aufwände oder Erträge über mehrere Perioden. Beispiel: Eine Jahresmiete (12.000 €) soll monatlich mit 1.000 € gebucht werden.

```al
field("Deferral %"; Decimal)             // z.B. 8.33 (=100%/12)
field("Deferral Starting Date"; Date)
field("Deferral Ending Date"; Date)
field("No. of Periods"; Integer)
```

**Beispiel 1 — Jahresversicherung wird periodengerecht abgegrenzt:**
> Das Unternehmen bezahlt am 01.01. eine Jahresversicherung über 6.000 €.
> ➜ `Deferral Template = VERSICHERUNG`, 12 Monate, `Deferral % = 8.33`, `Deferral Starting Date = 01.01.2026`.
> **Ergebnis:** Bei jeder Fibu-Buchung über 6.000 € an Versicherungsaufwand erzeugt das System zusätzlich 12 monatliche Abgrenzungszeilen: 500 € Soll VersAufwand / 500 € Haben ARA. Monatlich korrekt periodisiert.

**Beispiel 2 — Vorauszahlung eines Kunden wird aufgelöst:**
> Ein Kunde zahlt 24.000 € für einen 2-Jahres-Wartungsvertrag. Der Ertrag soll monatlich mit 1.000 € gebucht werden.
> ➜ Deferral Line: 24 Monate, `Deferral % = 4.17`, `Deferral Starting Date = 01.01.2026`.
> **Ergebnis:** Monatlich wird 1.000 € Ertrag aus der Abgrenzung aufgelöst. Die GuV zeigt monatlich den korrekten Service-Ertrag.

**Beispiel 3 — Abgrenzungsvorlagen für Standard-Fälle:**
> Das Unternehmen hat 5 Standard-Abgrenzungen (Versicherung, Miete, Wartung, Leasing, Lizenzen).
> ➜ 5 `Deferral Templates` anlegen. Der Benutzer wählt nur noch die Vorlage aus — Startdatum und Prozentsatz werden automatisch gesetzt.
> **Ergebnis:** Standardisierung und Fehlervermeidung — kein manuelles Berechnen der Prozentsätze.

---

## 4.21 Intercompany (IC-Partner)

> **Namensraum:** `Microsoft.Finance.Intercompany`
> **Kern-Tabellen:** `ICPartner.Table.al` (410), `ICInboxTransaction.Table.al` (420), `ICOutboxTransaction.Table.al` (421)

Intercompany ermöglicht buchhalterische Transaktionen zwischen Mandanten eines Konzerns — automatische Gegenbuchung und IC-Bestellungen.

**Beispiel 1 — Management Fees von Mutter an Tochter:**
> Die DE-Holding (IC-Partner-Code: MUTTER) verrechnet monatlich 50.000 € Management Fees an die AT-Tochter (IC-Partner-Code: TOCHTER).
> ➜ Fibu-Buchung: Soll IC-Partner TOCHTER / Haben Erlöse 4800. Der `ICOutboxTransaction` wird automatisch erzeugt.
> **Ergebnis:** AT kann die `ICInboxTransaction` abrufen und buchen — Soll Aufwand 6100 / Haben IC-Partner MUTTER. Vollautomatische Gegenbuchung.

**Beispiel 2 — IC-Bestellung mit automatischer Gegenverkauf:**
> Die DE-Mutter bestellt bei der AT-Tochter 100 Einheiten eines Artikels.
> ➜ IC-Verkaufsauftrag in DE erfassen. Das System erzeugt automatisch einen IC-Einkaufsauftrag in AT.
> **Ergebnis:** Einkäufer und Verkäufer müssen nur EINEN Beleg im System erfassen — die Gegenbuchung entsteht automatisch.

**Beispiel 3 — IC-Transaktionen mit Dimensionen:**
> Der Konzern verwendet die Dimension PROJECT. IC-Transaktionen müssen die Projektnummer transportieren.
> ➜ In der `IC Setup` die Dimension PROJECT als IC-Dimension definieren.
> **Ergebnis:** Bei jeder IC-Transaktion wird die Projektdimension automatisch in den Zielmandanten übertragen.

---

---

| [← Zurück zur Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }}) | [Querschnitt →]({{ '/04-finance/querschnitt/' | relative_url }}) |
