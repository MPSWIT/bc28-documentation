---
title: "Intrastat — Verarbeitung & Hintergrund"
---
# 7. Intrastat — Verarbeitung &amp; Hintergrund

<pre>
7. Lager &amp; Logistik
 │
 ├── <a href="{{ '/07-inventory/' | relative_url }}">Übersicht</a>
 ├── <a href="{{ '/07-inventory/intrastat/' | relative_url }}">Intrastat</a>
 │    ├── <a href="{{ '/07-inventory/intrastat/einrichtung/' | relative_url }}">Einrichtung &amp; Setup</a>
 │    ├─▶ Verarbeitung &amp; Hintergrund  ← Sie sind hier
 │    ├── <a href="{{ '/07-inventory/intrastat/export/' | relative_url }}">Export &amp; Hochladen</a>
 │    └── <a href="{{ '/07-inventory/intrastat/entwickler/' | relative_url }}">Entwickler-Referenz</a>
</pre>

---

## `Intrastat Report Header` (Tabelle 4811)

### Felder

| Feld-Nr. | Name | Typ | Beschreibung |
|----------|------|-----|-------------|
| 1 | No. | Code[20] | Meldungsnummer (aus Nummernserie) |
| 2 | Status | Enum `Intrastat Report Status` | Open / Released |
| 3 | Description | Text[100] | Beschreibung |
| 13 | Reported | Boolean | Bereits gemeldet? (berechnet) |
| 14 | Statistics Period | Code[10] | Meldeperiode, z.B. `2406` |
| 15 | Amounts in Add. Currency | Boolean | Beträge in Zusatzwährung? |
| 16 | Currency Identifier | Code[10] | Währungskennzeichen |
| 17 | No. Series | Code[20] | Nummernserie |
| 18 | Arrivals Reported | Boolean | Eingänge gemeldet? |
| 19 | Dispatches Reported | Boolean | Ausgänge gemeldet? |
| 20 | Type | Enum `Intrastat Report Type` | Purchases / Sales |
| 21 | Periodicity | Enum `Intrastat Report Periodicity` | Month / Quarter / Year |
| 22 | File Disk No. | Code[20] | Dateinummer (fortlaufend) |
| 23 | Corrective Entry | Boolean | Korrekturmeldung? |
| 24 | EU Service | Boolean | EU-Dienstleistungen? |
| 25 | Export Date | Date | Exportdatum |
| 26 | Export Time | Time | Exportzeit |

### Status-Workflow

```
Open ──→ Released ──→ (Export)
  ↑                     │
  └── Reopen ───────────┘
```

```al
// In IntrastatReportManagement:
procedure ReleaseIntrastatReport(var IntrastatReportHeader)
begin
    IntrastatReportHeader.Status := IntrastatReportHeader.Status::Released;
    IntrastatReportHeader.Modify();
end;

procedure ReopenIntrastatReport(var IntrastatReportHeader)
begin
    if IntrastatReportHeader.Reported then
        if not Confirm(PeriodAlreadyReportedQst) then exit;
    IntrastatReportHeader.Status := IntrastatReportHeader.Status::Open;
    IntrastatReportHeader.Modify();
end;
```

Nur im Status `Open` können Zeilen hinzugefügt, geändert oder gelöscht werden (`CheckStatusOpen()`).

### `Statistics Period` Validierung

```al
field(14; "Statistics Period"; Code[10])
{
    trigger OnValidate()
    begin
        TestField(Reported, false);
        if StrLen("Statistics Period") <> 4 then
            Error(StatistiscPeriodFormatErr, FieldCaption("Statistics Period"));
        Evaluate(Month, CopyStr("Statistics Period", 3, 2));
        if (Month < 1) or (Month > 12) then
            Error(MonthNrErr);
    end;
}
```

Aus der Statistics Period wird das Startdatum berechnet:

```al
procedure GetStatisticsStartDate(): Date
var
    Century, Year: Integer;
begin
    TestField("Statistics Period");
    Century := Date2DMY(WorkDate(), 3) div 100;
    Evaluate(Year, CopyStr("Statistics Period", 1, 2));
    Year := Year + Century * 100;
    Evaluate(Month, CopyStr("Statistics Period", 3, 2));
    exit(DMY2Date(1, Month, Year));
end;
```

---

## `Intrastat Report Line` (Tabelle 4812)

### Vollständige Feldliste

| Feld-Nr. | Name | Typ | Quelle / Herkunft |
|----------|------|-----|-------------------|
| 1 | Intrastat No. | Code[20] | FK → Header |
| 2 | Line No. | Integer | Auto (Schrittweite 10000) |
| 3 | Type | Enum `Line Type` | Receipt (Quantity>0) / Shipment (Quantity<0) |
| 4 | Date | Date | Posting Date des Quell-Eintrags |
| 5 | Tariff No. | Code[20] | Item."Tariff No." oder FA."Tariff No." |
| 6 | Tariff Description | Text[250] | Lookup aus Tariff Number |
| 7 | Country/Region Code | Code[10] | Partnerland (aus Setup "Shipments Based On") |
| 8 | Transaction Type | Code[10] | Aus Belegkopf übernommen |
| 9 | Transport Method | Code[10] | Aus Belegkopf übernommen |
| 10 | Source Type | Enum `Source Type` | Item Entry / Job Entry / FA Entry |
| 11 | Source Entry No. | Integer | FK zum Quell-Eintrag |
| 12 | Net Weight | Decimal | Item."Net Weight" × Quantity |
| 13 | Amount | Decimal | Berechneter Betrag (exkl. MwSt) |
| 14 | Quantity | Decimal | Menge aus Quell-Eintrag |
| 15 | Cost Regulation % | Decimal | Nebenkosten-Prozentsatz |
| 16 | Indirect Cost | Decimal | Nebenkosten-Betrag |
| 17 | Statistical Value | Decimal | Statistischer Wert = Amount + Indirect Cost |
| 18 | Document No. | Code[20] | Belegnummer |
| 19 | Item No. | Code[20] | Artikelnummer |
| 20 | Item Name | Text[100] | Artikelbezeichnung |
| 21 | Total Weight | Decimal | Net Weight × Quantity (berechnet) |
| 22 | Supplementary Units | Boolean | Ergänzungseinheiten aktiv? |
| 23 | Internal Ref. No. | Text[10] | Interne Referenz (fortlaufend pro Gruppe) |
| 24 | Country/Region of Origin Code | Code[10] | Ursprungsland |
| 25 | Entry/Exit Point | Code[10] | Ein-/Ausgangsort |
| 26 | Area | Code[10] | Gebiet |
| 27 | Transaction Specification | Code[10] | Transaktionsspezifikation |
| 28 | Shpt. Method Code | Code[10] | Versandart |
| 29 | Partner VAT ID | Text[50] | UID-Nummer des Partners |
| 30 | Location Code | Code[10] | Standort |
| 31 | Counterparty | Boolean | Gegenpartei? |
| 32 | Correction | Boolean | Korrekturzeile? |
| 33 | Suppl. Conversion Factor | Decimal | Umrechnungsfaktor Ergänzungseinheit |
| 34 | Suppl. Unit of Measure | Text[10] | Ergänzende Mengeneinheit |
| 35 | Supplementary Quantity | Decimal | Ergänzende Menge |
| 36 | Statistical System | Enum `Stat. System` | Statistiksystem |
| 37 | Currency Code | Code[10] | Währungscode |
| 38 | Source Currency Amount | Decimal | Betrag in Quellwährung |
| 39 | Corrective entry | Boolean | Korrekturbuchung? |
| 40 | Group Code | Code[10] | Gruppencode |
| 41 | Statistics Period | Code[10] | Meldeperiode |
| 42 | Reference Period | Code[10] | Referenzperiode (bei Korrektur) |
| 43 | Payment Method | Code[10] | Zahlungsmethode |
| 44 | Corrected Intrastat Report No. | Code[20] | Korrigierte Meldung |
| 45 | Country/Region of Payment Code | Code[10] | Zahlungsland |
| 48 | Corrected Document No. | Code[20] | Korrigierte Belegnummer |
| 100 | Record ID Filter | Text[250] | Interne Record-ID (Lesezeichen) |
| 101 | Intrastat Country/Region Code | Code[10] | EU-Intrastat-Ländercode |

### Schlüssel

| Key | Felder | Verwendung |
|-----|--------|-----------|
| Key1 (PK) | Intrastat No., Line No. | Primärschlüssel |
| Key2 | Source Type, Source Entry No. | Doppelerfassungs-Prüfung |
| Key5 | Intrastat No., Type, Country, Tariff, TransType, Transport, Origin, VAT | **Data-Exchange-Gruppierung** |
| Key6-7 | Varianten mit Area, Transaction Specification | Verschiedene Gruppierungsvarianten |
| Key8 | Type, Country, VAT, TransType, Tariff, Group, Transport, Spec, Origin, Area, Corrective | Länderspezifische Gruppierung |

### Trigger-Logik

```al
trigger OnInsert()
begin
    CheckHeaderStatusOpen();  // Header muss Open sein
end;

trigger OnModify()
begin
    CheckHeaderStatusOpen();
    Correction := true;  // Jede Änderung markiert Zeile als korrigiert
end;

trigger OnDelete()
begin
    CheckHeaderStatusOpen();
    // Fehlermeldungen löschen
    ErrorMessage.SetContext(IntrastatReportHeader);
    ErrorMessage.ClearLogRec(Rec);
end;
```

---

## Report `Intrastat Report Get Lines` (4810) — Detaillierter Ablauf

### Report-Struktur

```
Report 4810 "Intrastat Report Get Lines" (ProcessingOnly)
 │
 ├── OnInitReport()
 │    └── CompanyInfo, IntrastatReportSetup laden
 │
 ├── OnPreReport()
 │    ├── Bestehende Zeilen löschen (mit Bestätigung)
 │    ├── GLSetup laden (für Zusatzwährung)
 │    └── Währungsumrechnungsfaktor berechnen
 │
 ├── dataitem("Item Ledger Entry")
 │    ├── OnPreDataItem()
 │    │    ├── Filter: Posting Date = StartDate..EndDate
 │    │    ├── Filter: Entry Type = Purchase|Sale|Transfer
 │    │    ├── Filter: Correction = false
 │    │    ├── Report Receipts/Shipments → Quantity-Filter
 │    │    └── SkipNotInvoicedEntries → Invoiced Quantity <> 0
 │    │
 │    └── OnAfterGetRecord()
 │         ├── 1. Item.Get() → sonst Skip
 │         ├── 2. Item."Exclude from Intrastat Report" → Skip
 │         ├── 3. Bereits in Intrastat Line vorhanden? → Skip
 │         ├── 4. HasCrossedBorder() → sonst Skip
 │         ├── 5. IsService() oder IsServiceItem() → Skip
 │         ├── 6. Korrigierte Einträge (ItemApplicationEntry) → Skip
 │         ├── 7. CalculateTotals() → TotalAmt berechnen
 │         ├── 8. InsertItemLedgerLine()
 │
 ├── dataitem("Job Ledger Entry")
 │    ├── OnPreDataItem()
 │    │    └── Filter: Posting Date, Type=Item, Source Code<>'', Entry Type=Usage
 │    │
 │    └── OnAfterGetRecord()
 │         ├── Item.Get(), Exclude from Intrastat → Skip
 │         ├── Land = IntrastatReportMgt.GetIntrastatBaseCountryCode()
 │         ├── Bereits in Intrastat Line? → Skip
 │         ├── IsJobService() → Skip
 │         └── InsertJobLedgerLine()
 │
 └── dataitem("FA Ledger Entry")
      ├── OnPreDataItem()
      │    ├── Filter: FA Posting Date = StartDate..EndDate
      │    ├── Filter: FA Posting Type = Acquisition Cost (Purchases) oder
      │    │                              Proceeds on Disposal (Sales)
      │    ├── Filter: Document Type = Invoice (oder Credit Memo bei Corrective)
      │    └── Filter: FA Posting Category = ' '
      │
      └── OnAfterGetRecord()
           ├── FixedAsset.Get() + Exclude → Skip
           ├── Land-Prüfung → Skip
           └── InsertFALedgerLine()
```

### Request Page

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| Starting Date | Date | Beginn des Meldezeitraums (auto aus Statistics Period) |
| Ending Date | Date | Ende des Meldezeitraums (+1M-1D) |
| Amount incl. Item Charges | Boolean | Betrag inkl. Artikelzuschläge |
| Cost Regulation % | Decimal | Nebenkosten-Prozentsatz |
| Skip Recalc for Zero Amounts | Boolean | Keine Neuberechnung bei Nullbeträgen |
| Skip Zero Amounts | Boolean | Nullbeträge nicht aufnehmen |
| Skip Non-Invoiced Entries | Boolean | Nur fakturierte Einträge |

---

## `HasCrossedBorder()` — Grenzübertrittslogik

Die zentrale Entscheidungsfunktion, ob ein Item Ledger Entry in die Intrastat-Meldung aufgenommen wird:

```al
local procedure HasCrossedBorder(ItemLedgEntry): Boolean
begin
    // 1. Partnerland ermitteln (aus Setup: Ship-to/Sell-to/Bill-to)
    Country.Get(IntrastatReportMgt.GetIntrastatBaseCountryCode(ItemLedgEntry));

    // 2. Land muss Intrastat-Code haben (= EU-Land)
    if Country."Intrastat Code" = '' then exit(false);

    // 3. Fall: Drop Shipment
    if ItemLedgEntry."Drop Shipment" then begin
        if not IntrastatReportSetup."Include Drop Shipment" then exit(false);
        if Country.Code in [CompanyInfo."Country/Region Code", ''] then exit(false);
        // Prüfung der verknüpften Applies-to Entry
    end;

    // 4. Fall: Umlagerung (Transfer)
    if ItemLedgEntry."Entry Type" = Transfer then begin
        // Prüft ob Transfer zwischen verschiedenen Ländern
        // Transfer Receipt: Gegenstelle im Inland → kein Grenzübertritt
        // Transfer Shipment: Gegenstelle im Ausland → Grenzübertritt
    end;

    // 5. Fall: Standort mit Länderbezug
    if ItemLedgEntry."Location Code" <> '' then begin
        Location.Get(ItemLedgEntry."Location Code");
        if not CountryOfOrigin(Location."Country/Region Code") then exit(false);
    end
    // 6. Fall: Purchase → Prüfung gegen Ship-to Country des Unternehmens
    else if ItemLedgEntry."Entry Type" = Purchase then
        if not CountryOfOrigin(CompanyInfo."Ship-to Country/Region Code") then exit(false);
    // 7. Fall: Sale → Prüfung gegen eigenes Land
    else if ItemLedgEntry."Entry Type" = Sale then
        if not CountryOfOrigin(CompanyInfo."Country/Region Code") then exit(false);

    exit(true);
end;
```

---

## `CalculateTotals()` — Betragsberechnung

```al
local procedure CalculateTotals(ItemLedgerEntry)
begin
    // 1. Value Entries laden (Direct Cost)
    ValueEntry.SetRange("Item Ledger Entry No.", ItemLedgerEntry."Entry No.");

    // 2. Für jeden Value Entry:
    //    - Cost Amount (Actual) summieren
    //    - Item Charges separat (IndirectCost)
    //    - Bei Zusatzwährung: ACY-Beträge verwenden

    // 3. Falls Quantity ≠ Invoiced Quantity:
    //    - Expected-Werte hinzurechnen

    // 4. Für Purchase/Transfer: Cost Amount als Basis
    //    Für Sale: Sales Amount als Basis
    //    - Falls Cost Amount = 0 → AverageCost berechnen

    // 5. Für Sale mit Amount = 0 → Unit Price aus Item-Tabelle
end;
```

---

## `InsertItemLedgerLine()` — Zeilenerzeugung

```al
local procedure InsertItemLedgerLine()
begin
    IntrastatReportLine.Init();
    IntrastatReportLine."Intrastat No." := IntrastatReportHeader."No.";
    IntrastatReportLine."Line No." += 10000;
    IntrastatReportLine.Date := "Item Ledger Entry"."Posting Date";

    // Partnerland aus Setup "Shipments Based On"
    IntrastatReportLine."Country/Region Code" :=
        IntrastatReportMgt.GetIntrastatBaseCountryCode("Item Ledger Entry");

    // Intrastat-Code des Landes (z.B. "AT" aus "Austria")
    IntrastatReportLine."Intrastat Country/Region Code" :=
        IntrastatReportMgt.GetIntrastatCodeFromCountryRegion(
            IntrastatReportLine."Country/Region Code");

    // Felder aus ItemLedgerEntry übernehmen
    IntrastatReportLine."Transaction Type" := "Item Ledger Entry"."Transaction Type";
    IntrastatReportLine."Transport Method" := "Item Ledger Entry"."Transport Method";
    IntrastatReportLine."Source Entry No." := "Item Ledger Entry"."Entry No.";
    IntrastatReportLine.Quantity := "Item Ledger Entry".Quantity;
    IntrastatReportLine."Document No." := "Item Ledger Entry"."Document No.";
    IntrastatReportLine."Item No." := "Item Ledger Entry"."Item No.";
    IntrastatReportLine."Entry/Exit Point" := "Item Ledger Entry"."Entry/Exit Point";
    IntrastatReportLine.Area := "Item Ledger Entry".Area;
    IntrastatReportLine."Transaction Specification" :=
        "Item Ledger Entry"."Transaction Specification";
    IntrastatReportLine."Shpt. Method Code" := "Item Ledger Entry"."Shpt. Method Code";
    IntrastatReportLine."Location Code" := "Item Ledger Entry"."Location Code";

    // Betrag in Meldewährung
    if IntrastatReportHeader."Amounts in Add. Currency" then
        IntrastatReportLine.Amount := Round(Abs(TotalAmt),
            AddCurrency."Amount Rounding Precision", AmtRoundingDirection)
    else
        IntrastatReportLine.Amount := Round(Abs(TotalAmt),
            GLSetup."Amount Rounding Precision", AmtRoundingDirection);

    // Type aus Quantity
    if IntrastatReportLine.Quantity < 0 then
        IntrastatReportLine.Type := IntrastatReportLine.Type::Shipment
    else
        IntrastatReportLine.Type := IntrastatReportLine.Type::Receipt;

    // Item validieren → zieht Tariff No., Net Weight, Suppl. UOM
    IntrastatReportLine.Validate("Item No.");
    IntrastatReportLine.Validate("Source Type",
        IntrastatReportLine."Source Type"::"Item Entry");

    // Nebenkosten
    if AmountInclItemCharges then
        IntrastatReportLine.Validate("Cost Regulation %", IndirectCostPctReq);

    IntrastatReportLine.Insert();
end;
```

### Zeile = `Partner VAT ID` automatisch setzen? (Get Partner VAT For)

```al
// In IntrastatReportLine."Source Type".OnValidate():
IntrastatReportSetup.GetSetup();
if ((Type = Type::Shipment) and
    (IntrastatReportSetup."Get Partner VAT For" <>
     IntrastatReportSetup."Get Partner VAT For"::Receipt)) or
   ((Type = Type::Receipt) and
    (IntrastatReportSetup."Get Partner VAT For" <>
     IntrastatReportSetup."Get Partner VAT For"::Shipment))
then
    "Partner VAT ID" := GetPartnerID();
```

---

## `GetPartnerID()` — Partner-VAT-ID-Ermittlung

Die VAT-ID wird mehrstufig ermittelt:

```al
procedure GetPartnerID(): Text[50]
begin
    case "Source Type" of
        "Source Type"::"Item Entry": exit(GetPartnerIDFromItemEntry());
        "Source Type"::"Job Entry":  exit(GetPartnerIDFromJobEntry());
        "Source Type"::"FA Entry":   exit(GetPartnerIDFromFAEntry());
    end;
end;
```

### Für Item Entry:

```al
// 1. ItemLedgerEntry laden
// 2. Beleg-Dokument ermitteln (SalesHeader / PurchaseHeader)

// 3a. Verkauf: GetVATNoOrSource(Sales, DocumentNo, DocRecRef)
//     → Setup."Sales VAT No. Based On":
//       - Document: VAT Registration No. aus Belegkopf
//       - Sell-to VAT: Debitor aus Sell-to Customer No.
//       - Bill-to VAT: Debitor aus Bill-to Customer No.

// 3b. Einkauf: GetVATNoOrSource(Purchases, DocumentNo, DocRecRef)
//     → Setup."Purchase VAT No. Based On":
//       - Document: VAT Registration No. aus Belegkopf
//       - Buy-from VAT: Kreditor aus Buy-from Vendor No.
//       - Pay-to VAT: Kreditor aus Pay-to Vendor No.

// 4. Fallback: Kreditor/Debitor aus ItemLedgerEntry."Source No."
//    → VAT Registration No. mit GetVATRegNo() formatieren

// 5. GetPartnerIDForCountry() als letzter Schritt:
//    - Private Person → Def. Private Person VAT No.
//    - 3-Party Trade → Def. 3-Party Trade VAT No.
//    - EU-Land mit VAT → VAT Registration No.
//    - Sonst → Def. VAT for Unknown State
```

---

## `GetCountryOfOriginCode()` — Ursprungsland

```al
procedure GetCountryOfOriginCode(): Code[10]
begin
    // 1. FA Entry → FixedAsset."Country/Region of Origin Code"

    // 2. Item Entry / Job Entry:
    //    a. Serial No. → SerialNoInformation."Country/Region Code"
    //    b. Lot No. → LotNoInformation."Country/Region Code"
    //    c. Package No. → PackageNoInformation."Country/Region Code"
    //    d. Fallback → Item."Country/Region of Origin Code"

    // 3. Letzter Fallback → Company Information."Country/Region Code"
end;
```

Im AT-Modul wird diese Logik überschrieben:

```al
// AT: Bei Receipt → Origin Country = Item Origin,
//     wenn = eigenes Land → Partnerland
if Type = Type::Receipt then begin
    if (ItemFAOrigCountryCode = CompanyInfo."Ship-to Country/Region Code") or
       (ItemFAOrigCountryCode = '')
    then
        "Country/Region of Origin Code" := "Country/Region Code"
    else
        "Country/Region of Origin Code" := ItemFAOrigCountryCode;
end;
```

---

## `RecalculateWaightAndSupplUOMQty()` — Neuberechnung

```al
procedure RecalculateWaightAndSupplUOMQty(IntrastatReportHeader)
begin
    // Menü: Weight, Supplemental UOM Qty, Both
    Selection := StrMenu(UpdateSelectionQst, 3);

    // Für jede Zeile:
    //   Item/FixedAsset laden
    //   Net Weight / Suppl. UOM neu setzen
    //   Zeile modify(true)
end;
```
