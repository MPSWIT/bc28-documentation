---
title: "ToDo — Anlagen im Bau"
---
# 19.2 Anlagen im Bau (Assets under Construction)

> **Leitfrage:** Wie geht man in Business Central mit Anlagen im Bau um? Wie werden Baukosten gesammelt, bilanziell getrennt ausgewiesen und aktiviert?

> 📄 **[← ToDo-Übersicht]({{ '/19-todo/' | relative_url }})**

---

## 19.2.1 Das Problem

**Anlagen im Bau** (AiB, CIP = Construction in Progress) müssen bilanziell **getrennt** von fertigen Anlagen ausgewiesen werden. Erst nach Fertigstellung werden sie in die endgültige Anlagenklasse umgebucht und die planmäßige AfA beginnt.

Die Herausforderung in BC28: Es gibt **kein natives AiB-Feld** auf der Anlagenkarte. Es gibt drei gangbare Wege — mit unterschiedlicher konzeptioneller Sauberkeit.

---

## 19.2.2 Ansatz 1: Sachkonto-basiert (✅ empfohlen)

**Idee:** Während der Bauphase werden alle Kosten **ausschließlich** auf ein separates Sachkonto „Anlagen im Bau" gebucht — über Einkaufsrechnungen oder Fibu-Buchungsblätter. Das Anlagenmodul wird erst bei Aktivierung beteiligt.

```
Bauphase:                    Aktivierung:
┌──────────────────┐        ┌──────────────────────┐
│ EK-Rechnung       │        │ FA Journal            │
│ Soll: AiB-Konto   │        │ FA Posting Type =     │
│ Haben: Kreditor   │        │   Acquisition Cost    │
│                    │        │ Soll: Gebäude-Konto   │
│ KEIN FA-Eintrag!  │        │ Haben: AiB-Konto      │
└──────────────────┘        └──────────────────────┘
```

### Schritt 1 — Sachkonto „Anlagen im Bau" anlegen

```
G/L Account "0700 Anlagen im Bau"
  Account Category = Assets
  Account Subcategory = Tangible Assets
```

### Schritt 2 — Baukosten via Einkaufsrechnung erfassen

```
Purchase Invoice:
  Type = G/L Account
  No. = 0700          ← direkt auf AiB-Konto
  Amount = 50.000 €
```

> Kein FA No., kein FA Posting Type — die Rechnung berührt nur das Sachkonto.

### Schritt 3 — Bei Fertigstellung: Anlage aktivieren

```
1. Fixed Asset "MA-1000 Produktionshalle" anlegen
   FA Posting Group = GEBÄUDE
   Depreciation Book = LINEAR 33J

2. FA Journal erfassen:
   FA No.              = MA-1000
   FA Posting Type     = Acquisition Cost
   Amount              = 140.000 €
   Bal. Account No.    = 0700    ← Gegenkonto: AiB
```

**Ergebnis:**
- `FA Ledger Entry`: `Acquisition Cost` = 140.000 €
- `G/L Entry`: Gebäude 140.000 € an Anlagen im Bau 140.000 €
- AfA-Beginn ab Buchungsdatum

**Vorteile:**
- ✅ Klare bilanzielle Trennung (AiB auf eigenem Konto)
- ✅ Keine Vermischung mit Instandhaltungs-Aufwand
- ✅ Anlagenmodul bleibt sauber (nur fertige Anlagen)
- ✅ Keine Hilfskonstrukte nötig

**Nachteile:**
- ❌ Keine FA-Ledger-Detailhistorie der Bauphase im Anlagenmodul
- ❌ Manuelle Abstimmung AiB-Konto ↔ FA bei Aktivierung

---

## 19.2.3 Ansatz 2: Separate FA-Buchungsgruppe für AiB

**Idee:** Eine zweite FA-Posting-Group, deren `Acquisition Cost Account` auf das AiB-Sachkonto zeigt. Während der Bauphase werden Acquisition-Cost-Buchungen auf diese Gruppe vorgenommen.

```
1. FA Posting Group "ANLAGEN IM BAU":
   Acquisition Cost Account = 0700 (Anlagen im Bau)
   (andere Konten irrelevant — keine AfA während Bauphase)

2. Fixed Asset "MA-1000" anlegen:
   FA Posting Group = ANLAGEN IM BAU
   Depreciation Book = KEIN BUCH (keine AfA während Bau)

3. Baukosten: FA Journal mit FA Posting Type = Acquisition Cost
   → FA Ledger Entry entsteht
   → G/L: AiB-Konto an Kreditor/Bank

4. Aktivierung: FA Posting Group umschlüsseln auf GEBÄUDE
   oder: Disposal aus AiB-Anlage + Acquisition Cost auf Ziel-Anlage
```

**Vorteile:**
- ✅ FA Ledger Entry-Historie bleibt erhalten
- ✅ Bilanzielle Trennung durch eigenes AiB-Konto
- ✅ Keine Maintenance-Fehlinterpretation

**Nachteile:**
- ❌ Höherer Einrichtungsaufwand (eigene FA-Buchungsgruppe)
- ❌ Bei Umgruppierung entstehen Disposal-/Acquisition-Datensätze

---

## 19.2.4 Ansatz 3: Maintenance-Mechanismus (⚠️ nur bedingt geeignet)

Der Maintenance-Ansatz wurde bereits beschrieben — hier die **Probleme im Detail**:

```al
// FAPostingGroup.Table.al, Zeile 243
field(22; "Maintenance Expense Account"; Code[20])
{
    Caption = 'Maintenance Expense Account';
    ToolTip = 'Specifies the general ledger account number to
               post maintenance EXPENSES for fixed assets to...';
}
//                           ^^^^^^^^^
// Dieses Konto ist ein AUFWANDS-Konto, kein Aktivkonto!
```

**Das Kernproblem:** Der `Maintenance Expense Account` wird in der GuV als Instandhaltungsaufwand gebucht. Für AiB bräuchte man ein **Aktivkonto**. Die einzige „Lösung": eine eigene FA-Buchungsgruppe anlegen, in der das `Maintenance Expense Account` auf das AiB-Aktivkonto zeigt — aber das ist eine **Zweckentfremdung** des Feldes.

**Zusätzliche Risiken:**
- Reports werten Maintenance als Instandhaltungsaufwand aus → verzerrtes Reporting
- Verwechslungsgefahr mit echter Instandhaltung (Reparaturen)
- Bei Steuerprüfung erklärungsbedürftig

> **Fazit:** Der Maintenance-Ansatz ist nur dann akzeptabel, wenn man eine **eigene, dedizierte FA-Buchungsgruppe** nur für AiB anlegt und deren Maintenance-Konto auf das AiB-Sachkonto zeigt. Dann ist es aber konzeptionell identisch mit **Ansatz 2** (Separate FA-Buchungsgruppe) — nur mit dem semantisch falschen Posting Type.

---

## 19.2.5 Bilanzielle Darstellung im Vergleich

| Ansatz | AiB in Bilanz separat? | FA-Historie? | Konzeptionelle Sauberkeit |
|---|---|---|---|
| **1. Sachkonto-basiert** | ✅ Ja (eigenes AiB-Konto) | ❌ Nein | ✅✅✅ Optimal |
| **2. Eigene FA-Buchungsgruppe** | ✅ Ja (eigenes AiB-Konto) | ✅ Ja | ✅✅ Gut |
| **3. Maintenance-Mechanismus** | ⚠️ Nur mit Workaround | ✅ Ja | ⚠️ Fragwürdig |

---

## 19.2.6 Entwickler-Referenz

**Relevante Tabellen und Codeunits:**

| Objekt | ID | Zweck |
|---|---|---|
| `FA Posting Group` (Table) | — | 30+ Kontenfelder inkl. `Maintenance Expense Account` |
| `FA Journal Line FA Posting Type` (Enum) | 5602 | Acquisition Cost, Maintenance, Depreciation, … |
| `FA Get G/L Account No.` (Codeunit) | 5612 | Ermittelt Fibu-Konto via `GetMaintenanceExpenseAccount()` |
| `FA Jnl.-Post Line` (Codeunit) | 5600 | Buchung FA Journal → FA Ledger Entry |
| `FA Insert Ledg. Entry` (Codeunit) | 5610 | Anlage FA Ledger Entry-Datensätze |

---

| [← ToDo-Übersicht]({{ '/19-todo/' | relative_url }}) |
