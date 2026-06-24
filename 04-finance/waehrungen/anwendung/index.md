---
title: "Währungen — Anwendung & Beispiele"
---
# 4. Währungen — Anwendung &amp; Beispiele

<pre>
4. Finanzwesen
 │
 ├── <a href="{{ '/04-finance/' | relative_url }}">Übersicht</a>
 ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}">Fibu-Einrichtung</a>
 ├── <a href="{{ '/04-finance/waehrungen/' | relative_url }}">Währungen &amp; Wechselkurse</a>
 │    ├── <a href="{{ '/04-finance/waehrungen/einrichtung/' | relative_url }}">Einrichtung &amp; Setup</a>
 │    ├── <a href="{{ '/04-finance/waehrungen/verarbeitung/' | relative_url }}">Verarbeitung &amp; Umrechnung</a>
 │    ├─▶ Anwendung &amp; Beispiele  ← Sie sind hier
 │    └── <a href="{{ '/04-finance/waehrungen/entwickler/' | relative_url }}">Entwickler-Referenz</a>
</pre>

---

## Übersicht

In diesem Abschnitt zeigen wir praktische Beispiele für den täglichen Umgang mit Fremdwährungen in Business Central:

1. Verkaufsrechnung in Fremdwährung erstellen und buchen
2. Kaufmännische Belege in Fremdwährung
3. Zahlungsausgleich mit Kursdifferenz
4. Kursanpassung zum Monatsende
5. Zusätzliche Berichtswährung (ACY)

---

## Beispiel 1: Verkaufsrechnung in USD

### Ausgangssituation

- **Mandantenwährung (LCY)**: EUR
- **Debitor**: US-Kunde, Rechnungs- und Zahlungswährung = USD
- **Wechselkurs**: 1 USD = 0,92 EUR (gültig ab 01.06.2024)
- **Rechnungsbetrag**: $10.000,00

### Belegerfassung

1. **Verkaufsrechnung** öffnen
2. Debitor auswählen → Währungscode **USD** wird automatisch gesetzt
3. Artikelpositionen erfassen, Summe = $10.000,00
4. Wechselkurs prüfen (Feld „Currency Factor"): `1,08696` (1 EUR = 1,08696 USD)
5. **Buchen**

### Buchungsbeleg

```
Sachkonto                 Betrag (LCY)    Betrag (USD)    Wechselkurs
─────────────────────────────────────────────────────────────────────
Debitor (1600)             9.200,00 €     $10.000,00      1,08696
   an Erlöse (4000)         9.200,00 €     $10.000,00      1,08696
   an USt (1750)            1.748,00 €     $ 1.900,00      1,08696
```

> **Wichtig:** Die USt in Fremdwährung wird nach dem zum Zeitpunkt der Lieferung geltenden Wechselkurs in EUR umgerechnet (für die USt-Erklärung in EUR).

---

## Beispiel 2: Verkaufsgutschrift in USD

### Situation

Kunde reklamiert 2 von 10 Artikeln. Verkaufsgutschrift über $2.000,00.

```al
// In BC:
SalesCrMemo."Currency Code" := 'USD';
SalesCrMemo."Document Date" := 15.06.2024;

// BC sucht den Wechselkurs zum 15.06.2024:
//  Starting Date ≤ 15.06.2024 → FindLast() → 1 USD = 0,93 EUR (geändert seit 10.06.)
```

**Buchung**:
```
Sachkonto                 Betrag (LCY)    Betrag (USD)
───────────────────────────────────────────────────────
Erlöse (4000)             1.860,00 €      $2.000,00
USt (1750)                  353,40 €      $  380,00
   an Debitor (1600)       2.213,40 €     $2.380,00
```

---

## Beispiel 3: Zahlungsausgleich mit Kursdifferenz

### Situation

Rechnung $10.000,00 gebucht zu 1 USD = 0,92 EUR → 9.200,00 €.
Zahlungseingang $10.000,00, aber Wechselkurs hat sich geändert: 1 USD = 0,95 EUR → 9.500,00 €.

### Zahlungserfassung

1. **Zahlungseingangsbuchblatt** öffnen
2. Debitorenposten-Nr. der offenen Rechnung auswählen
3. Zahlbetrag $10.000,00 eingeben
4. Buchblatt buchen

### Automatische Kursdifferenzberechnung

```al
// BC vergleicht:
// - Gebuchter Betrag der Rechnung: $10.000,00 → 9.200,00 €
// - Zahlungseingang: $10.000,00 → 9.500,00 €
// - Realisierter Kursgewinn: 300,00 €

// Buchung Zahlungsausgleich:
Bank (1200)                9.500,00 €
   an Debitor (1600)        9.200,00 €
   an Kursgewinne real. (4821)   300,00 €
```

### Anwendung (Exch. Rate beim Ausgleich)

```al
procedure ApplnExchangeAmtFCYToFCY(Date, FromCurrencyCode, ToCurrencyCode, Amount, var ExchRateFound)
begin
    // Funktioniert wie ExchangeAmtFCYToFCY, aber:
    // - FindApplnCurrency() statt FindCurrency()
    // - Bei fehlendem Kurs → ExchRateFound = false → Betrag = 0
    // - Kein automatischer Fehler bei fehlendem Kurs
end;
```

---

## Beispiel 4: Kursanpassung zum Monatsende

### Situation

- Offener Debitorenposten $5.000,00 (Restbetrag nach Teilzahlung)
- Gebucht zu 1 USD = 0,92 EUR → 4.600,00 €
- Monatsende: 1 USD = 0,90 EUR (Wechselkurs gesunken)

### Kursanpassung ausführen

1. **Kursanpassung** (Bericht) öffnen
2. Parameter festlegen:
   - Startdatum: 01.06.2024
   - Enddatum: 30.06.2024
   - Buchungsdatum: 30.06.2024
   - Währungsfilter: USD
   - Kunden anpassen: Ja
   - Kreditoren anpassen: Ja (falls relevant)
   - Vorschau: Nein (echte Buchung)
3. **OK** → Fortschrittsdialog zeigt Anpassungen

### Ergebnis

```
Debitorenposten vor Anpassung:
  Originalbetrag: $5.000,00
  Originalbetrag (LCY): 4.600,00 €
  Restbetrag: $5.000,00
  Restbetrag (LCY): 4.600,00 €

Nach Anpassung (1 USD = 0,90 EUR):
  Neuer LCY-Wert: $5.000,00 × 0,90 = 4.500,00 €
  Unrealisierter Verlust: 100,00 €

Automatische Buchung:
  Kursverluste unreal. (4830)  100,00 €
     an Debitor (1600)           100,00 €
```

### Exchange Rate Adjustment Register

Nach der Anpassung sind folgende Einträge vorhanden:

| Tabelle | Inhalt |
|---------|--------|
| Exch. Rate Adjmt. Register | Ein Register pro Postengruppe (Kunde, Kreditor, etc.) |
| Exch. Rate Adjmt. Ledg. Entry | Detail-Einträge zu jedem angepassten Posten |
| Cust. Ledger Entry | Kursanpassungs-Zeile (Entry Type = „Exch. Rate Adjmt.") |

---

## Beispiel 5: Zusätzliche Berichtswährung (ACY)

### Situation

- **LCY**: USD (20 Mio. USD Gesellschaft)
- **ACY**: EUR (Konzernmutter in Deutschland)
- **Anforderung**: Bilanz und GuV sowohl in USD als auch in EUR

### Einrichtung

1. **Fibu-Einrichtung** → „Zusätzliche Berichtswährung" = **EUR**
2. EUR als **Währung** anlegen (ISO-Code: EUR, Symbol: €)
3. Wechselkurse für EUR eintragen:
   ```al
   // 1 EUR = 1,08 USD (aus USD-Sicht)
   CurrencyExchangeRate.Init();
   CurrencyExchangeRate."Currency Code" := 'EUR';
   CurrencyExchangeRate."Starting Date" := 01.06.2024;
   CurrencyExchangeRate."Exchange Rate Amount" := 1;
   CurrencyExchangeRate."Relational Exch. Rate Amount" := 1.08;
   CurrencyExchangeRate.Insert();
   ```

4. Sachkonten für EUR einrichten:
   - EUR → `Realized G/L Gains Account` (separates Konto)
   - EUR → `Realized G/L Losses Account` (separates Konto)
   - Diese Konten müssen **Exchange Rate Adjustment = No Adjustment** haben!

### Sachkonto-Bewertungsmethoden

In der Sachkontokarte kann für jedes Konto einzeln festgelegt werden:

| Einstellung | Bedeutung | Typische Konten |
|-------------|-----------|-----------------|
| **No Adjustment** | Keine Anpassung | EUR-Gewinn-/Verlustkonten |
| **Adjust Amount** | Fremdwährungssaldo (ACY) bewerten → LCY-Differenz | Bilanzkonten |
| **Adjust Additional-Currency Amount** | LCY-Saldo bewerten → ACY-Differenz | Erfolgskonten |

### Kursanpassung für ACY

```al
// Bei "Adjust Amount":
// ACY-Saldo (in EUR) × aktueller Kurs → neuer LCY-Saldo (USD)
// Differenz zu gebuchtem LCY-Saldo = Anpassungsbetrag

// Bei "Adjust Additional-Currency Amount":
// LCY-Saldo (in USD) × aktueller Kurs → neuer ACY-Saldo (EUR)
// Differenz zu gebuchtem ACY-Saldo = Anpassungsbetrag
```

---

## Berichtsanforderung: Währungsübergreifende Auswertungen

### Debitoren-/Kreditoren-Salden nach Währung

Über die **Currency-Tabelle** (FlowFields) können Sie Salden abrufen:

```
Currency-Tabelle mit:
  Customer Filter = '' (alle)
  Global Dimension 1 Filter = '' (alle)
  Date Filter = 30.06.2024
```

Ergebnis:
- Customer Balance: Saldo in Fremdwährung
- Customer Balance (LCY): Saldo in Mandantenwährung

### Beispiele für FlowField-Abfragen

```al
Currency.Get('USD');
Currency.CalcFields("Customer Balance", "Vendor Balance",
    "Customer Balance (LCY)", "Vendor Balance (LCY)");

// USD-Debitorensaldo
Message('Debitoren USD: %1    (LCY: %2)',
    Currency."Customer Balance", Currency."Customer Balance (LCY)");
// → Debitoren USD: 15.000,00    (LCY: 13.500,00)
```

---

## Häufige Fehler und Lösungen

| Problem | Ursache | Lösung |
|---------|---------|--------|
| Belegbetrag in LCY = 0 | Kein Wechselkurs für das Belegdatum | Wechselkurs für das Datum eintragen |
| Kursanpassung läuft nicht | Posting Date außerhalb des erlaubten Bereichs | Datum in der Fibu-Einrichtung prüfen |
| „Nothing to adjust" | Alle Posten bereits ausgeglichen | Nur offene Posten werden angepasst |
| ACY-Fehler bei Kursanpassung | EUR-Gewinnkonten haben Exchange Rate Adjustment ≠ No | Exchange Rate Adjustment auf No setzen |
| Cross-Rate falsch | Fix Exchange Rate Amount steht auf „Both" | Auf „Currency" oder „Relational Currency" setzen |
| Symbol-Duplikat | Mehrere Währungen mit gleichem Symbol ($) | Individuelle Symbole vergeben oder keines |
