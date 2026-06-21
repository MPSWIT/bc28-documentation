---
title: "Finanzwesen"
---
# 4. Finanzwesen — Fibu-Einrichtung

> **Tabelle 98:** `GeneralLedgerSetup.Table.al`
> **Namensraum:** `Microsoft.Finance.GeneralLedger.Setup`
> **Seite:** `GeneralLedgerSetup.Page.al`
> **Typ:** Singleton (genau ein Datensatz, Primärschlüssel = `Primärschlüssel`)

Die Tabelle **Fibu-Einrichtung (98)** ist die zentrale Schaltstelle der gesamten Finanzbuchhaltung.
Sie steuert Buchungszeiträume, MwSt-Verhalten, Rundung, Dimensionen und Zahlungstoleranzen.

---

## 4.1 Buchungszeiträume

### Feld 2: `Allow Posting From` — Buchungen zugel. ab

Legt das früheste Datum fest, ab dem in die Finanzbuchhaltung gebucht werden darf.
Offizielle BC-Überschrift: **Buchungen zugel. ab**.

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

**Beispiel 1 — Das Unternehmen startet mit BC28 zum 01.01.2026:**
> Die Buchhaltung möchte alle Buchungen ab dem Geschäftsjahresbeginn am 01.01.2026 zulassen. Ein Enddatum wird nicht gesetzt, da das Geschäftsjahr noch läuft.
> ➜ `Allow Posting From = 01.01.2026`, `Allow Posting To = leer`
> **Ergebnis:** Jede Buchung mit Datum ≥ 01.01.2026 wird angenommen. Versehentliche Buchungen aus dem Vorjahr sind blockiert.

**Beispiel 2 — Der Monatsabschluss Juni 2026 läuft:**
> Die Finanzabteilung hat den Juni abgeschlossen und möchte nun die Juli-Buchungen kontrolliert freigeben. Nur Buchungen im Juni sollen noch möglich sein, um Abschlusskorrekturen zu erlauben.
> ➜ `Allow Posting From = 01.06.2026`, `Allow Posting To = 30.06.2026`
> **Ergebnis:** Versucht ein Sachbearbeiter am 05.07.2026 zu buchen, erscheint ein Fehler über `CheckAllowedPostingDates()`. Die Juni-Periode bleibt für Korrekturen offen, der Juli ist gesperrt.

**Beispiel 3 — Ein rollierender Buchungszeitraum mit dynamischer Formel:**
> Das Unternehmen möchte, dass immer der aktuelle und der vorherige Monat offen sind, ohne jedes Mal manuell Daten ändern zu müssen. Die Buchhaltung konfiguriert dynamische Datumsformeln.
> ➜ `Allow Posting From DateFormula = "-1M"`, `Allow Posting To DateFormula = "CM"`
> **Ergebnis:** Am 21.06.2026 sind Buchungen vom 21.05.2026 bis 30.06.2026 erlaubt. Am 01.07.2026 verschiebt sich das früheste Datum automatisch auf 01.06.2026 und das Enddatum auf 31.07.2026 — ohne dass jemand die Einrichtung anfassen muss.

**Wichtige Codestellen:**
- `CheckDateRange()` (lokal) — kombiniert feste Daten mit Datumsformeln
- `FirstAllowedPostingDate()` — berücksichtigt zusätzlich `Inventory Period` (keine Buchung in gesperrte Lagerperiode)
- `IsPostingAllowed(PostingDate: Date): Boolean` — Integrationsereignis `OnAfterIsPostingAllowed`
- `CheckAllowedPostingDates()` delegiert an **Codeunit "User Setup Management"**

---

### Feld 4: `Register Time` — Protokollzeit

Legt fest, ob zusätzlich zum Buchungsdatum auch die Uhrzeit der Buchung gespeichert wird.
Offizielle BC-Überschrift: **Protokollzeit**.

**Beispiel 1 — Ein Wirtschaftsprüfer verlangt minutengenaue Nachvollziehbarkeit:**
> Das Unternehmen wird testiert und der Prüfer möchte sehen, in welcher Reihenfolge die Buchungen am Abschlusstag vorgenommen wurden. Wer hat wann die letzte Abschlussbuchung erstellt?
> ➜ `Register Time = Ja`
> **Ergebnis:** Jede gebuchte Transaktion erhält zusätzlich zum Buchungsdatum einen Zeitstempel. Der Prüfer kann die chronologische Reihenfolge aller Buchungen am 31.12.2026 exakt nachvollziehen.

**Beispiel 2 — Ein Unternehmen mit geringem Transaktionsvolumen:**
> Eine kleine Agentur mit 50 Buchungen pro Monat benötigt keine minutiöse Zeiterfassung. Das Buchungsdatum reicht für die Nachvollziehbarkeit völlig aus.
> ➜ `Register Time = Nein`
> **Ergebnis:** Nur das Datum wird gespeichert. Die Ledger-Entry-Tabellen bleiben kleiner.

**Beispiel 3 — Hintergrundbuchungen mit Job Queue:**
> Das Unternehmen verarbeitet monatlich 5.000 automatische Zinsbuchungen über die Aufgabenwarteschlange. Die Finanzabteilung möchte dennoch wissen, wann genau welche Zinsbuchung verarbeitet wurde, um bei Rückfragen den exakten Verarbeitungszeitpunkt belegen zu können.
> ➜ `Post with Job Queue = Ja` + `Register Time = Ja`
> **Ergebnis:** Trotz Hintergrundverarbeitung erhält jede der 5.000 Buchungen einen individuellen Zeitstempel — vollständige Nachvollziehbarkeit auch bei Massenbuchungen.

---

### Feld 169: `Posting Preview Type` — Buchungsvorschautyp

Bestimmt, welche Vorschau dem Benutzer vor der endgültigen Buchung angezeigt wird.
Offizielle BC-Überschrift: **Buchungsvorschautyp**.

**Beispiel 1 — Ein neuer Buchhalter soll Buchungen vor dem Buchen prüfen:**
> Das Unternehmen hat einen neuen Junior-Buchhalter eingestellt. Der Finanzleiter möchte, dass vor jeder Buchung eine detaillierte Vorschau der entstehenden Sachposten, Debitoren/Kreditoren-Posten und MwSt-Posten angezeigt wird.
> ➜ `Posting Preview Type = Detailliert`
> **Ergebnis:** Bei jeder Buchung erscheint die Vorschautabelle **G/L Entry Posting Preview** mit allen betroffenen Konten. Der Junior kann prüfen und dann final buchen.

**Beispiel 2 — Erfahrene Sachbearbeiter buchen täglich hunderte Rechnungen:**
> In der Kreditorenbuchhaltung werden täglich 300 Eingangsrechnungen erfasst und gebucht. Eine Vorschau würde den Arbeitsfluss massiv verlangsamen. Die Sachbearbeiter kennen ihre Buchungsmatrix auswendig.
> ➜ `Posting Preview Type = Keine`
> **Ergebnis:** Keine Vorschau — Buchungen werden sofort ausgeführt. Deutlich schneller bei Routinevorgängen, aber ohne Kontrollmöglichkeit vor dem Buchen.

**Beispiel 3 — Die Vorschau dient als Vorab-Prüfung für den Vertrieb:**
> Ein Vertriebsmitarbeiter erstellt eine komplexe Ausgangsrechnung mit Teillieferungen, Anzahlungen und Skonto-Staffeln. Vor der finalen Buchung möchte er in der Vorschau sehen, ob die MwSt korrekt aufgeteilt wird und ob alle Debitorenkonten stimmen.
> ➜ `Posting Preview Type = Detailliert` — zusätzlich über das Ereignis `OnBeforePost` erweitert
> **Ergebnis:** Die Vorschau-Tabelle `VATEntryPostingPreview.Table.al` zeigt die MwSt-Aufteilung. Ein Fehler in der Buchungsmatrix fällt sofort auf, bevor echte Posten entstehen.

**Relevante Objekte:**
- `GLEntryPostingPreview.Table.al` — Sachposten-Vorschau
- `VATEntryPostingPreview.Table.al` — MwSt-Posting-Vorschau

---

## 4.2 Dimensionen

### Felder 79/80: `Global Dimension 1 Code` / `Global Dimension 2 Code` — Globaler Dimensionscode 1/2

Die beiden globalen Dimensionen, die im gesamten System für Analyse und Berichtswesen verwendet werden.
Offizielle BC-Überschrift: **Globaler Dimensionscode 1** / **Globaler Dimensionscode 2**.

```al
field(79; "Global Dimension 1 Code"; Code[20])
{
    Editable = false;       // Nicht direkt bearbeitbar!
    TableRelation = Dimension;
    trigger OnValidate()
    begin
        "Shortcut Dimension 1 Code" := "Global Dimension 1 Code";
    end;
}
```

Die globalen Dimensionen werden **nicht direkt** in dieser Tabelle gesetzt, sondern über die Dimension-Tabelle und `UpdateDimValueGlobalDimNo()`.

**Beispiel 1 — Ein Filialunternehmen analysiert nach Abteilung und Kostenstelle:**
> Das Unternehmen betreibt 5 Filialen und möchte alle Buchungen nach Abteilung (welche Filiale) und Kostenstelle (welcher Bereich: Verkauf, Lager, Verwaltung) auswerten können. Auf jedem Sachkonto werden Standardwerte hinterlegt, damit bei Buchungen nicht jedes Mal manuell erfasst werden muss.
> ➜ `Global Dimension 1 Code = "ABTEILUNG"`, `Global Dimension 2 Code = "KOSTENSTELLE"`
> **Ergebnis:** Jede Buchung MUSS eine Abteilung und eine Kostenstelle enthalten (wenn auf der Dimension `Code Mandatory = Ja`). Die FlowFields `Cust. Balances Due` und `Vendor Balances Due` auf der Fibu-Einrichtung liefern sofort die Offene-Posten-Liste gefiltert nach Filiale.

**Beispiel 2 — Ein Projektunternehmen gliedert nach Projekt und Region:**
> Eine Baufirma mit 30 laufenden Projekten in 3 Bundesländern möchte Deckungsbeiträge pro Projekt und Region sehen. Der Geschäftsführer ruft einmal pro Woche die Analyseansicht auf und filtert nach Projekt/Region.
> ➜ `Global Dimension 1 Code = "PROJEKT"`, `Global Dimension 2 Code = "REGION"`
> **Ergebnis:** Alle Berichte und Analyseansichten arbeiten standardmäßig mit diesen beiden Filtern. Die FlowFields `Cust. Balances Due` und `Vendor Balances Due` zeigen sofort: Welcher Kunde schuldet noch Geld — pro Projekt und Region getrennt.

**Beispiel 3 — Ein Einzelunternehmen braucht nur eine Dimension:**
> Eine Steuerberatungskanzlei mit 10 Mitarbeitern möchte nur nach Kostenstelle auswerten, braucht aber keine zweite globale Dimension. Die Shortcut-Dimensionen 3–8 werden optional für Kampagnen und Mandanten genutzt.
> ➜ `Global Dimension 1 Code = "KOSTENSTELLE"`, `Global Dimension 2 Code = ""`
> **Ergebnis:** Nur Kostenstellen sind global pflicht. Weitere Dimensionen (Shortcut 3–8) können bei Bedarf optional verwendet werden.

**FlowFields, die globale Dimensionen nutzen:**
```al
field(44; "Cust. Balances Due"; Decimal)
{
    FieldClass = FlowField;
    CalcFormula = sum("Detailed Cust. Ledg. Entry"."Amount (LCY)"
        where("Initial Entry Global Dim. 1" = field("Global Dimension 1 Filter"),
              "Initial Entry Global Dim. 2" = field("Global Dimension 2 Filter")));
}
```

> ⚠️ **Wichtig für Entwickler:** `Editable = false` — diese Felder werden nur über `UpdateDimValueGlobalDimNo()` (bei Dimensionsänderung) gesetzt. Direkter Zugriff über AL wird ignoriert.

---

### Felder 81–88: `Shortcut Dimension 1..8 Code` — Shortcutdimensionscode 1..8

Schnellzugriffs-Dimensionen, die auf Buchungsseiten direkt sichtbar sind.
Offizielle BC-Überschrift: **Shortcutdimensionscode 1** bis **Shortcutdimensionscode 8**.

**Beispiel 1 — Ein Vertriebsteam benötigt Vertreter und Kampagne:**
> Das Unternehmen hat 8 Außendienstmitarbeiter und führt regelmäßig Marketing-Kampagnen durch. Auf jeder Verkaufsbelegseite sollen Vertreter und Kampagne als Pflichtfelder erscheinen, damit die Provision und der Kampagnenerfolg auswertbar sind.
> ➜ Shortcut 1 = "ABTEILUNG" (aus Global Dim 1), Shortcut 2 = "KOSTENSTELLE" (aus Global Dim 2), Shortcut 3 = "VERTRETER", Shortcut 4 = "KAMPAGNE"
> **Ergebnis:** Auf jeder Buchungsseite erscheinen 4 Dimensionen oben im Dim-Bereich. Der Vertrieb kann direkt Vertreter und Kampagne zuordnen — keine separate Dimensionserfassung nötig.

**Beispiel 2 — Ein schlankes Setup für eine kleine Firma:**
> Ein Handwerksbetrieb mit 5 Mitarbeitern möchte nur Abteilung (Global Dim 1) und Kostenstelle (Global Dim 2) sehen. Weitere Dimensionen sind nicht nötig und würden nur verwirren.
> ➜ Shortcut 3–8 = leer
> **Ergebnis:** Auf Belegseiten sind nur 2 Dimensionen sichtbar. Übersichtlich für kleine Teams.

**Beispiel 3 — Umbenennung einer bestehenden Schnelldimension:**
> Das Unternehmen organisiert sich um: Statt "VERTRETER" soll nun "TEAM" verwendet werden. Der Administrator ändert Shortcut 3 von "VERTRETER" auf "TEAM".
> ➜ Shortcut 3 von "VERTRETER" auf "TEAM" ändern
> **Ergebnis:** `UpdateDimValueGlobalDimNo()` setzt für alle Dimensionswerte des neuen Codes die `Global Dimension No.` auf 3, für die alten Werte auf 0. Die `Dimension Set Entry`-Tabelle wird über `UpdateGlobalDimensionNo()` synchronisiert. Alle historischen Buchungen behalten ihre ursprünglichen Dimensionswerte — der neue Code gilt nur für zukünftige Buchungen.

**Relevante Codestellen:**
- `UpdateDimValueGlobalDimNo(xDimCode, DimCode, ShortcutDimNo)` — lokale Prozedur in Tabelle 98
- `OnAfterUpdateDimValueGlobalDimNo()` — Integrationsereignis für Erweiterungen
- **Codeunit "Dimension Management"** — `ShowDimensionSet()` für Dimensions-Popup

---

## 4.3 Mehrwertsteuer

### Feld 7: `VAT Reporting Date` — MwSt.-Standarddatum

Legt fest, nach welchem Datum die MwSt standardmäßig gemeldet wird.
Offizielle BC-Überschrift: **MwSt.-Standarddatum**.

**Beispiel 1 — Ein Unternehmen mit IST-Versteuerung:**
> Das Unternehmen versteuert nach vereinnahmten Entgelten (IST). Die MwSt soll immer dann gemeldet werden, wenn die Buchung tatsächlich erfolgt — unabhängig vom Rechnungsdatum.
> ➜ `VAT Reporting Date = Buchungsdatum`
> **Ergebnis:** Eine Rechnung vom 15.06., die erst am 02.07. gebucht wird, erscheint in der MwSt-Meldung Juli. Die MwSt folgt dem tatsächlichen Buchungsfluss.

**Beispiel 2 — Ein Unternehmen mit SOLL-Versteuerung:**
> Das Unternehmen versteuert nach vereinbarten Entgelten (SOLL). Die MwSt soll im Monat des Rechnungsdatums erscheinen, auch wenn die Buchung später erfolgt.
> ➜ `VAT Reporting Date = Belegdatum`
> **Ergebnis:** Eine Rechnung vom 15.06., die erst am 02.07. gebucht wird, erscheint in der MwSt-Meldung Juni — korrekt nach dem Rechnungsdatum.

**Beispiel 3 — VAT Date Usage schaltet die Vorgabe ab:**
> Das Unternehmen hat zwar `VAT Reporting Date = Belegdatum` gesetzt, aber der Steuerberater empfiehlt, für einen Übergangszeitraum temporär auf Buchungsdatum umzustellen.
> ➜ `VAT Reporting Date Usage = Deaktiviert`
> **Ergebnis:** Trotz der Einstellung `Belegdatum` fällt das System auf `Buchungsdatum` zurück. Gesteuert über das Enum `VAT Reporting Date Usage` (Feld 8).

**Wichtige Prozeduren:**
- `GetVATDate(PostingDate, DocumentDate): Date` — liefert das effektive MwSt-Datum
- `UpdateVATDate(NewDate, VATDateType, var VATDate)` — setzt MwSt-Datum wenn Typ übereinstimmt

---

### Feld 48: `Unrealized VAT` — Unrealisierte MwSt.

Aktiviert die Ist-Versteuerung mit nicht realisierter MwSt.
Offizielle BC-Überschrift: **Unrealisierte MwSt.**

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

**Beispiel 1 — Ein Unternehmen stellt auf IST-Versteuerung um:**
> Das Unternehmen wechselt von SOLL auf IST-Versteuerung. MwSt soll erst bei Zahlungseingang fällig werden, nicht schon bei Rechnungsstellung. Der Steuerberater hat die entsprechenden `VAT Posting Setup`-Einträge mit `Unrealized VAT Type = Percentage` angelegt.
> ➜ `Unrealized VAT = Ja`
> **Ergebnis:** Bei Buchung einer Ausgangsrechnung über 11.900 € (netto 10.000 + 1.900 MwSt) wird die MwSt als nicht realisiert markiert. Erst wenn der Kunde zahlt und der Zahlungsausgleich gebucht wird, wird die MwSt realisiert. Das System setzt automatisch `Prepayment Unrealized VAT = Ja`.

**Beispiel 2 — Das Unternehmen behält die SOLL-Versteuerung bei:**
> Ein Dienstleister mit wenigen, zuverlässigen Kunden bleibt bei der SOLL-Versteuerung. Keine `VAT Posting Setup`-Einträge mit `Unrealized VAT Type = Percentage` existieren.
> ➜ `Unrealized VAT = Nein`
> **Ergebnis:** MwSt wird sofort bei Rechnungsstellung fällig. Die Prüfung im `OnValidate` bestätigt, dass keine unrealisierten MwSt-Einrichtungen existieren.

**Beispiel 3 — Deaktivierung scheitert an bestehenden Einrichtungen:**
> Das Unternehmen möchte `Unrealized VAT` deaktivieren, aber es existiert noch ein `VAT Posting Setup`-Eintrag für `VAT Bus. Posting Group = INLAND` / `VAT Prod. Posting Group = NORMAL` mit `Unrealized VAT Type = Percentage`.
> ➜ Versuch: `Unrealized VAT = Nein`
> **Ergebnis:** Fehler _„VAT Posting Setup INLAND NORMAL have Unrealized VAT Type to Percentage."_ — Die Deaktivierung wird blockiert. Lösung: Erst den VAT Posting Setup-Eintrag bereinigen, dann `Unrealized VAT` deaktivieren.

---

### Feld 103: `Bill-to/Sell-to VAT Calc.` — Rech. an/Verk. an MwSt.-Berech.

Legt fest, ob die MwSt anhand der Rechnungsadresse (Rech. an) oder der Lieferadresse (Verk. an) ermittelt wird.
Offizielle BC-Überschrift: **Rech. an/Verk. an MwSt.-Berech.**

**Beispiel 1 — Standard: MwSt nach Rechnungsadresse:**
> Ein deutsches Unternehmen verkauft an einen Kunden mit Rechnungsadresse in Deutschland. Die Ware wird an die deutsche Niederlassung geliefert — klarer Fall.
> ➜ `Bill-to/Sell-to VAT Calc. = Rechnungsadresse`
> **Ergebnis:** Deutsche MwSt (19%). Sauber und einfach.

**Beispiel 2 — Ein Unternehmen liefert EU-weit:**
> Das Unternehmen verkauft an einen französischen Kunden (Rechnungsadresse Paris). Die Ware geht an das Warenlager in Lyon. Der Steuerberater rät, die MwSt nach Lieferadresse zu berechnen (innergemeinschaftliche Lieferung).
> ➜ `Bill-to/Sell-to VAT Calc. = Lieferadresse`
> **Ergebnis:** Französische MwSt-Regeln kommen zur Anwendung, das System prüft die französische USt-ID.

**Beispiel 3 — Cross-Border mit Schweiz:**
> Ein Schweizer Kunde (Rechnungsadresse Zürich) lässt an eine deutsche Tochtergesellschaft liefern (Lieferadresse München). Der Steuerberater muss entscheiden, welche Regel gilt.
> ➜ `Bill-to/Sell-to VAT Calc. = Rechnungsadresse` → Schweizer MwSt (keine EU-MwSt, Ausfuhr)
> ➜ `Bill-to/Sell-to VAT Calc. = Lieferadresse` → Deutsche MwSt (innergemeinschaftliche Lieferung)
> **Ergebnis:** Falsche Einstellung führt zu massiven MwSt-Fehlern in der UStVA! In diesem Fall muss der Steuerberater die korrekte Behandlung vorgeben.

---

### Feld 188: `Control VAT Period` — MwSt.-Zeitraum kontrollieren

Steuert die Validierung der MwSt-Zeiträume bei Buchungen.
Offizielle BC-Überschrift: **MwSt.-Zeitraum kontrollieren**.

**Beispiel 1 — Das Unternehmen möchte flexibel bleiben:**
> Die Buchhaltung erfasst manchmal Rechnungen verspätet und möchte keine starren Sperren. Der MwSt-Zeitraum wird im Nachhinein über die MwSt-Meldung korrigiert.
> ➜ `Control VAT Period = Deaktiviert`
> **Ergebnis:** Keine Validierung. Buchungen sind unabhängig vom MwSt-Zeitraum möglich.

**Beispiel 2 — Sanfte Warnung statt harter Blockade:**
> Das Unternehmen hat die Juli-MwSt-Meldung bereits abgegeben, aber es kommen noch vereinzelt Juli-Rechnungen rein. Die Buchhaltung möchte gewarnt werden, aber nicht blockiert.
> ➜ `Control VAT Period = Warnung`
> **Ergebnis:** Bei Buchung außerhalb des offenen MwSt-Zeitraums erscheint ein Warnhinweis. Die Buchung ist dennoch möglich. Ideal für Übergangsphasen.

**Beispiel 3 — Harte Sperre nach MwSt-Meldung:**
> Das Unternehmen wurde bei der letzten Betriebsprüfung gerügt, weil Buchungen in bereits gemeldete MwSt-Zeiträume vorgenommen wurden. Der Finanzleiter verfügt: Nach Abgabe der UStVA sind keine Buchungen mehr im gemeldeten Zeitraum erlaubt.
> ➜ `Control VAT Period = Fehler`
> **Ergebnis:** Jede Buchung außerhalb des offenen MwSt-Zeitraums wird mit einem Fehler abgelehnt. Der Wechsel dieser Einstellung wird über `FeatureTelemetry.LogUsage()` protokolliert.

---

### Feld 72: `VAT Exchange Rate Adjustment` — MwSt.-Kursregulierung

Legt fest, wie MwSt-Beträge bei Wechselkursanpassungen behandelt werden.
Offizielle BC-Überschrift: **MwSt.-Kursregulierung**.

**Beispiel 1 — Das Unternehmen bucht nur in Mandantenwährung:**
> Ein rein national tätiges Unternehmen hat keine Fremdwährungsbuchungen und benötigt keine MwSt-Kursanpassung.
> ➜ `VAT Exchange Rate Adjustment = Keine Anpassung`
> **Ergebnis:** Wechselkursdifferenzen werden nur für Sachkonten berechnet, MwSt bleibt unverändert.

**Beispiel 2 — Ein Importeur mit vielen Fremdwährungsrechnungen:**
> Das Unternehmen importiert Waren aus den USA und der Schweiz. Jede einzelne MwSt-Position soll mit dem aktuellen Kurs neu bewertet werden, um exakte MwSt-Differenzen auszuweisen.
> ➜ `VAT Exchange Rate Adjustment = Pro Eintrag`
> **Ergebnis:** Jeder MwSt-Posten wird einzeln mit dem neuen Wechselkurs bewertet. Detailliert, aber rechenintensiv.

**Beispiel 3 — Sammelbuchung der MwSt-Differenzen:**
> Das Unternehmen möchte MwSt-Differenzen nicht pro Eintrag, sondern gesammelt auf einem separaten MwSt-Differenzkonto buchen. Übersichtlicher für die UStVA.
> ➜ Verwendung des `VAT Posting Setup` für die Kontenfindung der Sammelbuchung.

---

## 4.4 Zahlungstoleranzen & Skonto

### Felder 94/95: `Payment Tolerance %` / `Max. Payment Tolerance Amount` — Zahlungstoleranz % / Max. Zahlungstoleranzbetrag

Maximale prozentuale und absolute Toleranz für den automatischen Zahlungsausgleich.
Offizielle BC-Überschriften: **Zahlungstoleranz %** / **Max. Zahlungstoleranzbetrag**.

> ⚠️ Diese Felder sind **nicht editierbar** (`Editable = false`). Sie werden vom System berechnet.

**Beispiel 1 — Kunde zahlt etwas zu wenig, Differenz soll automatisch ausgeglichen werden:**
> Ein Kunde überweist 980,00 € auf eine Rechnung über 1.000,00 € (Differenz 2%). Das Unternehmen möchte solche kleinen Differenzen automatisch ausgleichen lassen.
> ➜ `Payment Tolerance % = 2`, `Max. Payment Tolerance Amount = 50,00`
> **Ergebnis:** Die Differenz von 20 € liegt unter 2% und unter 50 € — automatischer Ausgleich. Der Kunde bekommt keinen Mahnbrief über 20 €, die Buchhaltung spart manuellen Aufwand.

**Beispiel 2 — Hoher Rechnungsbetrag, kleine Toleranz:**
> Ein Kunde zahlt 98.500 € auf eine Rechnung über 100.000 € (Differenz 1.500 € = 1,5%). Die Buchhaltung hat enge Toleranzen gesetzt.
> ➜ `Payment Tolerance % = 1`, `Max. Payment Tolerance Amount = 500,00`
> **Ergebnis:** Prozent überschritten (1,5 > 1,0) UND Betrag überschritten (1.500 > 500) — KEIN automatischer Ausgleich. Der Kunde muss die restlichen 1.500 € zahlen.

**Beispiel 3 — Keine Toleranz — jede Differenz bleibt offen:**
> Ein kleines Unternehmen mit geringem Zahlungsverkehr möchte jede Differenz manuell prüfen und klären. Automatischer Ausgleich ist unerwünscht.
> ➜ `Payment Tolerance % = 0`, `Max. Payment Tolerance Amount = 0`
> **Ergebnis:** `GetPmtToleranceVisible()` liefert `false` — die Toleranzfelder sind ausgeblendet. Jede Differenz bleibt als offener Posten stehen.

---

### Feld 99/92: `Payment Tolerance Posting` / `Pmt. Disc. Tolerance Posting` — Zahlungstoleranzbuchung / Skontotoleranzbuchung

Legt fest, auf welche Konten Toleranzbeträge gebucht werden.
Offizielle BC-Überschriften: **Zahlungstoleranzbuchung** / **Skontotoleranzbuchung**.

```al
OptionMembers = "Payment Tolerance Accounts","Payment Discount Accounts";
```

**Beispiel 1 — Toleranz sauber vom Skonto trennen:**
> Das Unternehmen möchte Toleranzbeträge strikt von Skontobeträgen trennen, damit die UStVA keine falschen Skontowerte zeigt.
> ➜ `Payment Tolerance Posting = Zahlungstoleranz-Konten`
> **Ergebnis:** Die 20 € Differenz werden auf das `Payment Tolerance Account` aus dem `Gen. Posting Setup` gebucht. Das Skontokonto bleibt unberührt — saubere Trennung für die UStVA.

**Beispiel 2 — Toleranz wie Skonto behandeln:**
> Das Unternehmen betrachtet kleine Differenzen als „nachträgliches Skonto" und möchte sie entsprechend buchen.
> ➜ `Payment Tolerance Posting = Skonto-Konten`
> **Ergebnis:** Die 20 € werden auf das Skontokonto gebucht. Die MwSt wird entsprechend korrigiert. Kann zu MwSt-Rundungsdifferenzen führen.

**Beispiel 3 — Warnung bei Toleranzüberschreitung:**
> Das Unternehmen hat `Pmt. Disc. Tolerance Warning = Ja` gesetzt und die Toleranz wird überschritten.
> ➜ Eine Warnung erscheint, aber die Buchung ist dennoch möglich. Mit `Pmt. Disc. Tolerance Warning = Nein` würde die Überschreitung stillschweigend gebucht werden.

---

### Feld 28: `Pmt. Disc. Excl. VAT` — Skonto v. Nettobetrag

Legt fest, ob Skonto nur vom Nettobetrag berechnet wird.
Offizielle BC-Überschrift: **Skonto v. Nettobetrag**.

```al
trigger OnValidate()
begin
    if "Pmt. Disc. Excl. VAT" then
        TestField("Adjust for Payment Disc.", false);   // Darf NICHT aktiv sein
    else
        TestField("VAT Tolerance %", 0);                 // Darf NICHT >0 sein
end;
```

**Beispiel 1 — B2B: Skonto nur auf den Nettobetrag:**
> Das Unternehmen verkauft eine Maschine für 11.900 € brutto (10.000 € netto + 1.900 € MwSt) mit 2% Skonto. Der Kunde erwartet Skonto nur auf den Nettopreis — branchenüblich im B2B.
> ➜ `Pmt. Disc. Excl. VAT = Ja`
> **Ergebnis:** 2% Skonto = 200 € (vom Netto). Die MwSt wird separat korrigiert. Voraussetzung: `Adjust for Payment Disc. = Nein`.

**Beispiel 2 — B2C: Skonto auf den Bruttobetrag:**
> Ein Online-Händler verkauft an Endverbraucher. Skonto wird immer auf den Gesamtbetrag gewährt — so steht es in den AGB.
> ➜ `Pmt. Disc. Excl. VAT = Nein`
> **Ergebnis:** 2% Skonto auf 11.900 € = 238 €. Die MwSt wird implizit mit-skontiert. Voraussetzung: `VAT Tolerance % = 0`.

**Beispiel 3 — Fehlerhafte Kombinationen:**
> Der Buchhalter versucht, `Pmt. Disc. Excl. VAT = Ja` und gleichzeitig `Adjust for Payment Disc. = Ja` zu setzen.
> **Ergebnis:** Die `OnValidate`-Prüfung schlägt fehl: `TestField("Adjust for Payment Disc.", false)` erzwingt einen Fehler. Diese Kombinationen schließen sich gegenseitig aus. Das System verhindert die widersprüchliche Konfiguration.

---

### Feld 49: `Adjust for Payment Disc.` — Skonto berichtigen

Automatische MwSt-Korrektur bei Skontoabzug.
Offizielle BC-Überschrift: **Skonto berichtigen**.

```al
InitValue = true;   // Standard: aktiviert
```

**Beispiel 1 — Automatische MwSt-Korrektur gewünscht (Standard):**
> Das Unternehmen gewährt 2% Skonto und möchte, dass die MwSt automatisch anteilig korrigiert wird. Der Buchhalter muss sich um nichts kümmern.
> ➜ `Adjust for Payment Disc. = Ja`
> **Ergebnis:** Bei Skontoabzug (23,80 € auf 1.190 € brutto) wird die MwSt automatisch korrigiert: Debitorenkonto 23,80, MwSt-Konto ~3,80. Voraussetzung: `Pmt. Disc. Excl. VAT = Nein`, `VAT Tolerance % = 0`.

**Beispiel 2 — Manuelle MwSt-Korrektur (nicht empfohlen):**
> Das Unternehmen möchte die MwSt-Korrektur manuell vornehmen, z.B. weil komplexe MwSt-Schlüssel im Spiel sind.
> ➜ `Adjust for Payment Disc. = Nein`
> **Ergebnis:** MwSt bleibt unverändert. Skonto wird vom Brutto abgezogen, die MwSt-Differenz bleibt als offen stehen. Der Buchhalter muss am Periodenende manuell korrigieren — zeitaufwändig und fehleranfällig.

**Beispiel 3 — Deaktivierung durch bestehende VAT Posting Setup-Einträge blockiert:**
> Das Unternehmen hat in der Vergangenheit `VAT Posting Setup`-Einträge mit `Adjust for Payment Discount = Ja` eingerichtet. Der Buchhalter versucht nun, `Adjust for Payment Disc.` auf `Nein` zu setzen.
> **Ergebnis:** Fehler: _„VAT Posting Setup [Name] use Adjust for Payment Discount."_ — Erst alle VAT Posting Setup-Einträge deaktivieren, dann die globale Einstellung ändern.

---

## 4.5 Rundung & Nachkommastellen

### Felder 58/59: `Inv. Rounding Precision (LCY)` / `Inv. Rounding Type (LCY)` — Rechnungsrundungspräz. (MW) / Rechnungsrundungsmethode (MW)

Genauigkeit und Verfahren für die Rechnungsrundung in Mandantenwährung (MW).
Offizielle BC-Überschriften: **Rechnungsrundungspräz. (MW)** / **Rechnungsrundungsmethode (MW)**.

**Beispiel 1 — Standard: kaufmännisches Runden auf Cent:**
> Ein deutsches Unternehmen erwartet ganz normale Cent-Rundung. Aus 123,455 wird 123,46.
> ➜ `Inv. Rounding Precision (LCY) = 0.01`, `Inv. Rounding Type (LCY) = Nearest`
> **Ergebnis:** Keine Überraschungen — alle Rechnungsbeträge auf 2 Nachkommastellen.

**Beispiel 2 — Schweizer Unternehmen mit 5-Rappen-Rundung:**
> Ein Unternehmen in der Schweiz stellt Rechnungen in CHF. Die kleinste Münze ist 5 Rappen, daher muss immer auf 5 Rappen aufgerundet werden.
> ➜ `Inv. Rounding Precision (LCY) = 0.05`, `Inv. Rounding Type (LCY) = Aufrunden`
> **Ergebnis:** Rechnungssumme 123,42 wird zu 123,45. Auch 123,41 würde auf 123,45 aufgerundet. Typisch für CHF.

**Beispiel 3 — Rundung ändern, aber es existieren bereits Buchungen:**
> Das Unternehmen hat bereits 2 Jahre mit `Amount Rounding Precision = 0.01` gebucht und möchte nun auf `0.001` wechseln.
> ➜ Versuch, `Amount Rounding Precision` zu ändern
> **Ergebnis:** `CheckRoundingError()` prüft ALLE Ledger-Entry-Tabellen (G/L, Item, Job, Resource, FA, Maintenance, Insurance). Da G/L Entry nicht leer ist → Fehler: _„Sie können den Inhalt des Feldes Amount Rounding Precision nicht ändern, weil es bereits gebuchte Posten gibt."_ Die Rundungsgenauigkeit kann NUR VOR der ersten Buchung gesetzt werden.

**Relevante Prozedur:**
```al
procedure CheckRoundingError(NameOfField: Text[100])
begin
    ErrorMessage := false;
    if GLEntry.FindFirst() then ErrorMessage := true;
    if ItemLedgerEntry.FindFirst() then ErrorMessage := true;
    // ... alle weiteren Ledger-Entry-Tabellen
    OnBeforeCheckRoundingError(ErrorMessage);   // Integrationsereignis
    if ErrorMessage then Error(Text018, NameOfField);
end;
```

---

### Feld 73: `Amount Rounding Precision` — Betragsrundungspräzision

Rundungsgenauigkeit für Geldbeträge (Initialwert = 0.01).
Offizielle BC-Überschrift: **Betragsrundungspräzision**.

**Beispiel 1 — Europäischer Standard mit 2 Nachkommastellen:**
> Eine GmbH in Österreich mit EUR als Mandantenwährung. Ein Positionspreis von 10,00 € × 1,19 = 11,90 € — passt perfekt.
> ➜ `Amount Rounding Precision = 0.01`
> **Ergebnis:** Alle Beträge auf 2 Nachkommastellen. Die `Amount Decimal Places`-Einstellung (Standard "2:2") sorgt für korrekte Anzeige.

**Beispiel 2 — Japanischer Yen ohne Dezimalstellen:**
> Ein japanisches Unternehmen in Tokyo bucht in JPY. Es gibt keine Yen-Cent — alle Beträge sind ganzzahlig.
> ➜ `Amount Rounding Precision = 1`
> **Ergebnis:** Nach dem Speichern erscheint die Meldung: _„Sie müssen das Programm schließen und neu starten, um die Betragsrundung zu aktivieren."_ Erst nach Client-Neustart gilt die neue Rundung.

**Beispiel 3 — Ölindustrie mit 3 Dezimalstellen:**
> Ein Mineralölhändler verkauft Treibstoff zu 1,199 € pro Liter. Die Abrechnung über mehrere tausend Liter erfordert 3 Dezimalstellen, um Rundungsdifferenzen zu vermeiden.
> ➜ `Amount Rounding Precision = 0.001`
> **Ergebnis:** 10.000 Liter × 1,199 € = 11.990,000 € statt 11.990,00 € — die dritte Nachkommastelle verhindert summierte Rundungsfehler über das Jahr.

---

## 4.6 Zusätzliche Berichtswährung

### Feld 68: `Additional Reporting Currency` — Berichtswährung

Währungscode für parallele Buchführung.
Offizielle BC-Überschrift: **Berichtswährung**.

```al
trigger OnValidate()
begin
    if ("Additional Reporting Currency" <> xRec."Additional Reporting Currency") and
       ("Additional Reporting Currency" <> '')
    then begin
        AdjAddReportingCurr.SetAddCurr("Additional Reporting Currency");
        AdjAddReportingCurr.RunModal();   // Bericht "Adjust Add. Reporting Currency"
        if not AdjAddReportingCurr.IsExecuted() then
            "Additional Reporting Currency" := xRec."Additional Reporting Currency";
    end;
    if (...ausgeführt...) then
        DeleteAnalysisView();  // Löscht alle Analyseansichten!
end;
```

**Beispiel 1 — Deutsche Tochter einer US-Muttergesellschaft:**
> Die Muttergesellschaft in New York verlangt Berichte in USD. Das Unternehmen bucht in EUR (Mandantenwährung), möchte aber parallel alle Buchungen in USD sehen.
> ➜ `Additional Reporting Currency = USD`
> **Ergebnis:** Der Bericht "Zusätzliche Berichtswährung anpassen" öffnet sich zur Wechselkurseingabe. Jede Buchung wird parallel in USD gespeichert. **Achtung:** Alle vorhandenen Analyseansichten werden über `DeleteAnalysisView()` gelöscht und müssen neu aufgebaut werden.

**Beispiel 2 — Nur Mandantenwährung (Standard):**
> Ein rein national tätiges Unternehmen ohne Auslandsbezug benötigt keine Parallelwährung.
> ➜ `Additional Reporting Currency = ''` (leer)
> **Ergebnis:** Nur in EUR (Mandantenwährung) buchen. Keine Parallelwährung. Analyseansichten bleiben erhalten.

**Beispiel 3 — Währungsumstellung von USD auf CHF:**
> Die Muttergesellschaft wechselt den Konzernstandard von USD auf CHF. Der Bericht "Zusätzliche Berichtswährung anpassen" öffnet sich. Der Buchhalter stellt die Kurse ein.
> ➜ Wenn der Benutzer den Dialog abbricht (`IsExecuted() = false`), wird der alte Wert beibehalten. Wenn er abschließt, werden die Analyseansichten gelöscht und mit den neuen CHF-Kursen neu aufgebaut.

---

## 4.7 Aufgabenwarteschlange & Hintergrundbuchung

### Feld 50: `Post with Job Queue` — Mit Aufgabenwarteschlange buchen

Ermöglicht das Buchen im Hintergrund über die Aufgabenwarteschlange.
Offizielle BC-Überschrift: **Mit Aufgabenwarteschlange buchen**.

```al
trigger OnValidate()
begin
    if not "Post with Job Queue" then
        "Post & Print with Job Queue" := false;
end;
```

**Beispiel 1 — Ein Buchhalter bucht einzelne Rechnungen direkt:**
> Die Finanzabteilung bucht täglich 20–30 Eingangsrechnungen. Der Buchhalter möchte sofort sehen, ob die Buchung erfolgreich war.
> ➜ `Post with Job Queue = Nein`
> **Ergebnis:** Jede Buchung wird sofort ausgeführt, die GUI zeigt Fortschritt und Ergebnis an. Der Buchhalter wartet kurz auf den Abschluss.

**Beispiel 2 — Monatsabschluss mit 5.000 Buchungen im Batch:**
> Zum Monatsende müssen 5.000 automatische Abschlussbuchungen (Abgrenzungen, Rückstellungen, Abschreibungen) verarbeitet werden. Die Buchhaltung startet den Batch und möchte weiterarbeiten können.
> ➜ `Post with Job Queue = Ja`, `Job Queue Category Code = "MONATSABSCHLUSS"`, `Job Queue Priority for Post = 1000`, `Notify On Success = Ja`
> **Ergebnis:** Die 5.000 Buchungen werden in der Aufgabenwarteschlange eingereiht. Der Buchhalter kann sofort andere Aufgaben erledigen. Bei Abschluss erscheint eine Benachrichtigung.

**Beispiel 3 — Buchen und Drucken im Hintergrund:**
> Das Unternehmen verschickt täglich 200 Ausgangsrechnungen per E-Mail. Nach der Buchung soll automatisch das PDF erzeugt und versendet werden — alles ohne Benutzerinteraktion.
> ➜ `Post & Print with Job Queue = Ja` (erzwingt `Post with Job Queue = Ja`), `Job Q. Prio. for Post & Print = 1000`
> **Ergebnis:** Buchung + PDF-Erzeugung + E-Mail-Versand in einem automatisierten Durchlauf. Der Sachbearbeiter startet den Stapel und widmet sich anderen Aufgaben.

**Relevante Codestellen:**
- `JobQueueActive(): Boolean` — prüft, ob Aufgabenwarteschlange aktiv ist
- **Codeunit "Job Queue"** — verwaltet die Warteschlangeneinträge

---

## 4.8 Sonstige wichtige Felder

### Feld 164: `Show Amounts` — Beträge anzeigen

Steuert, wie Beträge in Sachposten und Berichten angezeigt werden.
Offizielle BC-Überschrift: **Beträge anzeigen**.

```al
OptionMembers = "Amount Only","Debit/Credit Only","All Amounts";
```

**Beispiel 1 — Der Buchhalter möchte nur die Nettobeträge sehen:**
> Ein kleines Unternehmen mit einfacher Buchhaltung benötigt keine Soll/Haben-Trennung.
> ➜ `Show Amounts = Nur Betrag`
> **Ergebnis:** Auf der Sachposten-Seite wird nur `Amount` angezeigt. Soll/Haben separat ist ausgeblendet.

**Beispiel 2 — Der Steuerberater benötigt Soll/Haben-Ansicht:**
> Der Steuerberater prüft die Buchhaltung und besteht auf der klassischen Soll/Haben-Darstellung. Viele Auswertungen setzen diese Ansicht voraus.
> ➜ `Show Amounts = Nur Soll/Haben`
> **Ergebnis:** Zeigt `Debit Amount` und `Credit Amount`. Zahlreiche Codestellen prüfen auf diese Einstellung, z.B. in `AppliedVendorEntries.Page.al`:
> ```al
> AmountVisible := not (GLSetup."Show Amounts" = GLSetup."Show Amounts"::"Debit/Credit Only");
> ```

**Beispiel 3 — Maximale Transparenz für die Konzernberichterstattung:**
> Die Konzernmutter verlangt vollständige Transparenz — Betrag, Soll und Haben in allen Berichten.
> ➜ `Show Amounts = Alle Beträge`
> **Ergebnis:** `Amount`, `Debit Amount` und `Credit Amount` sind sichtbar. Höherer Platzbedarf, aber maximale Nachvollziehbarkeit.

---

### Feld 65: `Summarize G/L Entries` — Sachposten zusammenfassen

Fasst identische Sachposten (gleiches Konto, Datum, Dimensionen) zusammen.
Offizielle BC-Überschrift: **Sachposten zusammenfassen**.

**Beispiel 1 — Ein Großhändler mit Massenbuchungen möchte Speicherplatz sparen:**
> Das Unternehmen bucht täglich 1.000 Rechnungen an denselben Großkunden. Hunderte identische Sachposten würden die Datenbank aufblähen.
> ➜ `Summarize G/L Entries = Ja`
> **Ergebnis:** Alle Buchungen auf dasselbe Konto mit gleichem Datum und gleichen Dimensionen werden zu EINEM Sachposten zusammengefasst. Weniger Datensätze, schnellere Reports.

**Beispiel 2 — Der Wirtschaftsprüfer verlangt Einzelaufzeichnung:**
> Bei der letzten Betriebsprüfung wurde die Zusammenfassung kritisiert. Der Prüfer verlangt jede Buchung als separaten Sachposten.
> ➜ `Summarize G/L Entries = Nein`
> **Ergebnis:** Jede Buchung = ein eigener Sachposten. Maximale Detailtiefe, mehr Speicherplatz, aber prüfungssicher.

**Beispiel 3 — Performance bei Massenbuchungen:**
> Bei 10.000 Rechnungen an denselben Debitor mit identischen Dimensionen reduziert `Summarize = Ja` die Sachposten von 10.000 auf ca. 30 (je nach Kontenanzahl). In der Vorschau (`GLEntryPostingPreview.Table.al`) bereits sichtbar.

---

### Feld 56: `Mark Cr. Memos as Corrections` — Gutschriften als Storno mark.

Markiert Gutschriften als Korrekturbuchungen für MwSt und Berichtswesen.
Offizielle BC-Überschrift: **Gutschriften als Storno mark.**

**Beispiel 1 — Deutsche UStVA verlangt Trennung von Korrekturen:**
> Das Unternehmen muss in der Umsatzsteuer-Voranmeldung Korrekturgutschriften separat in Zeile 56 ausweisen. Das Finanzamt akzeptiert keine vermischten Werte.
> ➜ `Mark Cr. Memos as Corrections = Ja`
> **Ergebnis:** Gutschriften erhalten im Feld `Correction` auf dem Sachposten den Wert `True`. Die MwSt-Meldung wertet diese separat aus — korrekte UStVA.

**Beispiel 2 — Gutschriften als normale Minus-Buchungen:**
> Ein Unternehmen ohne besondere MwSt-Anforderungen behandelt Gutschriften wie normale Buchungen mit negativem Vorzeichen.
> ➜ `Mark Cr. Memos as Corrections = Nein`
> **Ergebnis:** Gutschriften erscheinen als negative Umsätze. In der MwSt-Meldung können sie fälschlich in der falschen Zeile landen — problematisch bei Betriebsprüfungen.

**Beispiel 3 — Internationale Konzerne mit abweichenden Anforderungen:**
> Die österreichische Tochter benötigt `Mark Cr. Memos as Corrections = Ja` für die UVA, die Schweizer Tochter nicht.
> ➜ Pro Unternehmen separat einstellbar (Mandantenkonfiguration). Die Auswirkung ist direkt im Feld `Correction` der Sachposten sichtbar.

---

## 4.9 Kontenplan & Sachkonten (Tabelle 15)

> **Tabelle 15:** `GLAccount.Table.al`
> **Namensraum:** `Microsoft.Finance.GeneralLedger.Account`
> **Seiten:** `ChartofAccounts.Page.al`, `GLAccountCard.Page.al`

Der **Kontenplan** ist das Fundament der Finanzbuchhaltung. Jedes Sachkonto (`G/L Account`) besitzt eine eindeutige Nummer, eine Bilanz/GuV-Klassifikation, Buchungsgruppen-Zuweisungen und einen Kontosperrstatus. Die Hierarchie wird über Einrückung (`Indentation`) abgebildet.

### Gliederung: Bilanz vs. GuV

```al
field(4; "Income/Balance"; Enum "G/L Account Income/Balance")
{
    Caption = 'Income/Balance';
}
```
Optionen: `Income Statement` (GuV) | `Balance Sheet` (Bilanz)

> ⚠️ Entscheidend für den Jahresabschluss: `CloseIncomeStatement.Report.al` bucht den Saldo aller GuV-Konten auf das Bilanzkonto für `Retained Earnings`.

### Buchungsgruppen-Zuweisung

Jedes Konto muss einer **Gen. Business Posting Group** und **Gen. Product Posting Group** zugewiesen werden:

```al
field(13; "Gen. Bus. Posting Group"; Code[20])
{
    TableRelation = "Gen. Business Posting Group";
    NotBlank = true;
}
field(14; "Gen. Prod. Posting Group"; Code[20])
{
    TableRelation = "Gen. Product Posting Group";
    NotBlank = true;
}
```

Diese beiden Felder steuern, welches **Gegenkonto** bei Buchungen verwendet wird — festgelegt in `General Posting Setup` (Tabelle 252).

**Beispiel 1 — Kontenplan eines Produktionsbetriebes anlegen:**
> Ein mittelständischer Maschinenbauer richtet seinen Kontenplan nach dem SKR04 ein. Aktivkonten (0–3), Passivkonten (4–6), GuV-Konten (7–9).
> ➜ `Income/Balance = Balance Sheet` für Konto 0800 (Fuhrpark), `Income/Balance = Income Statement` für Konto 8400 (Erlöse). `Gen. Bus. Posting Group = INLAND`, `Gen. Prod. Posting Group = MASCHINEN`.
> **Ergebnis:** Jede Rechnungsbuchung findet über die Buchungsmatrix ihr korrektes Gegenkonto. Der Jahresabschlussbericht gruppiert Bilanz- und GuV-Positionen automatisch.

**Beispiel 2 — Ein Automobilzulieferer muss Kapazitätskonten sperren:**
> Die Produktion wurde vorübergehend stillgelegt. Der Buchhalter sperrt die entsprechenden Aufwandskonten.
> ➜ `Blocked = All` auf Konto 4300 (Fremdleistungen).
> **Ergebnis:** Keine Buchung auf Konto 4300 möglich. Die `CheckGLAccountIsBlocked()`-Prüfung in `GenJnlCheckLine.Codeunit.al` schlägt mit Fehler _„Sachkonto 4300 ist gesperrt."_ fehl.

**Beispiel 3 — Kontenplan-Erweiterung für IFRS:**
> Die Muttergesellschaft verlangt IFRS-Reporting. Der Konzernbuchhalter erweitert den Kontenplan um IFRS-spezifische Konten (z.B. Leasingverbindlichkeiten).
> ➜ Neue Konten mit `Gen. Bus. Posting Group = KONZERN` und `Gen. Prod. Posting Group = IFRS` anlegen.
> **Ergebnis:** Über die separate Buchungsgruppe können IFRS-Buchungen von HGB-Buchungen getrennt ausgewertet werden.

**Querverweis:** → [Kap. 5 Vertrieb]({{ '/05-sales-marketing/' | relative_url }}) — wie Debitorenbuchungsgruppen die Fibu-Konten steuern

---

## 4.10 Buchungsgruppen — Die Buchungsmatrix

> **Kern-Tabellen:** `GenBusinessPostingGroup.Table.al` (110), `GenProductPostingGroup.Table.al` (111), `GeneralPostingSetup.Table.al` (252)
> **Namensraum:** `Microsoft.Finance.GeneralLedger.Setup`

Die Buchungsmatrix ist das zentrale Routing-System für jede Buchung. Sie ordnet jeder Kombination aus **Geschäftsbuchungsgruppe** (Debitor/Kreditor-Typ) und **Produktbuchungsgruppe** (Artikel-Typ) die korrekten Sachkonten zu.

### Wie die Buchungsmatrix arbeitet

```
Gen. Business Posting Group  ─┐
  (z.B. INLAND, EU, DRITT)    ├──→ Gen. Posting Setup (252) ──→ Sachkonten
Gen. Product Posting Group   ─┘     (Umsatzerlöse, COGS, Bestand, ...)
```

In `General Posting Setup` sind für jede Kombination folgende Konten hinterlegt:

| Feld | Debitoren-Seite (Verkauf) | Kreditoren-Seite (Einkauf) |
|---|---|---|
| `Sales Account` | Umsatzerlöse | — |
| `Purchase Account` | — | Aufwand/Einkauf |
| `Inventory Adjmt. Account` | Bestandsveränderung | Bestandsveränderung |
| `COGS Account` | Wareneinsatz | — |
| `Direct Cost Applied Account` | Verrechnete Einzelkosten | Verrechnete Einzelkosten |
| `Overhead Applied Account` | Verrechnete Gemeinkosten | Verrechnete Gemeinkosten |

### MwSt-Buchungsmatrix

Parallel dazu existiert `VAT Posting Setup` (Tabelle 325) für die MwSt-Kontenfindung:
```
VAT Bus. Posting Group  ─┐
  (z.B. INLAND, EU)       ├──→ VAT Posting Setup (325) ──→ MwSt-Konten
VAT Prod. Posting Group  ─┘     (VAT%, Vorsteuer, Umsatzsteuer, ...)
```

**Beispiel 1 — Ein Unternehmen verkauft an inländische, EU- und Drittlandkunden:**
> Das Unternehmen benötigt drei Geschäftsbuchungsgruppen:
> ➜ `Gen. Bus. Posting Group = INLAND`, `Gen. Bus. Posting Group = EU`, `Gen. Bus. Posting Group = DRITT`.
> Für jede Gruppe wird das `Gen. Posting Setup` hinterlegt — z.B. `Sales Account` für DRITT auf Konto 8330 (steuerfreie Ausfuhrlieferungen).
> **Ergebnis:** Bei einer Rechnung an einen Schweizer Kunden (DRITT) bucht das System automatisch auf Konto 8330 — ohne manuelle Kontierung.

**Beispiel 2 — Ein neues Produktsortiment erfordert getrennte Erlöskonten:**
> Das Unternehmen erweitert sein Sortiment um Dienstleistungen. Der Controller möchte Dienstleistungserlöse getrennt von Warenverkäufen sehen.
> ➜ Neue `Gen. Prod. Posting Group = DIENSTLEISTUNG` anlegen. Im `Gen. Posting Setup` für Kombination `INLAND/DIENSTLEISTUNG` das `Sales Account = 8401` (Erlöse Dienstleistungen 19% USt).
> **Ergebnis:** Verkaufsrechnungen werden automatisch auf das richtige Erlöskonto gebucht. Die GuV zeigt Waren und Dienstleistungen getrennt.

**Beispiel 3 — Copy-Funktion für neue Buchungsgruppe:**
> Das Unternehmen eröffnet eine Niederlassung in Österreich. Die Buchungsmatrix für INLAND existiert bereits, nun soll sie für `Gen. Bus. Posting Group = AT-INLAND` kopiert werden.
> ➜ `Copy General Posting Setup`-Bericht ausführen — Quelle: `INLAND`, Ziel: `AT-INLAND`.
> **Ergebnis:** Alle 25+ Kontenzeilen werden in einem Schritt kopiert. Nur abweichende Konten (z.B. österreichisches Erlöskonto) müssen manuell angepasst werden.

**Querverweise:**
- → [Kap. 6 Einkauf]({{ '/06-purchasing/' | relative_url }}) — Kreditorenbuchungsgruppen und Einkaufskonten
- → [Kap. 7 Lager]({{ '/07-inventory/' | relative_url }}) — Inventur- und Bestandsbuchungen und ihre Fibu-Auswirkungen

---

## 4.11 MwSt & Umsatzsteuer

> **Kern-Tabellen:** `VATPostingSetup.Table.al` (325), `VATEntry.Table.al` (254), `VATStatement.Table.al`
> **Namensräume:** `Microsoft.Finance.VAT.Calculation`, `Microsoft.Finance.VAT.Reporting`, `Microsoft.Finance.VAT.Setup`

Das MwSt-System in BC28 ist mehrschichtig: Einrichtung (VAT Posting Setup) → Berechnung (VAT Calculation) → Buchung (VAT Entries) → Meldung (VAT Statement).

### VAT Posting Setup (325)

```al
field("VAT Bus. Posting Group"; Code[20])       // z.B. INLAND, EU, DRITT
field("VAT Prod. Posting Group"; Code[20])       // z.B. NORMAL, ERMÄSSIGT, STEUERFREI
field("VAT %"; Decimal)                           // z.B. 19.00
field("Sales VAT Account"; Code[20])              // Umsatzsteuerkonto
field("Purchase VAT Account"; Code[20])           // Vorsteuerkonto
field("VAT Identifier"; Code[20])                 // MwSt-Schlüssel für Meldung
```

### MwSt-Abrechnung

`CalcandPostVATSettlement.Report.al` berechnet den MwSt-Saldo und bucht die Zahllast/Vorsteuerüberhang auf das entsprechende Konto.

**Beispiel 1 — Standard-Konfiguration für Deutschland:**
> Das Unternehmen benötigt 3 MwSt-Sätze: 19%, 7%, steuerfrei (0%).
> ➜ `VAT Prod. Posting Group = NORMAL` (19%), `ERMÄSSIGT` (7%), `STFREI` (0%). Für `VAT Bus. Posting Group = INLAND` werden alle drei Sätze im `VAT Posting Setup` hinterlegt.
> **Ergebnis:** Bei Buchung einer Rechnung über 1.190 € (1.000 + 190 USt) landen 190 € auf dem Konto _Umsatzsteuer 19%_ (1776). Die MwSt-Meldung erfasst diesen Betrag automatisch.

**Beispiel 2 — Innergemeinschaftliche Lieferung:**
> Das Unternehmen liefert eine Maschine nach Frankreich (EU). Die Rechnung muss ohne MwSt ausgestellt werden (USt-ID-Prüfung).
> ➜ `VAT Bus. Posting Group = EU`, `VAT Prod. Posting Group = NORMAL`. Im `VAT Posting Setup` wird `VAT % = 0`, `VAT Identifier = 41` (innergem. Lieferung) gesetzt.
> **Ergebnis:** Das System prüft automatisch die USt-ID des Kunden (`VATRegistrationNoCheck.Report.al`). Bei gültiger ID erscheint keine MwSt auf der Rechnung, aber die _Zusammenfassende Meldung_ erfasst den Umsatz.

**Beispiel 3 — IST-Versteuerung mit unrealisierter MwSt:**
> Das Unternehmen hat `Unrealized VAT = Ja` gesetzt (siehe §4.3). Das `VAT Posting Setup` benötigt für die betroffenen Kombinationen `Unrealized VAT Type = Percentage`.
> ➜ Im `VAT Posting Setup`: `Unrealized VAT Type = Percentage` für `INLAND/NORMAL`.
> **Ergebnis:** Die MwSt wird erst bei Zahlungseingang realisiert und gemeldet — nicht schon bei Rechnungsstellung. Vorteil für die Liquidität, mehr Aufwand in der Buchhaltung.

**Querverweis:** → [Kap. 4.1 §4.3 MwSt-Felder der Fibu-Einrichtung](#43-mehrwertsteuer) — `Unrealized VAT`, `VAT Reporting Date`, `Control VAT Period` in Tabelle 98

---

## 4.12 Fibu-Buchungsjournale

> **Kern-Tabellen:** `GenJournalTemplate.Table.al` (80), `GenJournalBatch.Table.al` (81), `GenJournalLine.Table.al` (82)
> **Namensraum:** `Microsoft.Finance.GeneralLedger.Journal`

Buchungsjournale sind der tägliche Arbeitsplatz für Fibu-Buchungen. Ein **Template** (z.B. ALLGEMEIN) enthält **Batches** (z.B. JANUAR2026), die wiederum **Zeilen** enthalten.

```al
// GenJournalLine: Kern-Felder
field("Account Type"; Option)         // G/L Account, Customer, Vendor, Bank Account, Fixed Asset, IC Partner
field("Account No."; Code[20])        // Kontennummer
field("Posting Date"; Date)
field("Document No."; Code[20])
field("Amount"; Decimal)
field("Bal. Account Type"; Option)    // Gegenkonto-Typ
field("Bal. Account No."; Code[20])   // Gegenkonto
```

Die Buchungslogik ist auf mehrere Codeunits verteilt:
- `GenJnlCheckLine.Codeunit.al` — Validierung jeder Zeile
- `GenJnlPostLine.Codeunit.al` — Einzelzeilenbuchung  
- `GenJnlPostBatch.Codeunit.al` — Stapelbuchung
- `GenJnlPostviaJobQueue.Codeunit.al` — Hintergrundbuchung

**Beispiel 1 — Buchhalter bucht monatliche Mietkosten:**
> Jeden Monatsersten wird die Büromiete fällig: 2.000 € + 380 € MwSt.
> ➜ `Account Type = G/L Account`, `Account No. = 4210` (Mietaufwand), `Amount = 2.000`, `Bal. Account Type = G/L Account`, `Bal. Account No. = 1600` (Verbindlichkeiten). MwSt über das Feld `VAT Prod. Posting Group` automatisch.
> **Ergebnis:** Soll an Mietaufwand 2.000 € / Haben an Verbindlichkeiten 2.380 € / Haben an Vorsteuer 380 €. Die Zeilen-Validierung (`GenJnlCheckLine`) prüft, ob Konto 4210 existiert und nicht gesperrt ist.

**Beispiel 2 — Wiederkehrende Buchung als Vorlage speichern:**
> Der Buchhalter bucht jeden Monat dieselben 5 Standardbuchungen (Miete, Leasing, Versicherung, Telefon, Reinigung).
> ➜ `Gen. Journal Template = ALLGEMEIN`, alle 5 Zeilen erfassen und über `Save as Standard Gen. Journal` speichern.
> **Ergebnis:** Im nächsten Monat wird der Standard-Journal geladen, nur die Belegnummern werden angepasst — 80% Zeiteinsparung.

**Beispiel 3 — IC-Partner-Buchung mit automatischer Gegenbuchung:**
> Die Holding (DE) verbucht eine Verrechnung an die Tochter (AT) über 50.000 € für Management Fees.
> ➜ `Account Type = IC Partner`, `Account No. = AT-TOCHTER`, `IC Partner Bal. Account No. = 4800` (Erlöse aus Beteiligung).
> **Ergebnis:** Das System erzeugt automatisch einen IC-Outbox-Eintrag für AT. Sobald die AT-Tochter die IC-Inbox verarbeitet, entsteht dort die korrespondierende Gegenbuchung — ohne doppelte Erfassung.

**Querverweise:**
→ [Kap. 4.14 Intercompany](#414-intercompany)
→ [Kap. 4.7 Aufgabenwarteschlange](#47-aufgabenwarteschlange--hintergrundbuchung) — `Post with Job Queue`

---

## 4.13 Debitoren & Kreditoren (Receivables & Payables)

> **Namensraum:** `Microsoft.Finance.ReceivablesPayables`
> **Kern-Tabellen:** `CustLedgerEntry.Table.al` (21), `VendLedgerEntry.Table.al` (25), `DetailedCustLedgEntry.Table.al` (379)

Die OP-Verwaltung (Offene Posten) verbindet Fakturierung mit Zahlung. Jede gebuchte Rechnung erzeugt Posten in `CustLedgerEntry` (Debitor) oder `VendLedgerEntry` (Kreditor) sowie den dazugehörigen `Detailed...LedgEntry` (Detailposten).

### Zahlungsausgleich

Der Ausgleich von Zahlungen mit offenen Rechnungen erfolgt über `GenJnlApply.Codeunit.al`. Pro Ausgleichszeile entsteht ein `CustLedgerEntry` vom Typ `Application`.

**Beispiel 1 — Ein Kunde zahlt per Bank und der Ausgleich läuft automatisch:**
> Kunde Müller GmbH zahlt 4.760 € auf zwei offene Rechnungen (2.380 € + 2.380 €). Die Zahlung wird über den Kontoauszug (`Bank Acc. Reconciliation`) importiert.
> ➜ `Account Type = Customer`, `Account No. = 10000`, `Amount = -4.760`, `Applies-to Doc No. = RE001, RE002`
> **Ergebnis:** Das System gleicht automatisch aus. Beide Rechnungen sind geschlossen. Kein manuelles OP-Abgleichen nötig.

**Beispiel 2 — Teilzahlung mit Skonto:**
> Kunde zahlt 1.166,20 € auf eine Rechnung über 1.190 € brutto — 2% Skonto (23,80 €) abgezogen.
> ➜ Zahlungseingang 1.166,20 €, Skontobetrag 23,80 €, `Pmt. Disc. Excl. VAT = Ja` → nur vom Netto skontiert.
> **Ergebnis:** Der Debitorenposten ist ausgeglichen, das Skontokonto und die MwSt werden korrekt korrigiert (s. §4.4). Der Kunde erhält keine Mahnung.

**Beispiel 3 — OP-Liste und Mahnung:**
> Das Unternehmen verschickt monatlich Mahnbriefe an säumige Zahler. Grundlage ist die OP-Liste pro Debitor.
> ➜ Bericht `Customer Statement` zeigt alle offenen Posten. `Reminder`-Codeunit erzeugt Mahnstufen (1, 2, 3) mit Mahngebühren.
> **Ergebnis:** Automatisierte Mahnläufe. Jede Mahnung erzeugt einen neuen Posten (`Reminder/Finance Charge Entry`).

**Querverweise:**
→ [Kap. 5 Vertrieb]({{ '/05-sales-marketing/' | relative_url }}) — Debitorenstamm, `Customer Posting Group`, `Customer`-Tabelle  
→ [Kap. 6 Einkauf]({{ '/06-purchasing/' | relative_url }}) — Kreditorenstamm, `Vendor Posting Group`, `Vendor`-Tabelle

---

## 4.14 Bank & Zahlungsverkehr

> **Namensräume:** `Microsoft.Bank.*`, `Microsoft.CashFlow.*`
> **Kern-Tabellen:** `BankAccount.Table.al` (270), `BankAccReconciliation.Table.al` (273), `BankAccReconLine.Table.al` (274)

### SEPA-Export & Zahlungsdateien

BC28 unterstützt SEPA Credit Transfers (Überweisungen) und Direct Debits (Lastschriften) über den `Payment Journal`.

**Beispiel 1 — Zahllauf für 50 Kreditorenrechnungen:**
> Das Unternehmen bezahlt jeden Dienstag alle fälligen Kreditorenrechnungen per SEPA-Überweisung.
> ➜ `Payment Journal` → Kreditorenrechnungen auswählen (`Suggest Vendor Payments`) → `Export SEPA Payment File`.
> **Ergebnis:** Eine XML-Datei (pain.001) wird erzeugt und an die Bank übermittelt. 50 Einzelüberweisungen in einer Sammeldatei.

**Beispiel 2 — Bankkontoauszug importieren und abstimmen:**
> Der Bankkontoauszug kommt als CAMT.053-Datei. Das Unternehmen importiert ihn und gleicht ihn mit den gebuchten Posten ab.
> ➜ `Bank Acc. Reconciliation` → `Import Bank Statement` → CAMT.053 auswählen.
> **Ergebnis:** Gebuchte Posten werden automatisch zugeordnet (`Match`). Differenzen (z.B. Bankgebühren, Zinsen) werden als neue Fibu-Buchungen erfasst — kompletter Bankabstimmungs-Workflow.

**Beispiel 3 — SEPA-Lastschrift für 200 Kunden:**
> Das Unternehmen zieht monatlich per SEPA-Basislastschrift bei 200 Kunden ein.
> ➜ `Payment Journal` mit `Gen. Journal Template = ZAHLUNG`, `Bal. Account Type = Bank Account`, `Account Type = Customer`.
> **Ergebnis:** Eine SEPA Direct Debit XML-Datei (pain.008) entsteht. Bei Buchung werden 200 Debitorenposten automatisch ausgeglichen. Keine manuelle Zahlungszuordnung.

**Querverweis:** → [Kap. 4.1 §4.7 Aufgabenwarteschlange](#47-aufgabenwarteschlange--hintergrundbuchung)

---

## 4.15 Anlagen (Fixed Assets)

> **Namensräume:** `Microsoft.FixedAssets.*`
> **Kern-Tabellen:** `FixedAsset.Table.al` (5600), `FADepreciationBook.Table.al` (5612), `FALedgerEntry.Table.al` (5614)

Anlagen (Gebäude, Maschinen, Fuhrpark) werden über Anlagenkarten verwaltet, linear oder degressiv abgeschrieben. Jede Abschreibungsbuchung erzeugt einen Fibu-Posten.

```al
// FADepreciationBook: AfA-Parameter
field("Depreciation Method"; Option)     // Straight-Line, Declining-Balance, ...
field("Depreciation %"; Decimal)
field("No. of Depreciation Years"; Integer)
```

**Beispiel 1 — Eine neue CNC-Maschine wird aktiviert und abgeschrieben:**
> Das Unternehmen kauft eine CNC-Maschine für 120.000 € netto, Nutzungsdauer 10 Jahre, lineare AfA.
> ➜ `Fixed Asset Card`: Anschaffungskosten = 120.000 €, `Depreciation Method = Straight-Line`, `No. of Depreciation Years = 10`.
> **Ergebnis:** Monatliche AfA-Buchung: 1.000 € Soll an AfA-Konto (4000), Haben an Anlagekonto (0700). Der `FALedgerEntry` enthält den Restbuchwert.

**Beispiel 2 — Vorzeitiger Abgang einer Anlage:**
> Ein Firmenwagen (Buchwert 15.000 €) wird nach 3 Jahren für 12.000 € verkauft — Verlust 3.000 €.
> ➜ Anlagenverkauf über `FA Journal`. Buchung: Bank 12.000 € / Erlös aus Anlageabgang 12.000 €, Restbuchwert 15.000 € / Anlageabgang 15.000 €, Verlust 3.000 €.
> **Ergebnis:** Das Anlagekonto wird vollständig aufgelöst. GuV-wirksamer Verlust von 3.000 €.

**Beispiel 3 — Geringwertige Wirtschaftsgüter (GWG):**
> Das Unternehmen kauft einen Bürostuhl für 450 € netto (unter 800 € Grenze) und schreibt ihn sofort voll ab.
> ➜ `Depreciation Method = Straight-Line`, `No. of Depreciation Years = 1` oder über das GWG-Profil: Sofortabschreibung im 1. Jahr.
> **Ergebnis:** 450 € werden im Anschaffungsjahr voll abgeschrieben. Keine Verteilung über mehrere Jahre.

**Querverweise:**
→ [Kap. 6 Einkauf]({{ '/06-purchasing/' | relative_url }}) — Anlagen aus Einkaufsbestellung aktivieren  
→ [Kap. 7 Lager]({{ '/07-inventory/' | relative_url }}) — Bestandsbewertung vs. Anlagenbewertung

---

## 4.16 Währung & Wechselkurse

> **Namensraum:** `Microsoft.Finance.Currency`
> **Kern-Tabellen:** `Currency.Table.al` (4), `CurrencyExchangeRate.Table.al` (330)

Fremdwährungsbuchungen erfordern Wechselkurse. BC28 bewertet offene Posten periodisch neu und bucht Kursdifferenzen.

```al
// ExchRateAdjmtReg: Kursanpassungs-Register
field("Currency Code"; Code[10])
field("Starting Date"; Date)
field("Ending Date"; Date)
field("Adjustment Date"; Date)
```

**Beispiel 1 — EUR-basierte Firma kauft in USD ein:**
> Das Unternehmen kauft Waren für 10.000 USD bei einem Kurs von 1,10 USD/EUR (= 9.090,91 €). Bei Zahlung (30 Tage später) beträgt der Kurs 1,12 USD/EUR (= 8.928,57 €).
> ➜ Kursgewinn: 162,34 €.
> **Ergebnis:** Bei Zahlung wird die Kursdifferenz automatisch auf das `Realized Gain`-Konto gebucht. Der Bericht `Adjust Exchange Rates` (Wechselkurse anpassen) berücksichtigt das.

**Beispiel 2 — Periodische Kursanpassung für OP-Liste:**
> Die USD-Debitorenposten (offene Kundenforderungen) werden zum Monatsultimo mit dem neuen Kurs bewertet.
> ➜ Bericht "`Adjust Exchange Rates`" ausführen. Parameter: `Adjustment Date = 30.06.2026`, `Currency Code = USD`.
> **Ergebnis:** Alle offenen USD-Posten erhalten eine Kursdifferenz-Buchung. Der unrealisierte Gewinn/Verlust wird auf das entsprechende Konto gebucht.

**Beispiel 3 — Wechselkurs automatisch über Service aktualisieren:**
> Das Unternehmen hat `CurrExchRateUpdateSetup` für die EZB konfiguriert.
> ➜ `Update Currency Exchange Rates`-Codeunit läuft täglich über die Aufgabenwarteschlange und holt die aktuellen EZB-Referenzkurse.
> **Ergebnis:** Die Wechselkurstabelle ist stets aktuell. Kein manuelles Einpflegen der Kurse.

---

## 4.17 Finanzberichte & Analyse

> **Namensraum:** `Microsoft.Finance.FinancialReports`
> **Kern-Tabellen:** `AccountScheduleName.Table.al` (91), `AccountScheduleLine.Table.al` (92), `ColumnLayout.Table.al` (93)

Account Schedules sind das zentrale Reporting-Tool. Zeilen (`AccountScheduleLine`) definieren WAS angezeigt wird, Spalten (`ColumnLayout`) definieren WIE (Periode, Vergleichszeitraum).

```al
// AccountScheduleLine: Zeilendefinition
field("Row No."; Integer)              // Zeilennummer
field("Description"; Text[100])         // Zeilenbeschreibung
field("Totaling Type"; Option)          // Posting Accounts, Formula, Net Change, ...
field("Totaling"; Text[250])            // Konten-Filter oder Formel
field("Show"; Option)                   // Ja/Nein, nur bei Betrag <> 0, ...
```

**Beispiel 1 — Eine einfache Bilanz aufbauen:**
> Das Unternehmen benötigt eine monatliche Bilanz (Aktiva + Passiva).
> ➜ Zeilen: `Row No. 10 = Sachanlagen (Konten 0100..0799)`, `Row No. 20 = Vorräte (Konten 1000..1999)`, `Row No. 30 = Summe Aktiva (Formel: 10+20)`. Spalten: `Column 1 = Aktueller Monat (Period), Column 2 = Vorjahresmonat (Comparison Period)`.
> **Ergebnis:** Der Bericht `Bilanz` zeigt alle Aktiva, Vorräte und Summe — mit Vorjahresvergleich.

**Beispiel 2 — GuV nach Kostenstellen:**
> Der Controller möchte eine GuV getrennt nach Kostenstellen sehen.
> ➜ `Dimension Perspective` für Kostenstelle anlegen (z.B. KST-VERTRIEB, KST-PRODUKTION). Die `AccountScheduleLine` referenziert die Dimension Perspective.
> **Ergebnis:** Eine GuV pro Kostenstelle — aus einer einzigen Account Schedule-Definition.

**Beispiel 3 — Monatliche Cashflow-Rechnung:**
> Der Finanzleiter möchte den Cashflow der letzten 12 Monate rollierend darstellen.
> ➜ `ColumnLayout` mit `Column Type = Formula` und `Comparison Period Formula = -1M..-12M`. In der Zeile: Konten der liquiden Mittel (1000..1299).
> **Ergebnis:** Der Bericht "Cashflow Statement" zeigt die monatliche Entwicklung — automatisch rollierend ohne manuelle Anpassung.

**Querverweis:** → [Kap. 4.1 Fibu-Einrichtung](#4-finanzen--fibu-einrichtung) — Felder `Fin. Rep. for Balance Sheet`, `Fin. Rep. for Income Stmt` in Tabelle 98

---

## 4.18 Budget

> **Namensraum:** `Microsoft.Finance.GeneralLedger.Budget`
> **Kern-Tabellen:** `GLBudgetName.Table.al` (89), `GLBudgetEntry.Table.al` (90)

Budgeteinträge werden pro Konto und Periode erfasst und mit IST-Werten in Account Schedules verglichen.

**Beispiel 1 — Jahresbudget 2026 für den Vertriebsbereich:**
> Der Vertriebsleiter plant 600.000 € Umsatz (Konto 8400) für 2026 — gleichmäßig verteilt auf 12 Monate.
> ➜ `Budget Name = UMSATZ2026`, `Budget Entry`: Konto 8400, Zeilen pro Monat à 50.000 €.
> **Ergebnis:** In der Account Schedule kann eine Spalte mit `Column Type = Budget` die IST- mit den BUDGET-Werten monatlich vergleichen.

**Beispiel 2 — Budget aus Excel importieren:**
> Der Controller hat das Budget in Excel erstellt und möchte es direkt nach BC importieren.
> ➜ Bericht `Import Budget from Excel` mit definierter Excel-Vorlage (`FinReportExcelTemplate`).
> **Ergebnis:** 500 Budgetzeilen werden in einem Rutsch importiert — keine manuelle Erfassung.

**Beispiel 3 — Budgetkontrolle mit Warnung:**
> Das Unternehmen möchte beim Buchen einer Bestellung gewarnt werden, wenn das Budget überschritten wird.
> ➜ `GLBudgetOpen.Codeunit.al` prüft bei Buchung: Budgetsumme vs. IST-Kosten. Bei Überschreitung: Warnung.
> **Ergebnis:** Echtzeit-Budgetkontrolle bereits bei der Bestellerfassung — nicht erst am Monatsende.

**Querverweis:** → [Kap. 6 Einkauf]({{ '/06-purchasing/' | relative_url }}) — Budgetkontrolle bei Bestellfreigabe

---

## 4.19 Konsolidierung

> **Namensraum:** `Microsoft.Finance.Consolidation`
> **Kern-Tabellen:** `ConsolidationSetup.Table.al` (96), `BusinessUnit.Table.al` (97), `ConsolidationAccount.Table.al` (95)

Konsolidierung fasst mehrere Mandanten (Business Units) in einem Konsolidierungsmandanten zusammen — essenziell für Konzerne.

**Beispiel 1 — Holding mit 3 Tochtergesellschaften konsolidieren:**
> Die Holding (DE) konsolidiert ihre Töchter AT, CH, NL monatlich.
> ➜ Jede Tochter als `Business Unit` anlegen, Konsolidierungskonten (`Consolidation Account`) aus dem Quellmandanten zuweisen.
> **Ergebnis:** Der Bericht `Consolidate` überträgt die Salden aller Töchter in den Konsolidierungsmandanten. Die Konzernbilanz und -GuV entstehen automatisch.

**Beispiel 2 — Währungsumrechnung für die Konsolidierung:**
> Die CH-Tochter bucht in CHF, die Konzernmutter in EUR.
> ➜ In der `Consolidation Setup` wird der CHF-zu-EUR-Wechselkurs hinterlegt. Die Konsolidierung rechnet alle CHF-Posten zu Stichtagskurs um.
> **Ergebnis:** Konzernabschluss in EUR. Währungskursdifferenzen werden auf das `Currency Translation Adjustment`-Konto gebucht.

**Beispiel 3 — Intercompany-Eliminierungen:**
> Die DE-Holding hat Forderungen von 100.000 € gegen die AT-Tochter im Soll. Die AT-Tochter hat Verbindlichkeiten von 100.000 € gegen die DE-Holding im Haben.
> ➜ `G/L Consolidation Eliminations`-Bericht bucht automatisch eine Eliminierungszeile: Soll Verbindlichkeiten 100.000 € / Haben Forderungen 100.000 €.
> **Ergebnis:** Konzernbilanz ist frei von Intercompany-Salden.

**Querverweis:** → [Kap. 4.14 Intercompany](#414-intercompany)

---

## 4.20 Abgrenzungen (Deferrals)

> **Namensraum:** `Microsoft.Finance.Deferral`
> **Kern-Tabellen:** `DeferralHeader.Table.al` (1700), `DeferralLine.Table.al` (1701)

Abgrenzungen verteilen Aufwände oder Erträge über mehrere Perioden. Beispiel: Eine Jahresmiete (12.000 €) soll monatlich mit 1.000 € gebucht werden.

```al
field("Deferral %"; Decimal)             // z.B. 8.33 (=100%/12)
field("Deferral Starting Date"; Date)
field("Deferral Ending Date"; Date)
field("No. of Periods"; Integer)
```

**Beispiel 1 — Jahresversicherung wird periodengerecht abgegrenzt:**
> Das Unternehmen bezahlt am 01.01. eine Jahresversicherung über 6.000 €.
> ➜ `Deferral Template = VERSICHERUNG`, 12 Monate, `Deferral % = 8.33`, `Deferral Starting Date = 01.01.2026`.
> **Ergebnis:** Bei jeder Fibu-Buchung über 6.000 € an Versicherungsaufwand erzeugt das System zusätzlich 12 monatliche Abgrenzungszeilen: 500 € Soll VersAufwand / 500 € Haben ARA. Monatlich korrekt periodisiert.

**Beispiel 2 — Vorauszahlung eines Kunden wird aufgelöst:**
> Ein Kunde zahlt 24.000 € für einen 2-Jahres-Wartungsvertrag. Der Ertrag soll monatlich mit 1.000 € gebucht werden.
> ➜ Deferral Line: 24 Monate, `Deferral % = 4.17`, `Deferral Starting Date = 01.01.2026`.
> **Ergebnis:** Monatlich wird 1.000 € Ertrag aus der Abgrenzung aufgelöst. Die GuV zeigt monatlich den korrekten Service-Ertrag.

**Beispiel 3 — Abgrenzungsvorlagen für Standard-Fälle:**
> Das Unternehmen hat 5 Standard-Abgrenzungen (Versicherung, Miete, Wartung, Leasing, Lizenzen).
> ➜ 5 `Deferral Templates` anlegen. Der Benutzer wählt nur noch die Vorlage aus — Startdatum und Prozentsatz werden automatisch gesetzt.
> **Ergebnis:** Standardisierung und Fehlervermeidung — kein manuelles Berechnen der Prozentsätze.

---

## 4.21 Intercompany (IC-Partner)

> **Namensraum:** `Microsoft.Finance.Intercompany`
> **Kern-Tabellen:** `ICPartner.Table.al` (410), `ICInboxTransaction.Table.al` (420), `ICOutboxTransaction.Table.al` (421)

Intercompany ermöglicht buchhalterische Transaktionen zwischen Mandanten eines Konzerns — automatische Gegenbuchung und IC-Bestellungen.

**Beispiel 1 — Management Fees von Mutter an Tochter:**
> Die DE-Holding (IC-Partner-Code: MUTTER) verrechnet monatlich 50.000 € Management Fees an die AT-Tochter (IC-Partner-Code: TOCHTER).
> ➜ Fibu-Buchung: Soll IC-Partner TOCHTER / Haben Erlöse 4800. Der `ICOutboxTransaction` wird automatisch erzeugt.
> **Ergebnis:** AT kann die `ICInboxTransaction` abrufen und buchen — Soll Aufwand 6100 / Haben IC-Partner MUTTER. Vollautomatische Gegenbuchung.

**Beispiel 2 — IC-Bestellung mit automatischer Gegenverkauf:**
> Die DE-Mutter bestellt bei der AT-Tochter 100 Einheiten eines Artikels.
> ➜ IC-Verkaufsauftrag in DE erfassen. Das System erzeugt automatisch einen IC-Einkaufsauftrag in AT.
> **Ergebnis:** Einkäufer und Verkäufer müssen nur EINEN Beleg im System erfassen — die Gegenbuchung entsteht automatisch.

**Beispiel 3 — IC-Transaktionen mit Dimensionen:**
> Der Konzern verwendet die Dimension PROJECT. IC-Transaktionen müssen die Projektnummer transportieren.
> ➜ In der `IC Setup` die Dimension PROJECT als IC-Dimension definieren.
> **Ergebnis:** Bei jeder IC-Transaktion wird die Projektdimension automatisch in den Zielmandanten übertragen.

---

## 4.22 Fibu-Relevanz anderer Module (Querschnitt)

Jeder buchhalterisch relevante Vorgang in BC28 mündet in die Finanzbuchhaltung. Dieser Abschnitt zeigt die Verbindungsstellen.

### Einkauf (→ Kap. 6)

| Quelle | Fibu-Auswirkung |
|---|---|
| Einkaufsbestellung (Purchase Order) | Wareneingang → `Inventory Adjmt. Account`, Rechnungseingang → `Purchase Account` |
| Kreditoren-Rechnung (Purch. Invoice) | Erzeugt `VendLedgerEntry` und `VATEntry` |
| Einkaufsgutschrift (Purch. Cr. Memo) | Bucht die entsprechende negativ-Zeile und optional `Correction`-Marker |
| Kreditoren-Zahlung | Gleicht `VendLedgerEntry` aus, bucht Bankausgang (und ggf. Skonto) |

### Verkauf (→ Kap. 5)

| Quelle | Fibu-Auswirkung |
|---|---|
| Verkaufsauftrag (Sales Order) | Warenausgang → `COGS Account`, Fakturierung → `Sales Account` |
| Verkaufsrechnung (Sales Invoice) | Erzeugt `CustLedgerEntry` und `VATEntry` |
| Verkaufsgutschrift (Sales Cr. Memo) | Negative Rechnung — optional `Correction` für MwSt (`Mark Cr. Memos as Corrections`) |
| Debitoren-Zahlung | Gleicht `CustLedgerEntry` aus, bucht Bankeingang (und ggf. Skonto) |

### Lager & Logistik (→ Kap. 7)

| Quelle | Fibu-Auswirkung |
|---|---|
| Wareneingang (Invt. Receipt) | Soll Bestand / Haben `Inventory Adjmt. Account` |
| Warenausgang (Invt. Shipment) | Soll `COGS Account` / Haben Bestand |
| Inventur (Phys. Inventory) | Differenzbuchung: Soll/Haben `Inventory Adjmt. Account` |
| Umbuchung (Item Reclass.) | Umbuchung zwischen Bestandskonten |
| Montageauftrag (Assembly Order) | Bestandsverbrauch-Komponenten + Zugang Montageartikel |

### Produktion (→ Kap. 8)

| Quelle | Fibu-Auswirkung |
|---|---|
| Fertigungsauftrag (Prod. Order) | Verbrauch Material → FiFo/LiFo-Bewertung, Istmeldung → `WIP Account` |
| Kapazitätsbuchung (Capacity Ledger) | Fertigungslöhne → `Direct Cost Applied Account` |
| Geplante Kosten (Expected Cost) | `WIP Method` → `WIP Account` + `COGS Account` |
| Ist-Kosten abrechnen | Auflösung WIP → `COGS Account` |

### Projekte (→ Kap. 9)

| Quelle | Fibu-Auswirkung |
|---|---|
| Projekt-Buchungszeilen (Job Journal) | Soll Projekt-Aufwandskonto / Haben Gegenkonto |
| Projekt-Abrechnung (Job Invoice) | Soll Debitorenkonto / Haben Projekt-Erlöskonto |
| WIP-Berechnung (Job WIP) | Periodische Aktivierung/Passivierung unfertiger Projekte |

### Service (→ Kap. 10)

| Quelle | Fibu-Auswirkung |
|---|---|
| Serviceauftrag (Service Order) | Materialverbrauch → `COGS`, Arbeitszeit → Erlöse Service |
| Service-Rechnung (Service Invoice) | Wie Verkaufsrechnung, aber mit Service-Preisfindung |
| Garantieabwicklung | Kosten auf Herstellerkonto oder eigenes Garantiekonto |

### Personal (→ Kap. 11)

| Quelle | Fibu-Auswirkung |
|---|---|
| Gehaltsimport (Payroll Import) | Über `Import Payroll` → Fibu-Buchungszeilen je Mitarbeiter |
| Urlaubsrückstellung | Monatliche Abgrenzungsbuchung (Rückstellung + Aufwand) |

### Beispiel — Durchgängige Buchungskette vom Einkauf bis zur Fibu:

> Das Unternehmen bestellt Rohmaterial für die Produktion:
> 1. **Einkaufsbestellung** (Kap. 6): 10.000 kg Stahl für 5,00 €/kg = 50.000 € netto
> 2. **Wareneingang** (Kap. 7): Soll Bestand 50.000 € / Haben `Inventory Adjmt. Account` 50.000 €
> 3. **Rechnungseingang** (Kap. 4): Soll `Purchase Account` 50.000 €, Soll Vorsteuer 9.500 € / Haben Verbindlichkeiten 59.500 €
> 4. **Zahlung** (Kap. 4.16): Bankabgang 59.500 € / Verbindlichkeiten 59.500 € — Ausgleich.
> 5. **Produktion** (Kap. 8): Materialverbrauch 30.000 kg = 150.000 € → `COGS Account` bei Warenausgang.
> 6. **Verkauf** (Kap. 5): Fertigprodukte verkauft → `Sales Account` / Debitor.
> 7. **Zahlungseingang** (Kap. 4.15): Debitor bezahlt → Bankeingang.
> 8. **Monatsabschluss** (Kap. 4.9): `Close Income Statement` — Saldo GuV → Bilanzgewinn.

---

## 4.23 Integrationsereignisse (für Entwickler)

| Ereignis | Parameter | Verwendung |
|---|---|---|
| `OnBeforeCheckRoundingError` | `var ErrorMessage: Boolean` | Zusätzliche Prüfungen vor Rundungsänderung |
| `OnAfterIsPostingAllowed` | `GLSetup, PostingDate, var Result` | Eigene Buchungsdatum-Validierung |
| `OnBeforeFirstAllowedPostingDate` | `GLSetup, var AllowedPostingDate, var IsHandled` | Eigener Algorithmus für erstes Buchungsdatum |
| `OnAfterUpdateDimValueGlobalDimNo` | `ShortcutDimNo, OldDimCode, NewDimCode` | Eigene Logik bei Dimensionsänderung |

**Abonnenten-Beispiel (AL):**
```al
codeunit 50100 "Meine Buchungsprüfung"
{
    [EventSubscriber(ObjectType::Table, Database::"General Ledger Setup",
        OnAfterIsPostingAllowed, '', false, false)]
    local procedure PrüfeBuchungsperiode(
        GeneralLedgerSetup: Record "General Ledger Setup";
        PostingDate: Date;
        var Result: Boolean)
    begin
        if PostingDate < MeineGeschäftsjahresAnfang() then
            Result := false;
    end;
}
```

---

## 4.24 Abhängige Tabellen & Codeunits

| Tabelle/Codeunit | ID | Verwendung |
|---|---|---|
| **General Ledger Setup** | 98 | Singleton-Einrichtung |
| **G/L Account** | 15 | Sachkonten |
| **G/L Entry** | 17 | Gebuchte Sachposten |
| **Gen. Journal Line** | 81 | Fibu-Buchungszeilen |
| **VAT Posting Setup** | 325 | MwSt-Buchungsmatrix |
| **Currency** | 4 | Währungen |
| **Dimension** | 348 | Dimensionen |
| **Dimension Value** | 349 | Dimensionswerte |
| **User Setup Management** | Codeunit | Buchungsdatum-Prüfungen, Datumsformel-Berechnung |
| **Dimension Management** | Codeunit | Dimensionsset-Verwaltung, Popup |
| **Feature Telemetry** | Codeunit | Nutzungsverfolgung |

---

| [← Zurück zur Übersicht]({{ '/index' | relative_url }}) | [Weiter: Vertrieb & Marketing →]({{ '/05-sales-marketing/' | relative_url }}) |
