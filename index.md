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

| Kapitel | Inhalt |
|---------|--------|
| [1. Überblick & Architektur](01-overview/) | App-Modell, Abhängigkeiten, AL-Sprachfeatures |
| [2. System Application](02-system-application/) | Plattform-Module (Azure AD, Encryption, Telemetry…) |
| [3. Business Foundation](03-business-foundation/) | No. Series, Dimensionen, Workflow |
| [4. Finanzwesen](04-finance/) | Sachkonten, Kreditoren, Debitoren, Anlagen, MwSt |
| [5. Vertrieb & Marketing](05-sales-marketing/) | Verkaufsbelege, Preise, Mahnwesen |
| [6. Einkauf](06-purchasing/) | Einkaufsbelege, Lieferanten |
| [7. Lager & Logistik](07-inventory/) | Artikel, Lagerlogistik, Montage, Nachkalkulation |
| [8. Produktion](08-manufacturing/) | Stücklisten, Arbeitspläne, Fertigungsaufträge |
| [9. Projekte](09-jobs/) | Projektverwaltung, Ressourcen |
| [10. Service](10-service/) | Serviceverträge, Serviceaufträge |
| [11. Personal](11-hr/) | Mitarbeiter, Abwesenheiten |
| [12. Kontaktmanagement](12-crm/) | Kontakte, Interaktionen, Verkaufschancen |
| [13. Company Hub](13-company-hub/) | Mandantenübergreifende Verwaltung |
| [14. Rollencenter & UX](14-role-centers-ux/) | Rollencenter, Pages, UI Patterns |
| [15. APIs & Integration](15-apis-integration/) | REST, OData, Power Platform |
| [16. AL-Entwicklung](16-al-development/) | AL-Syntax, Events, Extensions, Testing |
| [17. Administration & Betrieb](17-administration/) | Installation, Benutzer, Monitoring |
| [18. Migration & Upgrade](18-migration-upgrade/) | Upgrade-Pfade, Breaking Changes |

## Quellen

- [Microsoft Learn — Business Central](https://learn.microsoft.com/dynamics365/business-central/)
- [BCApps Reference](https://microsoft.github.io/BCApps)
- BC28 OnPrem Artifacts (Version 28.1.49838.49886)

> **Hinweis:** Diese Seite ist nicht für Suchmaschinen indiziert (`noindex, nofollow`).
