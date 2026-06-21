---
title: "Einkauf — Einkaufsbelege"
---
# 6.2 Einkaufsbelege (Purchase Documents)

> **Tabellen:** 38 (`Purchase Header`), 39 (`Purchase Line`), 120 (`Purch. Rcpt. Header`), 122 (`Purch. Inv. Header`)
> **Namensraum:** `Microsoft.Purchases.Document`, `Microsoft.Purchases.History`
> **Codeunit:** `Purch.-Post` (90)

Die Einkaufsbelege bilden das Spiegelbild zum Vertrieb und führen von der Anfrage über Bestellung und Wareneingang zur Rechnungsprüfung.

> 📄 **[← Zurück zur Einkaufsübersicht]({{ '/06-purchasing/' | relative_url }}/)**
> 📋 Nächstes: [Preise & Rabatte →](preise-rabatte)

---

## 6.2.1 Purchase Header (Tabelle 38)

```al
table 38 "Purchase Header"
{
    fields
    {
        field(1; "Document Type"; Option)
        // Quote, Order, Invoice, Credit Memo, Blanket Order, Return Order

        field(2; "No."; Code[20])
        field(3; "Buy-from Vendor No."; Code[20])
        field(4; "Pay-to Vendor No."; Code[20])
        field(21; "Vendor Invoice No."; Code[35])   // Lieferantenrechnungsnr.
        field(22; "Your Reference"; Code[35])        // Bestellnummer beim Lieferanten
        field(30; "Posting Description"; Text[100])
        field(44; "Prices Including VAT"; Boolean)   // Bruttopreise
    }
}
```

### Belegtyp-Automat

```
Anfrage → Bestellung → Wareneingang → Rechnung
   │
   └──→ Rahmenbestellung → (mehrere Bestellungen)
```

## 6.2.2 Wareneingang (Purchase Receipt)

```al
table 120 "Purch. Rcpt. Header"    // Gebuchte Wareneingänge
table 121 "Purch. Rcpt. Line"
```

Der Wareneingang ist das buchhalterische und logistische Pendant zum Warenausgang:
- Bestandszugang im Lager (→ `Item Ledger Entry`, Kap. 7)
- Einstandswert-Buchung (Cost of Goods — erwartete Kosten)
- Mengenänderung: `Qty. to Receive` → `Qty. Received` → `Outstanding Qty.`

**Beispiel 1 — Teillieferung prüfen:**
> Die *BauProfi GmbH* bestellt 500 Sack Zement. Der Lieferant liefert zunächst 200.
> ➜ `Qty. to Receive = 200` → Wareneingang buchen → Restmenge 300 bleibt offen.
> **Ergebnis:** Die Bestellung zeigt `Outstanding Qty. = 300`. Nächste Teillieferung kann gebucht werden.

## 6.2.3 Einkaufsrechnung (Purchase Invoice)

```al
table 122 "Purch. Inv. Header"    // Gebuchte Einkaufsrechnungen
table 123 "Purch. Inv. Line"
```

Beim Buchen einer Einkaufsrechnung entstehen:
- **Kreditorenposten** (`Vend. Ledger Entry`, Tabelle 25) — Verbindlichkeit
- **Sachposten** (`G/L Entry`) — Aufwand, MwSt, Bestandsveränderung

Die **Rechnungsprüfung** (`Check Doc. Total Amounts`) vergleicht Rechnungsbetrag mit Bestellsumme.

**Beispiel 2 — Rechnung vor Wareneingang:**
> *MediTech GmbH* erhält eine Vorab-Rechnung für Laborgeräte. Der Wareneingang erfolgt 3 Wochen später.
> ➜ Einkaufsrechnung direkt erfassen (ohne vorherigen Wareneingang).
> **Ergebnis:** Die Verbindlichkeit wird sofort gebucht. Der Bestandszugang erfolgt später bei Wareneingang.

## 6.2.4 Codeunit 90 "Purch.-Post"

```al
codeunit 90 "Purch.-Post"
{
    procedure Post(GenJnlLine: Record "Gen. Journal Line");

    [IntegrationEvent(false, false)]
    procedure OnBeforePostPurchDoc(var PurchHeader: Record "Purchase Header");

    [IntegrationEvent(false, false)]
    procedure OnAfterPostPurchDoc(var PurchHeader: Record "Purchase Header");
}
```

---

| [← Zurück zur Übersicht]({{ '/06-purchasing/' | relative_url }}/) | [Preise & Rabatte →](preise-rabatte) |
