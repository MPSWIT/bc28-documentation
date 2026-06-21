---
title: "Einkauf — Preise & Rabatte"
---
# 6.3 Einkaufspreise & Rabatte

> **Tabellen:** 7012 (`Purchase Price`), 7014 (`Purchase Line Discount`), 7020 (`Purch. Invoice Disc.`)
> **Namensraum:** `Microsoft.Purchases.Pricing`

## 6.3.1 Einkaufspreise (Purchase Price, Tabelle 7012)

```al
table 7012 "Purchase Price"
{
    fields
    {
        field(1; "Vendor No."; Code[20])
        field(3; "Item No."; Code[20])
        field(16; "Direct Unit Cost"; Decimal)     // Einstandspreis
        field(19; "Starting Date"; Date)
        field(25; "Currency Code"; Code[10])
        field(30; "Minimum Quantity"; Decimal)      // Staffelmenge
    }
}
```

**Beispiel 1 — Staffeleinkaufspreis:**
> *FoodLogistics* kauft Kaffeebohnen beim Röster. Ab 100 kg 12,50 €/kg, ab 500 kg 11,20 €/kg.
> ➜ Zwei Einträge mit `Direct Unit Cost = 12.50` + `Min. Quantity = 100` und `Direct Unit Cost = 11.20` + `Min. Quantity = 500`
> **Ergebnis:** BC28 wählt automatisch den günstigeren Preis basierend auf der Bestellmenge.

## 6.3.2 Einkaufszeilenrabatte (Purchase Line Discount, Tabelle 7014)

```al
table 7014 "Purchase Line Discount"
{
    fields
    {
        field(4; "Line Discount %"; Decimal)
        field(8; "Minimum Quantity"; Decimal)
    }
}
```

## 6.3.3 Einkaufsrechnungsrabatte (Purch. Invoice Disc., Tabelle 7020)

```al
table 7020 "Purch. Invoice Disc."
{
    // Analog zu Cust. Invoice Disc. (7026)
    field(5; "Minimum Amount"; Decimal)
    field(6; "Discount %"; Decimal)
}
```

**Beispiel 2 — Lieferantenspezifischer Rechnungsrabatt:**
> *Industriebedarf Süd* erhält von seinem Hauptlieferanten 3 % Rechnungsrabatt ab 10.000 € Bestellwert.
> ➜ `Purch. Invoice Disc. = LIEF-3%`, `Min. Amount = 10.000`, `Discount % = 3`
> **Ergebnis:** Der Rabatt wird automatisch bei Rechnungsstellung abgezogen.

---

| [← Zurück zur Übersicht]({{ '/06-purchasing/' | relative_url }}) | [Kreditoren →]({{ '/06-purchasing/kreditoren/' | relative_url }}) |
