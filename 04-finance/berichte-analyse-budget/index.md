---
title: "Finanzberichte, Budget & Analyseansichten"
---
# 4. Finanzwesen — Finanzberichte, Budget & Analyseansichten

> 📄 **Zurück zur [Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }})**
> 📋 Account Schedules für Bilanz/GuV/Cashflow, Budget-Erfassung und -Kontrolle, OLAP-artige Analyseansichten.

---

## 4.17 Finanzberichte & Analyse

> **Namensraum:** `Microsoft.Finance.FinancialReports`
> **Kern-Tabellen:** `AccountScheduleName.Table.al` (91), `AccountScheduleLine.Table.al` (92), `ColumnLayout.Table.al` (93)

Account Schedules sind das zentrale Reporting-Tool. Zeilen (`AccountScheduleLine`) definieren WAS angezeigt wird, Spalten (`ColumnLayout`) definieren WIE (Periode, Vergleichszeitraum).

```al
// AccountScheduleLine: Zeilendefinition
field("Row No."; Integer)              // Zeilennummer
field("Description"; Text[100])         // Zeilenbeschreibung
field("Totaling Type"; Option)          // Posting Accounts, Formula, Net Change, ...
field("Totaling"; Text[250])            // Konten-Filter oder Formel
field("Show"; Option)                   // Ja/Nein, nur bei Betrag <> 0, ...
```

**Beispiel 1 — Eine einfache Bilanz aufbauen:**
> Das Unternehmen benötigt eine monatliche Bilanz (Aktiva + Passiva).
> ➜ Zeilen: `Row No. 10 = Sachanlagen (Konten 0100..0799)`, `Row No. 20 = Vorräte (Konten 1000..1999)`, `Row No. 30 = Summe Aktiva (Formel: 10+20)`. Spalten: `Column 1 = Aktueller Monat (Period), Column 2 = Vorjahresmonat (Comparison Period)`.
> **Ergebnis:** Der Bericht `Bilanz` zeigt alle Aktiva, Vorräte und Summe — mit Vorjahresvergleich.

**Beispiel 2 — GuV nach Kostenstellen:**
> Der Controller möchte eine GuV getrennt nach Kostenstellen sehen.
> ➜ `Dimension Perspective` für Kostenstelle anlegen (z.B. KST-VERTRIEB, KST-PRODUKTION). Die `AccountScheduleLine` referenziert die Dimension Perspective.
> **Ergebnis:** Eine GuV pro Kostenstelle — aus einer einzigen Account Schedule-Definition.

**Beispiel 3 — Monatliche Cashflow-Rechnung:**
> Der Finanzleiter möchte den Cashflow der letzten 12 Monate rollierend darstellen.
> ➜ `ColumnLayout` mit `Column Type = Formula` und `Comparison Period Formula = -1M..-12M`. In der Zeile: Konten der liquiden Mittel (1000..1299).
> **Ergebnis:** Der Bericht "Cashflow Statement" zeigt die monatliche Entwicklung — automatisch rollierend ohne manuelle Anpassung.

**Querverweis:** → [Kap. 4.1 Fibu-Einrichtung](#4-finanzen--fibu-einrichtung) — Felder `Fin. Rep. for Balance Sheet`, `Fin. Rep. for Income Stmt` in Tabelle 98

---

## 4.18 Budget

> **Namensraum:** `Microsoft.Finance.GeneralLedger.Budget`
> **Kern-Tabellen:** `GLBudgetName.Table.al` (89), `GLBudgetEntry.Table.al` (90)

Budgeteinträge werden pro Konto und Periode erfasst und mit IST-Werten in Account Schedules verglichen.

**Beispiel 1 — Jahresbudget 2026 für den Vertriebsbereich:**
> Der Vertriebsleiter plant 600.000 € Umsatz (Konto 8400) für 2026 — gleichmäßig verteilt auf 12 Monate.
> ➜ `Budget Name = UMSATZ2026`, `Budget Entry`: Konto 8400, Zeilen pro Monat à 50.000 €.
> **Ergebnis:** In der Account Schedule kann eine Spalte mit `Column Type = Budget` die IST- mit den BUDGET-Werten monatlich vergleichen.

**Beispiel 2 — Budget aus Excel importieren:**
> Der Controller hat das Budget in Excel erstellt und möchte es direkt nach BC importieren.
> ➜ Bericht `Import Budget from Excel` mit definierter Excel-Vorlage (`FinReportExcelTemplate`).
> **Ergebnis:** 500 Budgetzeilen werden in einem Rutsch importiert — keine manuelle Erfassung.

**Beispiel 3 — Budgetkontrolle mit Warnung:**
> Das Unternehmen möchte beim Buchen einer Bestellung gewarnt werden, wenn das Budget überschritten wird.
> ➜ `GLBudgetOpen.Codeunit.al` prüft bei Buchung: Budgetsumme vs. IST-Kosten. Bei Überschreitung: Warnung.
> **Ergebnis:** Echtzeit-Budgetkontrolle bereits bei der Bestellerfassung — nicht erst am Monatsende.

**Querverweis:** → [Kap. 6 Einkauf]({{ '/06-purchasing/' | relative_url }}) — Budgetkontrolle bei Bestellfreigabe

---

## 4.31 Analyseansichten (Analysis Views)

> **Namensraum:** `Microsoft.Finance.Analysis`
> **Kern-Tabellen:** `AnalysisView.Table.al` (110), `AnalysisViewEntry.Table.al` (111), `AnalysisViewFilter.Table.al` (112)
>
> Analyseansichten verdichten Sachposten nach benutzerdefinierten Dimensionen für schnelle Ad-hoc-Analysen — vergleichbar mit OLAP-Cubes, direkt in BC.

```al
field("Analysis View Code"; Code[10])
field("Analysis View Name"; Text[50])
field("Dimension 1..4 Code"; Code[20])    // Bis zu 4 Dimensionen
field("Account Source"; Enum)              // G/L Accounts oder Cash Flow
field("Date Compression"; Enum)            // Day, Week, Month, Quarter, Year
```

**Beispiel 1 — Controller erstellt eine Umsatzanalyse nach Vertreter und Region:**
> Der Vertriebscontroller möchte Umsätze nach Vertreter und Region auswerten — rollierend über 12 Monate.
> ➜ `Analysis View = UMSATZANALYSE`: Dimensionen = VERTRETER, REGION, `Account Source = G/L Accounts`, Filter auf Erlöskonten 8400..8499.
> **Ergebnis:** Die Matrix-Ansicht (`Analysis by Dimensions`) zeigt: Vertreter Müller hat in Region NORD 245.000 € Umsatz. Drill-Down in die Original-Sachposten möglich.

**Beispiel 2 — Budget-Ist-Vergleich in Analyseansicht:**
> Der Abteilungsleiter möchte ein Budget-Ist-Diagramm für seine Kostenstelle.
> ➜ `Analysis View = BUDGET-KST1`: Budget-Daten einbeziehen (`AnalysisAccountSource = GL Accounts & Budgets`), Dimension KOSTENSTELLE = KST1.
> **Ergebnis:** Die Matrix zeigt IST-Werte und BUDGET-Werte nebeneinander — Abweichungen farblich markiert.

---

---

| [← Zurück zur Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }}) | [Konsolidierung, Abgrenzungen & IC →]({{ '/04-finance/konsolidierung-abgrenzung-ic/' | relative_url }}) |
