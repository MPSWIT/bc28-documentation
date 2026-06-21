---
title: "Fibu-Relevanz aller Module & Dimensionskorrektur"
---
# 4. Finanzwesen — Fibu-Relevanz aller Module & Dimensionskorrektur

> 📄 **Zurück zur [Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }})**
> 📋 Querschnitt: Wie Einkauf, Verkauf, Lager, Produktion, Service, Projekte und HR in die Fibu buchen. Nachträgliche Dimensionskorrektur auf gebuchten Posten.

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

## 4.25 Dimensionskorrektur (Dimension Correction)

> **Namensraum:** `Microsoft.Finance.Dimension.Correction`
> **Kern-Codeunit:** `DimCorrectionRun.Codeunit.al`

Nachträgliche Korrektur von Dimensionswerten auf **bereits gebuchten Posten** — ohne Storno und Neubuchung.

**Beispiel 1 — Falsche Kostenstelle auf 50 Sachposten korrigieren:**
> Ein Buchhalter hat 50 Rechnungen fälschlich auf Kostenstelle KST-A statt KST-B gebucht. Storno und Neubuchung würden 100 Buchungen bedeuten.
> ➜ `Dimensionskorrektur`-Seite öffnen, Filter auf die 50 Sachposten, neue Kostenstelle = KST-B.
> **Ergebnis:** Alle 50 Posten werden in einem Lauf korrigiert. Die `Dimension Set Entry`-Tabelle wird aktualisiert, ein Korrektur-Log (`DimCorrectionEntryLog`) dokumentiert die Änderung. Keine neue Fibu-Buchung nötig.

**Beispiel 2 — Vor dem Jahresabschluss Dimensionsfehler bereinigen:**
> Der Controller stellt fest, dass drei Monate Projektbuchungen ohne Projekt-Dimension gebucht wurden. Die Abschluss-GuV wäre verfälscht.
> ➜ Dimensionskorrektur mit Datumsfilter 01.04.–30.06., Setzen der fehlenden Projekt-Dimension.
> **Ergebnis:** Die GuV-Auswertung nach Projekten zeigt nun die korrekten Werte. Der Wirtschaftsprüfer sieht die Korrektur im Log.

---

---

| [← Zurück zur Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }}) | [Entwickler-Referenz →]({{ '/04-finance/entwickler/' | relative_url }}) |
