---
title: "Überblick & Architektur"
---
# 1. Überblick & Architektur

Business Central 28 (Release Wave 1 2025) besteht aus einer **modularen App-Architektur** auf Basis der AL-Plattform.
Diese Doku beschreibt die OnPrem-Version **28.1.49838.49886** (AT-Lokalisierung).

## 1.1 App-Stack

BC28 hat drei Haupt-Apps, die hierarchisch voneinander abhängen:

```
┌──────────────────────────────────────────────────┐
│  Base Application  (437dbf0e-...)                │
│  → Finance, Sales, Purchasing, Inventory,        │
│    Manufacturing, Jobs, Service, HR, CRM         │
│  idRanges: 1..49999, 104000..104999 etc.        │
├──────────────────────────────────────────────────┤
│  Business Foundation  (f3552374-...)             │
│  → No. Series, Dimensions, Workflow              │
├──────────────────────────────────────────────────┤
│  System Application  (63ca2fa4-...)              │
│  → Azure AD, Encryption, Telemetry,              │
│    Guided Experience, VS Code Integration etc.   │
│  idRanges: 1..9999                               │
└──────────────────────────────────────────────────┘
```

**Abhängigkeiten (Base Application `app.json`):**
```json
"dependencies": [
  { "id": "63ca2fa4-...", "name": "System Application" },
  { "id": "f3552374-...", "name": "Business Foundation" }
]
```

Die System Application hat **keine Abhängigkeiten** (`"dependencies": []`) — sie ist das Fundament.

## 1.2 Namespace-Struktur

Jedes Modul hat einen eigenen Namespace nach dem Schema `Microsoft.<Area>.<Module>`:

| Namespace | Bereich | Typische Objekte |
|---|---|---|
| `Microsoft.Finance.GeneralLedger.Setup` | Fibu-Einrichtung | Table 98 "General Ledger Setup" |
| `Microsoft.Finance.GeneralLedger.Ledger` | Fibu-Buchungen | G/L Entry, G/L Register |
| `Microsoft.Finance.GeneralLedger.Journal` | Fibu-Journale | Gen. Journal, Gen. Journal Line |
| `Microsoft.Finance.Dimension` | Dimensionen | Dimension, Dimension Value |
| `Microsoft.Finance.VAT.Setup` | MwSt-Einrichtung | VAT Posting Setup |
| `Microsoft.Sales.Receivables` | Verkauf/Debitoren | Sales Header/Line, Cust. Ledger Entry |
| `Microsoft.Purchases.Payables` | Einkauf/Kreditoren | Purchase Header/Line, Vendor Ledger Entry |
| `Microsoft.Inventory.Ledger` | Lager | Item Ledger Entry |
| `Microsoft.Inventory.Setup` | Lager-Einrichtung | Inventory Setup |
| `Microsoft.Foundation.NoSeries` | Nummernserien | No. Series, No. Series Line |
| `System.Environment` | System | Environment Information |

## 1.3 Wichtige AL-Sprachfeatures in BC28

- **Namespaces** (seit BC26 verpflichtend): `namespace Microsoft.Finance.GeneralLedger.Setup;`
- **using-Direktiven**: `using Microsoft.Finance.Dimension;`
- **Events**: `[IntegrationEvent]`, `[BusinessEvent]` für Erweiterbarkeit
- **Enums statt Options**: Zunehmend werden `Enum`-Typen statt `Option`-Felder verwendet
- **ObsoleteState**: Felder werden mit `ObsoleteState = Removed` markiert statt sofort gelöscht
- **DataClassification**: `CustomerContent`, `SystemMetadata` für DSGVO-Compliance
- **AccessByPermission**: `TableData "Bank Account" = R` für indirekte Berechtigungen

## 1.4 Objekt-ID-Ranges (Base Application)

Die Base Application belegt folgende ID-Bereiche (aus `app.json`):

```json
"idRanges": [
  { "from": 1, "to": 49999 },          // Hauptbereich
  { "from": 104000, "to": 104999 },     // Erweiterungen
  { "from": 5000000, "to": 5005399 }    // Manufacturing
  // ... weitere Bereiche
]
```

> **Für eigene Extensions:** ID-Range im `app.json` NIE mit diesen Bereichen überlappen lassen.

## 1.5 Verzeichnisstruktur der Source

```
Base Application/
├── Finance/
│   ├── GeneralLedger/
│   │   ├── Setup/          → GeneralLedgerSetup.Table.al (Tabelle 98)
│   │   ├── Account/        → GLAccount.Table.al, ChartofAccounts.Page.al
│   │   ├── Journal/        → GenJournal, GenJournalLine
│   │   └── Ledger/         → GLEntry, GLRegister
│   ├── VAT/                → MwSt-Einrichtung & -Buchungen
│   ├── Consolidation/      → Konsolidierung
│   └── Dimension/          → Dimension, DimensionValue
├── Sales/                  → Verkauf, Debitoren
├── Purchases/              → Einkauf, Kreditoren
├── Inventory/              → Lager, Artikel
├── Manufacturing/          → Produktion
└── ...
```

## 1.6 Erweiterbarkeit durch Events

Fast jede Setup-Tabelle und jedes Posting-Codeunit stellt **Integration Events** bereit:

```al
// Beispiel aus Table 98 "General Ledger Setup"
[IntegrationEvent(true, false)]
local procedure OnAfterIsPostingAllowed(
    GeneralLedgerSetup: Record "General Ledger Setup";
    PostingDate: Date;
    var Result: Boolean)
begin
end;
```

`(true, false)` bedeutet: `Isolated = true` (Fehler im Subscriber brechen nicht den Aufrufer), `NoOfParamsForV2 = false`.

---

| [← Zurück zur Übersicht]({{ '/index' | relative_url }}) | [Weiter: System Application →]({{ '/02-system-application/' | relative_url }}) |
