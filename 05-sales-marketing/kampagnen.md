---
title: "Vertrieb & Marketing — Kampagnen"
---
# 5.6 Kampagnen & Segmente (Campaigns & Segments)

> **Tabellen:** 5070 (`Campaign`), 5090 (`Segment Header`), 5091 (`Segment Line`)
> **Namensraum:** `Microsoft.Sales.Campaign`, `Microsoft.Sales.Segment`, `Microsoft.Sales.Comment`
> **CRM-Verknüpfung:** Contacts (5050), Interactions (5065), Opportunities (5093)

Das Kampagnen-Modul verwaltet Marketing-Aktionen, Kundensegmentierung und Kundeninteraktionen.

> 📄 **[← Zurück zur Vertriebsübersicht]({{ '/05-sales-marketing/' | relative_url }}/)**
> 📋 Nächstes: [Entwickler-Referenz →](entwickler)

---

## 5.6.1 Kampagnen (Campaign, Tabelle 5070)

```al
table 5070 Campaign
{
    fields
    {
        field(1; "No."; Code[20])
        field(2; Description; Text[100])
        field(3; "Status"; Enum)               // Planned, Active, Finished
        field(10; "Starting Date"; Date)
        field(11; "Ending Date"; Date)
        field(15; "Salesperson Code"; Code[20])
        field(20; "Campaign Total Cost"; Decimal)
        field(25; "Campaign Total Sales (LCY)"; Decimal)
    }
}
```

**Beispiel 1 — Mail-Kampagne für Neukundenakquise:**
> Der Softwareanbieter *CodeBase AG* führt eine LinkedIn-Werbekampagne "Cloud2026" durch. Budget: 15.000 €.
> ➜ `Campaign = CLOUD2026`, `Campaign Total Cost = 15.000`, `Status = Active`
> **Ergebnis:** Alle aus der Kampagne resultierenden Verkaufschancen und Aufträge werden der Kampagne zugeordnet. ROI-Analyse möglich.

---

## 5.6.2 Segmente (Segment Header, Tabelle 5090)

Segmente sind dynamische oder statische Gruppen von Kontakten oder Debitoren, die für Kampagnen selektiert werden.

```al
table 5090 "Segment Header"
{
    fields
    {
        field(1; "No."; Code[20])
        field(2; Description; Text[100])
        field(5; "Segment Type"; Enum)          // Contact, Customer
        field(10; "No. of Lines"; Integer)       // Anzahl segmentierte Datensätze
    }
}

table 5091 "Segment Line"
{
    // Einzelne Kontakte/Debitoren im Segment
    // Felder: Contact No., Customer No., Salesperson Code
}
```

**Beispiel 2 — Segmentierung nach Branche und Region:**
> *Industriebedarf Süd* möchte alle Kunden der Branche "Metallverarbeitung" in PLZ-Gebiet 8xxxx für eine Kampagne ansprechen.
> ➜ Segment-Filter: `Customer → Global Dimension 2 = METALL` + `Post Code = 8*`
> ➜ Danach: Aktion "Kontakte hinzufügen"
> **Ergebnis:** 120 passende Kunden werden im Segment gesammelt und können per E-Mail oder Brief angeschrieben werden.

---

## 5.6.3 Kontakte & Interaktionen

Im CRM-Bereich ([Kap. 12]({{ '/12-crm/' | relative_url }})) werden Kontakte (`Contact`, Tabelle 5050) und Interaktionen (`Interaction Log Entry`, Tabelle 5065) verwaltet.

```al
table 5065 "Interaction Log Entry"
{
    fields
    {
        field(5; "Interaction Template Code"; Code[10])
        // Meeting, Phone Call, Email, Letter, Note...
        field(10; "Contact No."; Code[20])
        field(15; "Campaign No."; Code[20])
        field(30; "Description"; Text[100])
    }
}
```

---

## 5.6.4 Verkaufschancen (Opportunities, Tabelle 5093)

```al
table 5093 Opportunity
{
    fields
    {
        field(1; "No."; Code[20])
        field(3; "Contact No."; Code[20])
        field(5; "Salesperson Code"; Code[20])
        field(10; "Status"; Enum)              // Lead, Qualified, Proposal, Won, Lost
        field(15; "Estimated Value (LCY)"; Decimal)
        field(20; "Probability %"; Integer)
        field(25; "Campaign No."; Code[20])
    }
}
```

**Beispiel 3 — Pipeline-Management mit Verkaufschancen:**
> Der Vertriebsleiter der *Maschinenbau Wolf GmbH* trackt drei Verkaufschancen:
> - *Großauftrag Automotive*: 500.000 €, Probability 60 %, Status = Proposal
> - *Ersatzteil-Rahmenvertrag*: 80.000 €, Probability 90 %, Status = Qualified
> - *Neukunde Logistik*: 200.000 €, Probability 20 %, Status = Lead
> **Ergebnis:** Gewichteter Pipeline-Wert: 300.000 + 72.000 + 40.000 = 412.000 €

---

| [← Zurück zur Übersicht]({{ '/05-sales-marketing/' | relative_url }}/) | [Entwickler-Referenz →](entwickler) |
