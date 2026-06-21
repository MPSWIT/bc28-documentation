---
title: "Vertrieb & Marketing — Entwickler-Referenz"
---
# 5.7 Entwickler-Referenz

> **Namensräume:** `Microsoft.Sales.*`
> **Haupt-Codeunits:** `Sales-Post` (80), `Sales-Post + Ship` (81), `Reminder-Make` (392), `Price Calculation Mgt.` (700008)
> **Integrationsereignisse:** 20+ OnBefore/OnAfter-Events

Diese Seite fasst alle Integrationsereignisse, Codeunits und abhängigen Tabellen für Vertriebserweiterungen zusammen.

> 📄 **[← Zurück zur Vertriebsübersicht]({{ '/05-sales-marketing/' | relative_url }})**

---

## 5.7.1 Integrationsereignisse

### Sales-Post (Codeunit 80)

```al
[IntegrationEvent(false, false)]
procedure OnBeforePostSalesDoc(var SalesHeader: Record "Sales Header")
// Vor dem Buchen eines Verkaufsbelegs

[IntegrationEvent(false, false)]
procedure OnAfterPostSalesDoc(var SalesHeader: Record "Sales Header")
// Nach erfolgreicher Buchung

[IntegrationEvent(false, false)]
procedure OnBeforePostSalesDocOnBeforeCreateDim(
    var SalesHeader: Record "Sales Header"; var GenJnlLine: Record "Gen. Journal Line")
// Vor der Dimensionserstellung

[IntegrationEvent(false, false)]
procedure OnAfterCreateSalesHeader(
    var SalesHeader: Record "Sales Header"; var SalesLine: Record "Sales Line")
// Nach Erstellen eines Verkaufskopfs
```

### Reminder-Make (Codeunit 392)

```al
[IntegrationEvent(false, false)]
procedure OnBeforeCreateReminder(
    var ReminderHeader: Record "Reminder Header")

[IntegrationEvent(false, false)]
procedure OnAfterIssueReminder(
    var ReminderHeader: Record "Reminder Header"; var GenJnlLine: Record "Gen. Journal Line")
```

### PEPPOL-Export (SalesCrMemoPEPPOLBIS30, XmlPort)

```al
[IntegrationEvent(false, false)]
procedure OnBeforeCalculateLineTotals(
    var SalesCrMemoLine: Record "Sales Cr.Memo Line"; var SalesLine: Record "Sales Line")
```

## 5.7.2 Codeunit-Übersicht Sales

| ID | Codeunit | Namensraum | Beschreibung |
|---|---|---|---|
| 80 | Sales-Post | `Microsoft.Sales.Document` | Haupt-Buchungslogik Verkauf |
| 81 | Sales-Post + Ship | `Microsoft.Sales.Document` | Kombinierte Lieferung+Rechnung |
| 82 | Sales-Post (Yes/No) | `Microsoft.Sales.Document` | Interaktiver Batch-Buchungsdialog |
| 84 | Sales-Post (Yes/No) | `Microsoft.Sales.Document` | Buchungsdialog-Variante |
| 87 | Sales-Post (Print) | `Microsoft.Sales.Document` | Buchung mit Druck |
| 392 | Reminder-Make | `Microsoft.Sales.Reminder` | Mahnungserstellung |
| 396 | FinanceChargeMemo-Make | `Microsoft.Sales.FinanceCharge` | Zinsrechnungserstellung |
| 700008 | Price Calculation Mgt. | `Microsoft.Pricing` | Preisermittlung und -validierung |
| 700040 | SalesCalcDiscountByType | `Microsoft.Sales.Pricing` | Verkaufsrabatt-Berechnung |

## 5.7.3 Abhängige Tabellen (Sales-Bereich)

| ID | Tabelle | Verwendung |
|---|---|---|
| 18 | Customer | Debitoren-Stammdaten |
| 21 | Cust. Ledger Entry | Gebuchte Debitorenbewegungen |
| 22 | Customer Posting Group | Debitoren-Buchungsgruppe |
| 36 | Sales Header | Offene Verkaufsbelege |
| 37 | Sales Line | Offene Verkaufsbelegzeilen |
| 110 | Sales Shipment Header | Gebuchte Warenausgänge |
| 111 | Sales Shipment Line | Warenausgangszeilen |
| 112 | Sales Invoice Header | Gebuchte Verkaufsrechnungen |
| 113 | Sales Invoice Line | Rechnungszeilen |
| 114 | Sales Cr.Memo Header | Gebuchte Gutschriften |
| 115 | Sales Cr.Memo Line | Gutschriftzeilen |
| 293 | Reminder Terms | Mahnmethoden |
| 294 | Reminder Level | Mahnstufen |
| 296 | Reminder Header | Mahnungen |
| 297 | Reminder Line | Mahnzeilen |
| 302 | Finance Charge Memo Header | Zinsrechnungen |
| 311 | Sales & Receivables Setup | Vertriebseinrichtung |
| 5050 | Contact | CRM-Kontakte |
| 5065 | Interaction Log Entry | Interaktionen |
| 5070 | Campaign | Kampagnen |
| 5090 | Segment Header | Kundensegmente |
| 5093 | Opportunity | Verkaufschancen |
| 5107 | Sales Header Archive | Archivierte Verkaufsbelege |
| 7000 | Price List Header | Preislisten |
| 7001 | Sales Price | Klassische Verkaufspreise |
| 7004 | Sales Line Discount | Zeilenrabatte |
| 7026 | Cust. Invoice Disc. | Rechnungsrabatte |
| 6650 | Return Receipt Header | Rücksendungen |
| 5109 | Sales Comment Line | Belegkommentare |

## 5.7.4 Beispiel: ISV-Erweiterung mit OnBeforePostSalesDoc

```al
codeunit 50100 "My Sales Validation"
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Sales-Post",
        'OnBeforePostSalesDoc', '', false, false)]
    local procedure MySalesValidation(var SalesHeader: Record "Sales Header")
    begin
        // Preise unter Einkaufspreis blockieren
        SalesLine.SetRange("Document Type", SalesHeader."Document Type");
        SalesLine.SetRange("Document No.", SalesHeader."No.");
        if SalesLine.FindFirst() then
            repeat
                if SalesLine."Unit Price" < GetCostPrice(SalesLine."Item No.") then
                    Error('Verkaufspreis für %1 unter Einstandspreis.',
                        SalesLine."Item No.");
            until SalesLine.Next() = 0;
    end;
}
```

---

| [← Zurück zur Übersicht]({{ '/05-sales-marketing/' | relative_url }}) | [→ Nächstes Kapitel: Einkauf]({{ '/06-purchasing/' | relative_url }}) |
