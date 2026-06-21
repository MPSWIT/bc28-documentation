---
title: "Überblick — App-Stack & Namespaces"
---
# 1.1 App-Stack & Namespaces

> 📄 **[← Zurück zur Übersicht]({{ '/01-overview/' | relative_url }})**

## 1.1.1 App-Stack

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

## 1.1.2 Namespace-Struktur

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

## 1.1.3 Objekt-ID-Bereiche (Basisanwendung)

Die Basisanwendung belegt folgende ID-Bereiche (aus `app.json`):

```json
"idRanges": [
  { "from": 1, "to": 49999 },          // Hauptbereich
  { "from": 104000, "to": 104999 },     // Erweiterungen
  { "from": 5000000, "to": 5005399 }    // Produktion
]
```

> **Für eigene Erweiterungen:** Den ID-Bereich im `app.json` NIEMALS mit diesen Bereichen überlappen lassen.

## 1.1.4 Quellcode-Verzeichnisstruktur

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

---

| [← Zurück zur Übersicht]({{ '/01-overview/' | relative_url }}) | [AL & Erweiterbarkeit →]({{ '/01-overview/al-erweiterbarkeit/' | relative_url }}) |
