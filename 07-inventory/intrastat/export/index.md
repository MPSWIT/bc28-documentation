---
title: "Intrastat — Export & Hochladen"
---
# 7. Intrastat — Export &amp; Hochladen

<pre>
7. Lager &amp; Logistik
 │
 ├── <a href="{{ '/07-inventory/' | relative_url }}">Übersicht</a>
 ├── <a href="{{ '/07-inventory/intrastat/' | relative_url }}">Intrastat</a>
 │    ├── <a href="{{ '/07-inventory/intrastat/einrichtung/' | relative_url }}">Einrichtung &amp; Setup</a>
 │    ├── <a href="{{ '/07-inventory/intrastat/verarbeitung/' | relative_url }}">Verarbeitung &amp; Hintergrund</a>
 │    ├─▶ Export &amp; Hochladen  ← Sie sind hier
 │    └── <a href="{{ '/07-inventory/intrastat/entwickler/' | relative_url }}">Entwickler-Referenz</a>
</pre>

---

## `ExportWithDataExch()` — Gesamtablauf

```al
procedure ExportWithDataExch(IntrastatReportHeader, ExportType)
var
    DataExch1, DataExch2: Record "Data Exch.";
    IntrastatReportSetup: Record "Intrastat Report Setup";
    TempBlob: Codeunit "Temp Blob";
begin
    // 1. Prüfung: Bereits gemeldet? → Warnung
    if IntrastatReportHeader.Reported then
        if not Confirm(PeriodAlreadyReportedQst) then exit;

    // 2. Dateinamen festlegen:
    //    FileName      = "Intrastat-{Period}.txt"
    //    ReceiptFileName = "Receipt-{Period}.txt"
    //    ShipmentFileName = "Shipment-{Period}.txt"
    //    ZipFileName   = "Intrastat-{Period}.zip"

    IntrastatReportSetup.Get();

    // 3a. Split Files = true:
    if IntrastatReportSetup."Split Files" then begin
        // Auswahlmenü: Receipts, Shipments, Both
        Selection := StrMenu(ExportSelectionQst, 3);

        if Selection <> 2 then begin
            ExportOneDataExchangeDef(IntrastatReportHeader,
                IntrastatReportSetup."Data Exch. Def. Code - Receipt",
                1, // ExportType=1 (Receipt)
                DataExch1,
                IntrastatReportSetup."Max. No. of Lines in File");
            IntrastatReportHeader.Validate("Arrivals Reported", true);
        end;

        if Selection <> 1 then begin
            ExportOneDataExchangeDef(IntrastatReportHeader,
                IntrastatReportSetup."Data Exch. Def. Code - Shpt.",
                2, // ExportType=2 (Shipment)
                DataExch2,
                IntrastatReportSetup."Max. No. of Lines in File");
            IntrastatReportHeader.Validate("Dispatches Reported", true);
        end;

        if not (DataExch1."File Content".HasValue() or
                DataExch2."File Content".HasValue()) then
            Error(ExternalContentErr, ...);
    end
    // 3b. Split Files = false:
    else begin
        ExportOneDataExchangeDef(IntrastatReportHeader,
            IntrastatReportSetup."Data Exch. Def. Code",
            0, // ExportType=0 (gemeinsam)
            DataExch1,
            IntrastatReportSetup."Max. No. of Lines in File");

        IntrastatReportHeader.Validate("Dispatches Reported", true);
        IntrastatReportHeader.Validate("Arrivals Reported", true);
    end;

    // 4. Ausgabeformat
    if (Selection = 3) or IntrastatReportSetup."Zip Files" or
       (IntrastatReportSetup."Max. No. of Lines in File" > 0)
    then
        ExportToZip(DataExch1, DataExch2, ...)
    else
        if DataExch1."File Content".HasValue then
            ExportToFile(DataExch1, TempBlob, FileName)
        else
            ExportToFile(DataExch2, TempBlob, ShipmentFileName);

    // 5. Export-Zeitstempel setzen
    IntrastatReportHeader."Export Date" := Today;
    IntrastatReportHeader."Export Time" := Time;
    IntrastatReportHeader.Modify();
end;
```

---

## `ExportOneDataExchangeDef()` — Data Exchange pro Definition

```al
procedure ExportOneDataExchangeDef(
    IntrastatReportHeader, DataExchDefCode, ExportType,
    var DataExch, MaxNoOfLines)
begin
    // 1. Data Exchange Mapping für Tabelle 4812 laden
    DataExchMapping.SetRange("Data Exch. Def Code", DataExchDefCode);
    DataExchMapping.SetRange("Table ID", Database::"Intrastat Report Line");
    if not DataExchMapping.FindFirst() then
        Error(NoDataExchMappingErr, ...);

    // 2. Intrastat-Zeilen nach Type filtern
    IntrastatReportLine.SetRange("Intrastat No.", IntrastatReportHeader."No.");
    if ExportType = 1 then  // Receipt
        IntrastatReportLine.SetRange(Type, IntrastatReportLine.Type::Receipt);
    if ExportType = 2 then  // Shipment
        IntrastatReportLine.SetRange(Type, IntrastatReportLine.Type::Shipment);

    // 3. Interne Referenznummer setzen (fortlaufend pro Gruppe)
    DataExchFieldGrouping.SetRange("Data Exch. Def Code", DataExchMapping."Data Exch. Def Code");
    DataExchFieldGrouping.SetRange("Data Exch. Line Def Code", DataExchMapping."Data Exch. Line Def Code");
    DataExchFieldGrouping.SetRange("Table ID", DataExchMapping."Table ID");
    SetInternalRefNo(IntrastatReportLine, DataExchFieldGrouping, IntrastatReportHeader);

    // 4. Data Exch Einträge anlegen & exportieren
    if IntrastatReportLine.FindSet() then
        repeat
            // Bei MaxNoOfLines: Zeilen in Blöcke aufteilen
            if MaxNoOfLines > 0 then begin
                FirstLineNo := IntrastatReportLine."Line No.";
                IntrastatReportLine.Next(MaxNoOfLines - 1);
                LastLineNo := IntrastatReportLine."Line No.";
                IntrastatReportLine.SetRange("Line No.", FirstLineNo, LastLineNo);
            end;

            // Data Exch. Eintrag erstellen
            DataExch.Init();
            DataExch."Data Exch. Def Code" := DataExchMapping."Data Exch. Def Code";
            DataExch."Data Exch. Line Def Code" := DataExchMapping."Data Exch. Line Def Code";

            // Filter als Stream speichern
            DataExch."Table Filters".CreateOutStream(OutStreamFilters);
            OutStreamFilters.WriteText(IntrastatReportLine.GetView(false));

            DataExch.Insert(true);

            // Export ausführen (Data Exchange Framework)
            DataExch.ExportFromDataExch(DataExchMapping);
            DataExch.Modify(true);

            // Filter zurücksetzen für nächsten Block
            if MaxNoOfLines > 0 then
                IntrastatReportLine.SetRange("Line No.");
        until IntrastatReportLine.Next() = 0;
end;
```

---

## `ExportToZip()` — ZIP-Erstellung

```al
local procedure ExportToZip(var DataExch1, DataExch2, StatisticsPeriod,
    FileName, ReceiptFileName, ShipmentFileName, ZipFileName)
var
    DataCompression: Codeunit "Data Compression";
begin
    DataCompression.CreateZipArchive();

    // Receipt-Dateien hinzufügen
    if DataExch1.FindSet() then
        repeat
            DataExch1.CalcFields("File Content");
            DataExch1."File Content".CreateInStream(ServerReceiptsInStream);
            DataCompression.AddEntry(ServerReceiptsInStream,
                GetFileName(FileName, I, DataExch1.Count() > 1));
        until DataExch1.Next() = 0;

    // Shipment-Dateien hinzufügen
    if DataExch2.FindSet() then
        repeat
            DataExch2.CalcFields("File Content");
            DataExch2."File Content".CreateInStream(ServerShipmentsInStream);
            DataCompression.AddEntry(ServerShipmentsInStream,
                GetFileName(ShipmentFileName, I, DataExch2.Count() > 1));
        until DataExch2.Next() = 0;

    // ZIP als Download bereitstellen
    ZipTempBlob.CreateOutStream(ZipOutStream);
    DataCompression.CloseZipArchive(ZipOutStream);
    DownloadFromStream(...);
end;
```

**Dateibenennung bei MaxNoOfLines > 1:**

```
Intrastat-2406.txt     (1 Datei)
Intrastat-2406_1.txt   (Teil 1 von N)
Intrastat-2406_2.txt   (Teil 2 von N)
```

---

## Interner Referenznummern-Mechanismus

Die `Internal Ref. No.` (Feld 23) ist eine fortlaufende Nummer innerhalb jeder Gruppierung:

```al
procedure SetInternalRefNo(var IntrastatReportLine, DataExchFieldGrouping,
    IntrastatReportHeader)
var
    IntrastatReportLine2: Record "Intrastat Report Line";
    InternalRefNo: Integer;
begin
    IntrastatReportLine2.Copy(IntrastatReportLine);
    IntrastatReportLine2.SetRange("Intrastat No.", IntrastatReportHeader."No.");

    // Gruppierungsfelder setzen
    IntrastatReportLine2.SetRange(Type, IntrastatReportLine.Type);
    IntrastatReportLine2.SetRange("Country/Region Code",
        IntrastatReportLine."Country/Region Code");
    // ... alle Grouping-Felder

    // Letzte Ref. No. finden
    if IntrastatReportLine2.FindLast() then
        InternalRefNo := Evaluate(IntrastatReportLine2."Internal Ref. No.") + 1
    else
        InternalRefNo := 1;

    // Allen Zeilen der Gruppe zuweisen
    IntrastatReportLine2.SetRange("Line No.");
    IntrastatReportLine2.Copy(IntrastatReportLine);
    if IntrastatReportLine2.FindSet() then
        repeat
            IntrastatReportLine2."Internal Ref. No." := Format(InternalRefNo);
            IntrastatReportLine2.Modify();
        until IntrastatReportLine2.Next() = 0;
end;
```

---

## Data Exchange Mapping — Feldzuordnung (Standard `INTRA-2022`)

| Spalte | Name | DB-Feld | Transformation | Länge | Padding |
|--------|------|---------|---------------|-------|---------|
| 1 | Tariff No. | 5 (Tariff No.) | TRIMALL | 8 | 0, links |
| 2 | Country/Region Code | 101 (Intrastat Ctry Code) | — | 3 | Leerzeichen, rechts |
| 3 | Transaction Type | 8 (Transaction Type) | — | 2 | Leerzeichen, rechts |
| 4 | Quantity | 14 (Quantity) | ROUNDTOINT | 11 | 0, links |
| 5 | Total Weight | 21 (Total Weight) | ROUNDUPTOINT | 10 | 0, links |
| 6 | Statistical Value | 17 (Statistical Value) | ROUNDTOINT | 11 | 0, links |
| 7 | Internal Ref. No. | 23 (Internal Ref. No.) | — | 30 | Leerzeichen, rechts |
| 8 | Partner Tax ID | 29 (Partner VAT ID) | — | 20 | Leerzeichen, rechts |
| 9 | Country of Origin | 24 (Ctry/Region of Origin) | EUCOUNTRYCODELOOKUP | 3 | Leerzeichen, rechts |

### Beispiel-Exportdatei

```
84713000001   112       100000000000000000100000000001                             ATU12345678001
84713000001   222       500000000000000000050000000000002                             ATU12345678001
```

**Erklärung Zeile 1:**
- `84713000` = Tarifnummer (8-stellig, TRIMALL)
- `001` = Ländercode (Intrastat-Code)
- `12` = Transaction Type
- `00000000001` = Quantity (11-stellig, auf Integer gerundet)
- `0000000001` = Total Weight (10-stellig, aufgerundet)
- `00000000001` = Statistical Value (11-stellig)
- `1` = Internal Ref. No. (30-stellig, rechts gepolstert)
- `ATU12345678` = Partner VAT ID (20-stellig, rechts gepolstert)
- `001` = Country of Origin (EUCOUNTRYCODELOOKUP)

---

## AT-spezifischer Export (`INTRA-2022-AT`)

Das AT-Modul definiert ein eigenes Format mit 10 Spalten:

| Spalte | Name | DB-Feld | Länge | Format |
|--------|------|---------|-------|--------|
| 1 | Tariff No. | 5 | 8 | 0-gepolstert |
| 2 | Tariff Description | 6 | — | |
| 3 | Country/Region Code | 101 | — | |
| 4 | Country/Region of Origin | 24 | — | |
| 5 | Nature of Transaction | 8 | 2 | |
| 6 | Total Weight | 21 | 14 | Decimal(3:3), Komma |
| 7 | Supplementary Quantity | 35 | 14 | Decimal(3:3) |
| 8 | Amount | 13 | 13 | Decimal(2:2) |
| 9 | Statistical Value | 17 | 13 | Decimal(2:2) |
| 10 | Partner VAT ID | 29 | — | |

```al
// AT DataExchangeXML:
<DataExchColumnDef ColumnNo="6" Name="Total Weight" DataType="2"
  DataFormat="<Precision,3:3><Integer><Decimals><Comma,,>"
  DataFormattingCulture="en-US" Length="14"
  TextPaddingRequired="true" PadCharacter="0" Justification="0" />
```

**AT-Besonderheiten:**
- `Split Files` = true (getrennte Dateien für Receipts/Shipments)
- `Zip Files` = true
- Data Exch. Def. Code für beide Richtungen = `INTRA-2022-AT`
- VAT-IDs im Format: Privatperson=`QN999999999999`, Dreiecksgeschäft/Unbekannt=`QV999999999999`
- `Country/Region of Origin Code` wird bei Receipt angepasst (siehe Verarbeitung)

---

## Export-Flow als Diagramm

```
Intrastat Report Header (Status = Released)
 │
 ├── Split Files? ──┬── Ja ──→ Auswahl: Receipts / Shipments / Both
 │                  │
 │                  │         ┌─ Data Exch. Def. Code - Receipt
 │                  │         │   → ExportOneDataExchangeDef(ExportType=1)
 │                  │         │   → Filter: Type = Receipt
 │                  │         │   → Arrivals Reported = true
 │                  │         │
 │                  │         └─ Data Exch. Def. Code - Shpt.
 │                  │             → ExportOneDataExchangeDef(ExportType=2)
 │                  │             → Filter: Type = Shipment
 │                  │             → Dispatches Reported = true
 │                  │
 │                  └── Nein ─→ Data Exch. Def. Code
 │                               → ExportOneDataExchangeDef(ExportType=0)
 │                               → Beide Typen gemeinsam
 │
 ├── Zip Files? oder MaxNoOfLines > 0?
 │    ├── Ja → ExportToZip()
 │    │         → ZIP-Archiv mit Receipt- und/oder Shipment-Dateien
 │    │         → Download
 │    │
 │    └── Nein → ExportToFile()
 │               → Einzeldatei als Download
 │
 └── Zeitstempel: Export Date, Export Time speichern
```

---

## Manuelles Test-Szenario: Export

1. Intrastat-Meldung erstellen (z.B. Type=Purchases, Statistics Period=2406)
2. `Get Lines` ausführen → Zeilen prüfen
3. Status auf `Released` setzen
4. Export auslösen → Datei wird generiert und heruntergeladen
5. `Reported` = true (automatisch gesetzt)

### C/AL-Äquivalent zum Testen

```al
// Intrastat-Meldung erstellen
IntrastatReportHeader.Init();
IntrastatReportHeader."Statistics Period" := '2406';
IntrastatReportHeader.Type := IntrastatReportHeader.Type::Purchases;
IntrastatReportHeader.Insert(true);

// Zeilen abrufen
GetLinesReport.SetIntrastatReportHeader(IntrastatReportHeader);
GetLinesReport.Run();

// Freigeben und exportieren
IntrastatReportMgt.ReleaseIntrastatReport(IntrastatReportHeader);
IntrastatReportMgt.ExportWithDataExch(IntrastatReportHeader, 0);
```
