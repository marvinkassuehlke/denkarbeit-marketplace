# Denkarbeit — Skill-Paket

Marketplace für das Denkarbeit-Plugin: Kontext-System und Arbeits-Skills des Kurses „code your work".

## Installation (Trainee-Flow)

Ein Prompt an Claude Code genügt:

> Installiere bitte das Denkarbeit-Plugin: führe `claude plugin marketplace add <QUELLE>` aus und danach `claude plugin install denkarbeit@denkarbeit`. Starte anschließend das Setup mit /denkarbeit:setup

`<QUELLE>` ist der Marketplace-Ort:
- produktiv: `marvinkassuehlke/denkarbeit-marketplace` (GitHub)
- lokaler Test: `/Users/Shared/denkarbeit-marketplace`

Alternativ interaktiv in Claude Code: `/plugin marketplace add <QUELLE>` → `/plugin install denkarbeit@denkarbeit` → ggf. `/reload-plugins`.

## Enthaltene Skills

| Skill | Zweck |
|---|---|
| `denkarbeit:setup` | Kursbegleiter — geführtes Ersteinrichten + Gesamtcheck (`/denkarbeit:setup check`) |
| `denkarbeit:contextify` | `context.md` einer Ebene anlegen/pflegen (Backbone-konform) |
| `denkarbeit:remember` | Session-Wissen persistieren: `log.md`-Eintrag, Verdichtung in `memory.md` |
| `denkarbeit:cleanup` | Gewachsenen Workspace-Bereich in die Kontext-Logik überführen |
| `denkarbeit:research` | Web-Recherche → strukturiertes Wissensdokument (`research.md`) |

Die Spezifikation des Kontext-Systems liegt plugin-intern unter `plugins/denkarbeit/context_system/`.

## Update

- Git-gehosteter Marketplace: Plugins aktualisieren sich automatisch.
- Lokaler Pfad (Test): explizit `claude plugin marketplace update denkarbeit` ausführen.

## Pflege

Dieses Repo ist **Auslieferung, kein Master.** Master ist das Kurs-Repo (`repo/skills/` + `context_system/`); publiziert wird mit `repo/tools/publish_marketplace.sh`. Manifeste (`.claude-plugin/`) und dieses README werden manuell gepflegt.
