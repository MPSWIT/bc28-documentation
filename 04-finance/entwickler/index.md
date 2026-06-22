---
title: "Entwickler-Referenz: Ereignisse, Rollencenter & Tabellen"
---
# 4. Finanzwesen — Entwickler-Referenz: Ereignisse, Rollencenter & Tabellen

> 📄 **Zurück zur [Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }})**
> 📋 Integrationsereignisse für Erweiterungen, vorkonfigurierte Rollencenter für Finanzanwender, und alle abhängigen Tabellen/Codeunits.

<pre>
4. Finanzwesen
 │
 ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}">Fibu-Einrichtung (Tab. 98)</a>
 ├── <a href="{{ '/04-finance/kontenplan-buchungsgruppen/' | relative_url }}">Kontenplan &amp; Buchungsgruppen</a>
 ├── <a href="{{ '/04-finance/mwst-system/' | relative_url }}">MwSt-System</a>
 ├── <a href="{{ '/04-finance/journale-debitoren-kreditoren/' | relative_url }}">Journale, Debitoren/Kreditoren</a>
 ├── <a href="{{ '/04-finance/bank-anlagen-waehrung/' | relative_url }}">Bank, Anlagen &amp; Währung</a>
 ├── <a href="{{ '/04-finance/berichte-analyse-budget/' | relative_url }}">Berichte, Budget &amp; Analyse</a>
 ├── <a href="{{ '/04-finance/konsolidierung-abgrenzung-ic/' | relative_url }}">Konsolidierung, Abgrenzungen &amp; IC</a>
 ├── <a href="{{ '/04-finance/querschnitt/' | relative_url }}">Querschnitt — Fibu-Relevanz aller Module</a>
 └─▶ Entwickler-Referenz  ← Sie sind hier
</pre>

---

## 4.23 Integrationsereignisse (für Entwickler)

| Ereignis | Parameter | Verwendung |
|---|---|---|
| `OnBeforeCheckRoundingError` | `var ErrorMessage: Boolean` | Zusätzliche Prüfungen vor Rundungsänderung |
| `OnAfterIsPostingAllowed` | `GLSetup, PostingDate, var Result` | Eigene Buchungsdatum-Validierung |
| `OnBeforeFirstAllowedPostingDate` | `GLSetup, var AllowedPostingDate, var IsHandled` | Eigener Algorithmus für erstes Buchungsdatum |
| `OnAfterUpdateDimValueGlobalDimNo` | `ShortcutDimNo, OldDimCode, NewDimCode` | Eigene Logik bei Dimensionsänderung |

**Abonnenten-Beispiel (AL):**
```al
codeunit 50100 "Meine Buchungsprüfung"
{
    [EventSubscriber(ObjectType::Table, Database::"General Ledger Setup",
        OnAfterIsPostingAllowed, '', false, false)]
    local procedure PrüfeBuchungsperiode(
        GeneralLedgerSetup: Record "General Ledger Setup";
        PostingDate: Date;
        var Result: Boolean)
    begin
        if PostingDate < MeineGeschäftsjahresAnfang() then
            Result := false;
    end;
}
```

---

## 4.30 Rollencenter (Finanzwesen)

> **Namensraum:** `Microsoft.Finance.RoleCenters`
> **Profile/Seiten:** `Accountant.Profile.al`, `AccountingManager.Profile.al`, `BookKeeper.Profile.al`, `Finance.Profile.al`

BC28 liefert vorkonfigurierte Rollencenter für Finanzanwender:

| Profil | Zielgruppe | Kern-Cues |
|---|---|---|
| **Buchhalter (Accountant)** | Tägliche Buchhaltung, OP-Verwaltung | `Finance Cue`: OP-Liste, Liquidität |
| **Leiter Buchhaltung** | Monatsabschluss, Kontenplan, Budget | `Account Payable Cue`: Kreditorensalden |
| **Finanzmanager** | GuV, Bilanz, Cashflow-Reporting | `Finance Performance`: KPIs |
| **Bilanzbuchhalter (Bookkeeper)** | Abschlussvorbereitung, Abstimmung | Fibu-Journale, Sachposten |

**Beispiel 1 — Ein neuer Buchhalter startet mit dem Accountant-Rollencenter:**
> Der Buchhalter meldet sich an und sieht sofort: OP-Liste (Debitoren/Kreditoren), Fibu-Journale, Bankabstimmung — alles auf einer Seite.
> ➜ Profil `Accountant` zuweisen, keine weiteren Personalisierungen.
> **Ergebnis:** Der Buchhalter findet alle täglichen Aufgaben ohne Navigation durch Menüs. Die `Finance Cue`-Kachel zeigt offene Debitorenposten als rote Warnung wenn überfällig.

**Beispiel 2 — CFO-Dashboard mit KPIs:**
> Der Finanzchef benötigt keine Journal-Seiten, sondern aktuelle KPIs: Umsatz, EBIT, Cashflow, Forderungslaufzeit.
> ➜ Profil `Finance Manager` oder `Business Manager` mit `Finance Performance`-Seite.
> **Ergebnis:** Ein Dashboard mit Diagrammen und Ampeln. Drill-Through in die Account Schedules bei Abweichungen.

---

## 4.32 Abhängige Tabellen & Codeunits

| Tabelle/Codeunit | ID | Verwendung |
|---|---|---|
| **General Ledger Setup** | 98 | Singleton-Einrichtung |
| **G/L Account** | 15 | Sachkonten, Kontenplan |
| **G/L Entry** | 17 | Gebuchte Sachposten |
| **Gen. Journal Template** | 80 | Buchungsjournal-Vorlagen |
| **Gen. Journal Batch** | 81 | Buchungsjournal-Stapel |
| **Gen. Journal Line** | 82 | Fibu-Buchungszeilen |
| **Gen. Business Posting Group** | 110 | Geschäftsbuchungsgruppen |
| **Gen. Product Posting Group** | 111 | Produktbuchungsgruppen |
| **General Posting Setup** | 252 | Buchungsmatrix (Kontenzuordnung) |
| **VAT Posting Setup** | 325 | MwSt-Buchungsmatrix |
| **VAT Entry** | 254 | Gebuchte MwSt-Posten |
| **VAT Statement** | 317 | MwSt-Abrechnung |
| **VAT Clause** | 470 | MwSt-Klauseln |
| **VAT Rate Change Setup** | 5500 | MwSt-Satzänderung |
| **Currency** | 4 | Währungen |
| **Currency Exchange Rate** | 330 | Wechselkurse |
| **Dimension** | 348 | Dimensionen |
| **Dimension Value** | 349 | Dimensionswerte |
| **Account Schedule Name** | 91 | Finanzberichts-Definitionen |
| **Account Schedule Line** | 92 | Finanzberichts-Zeilen |
| **Column Layout** | 93 | Finanzberichts-Spalten |
| **Analysis View** | 110 | Analyseansichten |
| **Analysis View Entry** | 111 | Verdichtete Analyseposten |
| **GL Budget Name** | 89 | Budget-Namen |
| **GL Budget Entry** | 90 | Budget-Einträge |
| **Deferral Header** | 1700 | Abgrenzungs-Kopf |
| **Deferral Line** | 1701 | Abgrenzungs-Zeilen |
| **Allocation Account** | 1300 | Kontierungsschlüssel |
| **IC Partner** | 410 | Intercompany-Partner |
| **IC Outbox Transaction** | 421 | IC-Ausgangstransaktionen |
| **Consolidation Setup** | 96 | Konsolidierungseinrichtung |
| **Business Unit** | 97 | Konsolidierungs-Mandanten |
| **Cust. Ledger Entry** | 21 | Debitorenposten |
| **Vendor Ledger Entry** | 25 | Kreditorenposten |
| **Bank Account** | 270 | Bankkonten |
| **Bank Acc. Reconciliation** | 273 | Bankabstimmung |
| **Fixed Asset** | 5600 | Anlagenkarte |
| **FA Depreciation Book** | 5612 | AfA-Bücher |
| **FA Ledger Entry** | 5614 | Anlagenposten |
| **Tax Area** | 162 | Sales Tax Gebiete (US/CA) |
| **Tax Jurisdiction** | 163 | Sales Tax Jurisdiktionen |
| **User Setup Management** | Codeunit | Buchungsdatum-Prüfungen |
| **GenJnlCheckLine** | Codeunit | Journal-Zeilenvalidierung |
| **GenJnlPostLine** | Codeunit | Buchungslogik |
| **Dimension Management** | Codeunit | Dimensionsset-Verwaltung |
| **Payment Tolerance Management** | Codeunit | Zahlungstoleranz-Logik |
| **DimCorrectionRun** | Codeunit | Dimensionskorrektur |
| **GenJnlApply** | Codeunit | Zahlungsausgleich |

| [← Zurück zur Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }}) | [Weiter: Vertrieb & Marketing →]({{ '/05-sales-marketing/' | relative_url }}) |

---

| [← Zurück zur Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }}) | [→ Nächstes Hauptkapitel: Vertrieb]({{ '/05-sales-marketing/' | relative_url }}) |
