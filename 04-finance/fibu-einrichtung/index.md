---
title: "Fibu-Einrichtung (Tabelle 98)"
---
# 4. Finanzwesen — Fibu-Einrichtung (Tabelle 98)

> 📄 **Zurück zur [Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }})**
> 📋 Die 22 Felder der General Ledger Setup-Tabelle: Buchungszeiträume, Dimensionen, MwSt-Steuerung, Zahlungstoleranzen, Rundung, Berichtswährung, Aufgabenwarteschlange.

<pre>
4. Finanzwesen
 │
 ├─▶ Fibu-Einrichtung (Tab. 98)  ← Sie sind hier
 ├── <a href="{{ '/04-finance/kontenplan-buchungsgruppen/' | relative_url }}">Kontenplan &amp; Buchungsgruppen</a>
 ├── <a href="{{ '/04-finance/mwst-system/' | relative_url }}">MwSt-System</a>
 ├── <a href="{{ '/04-finance/journale-debitoren-kreditoren/' | relative_url }}">Journale, Debitoren/Kreditoren</a>
 ├── <a href="{{ '/04-finance/bank-anlagen-waehrung/' | relative_url }}">Bank, Anlagen &amp; Währung</a>
 ├── <a href="{{ '/04-finance/berichte-analyse-budget/' | relative_url }}">Berichte, Budget &amp; Analyse</a>
 ├── <a href="{{ '/04-finance/konsolidierung-abgrenzung-ic/' | relative_url }}">Konsolidierung, Abgrenzungen &amp; IC</a>
 ├── <a href="{{ '/04-finance/querschnitt/' | relative_url }}">Querschnitt — Fibu-Relevanz aller Module</a>
 └── <a href="{{ '/04-finance/entwickler/' | relative_url }}">Entwickler-Referenz</a>
</pre>

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

## 4.9 Aktionen der Fibu-Einrichtungsseite

Die Seite **Fibu Einrichtung** (Page 118) bietet zwei Aktionsgruppen — **Verarbeitung** und **Navigation** — sowie die hervorgehobenen Aktionen in der Aktionsleiste. Jede Aktion stößt einen komplexen Prozess an, der im Folgenden detailliert beschrieben wird.

---

### 🔄 Aktion: Globale Dimensionscodes ändern

> **Ort:** Verarbeitung → Funktionen → **Globale Dimensionscodes ändern**
> **Öffnet:** Page 577 "Change Global Dimensions"
> **Codeunit:** 483 "Change Global Dimensions"
> **Berechtigung:** `TableData Dimension = M` (Ändern)

Die Aktion öffnet eine eigene Seite, die den Austausch einer oder beider globaler Dimensionen ermöglicht (z. B. Wechsel von „Abteilung/Projekt" auf „Kostenstelle/Division"). **Die globalen Dimensionscodes sind auf der Fibu-Einrichtungsseite gesperrt** (`Editable = false`) — diese Aktion ist der einzige Weg, sie zu ändern.

**Ablauf Schritt für Schritt:**

1. **Seite öffnet sich** — sie zeigt die aktuellen Global Dimension 1/2 und lässt neue Codes auswählen. Ein Stilwechsel („Ambiguous") markiert visuell, welche Codes geändert werden.

2. **Verarbeitungsmodus wählen:**
   - **Sequenziell (Start):** Läuft in der aktuellen Session. Alle betroffenen Tabellen werden gesperrt — andere Benutzer können währenddessen nicht auf diese Tabellen zugreifen. Geeignet für kleine Datenmengen.
   - **Parallel (Prepare → Start):** Zweistufiger Prozess:
     - **Prepare:** Füllt das Log-Infobox mit der Liste aller betroffenen Tabellen. Der Benutzer sieht den Umfang der Änderung.
     - **Start:** Startet einen Hintergrundjob über die Aufgabenwarteschlange. **Wichtig:** Vor dem Start muss der Benutzer sich ab- und wieder anmelden, damit die Session keine Sperren auf den zu ändernden Tabellen hält.
   - **Reset:** Bricht eine vorbereitete Änderung ab, bevor sie gestartet wurde (`ResetState()`).

3. **Was wird aktualisiert?** Der Codeunit 483 durchläuft **alle** folgenden Tabellen und aktualisiert die globalen Dimensionscodes:
   - **Finanzbuchhaltung:** G/L Entry, VAT Entry, Detailed Cust./Vendor Ledg. Entry
   - **Debitoren/Kreditoren:** Cust. Ledger Entry, Vendor Ledger Entry
   - **Verkauf (alle gebuchten Belege):** Sales Shipment Header/Line, Sales Invoice Header/Line, Sales Cr.Memo Header/Line
   - **Einkauf (alle gebuchten Belege):** Purch. Rcpt. Header/Line, Purch. Inv. Header/Line, Purch. Cr. Memo Hdr./Line
   - **Lager:** Item Ledger Entry, Phys. Inventory Ledger Entry, Invt. Receipt Header/Line, Invt. Shipment Header/Line, Posted Assembly Header/Line
   - **Anlagen:** FA Ledger Entry, Maintenance Ledger Entry, Ins. Coverage Ledger Entry
   - **Bank:** Bank Account Ledger Entry
   - **Projekte/Ressourcen:** Job Ledger Entry, Res. Ledger Entry, Job WIP G/L Entry
   - **Personal:** Employee Ledger Entry, Detailed Employee Ledger Entry
   - **Mahnwesen/Zinsen:** Issued Reminder Header/Line, Reminder/Fin. Charge Entry, Issued Fin. Charge Memo Header/Line
   - **Fertigung (vor BC28):** Production Order, Prod. Order Line/Component/Routing Line usw.

4. **Fortschrittsverfolgung:** Über das eingebettete `Change Global Dim. Log Entries`-Part kann der Status jeder Tabelle verfolgt werden.

5. **Nach Abschluss:** Die globalen Dimensionscodes in der Fibu-Einrichtung sind aktualisiert. Shortcut Dim 1/2 werden automatisch mitgezogen (siehe `OnValidate` der Tabelle 98).

**Beispiel — Ein Filialunternehmen reorganisiert seine Dimensionen:**
> Das Unternehmen hat bisher nach Abteilung und Projekt analysiert. Nach einer Umstrukturierung soll nun nach Kostenstelle und Region ausgewertet werden. 150.000 Sachposten, 80.000 Debitorenposten und 50.000 Kreditorenposten sind betroffen.
> ➜ Administrator öffnet „Globale Dimensionscodes ändern", wählt „Parallel", klickt „Prepare" → sieht die Liste der 30 betroffenen Tabellen → meldet sich ab und wieder an → klickt „Start".
> **Ergebnis:** Der Hintergrundjob läuft ca. 20 Minuten. Alle historischen Buchungen behalten ihre Dimensionswerte — die neuen Codes gelten für zukünftige Buchungen.

---

### 💰 Aktion: Zahlungstoleranz ändern

> **Ort:** Verarbeitung → Funktionen → **Zahlungstoleranz ändern**
> **Öffnet:** Report 34 "Change Payment Tolerance" (Bericht mit Request Page)
> **Berechtigung:** `TableData Currency = rm`, `TableData Cust./Vendor Ledger Entry = rm`, `TableData General Ledger Setup = rm`

Die Aktion öffnet einen Verarbeitungsbericht, der Zahlungstoleranz-Prozentsatz und -Maximalbetrag auf einmal ändern kann — entweder für alle Währungen gleichzeitig oder für eine einzelne Währung.

**Ablauf Schritt für Schritt:**

1. **Request Page öffnet sich** mit folgenden Optionen:
   - **Alle Währungen:** Wenn aktiviert, wird derselbe Toleranzwert auf LCY und ALLE Fremdwährungen angewendet. Die Felder für einzelne Währung werden deaktiviert.
   - **Währungscode:** Eine spezifische Währung auswählen (Lookup auf die Währungstabelle). Leer = Mandantenwährung (LCY).
   - **Zahlungstoleranz %:** Neuer Prozentsatz (0–100, 5 Dezimalstellen).
   - **Max. Zahlungstoleranzbetrag:** Neuer Maximalbetrag (Dezimalstellen gemäß `Amount Decimal Places` der gewählten Währung).

2. **Beim Ausführen (OnPostReport):**
   - Fall **Alle Währungen:** Iteriert über ALLE Währungsdatensätze (`Currency.Find('-')`) und setzt `Payment Tolerance %` und `Max. Payment Tolerance Amount` auf die neuen Werte. Anschließend werden die Werte auch in der **General Ledger Setup**-Tabelle gesetzt (für LCY).
   - Fall **Einzelne Währung:** Aktualisiert nur den gewählten Währungsdatensatz (oder GL Setup, wenn LCY).
   - Alle Beträge werden auf `Amount Rounding Precision` der jeweiligen Währung gerundet.

3. **Bestätigungsdialog:** *„Möchten Sie alle offenen Posten für jeden Debitor und Kreditor ändern, der nicht gesperrt ist?"*
   - **Wenn Ja:** Der Report iteriert durch ALLE Kunden (Tabelle Customer) und ALLE Kreditoren (Tabelle Vendor).
   - Für jeden Kunden/Kreditor, bei dem **Block Payment Tolerance = Nein**, werden ALLE offenen Posten (`Open = true`) vom Typ Rechnung/Gutschrift in der jeweiligen Währung geladen.
   - **Neuberechnung pro Posten:**
     ```al
     Max. Payment Tolerance = Round(NewPaymentTolerancePct × Remaining Amount / 100, AmountRoundingPrecision)
     ```
     Wenn der errechnete Wert 0 ist und ein Maximalbetrag gesetzt wurde, ODER wenn der errechnete Wert den Maximalbetrag überschreitet, wird der Maximalbetrag verwendet.
   - Wenn der verbleibende Betrag kleiner als die Toleranz ist, wird die Toleranz auf den verbleibenden Betrag begrenzt.
   - Integration-Events `OnChangeCustLedgEntriesOnBeforeModifyCustLedgEntry` und `OnChangeVendLedgEntryOnBeforeModifyVendLedgEntry` ermöglichen Erweiterungen.

**Beispiel — Globales Toleranz-Update nach Betriebsprüfung:**
> Der Steuerberater empfiehlt, die Zahlungstoleranz von 2% auf 1% zu senken und den Maximalbetrag von 100 € auf 50 € zu reduzieren. Das Unternehmen hat 3 Fremdwährungskonten (USD, CHF, GBP) und 2.300 offene Debitorenposten.
> ➜ Buchhalter öffnet „Zahlungstoleranz ändern" → aktiviert „Alle Währungen" → setzt 1% und 50 € → führt aus → bestätigt die Aktualisierung der offenen Posten.
> **Ergebnis:** Die Währungstabelle wird aktualisiert (EUR, USD, CHF, GBP), die GL Setup-Tabelle wird aktualisiert. Anschließend werden 2.300 Debitorenposten und 800 Kreditorenposten neu berechnet. Der Vorgang dauert je nach Datenmenge 2–5 Minuten.

---

### 🧭 Navigationsaktionen

Die folgenden Aktionen öffnen verwandte Einrichtungsseiten und dienen der schnellen Navigation:

| Aktion | Öffnet | Beschreibung |
|--------|--------|-------------|
| **Buchungsperioden** | Page "Accounting Periods" | Geschäftsjahresperioden einrichten (z. B. 12 Monatsperioden). |
| **Dimensionen** | Page "Dimensions" | Alle Dimensionen verwalten (Abteilung, Projekt, Kostenstelle usw.). |
| **Benutzereinrichtung** | Page "User Setup" | Buchungsberechtigungen pro Benutzer einschränken. |
| **Cashflow Einrichtung** | Page "Cash Flow Setup" | Konten für Cashflow-Prognosen festlegen. |
| **Bankexport/-import Einr.** | Page "Bank Export/Import Setup" | Formate für Bankauszugsimport und Zahlungsexport. |
| **Allgemeine Buchungsmatrix** | Page "General Posting Setup" | Kombinationen aus Gen. Geschäftsbuchungsgruppe × Gen. Produktbuchungsgruppe auf Sachkonten abbilden. |
| **Gen. Geschäftsbuchungsgruppe** | Page "Gen. Business Posting Groups" | Handelsbezogene Buchungsgruppen (Inland, EU, Export usw.). |
| **Gen. Produktbuchungsgruppe** | Page "Gen. Product Posting Groups" | Artikelbezogene Buchungsgruppen (Einzelhandel, Großhandel usw.). |
| **MwSt.-Buchungsmatrix** | Page "VAT Posting Setup" | Kombinationen aus MwSt.-Geschäftsbuchungsgruppe × MwSt.-Produktbuchungsgruppe auf MwSt-Konten abbilden. |
| **MwSt. Geschäftsbuchungsgruppe** | Page "VAT Business Posting Groups" | Handelstypen für MwSt (Inland, EU, Drittland usw.). |
| **MwSt. Produktbuchungsgruppe** | Page "VAT Product Posting Groups" | Artikelspezifische MwSt-Gruppen (Normal, Ermäßigt usw.). |
| **MwSt.-Abrechnung einrichten** | Page "VAT Report Setup" | Nummernserien und Optionen für die UStVA. |
| **Bankbuchungsgruppe** | Page "Bank Account Posting Groups" | Bankkontobuchungsgruppen für Zahlungsein- und -ausgänge. |
| **Allgemeine Buch.-Blattvorlagen** | Page "General Journal Templates" | Vorlagen und Batch-Namen für Fibu-Buch.-Blätter. |
| **MwSt.-Abrechnungsvorlagen** | Page "VAT Statement Templates" | Vorlagen für MwSt-Abrechnungen. |

---

### 📌 Hervorgehobene Aktionen (Aktionsleiste)

Die Aktionsleiste gruppiert die wichtigsten Aktionen in thematische Kategorien:

| Kategorie | Aktionen |
|-----------|----------|
| **Prozess** | Zahlungstoleranz ändern, Globale Dimensionscodes ändern |
| **Buchung** | Allgemeine Buchungsmatrix, Gen. Geschäftsbuchungsgruppe, Gen. Produktbuchungsgruppe |
| **Allgemein** | Buchungsperioden, Dimensionen, Benutzereinrichtung, Cashflow Einrichtung |
| **MwSt.** | MwSt.-Abrechnungsvorlagen, MwSt.-Buchungsmatrix, MwSt. Geschäftsbuchungsgruppe, MwSt. Produktbuchungsgruppe, MwSt.-Abrechnung einrichten |
| **Bank** | Bankexport/-import Einr., Bankbuchungsgruppe |
| **Buch.-Blattvorlagen** | Allgemeine Buch.-Blattvorlagen |

---


| [← Zurück zur Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }}) | [Kontenplan & Buchungsgruppen →]({{ '/04-finance/kontenplan-buchungsgruppen/' | relative_url }}) |
