---
title: "FAQ — Fachkonzepte"
---
# 19.1 Rabattbuchung: Rechnungsrabatt vs. Skonto

> **TL;DR:** Rechnungsrabatt = unabhängig vom Zahlungszeitpunkt. Skonto = für frühzeitige Zahlung. Unterschiedliche Konten in der Buchungsmatrix.

> 📄 **[← FAQ-Übersicht]({{ '/19-faq/' | relative_url }})**

---

## Die Frage

> *„Das Beispiel mit 2 Rabatten: a) Zeilenrabatt, b) Rechnungsrabatt. Im Beispiel wird von 2 % Rabatt bei Zahlung innerhalb 14 Tagen gesprochen. Hierbei handelt es sich um einen Skonto!"*

## Die Antwort

Es gibt **drei** verschiedene Rabatttypen in BC28 — nicht zwei:

| Typ | Bedeutung | Beispiel | Tabelle |
|---|---|---|---|
| **Zeilenrabatt** (Line Discount) | Abschlag pro Belegzeile, meist mengenabhängig | 10 % ab 100 Stück | `Sales/Purch Line Discount` |
| **Rechnungsrabatt** (Invoice Discount) | Abschlag auf die Gesamtsumme, **unabhängig vom Zahlungszeitpunkt** | 3 % ab 5.000 € Bestellwert | `Cust./Purch. Invoice Disc.` |
| **Skonto** (Payment Discount) | Abschlag für **frühzeitige Zahlung** | 2 % bei Zahlung innerhalb 14 Tagen | `Payment Terms` (Tab. 3) |

### Warum das wichtig ist

Diese drei Typen landen auf **unterschiedlichen Konten** in der Buchungsmatrix (`General Posting Setup`):

| Feld-ID | AL-Name | Deutsche XLF-Übersetzung | Rabatttyp |
|---|---|---|---|
| 15 | `Purch. Line Disc. Account` | Eink.-Zeilenrabattkonto | Zeilenrabatt |
| 16 | `Purch. Inv. Disc. Account` | Eink.-Rechnungsrabattkonto | Rechnungsrabatt |
| 17 | `Purch. Pmt. Disc. Credit Acc.` | **Eink.-Skonto Habenkonto** | Skonto |

Die `Discount Posting`-Option in der Einrichtung steuert **nur** Zeilen- und Rechnungsrabatte. Skonto wird über `Payment Terms` → `Discount %` + `Discount Date Calculation` konfiguriert und über das `Adjust for Payment Disc.`-Flag aktiviert.

---

| [← FAQ-Übersicht]({{ '/19-faq/' | relative_url }}) | [Methodik & Quellen →]({{ '/19-faq/methodik-quellen/' | relative_url }}) |
