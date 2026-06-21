---
title: "ToDo — Anlagen im Bau"
---
# 19.2 Anlagen im Bau (Assets under Construction)

> **Leitfrage:** Wie geht man in Business Central mit Anlagen im Bau um? Bilanzielle Trennung, Baukosten sammeln, Aktivierung.

> 📄 **[← ToDo-Übersicht]({{ '/19-todo/' | relative_url }})**

---

## 19.2.1 Das Problem

**Anlagen im Bau** (AiB, CIP) müssen bilanziell **getrennt** von fertigen Anlagen ausgewiesen werden. Erst nach Fertigstellung werden sie in die endgültige Anlagenklasse umgebucht und die AfA beginnt.

BC28 hat kein natives AiB-Feld auf der Anlagenkarte. Es gibt drei Wege, diese Anforderung abzubilden — die ersten beiden sind technisch solide, der dritte einfach, aber ohne Anlagenhistorie.

---

## 19.2.2 Ansatz 1: Maintenance-Mechanismus + Buchungsgruppenwechsel

> **Grundidee:** Während der Bauphase werden alle Kosten als `Maintenance` gebucht (keine AfA). Bei Fertigstellung wechselt die `FA Posting Group` von AiB auf die Zielgruppe, und die Maintenance-Summe wird auf `Acquisition Cost` transferiert.

### Warum Maintenance funktioniert — Quellcode-Beweis

Der Maintenance-Ansatz ist **nicht** auf Instandhaltungs-Aufwand beschränkt. Vier Fakten aus dem BC28-Quellcode belegen das:

```al
// 1. Keine Sperre: Maintenance kann OHNE vorhandene Acquisition Cost gebucht werden
// FAJnlCheckLine.Codeunit.al — CheckJnlLine() prüft NUR das G/L Integration-Flag:
case FAPostingType of
    FAPostingType::Maintenance:
        GLIntegration := DeprBook."G/L Integration - Maintenance";
end;
// → Kein Check auf vorhandene Acquisition Cost!

// 2. Maintenance zählt NICHT zum Buchwert / AfA-Basis
// FADepreciationBook.Table.al, field(15):
CalcFormula = sum("FA Ledger Entry".Amount where(
    "FA Posting Type" = const("Acquisition Cost")));
// → FlowField filtert exklusiv auf Acquisition Cost

// 3. "Bereit für Aktivierung" prüft NICHT auf vorhandene Acquisition Cost
// FADepreciationBook.Table.al — RecIsReadyForAcquisition():
if ("Depreciation Book Code" = FASetup."Default Depr. Book") and
   ("FA Posting Group" <> '') and
   ("Depreciation Starting Date" > 0D)
then ...  // → KEIN Check auf Acq. Cost > 0!

// 4. G/L-Integration ist pro Buchungstyp granular schaltbar
// DepreciationBook.Table.al:
field(10; "G/L Integration - Maintenance"; Boolean)
// → de-AT: "Fibu-Integr. - Wartung"
```

Das `Maintenance Expense Account` (Feld 22 in `FAPostingGroup.Table.al`) heißt zwar „Expense", aber wohin es zeigt, bestimmt der Anwender. Es kann auf ein **Aktivkonto** „Anlagen im Bau" zeigen — die Software erzwingt das nicht.

### Ausgangssituation

```
Depreciation Book (Tabelle "Depreciation Book"):
  ┌─────────────────────────────────────────┐
  │ Code = "LINEAR 33J"                     │
  │ G/L Integration - Acq. Cost    = ☑      │
  │ G/L Integration - Maintenance  = ☑      │
  │ G/L Integration - Depreciation = ☑      │
  └─────────────────────────────────────────┘
```

### Schritt 1 — Eigene FA-Buchungsgruppe „ANLAGEN IM BAU"

```
FA Posting Group: ANLAGEN IM BAU
  Acquisition Cost Account      = 0750 (AiB-Anschaffungskosten)
  Maintenance Expense Account   = 0700 (Anlagen im Bau)    ← Aktivkonto!
  Maintenance Bal. Acc.         = 9999 (Verrechnungskonto)
  Depreciation Expense Acc.     = 6520 (Abschreibung)
  (alle anderen Konten wie üblich)
```

### Schritt 2 — Anlage & AfA-Buch vorbereiten

```
Fixed Asset "MA-1000 Produktionshalle"
  FA Posting Group = ANLAGEN IM BAU

FA Depreciation Book:
  Depreciation Book Code    = LINEAR 33J
  FA Posting Group          = ANLAGEN IM BAU
  Depreciation Starting Date = 01.01.2026  (Zukunft — AfA startet später)
```

### Schritt 3 — Baukosten sammeln via FA Journal

```
FA Journal Batch "AiB-SAMMEL":
  Zeile 1: FA No. = MA-1000
           FA Posting Type = Maintenance
           Maintenance Code = AI-BAU25
           Amount = 50.000  (Rechnung Rohbau)
           Bal. Account No. = KRED-12345

  Zeile 2: FA No. = MA-1000
           FA Posting Type = Maintenance
           Maintenance Code = AI-BAU25
           Amount = 15.000  (Statik)
           Bal. Account No. = KRED-67890

  ... Summe: 140.000 €

→ FA Ledger Entry: 3× Maintenance à Summe 140.000 €
→ G/L Entry: Anlagen im Bau 140.000 € an Kreditor 140.000 €
→ AfA-Buch: Keine Veränderung (Maintenance wird nicht abgeschrieben)
```

### Schritt 4 — Aktivierung: Umbuchen + Buchungsgruppe wechseln

```
A) Maintenance-Summe gegenbuchen:
   FA Journal:
     FA Posting Type = Maintenance
     Maintenance Code = AI-BAU25
     Amount = -140.000 €
     Bal. Account No. = 0700

B) Acquisition Cost buchen:
   FA Journal:
     FA Posting Type = Acquisition Cost
     Amount = +140.000 €
     Bal. Account No. = 0700 (Anlagen im Bau)

   → AiB-Konto auf Null, Gebäude aktiviert

C) Buchungsgruppe umschlüsseln:
   Fixed Asset "MA-1000":
     FA Posting Group = GEBÄUDE    (statt ANLAGEN IM BAU)
   
   FA Depreciation Book:
     FA Posting Group = GEBÄUDE
     Depreciation Starting Date = 01.01.2025   ← Heute/Jetzt
```

**Ergebnis:**
- `FA Ledger Entry` enthält die vollständige Bauhistorie (Maintenance-Einträge)
- Danach: `Acquisition Cost` = 140.000 € → AfA-Buch aktualisiert Buchwert
- `G/L`: Anlagen im Bau-Konto = 0 €, Gebäude-Konto = 140.000 €
- AfA-Berechnung startet ab neuem Starting Date

**Vorteile:**
- ✅ FA Ledger Entry-Historie bleibt erhalten
- ✅ Bilanzielle Trennung durch eigenes AiB-Konto
- ✅ Keine Zweckentfremdung (FA-Buchungsgruppe sauber getrennt)

**Nachteile:**
- ❌ Einrichtungsaufwand (eigene FA-Buchungsgruppe)
- ❌ Manueller Wechsel der Buchungsgruppe bei Aktivierung

---

## 19.2.3 Ansatz 2: Mit der Fibu-Integration spielen

> **Grundidee:** Man verwendet ein AfA-Buch, in dem die G/L-Integration für bestimmte FA Posting Types **deaktiviert** wird. So können Maintenance-Buchungen im Anlagenmodul existieren, ohne in die Fibu zu gehen. Die Fibu wird separat bedient — entweder über das Sachkonto oder über ein zweites AfA-Buch.

### Das AfA-Buch (Depreciation Book) als Schaltzentrale

```al
// DepreciationBook.Table.al — die G/L Integration-Flags pro Buchungstyp
field(3;  "G/L Integration - Acq. Cost";    Boolean)
field(4;  "G/L Integration - Depreciation";  Boolean)
field(10; "G/L Integration - Maintenance";   Boolean)
//       → de-AT: "Fibu-Integr. - Wartung"
```

Jeder FA Posting Type hat einen eigenen Schalter, ob Buchungen in die Fibu durchgereicht werden.

### Variante A: Zwei AfA-Bücher — eines für AiB, eines für die fertige Anlage

```
Depreciation Book "AiB-SAMMEL":           (für Bauphase)
  G/L Integration - Acq. Cost    = ☑
  G/L Integration - Maintenance  = ☑    ← Fibu: AiB-Konto
  G/L Integration - Depreciation = ☐    ← keine AfA-Buchung in Fibu

Depreciation Book "LINEAR 33J":           (für fertige Anlage)
  G/L Integration - Acq. Cost    = ☑
  G/L Integration - Maintenance  = ☐
  G/L Integration - Depreciation = ☑
```

**Ablauf:**

```
1. Anlage MA-1000 bekommt beide Bücher zugewiesen

2. Bauphase: Maintenance-Buchungen ins AiB-SAMMEL-Buch
   → FA Ledger Entry mit Depr. Book = AiB-SAMMEL
   → Fibu: AiB-Konto

3. Aktivierung: 
   a) Maintenance-Summe gegenbuchen (AiB-SAMMEL)
   b) Acquisition Cost buchen (LINEAR 33J)
   c) AiB-SAMMEL-Buch deaktivieren oder löschen
```

**Vorteile:**
- ✅ Kein Wechsel der FA-Buchungsgruppe nötig
- ✅ Fibu-Integration granular steuerbar
- ✅ Jedes AfA-Buch kann eigene Buchungsgruppe haben (Feld 13 im FA Depr. Book)

**Nachteile:**
- ❌ Zwei AfA-Bücher zu pflegen
- ❌ Verwirrend für Anwender („Warum zwei Bücher?")

### Variante B: G/L-Integration temporär deaktivieren

```
Bauphase:
  Depreciation Book "LINEAR 33J":
    G/L Integration - Maintenance  = ☐
    → Maintenance erzeugt FA Ledger Entry, aber KEINE Fibu-Buchung

  Fibu-Buchung separat:
    Gen. Journal: AiB-Konto an Kreditor
    → Kosten sind in der Fibu, aber nicht im Anlagenmodul

Aktivierung:
  Depreciation Book "LINEAR 33J":
    G/L Integration - Acquisition Cost = ☑
  FA Journal: Acquisition Cost 140.000 €
    → FA Ledger Entry entsteht
    → Fibu: Gebäude an AiB-Konto
```

**Vorteile:**
- ✅ Nur ein AfA-Buch
- ✅ Maximale Flexibilität

**Nachteile:**
- ❌ Manuelle Fibu-Buchungen während der Bauphase
- ❌ Abstimmarbeit zwischen FA und Fibu nötig

---

## 19.2.4 Ansatz 3: Rein Sachkonto-basiert (einfach, aber ohne FA-Historie)

> Alle Baukosten direkt auf ein Sachkonto „Anlagen im Bau" via Einkaufsrechnungen buchen. Das Anlagenmodul wird erst bei Aktivierung beteiligt.

| Phase | Was passiert | FA beteiligt? |
|---|---|---|
| Bauphase | EK-Rechnung → Sachkonto 0700 „Anlagen im Bau" | Nein |
| Aktivierung | FA Journal: Acq. Cost → Gebäude-Konto, Gegenkonto 0700 | Ja |

**Vorteile:** Kein Einrichtungsaufwand, keine Workarounds.
**Nachteile:** Keine FA Ledger Entry-Historie der Baukosten, keine buchhalterische Nachvollziehbarkeit im Anlagenmodul.

---

## 19.2.5 Bewertung im Vergleich

| Kriterium | Ansatz 1 (Maintenance + Wechsel) | Ansatz 2 (Fibu-Integration) | Ansatz 3 (Sachkonto) |
|---|---|---|---|
| FA-Ledger-Historie | ✅ Ja | ✅ Ja | ❌ Nein |
| Bilanzielle Trennung | ✅ Ja (eigene Gruppe) | ✅ Ja (eigenes Buch) | ✅ Ja (eigenes Konto) |
| Einrichtungsaufwand | Mittel (eine extra Gruppe) | Hoch (zwei Bücher) | Gering |
| AfA-Beginn steuerbar | ✅ Via Starting Date | ✅ Via Starting Date | ✅ Via Starting Date |
| Berichtsanforderungen (Anlagenspiegel) | ✅ FA-Daten komplett | ✅ FA-Daten komplett | ⚠️ Nur aktivierte Werte |
| Betriebsprüfung | Gut erklärbar | Gut erklärbar | Einfach erklärbar |

> **Empfehlung:** Ansatz 1 (Maintenance + Buchungsgruppenwechsel) bietet den besten Kompromiss aus Nachvollziehbarkeit und Einrichtungsaufwand. Ansatz 2 ist die sauberste Lösung für Unternehmen mit komplexem Anlagenportfolio und mehreren AiB-Projekten parallel.

---

## 19.2.6 Abgleich mit Bilanzierungsanforderungen (freefinance.at / HGB / UGB)

Die fachlichen Anforderungen aus dem Bilanzrecht lassen sich wie folgt in BC28 abbilden:

| Bilanzielle Anforderung | Umsetzung in BC28 (Ansatz 1) | Status |
|---|---|---|
| **Materialkosten, Arbeitskosten aktivieren** | FA Journal: `FA Posting Type = Maintenance`, `Maintenance Code = AI-BAU25` → FA Ledger Entry | ✅ |
| **Fremdkapitalzinsen aktivieren** (wenn direkt zurechenbar) | Zusätzliche FA Journal-Zeile: Maintenance → AiB-FA-Buchungsgruppe | ✅ |
| **Nebenkosten aktivieren** | Wie Materialkosten — gleicher Maintenance-Code | ✅ |
| **Indirekte Kosten NICHT aktivieren** | Werden nicht auf AiB-Buchungsgruppe gebucht (manuelle Trennung) | ✅ |
| **Regelmäßige Überprüfung/Anpassung** | FA Ledger Entries sind stornierbar / korrigierbar via Gegenbuchung | ✅ |
| **Umbuchung bei Fertigstellung** | Maintenance-Summe gegenbuchen + Acquisition Cost buchen + Buchungsgruppe wechseln | ✅ |
| **AfA-Beginn ab Betriebsbereitschaft** | `Depreciation Starting Date` auf Fertigstellungsdatum setzen, `No. of Depreciation Years` definieren | ✅ |

> **Fazit:** Alle bilanzrechtlichen Anforderungen an Anlagen im Bau sind mit dem Maintenance-Ansatz + Buchungsgruppenwechsel **vollständig abbildbar**. Der einzige manuelle Schritt ist die Trennung direkter vs. indirekter Kosten — das kann BC nicht automatisch, weil die Zuordnung eine unternehmerische Entscheidung ist.

## 19.2.7 Entwickler-Referenz

### Relevante Quellcode-Objekte

```al
// FAJournalLineFAPostingType.Enum.al — die Buchungstypen (5602)
value(0; "Acquisition Cost")   // Anschaffungskosten
value(7; Maintenance)           // Instandhaltung (→ AiB-Mechanismus)

// FAPostingGroup.Table.al — Kontenzuordnung
field(22; "Maintenance Expense Account"; Code[20])
field(24; "Acquisition Cost Bal. Acc."; Code[20])

// DepreciationBook.Table.al — Fibu-Integration
field(3;  "G/L Integration - Acq. Cost";    Boolean)
field(10; "G/L Integration - Maintenance";   Boolean)

// FA Depreciation Book.Table.al — je Anlage+Buch
field(13; "FA Posting Group"; Code[20])  // pro Buch individuell!
field(28; "Maintenance Code Filter"; Code[10])
```

### Integration Events

```al
// FA Jnl.-Post Line (Codeunit 5600)
[IntegrationEvent(false, false)]
procedure OnBeforeInsertFALedgerEntry(
    var FAJournalLine: Record "FA Journal Line";
    var FALedgerEntry: Record "FA Ledger Entry")

// FA Get G/L Account No. (Codeunit 5612)
procedure GetMaintenanceAccNo(
    var MaintenanceLedgEntry: Record "Maintenance Ledger Entry"): Code[20]
```

---

| [← ToDo-Übersicht]({{ '/19-todo/' | relative_url }}) |
