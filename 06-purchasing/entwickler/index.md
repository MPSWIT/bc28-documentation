---
title: "Einkauf — Entwickler-Referenz"
---
# 6.6 Entwickler-Referenz

> **Namensräume:** `Microsoft.Purchases.*`
> **Haupt-Codeunits:** `Purch.-Post` (90), `Purch.-Post + Print` (93)
> **Integrationsereignisse:** OnBeforePostPurchDoc, OnAfterPostPurchDoc

> 📄 **[← Zurück zur Einkaufsübersicht]({{ '/06-purchasing/' | relative_url }})**

---

## 6.6.1 Integrationsereignisse

```al
// Codeunit 90 "Purch.-Post"
[IntegrationEvent(false, false)]
procedure OnBeforePostPurchDoc(
    var PurchHeader: Record "Purchase Header")

[IntegrationEvent(false, false)]
procedure OnAfterPostPurchDoc(
    var PurchHeader: Record "Purchase Header")

[IntegrationEvent(false, false)]
procedure OnBeforePostPurchDocOnBeforeCreateDim(
    var PurchHeader: Record "Purchase Header";
    var GenJnlLine: Record "Gen. Journal Line")
```

## 6.6.2 Codeunit-Übersicht

| ID | Codeunit | Namensraum | Beschreibung |
|---|---|---|---|
| 90 | Purch.-Post | `Microsoft.Purchases.Document` | Haupt-Buchungslogik Einkauf |
| 92 | Purch.-Post (Yes/No) | `Microsoft.Purchases.Document` | Batch-Dialog |
| 93 | Purch.-Post + Print | `Microsoft.Purchases.Document` | Buchung mit Druck |

## 6.6.3 Abhängige Tabellen

| ID | Tabelle | Verwendung |
|---|---|---|
| 23 | Vendor | Kreditorenstamm |
| 25 | Vend. Ledger Entry | Kreditorenposten |
| 26 | Vendor Posting Group | Kreditoren-Buchungsgruppe |
| 38 | Purchase Header | Offene Einkaufsbelege |
| 39 | Purchase Line | Einkaufsbelegzeilen |
| 120 | Purch. Rcpt. Header | Gebuchte Wareneingänge |
| 122 | Purch. Inv. Header | Gebuchte Einkaufsrechnungen |
| 312 | Purchases & Payables Setup | Einkaufseinrichtung |
| 246 | Requisition Line | Anforderungs-Arbeitsblatt |
| 7012 | Purchase Price | Einkaufspreise |
| 7014 | Purchase Line Discount | Einkaufszeilenrabatte |
| 7020 | Purch. Invoice Disc. | Einkaufsrechnungsrabatte |

## 6.6.4 Beispiel: Validierung vor Einkaufsbuchung

```al
codeunit 50101 "My Purchase Validation"
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Purch.-Post",
        'OnBeforePostPurchDoc', '', false, false)]
    local procedure ValidateMinOrderValue(
        var PurchHeader: Record "Purchase Header")
    begin
        if PurchHeader."Document Type" = PurchHeader."Document Type"::Order then
            if PurchHeader.Amount < 100 then
                Error('Mindestbestellwert 100 € unterschritten.');
    end;
}
```

---

| [← Zurück zur Übersicht]({{ '/06-purchasing/' | relative_url }}) | [→ Nächstes Kapitel: Lager]({{ '/07-inventory/' | relative_url }}) |
