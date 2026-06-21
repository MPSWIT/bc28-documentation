---
title: "Einkauf — Einrichtung (Tabelle 312)"
---
# 6.1 Einkaufseinrichtung — Purchases & Payables Setup

> **Tabelle 312:** `PurchasesPayablesSetup.Table.al` · 80+ Felder
> **Namensraum:** `Microsoft.Purchases.Setup`
> **Typ:** Singleton (ein Datensatz pro Mandant)

Das **Purchases & Payables Setup** ist das Spiegelbild zum Sales & Receivables Setup (Tab. 311). Es steuert Nummernserien, Wareneingang, Rabatte, Buchungsverhalten und die Rechnungsprüfung.

> 📄 **[← Zurück zur Einkaufsübersicht]({{ '/06-purchasing/' | relative_url }}/)**
> 📋 Nächstes: [Einkaufsbelege →](einkaufsbelege)

---

## 6.1.1 Allgemeine Buchungsoptionen

### Feld 2: `Discount Posting` — Rabattbuchung

```al
field(2; "Discount Posting"; Option)
{
    OptionMembers = "No Discounts","Invoice Discounts","Line Discounts","All Discounts";
}
```

**Beispiel 1 — Trennung von Zeilen- und Rechnungsrabatten:**
> Der Elektronikhändler *TechTrading* erhält vom Hersteller 10 % Zeilenrabatt auf Komponenten und zusätzlich 3 % Rechnungsrabatt ab 10.000 € Bestellwert (kein Skonto — der Rechnungsrabatt wird unabhängig vom Zahlungszeitpunkt gewährt).
> ➜ `Discount Posting = All Discounts`
> **Ergebnis:** Beide Rabattarten werden separat auf die Konten `Purch. Line Disc. Account` und `Purch. Inv. Disc. Account` gebucht. Skonti werden dagegen über die **Zahlungsbedingungen** (`Payment Terms`, Tab. 3) gesteuert — sie sind kein Rechnungsrabatt.

**Beispiel 1b — Unterscheidung Rechnungsrabatt vs. Skonto:**
> Derselbe Hersteller gewährt zusätzlich 2 % **Skonto** bei Zahlung innerhalb 14 Tagen. Dies wird NICHT über `Discount Posting` konfiguriert, sondern in den **Zahlungsbedingungen** (`Payment Terms → Discount % = 2`, `Discount Date Calc. = 14D`).
> **Ergebnis:** Der Skonto wird beim Zahlungsausgleich automatisch berechnet und auf das `Pmt. Disc. Account` (Buchungsmatrix) gebucht.

### Feld 6: `Receipt on Invoice` — Lieferschein b. EK-Rechnung

```al
field(6; "Receipt on Invoice"; Boolean)
```

**Beispiel 1 — Dienstleister ohne Wareneingang:**
> Die Werbeagentur *CreativeMind* kauft Druckdienstleistungen ein. Es gibt keinen physischen Wareneingang.
> ➜ `Receipt on Invoice = true`
> **Ergebnis:** Beim Rechnungsbuchen wird automatisch auch der Wareneingang gebucht.

### Feld 30: `Check Doc. Total Amounts` — Gesamtbeträge des Belegs prüfen

```al
field(30; "Check Doc. Total Amounts"; Boolean)
```

**Beispiel 1 — Rechnungsprüfung vor Zahlung:**
> Die *BauProfi GmbH* vergleicht Einkaufsrechnungen mit Bestellsummen. Abweichungen > 5 % müssen genehmigt werden.
> ➜ `Check Doc. Total Amounts = true`
> **Ergebnis:** BC28 warnt, wenn Rechnungsbetrag und Bestellsumme nicht übereinstimmen.

---

## 6.1.2 Nummernserien

| Feld | AL-Name | Deutsche Bezeichnung |
|---|---|---|
| 9 | `Vendor Nos.` | Kreditorennummern |
| 10 | `Quote Nos.` | Anfragenummern |
| 11 | `Order Nos.` | Bestellungsnummern |
| 12 | `Invoice Nos.` | Rechnungsnummern |
| 13 | `Posted Invoice Nos.` | Gebuchte Rechnungsnummern |
| 14 | `Credit Memo Nos.` | Gutschriftsnummern |
| 15 | `Posted Credit Memo Nos.` | Gebuchte Gutschriftsnummern |
| 16 | `Posted Receipt Nos.` | Gebuchte Lieferungsnummern |
| 23 | `Blanket Order Nos.` | Rahmenbestellungsnummern |
| 32 | `Delivery Reminder Nos.` | Lieferanmahnungsnummern |
| 33 | `Issued Delivery Reminder Nos.` | Reg. Lieferanmahnungsnummern |
| 21 | `Posted Prepmt. Inv. Nos.` | Geb. Vorauszahlungs-Rechnungsnr. |
| 22 | `Posted Prepmt. Cr. Memo Nos.` | Geb. Vorauszahlungs-Gutschriftennr. |
| 6600 | `Return Order Nos.` | Reklamationsnummern |
| 6601 | `Posted Return Shpt. Nos.` | Gebuchte Rücklieferungsnummern |

---

## 6.1.3 Rabatt & Preissteuerung

| Feld | Code | Deutsche Beschreibung |
|---|---|---|
| 24 | `Calc. Inv. Discount` | Rechnungsrab. berechnen |
| 31 | `Calc. Inv. Disc. per VAT ID` | Rech.Rab. pro MwSt.Kennz. ber. |
| 7000 | `Price Calculation Method` | Preisberechnungsmethode |
| 7003 | `Default Price List Code` | Standardpreislistencode |
| 7002 | `Allow Editing Active Price` | Bearbeiten des aktiven Preises zulassen |

## 6.1.4 Einkaufsanmahnung (Delivery Reminder)

Das Pendant zum Mahnwesen im Verkauf: Wenn Lieferanten zu spät liefern, kann eine **Lieferanmahnung** erstellt werden.

| Feld | Code | Beschreibung |
|---|---|---|
| 32 | `Delivery Reminder Nos.` | Lieferanmahnungsnummern |
| 33 | `Issued Delivery Reminder Nos.` | Registrierte Nummern |

## 6.1.5 Fibu-Verknüpfung (Journal Templates)

| Feld | Code | Beschreibung |
|---|---|---|
| 201 | `P. Invoice Template Name` | EK-Rechnungsvorlagenname |
| 202 | `P. Cr. Memo Template Name` | EK-Gutschriftvorlagenname |
| 203 | `P. Prep. Inv. Template Name` | EK-Vorb.-Rechnungsvorlagenname |
| 204 | `P. Prep. Cr.Memo Template Name` | EK-Vorb.-Gutschriftvorlagenname |
| 205 | `IC Purch. Invoice Templ. Name` | IC-Buch.-Bl.-Vorl.-EK-Rechnung |
| 206 | `IC Purch. Cr. Memo Templ. Name` | IC-Buch.-Bl.-Vorl.-EK-Gutschrift |

## 6.1.6 Aufgabenwarteschlange

Analog zum Vertrieb: `Post with Job Queue`, `Post & Print with Job Queue`, `Job Queue Category Code` etc.

## 6.1.7 Belegverhalten

| Feld | Code | Deutsche Beschreibung |
|---|---|---|
| 35 | `Default Posting Date` | Standardbuchungsdatum |
| 56 | `Default Del. Rem. Date Field` | Standard Lief.-Mahn. Datumsfeld |
| 34 | `Default Qty. to Receive` | Standardmenge akt. Lieferung |
| 49 | `Document Default Line Type` | Standardzeilentyp des Belegs |

## 6.1.8 Retoure & Reklamation

| Feld | Code | Beschreibung |
|---|---|---|
| 6600 | `Return Order Nos.` | Reklamationsnummern |
| 6601 | `Posted Return Shpt. Nos.` | Gebuchte Rücklieferungsnummern |
| 6602 | `Return Shipment on Credit Memo` | Rücklieferung bei Gutschrift |
| 6603 | `Exact Cost Reversing Mandatory` | Einst.-Pr.-Rückverfolg. notw. |

---

## 6.1.9 Entwickler-Referenz

**Wichtige Prozeduren:**
```al
procedure GetRecordOnce()
// Caching-Pattern wie im Sales Setup

procedure JobQueueActive(): Boolean
```

**Abhängigkeiten:** `Microsoft.Finance.*`, `Microsoft.Pricing.*`, `Microsoft.Foundation.NoSeries`

---

| [← Zurück zur Übersicht]({{ '/06-purchasing/' | relative_url }}/) | [Einkaufsbelege →](einkaufsbelege) |
