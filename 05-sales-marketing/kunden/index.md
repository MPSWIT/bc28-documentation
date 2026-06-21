---
title: "Vertrieb & Marketing — Kundenverwaltung"
---
# 5.5 Kundenverwaltung (Customer Management)

> **Tabelle 18:** `Customer.Table.al` (Debitorenkarte)
> **Namensraum:** `Microsoft.Sales.Customer`, `Microsoft.Sales.Receivables`
> **Verwandte Tabellen:** 21 (`Cust. Ledger Entry`), 22 (`Customer Posting Group`)

Der Kundenstamm ist der zentrale Datensatz für jeden Geschäftspartner im Vertrieb. Er steuert Buchungsgruppen, Zahlungsbedingungen, Kreditlimit, Mahnwesen und Währung.

> 📄 **[← Zurück zur Vertriebsübersicht]({{ '/05-sales-marketing/' | relative_url }})**
> 📋 Nächstes: [Kampagnen →]({{ '/05-sales-marketing/kampagnen/' | relative_url }})

---

## 5.5.1 Debitorenkarte (Customer, Tabelle 18)

```al
table 18 Customer
{
    Caption = 'Customer';
    DataCaptionFields = "No.", Name;

    fields
    {
        field(1; "No."; Code[20])                    // Debitorennummer
        field(2; Name; Text[100])                     // Firmenname
        // Adressen
        field(5; "Address"; Text[100])
        field(41; "Post Code"; Code[20])
        field(42; City; Text[30])
        field(43; "Country/Region Code"; Code[10])
        field(91; "Ship-to Code"; Code[10])           // Lieferadresse

        // Fibu-Verknüpfung
        field(21; "Customer Posting Group"; Code[20])
        field(23; "Gen. Bus. Posting Group"; Code[20])
        field(24; "VAT Bus. Posting Group"; Code[20])

        // Zahlungsbedingungen
        field(27; "Payment Terms Code"; Code[10])
        field(28; "Payment Method Code"; Code[10])

        // Kreditwesen
        field(63; "Credit Limit (LCY)"; Decimal)
        field(25; "Invoice Disc. Code"; Code[20])     // Rechnungsrabatt-Code
        field(22; "Reminder Terms Code"; Code[10])    // Mahnmethode
        field(26; "Fin. Charge Terms Code"; Code[10]) // Zinsberechnungsmethode

        // Währung
        field(29; "Currency Code"; Code[10])

        // Dimensionen
        field(57; "Global Dimension 1 Code"; Code[20])
        field(58; "Global Dimension 2 Code"; Code[20])
    }
}
```

**Beispiel 1 — Neukunde mit Kreditlimit und Rabattstaffel:**
> Der Großkunde *Metallbau Schmidt* erhält einen Kreditrahmen von 100.000 €, Zahlungsziel 30 Tage netto und 3 % Rechnungsrabatt.
> ➜ `Customer Posting Group = INLAND`, `Credit Limit = 100.000`, `Payment Terms Code = 30T`, `Invoice Disc. Code = STAFFEL-3`
> **Ergebnis:** Das System prüft bei jeder Auftragserfassung das Kreditlimit, schlägt das Zahlungsziel vor und berechnet den Rechnungsrabatt.

**Beispiel 2 — EU-Kunde mit abweichender Lieferadresse:**
> Die *EuroTech GmbH* hat Firmensitz in Berlin, das Logistikzentrum in Polen.
> ➜ `Country/Region Code = DE`, `Ship-to Code = LOGISTIK-PL` (Adresszeilen in PL)
> **Ergebnis:** Verkaufsbelege verwenden automatisch die polnische Lieferadresse.

---

## 5.5.2 Debitorenbuchungsgruppe (Customer Posting Group, Tabelle 22)

Die `Customer Posting Group` ordnet den Debitor einer Kategorie zu, die dann in der **Buchungsmatrix** ([Gen. Posting Setup, Kap. 4]({{ '/04-finance/kontenplan-buchungsgruppen' | relative_url }})) mit der `Gen. Bus. Posting Group` und `Gen. Prod. Posting Group` kombiniert wird.

```al
table 22 "Customer Posting Group"
{
    fields
    {
        field(1; Code; Code[20])
        field(2; "Receivables Account"; Code[20])  // Debitoren-Sammelkonto
    }
}
```

Typische Gruppierungen: `INLAND`, `EU`, `DRITTLAND`, `PRIVAT`.

---

## 5.5.3 Zahlungsbedingungen (Payment Terms, Tabelle 3)

```al
table 3 "Payment Terms"
{
    fields
    {
        field(1; Code; Code[10])
        field(3; "Due Date Calculation"; DateFormula)  // z.B. 30D, CM+30D
        field(4; "Discount Date Calculation"; DateFormula)
        field(5; "Discount %"; Decimal)                 // Skonto
        field(8; "Payment Discount Excl. VAT"; Boolean) // Skonto v. Nettobetrag
    }
}
```

**Beispiel 3 — Zahlungsbedingung "30 Tage netto, 2 % Skonto bei 10 Tagen":**
> Standard-Zahlungsbedingung für Stammkunden der *BüroProfi AG*:
> ➜ `Due Date Calculation = 30D`, `Discount Date Calculation = 10D`, `Discount % = 2`
> **Ergebnis:** Fälligkeit 30 Tage nach Rechnungsdatum. Bei Zahlung innerhalb 10 Tagen 2 % Skonto.

---

## 5.5.4 Debitorenposten (Cust. Ledger Entry, Tabelle 21)

Jede gebuchte Verkaufsrechnung, Gutschrift oder Zahlung erzeugt einen **Debitorenposten**:

```al
table 21 "Cust. Ledger Entry"
{
    fields
    {
        field(3; "Customer No."; Code[20])
        field(4; "Posting Date"; Date)
        field(5; "Document Type"; Enum)        // Invoice, Credit Memo, Payment
        field(7; "Document No."; Code[20])
        field(17; "Amount"; Decimal)            // Betrag in Mandantenwährung
        field(45; "Remaining Amount"; Decimal)  // Offener Betrag
        field(74; "Open"; Boolean)              // OP-Flag
        field(15; "Due Date"; Date)
    }
}
```

Die OP-Verwaltung (`Apply Entries`) verknüpft Zahlungen mit offenen Rechnungen. Das `Appln. between Currencies`-Feld im Sales Setup (Feld 25) steuert, ob mandantenübergreifende Währungsausgleiche erlaubt sind.

---

| [← Zurück zur Übersicht]({{ '/05-sales-marketing/' | relative_url }}) | [Kampagnen →]({{ '/05-sales-marketing/kampagnen/' | relative_url }}) |
