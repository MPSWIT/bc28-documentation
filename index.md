---
title: "Übersicht"
---
# Business Central 28 — Technische Dokumentation

Willkommen zur vollständigen technischen Dokumentation der Microsoft Dynamics 365 Business Central 28
App-Architektur.

## Was ist BC28?

Business Central 28 (Release Wave 1 2025) ist die aktuelle Version von Microsofts ERP-Plattform
für kleine und mittlere Unternehmen. Diese Dokumentation beschreibt die **App-Architektur**:
die Module, Namespaces, Tabellen, Pages und Codeunits, aus denen BC28 besteht.

## Aufbau dieser Dokumentation

Die Dokumentation folgt der App-Struktur von BC28:

<pre>
Business Central 28
 │
 ├─▶ <strong>Übersicht</strong>  ← Sie sind hier
 │
 ├── 1.  <a href="{{ '/01-overview/' | relative_url }}">Überblick &amp; Architektur</a>
 │    ├── <a href="{{ '/01-overview/architektur/' | relative_url }}">App-Stack &amp; Namespaces</a>
 │    └── <a href="{{ '/01-overview/al-erweiterbarkeit/' | relative_url }}">AL-Sprachmerkmale &amp; Erweiterbarkeit</a>
 │
 ├── 2.  System Application  🚧
 │
 ├── 3.  Business Foundation  🚧
 │
 ├── 4.  <a href="{{ '/04-finance/' | relative_url }}">Finanzwesen</a>
 │    ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}">Fibu-Einrichtung (Tab. 98)</a>
 │    ├── <a href="{{ '/04-finance/waehrungen/' | relative_url }}">Währungen &amp; Wechselkurse</a>
 │    │    ├── <a href="{{ '/04-finance/waehrungen/einrichtung/' | relative_url }}">Einrichtung &amp; Setup</a>
 │    │    ├── <a href="{{ '/04-finance/waehrungen/verarbeitung/' | relative_url }}">Verarbeitung &amp; Umrechnung</a>
 │    │    ├── <a href="{{ '/04-finance/waehrungen/anwendung/' | relative_url }}">Anwendung &amp; Beispiele</a>
 │    │    └── <a href="{{ '/04-finance/waehrungen/entwickler/' | relative_url }}">Entwickler-Referenz</a>
 │    │    ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}#41-buchungszeiträume">4.1 Buchungszeiträume</a>
 │    │    ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}#42-dimensionen">4.2 Dimensionen</a>
 │    │    ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}#43-mehrwertsteuer">4.3 Mehrwertsteuer</a>
 │    │    ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}#44-zahlungstoleranzen--skonto">4.4 Zahlungstoleranzen &amp; Skonto</a>
 │    │    ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}#45-rundung--nachkommastellen">4.5 Rundung &amp; Nachkommastellen</a>
 │    │    ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}#46-zusätzliche-berichtswährung">4.6 Berichtswährung</a>
 │    │    ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}#47-aufgabenwarteschlange--hintergrundbuchung">4.7 Aufgabenwarteschlange</a>
 │    │    ├── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}#48-sonstige-wichtige-felder">4.8 Sonstige Felder</a>
 │    │    └── <a href="{{ '/04-finance/fibu-einrichtung/' | relative_url }}#49-aktionen-der-fibu-einrichtungsseite">4.9 Aktionen</a>
 │    ├── <a href="{{ '/04-finance/kontenplan-buchungsgruppen/' | relative_url }}">Kontenplan &amp; Buchungsgruppen</a>
 │    ├── <a href="{{ '/04-finance/mwst-system/' | relative_url }}">MwSt-System</a>
 │    ├── <a href="{{ '/04-finance/journale-debitoren-kreditoren/' | relative_url }}">Journale, Debitoren/Kreditoren</a>
 │    ├── <a href="{{ '/04-finance/bank-anlagen-waehrung/' | relative_url }}">Bank, Anlagen &amp; Währung</a>
 │    ├── <a href="{{ '/04-finance/berichte-analyse-budget/' | relative_url }}">Berichte, Budget &amp; Analyse</a>
 │    ├── <a href="{{ '/04-finance/konsolidierung-abgrenzung-ic/' | relative_url }}">Konsolidierung, Abgrenzungen &amp; IC</a>
 │    ├── <a href="{{ '/04-finance/querschnitt/' | relative_url }}">Querschnitt — Fibu-Relevanz aller Module</a>
 │    └── <a href="{{ '/04-finance/entwickler/' | relative_url }}">Entwickler-Referenz</a>
 │
 ├── 5.  <a href="{{ '/05-sales-marketing/' | relative_url }}">Vertrieb &amp; Marketing</a>
 │    ├── <a href="{{ '/05-sales-marketing/einrichtung/' | relative_url }}">Einrichtung (Tab. 311)</a>
 │    ├── <a href="{{ '/05-sales-marketing/verkaufsbelege/' | relative_url }}">Verkaufsbelege</a>
 │    ├── <a href="{{ '/05-sales-marketing/preise-rabatte/' | relative_url }}">Preise &amp; Rabatte</a>
 │    ├── <a href="{{ '/05-sales-marketing/mahnwesen/' | relative_url }}">Mahnwesen</a>
 │    ├── <a href="{{ '/05-sales-marketing/kunden/' | relative_url }}">Kunden</a>
 │    ├── <a href="{{ '/05-sales-marketing/kampagnen/' | relative_url }}">Kampagnen</a>
 │    └── <a href="{{ '/05-sales-marketing/entwickler/' | relative_url }}">Entwickler-Referenz</a>
 │
 ├── 6.  <a href="{{ '/06-purchasing/' | relative_url }}">Einkauf</a>
 │    ├── <a href="{{ '/06-purchasing/einrichtung/' | relative_url }}">Einrichtung (Tab. 312)</a>
 │    ├── <a href="{{ '/06-purchasing/einkaufsbelege/' | relative_url }}">Einkaufsbelege</a>
 │    ├── <a href="{{ '/06-purchasing/preise-rabatte/' | relative_url }}">Preise &amp; Rabatte</a>
 │    ├── <a href="{{ '/06-purchasing/kreditoren/' | relative_url }}">Kreditoren</a>
 │    ├── <a href="{{ '/06-purchasing/anforderungen/' | relative_url }}">Anforderungen</a>
 │    └── <a href="{{ '/06-purchasing/entwickler/' | relative_url }}">Entwickler-Referenz</a>
 │
 ├── 7.  <a href="{{ '/07-inventory/' | relative_url }}">Lager &amp; Logistik</a>
 │    └── <a href="{{ '/07-inventory/intrastat/' | relative_url }}">Intrastat-Meldung</a>
 │         ├── <a href="{{ '/07-inventory/intrastat/einrichtung/' | relative_url }}">Einrichtung &amp; Setup</a>
 │         ├── <a href="{{ '/07-inventory/intrastat/verarbeitung/' | relative_url }}">Verarbeitung &amp; Hintergrund</a>
 │         ├── <a href="{{ '/07-inventory/intrastat/export/' | relative_url }}">Export &amp; Hochladen</a>
 │         └── <a href="{{ '/07-inventory/intrastat/entwickler/' | relative_url }}">Entwickler-Referenz</a>
 │
 ├── 8.  Produktion  🚧
 │
 ├── 9.  Projekte  🚧
 │
 ├── 10. Service  🚧
 │
 ├── 11. Personal  🚧
 │
 ├── 12. Kontaktmanagement  🚧
 │
 ├── 13. Company Hub  🚧
 │
 ├── 14. Rollencenter &amp; UX  🚧
 │
 ├── 15. APIs &amp; Integration  🚧
 │
 ├── 16. AL-Entwicklung  🚧
 │
 ├── 17. Administration &amp; Betrieb  🚧
 │
 ├── 18. Migration &amp; Upgrade  🚧
 │
 ├── 19. <a href="{{ '/19-todo/' | relative_url }}">ToDo</a>
 │    ├── <a href="{{ '/19-todo/wertefluss/' | relative_url }}">Wertefluss in Business Central</a>
 │    └── <a href="{{ '/19-todo/anlagen-im-bau/' | relative_url }}">Anlagen im Bau</a>
 │
 └── 20. <a href="{{ '/20-faq/' | relative_url }}">FAQ — Ausgewählte Fragen</a>
      ├── <a href="{{ '/20-faq/fachkonzepte/' | relative_url }}">Rabatt vs. Skonto</a>
      ├── <a href="{{ '/20-faq/methodik-quellen/' | relative_url }}">Methodik &amp; Quellen</a>
      └── <a href="{{ '/20-faq/jekyll-struktur/' | relative_url }}">Jekyll-Struktur</a>
</pre>

## Quellen

- [Microsoft Learn — Business Central](https://learn.microsoft.com/dynamics365/business-central/)
- [BCApps Reference](https://microsoft.github.io/BCApps)
- BC28 OnPrem Artifacts (Version 28.1.49838.49886)

> **Hinweis:** Diese Seite ist nicht für Suchmaschinen indiziert (`noindex, nofollow`).
