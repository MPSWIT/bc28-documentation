# BC28 Documentation — Todo List

> Business Central 28 — vollständige technische Dokumentation der App-Architektur, Module, Namespaces und AL-Objekte.
> Basis: BC28 OnPrem Artifacts (28.1.49838.49886), Microsoft Learn, und BCApps-Referenz.

---

## 1. Überblick & Architektur
- [x] 1.1 Einleitung: Was ist neu in BC28?
- [x] 1.2 App-Architektur: System Application, Base Application, Business Foundation, Erweiterungen
- [x] 1.3 AL-Sprachfeatures in BC28 (Übersicht)
- [x] 1.4 OnPrem vs. SaaS: Bereitstellungsmodelle

## 2. System Application
- [ ] 2.1 Übersicht aller Module (app.json-Struktur, Abhängigkeiten)
- [ ] 2.2 Guided Experience
- [ ] 2.3 Performance Profiler
- [ ] 2.4 Environment Information
- [ ] 2.5 Word Templates
- [ ] 2.6 Weitere System-Module (Azure AD, Encryption, etc.)

## 3. Business Foundation
- [ ] 3.1 Übersicht & Modulstruktur
- [ ] 3.2 No. Series (Nummernserien)
- [ ] 3.3 Dimensionen & Analyseansichten
- [ ] 3.4 Workflow-Engine
- [ ] 3.5 Weitere Foundation-Module

## 4. Base Application — Finance (Finanzwesen)
- [x] 4.1 General Ledger Setup — 22 Felder mit je 3 User-Story-Beispielen
- [x] 4.2 Kontenplan & Sachkonten (Tabelle 15, GLAccount)
- [x] 4.3 Buchungsgruppen — Gen. Posting Setup, die Buchungsmatrix
- [x] 4.4 MwSt & Umsatzsteuer (VAT System)
- [x] 4.5 Fibu-Buchungsjournale
- [x] 4.6 Debitoren & Kreditoren (Receivables & Payables)
- [x] 4.7 Bank & Zahlungsverkehr (SEPA, Bankabstimmung)
- [x] 4.8 Anlagen (Fixed Assets)
- [x] 4.9 Währung & Wechselkurse
- [x] 4.10 Finanzberichte & Analyse (Account Schedules)
- [x] 4.11 Budget
- [x] 4.12 Konsolidierung
- [x] 4.13 Abgrenzungen (Deferrals)
- [x] 4.14 Intercompany (IC-Partner)
- [x] 4.15 Fibu-Relevanz anderer Module (Querschnitt Einkauf/Verkauf/Lager/Produktion/Service/Projekte/HR)
- [x] 4.16 Integrationsereignisse & Entwickler-Referenz

## 5. Base Application — Sales & Marketing
- [x] 5.1 Sales & Receivables Setup — 73 Felder mit de-AT-Übersetzungen
- [x] 5.2 Sales Documents (Angebot, Auftrag, Rechnung, Gutschrift, Retoure)
- [x] 5.3 Sales Pricing & Discounts (Preislisten, Zeilen-, Rechnungsrabatte)
- [x] 5.4 Reminder / Mahnwesen (Mahnmethoden, -stufen, Zinsrechnung)
- [x] 5.5 Customer Management (Debitorenkarte, Buchungsgruppen, OP-Verwaltung)
- [x] 5.6 Campaigns & Segments
- [x] 5.7 Entwickler-Referenz (Integrationsereignisse, Codeunits, Tabellen)

## 6. Base Application — Purchasing (Einkauf)
- [x] 6.1 Purchases & Payables Setup — 80+ Felder mit de-AT-Übersetzungen
- [x] 6.2 Purchase Documents (Anfrage, Bestellung, Wareneingang, Rechnung)
- [x] 6.3 Purchase Pricing & Discounts
- [x] 6.4 Vendor Management (Kreditorenkarte, OP-Verwaltung)
- [x] 6.5 Requisition (Einkaufsanforderungen)
- [x] 6.6 Entwickler-Referenz (Integrationsereignisse, Codeunits, Tabellen)

## 7. Base Application — Inventory (Lager)
- [ ] 7.1 Item Management (Artikel)
- [ ] 7.2 Warehousing (Lagerlogistik: Einlagerung, Kommissionierung, Warenausgang)
- [ ] 7.3 Assembly (Montage)
- [ ] 7.4 Item Tracking (Chargen/Seriennummern)
- [ ] 7.5 Inventory Costing (Lagernachkalkulation)

## 8. Base Application — Manufacturing (Produktion)
- [ ] 8.1 Production BOM (Stücklisten)
- [ ] 8.2 Routings (Arbeitspläne)
- [ ] 8.3 Production Orders (Fertigungsaufträge)
- [ ] 8.4 Capacity Planning (Kapazitätsplanung)
- [ ] 8.5 Shop Floor / MES-Integration

## 9. Base Application — Jobs / Projekte
- [ ] 9.1 Job Management (Projektverwaltung)
- [ ] 9.2 Job Planning & Budgeting
- [ ] 9.3 Resource Management (Ressourcen)

## 10. Base Application — Service
- [ ] 10.1 Service Management (Serviceverträge, Serviceaufträge)
- [ ] 10.2 Service Items & BOM

## 11. Base Application — HR / Personal
- [ ] 11.1 Employee Management
- [ ] 11.2 Absence Management

## 12. Base Application — CRM / Kontaktmanagement
- [ ] 12.1 Contacts (Kontakte)
- [ ] 12.2 Interactions (Interaktionen)
- [ ] 12.3 Opportunities (Verkaufschancen)

## 13. Company Hub
- [ ] 13.1 COHUB Architektur & Entitlements
- [ ] 13.2 Companies Overview
- [ ] 13.3 User Tasks & Delegation

## 14. Role Centers & UX
- [ ] 14.1 Business Manager Role Center
- [ ] 14.2 Weitere Rollencenter (Buchhalter, Lagermitarbeiter, etc.)
- [ ] 14.3 Profiles & Personalisierung
- [ ] 14.4 Pages & UI Patterns

## 15. APIs & Integration
- [ ] 15.1 REST APIs (Standard-APIs, Custom APIs)
- [ ] 15.2 OData / SOAP Web Services
- [ ] 15.3 System-Integration-Events (Event-Driven Architecture)
- [ ] 15.4 Power Platform / Power Automate-Integration
- [ ] 15.5 Azure Services Integration

## 16. AL Development Guide
- [ ] 16.1 AL-Syntax-Referenz (Types, Procedures, Triggers)
- [ ] 16.2 Event-Modell (Business-, Integration-, Database-Trigger-Events)
- [ ] 16.3 Extension-Entwicklung (TableExtension, PageExtension, etc.)
- [ ] 16.4 Testing (Test Codeunits, Test Runner)
- [ ] 16.5 Debugging & Performance Profiling
- [ ] 16.6 Upgrade-Code (Upgrade-Triggers, Datenmigration)

## 17. Administration & Betrieb
- [ ] 17.1 OnPrem-Installation & Konfiguration
- [ ] 17.2 SaaS-Administration (Admin Center)
- [ ] 17.3 User & Permission Management
- [ ] 17.4 Datenbank-Konfiguration & -Wartung
- [ ] 17.5 Monitoring & Telemetrie

## 18. Migration & Upgrade
- [ ] 18.1 Upgrade von BC26/27 auf BC28
- [ ] 18.2 Breaking Changes in BC28
- [ ] 18.3 Datenmigration (RapidStart, Configuration Packages)

## 19. ToDo
- [x] 19.1 Wertefluss in Business Central (Posten-Ökosystem, Buchungskette, Buchungsmatrix)
- [ ] Weitere Themen folgen

## 20. FAQ — Ausgewählte Fragen
- [x] 20.1 Rabatt vs. Skonto (Fachkonzept)
- [x] 20.2 Methodik & Quellen (XLF, RAG, Quellcode-Verifikation)
- [x] 20.3 Jekyll-Struktur (flache Dateien vs. Unterordner)

---

**Status:** 🟢 5/20 Kapitel abgeschlossen (1, 4, 5, 6, 20) — alle mit Subpages  
**Quellen:** BC28 OnPrem Artifacts (`~/ai/onprem/28.1.49838.49886/at/`), [BCApps Reference](https://microsoft.github.io/BCApps), [Microsoft Learn](https://learn.microsoft.com/dynamics365/business-central/)
