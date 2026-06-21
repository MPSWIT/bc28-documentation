---
title: "ToDo — Anlagen im Bau"
---
# 19.2 Anlagen im Bau (Assets under Construction)

> **Leitfrage:** Wie geht man in Business Central mit Anlagen im Bau um? Wie werden Baukosten gesammelt und aktiviert?

> 📄 **[← ToDo-Übersicht]({{ '/19-todo/' | relative_url }})**

---

## 19.2.1 Das Konzept

Eine fertig angeschaffte Anlage geht sofort in die Abschreibung. **Anlagen im Bau** (AiB, engl. CIP = Construction in Progress) dagegen durchlaufen eine **Sammelphase**, in der Baukosten auflaufen **ohne dass abgeschrieben wird**. Erst nach Fertigstellung und **Aktivierung** beginnt die planmäßige AfA.

BC28 kennt **kein spezielles AiB-Feld** auf der Anlagenkarte. Stattdessen nutzt man den **Maintenance-Mechanismus**:

| Phase | FA Posting Type | Wirkung |
|---|---|---|
| **Bauphase** | `Maintenance` + Maintenance Code | Kosten werden gesammelt, **keine AfA** |
| **Aktivierung** | Transfer auf `Acquisition Cost` | Summe wird Anschaffungskosten → AfA beginnt |

---

## 19.2.2 Die FA-Posting-Typen (Enum 5602)

```al
// FAJournalLineFAPostingType.Enum.al
enum 5602 "FA Journal Line FA Posting Type"
{
    value(0; "Acquisition Cost")      // Anschaffungskosten
    value(1; Depreciation)             // Abschreibung
    value(2; "Write-Down")             // Außerplanmäßige AfA
    value(3; Appreciation)             // Zuschreibung
    value(4; "Custom 1")               // Benutzerdef. 1
    value(5; "Custom 2")               // Benutzerdef. 2
    value(6; Disposal)                 // Abgang
    value(7; Maintenance)              // Instandhaltung ← AiB
    value(8; "Salvage Value")          // Restwert
    value(10; "Bonus Depreciation")    // Bonus-AfA
}
```

> `Maintenance`-Buchungen werden bei aktivierter Fibu-Integration in G/L Entry übernommen, **aber nicht im AfA-Buch** (`Depreciation Book`) als Abschreibungsbasis berücksichtigt.

---

## 19.2.3 Maintenance Code (Feld 26)

Der **Maintenance Code** ist ein Pflichtfeld bei `FA Posting Type = Maintenance` und dient der Gruppierung:

```al
// FAJournalLine.Table.al, Zeile 229
field(26; "Maintenance Code"; Code[10])
{
    Caption = 'Maintenance Code';
    TableRelation = Maintenance;   // Tabelle Maintenance Code

    trigger OnValidate()
    begin
        if "Maintenance Code" <> '' then
            TestField("FA Posting Type",
                       "FA Posting Type"::Maintenance);  // ← nur bei Maintenance
    end;
}
```

**Praxis:** Ein eigener Maintenance Code pro Bauprojekt, z.B.:
- `AI-BAU25` → Neubau Produktionshalle 2025
- `AI-IT`    → IT-Infrastruktur
- `AI-MASCH` → Sondermaschine

---

## 19.2.4 Schritt-für-Schritt: Anlage im Bau

### Schritt 1 — Anlagenkarte anlegen

```
Fixed Asset "MA-1000 Produktionshalle"
  FA Posting Group = GEBÄUDE
  Depreciation Book = LINEAR 33J
```

> Die Anlage bekommt bereits eine AfA-Buch-Gruppe — aber die AfA startet erst nach Aktivierung.

### Schritt 2 — Baukosten als Maintenance buchen

Jede Baukostenrechnung wird über das **Anlagen-Buchungsblatt** (`FA Journal`) gebucht:

```
FA Journal Line:
  FA No.              = MA-1000
  FA Posting Type     = Maintenance
  Maintenance Code    = AI-BAU25
  Amount              = 50.000 €
  Gen. Bus. Posting Group = INLAND
  Gen. Prod. Posting Group = KEINE
```

Das erzeugt:
- **FA Ledger Entry** (Tab. 5602): `FA Posting Type = Maintenance`, Amount = 50.000 €
- **G/L Entry**: Anlagenzugangskonto (Aktivseite) an Kreditor / Bank

### Schritt 3 — Weitere Baukosten sammeln

```
Rechnung Statikbüro:    Maintenance 15.000 €
Rechnung Dachdecker:    Maintenance 45.000 €
Rechnung Elektriker:    Maintenance 30.000 €
                        ─────────────────
Summe Maintenance       140.000 €
```

Nach jeder Buchung: `FA Ledger Entry` mit `Maintenance Code = AI-BAU25`. Keine AfA-Berechnung.

### Schritt 4 — Aktivierung: Transfer auf Acquisition Cost

Nach Fertigstellung wird die Summe von Maintenance auf Acquisition Cost umgebucht:

```
FA Journal Line:
  FA No.              = MA-1000
  FA Posting Type     = Maintenance
  Maintenance Code    = AI-BAU25
  Amount              = -140.000 €     ← gegenbuchen

FA Journal Line:
  FA No.              = MA-1000
  FA Posting Type     = Acquisition Cost
  Amount              = +140.000 €     ← aktivieren
```

> Oder über die Funktion **„Instandhaltung auf Anschaffungskosten buchen"** (Standard-BC-Funktion).

**Ergebnis nach Aktivierung:**
- `FA Ledger Entry`: `Acquisition Cost` = 140.000 €
- `FA Depreciation Book`: `Book Value` = 140.000 €
- **AfA-Berechnung beginnt** ab Buchungsdatum der Aktivierung

---

## 19.2.5 Buchhalterische Abbildung

```
Bauphase (Bilanz):
  Anlagen im Bau (Aktivkonto)    140.000 €
  an Kreditor / Bank                     140.000 €

Aktivierung (Umbuchung):
  Gebäude (Aktivkonto)           140.000 €
  an Anlagen im Bau (Aktivkonto)         140.000 €

Laufende AfA (GuV):
  Abschreibung Gebäude             4.242 € p.a.
  an kumulierte AfA Gebäude                4.242 € p.a.
```

> **Der Wertefluss:** `Maintenance` → `Acquisition Cost` → `Depreciation` — alle drei sind `FA Posting Types`, die über die Buchungsmatrix in die Fibu gelangen.

---

## 19.2.6 Alternativer Ansatz: Komponenten-Anlage

Statt Maintenance-Buchungen kann man auch eine **Komponenten-Anlage** (Sub-Asset) verwenden:

```
HAUPT-1000  "Produktionshalle"       ← Hauptanlage (aktiviert)
  ├── MA-1001 "Rohbau"               ← Komponente
  ├── MA-1002 "Dach & Fassade"       ← Komponente
  └── MA-1003 "Elektroinstallation"  ← Komponente
```

Jede Komponente wird einzeln mit `Acquisition Cost` bebucht. Nach Fertigstellung erfolgt die **Konsolidierung** via `Disposal` + `Acquisition Cost` auf die Hauptanlage. Dieser Weg bietet feinere Kontrolle über Teil-Aktivierungen, ist aber aufwändiger.

---

## 19.2.7 Entwickler-Referenz

**Integration Events bei FA-Buchung:**

```al
// FAInsertLedgerEntry.Codeunit.al
[IntegrationEvent(false, false)]
procedure OnBeforeInsertFALedgerEntry(
    var FAJournalLine: Record "FA Journal Line";
    var FALedgerEntry: Record "FA Ledger Entry")
begin
end;
```

**Wichtige Codeunits:**

| Codeunit | Aufgabe |
|---|---|
| 5600 `FA Jnl.-Post Line` | Buchung der FA Journal Line → FA Ledger Entry |
| 5610 `FA Insert Ledg. Entry` | Anlage der FA Ledger Entry-Datensätze |
| 5612 `FA Get G/L Account No.` | Ermittlung Fibu-Konto aus FA Posting Group + FA Posting Type |

---

| [← ToDo-Übersicht]({{ '/19-todo/' | relative_url }}) |
