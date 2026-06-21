---
title: "Überblick & Architektur"
---
# 1. Überblick & Architektur

Business Central 28 (Release Wave 1 2025) besteht aus einer **modularen App-Architektur** auf Basis der AL-Plattform.
Diese Dokumentation beschreibt die OnPrem-Version **28.1.49838.49886** (AT-Lokalisierung).

## 1.1 App-Stack

BC28 hat drei Haupt-Apps, die hierarchisch voneinander abhängen:

```
┌──────────────────────────────────────────────────┐
│  Basisanwendung  (437dbf0e-...)                  │
│  → Finanzwesen, Verkauf, Einkauf, Lager,         │
│    Produktion, Projekte, Service, Personal, CRM  │
│  idRanges: 1..49999, 104000..104999 u.a.        │
├──────────────────────────────────────────────────┤
│  Business Foundation  (f3552374-...)             │
│  → Nummernserien, Dimensionen, Workflow          │
├──────────────────────────────────────────────────┤
│  Systemanwendung  (63ca2fa4-...)                 │
│  → Azure AD, Verschlüsselung, Telemetrie,        │
│    Geführte Einrichtung, VS Code-Integration     │
│  idRanges: 1..9999                               │
└──────────────────────────────────────────────────┘
```

**Abhängigkeiten (Basisanwendung `app.json`):**
```json
"dependencies": [
  { "id": "63ca2fa4-...", "name": "System Application" },
  { "id": "f3552374-...", "name": "Business Foundation" }
]
```

Die Systemanwendung hat **keine Abhängigkeiten** (`"dependencies": []`) — sie ist das Fundament.

## 1.2 Namespace-Struktur

Jedes Modul hat einen eigenen Namensraum nach dem Schema `Microsoft.<Bereich>.<Modul>`:

| Namensraum | Bereich | Typische Objekte |
|---|---|---|
| `Microsoft.Finance.GeneralLedger.Setup` | Fibu-Einrichtung | Tabelle 98 "General Ledger Setup" |
| `Microsoft.Finance.GeneralLedger.Ledger` | Fibu-Buchungen | G/L Entry, G/L Register |
| `Microsoft.Finance.GeneralLedger.Journal` | Fibu-Buchungsblätter | Gen. Journal, Gen. Journal Line |
| `Microsoft.Finance.Dimension` | Dimensionen | Dimension, Dimension Value |
| `Microsoft.Finance.VAT.Setup` | MwSt-Einrichtung | VAT Posting Setup |
| `Microsoft.Sales.Receivables` | Verkauf/Debitoren | Sales Header/Line, Cust. Ledger Entry |
| `Microsoft.Purchases.Payables` | Einkauf/Kreditoren | Purchase Header/Line, Vendor Ledger Entry |
| `Microsoft.Inventory.Ledger` | Lager | Item Ledger Entry |
| `Microsoft.Inventory.Setup` | Lager-Einrichtung | Inventory Setup |
| `Microsoft.Foundation.NoSeries` | Nummernserien | No. Series, No. Series Line |
| `System.Environment` | System | Environment Information |

## 1.3 Wichtige AL-Sprachmerkmale in BC28

- **Namensräume** (seit BC26 verpflichtend): `namespace Microsoft.Finance.GeneralLedger.Setup;`
- **using-Direktiven**: `using Microsoft.Finance.Dimension;`
- **Ereignisse**: `[IntegrationEvent]`, `[BusinessEvent]` für Erweiterbarkeit
- **Enums statt Optionen**: Zunehmend werden `Enum`-Typen statt `Option`-Felder verwendet
- **ObsoleteState**: Felder werden mit `ObsoleteState = Removed` markiert statt sofort gelöscht
- **DataClassification**: `CustomerContent`, `SystemMetadata` für DSGVO-Konformität
- **AccessByPermission**: `TableData "Bank Account" = R` für indirekte Berechtigungen

## 1.4 Objekt-ID-Bereiche (Basisanwendung)

Die Basisanwendung belegt folgende ID-Bereiche (aus `app.json`):

```json
"idRanges": [
  { "from": 1, "to": 49999 },          // Hauptbereich
  { "from": 104000, "to": 104999 },     // Erweiterungen
  { "from": 5000000, "to": 5005399 }    // Produktion
  // ... weitere Bereiche
]
```

> **Für eigene Erweiterungen:** Den ID-Bereich im `app.json` NIEMALS mit diesen Bereichen überlappen lassen.

## 1.5 Quellcode-Verzeichnisstruktur

```
Basisanwendung/
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

## 1.6 Erweiterbarkeit durch Ereignisse

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

`(true, false)` bedeutet: `Isolated = true` (Fehler im Abonnenten brechen nicht den Aufrufer), `NoOfParamsForV2 = false`.

---

| [← Zurück zur Übersicht]({{ '/index' | relative_url }}) | [Weiter: Systemanwendung →]({{ '/02-system-application/' | relative_url }}) |
