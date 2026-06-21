---
title: "Vertrieb & Marketing — Einrichtung (Tabelle 311)"
---
# 5.1 Vertriebseinrichtung — Sales & Receivables Setup

> **Tabelle 311:** `SalesReceivablesSetup.Table.al` · 73 Felder
> **Namensraum:** `Microsoft.Sales.Setup`
> **Typ:** Singleton (ein Datensatz pro Mandant)
> **Seite:** `Sales & Receivables Setup` (Page)

Das **Sales & Receivables Setup** ist die zentrale Konfigurationstabelle für das gesamte Vertriebsmodul. Hier werden Nummernserien, Buchungsverhalten, Preisberechnung, Archivierung, Mahnwesen, Hintergrundbuchung und die Intercompany-Integration festgelegt. Die Einstellungen wirken mandantenweit auf alle Verkaufsbelege, Debitoren und Mahnvorgänge.

> 📄 **[← Zurück zur Vertriebsübersicht]({{ '/05-sales-marketing/' | relative_url }}/)**
> 📋 Nächstes: [Verkaufsbelege →](verkaufsbelege)

---

## 5.1.1 Allgemeine Buchungsoptionen

Diese Felder steuern die grundlegende Verarbeitung von Rabatten, Kreditwarnungen und Bestandsprüfungen beim Erfassen von Verkaufsbelegen.

### Feld 2: `Discount Posting` — Rabattbuchung

Steuert, ob und welche Verkaufsrabatte separat auf Sachkonten gebucht werden.

```al
field(2; "Discount Posting"; Option)
{
    OptionMembers = "No Discounts","Invoice Discounts","Line Discounts","All Discounts";
}
```

**Beispiel 1 — Einzelhandel bucht Zeilenrabatte separat aus:**
> Die Einzelhandelskette *Modehaus Berger* gewährt auf Sommerkollektionen 15 % Zeilenrabatt. Die Geschäftsleitung möchte in der GuV genau sehen, wie hoch die gewährten Rabatte waren.
> ➜ `Discount Posting = Line Discounts`
> **Ergebnis:** Jeder Zeilenrabatt wird auf das Konto aus `Gen. Posting Setup → Sales Line Disc. Account` gebucht.

**Beispiel 2 — B2B-Großhändler bucht Rechnungsrabatte:**
> Der Großhändler *Industriebedarf Süd* gewährt Stammkunden ab 5.000 € Auftragswert einen Rechnungsrabatt von 3 %. Dieser Rabatt gilt unabhängig vom Zahlungszeitpunkt.
> ➜ `Discount Posting = Invoice Discounts`
> **Ergebnis:** Rechnungsrabatte werden separat auf `Sales Inv. Disc. Account` gebucht. Transparente Ausweisung in der GuV. (Hinweis: 2 % Skonto bei Zahlung innerhalb 10 Tagen ist dagegen kein Rechnungsrabatt, sondern ein **Skonto** — konfiguriert in den **Zahlungsbedingungen** `Payment Terms`.)

**Beispiel 3 — Kleinstbetrieb ohne Rabattaufteilung:**
> Der Handwerksbetrieb *Meier Sanitär* gewährt selten Rabatte und möchte seine Buchhaltung einfach halten.
> ➜ `Discount Posting = No Discounts`
> **Ergebnis:** Rabatte werden vom Bruttobetrag abgezogen, bevor die Buchung erfolgt. Keine separaten Rabattkonten nötig.

**Wichtige Codestellen:**
- `OnValidate()` — Löst `DiscountNotificationMgt.NotifyAboutMissingSetup()` aus, um fehlende Buchungsmatrix-Einträge zu melden.

### Feld 4: `Credit Warnings` — Kreditlimitwarnung

Legt fest, bei welchen Kundenstatus-Warnungen das System beim Erstellen von Verkaufsaufträgen warnt.

```al
field(4; "Credit Warnings"; Option)
{
    OptionMembers = "Both Warnings","Credit Limit","Overdue Balance","No Warning";
}
```

**Beispiel 1 — Konservative Kreditpolitik mit Doppelwarnung:**
> Der Baustoffhändler *BauProfi GmbH* beliefert viele Kleinbetriebe und möchte vor Zahlungsausfällen geschützt sein.
> ➜ `Credit Warnings = Both Warnings`
> **Ergebnis:** Beim Erfassen eines Auftrags erscheint eine Warnung, wenn das Kreditlimit **oder** der offene Saldo überschritten ist.

**Beispiel 2 — Nur offene Posten überwachen:**
> Der Softwareanbieter *CodeBase AG* hat flexible Kreditlimits, aber offene Rechnungen über 60 Tage sind kritisch.
> ➜ `Credit Warnings = Overdue Balance`
> **Ergebnis:** Das System warnt nur bei überfälligen Debitorenposten, ignoriert das Kreditlimit.

### Feld 5: `Stockout Warning` — Bestandswarnung

Aktiviert eine Warnung, wenn die eingegebene Menge den verfügbaren Bestand unter Null bringen würde.

```al
field(5; "Stockout Warning"; Boolean)
{
    InitValue = true;
}
```

**Beispiel 1 — Produktionsbetrieb mit Just-in-Time:**
> Der Automobilzulieferer *PrecisionParts* arbeitet mit Mindestbeständen. Verkauf darf nur verfügbaren Bestand übersteigen, wenn explizit gewollt.
> ➜ `Stockout Warning = true`
> **Ergebnis:** Bei jeder Zeile, die den Bestand unter Null treibt, erscheint eine Warnmeldung.

**Beispiel 2 — Handelsunternehmen mit Nachlieferung:**
> Der Elektrogroßhändler *ElectroWorld* nimmt bewusst Aufträge über Lagerbestand hinaus an und liefert nach.
> ➜ `Stockout Warning = false`
> **Ergebnis:** Keine Warnungen — das Verkaufsteam kann ungehindert Aufträge erfassen.

### Feld 6: `Shipment on Invoice` — Lieferschein b. VK-Rechnung

Legt fest, ob beim Buchen einer Verkaufsrechnung automatisch auch ein Warenausgang gebucht wird.

```al
field(6; "Shipment on Invoice"; Boolean)
```

**Beispiel 1 — Dienstleister ohne Lager:**
> Die Unternehmensberatung *StratEx* fakturiert Beratungsleistungen direkt. Es gibt keine physische Ware.
> ➜ `Shipment on Invoice = true`
> **Ergebnis:** Beim Rechnungsbuchen entfällt der separate Warenausgang. Ein Klick genügt.

**Beispiel 2 — Lagergeführtes Handelsunternehmen:**
> Der Großhändler *FoodLogistics* kommissioniert Ware vor der Fakturierung. Warenausgang und Rechnung sind getrennte Schritte.
> ➜ `Shipment on Invoice = false`
> **Ergebnis:** Lagermitarbeiter buchen den Warenausgang (→ Bestandsabgang), Buchhaltung fakturiert später.

### Feld 7: `Invoice Rounding` — Rechnungsrundung

Aktiviert die Rechnungsrundung gemäß der in der Fibu-Einrichtung definierten Rundungspräzision.

```al
field(7; "Invoice Rounding"; Boolean)
```

> Siehe **[Fibu-Einrichtung → Rundung]({{ '/04-finance/fibu-einrichtung' | relative_url }}#46-rundung--berichtsw%C3%A4hrung)** für die zugehörigen Felder `Inv. Rounding Precision` und `Inv. Rounding Type`.

**Beispiel 1 — Schweizer KMU mit 5-Rappen-Rundung:**
> Die *Alpenblick AG* fakturiert in CHF. Die Fibu-Rundung ist auf 0.05 gesetzt.
> ➜ `Invoice Rounding = true`
> **Ergebnis:** Jede Verkaufsrechnung wird auf 5 Rappen gerundet. Die Rundungsdifferenz landet auf dem `Inv. Rounding Account` aus der Buchungsmatrix.

### Feld 8: `Ext. Doc. No. Mandatory` — Ext. Belegnr. erforderlich

Erzwingt die Eingabe einer externen Belegnummer auf Verkaufsköpfen und Fibu-Buchungszeilen.

```al
field(8; "Ext. Doc. No. Mandatory"; Boolean)
```

**Beispiel 1 — Buchhaltung verlangt Referenznummer des Kunden:**
> Die Steuerkanzlei *TaxPartner* verlangt, dass jedes Rechnungsdokument die Bestellnummer des Kunden enthält.
> ➜ `Ext. Doc. No. Mandatory = true`
> **Ergebnis:** Ohne `External Document No.` kann ein Verkaufsbeleg nicht gebucht werden.

### Feld 29: `Allow VAT Difference` — MwSt.-Differenz zulassen

Erlaubt die manuelle Anpassung von MwSt-Beträgen innerhalb der zulässigen MwSt-Differenz (definiert in der [Fibu-Einrichtung → MwSt]({{ '/04-finance/fibu-einrichtung' | relative_url }}#43-mehrwertsteuer)).

```al
field(29; "Allow VAT Difference"; Boolean)
```

**Beispiel 1 — Rundungsdifferenzen bei vielen Kleinpositionen:**
> Der Bürobedarfshändler *OfficeDirect* hat Rechnungen mit 50+ Positionen. Cent-Differenzen bei der MwSt sind unvermeidbar.
> ➜ `Allow VAT Difference = true`
> **Ergebnis:** MwSt-Beträge können manuell innerhalb der Toleranz korrigiert werden.

### Feld 210: `Copy Line Descr. to G/L Entry` — Zeilenbeschreibung zu Sachposten kopieren

Überträgt die Beschreibung von Sachkontenzeilen in die resultierenden Sachposten.

```al
field(210; "Copy Line Descr. to G/L Entry"; Boolean)
```

**Beispiel 1 — Nachvollziehbare Fibu-Buchungen:**
> Der Controller von *MediTech GmbH* möchte in den Sachposten sofort erkennen, wofür eine Buchung erfolgte.
> ➜ `Copy Line Descr. to G/L Entry = true`
> **Ergebnis:** Jede Rechnungszeile vom Typ "Sachkonto" erscheint mit ihrer Beschreibung im Sachposten.

---

## 5.1.2 Nummernserien

Die folgenden 20 Felder definieren, welche **Nummernserien** ([Kap. 3]({{ '/03-business-foundation/' | relative_url }})) für die verschiedenen Verkaufsbelege verwendet werden. Jedes Feld referenziert eine `No. Series` (Tabelle 308).

| Feld-Nr. | AL-Name | Deutsche Bezeichnung | Belegtyp |
|---|---|---|---|
| 9 | `Customer Nos.` | Debitorennummern | Kundenstamm |
| 10 | `Quote Nos.` | Angebotsnummern | Verkaufsangebot |
| 11 | `Order Nos.` | Auftragsnummern | Verkaufsauftrag |
| 12 | `Invoice Nos.` | Rechnungsnummern | Verkaufsrechnung |
| 13 | `Posted Invoice Nos.` | Gebuchte Rechnungsnummern | Gebuchte Verkaufsrechnung |
| 14 | `Credit Memo Nos.` | Gutschriftsnummern | Verkaufsgutschrift |
| 15 | `Posted Credit Memo Nos.` | Gebuchte Gutschriftsnummern | Gebuchte VK-Gutschrift |
| 16 | `Posted Shipment Nos.` | Gebuchte Lieferungsnummern | Gebuchter Warenausgang |
| 17 | `Reminder Nos.` | Mahnungsnummern | Mahnung |
| 18 | `Issued Reminder Nos.` | Registrierte Mahnungsnummern. | Registrierte Mahnung |
| 19 | `Fin. Chrg. Memo Nos.` | Zinsrechnungsnummern | Zinsrechnung |
| 20 | `Issued Fin. Chrg. M. Nos.` | Regist. Zinsrechnungsnummern | Registrierte Zinsrechnung |
| 21 | `Posted Prepmt. Inv. Nos.` | Geb. Vorauszahlungs-Rechnungsnr. | Gebuchte Vorausz.-Rechnung |
| 22 | `Posted Prepmt. Cr. Memo Nos.` | Geb. Vorauszahlungs-Gutschriftennr. | Gebuchte Vorausz.-Gutschrift |
| 23 | `Blanket Order Nos.` | Rahmenauftragsnummern | Rahmenauftrag |
| 45 | `Direct Debit Mandate Nos.` | Lastschrift-Mandat-Nr. | SEPA-Lastschriftmandat |
| 393 | `Canceled Issued Reminder Nos.` | Stornierte registrierte Mahnungsnummern | Stornierte Mahnung |
| 395 | `Canc. Iss. Fin. Ch. Mem. Nos.` | Stornierte registrierte Zinsrechnungsnummern | Stornierte Zinsrechnung |
| 6600 | `Return Order Nos.` | Reklamationsnummern | Verkaufsreklamation |
| 5800 | `Posted Return Receipt Nos.` | Gebuchte Rücksendungsnummern | Gebuchte Rücksendung |
| 7001 | `Price List Nos.` | Preislistenummern | Verkaufspreisliste |

**Beispiel — Mandant mit unterschiedlichen Nummernkreisen pro Land:**
> Die *HelvetiaTools AG* betreibt Mandanten für CH und DE. Jeder Mandant hat eigene Nummernserien:
> ➜ `Order Nos. = V-AUFTR-` → Ergebnis: `V-AUFTR-0042`
> ➜ `Invoice Nos. = RG-` → Ergebnis: `RG-2026-0103`

---

## 5.1.3 Rabatt & Preissteuerung

### Feld 24: `Calc. Inv. Discount` — Rechnungsrab. berechnen

Aktiviert die automatische Berechnung von Rechnungsrabatten gemäß den `Cust. Invoice Discounts`.

```al
field(24; "Calc. Inv. Discount"; Boolean)
```

**Beispiel 1 — Staffelrabatt für Stammkunden:**
> Der Getränkegroßhändler *DrinkStar* gewährt Kunden ab 10.000 € Umsatz 2 %, ab 50.000 € 5 % Rechnungsrabatt.
> ➜ `Calc. Inv. Discount = true`
> **Ergebnis:** Der Rabatt wird aus der Kundenkarte (`Invoice Disc. Code`) automatisch ermittelt und angewendet.

### Feld 30: `Calc. Inv. Disc. per VAT ID` — Rech.Rab. pro MwSt.Kennz. ber.

Berechnet den Rechnungsrabatt separat pro MwSt-Geschäftsbuchungsgruppe auf dem Beleg.

```al
field(30; "Calc. Inv. Disc. per VAT ID"; Boolean)
```

**Beispiel 1 — EU-Unternehmen mit gemischten Steuersätzen:**
> *EuroPharma GmbH* verkauft Medikamente (7 % MwSt) und Kosmetik (19 % MwSt) auf derselben Rechnung.
> ➜ `Calc. Inv. Disc. per VAT ID = true`
> **Ergebnis:** Der Rechnungsrabatt wird getrennt nach MwSt-Kennzeichen berechnet — 7%ige und 19%ige Posten erhalten proportional ihren Anteil.

### Feld 7000: `Price Calculation Method` — Preisberechnungsmethode

Definiert die Standard-Preisberechnungsmethode für Verkaufstransaktionen.

```al
field(7000; "Price Calculation Method"; Enum "Price Calculation Method")
{
    InitValue = "Lowest Price";
}
```

**Beispiel 1 — Immer den niedrigsten Preis nehmen:**
> Der Discount-Markt *PreisHammer* hat mehrere Preislisten (Aktionspreise, Kundenpreise) und will immer den günstigsten.
> ➜ `Price Calculation Method = Lowest Price`
> **Ergebnis:** BC28 sucht alle gültigen Preislisten durch und nimmt den niedrigsten Preis.

### Feld 7003: `Default Price List Code` — Standardpreislistencode

Definiert die Standard-Preisliste, in der neue Preise aus der Preisarbeitsmappe gespeichert werden.

```al
field(7003; "Default Price List Code"; Code[20])
```

**Beispiel 1 — Zentrale Verkaufspreisliste:**
> *BüroProfi AG* pflegt alle Preise zentral in einer "STANDARDVK"-Preisliste.
> ➜ `Default Price List Code = STANDARDVK`
> **Ergebnis:** Neue Preise aus der `Price Worksheet` landen automatisch in dieser Liste.

---

## 5.1.4 Archivierung

Steuert, ob und wann Verkaufsbelege archiviert werden. Archivierte Belege landen in der Tabelle `Sales Header Archive` (5107).

| Feld | Code | Deutsche Beschreibung | Optionen |
|---|---|---|---|
| 52 | `Archive Quotes` | Angebote archivieren | Never / Question / Always |
| 53 | `Archive Orders` | Aufträge archivieren | Boolean |
| 54 | `Archive Blanket Orders` | Rahmenaufträge archivieren | Boolean |
| 55 | `Archive Return Orders` | Reklamationen archivieren | Boolean |

```al
field(52; "Archive Quotes"; Option)
{
    OptionMembers = Never,Question,Always;
}
```

**Beispiel 1 — Industrieunternehmen archiviert alles:**
> *Maschinenbau Wolf GmbH* muss alle Angebote, Aufträge und Reklamationen aus Compliance-Gründen archivieren.
> ➜ `Archive Quotes = Always`, `Archive Orders = true`, `Archive Return Orders = true`
> **Ergebnis:** Jeder gelöschte/gebuchte Beleg wird vor dem Löschen in die Archiv-Tabellen kopiert.

---

## 5.1.5 Aufgabenwarteschlange (Hintergrundbuchung)

BC28 kann Verkaufsbelege im Hintergrund über die **Aufgabenwarteschlange** ([Fibu-Einrichtung → Aufgabenwarteschlange]({{ '/04-finance/fibu-einrichtung' | relative_url }}#47-aufgabenwarteschlange--hintergrundbuchung)) buchen.

| Feld | Code | Deutsche Beschreibung |
|---|---|---|
| 38 | `Post with Job Queue` | Mit Aufgabenwarteschlange buchen |
| 39 | `Job Queue Category Code` | Kategorie-Code |
| 40 | `Job Queue Priority for Post` | Priorität für Buchung (Standard 1000) |
| 41 | `Post & Print with Job Queue` | Buchen & Drucken |
| 42 | `Job Q. Prio. for Post & Print` | Priorität für Buchen & Drucken |
| 47 | `Report Output Type` | Art der Berichtsausgabe (PDF / Print) |

**Beispiel 1 — Sammelrechnungslauf über Nacht:**
> *Industriebedarf Süd* erstellt Monatsrechnungen für 200 Kunden. Der Vorgang dauert 90 Minuten.
> ➜ `Post with Job Queue = true`, `Job Queue Category Code = NACHTLAUF`, `Job Queue Priority for Post = 500`
> **Ergebnis:** Die Rechnungen werden als Aufgabenwarteschlangen-Eintrag eingeplant und über Nacht abgearbeitet. Der Anwender kann BC normal weiterverwenden.

---

## 5.1.6 Fibu-Verknüpfung (Journal Templates)

Diese Felder verknüpfen Sales-Belege mit den entsprechenden Fibu-Buchungsblattvorlagen aus dem General Journal ([Kap. 4, Journale]({{ '/04-finance/journale-debitoren-kreditoren' | relative_url }})).

| Feld | Code | Verwendung | Filter |
|---|---|---|---|
| 201 | `S. Invoice Template Name` | Verkaufsrechnungsbuch.-Blattvorlage | Type = Sales |
| 202 | `S. Cr. Memo Template Name` | Verkaufsgutschrift-Buch.-Blattvorlage | Type = Sales |
| 203 | `S. Prep. Inv. Template Name` | Verkaufsvorb.-Rechnungsvorlagenname | Type = Sales |
| 204 | `S. Prep. Cr.Memo Template Name` | Verkaufsvorb.-Guts.-Vorlagenname | Type = Sales |
| 205 | `IC Sales Invoice Template Name` | IC-Verkaufsrechnungsvorlagenname | Type = Intercompany |
| 206 | `IC Sales Cr. Memo Templ. Name` | IC-Verkaufs-Guts.-Vorlagenname | Type = Intercompany |
| 207 | `Fin. Charge Jnl. Template Name` | Buch.-Blattvorlagenname für Zinsrechnung | Type = Sales |
| 208 | `Reminder Journal Template Name` | Mahnung Buch.-Blattvorlagenname | Type = Sales |

**Beispiel 1 — Getrennte Journale für Inland und EU:**
> Die Exportabteilung der *GlobalTrade AG* verwendet separate Fibu-Buchungsjournale.
> ➜ `IC Sales Invoice Template Name = IC-VERKAUF`
> ➜ `Reminder Journal Template Name = MAHNUNG`
> **Ergebnis:** Intercompany-Rechnungen und Mahnungen landen in unterschiedlichen Journalen.

---

## 5.1.7 Belegverhalten

| Feld | Code | Deutsche Beschreibung | Werte |
|---|---|---|---|
| 35 | `Default Posting Date` | Standardbuchungsdatum | Work Date / No Date |
| 36 | `Default Quantity to Ship` | Zu liefernde Standardmenge | Remainder / Blank |
| 49 | `Document Default Line Type` | Standardzeilentyp des Belegs | Item / G/L Account / ... |
| 50 | `Default Item Quantity` | Standardartikelmenge | Boolean |
| 56 | `Default G/L Account Quantity` | Standardmäßige Sachkontomenge | Boolean |
| 51 | `Create Item from Description` | Artikel aus Beschreibung erstellen | Boolean |
| 61 | `Ignore Updated Addresses` | Aktualisierte Adressen ignorieren | Boolean |
| 65 | `Skip Manual Reservation` | Manuelle Reservierung überspringen | Boolean |
| 160 | `Disable Search by Name` | Suche nach Namen deaktivieren | Boolean |
| 200 | `Quote Validity Calculation` | Angebotsüberprüfungsberechnung | DateFormula |
| 7104 | `Link Doc. Date to Posting Date` | Belegdatum mit Buchungsdatum verknüpfen | Boolean |
| 10500 | `Posting Date Check on Posting` | Buchungsdatum bei Buchung prüfen | Boolean |

**Beispiel 1 — Verkäufer will "Work Date" als Buchungsdatum:**
> Der Außendienstler *Müller* erfasst einen Auftrag am Folgetag. Das Buchungsdatum soll das Arbeitsdatum sein.
> ➜ `Default Posting Date = Work Date`
> **Ergebnis:** Jeder neue Verkaufsbeleg übernimmt das Arbeitsdatum als Buchungsdatum.

**Beispiel 2 — Angebote laufen automatisch nach 30 Tagen ab:**
> Die *IT-Solutions GmbH* gibt Kunden 30 Tage Zeit, ein Angebot anzunehmen.
> ➜ `Quote Validity Calculation = 30D`
> **Ergebnis:** Bei Angebotserstellung wird das Ablaufdatum automatisch auf `Document Date + 30T` gesetzt.

---

## 5.1.8 Retoure & Reklamation

| Feld | Code | Deutsche Beschreibung |
|---|---|---|
| 6600 | `Return Order Nos.` | Reklamationsnummern |
| 6601 | `Return Receipt on Credit Memo` | Rücksendung bei Gutschrift |
| 6602 | `Exact Cost Reversing Mandatory` | Einst.-Pr.-Rückverfolg. notw. |
| 5800 | `Posted Return Receipt Nos.` | Gebuchte Rücksendungsnummern |
| 5801 | `Copy Cmts Ret.Ord. to Ret.Rcpt` | Bem. Rekl. in Rücksendung kop. |
| 5802 | `Copy Cmts Ret.Ord. to Cr. Memo` | Bem. Rekl. in Gutschrift kop. |
| 5775 | `Auto Post Non-Invt. via Whse.` | Lag-unab. aut. über Lager buchen |

```al
field(6602; "Exact Cost Reversing Mandatory"; Boolean)
```

**Beispiel 1 — Elektronikhändler mit exakter Kostenrückverfolgung:**
> *TechReturn AG* nimmt Retouren entgegen und muss die exakten Einstandspreise der retournierten Ware rückbuchen.
> ➜ `Exact Cost Reversing Mandatory = true`
> **Ergebnis:** Eine Rückgabezeile kann nur mit Bezug auf einen konkreten Artikelposten gebucht werden. Keine pauschalen Kostenrückbuchungen.

---

## 5.1.9 D365 Sales Integration

| Feld | Code | Deutsche Beschreibung |
|---|---|---|
| 5329 | `Write-in Product Type` | Manuell einzutragender Produkttyp (Item/Resource) |
| 5330 | `Write-in Product No.` | Nr. des manuell einzutragenden Produkts |
| 7103 | `Freight G/L Acc. No.` | Fracht-Sachkontonr. |

**Beispiel 1 — D365 Sales-Anbindung für Freitext-Produkte:**
> *DynaVertrieb* synchronisiert Verkaufschancen aus Dynamics 365 Sales. Nicht im Katalog vorhandene Produkte werden als "Non-Inventory Item" angelegt.
> ➜ `Write-in Product Type = Item`, `Write-in Product No. = FREITEXT`
> **Ergebnis:** Write-in Produkte aus D365 Sales werden automatisch als Artikel `FREITEXT` in der Auftragszeile hinterlegt.

---

## 5.1.10 Sonstige Felder

| Feld | Code | Deutsche Beschreibung |
|---|---|---|
| 25 | `Appln. between Currencies` | Währungsausgleich (None / EMU / All) |
| 31 | `Logo Position on Documents` | Logoposition auf Belegen |
| 32 | `Check Prepmt. when Posting` | Vorauszahlung beim Buchen prüfen |
| 33 | `Prepmt. Auto Update Frequency` | Häufigkeit für autom. Aktualisierungen zu Vorauszahlungen |
| 44 | `VAT Bus. Posting Gr. (Price)` | MwSt.-Geschäftsbuch.-G. (Preis) |
| 46 | `Allow Document Deletion Before` | Löschen des Belegs zulassen vor |
| 175 | `Allow Multiple Posting Groups` | Mehrere Buchungsgruppen zulassen |
| 176 | `Check Multiple Posting Groups` | Mehrere Buchungsgruppen überprüfen |
| 7101 | `Customer Group Dimension Code` | Debitorengruppen-Dim.-Code |
| 7102 | `Salesperson Dimension Code` | Verkäufer-Dimensionscode |

---

## 5.1.11 Komplette Feldliste (alle 73 Felder)

| ID | AL-Feldname | Deutsche Bezeichnung | Typ |
|---|---|---|---|
| 1 | `Primary Key` | Primärschlüssel | Code |
| 2 | `Discount Posting` | Rabattbuchung | Option |
| 4 | `Credit Warnings` | Kreditlimitwarnung | Option |
| 5 | `Stockout Warning` | Bestandswarnung | Boolean |
| 6 | `Shipment on Invoice` | Lieferschein b. VK-Rechnung | Boolean |
| 7 | `Invoice Rounding` | Rechnungsrundung | Boolean |
| 8 | `Ext. Doc. No. Mandatory` | Ext. Belegnr. erforderlich | Boolean |
| 9 | `Customer Nos.` | Debitorennummern | Code |
| 10 | `Quote Nos.` | Angebotsnummern | Code |
| 11 | `Order Nos.` | Auftragsnummern | Code |
| 12 | `Invoice Nos.` | Rechnungsnummern | Code |
| 13 | `Posted Invoice Nos.` | Gebuchte Rechnungsnummern | Code |
| 14 | `Credit Memo Nos.` | Gutschriftsnummern | Code |
| 15 | `Posted Credit Memo Nos.` | Gebuchte Gutschriftsnummern | Code |
| 16 | `Posted Shipment Nos.` | Gebuchte Lieferungsnummern | Code |
| 17 | `Reminder Nos.` | Mahnungsnummern | Code |
| 18 | `Issued Reminder Nos.` | Registrierte Mahnungsnummern. | Code |
| 19 | `Fin. Chrg. Memo Nos.` | Zinsrechnungsnummern | Code |
| 20 | `Issued Fin. Chrg. M. Nos.` | Regist. Zinsrechnungsnummern | Code |
| 21 | `Posted Prepmt. Inv. Nos.` | Geb. Vorauszahlungs-Rechnungsnr. | Code |
| 22 | `Posted Prepmt. Cr. Memo Nos.` | Geb. Vorauszahlungs-Gutschriftennr. | Code |
| 23 | `Blanket Order Nos.` | Rahmenauftragsnummern | Code |
| 24 | `Calc. Inv. Discount` | Rechnungsrab. berechnen | Boolean |
| 25 | `Appln. between Currencies` | Währungsausgleich | Option |
| 26 | `Copy Comments Blanket to Order` | Bem. Rahmenauf. in Auftr. kop. | Boolean |
| 27 | `Copy Comments Order to Invoice` | Bem. Auftrag in Rechnung kop. | Boolean |
| 28 | `Copy Comments Order to Shpt.` | Bem. Auftrag in Lieferung kop. | Boolean |
| 29 | `Allow VAT Difference` | MwSt.-Differenz zulassen | Boolean |
| 30 | `Calc. Inv. Disc. per VAT ID` | Rech.Rab. pro MwSt.Kennz. ber. | Boolean |
| 31 | `Logo Position on Documents` | Logoposition auf Belegen | Option |
| 32 | `Check Prepmt. when Posting` | Vorauszahlung beim Buchen prüfen | Boolean |
| 33 | `Prepmt. Auto Update Frequency` | Häufigkeit für autom. Aktualisierungen | Option |
| 35 | `Default Posting Date` | Standardbuchungsdatum | Enum |
| 36 | `Default Quantity to Ship` | Zu liefernde Standardmenge | Option |
| 38 | `Post with Job Queue` | Mit Aufgabenwarteschlange buchen | Boolean |
| 39 | `Job Queue Category Code` | Aufgabenwarteschlange - Kategoriencode | Code |
| 40 | `Job Queue Priority for Post` | Aufgabenwarteschlange - Priorität | Integer |
| 41 | `Post & Print with Job Queue` | Mit Aufgabenwarteschlange buchen & drucken | Boolean |
| 42 | `Job Q. Prio. for Post & Print` | Aufgabenwarteschlange für Buchen und Drucken | Integer |
| 43 | `Notify On Success` | Bei Erfolg benachrichtigen (Legacy) | Boolean |
| 44 | `VAT Bus. Posting Gr. (Price)` | MwSt.-Geschäftsbuch.-G. (Preis) | Code |
| 45 | `Direct Debit Mandate Nos.` | Lastschrift-Mandat-Nr. | Code |
| 46 | `Allow Document Deletion Before` | Löschen des Belegs zulassen vor | Date |
| 47 | `Report Output Type` | Art der Berichtsausgabe | Enum |
| 49 | `Document Default Line Type` | Standardzeilentyp des Belegs | Enum |
| 50 | `Default Item Quantity` | Standardartikelmenge | Boolean |
| 51 | `Create Item from Description` | Artikel aus Beschreibung erstellen | Boolean |
| 52 | `Archive Quotes` | Angebote archivieren | Option |
| 53 | `Archive Orders` | Aufträge archivieren | Boolean |
| 54 | `Archive Blanket Orders` | Rahmenaufträge archivieren | Boolean |
| 55 | `Archive Return Orders` | Reklamationen archivieren | Boolean |
| 56 | `Default G/L Account Quantity` | Standardmäßige Sachkontomenge | Boolean |
| 58 | `Copy Customer Name to Entries` | Debitorenname in Posten kopieren | Boolean |
| 61 | `Ignore Updated Addresses` | Aktualisierte Adressen ignorieren | Boolean |
| 65 | `Skip Manual Reservation` | Manuelle Reservierung überspringen | Boolean |
| 160 | `Disable Search by Name` | Suche nach Namen deaktivieren | Boolean |
| 175 | `Allow Multiple Posting Groups` | Mehrere Buchungsgruppen zulassen | Boolean |
| 176 | `Check Multiple Posting Groups` | Mehrere Buchungsgruppen überprüfen | Enum |
| 200 | `Quote Validity Calculation` | Angebotsüberprüfungsberechnung | DateFormula |
| 201 | `S. Invoice Template Name` | Verkaufsrechnungsbuch.-Blattvorlage | Code |
| 202 | `S. Cr. Memo Template Name` | Verkaufsgutschrift-Buch.-Blattvorlage | Code |
| 203 | `S. Prep. Inv. Template Name` | Verkaufsvorb.-Rechnungsvorlagenname | Code |
| 204 | `S. Prep. Cr.Memo Template Name` | Verkaufsvorb.-Guts.-Vorlagenname | Code |
| 205 | `IC Sales Invoice Template Name` | IC-Verkaufsrechnungsvorlagenname | Code |
| 206 | `IC Sales Cr. Memo Templ. Name` | IC-Verkaufs-Guts.-Vorlagenname | Code |
| 207 | `Fin. Charge Jnl. Template Name` | Buch.-Blattvorlagenname für Zinsrechnung | Code |
| 208 | `Reminder Journal Template Name` | Mahnung Buch.-Blattvorlagenname | Code |
| 393 | `Canceled Issued Reminder Nos.` | Stornierte registrierte Mahnungsnummern | Code |
| 395 | `Canc. Iss. Fin. Ch. Mem. Nos.` | Stornierte registrierte Zinsrechnungsnummern | Code |
| 5329 | `Write-in Product Type` | Manuell einzutragender Produkttyp | Option |
| 5330 | `Write-in Product No.` | Nr. des manuell einzutragenden Produkts | Code |
| 5775 | `Auto Post Non-Invt. via Whse.` | Lag-unab. aut. über Lager buchen | Enum |
| 5800 | `Posted Return Receipt Nos.` | Gebuchte Rücksendungsnummern | Code |
| 6600 | `Return Order Nos.` | Reklamationsnummern | Code |
| 6601 | `Return Receipt on Credit Memo` | Rücksendung bei Gutschrift | Boolean |
| 6602 | `Exact Cost Reversing Mandatory` | Einst.-Pr.-Rückverfolg. notw. | Boolean |
| 7000 | `Price Calculation Method` | Preisberechnungsmethode | Enum |
| 7001 | `Price List Nos.` | Preislistenummern | Code |
| 7002 | `Allow Editing Active Price` | Bearbeiten des aktiven Preises zulassen | Boolean |
| 7003 | `Default Price List Code` | Standardpreislistencode | Code |
| 7005 | `Use Customized Lookup` | Benutzerdefinierte Suche verwenden | Boolean |
| 7101 | `Customer Group Dimension Code` | Debitorengruppen-Dim.-Code | Code |
| 7102 | `Salesperson Dimension Code` | Verkäufer-Dimensionscode | Code |
| 7103 | `Freight G/L Acc. No.` | Fracht-Sachkontonr. | Code |
| 7104 | `Link Doc. Date to Posting Date` | Belegdatum mit Buchungsdatum verknüpfen | Boolean |
| 10500 | `Posting Date Check on Posting` | Buchungsdatum bei Buchung prüfen | Boolean |

---

## 5.1.12 Entwickler-Referenz

**Wichtige Prozeduren in Tabelle 311:**

```al
procedure GetRecordOnce()
// Liest den Setup-Datensatz einmal pro Session (Caching-Pattern)

procedure JobQueueActive(): Boolean
// True, wenn Hintergrundbuchung aktiv ist

procedure GetLegalStatement(): Text
// Überschreibbar in lokalen Versionen für Rechtshinweise auf Belegen
```

**Namensraum-Objekte:**
- `page "Sales & Receivables Setup"` — Die zugehörige Einrichtungsseite (Card)
- `codeunit "Discount Notification Mgt."` — Benachrichtigt über fehlende Buchungsmatrix-Einträge
- `codeunit "Prepayment Mgt."` — Verwaltet Aufgabenwarteschlangen-Einträge für Vorauszahlungen
- `codeunit "Price Calculation Mgt."` — Validiert Preisberechnungsmethode

**Abhängige Namensräume:**
- `Microsoft.Finance.GeneralLedger.*` — Buchungsmatrix, Sachkonten
- `Microsoft.Pricing.*` — Preis- und Rabattberechnung
- `Microsoft.Foundation.NoSeries` — Nummernserien
- `Microsoft.Integration.D365Sales` — Dynamics 365 Sales-Integration
- `Microsoft.Finance.Dimension` — Dimensionen für Analyse

---

| [← Zurück zur Übersicht]({{ '/05-sales-marketing/' | relative_url }}/) | [Verkaufsbelege →](verkaufsbelege) |
