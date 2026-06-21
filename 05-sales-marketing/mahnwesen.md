---
title: "Vertrieb & Marketing — Mahnwesen"
---
# 5.4 Mahnwesen (Reminders & Finance Charges)

> **Tabellen:** 296 (`Reminder Header`), 297 (`Reminder Line`), 302 (`Finance Charge Memo Header`)
> **Namensraum:** `Microsoft.Sales.Reminder`, `Microsoft.Sales.FinanceCharge`
> **Zugehörig:** `Reminder Terms`, `Reminder Levels`, `Finance Charge Terms`

Das Mahnwesen automatisiert die Überwachung überfälliger Debitorenposten und erstellt gestaffelte Mahnungen sowie Zinsrechnungen.

> 📄 **[← Zurück zur Vertriebsübersicht]({{ '/05-sales-marketing/' | relative_url }}/)**
> 📋 Nächstes: [Kundenverwaltung →](kunden)

---

## 5.4.1 Mahnmethoden & Mahnstufen

### Mahnmethode (Reminder Terms, Tabelle 293)

Jeder Debitor hat eine **Mahnmethode**, die die maximale Anzahl Mahnstufen, den Mindestbetrag und die Gebühren definiert.

```al
table 293 "Reminder Terms"
{
    fields
    {
        field(1; "Code"; Code[10])
        field(2; "Description"; Text[50])
        field(4; "Max. No. of Reminders"; Integer)  // Max. Mahnstufen
        field(10; "Grace Period"; DateFormula)       // Toleranztage (z.B. 3D)
    }
}
```

### Mahnstufe (Reminder Level, Tabelle 294)

Definiert pro Stufe:
- **Zinsberechnung**: `Calculate Interest` (Boolean) + `Interest Rate` (Decimal)
- **Zusätzliche Gebühr**: `Additional Fee` — fester Betrag pro Mahnung
- **Buchungsgruppe**: `Gen. Bus. Posting Group` für die Zinsertrags-Buchung

```al
table 294 "Reminder Level"
{
    fields
    {
        field(3; "Grace Period"; DateFormula)   // Tage zwischen Mahnungen
        field(4; "Calculate Interest"; Boolean)
        field(5; "Interest Rate"; Decimal)
        field(8; "Additional Fee"; Decimal)
    }
}
```

**Beispiel 1 — Dreistufiges Mahnwesen mit Zinsen:**
> Der Großhändler *FoodLogistics* mahnt in drei Stufen:
> - **Stufe 1 — Zahlungserinnerung**: Grace Period 7D, keine Zinsen, keine Gebühr
> - **Stufe 2 — 1. Mahnung**: Grace Period 14D, 5 % Zinsen p.a., 5 € Gebühr
> - **Stufe 3 — 2. Mahnung (letzte)**: Grace Period 30D, 9 % Zinsen p.a., 15 € Gebühr
> **Ergebnis:** Der `Create Reminders`-Batch-Job (Bericht 109 "Reminder") erstellt automatisch Mahnungen basierend auf dem Fälligkeitsdatum jedes Debitorenpostens.

---

## 5.4.2 Mahnung erstellen (Reminder Header)

```al
table 296 "Reminder Header"
{
    fields
    {
        field(1; "No."; Code[20])                  // Mahnungsnummer
        field(2; "Customer No."; Code[20])
        field(10; "Reminder Level"; Integer)        // Aktuelle Mahnstufe
        field(15; "Posting Date"; Date)
        field(18; "Document Date"; Date)
        field(100; "Interest Amount"; Decimal)      // Berechnete Zinsen
        field(101; "Additional Fee"; Decimal)       // Mahngebühr
        field(102; "Total Amount"; Decimal)         // Gesamtforderung
    }
}
```

Jede Mahnungszeile (Tabelle 297 `Reminder Line`) referenziert einen konkreten `Cust. Ledger Entry` — den überfälligen Posten.

**Beispiel 2 — Mahnlauf mit Mindestbetrag:**
> *MediTech GmbH* stellt monatliche Rechnungen an Krankenhäuser. Mahnungen unter 50 € werden unterdrückt.
> ➜ `Reminder Terms → Min. Amount = 50`
> **Ergebnis:** Die automatische Mahnerstellung ignoriert Kleinbeträge und vermeidet unnötige Portokosten.

---

## 5.4.3 Zinsrechnung (Finance Charge Memo)

Unabhängig von Mahnungen können **Zinsrechnungen** erstellt werden:

```al
table 302 "Finance Charge Memo Header"
{
    fields
    {
        field(1; "No."; Code[20])
        field(2; "Customer No."; Code[20])
        field(100; "Interest Amount"; Decimal)
        field(103; "Interest Rate"; Decimal)
    }
}
```

**Beispiel 3 — Zinsrechnung ohne Mahnung:**
> Die *BauProfi GmbH* vereinbart mit einem Kunden eine Ratenzahlung. Die Verzugszinsen (6 % p.a.) werden vierteljährlich per Zinsrechnung abgerechnet.
> ➜ `Finance Charge Terms` einrichten → `Create Finance Charge Memos` (Bericht 110)
> **Ergebnis:** Separate Zinsrechnungen, die nicht als Mahnung zählen, sondern als eigenständige Forderungen.

---

## 5.4.4 Buchung von Mahnungen

Beim Registrieren (`Issue`) einer Mahnung entstehen:
- Debitorenposten für Zinsen und Gebühren
- Sachposten für Zinserträge (`Gen. Bus. Posting Group`)
- Die Mahnstufe des Debitors wird hochgesetzt

Die Buchungsjournal-Verknüpfung erfolgt über:
- `Reminder Journal Template Name` (Feld 208 in Sales Setup)
- `Reminder Journal Batch Name` (Feld 209)

---

## 5.4.5 Entwickler-Referenz

```al
codeunit 392 "Reminder-Make"
{
    // Erstellt Mahnvorschläge aus überfälligen Cust. Ledger Entries
    procedure CreateReminder(...)
}

codeunit 396 "FinanceChargeMemo-Make"
{
    procedure CreateFinChargeMemo(...)
}

// Integrationsereignisse
[IntegrationEvent(false, false)]
procedure OnBeforeCreateReminder(ReminderHeader: Record "Reminder Header")

[IntegrationEvent(false, false)]
procedure OnAfterIssueReminder(ReminderHeader: Record "Reminder Header")
```

---

| [← Zurück zur Übersicht]({{ '/05-sales-marketing/' | relative_url }}/) | [Kundenverwaltung →](kunden) |
