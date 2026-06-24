---
title: "Währungen — Verarbeitung & Umrechnung"
---
# 4. Währungen — Verarbeitung &amp; Umrechnung

<pre>
4. Finanzwesen
 │
 ├── <a href="{{ '/04-finance/' | relative_url }}">Übersicht</a>
 ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}">Fibu-Einrichtung</a>
 ├── <a href="{{ '/04-finance/waehrungen/' | relative_url }}">Währungen &amp; Wechselkurse</a>
 │    ├── <a href="{{ '/04-finance/waehrungen/einrichtung/' | relative_url }}">Einrichtung &amp; Setup</a>
 │    ├─▶ Verarbeitung &amp; Umrechnung  ← Sie sind hier
 │    ├── <a href="{{ '/04-finance/waehrungen/anwendung/' | relative_url }}">Anwendung &amp; Beispiele</a>
 │    └── <a href="{{ '/04-finance/waehrungen/entwickler/' | relative_url }}">Entwickler-Referenz</a>
</pre>

---

## Die Wechselkurs-Tabelle 330 im Detail

### FindCurrency() — Der Suchalgorithmus

```al
procedure FindCurrency(Date: Date; CurrencyCode: Code[10]; CacheNo: Integer)
begin
    // Cache-Prüfung
    ShouldUseCache := (CurrencyCode2[CacheNo] = CurrencyCode) and (Date2[CacheNo] = Date);
    if ShouldUseCache then
        Rec := CurrencyExchRate2[CacheNo]
    else begin
        if Date = 0D then
            Date := WorkDate();

        // Filter: CurrencyCode + StartingDate ≤ Suchdatum
        CurrencyExchRate2[CacheNo].SetRange("Currency Code", CurrencyCode);
        CurrencyExchRate2[CacheNo].SetRange("Starting Date", 0D, Date);

        // Letzten (aktuellsten) Satz holen
        CurrencyExchRate2[CacheNo].FindLast();
        Rec := CurrencyExchRate2[CacheNo];

        // Cache aktualisieren
        CurrencyCode2[CacheNo] := CurrencyCode;
        Date2[CacheNo] := Date;
    end;
end;
```

**Cache-Mechanismus**: Es gibt zwei Cache-Slots (`CurrencyExchRate2[1]` und `[2]`, plus `CurrencyExchRate3[1..3]` für Cross-Rate-Berechnungen). Der Cache wird anhand von CurrencyCode und Date validiert.

---

## Die Umrechnungsfunktionen

### ExchangeAmtFCYToLCY — Fremdwährung → Mandantenwährung

```al
procedure ExchangeAmtFCYToLCY(Date, CurrencyCode, Amount, Factor): Decimal
begin
    if CurrencyCode = '' then exit(Amount);

    FindCurrency(Date, CurrencyCode, 1);

    // Bei Kursanpassung: Adjustment-Kurse verwenden
    if UseAdjmtAmounts then begin
        TestField("Adjustment Exch. Rate Amount");
        TestField("Relational Adjmt Exch Rate Amt");
        "Exchange Rate Amount" := "Adjustment Exch. Rate Amount";
        "Relational Exch. Rate Amount" := "Relational Adjmt Exch Rate Amt";
    end else begin
        TestField("Exchange Rate Amount");
        TestField("Relational Exch. Rate Amount");
    end;
```

#### Fall 1: Keine Relational Currency (einfachster Fall)
```
Bedeutung: 1 [Currency] = X [LCY]

Ergebnis = Amount / Factor
         = Amount / (ExchangeRateAmount / RelExchRateAmount)
```

#### Fall 2: Keine Relational Currency + Fix Exchange Rate = Both
```
Ergebnis = (Amount / ExchangeRateAmount) × RelExchRateAmount
```

#### Fall 3: Relationale Währung vorhanden (z.B. USD→EUR→LCY)
```
Faktor = (ExchangeRateAmt × ExchRateRelCurrency) /
         (RelExchangeRateAmt × RelExchRateRelCurrency)

Ergebnis = Amount / Faktor
```

### ExchangeAmtLCYToFCY — Mandantenwährung → Fremdwährung

```al
procedure ExchangeAmtLCYToFCY(Date, CurrencyCode, Amount, Factor): Decimal
begin
    // Umgekehrte Logik:
    // Fall 1: Amount = Amount × Factor
    // Fall 2: Amount = (Amount / RelExchRateAmt) × ExchRateAmt
    // Fall 3: Dreiecksumrechnung wie oben, aber umgekehrt
end;
```

### ExchangeRate() — Kursfaktor ermitteln

```al
procedure ExchangeRate(Date, CurrencyCode): Decimal
begin
    if CurrencyCode = '' then exit(1);

    FindCurrency(Date, CurrencyCode, 1);

    RelExchangeRateAmt := "Relational Exch. Rate Amount";
    ExchangeRateAmt := "Exchange Rate Amount";

    if "Relational Currency Code" = '' then
        CurrencyFactor := ExchangeRateAmt / RelExchangeRateAmt
    else begin
        // Dreiecksumrechnung
        FindCurrency(Date, RelCurrencyCode, 2);
        CurrencyFactor :=
          (ExchangeRateAmt × "Exchange Rate Amount"[2]) /
          (RelExchangeRateAmt × "Relational Exch. Rate Amount"[2]);
    end;

    exit(CurrencyFactor);
end;
```

### ExchangeAmtFCYToFCY — Zwischen zwei Fremdwährungen umrechnen

Diese Funktion rechnet direkt zwischen zwei Fremdwährungen um, ohne den Umweg über LCY:

```al
procedure ExchangeAmtFCYToFCY(Date, FromCurrencyCode, ToCurrencyCode, Amount): Decimal
begin
    // Kurse für beide Währungen laden
    FindCurrency2(Date, FromCurrencyCode, 1);
    FindCurrency2(Date, ToCurrencyCode, 2);

    // Fall: FromCurrency.RelationalCurrency == ToCurrency
    if CurrencyExchRate3[1]."Currency Code" = CurrencyExchRate3[2]."Relational Currency Code" then
        exit((Amount / CurrencyExchRate3[2]."Relational Exch. Rate Amount") *
              CurrencyExchRate3[2]."Exchange Rate Amount");

    // Fall: ToCurrency.RelationalCurrency == FromCurrency
    if CurrencyExchRate3[1]."Relational Currency Code" = CurrencyExchRate3[2]."Currency Code" then
        exit((Amount / CurrencyExchRate3[1]."Exchange Rate Amount") *
              CurrencyExchRate3[1]."Relational Exch. Rate Amount");

    // Fall: Beide haben dieselbe Relational Currency → Umweg über diese
    if CurrencyExchRate3[1]."Relational Currency Code" = CurrencyExchRate3[2]."Relational Currency Code" then
        // Amount über gemeinsame Relational Currency umrechnen
        ...
end;
```

---

## Fix Exchange Rate Amount — Detaillierte Berechnung

Das Enum `Fix Exch. Rate Amount Type` steuert, welcher Teil des Wechselkurses bei Cross-Rate-Berechnungen konstant bleibt:

### Fix Exchange Rate = Relational Currency

```al
case FixExchangeRateAmt of
    "Fix Exchange Rate Amount"::"Relational Currency":
        ExchangeRateAmt :=
          (RelExchangeRateAmt × "Relational Exch. Rate Amount"[2]) /
          ("Exchange Rate Amount"[2] × Factor);
```

**Bedeutung**: Die Bezugswährung (z.B. EUR) bleibt in ihrem Kurs zur LCY konstant. Die andere Währung (USD) wird so angepasst, dass die Relation USD→EUR erhalten bleibt.

### Fix Exchange Rate = Currency

```al
case FixExchangeRateAmt of
    "Fix Exchange Rate Amount"::Currency:
        RelExchangeRateAmt :=
          ((Factor × ExchangeRateAmt × "Exchange Rate Amount"[2]) /
           "Relational Exch. Rate Amount"[2]);
```

**Bedeutung**: Die Währung selbst bleibt in ihrem Kurs zur LCY konstant. Die Bezugswährung wird angepasst.

### Fix Exchange Rate = Both

```al
case FixExchangeRateAmt of
    "Fix Exchange Rate Amount"::Both:
        case "Fix Exchange Rate Amount" of
            "Fix Exchange Rate Amount"::"Relational Currency":
                // Bezugswährung wird neu berechnet
                "Exchange Rate Amount"[2] :=
                  (Factor × RelExchangeRateAmt × "Relational Exch. Rate Amount"[2]) /
                  ExchangeRateAmt;
            "Fix Exchange Rate Amount"::Currency:
                // Währung wird neu berechnet
                "Relational Exch. Rate Amount"[2] :=
                  ((Factor × ExchangeRateAmt × "Exchange Rate Amount"[2]) /
                   RelExchangeRateAmt);
        end;
```

---

## Die Kursanpassung (Exchange Rate Adjustment)

Die Kursanpassung (Codeunit **699** `Exch. Rate Adjmt. Process`) bewertet offene Fremdwährungsposten zum Stichtag neu und erzeugt die entsprechenden Buchungen.

### Anpassungskategorien

```
RunAdjustment()
 ├── AdjustCurrency()
 │    └── Bankkonten (BankAcc)
 │
 ├── AdjustCustomers()
 │    └── Debitorenposten (Detailed Cust. Ledg. Entry)
 │
 ├── AdjustVendors()
 │    └── Kreditorenposten (Detailed Vendor Ledg. Entry)
 │
 ├── AdjustEmployees()
 │    └── Personalposten
 │
 └── AdjustGLAccountsAndVATEntries()
      ├── AdjustVAT()  (MwSt-Posten)
      └── AdjustGLAccounts()  (Sachkonten)
```

### Anpassungsberechnung am Beispiel Debitoren

```al
// Für jeden offenen Debitorenposten:
// 1. Ursprünglicher Fremdwährungsbetrag (OriginalAmtFCY)
// 2. Gebuchter LCY-Betrag (OriginalAmtLCY)
// 3. Neuer LCY-Betrag mit aktuellem Anpassungskurs:
//    NewAmtLCY = ExchangeAmtFCYToLCYAdjmt(PostingDate, CurrencyCode, OriginalAmtFCY)
// 4. Differenz = NewAmtLCY - OriginalAmtLCY
// 5. Wenn Differenz ≠ 0 → Kursdifferenz buchen
```

### Geschäftslogik für Debitorenposten

```al
local procedure AdjustCustomerLedgerEntry(Customer, CustLedgerEntry, PostingDate, IsDetailedEntry)
begin
    // 1. Währung laden
    Currency.Get(CustLedgerEntry."Currency Code");

    // 2. Anpassungsbetrag berechnen
    AdjmtBase := CustLedgerEntry."Original Amount" - CustLedgerEntry."Remaining Amount";
    AdjmtBaseLCY := CustLedgerEntry."Original Amt. (LCY)" - CustLedgerEntry."Remaining Amt. (LCY)";

    // 3. Neu bewerten mit Adjustment-Kurs
    AdjmtAmount :=
        Round(
            ExchangeAmtFCYToLCYAdjmt(PostingDate, CurrencyCode,
                AdjmtBase, Currency."Currency Factor")) -
        AdjmtBaseLCY;

    // 4. Bei Differenz: Buchung erzeugen
    if AdjmtAmount <> 0 then begin
        ExchRateAdjmtBufferUpdate(
            CurrencyCode, CustLedgerEntry."Customer Posting Group",
            CustLedgerEntry."Currency Code", AdjmtBase, AdjmtBaseLCY,
            AdjmtAmount, ...);

        // 5. Gegenkonto: Realized Gains/Losses
        if AdjmtAmount > 0 then
            PostAdjmt(GetRealizedGainsAccount(Currency), -AdjmtAmount, ...)
        else
            PostAdjmt(GetRealizedLossesAccount(Currency), -AdjmtAmount, ...);
    end;
end;
```

### Progress Bar / Dialog

Während der Kursanpassung zeigt BC einen Fortschrittsdialog:

```
Adjusting exchange rates...
Bank Account    @1@@@@@@@@@@@@@
Customer        @2@@@@@@@@@@@@@ \
Vendor          @3@@@@@@@@@@@@@ \
Employee          @5@@@@@@@@@@@@@\
Adjustment      #4#############
```

Jede Kategorie zeigt einen eigenen Fortschrittsbalken (0-10000). `@1`-`@5` sind Progress-Indikatoren, `#4` zeigt den kumulierten Anpassungsbetrag.

---

## G/L Account Exchange Rate Adjustment Types

In der Sachkontokarte kann für jedes Konto eine Wechselkursanpassungsmethode festgelegt werden:

| Wert | Bedeutung |
|------|-----------|
| **No Adjustment** | Keine Anpassung (Standard für nicht-fremdwährungsrelevante Konten) |
| **Adjust Amount** | Betrag anpassen: Original-Fremdwährungsbetrag × neuer Kurs → LCY-Differenz |
| **Adjust Additional-Currency Amount** | ACY-Betrag anpassen: Original-LCY-Betrag × neuer Kurs → ACY-Differenz |

### Berechnung bei Sachkonten

```al
case GLAccount."Exchange Rate Adjustment" of
    "Exchange Rate Adjustment"::"Adjust Amount":
        // Differenz in LCY:
        AdjmtAmount :=
            Round(ExchangeAmtFCYToLCYAdjmt(
                PostingDate, AdditionalCurrencyCode,
                GLAccount."Additional-Currency Net Change",
                AddCurrCurrencyFactor)) -
            GLAccount."Net Change";

    "Exchange Rate Adjustment"::"Adjust Additional-Currency Amount":
        // Differenz in ACY:
        AdjmtAmount :=
            Round(ExchangeAmtLCYToFCY(
                PostingDate, AdditionalCurrencyCode,
                GLAccount."Net Change",
                AddCurrCurrencyFactor)) -
            GLAccount."Additional-Currency Net Change";
end;
```

---

## Zusammenspiel: Rechnungserfassung → Buchung → Anpassung

```
1. Verkaufsrechnung in USD (Wechselkurs: 1 USD = 0,92 EUR)
   ├── Rechnungsbetrag: $1.000,00
   ├── LCY-Betrag: 920,00 €
   ├── Buchung: Debitorenkonto (920,00) / Erlöskonto (920,00)
   └── Original Amount: $1.000,00

2. Zahlungseingang in USD (Wechselkurs: 1 USD = 0,95 EUR)
   ├── Zahlbetrag: $1.000,00
   ├── LCY-Betrag: 950,00 €
   ├── Buchung: Bank (950,00) / Debitorenkonto (950,00)
   └── Realisierter Kursgewinn: 30,00 €

3. Offener Posten (50% bezahlt, 50% offen)
   └── Saldo: $500,00 / 460,00 €

4. Kursanpassung zum Monatsende (1 USD = 0,90 EUR)
   ├── Neuer LCY-Wert: $500,00 × 0,90 = 450,00 €
   ├── Unrealisierter Verlust: 460,00 - 450,00 = 10,00 €
   └── Buchung: Kursverlust (10,00) / Debitorenkonto (10,00)
```
