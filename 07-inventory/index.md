---
title: "Lager & Logistik"
---
# 7. Lager &amp; Logistik

<pre>
7. Lager &amp; Logistik
 │
 ├─▶ Übersicht  ← Sie sind hier
 │
 ├── <a href="{{ '/07-inventory/intrastat/' | relative_url }}">Intrastat-Meldung</a>
 │    ├── <a href="{{ '/07-inventory/intrastat/einrichtung/' | relative_url }}">Einrichtung &amp; Setup</a>
 │    ├── <a href="{{ '/07-inventory/intrastat/verarbeitung/' | relative_url }}">Verarbeitung &amp; Hintergrund</a>
 │    ├── <a href="{{ '/07-inventory/intrastat/export/' | relative_url }}">Export &amp; Hochladen</a>
 │    └── <a href="{{ '/07-inventory/intrastat/entwickler/' | relative_url }}">Entwickler-Referenz</a>
 │
 └── (weitere Themen folgen)
</pre>

---

## Intrastat-Meldung

Die Intrastat-Meldung ist eine EU-weite Statistikpflicht für den grenzüberschreitenden Warenverkehr innerhalb der EU. Business Central 28 stellt ein vollständiges Modul (App `Intrastat Core`, ID `70912191-3c4c-49fc-a1de-bc6ea1ac9da6`, Objekte 4810–4840) zur Verfügung, das folgende Bereiche abdeckt:

- **Einrichtung:** Setup-Tabelle 4810, Stammdatenerweiterungen (Item, Vendor, Customer, Fixed Asset, Country/Region)
- **Verarbeitung:** Automatische Zeilenermittlung aus Item/Job/FA Ledger Entries über Report 4810
- **Validierung:** Konfigurierbare Checkliste (Pflichtfelder, bedingte Felder)
- **Export:** Data-Exchange-Framework mit konfigurierbarem Format (Fixed-Width, CSV, etc.)
- **Ländererweiterungen:** Plugin-Architektur über Event-Subscriber (z.B. Intrastat AT)

→ [Intrastat-Meldung — vollständige Dokumentation]({{ '/07-inventory/intrastat/' | relative_url }})
