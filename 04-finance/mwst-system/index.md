---
title: "MwSt-System: Setup, Klauseln, Satzänderung & Sales Tax"
---
# 4. Finanzwesen — MwSt-System: Setup, Klauseln, Satzänderung & Sales Tax

> 📄 **Zurück zur [Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }})**
> 📋 Das komplette Mehrwertsteuer-System: VAT Posting Setup, MwSt-Abrechnung, Textbausteine (Klauseln), Satzänderungs-Tool und US Sales Tax.

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

**Querverweis:** → [Kap. 4.1 §4.3 MwSt-Felder der Fibu-Einrichtung](fibu-einrichtung#43-mehrwertsteuer) — `Unrealized VAT`, `VAT Reporting Date`, `Control VAT Period` in Tabelle 98

---

## 4.27 MwSt-Klauseln (VAT Clauses)

> **Namensraum:** `Microsoft.Finance.VAT.Clause`
> **Kern-Tabelle:** `VATClause.Table.al` (470)

MwSt-Klauseln sind Textbausteine, die auf Rechnungen gedruckt werden — z.B. „Steuerfreie innergemeinschaftliche Lieferung“ oder „Reverse Charge".

```al
field("VAT Clause Code"; Code[20])
field("Description"; Text[100])    // Textbaustein für den Belegdruck
```

**Beispiel 1 — Innergemeinschaftliche Lieferung korrekt ausweisen:**
> Das Unternehmen liefert nach Frankreich. Auf der Rechnung muss der Hinweis „Steuerfreie innergemeinschaftliche Lieferung (§4 Nr. 1b UStG)" erscheinen.
> ➜ `VAT Clause = IGLIEFERUNG` mit dem Textbaustein. Im VAT Posting Setup wird der Klausel-Code der Kombination `EU/NORMAL` zugewiesen.
> **Ergebnis:** Jede EU-Rechnung druckt automatisch den korrekten Gesetzestext — kein manuelles Einfügen durch den Sachbearbeiter.

**Beispiel 2 — Reverse Charge bei Bauleistungen:**
> Das Unternehmen beauftragt einen Subunternehmer (Bauleistung §13b UStG). Die Rechnungseingangs-Prüfung erfordert den Vermerk „Steuerschuldnerschaft des Leistungsempfängers".
> ➜ `VAT Clause = REV-CHARGE` im `VAT Posting Setup` für die Einkaufskombination.
> **Ergebnis:** Die Eingangsrechnung zeigt den Hinweis, der Buchhalter weiß, dass er die Reverse-Charge-Meldung abgeben muss.

---

## 4.28 MwSt-Satzänderung (VAT Rate Change)

> **Namensraum:** `Microsoft.Finance.VAT.RateChange`
> **Kern-Tabellen:** `VATRateChangeSetup.Table.al` (5500), `VATRateChangeConversion.Table.al` (5501)

Tool zum Umstellen aller MwSt-Einrichtungen bei gesetzlichen Satzänderungen — z.B. von 19% auf 16% und zurück.

**Beispiel 1 — Temporäre MwSt-Senkung 19% → 16%:**
> Die Regierung senkt die MwSt befristet von 19% auf 16% (6 Monate). Alle `VAT Posting Setup`-Einträge mit 19% müssen umgestellt werden.
> ➜ `VAT Rate Change Setup`: Quelle 19%, Ziel 16%, `VAT Bus. Posting Group` = alle. `VAT Product Posting Group` = NORMAL. Ausführen.
> **Ergebnis:** Alle 19%-Einträge in `VAT Posting Setup` werden auf 16% geändert. Die betroffenen `Gen. Product Posting Group`-Konvertierungen werden protokolliert.

**Beispiel 2 — Rückumstellung mit Protokollierung:**
> Nach 6 Monaten steigt die MwSt wieder auf 19%. Der Buchhalter führt die Rate Change rückwärts aus.
> ➜ `VAT Rate Change Setup`: Quelle 16%, Ziel 19%. Der `VATRateChangeLogEntry` protokolliert alte und neue Werte.
> **Ergebnis:** Revisionssichere Dokumentation aller MwSt-Änderungen. Kein manuelles Anpassen von 50+ VAT Posting Setup-Zeilen.

---

## 4.29 Sales Tax (US/CA)

> **Namensraum:** `Microsoft.Finance.SalesTax`
> **Kern-Tabellen:** `TaxSetup.Table.al` (161), `TaxArea.Table.al` (162), `TaxJurisdiction.Table.al` (163)

Sales Tax ist das nordamerikanische Pendant zur MwSt — komplexer durch Bundesstaaten-, County- und City-Steuern (Tax Areas + Tax Jurisdictions).

**Beispiel 1 — Ein US-Unternehmen richtet Sales Tax für Texas ein:**
> Das Unternehmen verkauft in Texas (State Tax 6.25%) und Dallas (City Tax 1%).
> ➜ `Tax Area = TEXAS-DALLAS` mit `Tax Jurisdiction = TEXAS (6.25%)` + `DALLAS (1.0%)`. `Tax Setup` verknüpft die Kombination mit dem korrekten Steuerkonto.
> **Ergebnis:** Bei Rechnung an einen texanischen Kunden berechnet das System automatisch 7.25% Sales Tax — aufgeteilt auf State- und City-Steuer für die Meldung.

**Beispiel 2 — Sales Tax Groups für Kundensteuerbefreiung:**
> Ein Kunde hat eine Tax Exemption (Steuerbefreiungsnummer).
> ➜ `Tax Group = STEUERFREI` mit `Tax % = 0` auf dem Kunden hinterlegen. Im `Tax Setup` wird `Tax Area = TEXAS-DALLAS` mit der Tax Group verknüpft.
> **Ergebnis:** Das System erkennt die Steuerbefreiung und berechnet 0% — trotz Texas-Dallas-Konfiguration.

> ⚠️ **Hinweis:** Sales Tax (`Microsoft.Finance.SalesTax`) ist technisch getrennt von VAT (`Microsoft.Finance.VAT.*`). EU-Unternehmen arbeiten ausschließlich mit VAT.

---

---

| [← Zurück zur Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }}) | [Journale & Debitoren/Kreditoren →]({{ '/04-finance/journale-debitoren-kreditoren/' | relative_url }}) |
