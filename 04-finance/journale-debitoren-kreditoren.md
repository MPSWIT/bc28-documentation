---
title: "Fibu-Journale, Debitoren/Kreditoren & Stornobuchungen"
---
# 4. Finanzwesen — Fibu-Journale, Debitoren/Kreditoren & Stornobuchungen

> 📄 **Zurück zur [Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }})**
> 📋 Tägliche Buchungsarbeit: Fibu-Journale, OP-Verwaltung (Debitoren/Kreditoren), Zahlungsausgleich und revisionssichere Stornobuchungen.

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
→ [Kap. 4.14 Intercompany](konsolidierung-abgrenzung-ic#421-intercompany-ic-partner)
→ [Kap. 4.7 Aufgabenwarteschlange](fibu-einrichtung#47-aufgabenwarteschlange--hintergrundbuchung) — `Post with Job Queue`

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

## 4.26 Stornobuchungen (Reversal)

> **Namensraum:** `Microsoft.Finance.GeneralLedger.Reversal`
> **Kern-Tabellen:** `ReversalEntry.Table.al` (1850)

Stornobuchungen machen fehlerhafte Buchungen rückgängig, ohne die Originalbuchung zu verändern — zwingend für Revisionssicherheit.

```al
field("Reversal Entry No."; Integer)
field("Reversed by Entry No."; Integer)   // Verweist auf den Originalposten
```

**Beispiel 1 — Eine falsch gebuchte Monatsmiete stornieren:**
> Der Buchhalter hat die Miete von 2.000 € versehentlich auf Konto 4211 (Lager-Miete) statt 4210 (Büro-Miete) gebucht. Die Originalbuchung vom 01.06. soll erhalten bleiben.
> ➜ Auf dem gebuchten Sachposten → Funktion „Transaktion stornieren" → Stornobuchung erzeugen.
> **Ergebnis:** System erzeugt eine Gegenbuchung (Haben an 4211 2.000 € / Soll an 4210 2.000 €) mit `Reversed by Entry No.` = Original-Posten-Nr. Der Prüfpfad bleibt erhalten.

**Beispiel 2 — Massenstorno bei fehlerhaftem Import:**
> Ein Import hat 500 Buchungszeilen mit falschem Buchungsdatum (01.07. statt 01.06.) erzeugt. Der Fehler fällt nach dem Buchen auf.
> ➜ Filter auf die 500 gebuchten Zeilen → `Reverse`-Funktion mit `Reverse Posting Date = 01.06.`.
> **Ergebnis:** 500 Stornobuchungen + 500 korrekte Neubuchungen in einem Lauf.

---

---

| [← Zurück zur Finanzwesen-Übersicht]({{ '/04-finance/' | relative_url }}) | [Bank, Anlagen & Währung →](bank-anlagen-waehrung) |
