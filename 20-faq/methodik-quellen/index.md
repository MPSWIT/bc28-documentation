---
title: "FAQ — Methodik & Quellen"
---
# 20.2 Methodik & Quellen

> **TL;DR:** Alle Feldübersetzungen und fachlichen Details werden gegen den lokalen Quellcode und die offizielle XLF-Übersetzungsdatei verifiziert — nicht aus dem LLM-Trainingswissen.

> 📄 **[← FAQ-Übersicht]({{ '/20-faq/' | relative_url }})**

---

## Die Frage

> *„Woher hast du diese Information?"* — bezogen auf die Behauptung, dass Skonto-Konten separat in der Buchungsmatrix existieren.

## Quellen-Pyramide

```
      ▲  LLM-Trainingswissen (ungenau, veraltet)
     ┃
    ┃┃  RAG-Vektorsuche (liefert Code-Snippets mit Kontext)
   ┃ ┃
  ┃  ┃  XLF-Übersetzungsdatei (amtliche de-AT-Kontenbezeichnungen)
 ┃   ┃
━━━━━━━ Lokaler Quellcode (GeneralPostingSetup.Table.al) ━━ definitiv
```

Bei jeder fachlichen Behauptung gilt:

1. **RAG-Suche** im Collection `28.1.49838.49886.at` gibt erste Hinweise
2. **Quellcode lesen** (z.B. `GeneralPostingSetup.Table.al` Zeile 120–350) liefert die XML-Dokumentation und Feldnamen
3. **XLF-Datei** (`Base Application.de-AT.xlf`) liefert die offizielle deutsche Übersetzung jedes Feldnamens

### Beispiel: Skonto-Konten

Der RAG-Testcode `ERMPmtDiscAndVATCustVend.Codeunit.al` zeigte die Prozedur `GetSalesPmtDiscountAccount()`. 
Durch Lesen von `GeneralPostingSetup.Table.al` wurden die Felder 13, 17, 30, 31 gefunden:

```al
// Zeile 123 — GeneralPostingSetup.Table.al
field(13; "Sales Pmt. Disc. Debit Acc."; Code[20])
{
    // XML-Doc: "G/L account used for posting sales payment discount debit entries"
}
```

Die XLF-Datei bestätigte: `"Sales Pmt. Disc. Debit Acc."` → **„Verk.-Skonto Sollkonto"**

### Verifikation in Zahlen

- **150+ Feldübersetzungen** in den Kapiteln 4–6 sind XLF-geprüft
- **Jede Setup-Tabelle** wurde gegen ihren Quellcode abgeglichen (`GeneralLedgerSetup`, `SalesReceivablesSetup`, `PurchasesPayablesSetup`)
- **Jede Buchungsmatrix-Referenz** wurde im `GeneralPostingSetup.Table.al` verifiziert

---

## Die Frage

> *„Also hast du diese Information nicht über API aus dem Modell sondern vom lokalen Code?"*

Ja. Die konkreten Feldnamen, XML-Dokumentation und deutschen Übersetzungen stammen aus **drei lokalen Dateien**:

| Datei | Pfad | Inhalt |
|---|---|---|
| `GeneralPostingSetup.Table.al` | `…/Finance/GeneralLedger/Setup/` | Felddefinitionen + XML-Docs |
| `Base Application.de-AT.xlf` | `…/Translations/` | Deutsche Übersetzungen |
| RAG-Index | Qdrant `28.1.49838.49886.at` | Code-Snippets für erste Orientierung |

---

| [← FAQ-Übersicht]({{ '/20-faq/' | relative_url }}) | [Jekyll-Struktur →]({{ '/20-faq/jekyll-struktur/' | relative_url }}) |
