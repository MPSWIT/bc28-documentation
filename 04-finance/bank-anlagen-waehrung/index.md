---
title: "Bank, Anlagen & Währung"
---
# 4. Finanzwesen — Bank, Anlagen & Währung

> 📄 **Zurück zur [Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }})**
> 📋 SEPA-Zahlungsverkehr und Bankabstimmung, Anlagenverwaltung mit Abschreibungen, Fremdwährungen und Wechselkursanpassungen.

---

## 4.14 Bank & Zahlungsverkehr

> **Namensräume:** `Microsoft.Bank.*`, `Microsoft.CashFlow.*`
> **Kern-Tabellen:** `BankAccount.Table.al` (270), `BankAccReconciliation.Table.al` (273), `BankAccReconLine.Table.al` (274)

### SEPA-Export & Zahlungsdateien

BC28 unterstützt SEPA Credit Transfers (Überweisungen) und Direct Debits (Lastschriften) über den `Payment Journal`.

**Beispiel 1 — Zahllauf für 50 Kreditorenrechnungen:**
> Das Unternehmen bezahlt jeden Dienstag alle fälligen Kreditorenrechnungen per SEPA-Überweisung.
> ➜ `Payment Journal` → Kreditorenrechnungen auswählen (`Suggest Vendor Payments`) → `Export SEPA Payment File`.
> **Ergebnis:** Eine XML-Datei (pain.001) wird erzeugt und an die Bank übermittelt. 50 Einzelüberweisungen in einer Sammeldatei.

**Beispiel 2 — Bankkontoauszug importieren und abstimmen:**
> Der Bankkontoauszug kommt als CAMT.053-Datei. Das Unternehmen importiert ihn und gleicht ihn mit den gebuchten Posten ab.
> ➜ `Bank Acc. Reconciliation` → `Import Bank Statement` → CAMT.053 auswählen.
> **Ergebnis:** Gebuchte Posten werden automatisch zugeordnet (`Match`). Differenzen (z.B. Bankgebühren, Zinsen) werden als neue Fibu-Buchungen erfasst — kompletter Bankabstimmungs-Workflow.

**Beispiel 3 — SEPA-Lastschrift für 200 Kunden:**
> Das Unternehmen zieht monatlich per SEPA-Basislastschrift bei 200 Kunden ein.
> ➜ `Payment Journal` mit `Gen. Journal Template = ZAHLUNG`, `Bal. Account Type = Bank Account`, `Account Type = Customer`.
> **Ergebnis:** Eine SEPA Direct Debit XML-Datei (pain.008) entsteht. Bei Buchung werden 200 Debitorenposten automatisch ausgeglichen. Keine manuelle Zahlungszuordnung.

**Querverweis:** → [Kap. 4.1 §4.7 Aufgabenwarteschlange](fibu-einrichtung#47-aufgabenwarteschlange--hintergrundbuchung)

---

## 4.15 Anlagen (Fixed Assets)

> **Namensräume:** `Microsoft.FixedAssets.*`
> **Kern-Tabellen:** `FixedAsset.Table.al` (5600), `FADepreciationBook.Table.al` (5612), `FALedgerEntry.Table.al` (5614)

Anlagen (Gebäude, Maschinen, Fuhrpark) werden über Anlagenkarten verwaltet, linear oder degressiv abgeschrieben. Jede Abschreibungsbuchung erzeugt einen Fibu-Posten.

```al
// FADepreciationBook: AfA-Parameter
field("Depreciation Method"; Option)     // Straight-Line, Declining-Balance, ...
field("Depreciation %"; Decimal)
field("No. of Depreciation Years"; Integer)
```

**Beispiel 1 — Eine neue CNC-Maschine wird aktiviert und abgeschrieben:**
> Das Unternehmen kauft eine CNC-Maschine für 120.000 € netto, Nutzungsdauer 10 Jahre, lineare AfA.
> ➜ `Fixed Asset Card`: Anschaffungskosten = 120.000 €, `Depreciation Method = Straight-Line`, `No. of Depreciation Years = 10`.
> **Ergebnis:** Monatliche AfA-Buchung: 1.000 € Soll an AfA-Konto (4000), Haben an Anlagekonto (0700). Der `FALedgerEntry` enthält den Restbuchwert.

**Beispiel 2 — Vorzeitiger Abgang einer Anlage:**
> Ein Firmenwagen (Buchwert 15.000 €) wird nach 3 Jahren für 12.000 € verkauft — Verlust 3.000 €.
> ➜ Anlagenverkauf über `FA Journal`. Buchung: Bank 12.000 € / Erlös aus Anlageabgang 12.000 €, Restbuchwert 15.000 € / Anlageabgang 15.000 €, Verlust 3.000 €.
> **Ergebnis:** Das Anlagekonto wird vollständig aufgelöst. GuV-wirksamer Verlust von 3.000 €.

**Beispiel 3 — Geringwertige Wirtschaftsgüter (GWG):**
> Das Unternehmen kauft einen Bürostuhl für 450 € netto (unter 800 € Grenze) und schreibt ihn sofort voll ab.
> ➜ `Depreciation Method = Straight-Line`, `No. of Depreciation Years = 1` oder über das GWG-Profil: Sofortabschreibung im 1. Jahr.
> **Ergebnis:** 450 € werden im Anschaffungsjahr voll abgeschrieben. Keine Verteilung über mehrere Jahre.

**Querverweise:**
→ [Kap. 6 Einkauf]({{ '/06-purchasing/' | relative_url }}) — Anlagen aus Einkaufsbestellung aktivieren  
→ [Kap. 7 Lager]({{ '/07-inventory/' | relative_url }}) — Bestandsbewertung vs. Anlagenbewertung

---

## 4.16 Währung & Wechselkurse

> **Namensraum:** `Microsoft.Finance.Currency`
> **Kern-Tabellen:** `Currency.Table.al` (4), `CurrencyExchangeRate.Table.al` (330)

Fremdwährungsbuchungen erfordern Wechselkurse. BC28 bewertet offene Posten periodisch neu und bucht Kursdifferenzen.

```al
// ExchRateAdjmtReg: Kursanpassungs-Register
field("Currency Code"; Code[10])
field("Starting Date"; Date)
field("Ending Date"; Date)
field("Adjustment Date"; Date)
```

**Beispiel 1 — EUR-basierte Firma kauft in USD ein:**
> Das Unternehmen kauft Waren für 10.000 USD bei einem Kurs von 1,10 USD/EUR (= 9.090,91 €). Bei Zahlung (30 Tage später) beträgt der Kurs 1,12 USD/EUR (= 8.928,57 €).
> ➜ Kursgewinn: 162,34 €.
> **Ergebnis:** Bei Zahlung wird die Kursdifferenz automatisch auf das `Realized Gain`-Konto gebucht. Der Bericht `Adjust Exchange Rates` (Wechselkurse anpassen) berücksichtigt das.

**Beispiel 2 — Periodische Kursanpassung für OP-Liste:**
> Die USD-Debitorenposten (offene Kundenforderungen) werden zum Monatsultimo mit dem neuen Kurs bewertet.
> ➜ Bericht "`Adjust Exchange Rates`" ausführen. Parameter: `Adjustment Date = 30.06.2026`, `Currency Code = USD`.
> **Ergebnis:** Alle offenen USD-Posten erhalten eine Kursdifferenz-Buchung. Der unrealisierte Gewinn/Verlust wird auf das entsprechende Konto gebucht.

**Beispiel 3 — Wechselkurs automatisch über Service aktualisieren:**
> Das Unternehmen hat `CurrExchRateUpdateSetup` für die EZB konfiguriert.
> ➜ `Update Currency Exchange Rates`-Codeunit läuft täglich über die Aufgabenwarteschlange und holt die aktuellen EZB-Referenzkurse.
> **Ergebnis:** Die Wechselkurstabelle ist stets aktuell. Kein manuelles Einpflegen der Kurse.

---

---

| [← Zurück zur Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }}) | [Berichte, Budget & Analyse →]({{ '/04-finance/berichte-analyse-budget/' | relative_url }}) |
