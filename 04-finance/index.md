---
title: "Finanzwesen"
---
# 4. Finanzwesen — Übersicht

<pre>
4. Finanzwesen
 │
 ├─▶ Übersicht  ← Sie sind hier
 │
 ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}">Fibu-Einrichtung (Tab. 98)</a>
 ├── <a href="{{ '/04-finance/kontenplan-buchungsgruppen/' | relative_url }}">Kontenplan &amp; Buchungsgruppen</a>
 ├── <a href="{{ '/04-finance/mwst-system/' | relative_url }}">MwSt-System</a>
 ├── <a href="{{ '/04-finance/journale-debitoren-kreditoren/' | relative_url }}">Journale, Debitoren/Kreditoren</a>
 ├── <a href="{{ '/04-finance/bank-anlagen-waehrung/' | relative_url }}">Bank, Anlagen &amp; Währung</a>
 ├── <a href="{{ '/04-finance/waehrungen/' | relative_url }}">Währungen &amp; Wechselkurse</a>
 │    ├── <a href="{{ '/04-finance/waehrungen/einrichtung/' | relative_url }}">Einrichtung &amp; Setup</a>
 │    ├── <a href="{{ '/04-finance/waehrungen/verarbeitung/' | relative_url }}">Verarbeitung &amp; Umrechnung</a>
 │    ├── <a href="{{ '/04-finance/waehrungen/anwendung/' | relative_url }}">Anwendung &amp; Beispiele</a>
 │    └── <a href="{{ '/04-finance/waehrungen/entwickler/' | relative_url }}">Entwickler-Referenz</a>
 ├── <a href="{{ '/04-finance/berichte-analyse-budget/' | relative_url }}">Berichte, Budget &amp; Analyse</a>
 ├── <a href="{{ '/04-finance/konsolidierung-abgrenzung-ic/' | relative_url }}">Konsolidierung, Abgrenzungen &amp; IC</a>
 ├── <a href="{{ '/04-finance/querschnitt/' | relative_url }}">Querschnitt — Fibu-Relevanz aller Module</a>
 └── <a href="{{ '/04-finance/entwickler/' | relative_url }}">Entwickler-Referenz</a>
</pre>

> **30 Namespaces · 32 Themen · 45+ Tabellen · ~75 User-Story-Beispiele**
> Alle offiziellen BC28-de-AT Feldübersetzungen verwendet.

Die **Finanzbuchhaltung** (Fibu) ist das Herzstück von Business Central. Jede buchhalterisch relevante Transaktion — ob aus Einkauf, Verkauf, Lager, Produktion, Service, Projekten oder HR — mündet in die Fibu.

---

## 📋 Themen-Übersicht

| # | Thema | Beschreibung | Umfang |
|---|---|---|---|
| 1 | **[Fibu-Einrichtung (Tabelle 98)]({{ '/04-finance/fibu-einrichtung/' | relative_url }})** | Die 22 Felder des GL-Setup: Buchungszeiträume, Dimensionen, MwSt-Felder, Zahlungstoleranzen, Rundung, Berichtswährung, Aufgabenwarteschlange | 22 Felder, 66 Beispiele |
| 2 | **[Kontenplan & Buchungsgruppen]({{ '/04-finance/kontenplan-buchungsgruppen/' | relative_url }})** | Sachkonten (Tabelle 15), die Buchungsmatrix (Gen. Posting Setup), Kontierungsschlüssel | 3 Themen, 9 Beispiele |
| 3 | **[MwSt-System]({{ '/04-finance/mwst-system/' | relative_url }})** | VAT Posting Setup, MwSt-Abrechnung, Klauseln (Textbausteine), Satzänderungs-Tool, US Sales Tax | 4 Themen, 11 Beispiele |
| 4 | **[Journale, Debitoren/Kreditoren]({{ '/04-finance/journale-debitoren-kreditoren/' | relative_url }})** | Fibu-Buchungsjournale, OP-Verwaltung, Zahlungsausgleich, Stornobuchungen | 3 Themen, 9 Beispiele |
| 5 | **[Bank, Anlagen & Währung]({{ '/04-finance/bank-anlagen-waehrung/' | relative_url }})** | SEPA-Zahlungsverkehr, Bankabstimmung, Anlagenverwaltung (AfA), Fremdwährungen & Wechselkurse | 3 Themen, 9 Beispiele |
| 6 | **[Berichte, Budget & Analyse]({{ '/04-finance/berichte-analyse-budget/' | relative_url }})** | Account Schedules (Bilanz/GuV/Cashflow), Budget-Erfassung & -Kontrolle, Analyseansichten | 3 Themen, 8 Beispiele |
| 7 | **[Konsolidierung, Abgrenzungen & IC]({{ '/04-finance/konsolidierung-abgrenzung-ic/' | relative_url }})** | Konzernkonsolidierung, periodengerechte Abgrenzungen, Intercompany-Partner | 3 Themen, 9 Beispiele |
| 8 | **[Fibu-Relevanz aller Module]({{ '/04-finance/querschnitt/' | relative_url }})** | Querschnitt: Wie Einkauf/Verkauf/Lager/Produktion/Service/Projekte/HR in die Fibu buchen + Dimensionskorrektur | 2 Themen, durchgängige Buchungskette |
| 9 | **[Entwickler-Referenz]({{ '/04-finance/entwickler/' | relative_url }})** | Integrationsereignisse, Rollencenter für Finanzanwender, alle abhängigen Tabellen & Codeunits | 3 Themen, 45+ Tabellen |

---

## 🗺️ Namensraum-Übersicht (30 Namespaces)

| Namespace | Thema | Abgedeckt in |
|---|---|---|
| `Microsoft.Finance.GeneralLedger.Setup` | Fibu-Einrichtung | Fibu-Einrichtung |
| `Microsoft.Finance.GeneralLedger.Account` | Sachkonten | Kontenplan |
| `Microsoft.Finance.GeneralLedger.Journal` | Buchungsjournale | Journale |
| `Microsoft.Finance.GeneralLedger.Ledger` | Sachposten | Journale |
| `Microsoft.Finance.GeneralLedger.Posting` | Buchungslogik | Journale |
| `Microsoft.Finance.GeneralLedger.Preview` | Buchungsvorschau | Fibu-Einrichtung |
| `Microsoft.Finance.GeneralLedger.Reports` | Fibu-Berichte | Berichte & Analyse |
| `Microsoft.Finance.GeneralLedger.Reversal` | Stornobuchungen | Journale |
| `Microsoft.Finance.GeneralLedger.Budget` | Budget | Berichte & Analyse |
| `Microsoft.Finance.Dimension` | Dimensionen | Fibu-Einrichtung |
| `Microsoft.Finance.Dimension.Correction` | Dimensionskorrektur | Querschnitt |
| `Microsoft.Finance.VAT.Setup` | MwSt-Einrichtung | MwSt-System |
| `Microsoft.Finance.VAT.Calculation` | MwSt-Berechnung | MwSt-System |
| `Microsoft.Finance.VAT.Ledger` | MwSt-Posten | MwSt-System |
| `Microsoft.Finance.VAT.Reporting` | MwSt-Meldung | MwSt-System |
| `Microsoft.Finance.VAT.Registration` | USt-ID-Prüfung | MwSt-System |
| `Microsoft.Finance.VAT.Clause` | MwSt-Klauseln | MwSt-System |
| `Microsoft.Finance.VAT.RateChange` | MwSt-Satzänderung | MwSt-System |
| `Microsoft.Finance.ReceivablesPayables` | Debitoren/Kreditoren | Journale |
| `Microsoft.Finance.Currency` | Währungen | Bank, Anlagen & Währung |
| `Microsoft.Finance.FinancialReports` | Finanzberichte | Berichte & Analyse |
| `Microsoft.Finance.Analysis` | Analyseansichten | Berichte & Analyse |
| `Microsoft.Finance.Consolidation` | Konsolidierung | Konsolidierung |
| `Microsoft.Finance.Deferral` | Abgrenzungen | Konsolidierung |
| `Microsoft.Finance.Intercompany` | IC-Partner | Konsolidierung |
| `Microsoft.Finance.AllocationAccount` | Kontierungsschlüssel | Kontenplan |
| `Microsoft.Finance.SalesTax` | US Sales Tax | MwSt-System |
| `Microsoft.Finance.Payroll` | Gehaltsimport | Querschnitt |
| `Microsoft.Finance.RoleCenters` | Rollencenter | Entwickler |

---

## 📖 Durchgängige Buchungskette (Einkauf → Fibu → Verkauf)

Jeder buchhalterisch relevante Vorgang in BC28 mündet in die Finanzbuchhaltung:

```
Einkaufsbestellung  →  Wareneingang  →  Rechnungseingang  →  Zahlung
     (Kap. 6)            (Kap. 7)          (Kap. 4)          (Kap. 4)
         │                   │                  │                │
         └──→ Gen. Posting Setup ←──┴──→ VAT Posting Setup ←───┘
                         │                        │
                    Sachkonten              MwSt-Konten
                         │                        │
                    G/L Entry               VAT Entry
                         └────────┬───────────────┘
                                  │
                          Finanzberichte
                          Account Schedules
                          UStVA / Bilanz / GuV
```

---

## 🔗 Verwandte Kapitel

| Kapitel | Fibu-Relevanz |
|---|---|
| [Kap. 5 — Vertrieb & Marketing]({{ '/05-sales-marketing/' | relative_url }}) | Debitoren, Verkaufsrechnung → `CustLedgerEntry` |
| [Kap. 6 — Einkauf]({{ '/06-purchasing/' | relative_url }}) | Kreditoren, Einkaufsrechnung → `VendLedgerEntry` |
| [Kap. 7 — Lager & Logistik]({{ '/07-inventory/' | relative_url }}) | Bestandsbewertung, Inventur → `ItemLedgerEntry` |
| [Kap. 8 — Produktion]({{ '/08-manufacturing/' | relative_url }}) | Fertigungsaufträge, WIP → `CapacityLedgerEntry` |
| [Kap. 9 — Projekte]({{ '/09-jobs/' | relative_url }}) | Projektabrechnung, WIP → `JobLedgerEntry` |
| [Kap. 10 — Service]({{ '/10-service/' | relative_url }}) | Service-Rechnung → `ServiceLedgerEntry` |
| [Kap. 11 — Personal]({{ '/11-hr/' | relative_url }}) | Gehaltsimport → Fibu-Buchungszeilen |

---

| [← Zurück zur Übersicht]({{ '/index' | relative_url }}) | [Weiter: Vertrieb & Marketing →]({{ '/05-sales-marketing/' | relative_url }}) |
