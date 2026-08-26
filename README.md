# Denkarbeit — Skill-Paket

Marketplace für das Denkarbeit-Plugin: Kontext-System und Arbeits-Skills des Kurses „code your work".

## Installation (Trainee-Flow)

Ein Satz an Claude Code genügt — die Kommandos kennt es selbst:

> Installier mir bitte das Denkarbeit-Plugin von `<QUELLE>`.

`<QUELLE>` ist der Marketplace-Ort:
- produktiv: `marvinkassuehlke/denkarbeit-marketplace` (GitHub)
- lokaler Test: ein Pfad auf einen Klon dieses Repos

Claude liest den Marketplace, prüft die Manifeste und fragt einmal nach Freigabe für
die Installation. Danach kann ein `/reload-plugins` oder ein Neustart der Session
nötig sein, damit die Skills auftauchen — dann `/denkarbeit:setup`.

Wer die Schritte selbst ausführen will: `claude plugin marketplace add <QUELLE>`,
dann `claude plugin install denkarbeit@denkarbeit`. Interaktiv geht auch
`/plugin marketplace add <QUELLE>` → `/plugin install denkarbeit@denkarbeit`.

## Enthaltene Skills

| Skill | Zweck |
|---|---|
| `denkarbeit:setup` | Kursbegleiter — geführtes Ersteinrichten + Gesamtcheck (`/denkarbeit:setup check`) |
| `denkarbeit:contextify` | `context.md` einer Ebene anlegen/pflegen (Backbone-konform) |
| `denkarbeit:remember` | Session-Wissen persistieren: `log.md`-Eintrag, Verdichtung in `memory.md` |
| `denkarbeit:cleanup` | Gewachsenen Workspace-Bereich in die Kontext-Logik überführen |
| `denkarbeit:research` | Web-Recherche → strukturiertes Wissensdokument (`research.md`) |

Die Skills tragen ihre Regeln selbst — es wird keine separate Spezifikation ausgeliefert und kein Pfad in dieses Plugin in eine fremde `CLAUDE.md` geschrieben.

## Update

- Git-gehosteter Marketplace: Plugins aktualisieren sich automatisch.
- Lokaler Pfad (Test): explizit `claude plugin marketplace update denkarbeit` ausführen.

## Pflege

Dieses Repo ist **Auslieferung, kein Master.** Master ist das Kurs-Repo (`repo/skills/`); publiziert wird mit `repo/tools/publish_marketplace.sh`, das dabei die Skill-Trigger auf `/denkarbeit:*` qualifiziert. Manifeste (`.claude-plugin/`) und dieses README werden manuell gepflegt.
