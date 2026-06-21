---
title: "ToDo — Wertefluss in Business Central"
---
# 19.1 Wertefluss in Business Central

> **Leitfrage:** Wie hängen die diversen Posten zusammen? Wann werden sie gebucht und wie kommen die Werte in die Fibu?

> 📄 **[← ToDo-Übersicht]({{ '/19-todo/' | relative_url }})**

---

BC28 ist ein **doppisches ERP**: Jeder Geschäftsvorfall erzeugt letztlich einen **Sachposten** (`G/L Entry`, Tabelle 17). Der Weg dorthin führt jedoch über verschiedene **Nebenbuchhaltungen** (Subledger), die Details speichern, bevor sie verdichtet in die Fibu gelangen.

## 19.1.1 Das Posten-Prinzip: Codeunits orchestrieren den Fluss

Die Buchung ist kein einzelner Schritt, sondern eine **Kette von Prozedur-Aufrufen** in dedizierten Codeunits:

```
Beleg (Sales Header) → Buchungs-Codeunit → Posten-Tabellen
                                              ├── Cust./Vend. Ledger Entry
                                              ├── Item Ledger Entry → Value Entry
                                              ├── VAT Entry
                                              └── G/L Entry
```

Die Haupt-Buchungs-Codeunits:

| Codeunit | Name | Aufgabe |
|---|---|---|
| 80 | `Sales-Post` | Verkaufsbelege buchen |
| 90 | `Purch.-Post` | Einkaufsbelege buchen |
| 22 | `Item Jnl.-Post` | Lagerjournal buchen |
| 12 | `Gen. Jnl.-Post` | Fibu-Buchungsblatt buchen |

---

## 19.1.2 Der Verkaufsfluss (Sales → Fibu)

Beim Buchen einer **Verkaufsrechnung** (`Sales Invoice`) läuft in Codeunit 80 `Sales-Post` folgende Kette ab:

```
┌─────────────────────────────────────────────────────────────┐
│ 1. PostSalesLine (jede Zeile)                               │
│    ├── PostItemLine → PostItemJnlLine                       │
│    │   └── Item Journal Line → Item Jnl.-Post (CU 22)       │
│    │       └── Item Ledger Entry (Tab. 32)                  │
│    │       └── Value Entry (Tab. 5802) ← Einstandswert      │
│    └── Aufbau des Posting Buffer (Zwischenspeicher)         │
├─────────────────────────────────────────────────────────────┤
│ 2. PostInvoice                                             │
│    ├── InvoicePostingInterface.PostLines()                  │
│    │   → Fibu-Zeilen + MwSt aus Posting Buffer              │
│    │   → G/L Entry (Sachposten, Tab. 17)                    │
│    │   → VAT Entry (MwSt-Posten, Tab. 254)                  │
│    ├── InvoicePostingInterface.PostLedgerEntry()            │
│    │   → Cust. Ledger Entry (Debitorenposten, Tab. 21)      │
│    └── InvoicePostingInterface.PostBalancingEntry()         │
│        → Gegenkonto (Bal. Account) → G/L Entry              │
└─────────────────────────────────────────────────────────────┘
```

**Beispiel — Verkauf eines Artikels für 1.000 € netto:**

> *TechTrading* verkauft 10 Monitore à 100 €. Buchungsgruppe `INLAND`, MwSt 20 %.

```
1. PostItemLine → Item Ledger Entry:
   Entry Type = Sales, Quantity = -10
   Cost Amount (Actual) = -500 €  (Einstandswert)
   → Value Entry: Cost Amount = -500 €

2. PostInvoice → G/L Entry (aus Buchungsmatrix):
   Debitorenkonto           1.200 € (Haben)
   Umsatzerlöse             1.000 € (Soll)
   MwSt-Konto                 200 € (Soll)
   Bestandsveränderung        500 € (Soll) ← aus Value Entry

3. Cust. Ledger Entry:
   Customer No. = K10000
   Amount = 1.200 €, Remaining Amount = 1.200 € (offen)
```

---

## 19.1.3 Der Einkaufsfluss (Purchase → Fibu)

Spiegelbild im Einkauf — Codeunit 90 `Purch.-Post`:

```
Bestellung → Purch.-Post
├── PostPurchLine → PostItemLine → Item Jnl.-Post (CU 22)
│   └── Item Ledger Entry (Zugang) + Value Entry (Einstandswert)
├── PostInvoice
│   ├── InvoicePostingInterface.PostLines() → G/L Entry + VAT Entry
│   ├── InvoicePostingInterface.PostLedgerEntry() → Vend. Ledger Entry
│   └── InvoicePostingInterface.PostBalancingEntry() → G/L Entry
```

---

## 19.1.4 Das komplette Posten-Ökosystem

```
                          ┌──────────────────┐
                          │   G/L Entry      │ ← Fibu (Ziel)
                          │   (Tabelle 17)   │
                          └──────┬───────────┘
                                 │
         ┌───────────────┬───────┼───────┬───────────────┐
         │               │       │       │               │
    ┌────▼─────┐   ┌─────▼────┐ ┌▼─────┐ ┌▼────────┐ ┌──▼──────────┐
    │Cust. L.E.│   │Vend. L.E.│ │VAT   │ │Bank L.E.│ │FA Ledger    │
    │(Tab. 21) │   │(Tab. 25) │ │Entry │ │(Tab. )  │ │Entry        │
    │Debitoren │   │Kreditoren│ │(254) │ │Bank     │ │(Tab. 5602)  │
    └──────────┘   └──────────┘ └──────┘ └─────────┘ │Anlagen     │
                                                      └─────────────┘
         │               │
    ┌────▼───────────────▼──────┐
    │   Item Ledger Entry       │ ← Lager (Mengen)
    │   (Tabelle 32)            │
    └────────────┬──────────────┘
                 │ 1:1 Beziehung
    ┌────────────▼──────────────┐
    │   Value Entry             │ ← Lager (Werte)
    │   (Tabelle 5802)          │
    └───────────────────────────┘
```

### Wann entstehen welche Posten?

| Vorgang | Entstehende Posten | Codeunit |
|---|---|---|
| Verkaufsrechnung buchen | `Cust. Ledger Entry` + `G/L Entry` + `VAT Entry` + `Item Ledger Entry` + `Value Entry` | 80 |
| Einkaufsrechnung buchen | `Vend. Ledger Entry` + `G/L Entry` + `VAT Entry` + `Item Ledger Entry` + `Value Entry` | 90 |
| Zahlungseingang buchen | `Cust. Ledger Entry` (Ausgleich) + `Bank Ledger Entry` + `G/L Entry` | 12 |
| Zahlungsausgang buchen | `Vend. Ledger Entry` (Ausgleich) + `Bank Ledger Entry` + `G/L Entry` | 12 |
| Lagerzugang (Einkauf) | `Item Ledger Entry` (Entry Type = Purchase) + `Value Entry` (Direct Cost) | 22 |
| Lagerabgang (Verkauf) | `Item Ledger Entry` (Entry Type = Sales) + `Value Entry` (Cost Amount Actual) | 22 |
| Fibu-Buchungsblatt | `G/L Entry` + ggf. `VAT Entry` | 12 |

---

## 19.1.5 Die Buchungsmatrix als Schlüssel

Die Abbildung Subledger → Fibu-Konto erfolgt über das **General Posting Setup** (Tabelle 98). Diese Matrix kombiniert:

```
Gen. Bus. Posting Group × Gen. Prod. Posting Group → Fibu-Konten

Beispiel: INLAND × HANDEL → 
  Sales Account           = 4000 (Umsatzerlöse)
  Sales Line Disc. Account = 4730 (Erlösschmälerungen)
  Sales Pmt. Disc. Debit Acc. = 4735 (Skonto)
  ...
```

Ohne gültigen Eintrag in dieser Matrix wird die Buchung mit einer Fehlermeldung abgewiesen.

---

## 19.1.6 Wertfluss-Beispiel über mehrere Module

> *BauProfi GmbH* kauft 500 Sack Zement (Stückliste), produziert Fertigbeton und verkauft ihn.

```
1. EINKAUF (CU 90):
   Item Ledger Entry: Qty +500, Entry Type = Purchase
   Value Entry: Direct Cost = 2.500 €
   Vend. Ledger Entry: Amount = 3.000 € (brutto)
   G/L Entry: Vorrat 2.500 € / Kreditor 3.000 € / VSt 500 €

2. PRODUKTION (CU 22):
   Item Ledger Entry: Zement -500 (Consumption), Beton +100 (Output)
   Value Entry: Zement Cost -2.500 €, Beton Direct Cost +2.500 €
   G/L Entry: Bestandsveränderung (Umbuchung innerhalb Vorrat)

3. VERKAUF (CU 80):
   Item Ledger Entry: Beton -100, Entry Type = Sales
   Value Entry: Cost Amount (Actual) = -2.500 €
   Cust. Ledger Entry: Amount = 4.800 €
   G/L Entry: Debitor 4.800 € / Umsatz 4.000 € / USt 800 €
             + Bestandsveränderung 2.500 € (Cost of Goods Sold)
```

Die **Wertschöpfungskette** ist lückenlos nachvollziehbar: Jeder Bestandszugang und -abgang erzeugt sowohl einen `Item Ledger Entry` (Menge) als auch einen `Value Entry` (Wert), der wiederum in den `G/L Entry` einfließt.

---

| [← ToDo-Übersicht]({{ '/19-todo/' | relative_url }}) |
