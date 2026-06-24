---
title: "Intrastat — Einrichtung & Setup"
---
# 7. Intrastat — Einrichtung &amp; Setup

<pre>
7. Lager &amp; Logistik
 │
 ├── <a href="{{ '/07-inventory/' | relative_url }}">Übersicht</a>
 ├── <a href="{{ '/07-inventory/intrastat/' | relative_url }}">Intrastat</a>
 │    ├─▶ Einrichtung &amp; Setup  ← Sie sind hier
 │    ├── <a href="{{ '/07-inventory/intrastat/verarbeitung/' | relative_url }}">Verarbeitung &amp; Hintergrund</a>
 │    ├── <a href="{{ '/07-inventory/intrastat/export/' | relative_url }}">Export &amp; Hochladen</a>
 │    └── <a href="{{ '/07-inventory/intrastat/entwickler/' | relative_url }}">Entwickler-Referenz</a>
</pre>

---

## Setup-Tabelle `Intrastat Report Setup` (4810)

Die Tabelle 4810 ist die zentrale Konfiguration (Singleton — es gibt nur einen Datensatz).

### Felder im Detail

| Feld-Nr. | Name | Typ | Beschreibung |
|----------|------|-----|-------------|
| 1 | Code | Code[10] | Eindeutiger Schlüssel |
| 2 | Report Receipts | Boolean | Wareneingänge melden? |
| 3 | Report Shipments | Boolean | Warenausgänge melden? |
| 4 | Default Trans. - Purchase | Code[10] | Standard-Transaktionstyp für Einkauf/Verkauf |
| 5 | Default Trans. - Return | Code[10] | Standard-Transaktionstyp für Retouren |
| 6 | Intrastat Contact Type | Enum | Ansprechpartner-Typ (Contact/Vendor) |
| 7 | Intrastat Contact No. | Code[20] | Ansprechpartner-Nummer |
| 9 | Cust. VAT No. on File | Enum `VAT File Fmt` | Format für Debitoren-UID im Export |
| 10 | Vend. VAT No. on File | Enum `VAT File Fmt` | Format für Kreditoren-UID im Export |
| 11 | Company VAT No. on File | Enum `VAT File Fmt` | Format für eigene UID im Export |
| 12 | Default Trans. Spec. Code | Code[10] | Standard-Transaktionsspezifikation |
| 14 | Intrastat Nos. | Code[20] | Nummernserie für Intrastat-Meldungen |
| 15 | Split Files | Boolean | Getrennte Dateien für Receipts/Shipments |
| 16 | Zip Files | Boolean | Ausgabe als ZIP-Datei(en) |
| 17 | Data Exch. Def. Code | Code[20] | Data-Exchange-Definition (gemeinsam) |
| 19 | Data Exch. Def. Code - Receipt | Code[20] | Data-Exchange-Definition (Receipt) |
| 21 | Data Exch. Def. Code - Shpt. | Code[20] | Data-Exchange-Definition (Shipment) |
| 23 | Shipments Based On | Enum `Shpt. Base` | Welches Land wird als Partnerland verwendet? |
| 25 | Def. Private Person VAT No. | Text[50] | Default-UID für Privatpersonen |
| 26 | Def. 3-Party Trade VAT No. | Text[50] | Default-UID für Dreiecksgeschäfte |
| 27 | Def. VAT for Unknown State | Text[50] | Default-UID bei unbekanntem Status |
| 29 | Suppl. Units Weight | Enum `Suppl. Weight` | Gewichtung der Ergänzungseinheiten |
| 30 | Get Partner VAT For | Enum `Line Type Sel` | Für welche Zeilenart wird VAT-ID aktualisiert? |
| 31 | Include Drop Shipment | Boolean | Streckengeschäfte einbeziehen? |
| 32 | Def. Country Code for Item Tr. | Enum `Ctry. Code-Item Track.` | Herkunftsland bei Item Tracking |
| 33 | Sales VAT No. Based On | Enum `VAT No. Base` | Quelle der VAT-ID im Verkauf |
| 34 | Purchase VAT No. Based On | Enum `Purch. VAT No. Base` | Quelle der VAT-ID im Einkauf |
| 35 | Project VAT No. Based On | Enum `Proj. VAT No. Base` | Quelle der VAT-ID bei Projekten |
| 36 | Sales Intrastat Info Based On | Enum `Sales Info Base` | Intrastat-Einstellungen vom Kunden holen? |
| 37 | Purch. Intrastat Info Based On | Enum `Purch. Info Base` | Intrastat-Einstellungen vom Kreditor holen? |
| 38 | Transaction Type Mandatory | Boolean | Transaktionstyp ist Pflichtfeld |
| 39 | Transaction Spec. Mandatory | Boolean | Transaktionsspezifikation ist Pflichtfeld |
| 40 | Transport Method Mandatory | Boolean | Transportmethode ist Pflichtfeld |
| 41 | Shipment Method Mandatory | Boolean | Versandart ist Pflichtfeld |
| 42 | Max. No. of Lines in File | Integer | Maximale Zeilenanzahl pro Datei (0 = unlimitiert) |

### Enum: `Intrastat Report VAT File Fmt`

| Wert | Beschreibung |
|------|-------------|
| `VAT Reg. No.` | UID-Nummer wie im Feld gespeichert |
| `EU Country Code + VAT Reg. No` | EU-Ländercode als Präfix (z.B. `ATU12345678`) |
| `VAT Reg. No. Without EU Country Code` | EU-Ländercode entfernen, falls vorhanden |

### Setup-Code: `IntrastatReportManagement.InitSetup()`

```al
// Vereinfachter Ablauf beim Erstaufruf des Setups:
procedure InitSetup()
begin
    if not IntrastatReportSetup.Get() then begin
        IntrastatReportSetup.Init();
        IntrastatReportSetup.Code := '';
        IntrastatReportSetup.Insert(true);
    end;
    // Data-Exchange-Definition aus XML-Label importieren
    ImportDefaultIntrastatDataExchDef();
    // Checkliste initialisieren
    InitCheckList();
end;
```

---

## Data Exchange Definition

Die Data-Exchange-Definition `INTRA-2022` ist als XML-String im Code hinterlegt und wird beim Setup automatisch importiert.

```xml
<DataExchDef Code="INTRA-2022" Name="Intrastat Report 2022" Type="5"
  ReadingWritingXMLport="1231" ExternalDataHandlingCodeunit="4813"
  ColumnSeparator="1" FileType="1" ReadingWritingCodeunit="1276">
  <DataExchLineDef Code="DEFAULT" ColumnCount="9">
    <DataExchColumnDef ColumnNo="1" Name="Tariff No." Length="8" PadCharacter="0" />
    <DataExchColumnDef ColumnNo="2" Name="Country/Region Code" Length="3" />
    <DataExchColumnDef ColumnNo="3" Name="Transaction Type" Length="2" />
    <DataExchColumnDef ColumnNo="4" Name="Quantity" Length="11" PadCharacter="0" />
    <DataExchColumnDef ColumnNo="5" Name="Total Weight" Length="10" PadCharacter="0" />
    <DataExchColumnDef ColumnNo="6" Name="Statistical Value" Length="11" PadCharacter="0" />
    <DataExchColumnDef ColumnNo="7" Name="Internal Ref. No." Length="30" />
    <DataExchColumnDef ColumnNo="8" Name="Partner Tax ID" Length="20" />
    <DataExchColumnDef ColumnNo="9" Name="Country of Origin Code" Length="3" />
    <DataExchMapping TableId="4812" KeyIndex="5" MappingCodeunit="1269">
      <DataExchFieldMapping ColumnNo="1" FieldID="5" TransformationRule="TRIMALL" />
      <DataExchFieldMapping ColumnNo="2" FieldID="101" />  <!-- Intrastat Country/Region Code -->
      <DataExchFieldMapping ColumnNo="3" FieldID="8" />
      <DataExchFieldMapping ColumnNo="4" FieldID="14" TransformationRule="ROUNDTOINT" />
      <DataExchFieldMapping ColumnNo="5" FieldID="21" TransformationRule="ROUNDUPTOINT" />
      <DataExchFieldMapping ColumnNo="6" FieldID="17" TransformationRule="ROUNDTOINT" />
      <DataExchFieldMapping ColumnNo="7" FieldID="23" />
      <DataExchFieldMapping ColumnNo="8" FieldID="29" />
      <DataExchFieldMapping ColumnNo="9" FieldID="24" TransformationRule="EUCOUNTRYCODELOOKUP" />
      <!-- Gruppierung: gleiche Schlüssel werden zusammengefasst -->
      <DataExchFieldGrouping FieldID="3" />  <!-- Type (Receipt/Shipment) -->
      <DataExchFieldGrouping FieldID="5" />  <!-- Tariff No. -->
      <DataExchFieldGrouping FieldID="7" />  <!-- Country/Region Code -->
      <DataExchFieldGrouping FieldID="8" />  <!-- Transaction Type -->
      <DataExchFieldGrouping FieldID="9" />  <!-- Transport Method -->
      <DataExchFieldGrouping FieldID="24" /> <!-- Country/Region of Origin Code -->
      <DataExchFieldGrouping FieldID="29" /> <!-- Partner VAT ID -->
    </DataExchMapping>
  </DataExchLineDef>
</DataExchDef>
```

**Transformationsregeln:**

| Regel | Beschreibung |
|-------|-------------|
| `TRIMALL` | Entfernt alle Leerzeichen (Tarifnummer) |
| `ROUNDTOINT` | Rundet auf Integer (=) |
| `ROUNDUPTOINT` | Rundet auf Integer auf (>) |
| `ALPHANUMERIC_ONLY` | Nur alphanumerische Zeichen |
| `EUCOUNTRYCODELOOKUP` | Wandelt ISO-Code in EU-Intrastat-Code um (Lookup Tabelle 9, Feld 7) |

---

## Stammdaten-Erweiterungen (TableExtensions)

Das Intrastat Core Modul erweitert folgende Basistabellen:

### Einkauf

| Basistabelle | Extension | Neue Felder |
|-------------|-----------|-------------|
| Vendor | `IntrastatReportVendor.TableExt` | VAT-Registrierungsfelder |
| Vendor Template | `IntrastatReportVendorTempl.TableExt` | Vorlagefelder |
| Purchase Header | `IntrastatReportPurchHead.TableExt` | Transaction Type, Transport Method, Entry/Exit Point, Transaction Specification, Area, Shipment Method Code |

### Verkauf

| Basistabelle | Extension | Neue Felder |
|-------------|-----------|-------------|
| Customer | `IntrastatReportCustomer.TableExt` | VAT-Registrierungsfelder |
| Customer Template | `IntrastatReportCustTempl.TableExt` | Vorlagefelder |
| Sales Header | `IntrastatReportSalesHead.TableExt` | Transaction Type, Transport Method, Entry/Exit Point, Transaction Specification, Area, Shipment Method Code |

### Artikel &amp; Anlagen

| Basistabelle | Extension | Neue Felder |
|-------------|-----------|-------------|
| Item | `IntrastatReportItem.TableExt` | Tariff No., Exclude from Intrastat Report, Supplementary UOM, Country/Region of Origin Code, Net Weight |
| Item Template | `IntrastatReportItemTempl.TableExt` | Vorlagefelder |
| Fixed Asset | `IntrastatReportFixedAsset.TableExt` | Tariff No., Exclude from Intrastat Report, Supplementary UOM, Country/Region of Origin Code |
| Tariff Number | `IntrastatReportTariffNumber.TableExt` | Supplementary Unit of Measure |

### Stammdaten

| Basistabelle | Extension | Neue Felder |
|-------------|-----------|-------------|
| Country/Region | `IntrastatReportCountry.TableExt` | Intrastat Code (z.B. AT=AT, DE=DE) |
| Contact | `IntrastatReportContact.TableExt` | Intrastat-Kontaktfelder |
| General Ledger Setup | `IntrastatReportGLSetup.TableExt` | Intrastat-bezogene Felder |

### Weitere Belege

| Basistabelle | Extension |
|-------------|-----------|
| Transfer Header | `IntrastatReportTrsfHeader.TableExt` |
| Service Header | `IntrastatReportServHead.TableExt` |

---

## Automatische Vorgabewerte: `Intrastat Report Doc. Compl.` (4812)

Diese Codeunit abonniert `OnBeforeInsertEvent` für Sales Header, Purchase Header und Service Header und setzt automatisch Standardwerte:

```al
codeunit 4812 "Intrastat Report Doc. Compl."
{
    [EventSubscriber(ObjectType::Table, Database::"Sales Header",
        'OnBeforeInsertEvent', '', false, false)]
    local procedure DefaultSalesDocuments(var Rec: Record "Sales Header";
        RunTrigger: Boolean)
    var
        IntrastatReportSetup: Record "Intrastat Report Setup";
    begin
        if not IntrastatReportSetup.Get() then exit;

        // Gutschrift / Retoure → Default Trans. - Return
        if Rec."Document Type" in [Rec."Document Type"::"Credit Memo",
                                    Rec."Document Type"::"Return Order"] then
            if Rec."Transaction Type" = '' then
                Rec."Transaction Type" :=
                    IntrastatReportSetup."Default Trans. - Return";

        // Rechnung / Auftrag → Default Trans. - Purchase
        if Rec."Document Type" in [Rec."Document Type"::Invoice,
                                    Rec."Document Type"::Order] then
            if Rec."Transaction Type" = '' then
                Rec."Transaction Type" :=
                    IntrastatReportSetup."Default Trans. - Purchase";
    end;
}
```

Zusätzlich werden bei `Sales Document - Test`, `Purchase Document - Test` und `Service Document - Test` die Pflichtfelder geprüft (Transaction Type, Transaction Spec., Transport Method, Shipment Method) und Fehler in der Belegprüfung ausgegeben.

---

## Checkliste (`Intrastat Report Checklist`, 4813)

Die Checkliste definiert, welche Felder auf einer Intrastat-Zeile Pflicht sind. Beispiel AT:

```al
// IntrastatReportManagementAT.OnBeforeInitCheckList:
IntrastatReportChecklist.Validate("Field No.", 5);   // Tariff No.
IntrastatReportChecklist.Validate("Field No.", 7);   // Country/Region Code
IntrastatReportChecklist.Validate("Field No.", 8);   // Transaction Type
IntrastatReportChecklist.Validate("Field No.", 14);  // Quantity
IntrastatReportChecklist.Validate("Field No.", 17);  // Statistical Value
IntrastatReportChecklist.Validate("Field No.", 21);  // Total Weight
IntrastatReportChecklist.Validate("Field No.", 24);  // Country/Region of Origin Code

// Bedingt:
IntrastatReportChecklist.Validate("Field No.", 14);
IntrastatReportChecklist.Validate("Filter Expression",
    'Supplementary Units: True');  // Suppl. Quantity nur wenn Suppl. Units aktiv

IntrastatReportChecklist.Validate("Field No.", 29);
IntrastatReportChecklist.Validate("Filter Expression",
    'Type: Shipment');  // Partner VAT ID nur bei Shipment
```

Die Validierung erfolgt über `IntrastatReportManagement.ValidateReportWithAdvancedChecklist()`:

```al
procedure ValidateReportWithAdvancedChecklist(
    IntrastatReportHeader: Record "Intrastat Report Header";
    ThrowError: Boolean): Boolean
begin
    ChecklistClearBatchErrors(IntrastatReportHeader);
    IntrastatReportLine.SetRange("Intrastat No.", IntrastatReportHeader."No.");
    if IntrastatReportLine.FindSet() then
        repeat
            ValidateReportLineWithAdvancedChecklist(IntrastatReportLine, ThrowError);
        until IntrastatReportLine.Next() = 0;
end;
```

---

## Schritt-für-Schritt: Einrichtung durchführen

1. **Intrastat Report Setup** öffnen und folgende Felder setzen:
   - `Report Receipts` = true
   - `Report Shipments` = true
   - `Intrastat Nos.` = Nummernserie wählen/anlegen
   - `Shipments Based On` = `Ship-to Country`
   - `Data Exch. Def. Code` = `INTRA-2022` (wird automatisch angelegt)

2. **Länder prüfen:** Jedes EU-Land in `Country/Region` benötigt einen `Intrastat Code`.

3. **Artikel konfigurieren:**
   - `Tariff No.` = Zolltarifnummer (z.B. `84713000`)
   - `Net Weight` = Nettogewicht pro Einheit
   - `Country/Region of Origin Code` = Ursprungsland
   - `Supplementary Unit of Measure` = Ergänzende Mengeneinheit (falls erforderlich)
   - `Exclude from Intrastat Report` = false

4. **Debitoren/Kreditoren:** VAT Registration No. im entsprechenden Feld hinterlegen.

5. **Setup-Assistent** öffnen (`Intrastat Report Setup Wizard`) — führt durch alle Schritte.
