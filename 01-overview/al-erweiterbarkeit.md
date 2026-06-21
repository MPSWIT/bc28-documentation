---
title: "Überblick — AL-Sprachmerkmale & Erweiterbarkeit"
---
# 1.2 AL-Sprachmerkmale & Erweiterbarkeit

> 📄 **[← Zurück zur Übersicht]({{ '/01-overview/' | relative_url }}/)**

## 1.2.1 Wichtige AL-Sprachmerkmale in BC28

- **Namensräume** (seit BC26 verpflichtend): `namespace Microsoft.Finance.GeneralLedger.Setup;`
- **using-Direktiven**: `using Microsoft.Finance.Dimension;`
- **Ereignisse**: `[IntegrationEvent]`, `[BusinessEvent]` für Erweiterbarkeit
- **Enums statt Optionen**: Zunehmend werden `Enum`-Typen statt `Option`-Felder verwendet
- **ObsoleteState**: Felder werden mit `ObsoleteState = Removed` markiert statt sofort gelöscht. `ObsoleteTag` gibt die Version an (z.B. `'27.0'`)
- **DataClassification**: `CustomerContent`, `SystemMetadata` für DSGVO-Konformität
- **AccessByPermission**: `TableData "Bank Account" = R` für indirekte Berechtigungen (Field-Level-Access)

### Enum-Beispiel aus BC28

```al
enum "Default Posting Date"
{
    value(0; "Work Date") { Caption = 'Work Date'; }
    value(1; "No Date")   { Caption = 'No Date'; }
}
```

### Obsolete-Muster

```al
field(57; "Create Item from Item No."; Boolean)
{
    ObsoleteReason = 'Discontinued function';
    ObsoleteState = Pending;
    ObsoleteTag = '27.0';
}
```

## 1.2.2 Erweiterbarkeit durch Ereignisse

Fast jede Einrichtungstabelle und jede Buchungs-Codeunit stellt **Integrationsereignisse** bereit:

```al
// Beispiel aus Tabelle 98 "General Ledger Setup"
[IntegrationEvent(true, false)]
local procedure OnAfterIsPostingAllowed(
    GeneralLedgerSetup: Record "General Ledger Setup";
    PostingDate: Date;
    var Result: Boolean)
begin
end;
```

**Ereignisparameter:**
- `Isolated = true` → Fehler im Abonnenten brechen nicht den Aufrufer
- `NoOfParamsForV2 = false` → Standard V1-Signatur
- `var Result` → Abonnent kann den Rückgabewert überschreiben

### Subscriber-Beispiel

```al
codeunit 50100 "My GL Validation"
{
    [EventSubscriber(ObjectType::Table, Database::"General Ledger Setup",
        'OnAfterIsPostingAllowed', '', false, false)]
    local procedure BlockPostingInClosedPeriod(
        GeneralLedgerSetup: Record "General Ledger Setup";
        PostingDate: Date;
        var Result: Boolean)
    begin
        if PostingDate < DMY2Date(1, 1, 2025) then begin
            Result := false;
            Error('Buchungen vor 01.01.2025 nicht erlaubt.');
        end;
    end;
}
```

## 1.2.3 Erweiterungsmuster (TableExtension / PageExtension)

```al
tableextension 50100 "MyCustomerExt" extends Customer
{
    fields
    {
        field(50100; "My Custom Field"; Decimal)
        {
            Caption = 'My Field';
            DataClassification = CustomerContent;
        }
    }
}
```

---

| [← Zurück zur Übersicht]({{ '/01-overview/' | relative_url }}/) | [→ Nächstes Kapitel: Systemanwendung]({{ '/02-system-application/' | relative_url }}) |
