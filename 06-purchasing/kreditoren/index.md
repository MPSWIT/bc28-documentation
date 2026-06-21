---
title: "Einkauf — Kreditorenverwaltung"
---
# 6.4 Kreditorenverwaltung (Vendor Management)

> **Tabelle 23:** `Vendor.Table.al` (Kreditorenkarte)
> **Namensraum:** `Microsoft.Purchases.Vendor`, `Microsoft.Purchases.Payables`

## 6.4.1 Kreditorenkarte (Vendor, Tabelle 23)

```al
table 23 Vendor
{
    Caption = 'Vendor';
    fields
    {
        field(1; "No."; Code[20])                     // Kreditorennummer
        field(2; Name; Text[100])
        field(21; "Vendor Posting Group"; Code[20])   // Kreditoren-Buchungsgruppe
        field(23; "Gen. Bus. Posting Group"; Code[20])
        field(27; "Payment Terms Code"; Code[10])     // Zahlungsbedingungen
        field(26; "Fin. Charge Terms Code"; Code[10])
        field(29; "Currency Code"; Code[10])
        field(34; "VAT Registration No."; Text[20])   // USt-ID
        field(40; "Payment Method Code"; Code[10])    // Zahlungsmethode
        field(57; "Global Dimension 1 Code"; Code[20])
        field(58; "Global Dimension 2 Code"; Code[20])
    }
}
```

**Beispiel 1 — EU-Lieferant mit USt-ID-Prüfung:**
> *EuroPharma GmbH* bezieht Wirkstoffe aus Spanien. Die USt-ID muss für Reverse-Charge gültig sein.
> ➜ `Vendor Posting Group = EU`, `VAT Registration No. = ESB12345678`, `Gen. Bus. Posting Group = EU`
> **Ergebnis:** Bei Buchung prüft BC28 die Kombination im `Gen. Posting Setup` ([Kap. 4]({{ '/04-finance/kontenplan-buchungsgruppen' | relative_url }})) und wendet Reverse-Charge an.

## 6.4.2 Kreditorenposten (Vendor Ledger Entry, Tabelle 25)

```al
table 25 "Vend. Ledger Entry"
{
    fields
    {
        field(4; "Posting Date"; Date)
        field(17; "Amount"; Decimal)              // Betrag MW
        field(45; "Remaining Amount"; Decimal)    // Offener Betrag
        field(74; "Open"; Boolean)
        field(15; "Due Date"; Date)
    }
}
```

Die OP-Verwaltung (`Apply Entries`) verknüpft Zahlungen mit offenen Verbindlichkeiten. Das `Appln. between Currencies`-Feld im Einkaufs-Setup steuert Währungsausgleiche.

## 6.4.3 Zahlungsvorschlag

```al
report 393 "Suggest Vendor Payments"
{
    // Berechnet fällige Zahlungen basierend auf:
    // - Fälligkeitsdatum
    // - Skontofristen (Payment Terms → Discount Date)
    // - Verfügbarer Liquidität
}
```

**Beispiel 2 — Zahllauf mit Skontooptimierung:**
> Die *BauProfi GmbH* führt wöchentlich einen Zahllauf durch und nutzt Skonti von 2–3 %.
> ➜ `Suggest Vendor Payments` → `Last Payment Date = 31.01.` → filtert alle Rechnungen mit Skontofrist bis 31.01.
> **Ergebnis:** Der Zahlungsvorschlag priorisiert Skonto-Rechnungen und vermeidet Verzugszinsen.

---

| [← Zurück zur Übersicht]({{ '/06-purchasing/' | relative_url }}) | [Anforderungen →]({{ '/06-purchasing/anforderungen/' | relative_url }}) |
