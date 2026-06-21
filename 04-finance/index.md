---
title: "Finanzwesen"
---
# 4. Finanzwesen — General Ledger Setup

> **Tabelle 98:** `GeneralLedgerSetup.Table.al`
> **Namespace:** `Microsoft.Finance.GeneralLedger.Setup`
> **Page:** `GeneralLedgerSetup.Page.al`
> **Typ:** Singleton (nur ein Datensatz, Primary Key = fixer Code)

Die Tabelle **General Ledger Setup (98)** ist die zentrale Schaltstelle der gesamten Finanzbuchhaltung.
Sie steuert Buchungszeiträume, MwSt-Verhalten, Rundung, Dimensionen und Zahlungstoleranzen.

---

## 4.1 Buchungszeiträume

### Feld 2: `Allow Posting From` (Date)
Earliest date allowed for posting transactions to the general ledger.

```al
field(2; "Allow Posting From"; Date)
{
    Caption = 'Allow Posting From';
    trigger OnValidate()
    begin
        CheckAllowedPostingDates(0);          // Codeunit "User Setup Management"
        if xRec."Allow Posting From" <> Rec."Allow Posting From" then begin
            if Rec."Allow Posting From" <> 0D then
                Evaluate(Rec."Allow Posting From DateFormula", '');
            CheckDateRange();
        end;
    end;
}
```

**Beispiel 1 — Geschäftsjahresbeginn:**
> `Allow Posting From = 01.01.2026`, `Allow Posting To = leer`
> → Alle Buchungen ab dem 01.01.2026 sind erlaubt. Kein Enddatum — Buchungen bis in die Zukunft möglich.

**Beispiel 2 — Monatsabschluss-Only:**
> `Allow Posting From = 01.06.2026`, `Allow Posting To = 30.06.2026`
> → Nur Buchungen im Juni 2026 sind erlaubt. Versucht jemand am 05.07.2026 zu buchen, erscheint ein Fehler via `CheckAllowedPostingDates()`.

**Beispiel 3 — Offene Periode mit DateFormula:**
> `Allow Posting From DateFormula = "-1M"`, `Allow Posting To DateFormula = "CM"` (aktueller Monat)
> → Dynamisch: Heute = 21.06.2026 → Buchungen vom 21.05.2026 bis 30.06.2026 erlaubt. Am 01.07.2026 verschiebt sich der Zeitraum automatisch.

**Wichtiger Code:**
- `CheckDateRange()` (lokal) — kombiniert feste Daten mit DateFormulas
- `FirstAllowedPostingDate()` — berücksichtigt auch `Inventory Period` (keine Buchung in gesperrte Lagerperiode)
- `IsPostingAllowed(PostingDate: Date): Boolean` — Integration-Event `OnAfterIsPostingAllowed`
- `CheckAllowedPostingDates()` delegiert an **Codeunit "User Setup Management"**

---

### Feld 4: `Register Time` (Boolean)
Records posting time in addition to posting date for audit trail.

**Beispiel 1:**
> `Register Time = True`
> → Jede gebuchte Transaktion erhält zusätzlich zum Buchungsdatum einen Zeitstempel. Nachvollziehbar, wann genau innerhalb eines Tages gebucht wurde. Nützlich für Revisionssicherheit.

**Beispiel 2:**
> `Register Time = False`
> → Default. Nur das Buchungsdatum wird gespeichert. Spart Speicherplatz in den Ledger-Entry-Tabellen.

**Beispiel 3 — Kombination mit Job Queue:**
> `Post with Job Queue = True` + `Register Time = True`
> → Auch Hintergrundbuchungen erhalten Zeitstempel. Bei Massenbuchungen (z.B. Zinsrechnung) kann man exakt sehen, welche Transaktion wann verarbeitet wurde.

---

### Feld 169: `Posting Preview Type` (Enum "Posting Preview Type")
Determines the type of posting preview shown to users before finalizing transactions.

**Beispiel 1:**
> `Posting Preview Type = Detailed`
> → Vor jeder Buchung wird eine Vorschau aller entstehenden Ledger-Entries (Sachposten, Debitoren/Kreditoren, MwSt) angezeigt. Der Benutzer kann prüfen, bevor final gebucht wird.

**Beispiel 2:**
> `Posting Preview Type = None` (Vorschau deaktiviert)
> → Keine Vorschau. Buchungen werden sofort ausgeführt. Erhöht die Geschwindigkeit bei Routine-Buchungen, birgt aber Fehlerrisiko.

**Beispiel 3 — Zugriff auf Vorschau-Objekte:**
> Bei `Detailed` wird die Preview-Tabelle **G/L Entry Posting Preview** befüllt. Entwickler können darauf im `OnBeforePost`-Event einer Sales-Posting-Codeunit zugreifen.

**Relevante Objekte:**
- `GLEntryPostingPreview.Table.al` — Vorschau-Tabelle
- `VATEntryPostingPreview.Table.al` — MwSt-Posting-Vorschau

---

## 4.2 Dimensionen

### Feld 79/80: `Global Dimension 1 Code` / `Global Dimension 2 Code` (Code[20])
Primary and secondary global dimension codes used throughout the system.

```al
field(79; "Global Dimension 1 Code"; Code[20])
{
    Editable = false;       // Nicht direkt editierbar!
    TableRelation = Dimension;
    trigger OnValidate()
    begin
        "Shortcut Dimension 1 Code" := "Global Dimension 1 Code";
    end;
}
```

Die Global Dimensions werden **nicht direkt** in dieser Tabelle gesetzt, sondern über die Dimension-Tabelle und `UpdateDimValueGlobalDimNo()`.

**Beispiel 1 — Abteilung + Kostenstelle:**
> `Global Dimension 1 Code = "ABTEILUNG"`, `Global Dimension 2 Code = "KOSTENSTELLE"`
> → Jede Buchung MUSS eine Abteilung und eine Kostenstelle haben (wenn als `Code Mandatory` auf der Dimension aktiviert). Auf Sachkonten können Standard-Dimensionswerte hinterlegt werden.

**Beispiel 2 — Projekt + Region:**
> `Global Dimension 1 Code = "PROJEKT"`, `Global Dimension 2 Code = "REGION"`
> → Alle Reports, FlowFields (`Cust. Balances Due`, `Vendor Balances Due`) und Analyseansichten arbeiten mit diesen beiden Filtern. Der Filter `Global Dimension 1 Filter` auf der GL Setup Page zeigt nur die Summe für die gefilterte Dimension.

**Beispiel 3 — Nur eine Global Dimension:**
> `Global Dimension 1 Code = "KOSTENSTELLE"`, `Global Dimension 2 Code = ""`
> → Nur Kostenstellen sind global pflicht. Andere Dimensionen (Shortcut 3–8) sind optional.

**FlowFields, die Global Dimensions nutzen:**
```al
field(44; "Cust. Balances Due"; Decimal)
{
    FieldClass = FlowField;
    CalcFormula = sum("Detailed Cust. Ledg. Entry"."Amount (LCY)"
        where("Initial Entry Global Dim. 1" = field("Global Dimension 1 Filter"),
              "Initial Entry Global Dim. 2" = field("Global Dimension 2 Filter")));
}
```

> ⚠️ **Wichtig für Entwickler:** `Editable = false` — diese Felder werden nur über `UpdateDimValueGlobalDimNo()` (auf Dim-Änderung) gesetzt. Direkter Zugriff über AL wird ignoriert.

---

### Felder 81–88: `Shortcut Dimension 1..8 Code` (Code[20])
Shortcut dimension codes for quick access in data entry forms.

**Beispiel 1 — Typische Konfiguration:**
> Shortcut 1 = Abteilung, Shortcut 2 = Kostenstelle (automatisch aus Global Dim 1 & 2)
> Shortcut 3 = Vertreter, Shortcut 4 = Region, Shortcut 5 = Kampagne
> → Auf jeder Buchungsseite erscheinen diese 5 Dimensionen als Schnellzugriff oben im Dim-Bereich.

**Beispiel 2 — Nur Shortcut 3–4 belegt:**
> Shortcut 3 = "PROJEKT", Shortcut 4 = "KOSTENTRÄGER", Shortcut 5–8 = leer
> → Auf Belegseiten sind nur 4 Dimensionen sichtbar (2 Global + 2 Shortcut). Spart Platz, reicht für einfache Anforderungen.

**Beispiel 3 — Umbenennung einer Shortcut-Dimension:**
> Shortcut 3 Code wird von "VERTRETER" auf "TEAM" geändert
> → `UpdateDimValueGlobalDimNo()` aktualisiert alle `Dimension Value`-Einträge: `Global Dimension No.` wird auf 3 gesetzt für den neuen Code, auf 0 für den alten. `Dimension Set Entry` wird ebenfalls via `UpdateGlobalDimensionNo()` synchronisiert.

**Relevanter Code:**
- `UpdateDimValueGlobalDimNo(xDimCode, DimCode, ShortcutDimNo)` — lokale Prozedur in Table 98
- `OnAfterUpdateDimValueGlobalDimNo()` — Integration Event für Extensions
- **Codeunit "Dimension Management"** — `ShowDimensionSet()` für Popup

---

## 4.3 MwSt / VAT

### Feld 7: `VAT Reporting Date` (Enum "VAT Reporting Date")
Default VAT reporting date calculation method.

```al
field(7; "VAT Reporting Date"; Enum "VAT Reporting Date")
{
    Caption = 'Default VAT Date';
}
```

**Beispiel 1 — Buchungsdatum als VAT-Datum:**
> `VAT Reporting Date = Posting Date`
> → Die MwSt wird immer mit dem Buchungsdatum gemeldet. Typisch für IST-Versteuerung.

**Beispiel 2 — Belegdatum als VAT-Datum:**
> `VAT Reporting Date = Document Date`
> → Die MwSt wird mit dem Belegdatum gemeldet. Bei einer Rechnung vom 15.06., die erst am 02.07. gebucht wird, erscheint die MwSt im Juni-Zeitraum. Relevant für SOLL-Versteuerung.

**Beispiel 3 — VAT Date Usage ergänzt:**
> `VAT Reporting Date = Document Date`, `VAT Reporting Date Usage = Disabled`
> → Die MwSt-Meldung verwendet trotz Einstellung NICHT das Belegdatum, sondern fällt auf Posting Date zurück. Gesteuert über Enum "VAT Reporting Date Usage" (Feld 8).

**Relevanter Code:**
- `GetVATDate(PostingDate, DocumentDate): Date` — liefert das effektive VAT-Datum
- `UpdateVATDate(NewDate, VATDateType, var VATDate)` — setzt VAT-Datum wenn Typ übereinstimmt

---

### Feld 48: `Unrealized VAT` (Boolean)
Enables unrealized VAT for cash-based VAT reporting.

```al
trigger OnValidate()
begin
    if not "Unrealized VAT" then begin
        // Prüft, ob es VAT Posting Setup mit Unrealized VAT Type >= Percentage gibt
        VATPostingSetup.SetFilter("Unrealized VAT Type", '>=%1', ...);
        if VATPostingSetup.FindFirst() then
            Error(...);  // Darf nicht deaktiviert werden!
    end;
    if "Unrealized VAT" then
        "Prepayment Unrealized VAT" := true
    else
        "Prepayment Unrealized VAT" := false;
end;
```

**Beispiel 1 — Ist-Versteuerung:**
> `Unrealized VAT = True`
> → MwSt wird erst bei Zahlungseingang fällig (nicht bei Rechnungsstellung). Bei Buchung einer Ausgangsrechnung entsteht eine **unrealisierte MwSt**, die erst beim Zahlungsausgleich realisiert wird. Benötigt entsprechende `VAT Posting Setup`-Einträge.

**Beispiel 2 — Soll-Versteuerung ohne Unrealized VAT:**
> `Unrealized VAT = False`
> → MwSt wird sofort bei Rechnungsstellung fällig. Die Prüfung im `OnValidate` stellt sicher, dass keine `VAT Posting Setup`-Einträge mit `Unrealized VAT Type = Percentage` existieren (sonst Error).

**Beispiel 3 — Aktivierung blockiert:**
> Es existiert ein `VAT Posting Setup` mit `Unrealized VAT Type = Percentage` für `VAT Bus. Posting Group = INLAND` und `VAT Prod. Posting Group = NORMAL`
> → Wenn jetzt `Unrealized VAT = False` gesetzt wird, erscheint Fehlermeldung _„VAT Posting Setup INLAND NORMAL have Unrealized VAT Type to Percentage.“_
> → Lösung: Erst VAT Posting Setup bereinigen, dann Unrealized VAT deaktivieren.

---

### Feld 103: `Bill-to/Sell-to VAT Calc.` (Enum "G/L Setup VAT Calculation")
Specifies whether VAT calculation is based on bill-to/sell-to or ship-to address.

**Beispiel 1 — Standard: Rechnungsadresse:**
> `Bill-to/Sell-to VAT Calc. = Bill-to/Sell-to`
> → Die MwSt wird anhand des Landes der Rechnungsadresse ermittelt. Ein Kunde in Deutschland mit Lieferadresse Österreich bekommt deutsche MwSt.

**Beispiel 2 — Lieferadresse:**
> `Bill-to/Sell-to VAT Calc. = Ship-to`
> → MwSt richtet sich nach dem Land der Lieferadresse. Wichtig für EU-Versand: Lieferung nach Frankreich → französische MwSt-Regeln.

**Beispiel 3 — Cross-Border-Szenario:**
> Ein Schweizer Kunde (Bill-to: CH) lässt nach Deutschland liefern (Ship-to: DE).
> `Bill-to/Sell-to VAT Calc. = Bill-to/Sell-to` → CH-MwSt (keine EU-MwSt)
> `Bill-to/Sell-to VAT Calc. = Ship-to` → DE-MwSt (innergemeinschaftliche Lieferung)
> → Falsche Einstellung kann zu massiven MwSt-Fehlern im Reporting führen.

---

### Feld 188: `Control VAT Period` (Enum "VAT Period Control")
Controls VAT period validation and posting restrictions.

**Beispiel 1 — Keine Kontrolle:**
> `Control VAT Period = Disabled`
> → Keine Validierung. Buchungen sind unabhängig vom MwSt-Zeitraum möglich.

**Beispiel 2 — Warnung:**
> `Control VAT Period = Warning`
> → Bei Buchung außerhalb des offenen MwSt-Zeitraums erscheint eine Warnung, aber die Buchung ist trotzdem möglich. Ideal für Übergangsphasen.

**Beispiel 3 — Fehler (hart):**
> `Control VAT Period = Error`
> → Buchung außerhalb des offenen MwSt-Zeitraums wird mit Fehler abgelehnt. Verhindert versehentliche Buchungen in bereits gemeldete MwSt-Perioden. Nutzt `FeatureTelemetry.LogUsage()` beim Wechsel der Einstellung.

---

### Feld 72: `VAT Exchange Rate Adjustment` (Enum "Exch. Rate Adjustment Type")
Method for adjusting VAT amounts during currency exchange rate adjustments.

**Beispiel 1 — Keine Anpassung:**
> `VAT Exchange Rate Adjustment = No Adjustment`
> → Wechselkursdifferenzen werden nur für das Sachkonto berechnet, nicht für MwSt. MwSt bleibt unverändert.

**Beispiel 2 — Anpassung pro Eintrag:**
> `VAT Exchange Rate Adjustment = Per Entry`
> → Jeder MwSt-Posten wird einzeln mit dem neuen Wechselkurs bewertet. Detailliert, aber rechenintensiv.

**Beispiel 3 — Zusammenfassende Anpassung:**
> Die MwSt-Differenzen werden gesammelt auf ein separates Konto gebucht. Verwendet `VAT Posting Setup` für die Kontenfindung.

---

## 4.4 Zahlungstoleranzen & Skonto

### Feld 94/95: `Payment Tolerance %` / `Max. Payment Tolerance Amount` (Decimal)
Payment tolerance allowed for customer and vendor payment applications (Editable = false).

> ⚠️ Diese Felder sind **nicht editierbar** (`Editable = false`). Sie werden vom System über die **Codeunit "Payment Tolerance"** berechnet.

**Beispiel 1 — Toleranz aktiv:**
> `Payment Tolerance % = 2`, `Max. Payment Tolerance Amount = 50,00`
> → Ein Kunde zahlt 980,00 € auf eine Rechnung über 1.000,00 €. Differenz = 20,00 € = 2%. Da unter 2% UND unter 50,00 € → automatischer Ausgleich möglich.

**Beispiel 2 — Nur Max-Betrag erreicht:**
> `Payment Tolerance % = 1`, `Max. Payment Tolerance Amount = 100,00` — Rechnung 10.000 €, Zahlung 9.850 € = 150 € Differenz = 1,5%. Prozentsatz überschritten, aber Betrag unter 100 €? Nein, 150 > 100 → Kein automatischer Ausgleich.

**Beispiel 3 — Toleranz deaktiviert:**
> `Payment Tolerance % = 0`, `Max. Payment Tolerance Amount = 0`
> → Jede Differenz führt zu offenem Posten. `GetPmtToleranceVisible()` returniert `false` → Toleranz-Felder auf der Page unsichtbar.

---

### Feld 99/92: `Payment Tolerance Posting` / `Pmt. Disc. Tolerance Posting` (Option)
Specifies how tolerance amounts are posted.

```al
OptionMembers = "Payment Tolerance Accounts","Payment Discount Accounts";
```

**Beispiel 1 — Auf Toleranz-Konten:**
> `Payment Tolerance Posting = Payment Tolerance Accounts`
> → Die 20 € Differenz aus obigem Beispiel werden auf das Konto aus `Gen. Posting Setup → Payment Tolerance Account` gebucht. Saubere Trennung vom Skonto.

**Beispiel 2 — Auf Skonto-Konten:**
> `Payment Tolerance Posting = Payment Discount Accounts`
> → Die Toleranz wird wie Skonto behandelt und auf das Skonto-Konto gebucht. Wird in der MwSt-Berechnung wie Skonto berücksichtigt. Kann MwSt-Differenzen erzeugen.

**Beispiel 3 — Kombination mit `Pmt. Disc. Tolerance Warning` (Feld 100):**
> `Pmt. Disc. Tolerance Warning = True` + Toleranz überschritten
> → Warnung erscheint, aber Buchung möglich. Ohne Warning = stillschweigende Buchung.

---

### Feld 28: `Pmt. Disc. Excl. VAT` (Boolean)
Calculates payment discounts excluding VAT amounts.

```al
trigger OnValidate()
begin
    if "Pmt. Disc. Excl. VAT" then
        TestField("Adjust for Payment Disc.", false);   // Darf NICHT aktiv sein
    else
        TestField("VAT Tolerance %", 0);                 // Darf NICHT >0 sein
end;
```

**Beispiel 1 — Skonto auf Netto:**
> `Pmt. Disc. Excl. VAT = True`
> → Rechnung 1.190 € brutto (1.000 € netto + 190 € MwSt). 2% Skonto = 20 € (nur vom Netto!). MwSt-Korrektur separat. Realistisches Szenario für B2B. **Bedingung:** `Adjust for Payment Disc. = False`.

**Beispiel 2 — Skonto auf Brutto:**
> `Pmt. Disc. Excl. VAT = False`
> → Rechnung 1.190 € brutto, 2% Skonto = 23,80 €. MwSt wird implizit mit-skontiert. **Bedingung:** `VAT Tolerance % = 0`.

**Beispiel 3 — Fehler bei Konflikt:**
> `Pmt. Disc. Excl. VAT = True` UND `Adjust for Payment Disc. = True`
> → Fehler beim Validieren! Diese Kombination schließen sich gegenseitig aus. `TestField` prüft auf `false`.

---

### Feld 49: `Adjust for Payment Disc.` (Boolean)
Automatically adjusts VAT when payment discounts are applied.

```al
InitValue = true;   // Standard: aktiviert
```

**Beispiel 1 — Automatische Anpassung:**
> `Adjust for Payment Disc. = True`
> → Bei Skontoabzug wird die MwSt automatisch korrigiert. Rechnung 1.190 €, Skonto 2% = 23,80 € → MwSt wird anteilig korrigiert (Debitorenkonto 23,80, MwSt-Konto ~3,80). **Bedingung:** `Pmt. Disc. Excl. VAT = False`, `VAT Tolerance % = 0`.

**Beispiel 2 — Keine automatische Anpassung:**
> `Adjust for Payment Disc. = False`
> → MwSt bleibt unverändert. Skonto wird nur vom Brutto abgezogen, MwSt-Differenz bleibt als offene MwSt stehen (später manuelle Korrektur nötig).

**Beispiel 3 — Blockiert durch VAT Posting Setup:**
> `Adjust for Payment Disc. = False` während ein `VAT Posting Setup`-Eintrag `Adjust for Payment Discount = True` hat.
> → Error: _"VAT Posting Setup [Name] use Adjust for Payment Discount."_ Erst VAT Posting Setup deaktivieren, dann `Adjust for Payment Disc.` ändern.

---

## 4.5 Rundung & Nachkommastellen

### Feld 58/59: `Inv. Rounding Precision (LCY)` / `Inv. Rounding Type (LCY)` (Decimal/Option)
Precision and method for invoice rounding in local currency.

**Beispiel 1 — Kaufmännisches Runden (Nearest):**
> `Inv. Rounding Precision (LCY) = 0.01`, `Inv. Rounding Type (LCY) = Nearest`
> → Standard. Rechnungssumme 123,455 → 123,46. Nichts Besonderes.

**Beispiel 2 — Schweizer 5-Rappen-Rundung:**
> `Inv. Rounding Precision (LCY) = 0.05`, `Inv. Rounding Type (LCY) = Up`
> → Rechnungssumme 123,42 → 123,45. Immer AUF die nächsten 5 Rappen. Typisch für CH-Franken.

**Beispiel 3 — Betrag ändern trotz Buchungen:**
> `Amount Rounding Precision` ändern, aber es existieren bereits G/L Entry-Datensätze
> → `CheckRoundingError()` prüft ALLE Ledger-Entry-Tabellen (G/L, Item, Job, Resource, FA, Maintenance, Insurance). Wenn einer nicht leer ist → Error: _"You cannot change the contents of the Amount Rounding Precision field because there are posted ledger entries."_

**Relevanter Code:**
```al
procedure CheckRoundingError(NameOfField: Text[100])
begin
    ErrorMessage := false;
    if GLEntry.FindFirst() then ErrorMessage := true;
    if ItemLedgerEntry.FindFirst() then ErrorMessage := true;
    // ... alle weiteren Ledger-Entry-Tabellen
    OnBeforeCheckRoundingError(ErrorMessage);   // Integration Event
    if ErrorMessage then Error(Text018, NameOfField);
end;
```

---

### Feld 73: `Amount Rounding Precision` (Decimal)
Precision for monetary amounts (InitValue = 0.01).

**Beispiel 1 — Standard:**
> `Amount Rounding Precision = 0.01`
> → Alle Beträge auf 2 Nachkommastellen. Rechnungsposition 10,00 € × 1,19 = 11,90 € passt.

**Beispiel 2 — Keine Nachkommastellen (JPY, KRW):**
> `Amount Rounding Precision = 1`
> → Beträge werden auf ganze Einheiten gerundet. 1190 Yen, keine Dezimalstellen. Nach Änderung: _"You must close the program and start again..."_

**Beispiel 3 — 3 Dezimalstellen:**
> `Amount Rounding Precision = 0.001`
> → Für Öl-/Gasindustrie mit Preis je 1000 Liter. 1,199 € pro Liter × 1000. Änderung erfordert Client-Neustart (Message `Text021`).

---

## 4.6 Zusätzliche Berichtswährung (Additional Reporting Currency)

### Feld 68: `Additional Reporting Currency` (Code[10])
Currency code for parallel accounting.

```al
trigger OnValidate()
begin
    if ("Additional Reporting Currency" <> xRec."Additional Reporting Currency") and
       ("Additional Reporting Currency" <> '')
    then begin
        AdjAddReportingCurr.SetAddCurr("Additional Reporting Currency");
        AdjAddReportingCurr.RunModal();   // Report "Adjust Add. Reporting Currency"
        if not AdjAddReportingCurr.IsExecuted() then
            "Additional Reporting Currency" := xRec."Additional Reporting Currency";
    end;
    if (...executed...) then
        DeleteAnalysisView();  // Löscht alle Analyseansichten!
end;
```

**Beispiel 1 — Konzernwährung USD:**
> LCY = EUR, `Additional Reporting Currency = USD`
> → Jede Buchung wird parallel in USD gespeichert. Report "Adjust Add. Reporting Currency" initialisiert Wechselkurse. Alle Analyseansichten werden via `DeleteAnalysisView()` gelöscht und müssen neu aufgebaut werden.

**Beispiel 2 — Keine Zusatzwährung:**
> `Additional Reporting Currency = ''` (leer)
> → Nur in LCY buchen. Keine parallele Währung. Analyseansichten bleiben erhalten.

**Beispiel 3 — Währungsumstellung:**
> `Additional Reporting Currency` von USD auf CHF ändern
> → `AdjAddReportingCurr.RunModal()` öffnet den Wechselkurs-Anpassungs-Dialog. Wenn der Benutzer abbricht (`IsExecuted() = false`), wird auf die alte Währung zurückgesetzt. Wenn ausgeführt, werden Analyseansichten gelöscht (`DeleteAnalysisView()`).

---

## 4.7 Job Queue & Hintergrundbuchung

### Feld 50: `Post with Job Queue` (Boolean)
Enables background posting using job queue.

```al
trigger OnValidate()
begin
    if not "Post with Job Queue" then
        "Post & Print with Job Queue" := false;
end;
```

**Beispiel 1 — Vordergrund-Buchung:**
> `Post with Job Queue = False`
> → Buchung erfolgt sofort, der Benutzer wartet auf Abschluss. GUI blockiert während der Buchung.

**Beispiel 2 — Hintergrund-Buchung:**
> `Post with Job Queue = True`, `Job Queue Category Code = "POSTING"`, `Job Queue Priority for Post = 1000`
> → Buchung wird als Job-Queue-Task eingereiht. Der Benutzer kann sofort weiterarbeiten. `Notify On Success = True` zeigt eine Benachrichtigung bei Abschluss.

**Beispiel 3 — Post & Print:**
> `Post & Print with Job Queue = True` (erzwingt `Post with Job Queue = True`)
> → Nach erfolgreicher Hintergrund-Buchung wird automatisch der Beleg gedruckt/als PDF erzeugt. `Job Q. Prio. for Post & Print = 1000`.

**Relevanter Code:**
- `JobQueueActive(): Boolean` — prüft, ob Job Queue aktiv
- **Codeunit "Job Queue"** — verwaltet die Einträge

---

## 4.8 Sonstige wichtige Felder

### Feld 164: `Show Amounts` (Option)
Controls amount display in G/L entries.

```al
OptionMembers = "Amount Only","Debit/Credit Only","All Amounts";
```

**Beispiel 1 — Nur Beträge:**
> `Show Amounts = Amount Only`
> → In G/L Entry-Seiten wird nur `Amount` angezeigt. Debit/Credit separat ausgeblendet. Ideal für einfache Buchhaltung.

**Beispiel 2 — Nur Soll/Haben:**
> `Show Amounts = Debit/Credit Only`
> → Zeigt `Debit Amount` und `Credit Amount`. Setzen viele Code-Stellen voraus, z.B. `AppliedVendorEntries.Page.al`:
> ```al
> AmountVisible := not (GLSetup."Show Amounts" = GLSetup."Show Amounts"::"Debit/Credit Only");
> ```

**Beispiel 3 — Alle Spalten:**
> `Show Amounts = All Amounts`
> → Sowohl `Amount`, `Debit Amount` als auch `Credit Amount` sichtbar. Maximale Transparenz, mehr Platzbedarf.

---

### Feld 65: `Summarize G/L Entries` (Boolean)
Combines G/L entries with identical account, posting date, and dimensions.

**Beispiel 1 — Zusammenfassen:**
> `Summarize G/L Entries = True`
> → Mehrmals dasselbe Konto am gleichen Tag mit gleichen Dimensionen → nur EIN G/L Entry. Weniger Einträge, bessere Performance, aber weniger Detail.

**Beispiel 2 — Detail:**
> `Summarize G/L Entries = False`
> → Jede Buchung = ein G/L Entry. Maximale Detailtiefe, mehr Speicherplatz.

**Beispiel 3 — Performance-kritisch:**
> Bei Massenbuchungen (z.B. 10.000 Rechnungen an denselben Debitor) reduziert `Summarize = True` die G/L Entries massiv. Sichtbar in `GLEntryPostingPreview.Table.al` während der Vorschau.

---

### Feld 56: `Mark Cr. Memos as Corrections` (Boolean)
Marks credit memos as corrections for VAT and financial reporting.

**Beispiel 1 — Korrektur-Markierung:**
> `Mark Cr. Memos as Corrections = True`
> → Gutschriften werden als Korrektur markiert (Feld `Correction` auf G/L Entry = True). Statistiken/MwSt-Meldungen werten diese separat aus.

**Beispiel 2 — Keine Markierung:**
> `Mark Cr. Memos as Corrections = False`
> → Gutschriften wie normale Buchungen. Können in MwSt-Meldungen fälschlich als positiver Umsatz erscheinen.

**Beispiel 3 — IDEP/MwSt-Meldung Deutschland:**
> Korrektur-Gutschriften müssen in der UStVA-Zeile 56 separat ausgewiesen werden. Ohne `Mark Cr. Memos as Corrections` landen sie in der falschen Zeile.

---

## 4.9 Integration Events (für Entwickler)

Table 98 stellt folgende `[IntegrationEvent]`-Prozeduren bereit:

| Event | Parameter | Verwendung |
|---|---|---|
| `OnBeforeCheckRoundingError` | `var ErrorMessage: Boolean` | Zusätzliche Prüfungen vor Rundungsänderung |
| `OnAfterIsPostingAllowed` | `GLSetup, PostingDate, var Result` | Custom Posting-Date-Validierung |
| `OnBeforeFirstAllowedPostingDate` | `GLSetup, var AllowedPostingDate, var IsHandled` | Eigenen Algorithmus für ersten Buchungstag |
| `OnAfterUpdateDimValueGlobalDimNo` | `ShortcutDimNo, OldDimCode, NewDimCode` | Custom Logic bei Dim-Änderung |

**Subscriber-Beispiel (AL):**
```al
codeunit 50100 "My Posting Validation"
{
    [EventSubscriber(ObjectType::Table, Database::"General Ledger Setup",
        OnAfterIsPostingAllowed, '', false, false)]
    local procedure CheckCustomPostingPeriod(
        GeneralLedgerSetup: Record "General Ledger Setup";
        PostingDate: Date;
        var Result: Boolean)
    begin
        if PostingDate < CustomGetFiscalYearStart() then
            Result := false;
    end;
}
```

---

## 4.10 Abhängige Tabellen & Codeunits

| Tabelle/Codeunit | ID | Verwendung |
|---|---|---|
| **General Ledger Setup** | 98 | Singleton-Konfiguration |
| **G/L Account** | 15 | Sachkonten |
| **G/L Entry** | 17 | Gebuchte Sachposten |
| **Gen. Journal Line** | 81 | Fibu-Buchungszeilen |
| **VAT Posting Setup** | 325 | MwSt-Buchungsmatrix |
| **Currency** | 4 | Währungen |
| **Dimension** | 348 | Dimensionen |
| **Dimension Value** | 349 | Dimensionswerte |
| **User Setup Management** | Codeunit | Posting-Date-Checks, Date-Formula-Berechnung |
| **Dimension Management** | Codeunit | DimensionSetID-Handling, Popup |
| **Feature Telemetry** | Codeunit | Feature-Usage-Tracking |

---

| [← Zurück zur Übersicht]({{ '/index' | relative_url }}) | [Weiter: Vertrieb & Marketing →]({{ '/05-sales-marketing/' | relative_url }}) |
