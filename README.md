# Business Central 28 — Technische Dokumentation

> Vollständige technische Dokumentation der Microsoft Dynamics 365 Business Central 28 App-Architektur.

Basierend auf den BC28 OnPrem Artifacts (Version 28.1.49838.49886) dokumentiert dieses Repository:
- **System Application** — Plattform-Module (Azure AD, Encryption, Telemetry, etc.)
- **Business Foundation** — Grundlegende Business-Logik (No. Series, Dimensions, Workflow)
- **Base Application** — Fachliche Module (Finance, Sales, Purchasing, Inventory, Manufacturing, etc.)
- **Company Hub** — Mandantenübergreifende Verwaltung
- **AL Development Guide** — Best Practices, Event-Modell, Testing
- **APIs & Integration** — REST, OData, Power Platform

## Struktur

```
bc28-documentation/
├── README.md
├── TODO.md                     # Kapitel-Plan & Fortschritt
├── 01-overview/                # Überblick & Architektur
├── 02-system-application/      # System Application Module
├── 03-business-foundation/     # Business Foundation
├── 04-finance/                 # Finanzwesen
├── 05-sales-marketing/         # Vertrieb & Marketing
├── 06-purchasing/              # Einkauf
├── 07-inventory/               # Lager & Logistik
├── 08-manufacturing/           # Produktion
├── 09-jobs/                    # Projekte
├── 10-service/                 # Service
├── 11-hr/                      # Personal
├── 12-crm/                     # Kontaktmanagement
├── 13-company-hub/             # Company Hub
├── 14-role-centers-ux/         # Rollencenter & UX
├── 15-apis-integration/        # APIs & Integration
├── 16-al-development/          # AL-Entwicklung
├── 17-administration/          # Administration & Betrieb
└── 18-migration-upgrade/       # Migration & Upgrade
```

## Quellen

- [Microsoft Learn — Business Central](https://learn.microsoft.com/dynamics365/business-central/)
- [BCApps Reference](https://microsoft.github.io/BCApps)
- BC28 OnPrem Artifacts (28.1.49838.49886)
