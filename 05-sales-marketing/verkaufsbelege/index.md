---
title: "Vertrieb & Marketing — Verkaufsbelege"
---
# 5.2 Verkaufsbelege (Sales Documents)

> **Tabellen:** 36 (`Sales Header`), 37 (`Sales Line`), 110 (`Sales Shipment Header`), 112 (`Sales Invoice Header`)
> **Namensraum:** `Microsoft.Sales.Document`, `Microsoft.Sales.History`
> **Belegtypen:** Quote, Order, Invoice, Credit Memo, Return Order, Blanket Order

Die Verkaufsbelege bilden den Kern des Sales-Prozesses und führen vom Angebot über Auftrag und Lieferung bis zur Fakturierung und Retoure.

> 📄 **[← Zurück zur Vertriebsübersicht]({{ '/05-sales-marketing/' | relative_url }})**
> 📋 Nächstes: [Preise & Rabatte →]({{ '/05-sales-marketing/preise-rabatte/' | relative_url }})

---

## 5.2.1 Sales Header (Tabelle 36)

Der **Sales Header** ist die zentrale Tabelle für alle aktiven Verkaufsbelege. Jeder Belegtyp wird über das Feld `Document Type` unterschieden.

```al
table 36 "Sales Header"
{
    DataCaptionFields = "No.", "Sell-to Customer Name";

    fields
    {
        field(1; "Document Type"; Option)
        {
            OptionMembers = Quote,Order,Invoice,"Credit Memo","Blanket Order","Return Order";
        }
        field(2; "No."; Code[20])           // Belegnummer aus Nummernserie
        field(3; "Sell-to Customer No."; Code[20])
        field(4; "Bill-to Customer No."; Code[20])
        // ... Status, Dates, Amounts, Shipment, VAT, Currency
    }
}
```

### Belegtyp-Automat

```
Angebot → Auftrag → Warenausgang → Rechnung
   │                    │
   └──→ Rahmenauftrag ←─┘
   
   Retoure → Rücksendung → Gutschrift
```

**Belegtypen im Überblick:**

| Typ | Englisch | Codeunit (Buchung) | Entsteht aus | Wird zu |
|---|---|---|---|---|
| Angebot | Quote | — | Neu / Kopie | Auftrag |
| Rahmenauftrag | Blanket Order | — | Neu | Auftrag(e) |
| Auftrag | Order | `Sales-Post` | Angebot / Rahmenauftrag | Warenausgang + Rechnung |
| Rechnung | Invoice | `Sales-Post` | Auftrag / Direkt | Gebuchte Rechnung |
| Gutschrift | Credit Memo | `Sales-Post` | Retoure / Direkt | Gebuchte Gutschrift |
| Retoure | Return Order | `Sales-Post` | Neu | Rücksendung + Gutschrift |

### Wichtige Statusfelder

```al
// Dokumentstatus steuert den Lebenszyklus
field(102; "Status"; Enum "Sales Document Status")
{
    // Open → Released → Pending Approval → Pending Prepayment → ...
}

// Mengensteuerung
field(47; "Shipment Method Code")
field(175; "Shipping Agent Service Code")
```

**Beispiel 1 — Vom Angebot zur Rechnung:**
> Der Außendienst der *BauProfi GmbH* erstellt im CRM ein Angebot über Baustoffe für 50.000 €. Der Kunde bestätigt telefonisch.
> ➜ `Document Type = Quote` → Funktion "Auftrag erstellen" → `Document Type = Order`
> ➜ Warenausgang buchen → Rechnung buchen
> **Ergebnis:** Durchgängiger Prozess mit automatisierter Nummernvergabe (`Quote Nos. → Order Nos. → Invoice Nos. → Posted Invoice Nos.`)

**Beispiel 2 — Teillieferung aus Rahmenauftrag:**
> Der Getränkegroßhändler *DrinkStar* hat einen Rahmenauftrag über 10.000 Kisten über 12 Monate.
> ➜ `Blanket Order` → Monatlich `Order` mit Teilmenge 850 Kisten erzeugen → Liefern → Fakturieren
> **Ergebnis:** Der Rahmenauftrag behält den Überblick über abgerufene und verbleibende Mengen.

---

## 5.2.2 Sales Line (Tabelle 37)

Jeder Verkaufsbeleg-Kopf hat beliebig viele Zeilen. Zeilen können verschiedene Typen haben:

```al
table 37 "Sales Line"
{
    fields
    {
        field(5; "Type"; Enum "Sales Line Type")
        {
            // Item, G/L Account, Resource, Fixed Asset, Charge (Item)
        }
        field(6; "No."; Code[20])         // Artikelnummer / Sachkontonr.
        field(15; "Quantity"; Decimal)     // Menge
        field(22; "Unit Price"; Decimal)   // Einzelpreis
        field(28; "Line Discount %"; Decimal)
        field(36; "Line Amount"; Decimal)  // Zeilenbetrag (berechnet)
        // ... Reservierung, Tracking, Dimensionen, MwSt
    }
}
```

### Pricing-Fluss in der Zeile

```
1. Artikel ausgewählt → Item-Tabelle (VK-Preis, Einstandspreis)
2. Preisliste greift → Price Calculation Method (aus Setup)
3. Zeilenrabatt % → Line Discount % (aus Kunden-/Artikelrabatten)
4. Rechnungsrabatt → Invoice Discount (am Kopf, nach Subtotal)
5. MwSt → VAT Posting Setup (Kap. 4)
```

---

## 5.2.3 Warenausgang & Lieferung

```al
table 110 "Sales Shipment Header"
{
    Caption = 'Sales Shipment Header';
    // Enthält Kopie aller Kopf-Felder zum Zeitpunkt der Lieferung
}

table 111 "Sales Shipment Line"
{
    // Zeilen: Qty. Shipped, Item-Tracking, Lagerplatz
}
```

Der **Warenausgang** ist die buchhalterische und logistische Bestätigung, dass Ware das Lager verlassen hat. Er löst aus:
- Bestandsabgang im Lager (→ `Item Ledger Entry`, Kap. 7)
- COGS-Buchung (Cost of Goods Sold, sofern die Nachkalkulation dies vorsieht)
- Ggf. `Exact Cost Reversing` bei Retouren

**Beispiel 3 — Kommissionierung und Teillieferung:**
> Ein Auftrag über 100 Stück kann in zwei Lieferungen à 50 Stück ausgeführt werden.
> ➜ `Qty. to Ship = 50` → Warenausgang buchen → `Posted Shipment`
> ➜ Restmenge bleibt im Auftrag als `Outstanding Quantity`.

---

## 5.2.4 Fakturierung

```al
table 112 "Sales Invoice Header"
{
    Caption = 'Sales Invoice Header';
    // Gebuchte Verkaufsrechnung — faktisch unveränderlich
}

table 113 "Sales Invoice Line"
```

Beim Buchen einer Verkaufsrechnung entstehen:
- **Debitorenposten** (`Cust. Ledger Entry`, Tabelle 21) — offene Forderung
- **Sachposten** (`G/L Entry`, Tabelle 17) — Erlöse, MwSt, COGS
- **MwSt-Posten** (`VAT Entry`, Tabelle 254) — Steuerverbindlichkeit
- **Gebuchte Verkaufsrechnung** (`Sales Invoice Header`, Tabelle 112) — unveränderlicher Beleg

Die Buchungslogik ist in `codeunit 80 "Sales-Post"` implementiert.

---

## 5.2.5 Gutschrift & Retoure

Retouren durchlaufen den umgekehrten Pfad:
1. `Return Order` (Reklamation) → `Return Receipt` (Rücksendung / Wareneingang) → `Credit Memo` (Gutschrift)
2. `Exact Cost Reversing` stellt sicher, dass die exakte Kostenbasis der Originalbuchung verwendet wird

```al
field(6602; "Exact Cost Reversing Mandatory"; Boolean)
// Zwingt: Appl.-from Item Entry muss gesetzt sein
```

---

## 5.2.6 Entwickler-Referenz: Codeunit 80 "Sales-Post"

```al
codeunit 80 "Sales-Post"
{
    // Haupt-Einstiegspunkt für alle Sales-Buchungen
    procedure Post(GenJnlLine: Record "Gen. Journal Line");

    // Integrationsereignisse
    [IntegrationEvent(false, false)]
    procedure OnBeforePostSalesDoc(SalesHeader: Record "Sales Header");

    [IntegrationEvent(false, false)]
    procedure OnAfterPostSalesDoc(SalesHeader: Record "Sales Header");
}
```

**Abhängige Codeunits:**
- `Sales-Post + Ship` (Codeunit 81) — Kombinierte Buchung Lieferung + Rechnung
- `Sales-Post (Yes/No)` (Codeunit 84) — Batch-Buchungsdialog ohne Benutzerinteraktion
- `Sales-Post (Print)` (Codeunit 87) — Buchung mit Rechnungsdruck

---

| [← Zurück zur Übersicht]({{ '/05-sales-marketing/' | relative_url }}) | [Preise & Rabatte →]({{ '/05-sales-marketing/preise-rabatte/' | relative_url }}) |
