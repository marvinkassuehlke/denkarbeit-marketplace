---
name: setup
description: Kursbegleiter — geführtes Ersteinrichten des Denkarbeit-Setups (Umgebung, CLAUDE.md, L1-Kontext, Sprachanker, CI) und Gesamtcheck. Triggers on /setup, „Setup starten", „richte mein Setup ein", „hab ich alles konfiguriert?".
---

## Zweck

Der Kursbegleiter richtet das Denkarbeit-Setup ein und prüft es. Ein Mechanismus, zwei Aufrufe:

- `/setup` — geführtes Einrichten: erst messen, dann fehlende Bausteine im Dialog schließen.
- `/setup check` — nur messen und ausweisen, nichts verändern.

**Setup = Check + Lücken schließen.** Dadurch ist der Skill resumierbar: Jeder Aufruf beginnt mit derselben Messung; erledigte Bausteine werden erkannt und übersprungen. Es gibt kein Status-File — der Zustand wird aus den Artefakten selbst abgeleitet (Kein-Status-Prinzip). Einzige Ausnahme: bewusste Skips stehen als Zeile in der L1-`context.md` unter „Offene Punkte" — sonst wären sie von „vergessen" nicht unterscheidbar.

**Scope-Linie:** Der Begleiter endet bei „System steht leer und korrekt konfiguriert". Die erste Domäne anlegen und befüllen ist Kursinhalt, nicht Setup.

Die Spezifikation des Kontext-Systems liegt unter `../../context_system/design.md` (mit `template_context.md` daneben); die Templates dieses Skills unter `templates/` relativ zum Skill-Basisverzeichnis.

## Bausteine & Messung

**Mess-Pflicht:** Vor jedem Handeln alle Bausteine mechanisch erheben — nicht schätzen, nicht aus dem Gedächtnis. Eine Prüfung ohne erhobenen Befund hat nicht stattgefunden.

| # | Baustein | Messung | Status |
|---|---|---|---|
| 0 | Umgebung | `git --version` läuft · Workspace-Ordner existiert · `{workspace}/.obsidian/` vorhanden (Vault zeigt auf den Workspace) | Pflicht |
| 1 | CLAUDE.md | `~/.claude/CLAUDE.md` existiert und enthält Workspace-Konvention + Session-Start-Regel + Pointer auf Spec und Sprachanker | Pflicht |
| 2 | L1-Kontext | `{workspace}/context.md` existiert mit Backbone-Kopf (Steckbrief, Motivation) | Pflicht |
| 3 | Sprachanker | `{workspace}/sprache.md` existiert (der Default zählt) | Pflicht — Default garantiert |
| 4 | CI / Gestaltung | `{workspace}/brand.yaml` existiert | Optional |
| 5 | Bestand | Workspace war beim Start nicht leer: Ordner/Dateien ohne Ebenen-Logik? | Situativ |

Workspace-Pfad finden: aus der CLAUDE.md (Baustein 1 verankert ihn). Fehlt die CLAUDE.md, den User fragen bzw. bei Neueinrichtung `~/workspace` vorschlagen.

## Workflow

1. **Modus bestimmen.** `check` im Aufruf → nur Schritte 2 und 5. Sonst geführtes Setup.

2. **Messen.** Alle Bausteine der Tabelle mechanisch erheben. Skips erkennen: Einträge „… übersprungen" in L1 „Offene Punkte".

3. **Lücken schließen** (nur fehlende Bausteine, in Reihenfolge):

   **B0 — Umgebung.**
   - git fehlt: macOS → `git --version` löst den Ein-Klick-Installationsdialog aus; Windows → `winget install Git.Git`.
   - Kommando nicht gefunden trotz Installation: Der Regelfall-Fix ist die PATH-Zeile im Shell-Profil — wörtlich anleiten, nicht diagnostizieren lassen.
   - Workspace-Ordner anlegen (Vorschlag `~/workspace`, User bestätigt).
   - Obsidian: Vault auf den Workspace-Ordner öffnen („Open folder as vault") — prüfbar an `{workspace}/.obsidian/`.

   **B1 — CLAUDE.md.** Aus `templates/CLAUDE_template.md` erzeugen; Platzhalter füllen: `{{NAME}}`, `{{WORKSPACE}}` (absoluter Pfad), `{{SPEC_PATH}}` (absoluter Pfad zur design.md — vom Skill-Basisverzeichnis aus auflösen). **Existiert bereits eine CLAUDE.md: nie überschreiben** — fehlende Blöcke ergänzen vorschlagen, Konflikte flaggen.

   **B2 — L1-Kontext (Mini-Interview).** Das Interview ist der Primärweg, Dokumente sind Beschleuniger — es gibt keinen „keine Quellen"-Fall. 5–7 Fragen: Rolle und Organisation · für wen/was gearbeitet wird (Domänen, Stakeholder) · Kern-Arbeitsinhalte · laufende Vorhaben · Ziel mit dem Setup. Quellen, falls vorhanden: CV-PDF · LinkedIn-Export (im Profil „Als PDF speichern" — URLs scheitern oft an der Login-Wall) · **ein selbstverfasstes Arbeitsdokument** (die wertvollste Quelle: verrät Rolle und Sprache, füttert auch B3). L1 braucht Gegenwart, nicht Werdegang — CV-Vergangenheit nur als Steckbrief-Hintergrund. Die Zielgruppe weiß, was sie will: Interview kurz halten, keine Use-Case-Findung. Dann `{workspace}/context.md` gegen das Backbone der Spezifikation erzeugen (contextify-Logik; die Regeln stehen dort, nicht hier).

   **B3 — Sprachanker.** Fehlt `{workspace}/sprache.md`: `templates/sprache_default.md` dorthin kopieren, Frontmatter-Daten auf heute setzen — **der Anker existiert danach immer.** Personalisierung anbieten: Merkmale aus einem eigenen Arbeitsdokument ableiten und ergänzen, oder Default belassen (beides legitim).

   **B4 — CI.** Kurz-Interview (Farben, Schrift, Logo-Pfad) → `{workspace}/brand.yaml`. Skip ist legitim → Zeile in L1 „Offene Punkte": „CI-Konfiguration (brand.yaml) übersprungen — bei Bedarf `/setup` erneut."

   **B5 — Bestand.** War der Workspace nicht leer: Befund erheben (Ordnerzahl, lose Dateien, erkennbare Ebenen-Träger — Erkennungskette Spec §1; Repos/Symlinks mechanisch: `find -name .git`, `find -type l`) und **an `/cleanup` übergeben** — der hat Plan-Gate und Safety-Gates. Der Begleiter räumt nie selbst auf (keine Duplikation der cleanup-Logik).

4. **Skips festhalten.** Jeden bewusst übersprungenen Baustein als Zeile in L1 „Offene Punkte" vermerken.

5. **Ausweis (Pflicht, nie weglassen).** Tabelle aller Bausteine mit genau einem Status je Zeile: **✓** konfiguriert · **○** offen (optional) · **–** bewusst übersprungen · **✗** fehlt (Pflicht). Dazu die Plugin-Version (aus der plugin.json des Plugins, falls als Plugin installiert). Ohne diese Tabelle gilt der Lauf als nicht geprüft. Im Check-Modus zusätzlich: Spec-Pointer in der CLAUDE.md verifizieren (Pfad existiert?) — bei totem Pointer neu verankern.

## Harte Gates

- Messen vor Handeln; Ausweis-Tabelle ist Pflicht.
- Bestehende Dateien (CLAUDE.md, context.md, sprache.md, brand.yaml) nie überschreiben — ergänzen oder flaggen.
- Kein Aufräumen im Bestand — an `/cleanup` delegieren.
- Interview kurz, keine Use-Case-Findung, kein Psychogramm (Personen-Regeln der Spec gelten).
- Skips in L1 „Offene Punkte" festhalten.
- Scope-Ende: keine Domänen anlegen oder befüllen — das ist Kursinhalt.
