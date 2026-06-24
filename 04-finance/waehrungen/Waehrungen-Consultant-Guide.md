# Währungen & Wechselkurse in Business Central 28

## Ein Leitfaden für Consultants und Anwender

---

## 1. Einführung: Währungen in Business Central

Business Central unterstützt standardmäßig die Arbeit mit **unbegrenzt vielen Fremdwährungen** — vom
einfachen USD-Verkauf bis zur mandantenübergreifenden Konsolidierung mit
Zusatzberichtswährung.

Jeder Beleg, jede Buchung und jeder Posten kann in einer beliebigen Währung erfasst werden.
BC übernimmt automatisch:

- Die **Umrechnung** in die Mandantenwährung (LCY)
- Die **Verwaltung** täglicher Wechselkurse
- Die **periodische Kursanpassung** offener Posten
- Die **Auswertung** währungsübergreifender Salden

---

## 2. Geschäftlicher Nutzen und Relevanz

### 2.1 Wann brauchen Sie Fremdwährungen?

- Sie kaufen oder verkaufen Waren/Dienstleistungen in einer anderen Währung
- Sie haben Kunden oder Lieferanten im Ausland
- Sie führen Bankkonten in Fremdwährung
- Sie müssen Konzernabschlüsse in einer einheitlichen Währung erstellen

### 2.2 Rechtliche und kaufmännische Pflichten

Bei Fremdwährungsgeschäften müssen Sie:

1. **Wechselkurse dokumentieren** — Jede Buchung muss den verwendeten Kurs nachvollziehbar machen
2. **Kursdifferenzen ausweisen** — Realisierte und unrealisierte Gewinne/Verluste separat
3. **USt korrekt umrechnen** — Für USt-Erklärungen gelten spezifische Umrechnungsregeln
4. **Jahresabschluss** — Offene Fremdwährungsposten zum Stichtagskurs bewerten

### 2.3 Was passiert ohne Kursanpassung?

| Konsequenz | Auswirkung |
|------------|------------|
| Falscher Debitoren-/Kreditorensaldo | GuV verzerrt |
| Keine Kursdifferenzen gebucht | Steuerliche Problemzone |
| Bankabstimmung unvollständig | Cashflow-Fehler |
| Konzernkonsolidierung unmöglich | Reporting-Lücke |

---

## 3. Das Währungssystem im Überblick

### 3.1 Die drei Ebenen

BC kennt drei Währungsebenen:

| Ebene | Abkürzung | Bedeutung |
|-------|-----------|-----------|
| **Mandantenwährung** | LCY | Die „Heimatwährung" Ihres Unternehmens, z.B. EUR |
| **Fremdwährung** | FCY | Jede andere Währung, z.B. USD, GBP, CHF |
| **Zusätzliche Berichtswährung** | ACY | Optional: Zweite Währung für Konzernreporting, z.B. EUR bei USD-Mandant |

### 3.2 Die zentralen Bausteine

| Baustein | Wo zu finden | Zweck |
|----------|-------------|-------|
| **Währungsstamm** | Suche „Währungen" | Währungsdefinition, Sachkonten, Rundung |
| **Wechselkurse** | Suche „Wechselkurse" | Tägliche Kurse pro Währung |
| **Wechselkursdienste** | Suche „Wechselkursdienst einrichten" | Automatische Kursabfrage (EZB, Banken) |
| **Kursanpassung** | Suche „Wechselkurse regulieren" | Periodische Neubewertung |

### 3.3 So funktioniert der Wechselkurs

BC speichert Wechselkurse **relational** — nicht einfach „1 USD = 0,92 EUR", sondern als
Beziehung zwischen zwei Währungen:

```
Währung:      USD
Bezugs-Währung: EUR
Kursbetrag:   1
Bezugs-Kursbetrag: 0,92
→ Lesart: 1 USD entspricht 0,92 EUR
```

Die Mandantenwährung (LCY) ist immer die implizite Bezugswährung, wenn keine andere angegeben wird.

---

## 4. Einrichtung Schritt für Schritt

### 4.1 Währung anlegen

1. **Währungen** suchen und öffnen
2. **Neu** klicken
3. Folgende Felder ausfüllen:

| Feld | Beispiel (USD) | Pflicht? |
|------|---------------|----------|
| Code | USD | ✅ Ja |
| ISO-Code | USD | Empfohlen |
| ISO-Nummer | 840 | Empfohlen |
| Symbol | $ | Optional |
| Beschreibung | US-Dollar | Optional |
| E.M.U.-Währung | (leer) | Nur für Euro-Teilnehmerländer |
| Rechnungsrundungspräzision | 0,01 | Empfohlen |
| Betragsrundungspräzision | 0,01 | Empfohlen |
| Einheitsbetragsrundungspräzision | 0,00001 | Empfohlen |
| Betragsdezimalstellen | '2:2' | Empfohlen |

### 4.2 Sachkonten hinterlegen

Jede Währung benötigt **vier bis acht Sachkonten** für Kursgewinne und -verluste:

| Feld | Konto-Typ | Empfohlenes Konto |
|------|-----------|-------------------|
| Unrealized Gains Acc. | Nicht realisierte Kursgewinne | Debitoren-seitig |
| Realized Gains Acc. | Realisierte Kursgewinne | Debitoren-seitig |
| Unrealized Losses Acc. | Nicht realisierte Kursverluste | Debitoren-seitig |
| Realized Losses Acc. | Realisierte Kursverluste | Debitoren-seitig |
| Realized G/L Gains Account | Realisierte Sachkonto-Kursgewinne | Fibu-seitig |
| Realized G/L Losses Account | Realisierte Sachkonto-Kursverluste | Fibu-seitig |
| Residual Gains Account | Rundungsdifferenz-Gewinne | Optional |
| Residual Losses Account | Rundungsdifferenz-Verluste | Optional |

> **Wichtig für EUR-User:** Die EUR-Währung selbst braucht keine Sachkonten — EUR = LCY.
> Bei der Buchung in EUR entstehen keine Kursdifferenzen.

### 4.3 Wechselkurse manuell erfassen

1. **Wechselkurse** suchen und öffnen
2. Währungscode auswählen (z.B. USD)
3. **Neu** → Folgende Felder:

| Feld | Wert | Bedeutung |
|------|------|-----------|
| Starting Date | 01.06.2024 | Gültig ab diesem Datum |
| Exchange Rate Amount | 1 | 1 Einheit dieser Währung |
| Relational Exch. Rate Amount | 0,92 | Entspricht 0,92 Einheiten der Bezugswährung |
| Adjustment Exch. Rate Amount | 1 | Kurs für die monatliche Neubewertung |
| Relational Adjmt Exch Rate Amt | 0,90 | Bezugs-Kurs für die Neubewertung |

> **Tipp:** `Adjustment Exch. Rate Amount` kann vom Tageskurs abweichen.
> Die EZB z.B. veröffentlicht einen separaten Referenzkurs für Bilanzierungszwecke.

### 4.4 Wechselkursdienste einrichten (automatische Kurse)

BC kann Wechselkurse **automatisch** von externen Anbietern abrufen:

1. **Wechselkursdienst einrichten** suchen
2. Dienst auswählen (z.B. EZB, Federal Reserve, FloatRates)
3. API-Schlüssel hinterlegen (falls erforderlich)
4. Zeitplan einrichten: Täglich um 08:00 Uhr
5. Zu aktualisierende Währungen zuweisen

> **Hinweis für Österreich:** Die **EZB-Referenzkurse** sind der Standard für BC28.
> Sie werden täglich um ca. 16:00 Uhr veröffentlicht und stehen am nächsten Arbeitstag
> automatisch zur Verfügung.

### 4.5 Zusätzliche Berichtswährung (ACY) einrichten

Wenn Sie eine zweite Währung für Konzernreporting benötigen:

1. **Fibu-Einrichtung** → Feld „Zusätzliche Berichtswährung" = z.B. „EUR"
2. Die ACY-Währung muss im Währungsstamm angelegt sein
3. Wechselkurse für die ACY-Währung erfassen
4. **Separate Gewinn-/Verlustkonten** für die ACY-Währung einrichten:
   - Diese Konten müssen **Exchange Rate Adjustment = No Adjustment** haben!
5. Sachkonten einzeln konfigurieren:
   - Bilanzkonten → **Adjust Amount** (ACY → LCY bewerten)
   - Erfolgskonten → **Adjust Additional-Currency Amount** (LCY → ACY bewerten)

### 4.6 Währung einem Debitor/Kreditor zuweisen

1. Debitor/Kreditor öffnen
2. Feld **Währungscode** = z.B. „USD"
3. Alle Belege für diesen Debitor werden automatisch in USD geführt

---

## 5. Tägliche Arbeit mit Fremdwährungen

### 5.1 Verkaufsrechnung in Fremdwährung

**Szenario:** US-Kunde, Rechnungsbetrag $10.000,00, LCY = EUR

1. Verkaufsrechnung öffnen, Debitor = US-Kunde
2. Währungscode **USD** wird automatisch gesetzt
3. Positionen erfassen → Summe $10.000,00
4. BC sucht Wechselkurs: **1 USD = 0,92 EUR** (Datum ≤ Rechnungsdatum)
5. Buchen

**Buchungsbeleg:**
```
Sachkonto                   Betrag (EUR)    Betrag (USD)
─────────────────────────────────────────────────────────
Debitor                      9.200,00       $10.000,00
   an Erlöse                  8.750,00       $ 9.509,20
   an USt                     1.750,00       $ 1.902,00
```

### 5.2 Einkaufsrechnung in Fremdwährung

**Szenario:** UK-Lieferant, Rechnungsbetrag £5.000,00, LCY = EUR

Ablauf analog zur Verkaufsrechnung — BC sucht den GBP-Wechselkurs und rechnet um:

```
Kreditor                     5.850,00 €     £5.000,00
   an Aufwand                 5.850,00 €     £5.000,00
```

### 5.3 Gutschrift in Fremdwährung

**Szenario:** Vom US-Kunden werden $2.000,00 gutgeschrieben

- Der Wechselkurs des Gutschriftsdatums wird verwendet
- Falls sich der Kurs seit der Rechnungsstellung geändert hat, entsteht eine **Kursdifferenz**,
  die beim Zahlungsausgleich realisiert wird

### 5.4 Zahlungsausgleich mit Kursdifferenz

**Szenario:** Rechnung $10.000 zu 0,92 EUR/USD → Zahlung $10.000 zu 0,95 EUR/USD

1. Zahlungseingangsbuchblatt öffnen
2. Offenen Debitorenposten auswählen
3. Zahlbetrag $10.000,00 buchen

**Automatische Buchung (BC rechnet selbst):**
```
Bank                          9.500,00 €
   an Debitor                  9.200,00 €
   an Kursgewinne realisiert     300,00 €
```

> **Ohne Ihr Zutun** wird die Kursdifferenz von **300,00 €** automatisch gebucht!

### 5.5 Manuelle Kursänderung auf einem Beleg

Falls Sie einen **aktuelleren** Kurs verwenden möchten:

1. Beleg öffnen (z.B. Verkaufsrechnung)
2. Im Feld „Währungsfaktor" den aktuellen Wert eingeben
3. BC rechnet alle Beträge mit dem neuen Kurs um

---

## 6. Die monatliche Kursanpassung

### 6.1 Warum Kursanpassung?

Zum Monatsende müssen **offene Fremdwährungsposten** zum aktuellen Kurs bewertet werden.
Kursgewinne und -verluste sind zu buchen — auch wenn die Zahlung noch aussteht.

**Ohne Kursanpassung** weist Ihre Bilanz veraltete Werte aus.

### 6.2 Schritt-für-Schritt: Kursanpassung ausführen

1. **Wechselkurse regulieren** suchen und öffnen
2. Parameter eintragen:

| Feld | Empfehlung | Bedeutung |
|------|-----------|-----------|
| Startdatum | 01. des Monats | Ab wann Posten bewertet werden |
| Enddatum | Letzter Tag des Monats | Bis wann Posten bewertet werden |
| Buchungsdatum | Letzter Tag des Monats | Datum der Anpassungsbuchung |
| Belegnummer | Juni2024 o.ä. | Identifikation der Anpassung |
| Buchungsbeschreibung | mit Platzhaltern %1 %2 | Text für Buchungszeilen |
| Kunden anpassen | ✅ Ja | Debitorenposten bewerten |
| Kreditoren anpassen | ✅ Ja | Kreditorenposten bewerten |
| Bank anpassen | ✅ Ja | Bankposten bewerten |
| Mitarbeiterposten anpassen | ✅ Ja | (falls genutzt) |
| Sachkonto regulieren | ✅ Ja | Sachkonten mit Fremdwährung |
| MwSt-Posten regulieren | ✅ Ja | MwSt auf Fremdwährung |
| Vorschau | Nein | Erst beim Testen aktivieren |

3. **OK** klicken
4. Fortschrittsdialog beobachten (läuft in wenigen Sekunden durch)
5. Ergebnis in **Kursanpassungs-Register** prüfen

### 6.3 Was passiert bei der Anpassung?

Für jeden offenen Debitorenposten in USD:

```
Vor Anpassung:
  Offener Betrag:       $5.000,00
  Gebuchter LCY-Wert:    4.600,00 €  (gebucht zu 1 USD = 0,92 EUR)

Nach Anpassung (aktueller Kurs: 1 USD = 0,90 EUR):
  Neuer LCY-Wert:        4.500,00 €  ($5.000,00 × 0,90)

Differenz:               -100,00 €   → Unrealisierter Verlust

Automatische Buchung:
  Kursverluste unrealisiert  100,00 €
     an Debitor                100,00 €
```

**Analog für Kreditoren, Bankkonten und Sachkonten.**

### 6.4 Anzeige und Rückverfolgbarkeit

Nach der Kursanpassung existieren folgende Einträge:

| Register/Liste | Wo zu finden | Inhalt |
|---------------|-------------|--------|
| **Kursanpassungs-Register** | Suche „Reg. Wechselkursanp." | Ein Register pro Anpassungslauf |
| **Kursanpassungs-Posten** | Im Register → Posten | Detail-Einträge pro angepasstem Posten |
| **Debitoren-/Kreditorenposten** | Debitoren-Liste → Posten | Neue Zeile mit Entry Type = Exch. Rate Adjmt. |

### 6.5 Vorschau nutzen

Vor dem ersten scharfen Lauf:

1. Kursanpassung öffnen
2. **Vorschau** = Ja
3. Nur **eine Währung** filtern (z.B. USD)
4. Ergebnis prüfen — noch keine Buchungen
5. Bei korrekten Werten: Vorschau = Nein, erneut ausführen

---

## 7. Zusätzliche Berichtswährung (ACY)

### 7.1 Wofür wird die ACY gebraucht?

Typische Szenarien:

| Szenario | LCY | ACY | Zweck |
|----------|-----|-----|-------|
| **US-Tochter** einer deutschen Konzernmutter | USD | EUR | Konsolidierung in EUR |
| **Schweizer Firma** mit internationalen Aktionären | CHF | USD | Investor-Reporting |
| **Osteuropa-Niederlassung** | PLN | EUR | Euro-Bilanz für Headquarter |

### 7.2 Wichtige Regeln

1. Die ACY-Gewinn-/Verlustkonten müssen **Exchange Rate Adjustment = No Adjustment** haben
2. Bilanzkonten → **Adjust Amount** (Fremdwährungssaldo in ACY bewerten)
3. Erfolgskonten → **Adjust Additional-Currency Amount** (LCY-Saldo in ACY bewerten)
4. Die Kursanpassung für ACY kann getrennt von der normalen Kursanpassung laufen

---

## 8. Spezialthemen

### 8.1 Fix Exchange Rate Amount

In der Wechselkurstabelle steuert dieses Enum, wie Cross-Rate-Berechnungen durchgeführt werden:

| Einstellung | Bedeutung | Anwendung |
|-------------|-----------|-----------|
| **Relational Currency** | Bezugswährung (z.B. EUR) ist fix, andere Währung wird angepasst | Standard für die meisten Fälle |
| **Currency** | Währung selbst ist fix, Bezugswährung wird angepasst | Wenn die Fremdwährung als Leitwährung dient |
| **Both** | Beide Seiten werden neu berechnet | Selten; bei Triangulation über LCY |

> **Empfehlung:** Lassen Sie den Standard „Relational Currency" — das deckt 95 % der Fälle ab.

### 8.2 EMU-Währungen (Euro-Einführung)

Das Feld **EMU Currency** markiert Währungen, die am Euro teilgenommen haben
(DEM, FRF, ATS, ...). Für diese Währungen sind die Wechselkurse **irrevocable**
(unwiderruflich festgelegt).

```al
// Beispiele für EMU-Währungen
DEM:  1,95583 DEM = 1 EUR
ATS: 13,76030 ATS = 1 EUR
FRF:  6,55957 FRF = 1 EUR
```

In BC wird das durch dauerhafte Wechselkurseinträge ohne Enddatum abgebildet.

### 8.3 Rundungspräzisionen

Jede Währung hat drei Rundungseinstellungen:

| Feld | Beispiel | Bedeutung |
|------|----------|-----------|
| Invoice Rounding Precision | 0,01 | Rechnungsbeträge werden auf 1 Cent gerundet |
| Amount Rounding Precision | 0,01 | Allgemeine Beträge auf 1 Cent |
| Unit-Amount Rounding Precision | 0,00001 | Stückpreise auf 5 Dezimalstellen |

> **Achtung bei JPY:** Japanische Yen haben keine Cent. Setzen Sie die Rundungspräzision auf 1.

### 8.4 Währungssymbol-Position

BC28 kann das Währungssymbol **vor** oder **nach** dem Betrag platzieren:

| Einstellung | Anzeige |
|-------------|---------|
| **After** | 1.000,00 € |
| **Before** | $ 1.000,00 |

---

## 9. Häufige Fragen (FAQ)

### Einrichtung

**Q: Muss ich für EUR eine Währung anlegen?**
A: Ja — EUR muss als Währung existieren, auch wenn es Ihre Mandantenwährung ist.
   Aber: EUR benötigt **keine** Sachkonten für Kursgewinne/-verluste.

**Q: Wie viele Wechselkurse brauche ich pro Währung?**
A: Mindestens einen pro Tag, an dem gebucht wird. Mit dem Wechselkursdienst
   passiert das automatisch.

**Q: Kann ich Wechselkurse mit mehr als 5 Nachkommastellen erfassen?**
A: Ja — das Feld `Relational Exch. Rate Amount` erlaubt hohe Präzision.
   Der Currency Factor wird daraus automatisch berechnet.

### Anwendung

**Q: Warum ist mein Belegbetrag in LCY = 0?**
A: Es fehlt ein Wechselkurs für das Belegdatum. Wechselkurs eintragen oder korrigieren.

**Q: Ändert sich der Wechselkurs automatisch auf bestehenden Belegen?**
A: Nein — der Kurs wird zum Zeitpunkt der **Belegerfassung** fixiert und ändert sich nicht mehr.

**Q: Kann eine Rechnung in USD an einen Kunden mit EUR-Währungscode gehen?**
A: Nein — der Währungscode des Kunden wird auf den Beleg übertragen. Sie müssten den
   Code manuell überschreiben, was aber nicht empfohlen wird.

### Kursanpassung

**Q: Die Kursanpassung meldet „Nothing to adjust" — was läuft falsch?**
A: Das ist korrekt, wenn **alle Posten bereits ausgeglichen** sind. Nur offene Posten
   werden bewertet.

**Q: Die Anpassung erzeugt einen Fehler — Buchungsdatum nicht zulässig?**
A: Prüfen Sie:
   1. Fibu-Einrichtung → Allow Posting From/To (Buchungsperiode muss offen sein)
   2. Geschäftsjahr (der Monat muss im offenen Wirtschaftsjahr liegen)
   3. Das Buchungsdatum selbst darf nicht vor dem Startdatum der Anpassung liegen

**Q: Wie oft soll ich die Kursanpassung ausführen?**
A: In der Praxis **monatlich** zum Monatsletzten. Für große Fremdwährungsbestände
   kann auch eine wöchentliche Anpassung sinnvoll sein.

**Q: Was passiert, wenn ich die Kursanpassung vergesse?**
A: Die Posten bleiben mit ihrem ursprünglichen Kurs bewertet. Bei großen
   Kursbewegungen kann das die Bilanz erheblich verzerren. Nachholung ist möglich —
   einfach mit korrektem Datumsbereich neu ausführen.

### Zusätzliche Berichtswährung

**Q: Meine ACY-Kursanpassung erzeugt falsche Werte?**
A: Prüfen Sie:
   1. ACY-Gewinn-/Verlustkonten haben **Exchange Rate Adjustment = No**
   2. Alle anderen Sachkonten haben **Exchange Rate Adjustment ≠ No**
   3. Wechselkurse für die ACY-Währung sind korrekt

**Q: Kann ich mehr als eine ACY haben?**
A: Im Standard nur eine. Für mehrere Berichtswährungen sind Drittanbieter-Lösungen
   oder Custom-Erweiterungen notwendig.

---

## 10. Praxis-Checkliste Monatsende

Vor dem Monatsabschluss in folgender Reihenfolge abarbeiten:

- [ ] **Wechselkurse aktualisieren** (Wechselkursdienst prüfen)
- [ ] **Fehlende Kurse manuell eintragen**
- [ ] **Zahlungen verbuchen** (Zahlungseingang/-ausgang)
- [ ] **Kursanpassung VORSCHAU** für alle Währungen
- [ ] **Vorschaubuchblatt prüfen** (auffällige Beträge?)
- [ ] **Kursanpassung SCHARF** für alle Währungen
- [ ] **Register kontrollieren** (Gesamtbetrag plausibel?)
- [ ] **Sachkontensalden prüfen** (Gewinn-/Verlustkonten korrekt?)

---

## 11. Wichtige Datenquellen und Links

| Ressource | URL / Pfad in BC |
|-----------|-----------------|
| Währungsstamm | Suche „Währungen" |
| Wechselkurse | Suche „Wechselkurse" |
| Wechselkursdienst einrichten | Suche „Wechselkursdienst einrichten" |
| Kursanpassung ausführen | Suche „Wechselkurse regulieren" |
| Kursanpassungs-Register | Suche „Reg. Wechselkursanp." |
| Fibu-Einrichtung | Suche „Fibu-Einrichtung" |
| EZB-Referenzkurse | https://www.ecb.europa.eu/stats/policy_and_exchange_rates/ |
| ISO-4217 Währungscodes | https://www.iso.org/iso-4217-currency-codes.html |

---

*Dokumentation basiert auf Business Central 28 (On-Premise). Stand: Juni 2024.*
