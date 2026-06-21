---
title: "Einkauf — Anforderungen"
---
# 6.5 Einkaufsanforderungen (Requisitions)

> **Tabellen:** 246 (`Requisition Line`), 242 (`Purchase Worksheet Template`)
> **Namensraum:** `Microsoft.Purchases.Requisition`

Einkaufsanforderungen (Purchase Requisitions) sind interne Bedarfsmeldungen, die vor der eigentlichen Bestellung stehen.

> 📄 **[← Zurück zur Einkaufsübersicht]({{ '/06-purchasing/' | relative_url }}/)**
> 📋 Nächstes: [Entwickler-Referenz →](entwickler)

---

## 6.5.1 Anforderungs-Arbeitsblatt (Requisition Worksheet)

```al
table 246 "Requisition Line"
{
    fields
    {
        field(1; "Template Name"; Code[10])
        field(2; "Name"; Code[10])       // Batch-Name
        field(3; "Line No."; Integer)
        field(5; "Type"; Enum)           // Item, G/L Account, ...
        field(6; "No."; Code[20])
        field(15; "Quantity"; Decimal)
        field(18; "Description"; Text[100])
        field(25; "Due Date"; Date)
        field(28; "Vendor No."; Code[20]) // Vorgeschlagener Lieferant
    }
}
```

**Beispiel 1 — Abteilungsbedarf sammeln:**
> Die IT-Abteilung der *CodeBase AG* benötigt 15 neue Monitore. Der Abteilungsleiter erfasst eine Anforderung.
> ➜ `Requisition Worksheet` → `Type = Item`, `No. = MON-27`, `Quantity = 15`, `Due Date = 01.07.`
> **Ergebnis:** Der Einkäufer sieht die Anforderung in seinem Arbeitsblatt und kann sie in eine Bestellung umwandeln.

---

## 6.5.2 Von Anforderung zur Bestellung

```
Anforderungs-Arbeitsblatt
     │
     ├──→ "Aktion → Bestellung erstellen"
     │
     └──→ Purchase Order (Tabelle 38) mit kopierten Zeilen
```

**Beispiel 2 — Sammelbestellung:**
> Drei Abteilungen haben Monitor-Anforderungen erfasst. Der Einkäufer fasst sie zu einer Bestellung zusammen.
> ➜ `Requisition Worksheet` → Filter `Item No. = MON-27` → "Carry Out Action Message" → `Create Purchase Order`
> **Ergebnis:** Eine konsolidierte Bestellung über 45 Monitore mit Mengenrabatt.

---

| [← Zurück zur Übersicht]({{ '/06-purchasing/' | relative_url }}/) | [Entwickler-Referenz →](entwickler) |
