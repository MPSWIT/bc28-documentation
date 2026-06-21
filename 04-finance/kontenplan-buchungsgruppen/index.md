---
title: "Kontenplan, Buchungsgruppen & Kontierungsschlüssel"
---
# 4. Finanzwesen — Kontenplan, Buchungsgruppen & Kontierungsschlüssel

> 📄 **Zurück zur [Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }})**
> 📋 Der Kontenplan als Fundament, die Buchungsmatrix (Gen. Posting Setup), und automatische Kostenverteilung mit Kontierungsschlüsseln.

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

## 4.24 Kontierungsschlüssel (Allocation Accounts)

> **Namensraum:** `Microsoft.Finance.AllocationAccount`
> **Kern-Tabellen:** `AllocationAccount.Table.al` (1300), `AllocationLine.Table.al` (1301)

Kontierungsschlüssel verteilen Buchungsbeträge automatisch auf mehrere Konten oder Dimensionen — z.B. Mietkosten nach Quadratmetern auf Abteilungen.

```al
field("Distribution %"; Decimal)         // Verteilungsschlüssel in %
field("Account No."; Code[20])           // Zielkonto
field("Dimension Value Code"; Code[20])  // Dimensionswert
```

**Beispiel 1 — Gebäudekosten nach Fläche verteilen:**
> Das Unternehmen hat 3 Abteilungen (Vertrieb 200m², Produktion 500m², Verwaltung 300m²). Die Monatsmiete von 10.000 € soll automatisch verteilt werden.
> ➜ `Allocation Account` anlegen: Vertrieb 20%, Produktion 50%, Verwaltung 30%. Der Kontierungsschlüssel wird in der Kreditoren-Fibu-Zeile ausgewählt.
> **Ergebnis:** Aus einer 10.000 €-Kreditorenrechnung werden automatisch drei Fibu-Zeilen: Vertrieb 2.000 €, Produktion 5.000 €, Verwaltung 3.000 € — je auf das richtige Kostenstellenkonto.

**Beispiel 2 — Werbekosten auf Kampagnen-Dimensionen splitten:**
> Eine Marketing-Agentur-Rechnung über 30.000 € soll auf 3 Kampagnen (K1 40%, K2 35%, K3 25%) verteilt werden.
> ➜ `Allocation Account` mit Dimensions-Zielwerten K1/K2/K3.
> **Ergebnis:** Die Buchungsvorschau zeigt 3 Fibu-Zeilen — jede mit der korrekten Kampagnen-Dimension. Der Buchhalter muss nicht manuell splitten.

---

---

| [← Zurück zur Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }}) | [MwSt-System →]({{ '/04-finance/mwst-system/' | relative_url }}) |
