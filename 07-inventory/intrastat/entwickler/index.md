---
title: "Intrastat — Entwickler-Referenz"
---
# 7. Intrastat — Entwickler-Referenz

<pre>
7. Lager &amp; Logistik
 │
 ├── <a href="{{ '/07-inventory/' | relative_url }}">Übersicht</a>
 ├── <a href="{{ '/07-inventory/intrastat/' | relative_url }}">Intrastat</a>
 │    ├── <a href="{{ '/07-inventory/intrastat/einrichtung/' | relative_url }}">Einrichtung &amp; Setup</a>
 │    ├── <a href="{{ '/07-inventory/intrastat/verarbeitung/' | relative_url }}">Verarbeitung &amp; Hintergrund</a>
 │    ├── <a href="{{ '/07-inventory/intrastat/export/' | relative_url }}">Export &amp; Hochladen</a>
 │    └─▶ Entwickler-Referenz  ← Sie sind hier
</pre>

---

## Integration Events (IntrastatReportManagement, 4810)

### Export-Pipeline

| Event | Parameter | Beschreibung |
|-------|-----------|-------------|
| `OnBeforeExportIntrastatHeader` | `IntrastatReportHeader, var IsHandled` | Vor Exportbeginn |
| `OnBeforeDefineFileNames` | `IntrastatReportHeader, var FileName, ReceiptFileName, ShipmentFileName, ZipFileName, var IsHandled` | Dateinamen anpassen |
| `OnExportWithDataExchOnAfterGetIntrastatReportSetup` | `IntrastatReportSetup, IntrastatReportHeader` | Nach Setup-Laden |
| `OnBeforeExportIntrastatReportLines` | `var IntrastatReportLine, IntrastatReportHeader` | Vor Zeilenexport |
| `OnBeforeExportOneDataExchangeDef` | `IntrastatReportHeader, DataExchDefCode, ExportType, var DataExch, var IsHandled` | Vor Data-Exchange-Export |
| `OnBeforeExportToZip` | `DataExch1, DataExch2, StatisticsPeriod, FileName, ReceiptFileName, ShipmentFileName, ZipFileName, var IsHandled` | Vor ZIP-Erstellung |
| `OnAfterExportIntrastatHeader` | `IntrastatReportHeader, var IsHandled` | Nach Export |

### Freigabe-Pipeline

| Event | Parameter | Beschreibung |
|-------|-----------|-------------|
| `OnBeforeReleaseIntrastatHeader` | `IntrastatReportHeader, var IsHandled` | Vor Freigabe |
| `OnAfterReleaseIntrastatHeader` | `IntrastatReportHeader` | Nach Freigabe |
| `OnBeforeReopenIntrastatHeader` | `IntrastatReportHeader, var IsHandled` | Vor Wiedereröffnung |
| `OnAfterReopenIntrastatHeader` | `IntrastatReportHeader` | Nach Wiedereröffnung |

### Setup-Initialisierung

| Event | Parameter | Beschreibung |
|-------|-----------|-------------|
| `OnBeforeInitSetup` | `var IntrastatReportSetup, var IsHandled` | Vor Setup-Init (AT überschreibt hier) |
| `OnBeforeInitCheckList` | `var IsHandled` | Vor Checklist-Init |
| `OnBeforeCreateDefaultDataExchangeDef` | `var IsHandled` | Vor Data-Exchange-Def-Import |

### Währung &amp; Beträge

| Event | Parameter | Beschreibung |
|-------|-----------|-------------|
| `OnGetOriginalCurrencyFromItemLedgerElseCase` | `var ItemLedgerEntry, var CurrencyCode` | Währung für sonstige Belegtypen |
| `OnGetIntrastatBaseCountryCodeFromItemLedgerElseCase` | `ItemLedgEntry, IntrastatReportSetup, var CountryCode` | Land für sonstige Belegtypen |
| `OnRecalculateWeightAndSupplUOMQtyOnAfterCalculateNetWeight` | `IntrastatReportLine, var NetWeight` | Net Weight anpassen |
| `OnGetCompanyVATRegNoOnAfterGetIntrastatReportSetup` | `CompanyInformation, IntrastatReportSetup` | Eigene VAT-ID anpassen |

---

## Integration Events (IntrastatReportLine, Tabelle 4812)

| Event | Parameter | Beschreibung |
|-------|-----------|-------------|
| `OnBeforeGetItemDescription` | `var IsHandled, var IntrastatReportLine` | Tarifbeschreibung anpassen |
| `OnBeforeGetCountryOfOriginCode` | `var IntrastatReportLine, var CountryOfOriginCode, var IsHandled` | Ursprungsland-Logik überschreiben |
| `OnAfterGetCountryOfOriginCode` | `var IntrastatReportLine, var CountryOfOriginCode` | Nach Ursprungsland-Ermittlung |
| `OnBeforeGetPartnerID` | `var IntrastatReportLine, var PartnerID, var IsHandled` | VAT-ID-Ermittlung überschreiben |
| `OnBeforeGetPartnerIDFromItemEntry` | `var IntrastatReportLine, var PartnerID, var IsHandled` | VAT-ID aus Item Entry |
| `OnBeforeGetCustomerPartnerIDFromItemEntry` | `var Customer, EU3rdPartyTrade, var PartnerID, var IsHandled` | Debitor-VAT anpassen |
| `OnBeforeGetVendorPartnerIDFromItemEntry` | `var Vendor, var PartnerID, var IsHandled` | Kreditor-VAT anpassen |
| `OnBeforeGetPartnerIDFromJobEntry` | `var IntrastatReportLine, var PartnerID, var IsHandled` | VAT aus Projekt |
| `OnBeforeGetCustomerPartnerIDFromJobEntry` | `var Customer, var PartnerID, var IsHandled` | Kunden-VAT aus Projekt |
| `OnBeforeGetPartnerIDFromFAEntry` | `var IntrastatReportLine, var PartnerID, var IsHandled` | VAT aus Anlage |
| `OnBeforeGetVendorPartnerIDFromFAEntry` | `var Vendor, var PartnerID, var IsHandled` | Kreditor-VAT aus Anlage |
| `OnBeforeGetCustomerPartnerIDFromFAEntry` | `var Customer, var PartnerID, var IsHandled` | Debitor-VAT aus Anlage |
| `OnBeforeGetPartnerIDForCountry` | `CountryRegionCode, VATRegistrationNo, IsPrivatePerson, IsThirdPartyTrade, var PartnerID, var IsHandled` | Finale VAT-ID (AT überschreibt) |
| `OnBeforeCheckDateInRange` | `var IntrastatReportLine, var IsHandled` | Datumsprüfung |
| `OnAfterSetIntrastatReportHeaderFilters` | `var IntrastatReportHeaderWithFiltersSet, IntrastatReportHeaderForLine` | Filter für korrigierte Meldungen |

---

## Integration Events (IntrastatReportHeader, Tabelle 4811)

| Event | Parameter | Beschreibung |
|-------|-----------|-------------|
| `OnBeforeCheckStatusOpen` | `xIntrastatReportHeader, IntrastatReportHeader, var IsHandled` | Status-Prüfung überschreiben |
| `OnBeforeAssistEdit` | `var IntrastatReportHeader, var xIntrastatReportHeader, var Result, var IsHandled` | Nummernvergabe anpassen |
| `OnBeforeInitIntrastatNo` | `var IntrastatReportHeader, var xIntrastatReportHeader, var IsHandled` | Nummerninitialisierung |

---

## Integration Events (Intrastat Report Get Lines, Report 4810)

| Event | Parameter | Beschreibung |
|-------|-----------|-------------|
| `OnBeforeFilterItemLedgerEntry` | `IntrastatReportHeader, ItemLedgerEntry, StartDate, EndDate, var IsHandled` | Filter anpassen |
| `OnBeforeFilterFALedgerEntry` | `IntrastatReportHeader, FALedgerEntry, StartDate, EndDate, var IsHandled` | FA-Filter anpassen |
| `OnBeforeFilterValueEntry` | `IntrastatReportHeader, ValueEntry, ItemLedgerEntry, StartDate, EndDate, var IsHandled` | Value-Entry-Filter |
| `OnAfterCheckItemLedgerEntry` | `IntrastatReportHeader, ItemLedgerEntry, var CurrReportSkip` | Zusätzliche Skip-Logik |
| `OnBeforeCalculateTotalsCall` | `Header, Line, ValueEntry, ItemLedgerEntry, ..., var CurrReportSkip, var IsHandled` | Totals überspringen |
| `OnBeforeCalculateTotals` | `ItemLedgerEntry, Header, TotalAmt, ..., var IsHandled` | Totals selbst berechnen |
| `OnAfterCalculateTotals` | `ItemLedgerEntry, Header, TotalAmt, ...` | Nach Totals-Berechnung |
| `OnBeforeInsertItemLedgerLineCall` | `Header, Line, ValueEntry, ItemLedgerEntry, var IsHandled` | Insert überspringen |
| `OnBeforeInsertItemLedgerLine` | `var IntrastatReportLine, ItemLedgerEntry, var IsHandled` | Vor Insert |
| `OnBeforeInsertJobLedgerLine` | `var IntrastatReportLine, JobLedgerEntry, var IsHandled` | Vor Job-Insert |
| `OnBeforeInsertFALedgerLine` | `var IntrastatReportLine, FALedgerEntry, var IsHandled` | Vor FA-Insert |
| `OnBeforeValidateItemLedgerLineFields` | `IntrastatReportLine, ItemLedgerEntry, var IsHandled` | Validierung überspringen |
| `OnBeforeValidateJobLedgerLineFields` | `IntrastatReportLine, JobLedgerEntry, var IsHandled` | Job-Validierung |
| `OnBeforeValidateFALedgerLineFields` | `IntrastatReportLine, FALedgerEntry, var IsHandled` | FA-Validierung |
| `OnBeforeHasCrossedBorder` | `ItemLedgEntry, var Result, var IsHandled` | Grenzübertritt-Logik |
| `OnBeforeCheckDropShipment` | `IntrastatReportHeader, ItemLedgEntry, Country, var Result, var IsHandled` | Drop-Shipment-Prüfung |
| `OnAfterInitRequestPage` | `IntrastatReportHeader, AmountInclItemCharges, StartDate, EndDate, CostRegulationEnable` | Request Page initialisiert |
| `OnAfterIntrastatReportLineType` | `FALedgerEntry, var IntrastatReportLineType` | FA-Zeilen-Typ |
| `OnAfterGetAmtRoundingDirection` | `var Direction` | Rundungsrichtung |
| `OnAfterItemLedgerEntryOnPreDataItem` | `ItemLedgerEntry` | Nach PreDataItem |
| `OnAfterSkipValueEntry` | `StartDate, EndDate, ValueEntry, ItemLedgerEntry, var IsSkipped` | Value-Entry überspringen |
| `OnCalculateTotalsOnBeforeSumTotals` | `ItemLedgerEntry, Header, TotalAmt, TotalCostAmt` | Vor Summierung |

---

## Erweiterungsbeispiel: Eigenes Land-Plugin

```al
codeunit 50200 "Intrastat DE Management"
{
    [EventSubscriber(ObjectType::Codeunit,
        Codeunit::IntrastatReportManagement,
        'OnBeforeInitSetup', '', true, true)]
    local procedure InitDESetup(
        var IntrastatReportSetup: Record "Intrastat Report Setup";
        var IsHandled: Boolean)
    begin
        IsHandled := true;
        IntrastatReportSetup."Shipments Based On" :=
            IntrastatReportSetup."Shipments Based On"::"Ship-to Country";
        IntrastatReportSetup."Split Files" := false;
        IntrastatReportSetup."Zip Files" := false;
        IntrastatReportSetup."Def. Private Person VAT No." := 'QD999999999999';
        IntrastatReportSetup.Modify();

        // Eigene Data-Exchange-Definition importieren
        CreateDEDataExchangeDef();
    end;

    local procedure CreateDEDataExchangeDef()
    var
        DataExchDef: Record "Data Exch. Def";
        TempBlob: Codeunit "Temp Blob";
        XMLOutStream: OutStream;
        XMLInStream: InStream;
    begin
        // XML-Definition aus Label schreiben
        TempBlob.CreateOutStream(XMLOutStream);
        XMLOutStream.WriteText(DEDataExchangeXMLTxt);
        TempBlob.CreateInStream(XMLInStream);
        Xmlport.Import(Xmlport::"Imp / Exp Data Exch Def & Map", XMLInStream);
    end;

    [EventSubscriber(ObjectType::Table, Database::"Intrastat Report Line",
        'OnBeforeGetPartnerIDForCountry', '', true, true)]
    local procedure GetDEPartnerID(
        CountryRegionCode: Code[10];
        VATRegistrationNo: Text[50];
        IsPrivatePerson: Boolean;
        IsThirdPartyTrade: Boolean;
        var PartnerID: Text[50];
        var IsHandled: Boolean)
    begin
        IsHandled := true;
        // DE-spezifische VAT-ID-Logik
        if IsPrivatePerson then
            PartnerID := 'QD999999999999'
        else if IsThirdPartyTrade then
            PartnerID := 'QD888888888888'
        else
            PartnerID := VATRegistrationNo;
    end;

    var
        DEDataExchangeXMLTxt: Label '<?xml version="1.0" ...>...</root>',
            Locked = true;
}
```

---

## Test-Code — Übersicht (aus `IntrastatReportTest.Codeunit.al`, 5525 Zeilen)

Die Test-Codeunit 139550 enthält umfangreiche Tests. Hier die wichtigsten Test-Methoden:

### Einkauf-Tests

| Test | Szenario |
|------|----------|
| `ItemLedgerEntryForPurchase` | Item Ledger Entry nach Einkaufsbuchung prüfen |
| `IntrastatReportLineForPurchase` | Intrastat-Zeile für Einkauf |
| `IntrastatLineAfterUndoPurchase` | Keine Zeile nach Undo Einkauf |

### Verkauf-Tests

| Test | Szenario |
|------|----------|
| `ItemLedgerEntryForSales` | Item Ledger Entry nach Verkaufsbuchung |
| `IntrastatLineForSales` | Intrastat-Zeile für Verkauf |
| `NoIntrastatLineForSales` | Keine Zeile nach Löschen |
| `UndoSalesShipment` | Undo prüft Quantity auf Sales Shipment Line |
| `IntrastatLineAfterUndoSales` | Keine Zeile nach Undo Verkauf |

### Beispiel-Teststruktur

```al
// Handler-Funktionen unterdrücken Dialoge
[Test]
[Scope('OnPrem')]
[HandlerFunctions('IntrastatReportGetLinesPageHandler')]
procedure IntrastatReportLineForPurchase()
var
    PurchaseLine: Record "Purchase Line";
    IntrastatReportLine: Record "Intrastat Report Line";
    DocumentNo: Code[20];
begin
    // [GIVEN] Bestellung anlegen und buchen
    Initialize();
    DocumentNo := LibraryIntrastat.CreateAndPostPurchaseOrder(
        PurchaseLine, WorkDate());

    // [WHEN] + [THEN] Intrastat-Zeile prüfen
    CreateAndVerifyIntrastatLine(DocumentNo, PurchaseLine."No.",
        PurchaseLine.Quantity, IntrastatReportLine.Type::Receipt);
end;
```

### Verwendete Library-Codeunits

| Library | Funktion |
|---------|----------|
| `Library - Intrastat` | Intrastat-spezifische Hilfsfunktionen |
| `Library - Purchase` | Einkaufsbelege anlegen und buchen |
| `Library - Sales` | Verkaufsbelege anlegen und buchen |
| `Library - Inventory` | Artikel, Lagerbewegungen |
| `Library - ERM` | Fibu-Einrichtung |
| `Library - Service` | Servicebelege |
| `Library - Random` | Zufallsdaten |
| `Library - Item Tracking` | Serien-/Chargennummern |

---

## Wichtige Design-Patterns

### 1. Singleton Setup mit Cache

```al
// IntrastatReportSetup.GetSetup()
var
    SetupRead: Boolean;

procedure GetSetup()
begin
    if not SetupRead then begin
        Get();
        SetupRead := true;
    end;
end;
```

### 2. LockTable() vor Delete/Modify

```al
// Report 4810, OnPreReport():
IntrastatReportLine.SetRange("Intrastat No.", IntrastatReportHeader."No.");
IntrastatReportLine.LockTable();
if not IntrastatReportLine.IsEmpty() then
    if not Confirm(LinesDeletionConfirmationQst) then
        CurrReport.Quit();
IntrastatReportLine.DeleteAll();
```

### 3. Source Entry No. für Idempotenz

```al
IntrastatReportLine2.SetRange("Source Entry No.", "Entry No.");
if IntrastatReportLine2.FindFirst() then
    CurrReport.Skip();  // Zeile existiert bereits → überspringen
```

### 4. Integration Events für Länder-Plugins

Jedes Land (AT, DE, etc.) abonniert:
- `OnBeforeInitSetup` → Setup vorkonfigurieren
- `OnBeforeInitCheckList` → Pflichtfelder definieren
- `OnBeforeGetPartnerIDForCountry` → VAT-ID-Formatierung
- `OnBeforeCreateDefaultDataExchangeDef` → Eigenes Exportformat

### 5. Data Exchange Framework Integration

- Export-Definition als XML-Label in der Codeunit (nicht als Datei)
- Import via `Xmlport.Import(Xmlport::"Imp / Exp Data Exch Def & Map")`
- Mapping auf Tabelle 4812, Key Index 5 (Gruppierung)
- Transformation Rules: `TRIMALL`, `ROUNDTOINT`, `ROUNDUPTOINT`, `EUCOUNTRYCODELOOKUP`

---

## ID-Ranges für eigene Erweiterungen

| App | ID-Range | Verwendung |
|-----|----------|-----------|
| Intrastat Core | 4810–4840 | Basis-Objekte |
| Intrastat AT | 11150–11161 | AT-Erweiterung |
| Eigene Erweiterung | > 50000 (empfohlen) | Benutzerdefinierte Objekte |

### Eigene Intrastat-Erweiterung (app.json)

```json
{
  "id": "your-guid-here",
  "name": "Intrastat Custom",
  "publisher": "Your Company",
  "dependencies": [
    {
      "id": "70912191-3c4c-49fc-a1de-bc6ea1ac9da6",
      "name": "Intrastat Core",
      "publisher": "Microsoft",
      "version": "28.0.0.0"
    }
  ],
  "idRanges": [
    { "from": 50200, "to": 50249 }
  ]
}
```
