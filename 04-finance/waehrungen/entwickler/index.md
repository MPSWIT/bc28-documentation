---
title: "Währungen — Entwickler-Referenz"
---
# 4. Währungen — Entwickler-Referenz

<pre>
4. Finanzwesen
 │
 ├── <a href="{{ '/04-finance/' | relative_url }}">Übersicht</a>
 ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}">Fibu-Einrichtung</a>
 ├── <a href="{{ '/04-finance/waehrungen/' | relative_url }}">Währungen &amp; Wechselkurse</a>
 │    ├── <a href="{{ '/04-finance/waehrungen/einrichtung/' | relative_url }}">Einrichtung &amp; Setup</a>
 │    ├── <a href="{{ '/04-finance/waehrungen/verarbeitung/' | relative_url }}">Verarbeitung &amp; Umrechnung</a>
 │    ├── <a href="{{ '/04-finance/waehrungen/anwendung/' | relative_url }}">Anwendung &amp; Beispiele</a>
 │    └─▶ Entwickler-Referenz  ← Sie sind hier
</pre>

---

## Integration Events

### Currency Table (Tabelle 4)

| Event | Parameter | Beschreibung |
|-------|-----------|-------------|
| `OnAfterInitRoundingPrecision` | Rec, xRec, GLSetup | Nach Initialisierung der Rundungspräzision |
| `OnBeforeResolveCurrencySymbol` | Rec, CurrencyCode | Vor Symbol-Auflösung |
| `OnBeforeGetGainLossAccount` | Rec, DtldCVLedgEntryBuf | Vor Ermittlung Gewinn/Verlust-Konto |

### Currency Exchange Rate (Tabelle 330)

| Event | Parameter | Beschreibung |
|-------|-----------|-------------|
| `OnAfterFindCurrency` | CurrencyExchangeRate, CurrencyExchangeRateArray[2], Date, CurrencyCode, CacheNo | Nach Kurs-Suche |
| `OnExchangeAmtFCYToLCYOnAfterSetRelationalExchRateAmount` | CurrencyExchangeRate | Nach Setzen des Adjustment-Relationskurses |
| `OnFindCurrencyOnAfterCalcShouldUseCache` | CurrencyExchangeRate, CacheNo, ShouldUseCache | Cache-Validierung |
| `OnFindCurrencyOnAfterCurrencyExchRate2SetFilters` | CurrencyExchRate2, CurrencyCode, Date | Filter-Manipulation |
| `OnExchangeRateOnAfterSetRelationalExchRateAmount` | CurrencyExchangeRate | Nach Setzen Relationskurs für ExchangeRate() |
| `OnFindCurrency2OnAfterCurrencyExchRate3SetFilters` | CurrencyExchRate3, CurrencyCode, Date | Cross-Rate-Filter |

### Update Currency Exchange Rates (Codeunit 1281)

| Event | Parameter | Beschreibung |
|-------|-----------|-------------|
| `OnBeforeGetCurrencyExchangeData` | CurrExchRateUpdateSetup, ResponseInStream, SourceName, Handled, TempBlobResponse | Vor Abruf Kursdaten (OnPrem) |
| `OnBeforeSyncCurrencyExchangeRatesLoop` | CurrExchRateUpdateSetup | Vor jedem Service-Durchlauf |

### Exch. Rate Adjmt. Process (Codeunit 699)

| Event | Parameter | Beschreibung |
|-------|-----------|-------------|
| `OnAfterRunAdjustment` | ExchRateAdjmtParameters | Nach Anpassung |
| `OnAfterProcessCustomerAdjustment` | TempCustLedgerEntry | Nach Kunden-Anpassung |
| `OnAfterProcessVendorAdjustment` | TempVendorLedgerEntry | Nach Kreditoren-Anpassung |
| `OnAfterProcessEmployeeAdjustment` | TempEmployeeLedgerEntry | Nach Personal-Anpassung |
| `OnAfterShouldAdjustCustLedgEntry` | CustLedgEntry, ShouldAdjust | Filter für Kundenposten |
| `OnAfterShouldAdjustVendLedgEntry` | VendLedgEntry, ShouldAdjust | Filter für Kreditorenposten |
| `OnAfterShouldAdjustEmplLedgEntry` | EmplLedgEntry, ShouldAdjust | Filter für Personalposten |
| `OnBeforePostAdjmt` | ExchRateAdjmtBuffer, TempDimSetEntry | Vor Buchung |
| `OnBeforeGetRealizedGainsAccount` | Currency, AccountNo, IsHandled | Konto für realisierte Gewinne |
| `OnBeforeGetRealizedLossesAccount` | Currency, AccountNo, IsHandled | Konto für realisierte Verluste |
| `OnBeforeGetCustAccountNo` | CustLedgerEntry, AccountNo, IsHandled | Kundenkonto für Buchung |
| `OnBeforeGetVendAccountNo` | VendLedgerEntry, AccountNo, IsHandled | Kreditorenkonto für Buchung |
| `OnBeforeGetEmplAccountNo` | EmplLedgerEntry, AccountNo, IsHandled | Personalkonto für Buchung |
| `OnBeforeGetBankAccountNo` | BankAccount, AccountNo, IsHandled | Bankkonto-Ermittlung |
| `OnBeforeGetLocalCustAccountNo` | CustLedgerEntry, AccountNo, IsHandled | Lokale Kundensachkonten |
| `OnBeforeGetLocalVendAccountNo` | VendLedgerEntry, AccountNo, IsHandled | Lokale Kreditorensachkonten |
| `OnBeforeAdjustGLAccountsAndVATEntries` | ExchRateAdjmtParameters, Currency, GenJnlPostLine | Vor Sachkontenanpassung |

---

## Test-Code — Auszug aus ERMExchRateAdjustment

### Test-Codeunit 134883

```al
codeunit 134883 "ERM Exch. Rate Adjustment"
{
    Subtype = Test;
    TestPermissions = NonRestrictive;

    [Test]
    procedure AdjustExchangeRates()
    begin
        // 1. Setup: Neue Währung mit Wechselkurs anlegen
        Initialize();
        CreateCurrencyWithExchangeRate(CurrencyExchangeRate);
        LibraryERM.SetAddReportingCurrency(CurrencyExchangeRate."Currency Code");

        // 2. Zwei Sachkonten mit unterschiedlichen Anpassungsmethoden
        CreateGLEntryForAccount(GLAccount, CurrencyExchangeRate."Currency Code",
            GLAccount."Exchange Rate Adjustment"::"Adjust Amount", WorkDate());
        CreateGLEntryForAccount(GLAccount2, CurrencyExchangeRate."Currency Code",
            GLAccount2."Exchange Rate Adjustment"::"Adjust Additional-Currency Amount", WorkDate());

        // 3. Neuen Wechselkurs anlegen
        CreateNewExchangeRate(CurrencyExchangeRate);

        // 4. Kursanpassung ausführen
        LibraryERM.RunExchRateAdjustment(
            CurrencyExchangeRate."Currency Code", WorkDate(), WorkDate(),
            ExchangeRateAdjmtTxt, WorkDate(), DocumentNo, true);

        // 5. Prüfen: GL Entry Amount ist angepasst
        VerifyGLEntryAdjustAmount(GLAccount, DocumentNo, CurrencyExchangeRate."Currency Code");
        VerifyGLEntry(DocumentNo, GLAccount2."No.");
    end;

    [Test]
    procedure AdjustExchRateACYStartDate()
    begin
        // Testet: StartDate-Filter für ACY-Anpassung
        // Posten mit Datum < StartDate werden ignoriert
        // Posten mit Datum ≥ StartDate werden angepasst
    end;

    [Test]
    procedure ExchRateAdjmtRegisterSyncWithLedgerEntries()
    begin
        // Szenario 612802:
        // Prüft, dass nach mehreren Anpassungen die Register
        // und Ledger Entries synchron sind
    end;
}
```

### Library — ERM (verwendete Hilfsfunktionen)

```al
// LibraryERM.CreateCurrency()
procedure CreateCurrency(var Currency: Record Currency)
begin
    Currency.Init();
    Currency.Code := RandomCode();
    Currency."ISO Code" := Currency.Code;
    Currency.Description := Currency.Code;
    Currency.Insert(true);
end;

// LibraryERM.CreateExchRate()
procedure CreateExchRate(var CER: Record "Currency Exchange Rate";
    CurrencyCode: Code[10]; StartingDate: Date)
begin
    CER.Init();
    CER."Currency Code" := CurrencyCode;
    CER."Starting Date" := StartingDate;
    CER."Exchange Rate Amount" := 1;
    CER."Relational Exch. Rate Amount" := 0.5;
    CER.Insert(true);
end;

// LibraryERM.RunExchRateAdjustment()
procedure RunExchRateAdjustment(CurrencyCode, StartDate, EndDate,
    PostingDescription, PostingDate, DocumentNo, AdjustGLAccounts)
begin
    ExchRateAdjmtParameters.Init();
    ExchRateAdjmtParameters."Currency Filter" := CurrencyCode;
    ExchRateAdjmtParameters."Start Date" := StartDate;
    ExchRateAdjmtParameters."End Date" := EndDate;
    ExchRateAdjmtParameters."Posting Date" := PostingDate;
    ExchRateAdjmtParameters."Posting Description" := PostingDescription;
    ExchRateAdjmtParameters."Adjust G/L Accounts" := AdjustGLAccounts;
    ExchRateAdjmtParameters."Adjust Bank Accounts" := true;
    ExchRateAdjmtParameters."Adjust Customers" := true;
    ExchRateAdjmtParameters."Adjust Vendors" := true;
    ExchRateAdjmtParameters."Document No." := DocumentNo;
    ExchRateAdjmtParameters.Run();
end;
```

---

## Wichtige Design-Patterns

### 1. Cache-Mechanismus für Performance

```al
// Zwei Cache-Slots in CurrencyExchangeRate:
var
    CurrencyExchRate2: array[2] of Record "Currency Exchange Rate";
    CurrencyExchRate3: array[3] of Record "Currency Exchange Rate";
    CurrencyCode2: array[2] of Code[10];
    Date2: array[2] of Date;

// Validierung:
ShouldUseCache := (CurrencyCode2[CacheNo] = CurrencyCode) and (Date2[CacheNo] = Date);
```

Der Cache vermeidet wiederholte Datenbankzugriffe bei mehreren Umrechnungen mit derselben Währung.

### 2. Adjustment-Kurse parallel zu Standard-Kursen

```al
// Standard-Kurs für tägliche Buchungen:
"Exchange Rate Amount" := 1;
"Relational Exch. Rate Amount" := 0.92;

// Anpassungskurs für Monatsendbewertung:
"Adjustment Exch. Rate Amount" := 1;
"Relational Adjmt Exch Rate Amt" := 0.90;

// Umschaltung über bool-Flag:
UseAdjmtAmounts: Boolean;
```

### 3. Singleton GLSetup-Cache

```al
// In ExchRateAdjmtProcess:
GetGLSetup();
// → GLSetup wird einmalig geladen und via GLSetupRead-Flag gecached

if not GLSetupRead then begin
    // Nur einmal laden
end;
```

### 4. Multistage Adjustment Pipeline

```al
RunAdjustment() → 5 Phasen mit Einzelabschluss:
1. AdjustCurrency()      → Bankkonten, Currency Factor
2. AdjustCustomers()     → HandlePostAdjmt(Customer)
3. AdjustVendors()       → HandlePostAdjmt(Vendor)
4. AdjustEmployees()     → HandlePostAdjmt(Employee)
5. AdjustGLAccountsAndVATEntries() → VAT + GL
```

Jede Phase addiert zur selben Fibu-Buchungszeile und wird am Ende gemeinsam gebucht.

### 5. Cross-Rate über drei Währungen

```al
// Währung A → Währung B über gemeinsame RelationalCurrency C:
// Kurs A×Ct = (ExchRateAmtA × ExchRateAmtC) / (RelExchRateAmtA × RelExchRateAmtC)
// Kurs Ct×B = ... 
// Endkurs = Kurs A×Ct × Kurs Ct×B
```

### 6. ApplnExchangeAmtFCYToFCY — Toleranz beim Zahlungsausgleich

```al
// Unterschied zu ExchangeAmtFCYToFCY:
// - Kein Error bei fehlendem Kurs
// - Rückgabe: ExchRateFound = false
// - Aufrufer entscheidet, ob 0-Betrag akzeptabel ist
```

---

## Eigene Währungserweiterung

```al
codeunit 50200 "My Currency Extension"
{
    [EventSubscriber(ObjectType::Codeunit,
        Codeunit::"Exch. Rate Adjmt. Process",
        'OnAfterRunAdjustment', '', true, true)]
    local procedure MyCustomAdjustment(
        var ExchRateAdjmtParameters: Record "Exch. Rate Adjmt. Parameters")
    begin
        // Eigene Anpassungslogik nach der Standard-Anpassung
        // z.B. Benachrichtigung, eigene Berichte, etc.
    end;

    [EventSubscriber(ObjectType::Table, Database::Currency,
        'OnBeforeResolveCurrencySymbol', '', true, true)]
    local procedure MyCustomSymbols(var Rec: Record Currency; CurrencyCode: Code[10])
    begin
        // Eigene Währungssymbole
        case CurrencyCode of
            'BTC': Rec.Symbol := '₿';
            'CHF': Rec.Symbol := 'Fr.';
        end;
    end;

    [EventSubscriber(ObjectType::Codeunit,
        Codeunit::"Update Currency Exchange Rates",
        'OnBeforeGetCurrencyExchangeData', '', true, true)]
    local procedure MyCustomExchangeRateProvider(
        var CurrExchRateUpdateSetup: Record "Curr. Exch. Rate Update Setup";
        var ResponseInStream: InStream;
        var SourceName: Text;
        var Handled: Boolean;
        var TempBlobResponse: Codeunit "Temp Blob")
    begin
        // Eigener Wechselkurs-Provider
        if CurrExchRateUpdateSetup."Service Provider" <> 'MY_PROVIDER' then
            exit;

        Handled := true;
        // Eigene API-Abruf-Logik
        // TempBlobResponse mit Kursdaten befüllen
    end;
}
```

---

## Performance-Tipps

1. **Cache nutzen**: `FindCurrency()` cached automatisch — wiederholte Aufrufe mit gleicher Währung und Datum sind kostenlos
2. **ExchangeRate() vs ExchangeAmtFCYToLCY()**: `ExchangeRate()` gibt nur den Faktor zurück (leichter). `ExchangeAmtFCYToLCY()` rechnet komplett um
3. **Auf leere CurrencyCode prüfen**: `if CurrencyCode = '' then exit(Amount)` — kein DB-Zugriff für LCY
4. **Adjustment-Kurse parallel führen**: Standard-Kurse für tägliche Buchungen, Adjustment-Kurse nur für Monatsabschluss
5. **UseAdjmtAmounts-Flag**: Setzen Sie es nur, wenn Sie tatsächlich Anpassungskurse brauchen

---

## Enums im Überblick

| Enum | Werte | Verwendung |
|------|-------|-----------|
| `Currency Symbol Position` | After, Before | Positions des Währungssymbols |
| `Fix Exch. Rate Amount Type` | Relational Currency, Currency, Both | Kurs-Fixierung |
| `Exch. Rate Adjmt. Account Type` | Customer, Vendor, Employee, Bank Account | Anpassungskategorie |
| Invoice Rounding Type (in Currency) | Nearest, Up, Down | Rechnungsrundung |
