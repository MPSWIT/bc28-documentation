---
title: "FAQ — Jekyll-Struktur"
---
# 19.3 Jekyll: Flache Dateien vs. Unterordner

> **TL;DR:** `thema/index.md` erzeugt saubere URLs (`/04-finance/fibu-einrichtung/`) ohne `.html`-Endung. Das ist das native Jekyll-Muster.

> 📄 **[← FAQ-Übersicht]({{ '/19-faq/' | relative_url }})**

---

## Die Frage

> *„Was passiert, wenn du im 04-finance Ordner Unterordner anlegst? Und in den Unterordnern die Doku machst?"*

## Vorher vs. Nachher

```
VORHER (flach):                          NACHHER (Unterordner):
04-finance/                              04-finance/
├── index.md          ← Hub              ├── index.md
├── fibu-einrichtung.md                  ├── fibu-einrichtung/
├── mwst-system.md                       │   └── index.md
└── entwickler.md                        ├── mwst-system/
                                         │   └── index.md
                                         └── entwickler/
                                             └── index.md
```

### Auswirkung auf URLs

| Datei | Flache URL | Unterordner-URL |
|---|---|---|
| `fibu-einrichtung.md` | `/04-finance/fibu-einrichtung.html` | `/04-finance/fibu-einrichtung/` |
| `mwst-system.md` | `/04-finance/mwst-system.html` | `/04-finance/mwst-system/` |

> **Jekyll-Konvention:** Ein Ordner mit `index.md` wird als Seite behandelt.
> Die URL ist der Ordnername — ohne `.html`, ohne `.md`.

### Link-Syntax (Liquid)

```liquid
<!-- Innerhalb eines Kapitels: absoluter Liquid-Pfad -->
[Nächstes Thema]({{ '/04-finance/mwst-system/' | relative_url }})

<!-- Zurück zum Hub -->
[← Übersicht]({{ '/04-finance/' | relative_url }})

<!-- Kapitelübergreifend -->
[→ Nächstes Kapitel]({{ '/05-sales-marketing/' | relative_url }})
```

`relative_url` sorgt dafür, dass die Pfade auch unter `mpswit.github.io/bc28-documentation/` korrekt aufgelöst werden.

### Warum der Umbau?

1. **Saubere URLs** — `/fibu-einrichtung/` statt `/fibu-einrichtung.html`
2. **Erweiterbarkeit** — pro Thema können später Unter-Unterseiten angelegt werden
3. **Jekyll-native Struktur** — GitHub Pages erwartet dieses Muster
4. **Bessere Navigation** — Breadcrumbs und Sidebars arbeiten damit

### Umbau-Methode

Ein Python-Skript hat alle 24 `.md`-Dateien der Kapitel 1, 4, 5, 6 automatisch umgezogen:

```python
# Pro Subpage:
# 1. Ordner anlegen (z.B. 04-finance/fibu-einrichtung/)
# 2. Datei verschieben → index.md
# 3. Alle relativen Links umschreiben → absolute Liquid-Pfade
# 4. Hub-index.md neu verlinken
```

---

| [← FAQ-Übersicht]({{ '/19-faq/' | relative_url }}) | [→ Zurück zur Startseite]({{ '/index' | relative_url }}) |
