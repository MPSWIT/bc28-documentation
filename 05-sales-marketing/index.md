---
title: "Vertrieb & Marketing"
---
# 5. Vertrieb & Marketing

> **Tabelle 311:** `SalesReceivablesSetup.Table.al`
> **Namensraum:** `Microsoft.Sales.*`
> **Belegtypen:** Angebot, Rahmenauftrag, Auftrag, Rechnung, Gutschrift, Retoure
> **Anzahl Setup-Felder:** 73

Das **Vertriebs- und Marketingmodul** verwaltet den gesamten Sales-Prozess — vom ersten Kundenkontakt über Angebote und Aufträge bis zur Fakturierung und dem Mahnwesen. Jede Verkaufstransaktion mündet über die Buchungsmatrix ([Kap. 4, Kontenplan & Buchungsgruppen]({{ '/04-finance/kontenplan-buchungsgruppen' | relative_url }})) in die Finanzbuchhaltung.

## 📋 Themen-Übersicht

| # | Thema | Beschreibung |
|---|---|---|
| 1 | **[Einrichtung (Tabelle 311)]({{ '/05-sales-marketing/einrichtung/' | relative_url }})** | Die 73 Felder des Sales & Receivables Setup: Rabattbuchung, Kreditwarnungen, Nummernserien, Archivierung, Hintergrundbuchung, Preisfindung |
| 2 | **[Verkaufsbelege]({{ '/05-sales-marketing/verkaufsbelege/' | relative_url }})** | Sales Header/Line (Tabelle 36/37): Angebot, Rahmenauftrag, Auftrag, Rechnung, Gutschrift, Retoure |
| 3 | **[Preise & Rabatte]({{ '/05-sales-marketing/preise-rabatte/' | relative_url }})** | Preisliste (Tabelle 7000), Verkaufspreise, Zeilenrabatte, Rechnungsrabatte, Preisberechnungsmethode |
| 4 | **[Mahnwesen]({{ '/05-sales-marketing/mahnwesen/' | relative_url }})** | Mahnstufen, Mahnmethoden, Zinsrechnung, Gebühren, Finanzbuchungsverkehr |
| 5 | **[Kundenverwaltung]({{ '/05-sales-marketing/kunden/' | relative_url }})** | Debitorenkarte (Tabelle 18), Buchungsgruppen, Kreditlimit, Zahlungsbedingungen, Lieferadressen |
| 6 | **[Kampagnen & Segmente]({{ '/05-sales-marketing/kampagnen/' | relative_url }})** | Marketing-Kampagnen, Kundensegmente, Interaktionen, Verkaufschancen |
| 7 | **[Entwickler-Referenz]({{ '/05-sales-marketing/entwickler/' | relative_url }})** | Integrationsereignisse, Codeunits, abhängige Tabellen |

---

## 🗺️ Namensraum-Übersicht

| Namespace | Inhalt | Abgedeckt in |
|---|---|---|
| `Microsoft.Sales.Setup` | Sales & Receivables Setup (T.311) | Einrichtung |
| `Microsoft.Sales.Document` | Sales Header/Line, Codeunits | Verkaufsbelege |
| `Microsoft.Sales.History` | Posted Sales Invoices etc. | Verkaufsbelege |
| `Microsoft.Sales.Archive` | Archivierte Belege | Einrichtung |
| `Microsoft.Sales.Pricing` | Preis- und Rabattberechnung | Preise & Rabatte |
| `Microsoft.Sales.Customer` | Debitorenkarte | Kunden |
| `Microsoft.Sales.Receivables` | Debitorenposten, OP-Verwaltung | Kunden |
| `Microsoft.Sales.Reminder` | Mahnungen | Mahnwesen |
| `Microsoft.Sales.FinanceCharge` | Finanzbuchungsverkehr | Mahnwesen |
| `Microsoft.Sales.Campaign` | Kampagnen | Kampagnen |
| `Microsoft.Sales.Segment` | Segmente | Kampagnen |
| `Microsoft.Sales.Comment` | Kommentare | Verkaufsbelege |
| `Microsoft.Sales.Peppol` | PEPPOL-Export | Verkaufsbelege |

---

## 📖 Durchgängige Buchungskette (Verkauf → Fibu)

```
Kundenkarte (Debitor)
     │
     ├──→ Verkaufsangebot → Auftrag → Warenausgang → Rechnung
     │                              (Lager, Kap. 7)     │
     │                                                   │
     ├──→ Preisfindung ←── Preisliste                    │
     │         │                                         │
     │    Zeilenrabatt / Rechnungsrabatt                 │
     │                                                   │
     └──→ Buchung:  Debitor / Erlöse / MwSt (Kap. 4)  ←─┘
                         │
                    OP-Verwaltung
                         │
                    Mahnwesen (überfällige Posten)
```

---

## 🔗 Verwandte Kapitel

| Kapitel | Relevanz |
|---|---|
| [Kap. 4 — Finanzwesen]({{ '/04-finance/' | relative_url }}) | Buchungsmatrix, Sachkonten, MwSt |
| [Kap. 6 — Einkauf]({{ '/06-purchasing/' | relative_url }}) | Spiegelbild: Kreditoren statt Debitoren |
| [Kap. 7 — Lager & Logistik]({{ '/07-inventory/' | relative_url }}) | Warenausgang, Kommissionierung |
| [Kap. 12 — CRM / Kontaktmanagement]({{ '/12-crm/' | relative_url }}) | Kampagnen, Segmente |

---

| [← Zurück zur Übersicht]({{ '/index' | relative_url }}) | [Weiter: Einrichtung →]({{ '/05-sales-marketing/einrichtung/' | relative_url }}) |
