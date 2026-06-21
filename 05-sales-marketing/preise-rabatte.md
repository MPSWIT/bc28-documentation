---
title: "Vertrieb & Marketing — Preise & Rabatte"
---
# 5.3 Preise & Rabatte (Sales Pricing)

> **Tabellen:** 7000 (`Price List Header`), 7001 (`Sales Price`), 7004 (`Sales Line Discount`), 7026 (`Cust. Invoice Disc.`)
> **Namensraum:** `Microsoft.Sales.Pricing`, `Microsoft.Pricing`
> **Codeunits:** `SalesCalcDiscountByType` (700040), `Price Calculation Mgt.` (700008)

BC28 bietet ein flexibles, mehrstufiges Preisfindungs- und Rabattsystem mit Preislisten, Zeilenrabatten und Rechnungsrabatten.

> 📄 **[← Zurück zur Vertriebsübersicht]({{ '/05-sales-marketing/' | relative_url }}/)**
> 📋 Nächstes: [Mahnwesen →](mahnwesen)

---

## 5.3.1 Preislisten (Price Lists)

Preislisten sind der moderne, gruppierte Ansatz zur Preisverwaltung. Sie ersetzen die klassische `Sales Price`-Tabelle nicht, sondern ergänzen sie.

```al
table 7000 "Price List Header"
{
    fields
    {
        field(1; "Code"; Code[20])               // Preislisten-Code
        field(2; "Price Type"; Enum)              // Sale / Purchase
        field(3; "Source Group"; Enum "Price Source Group")
        // Customer, Customer Price Group, Campaign, All Customers...
        field(10; "Starting Date"; Date)
        field(11; "Ending Date"; Date)
        field(30; "Status"; Enum)                 // Draft, Active, Inactive
        field(100; "Allow Updating Defaults"; Boolean)
    }
}
```

**Preislistenzeilen** (Tabelle 7002 `Price List Line`):
- `Product Type` + `Product No.` — Item, Item Category, ...
- `Unit Price` — Nettopreis
- `Line Discount %` — Zeilenrabatt direkt in der Preisliste
- `VAT Bus. Posting Gr.` — Steuerliche Einordnung
- `Minimum Quantity` — Staffelpreise

### Preisberechnungsmethoden

```al
enum "Price Calculation Method"
{
    "Lowest Price"         // Günstigster aus allen gefundenen
    "Highest Price"        // Höchster (selten im Verkauf)
    "First Matching"       // Erster Treffer in definierter Reihenfolge
    "Best Price List"      // Aus einer priorisierten Preisliste
}
```

**Beispiel 1 — Saisonale Preisliste:**
> Der Modehändler *Fashion AG* definiert eine Sommer-Aktionspreisliste für Juni–August.
> ➜ `Price List = SOMMER-AKTION`, `Starting Date = 01.06.`, `Ending Date = 31.08.`
> **Ergebnis:** Nur in diesem Zeitraum werden die reduzierten Preise gefunden. Danach greift wieder die Standard-Preisliste.

**Beispiel 2 — Staffelpreise über Minimum Quantity:**
> Der Elektronikgroßhändler *TechTrading* bietet Mengenrabatt: Ab 10 Stück 5 %, ab 50 Stück 10 %.
> ➜ Drei Preislistenzeilen für denselben Artikel mit `Minimum Quantity = 0 / 10 / 50`
> **Ergebnis:** BC28 wählt automatisch die passende Preisstaffel basierend auf der Bestellmenge.

---

## 5.3.2 Klassische Verkaufspreise (Sales Price)

```al
table 7001 "Sales Price"
{
    fields
    {
        field(1; "Sales Type"; Enum)     // Customer, Customer Price Group, Campaign, All...
        field(2; "Sales Code"; Code[20])
        field(3; "Item No."; Code[20])
        field(16; "Unit Price"; Decimal)
        field(19; "Starting Date"; Date)
        field(25; "Currency Code"; Code[10])
    }
}
```

Wichtig: Die `Price Calculation Method` in der Einrichtung (Tabelle 311, Feld 7000) bestimmt, ob BC28 die neue `Price List`-Methode oder die klassische `Sales Price`-Tabelle verwendet.

---

## 5.3.3 Zeilenrabatte (Sales Line Discounts)

```al
table 7004 "Sales Line Discount"
{
    fields
    {
        field(1; "Sales Type"; Enum)
        field(4; "Line Discount %"; Decimal)  // Prozentualer Rabatt pro Zeile
        field(8; "Minimum Quantity"; Decimal) // Ab welcher Menge?
    }
}
```

**Beispiel 3 — Kundenspezifischer Artikelrabatt:**
> *AutoParts GmbH* gewährt ihrer Werkstattkette "RepairPro" auf alle Ölfilter 15 % Rabatt ab Kartonmenge (12 Stück).
> ➜ `Sales Type = Customer`, `Sales Code = REPAIRPRO`, `Item No. = OELFILTER-123`, `Line Discount % = 15`, `Minimum Quantity = 12`
> **Ergebnis:** Sobald RepairPro mindestens 12 Ölfilter bestellt, wird der Rabatt automatisch angewendet.

---

## 5.3.4 Rechnungsrabatte (Invoice Discounts)

```al
table 7026 "Cust. Invoice Disc."
{
    fields
    {
        field(1; "Code"; Code[20])          // Verweis aus Kundenkarte
        field(2; "Currency Code"; Code[10])
        field(5; "Minimum Amount"; Decimal) // Ab welchem Rechnungsbetrag?
        field(6; "Discount %"; Decimal)     // Rabattsatz
    }
}
```

Rechnungsrabatte werden **am Kopf** berechnet, über alle Zeilen hinweg. Die Berechnungsmethode wird gesteuert durch:
- `Sales & Receivables Setup → Calc. Inv. Discount` (Feld 24)
- `Sales & Receivables Setup → Calc. Inv. Disc. per VAT ID` (Feld 30)

```al
// Berechnungsformel in SalesCalcDiscountByType (Codeunit 700040):
case SalesHeader."Invoice Discount Calculation" of
    "%":  Rechnungsbetrag * RabattProzent / 100
    "Amount":  Rabattbetrag (Fix)
```

---

## 5.3.5 Entwickler-Referenz

```al
codeunit 700008 "Price Calculation Mgt."
{
    procedure VerifyMethodImplemented(
        PriceCalcMethod: Enum "Price Calculation Method";
        PriceType: Enum "Price Type")
    // Validiert, dass die Methode für den Preistyp unterstützt wird

    procedure GetBestPrice(/* ... */): Decimal
    // Haupt-Preisermittlung
}

codeunit 700040 "SalesCalcDiscountByType"
{
    // Berechnet Zeilen- und Rechnungsrabatte
    procedure CalcLineDiscount(...)
    procedure CalcInvoiceDiscount(...)
}
```

---

| [← Zurück zur Übersicht]({{ '/05-sales-marketing/' | relative_url }}/) | [Mahnwesen →](mahnwesen) |
