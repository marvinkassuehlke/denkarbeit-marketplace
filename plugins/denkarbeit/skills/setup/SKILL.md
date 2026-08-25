---
name: setup
description: Der Begleiter — geführtes Ersteinrichten des Denkarbeit-Setups (Umgebung, CLAUDE.md, L1-Kontext, Sprachanker, CI, Systeme & Secrets) und Gesamtcheck. Triggers on /setup, „Setup starten", „richte mein Setup ein", „hab ich alles konfiguriert?".
---

## Zweck

Der Begleiter richtet das Denkarbeit-Setup ein und prüft es — für Kurs-Trainees wie für Gewerbe-Kunden (dort ist die Check-Tabelle zugleich das Abnahme-Dokument). Ein Mechanismus, zwei Aufrufe:

- `/setup` — geführtes Einrichten: erst messen, dann fehlende Bausteine im Dialog schließen.
- `/setup check` — nur messen und ausweisen, nichts verändern.

**Setup = Check + Lücken schließen.** Dadurch ist der Skill resumierbar: Jeder Aufruf beginnt mit derselben Messung; erledigte Bausteine werden erkannt und übersprungen. Es gibt kein Status-File — der Zustand wird aus den Artefakten selbst abgeleitet (Kein-Status-Prinzip). Einzige Ausnahme: bewusste Skips stehen als Zeile in der L1-`context.md` unter „Offene Punkte" — sonst wären sie von „vergessen" nicht unterscheidbar.

**Scope-Linie:** Der Begleiter endet bei „System steht leer und korrekt konfiguriert". Die erste Domäne anlegen und befüllen ist Kursinhalt, nicht Setup.

Die Spezifikation des Kontext-Systems liegt unter `../../context_system/design.md` relativ zum Skill-Basisverzeichnis (mit `template_context.md` daneben); existiert bereits eine globale CLAUDE.md mit Referenzen-Sektion, gilt deren Spec-Pfad. Die Templates dieses Skills liegen unter `templates/` relativ zum Skill-Basisverzeichnis.

## Bausteine & Messung

**Mess-Pflicht:** Vor jedem Handeln alle Bausteine mechanisch erheben — nicht schätzen, nicht aus dem Gedächtnis. Eine Prüfung ohne erhobenen Befund hat nicht stattgefunden.

| # | Baustein | Messung | Status |
|---|---|---|---|
| 0 | Umgebung | `git --version` läuft · Workspace-Ordner existiert · `{workspace}/.obsidian/` vorhanden (Vault zeigt auf den Workspace) | Pflicht |
| 1 | CLAUDE.md | `~/.claude/CLAUDE.md` existiert und enthält Workspace-Konvention + Session-Start-Regel + Pointer auf Spec und Sprachanker | Pflicht |
| 2 | L1-Kontext | `{workspace}/context.md` existiert mit Backbone-Kopf (Steckbrief, Motivation) | Pflicht |
| 3 | Sprache | Sprachregeln in der CLAUDE.md vorhanden (Baustein 1); `{workspace}/language.md` **optional**, nur mit geeignetem Referenzmaterial | Pflicht (Regeln) · optional (Referenz) |
| 4 | CI / Gestaltung | `{workspace}/brand.yaml` existiert | Optional |
| 5 | Bestand | Workspace war beim Start nicht leer: Ordner/Dateien ohne Ebenen-Logik? | Situativ |
| 6 | Systeme & Secrets | Nutzer arbeitet gegen APIs/CLIs/MCP-Server: `infra.md` der betroffenen Ebene existiert · ein Vault ist benannt (Passwort-Manager-CLI oder OS-Schlüsselbund) · CLAUDE.md trägt den Secrets-Block | Situativ — Pflicht, sobald zutreffend |

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

   **B1 — CLAUDE.md.** Aus `templates/CLAUDE_template.md` erzeugen; Platzhalter füllen: `{{NAME}}`, `{{WORKSPACE}}` (absoluter Pfad), `{{SPEC_PATH}}` (absoluter Pfad zur design.md — vom Skill-Basisverzeichnis aus auflösen). **Existiert bereits eine CLAUDE.md: nie überschreiben** — fehlende Blöcke ergänzen vorschlagen, Konflikte flaggen. Ergänzungen landen als **ein** zusammenhängender, markierter Block (`<!-- denkarbeit:begin -->` … `<!-- denkarbeit:end -->`), damit sie neben einer bestehenden Fremd-Konfiguration identifizierbar und rückbaubar bleiben; in fremde Blöcke wird nie hineingeschrieben.

   **B2 — L1-Kontext (Mini-Interview).** Das Interview ist der Primärweg, Dokumente sind Beschleuniger — es gibt keinen „keine Quellen"-Fall. 5–7 Fragen: Rolle und Organisation · für wen/was gearbeitet wird (Domänen, Stakeholder) · Kern-Arbeitsinhalte · laufende Vorhaben · Ziel mit dem Setup. Quellen, falls vorhanden: CV-PDF · LinkedIn-Export (im Profil „Als PDF speichern" — URLs scheitern oft an der Login-Wall) · **ein selbstverfasstes Arbeitsdokument** (die wertvollste Quelle: verrät Rolle und Sprache, füttert auch B3). L1 braucht Gegenwart, nicht Werdegang — CV-Vergangenheit nur als Steckbrief-Hintergrund. Die Zielgruppe weiß, was sie will: Interview kurz halten, keine Use-Case-Findung. Dann `{workspace}/context.md` gegen das Backbone der Spezifikation erzeugen (contextify-Logik; die Regeln stehen dort, nicht hier).

   **B3 — Sprach-Referenz (optional, bewusst kein Default-Artefakt).** Die Sprachregeln stehen in der CLAUDE.md (Baustein 1) und tragen für sich. Eine zusätzliche `{workspace}/language.md` lohnt nur mit **gutem Referenzmaterial** — ohne das entsteht ein Artefakt, das den Ton verschlechtert statt ihn zu schärfen. Deshalb: Angebot machen, Kriterien nennen, Nein akzeptieren (kein Skip-Vermerk nötig).

   Frage: „Hast du längere Texte, die du **selbst geschrieben** hast und für gut hältst?" Dann die Eignung klären:
   - **Geeignet:** eigene Autorenschaft (ein LLM darf geglättet, nicht formuliert haben — sonst imitiert das Modell sich selbst) · erklärende Sachprosa für ein Publikum · ein Autor, konsistente Handschrift · redigierte oder veröffentlichte Fassungen · Gesamtumfang etwa 1.500 bis 8.000 Wörter (mehr kostet Kontext ohne Zusatznutzen).
   - **Nicht geeignet:** Mails, Chat-Verläufe, Notizen (zu kurz und situativ, keine Satzarchitektur) · LLM-Entwürfe mit Politur · Sammlungen mehrerer Autoren (mitteln sich zum Durchschnitt, und der Durchschnitt ist genau der Maschinen-Ton) · reine Fachdokumentation · Übersetzungen.
   - **Nichts davon vorhanden:** kein Artefakt anlegen. Die CLAUDE.md-Regeln gelten ohnehin; die Referenz kann jederzeit später entstehen.

   Bei Zusage: Die stärksten Absätze **kuratieren**, nicht die Dateien kopieren, und mit einer Fallenliste (beobachtete Drifts mit Gegenbeispiel) in `language.md` ablegen. Aufbau: Referenzmaterial zum Imitieren, Fallenliste, Pflegehinweis — **keine Stilvorschriften.** Positive Stilregeln werden von Modellen als Checkliste abgearbeitet und übererfüllt; was im Original zweimal trägt, steht dann in jedem Absatz.

   **B4 — CI.** Kurz-Interview (Farben, Schrift, Logo-Pfad) → `{workspace}/brand.yaml`. Skip ist legitim → Zeile in L1 „Offene Punkte": „CI-Konfiguration (brand.yaml) übersprungen — bei Bedarf `/setup` erneut."

   **B5 — Bestand.** War der Workspace nicht leer: Befund erheben (Ordnerzahl, lose Dateien, erkennbare Ebenen-Träger — Erkennungskette Spec §1; Repos/Symlinks mechanisch: `find -name .git`, `find -type l`) und **an `/cleanup` übergeben** — der hat Plan-Gate und Safety-Gates. Der Begleiter räumt nie selbst auf (keine Duplikation der cleanup-Logik).

   **B6 — Systeme & Secrets (situativ; der Gewerbe-Regelfall).** Eine Interview-Frage: „Wird dein Claude mit Systemen arbeiten — APIs, Datenbanken, eigene Dienste, MCP-Server?" Wenn nein: kein Baustein, kein Skip-Vermerk. Wenn ja:
   1. **Vault klären** (Staffel, kein Tool-Zwang): vorhandener Passwort-Manager mit CLI (1Password `op` · Bitwarden `bw` · KeePassXC `keepassxc-cli`) → sonst der **OS-Schlüsselbund** (macOS Keychain via `security` · Windows Credential Manager/DPAPI via PowerShell) — der ist immer da und ist der Default-Fallback. Eine Klartext-Datei ist **kein** Fallback. Laufzeit-Muster einmal zeigen: Secret aus dem Vault in die Umgebung der Session laden, nie in eine Datei.
   2. **`infra.md`-Gerüst anlegen** — auf der Ebene, zu der das System gehört (Dienst-Infra; Spec §3), mit den Sektionen *System & Zugang* (Verweis-Muster: Vault + Eintragsname + Zugriffs-Kommando — nie der Wert), *Eigenheiten*, *Fehlerbilder & Fixes*. Maschinen-Infra (Tools, MCP-Konfig) → `~/.claude/reference/infra.md`.
   3. **Secrets-Block der CLAUDE.md** verifizieren (Baustein 1 legt ihn aus dem Template an; bei Alt-Installationen fehlt er → ergänzen vorschlagen).
   Liegen bereits Zugriffs-Praktiken in flüchtigen Notizen (Scratchpad, temp.md, Chat-Verläufe), werden sie in die `infra.md` **geerntet** — das ist der häufigste Alt-Fall.

4. **Skips festhalten.** Jeden bewusst übersprungenen Baustein als Zeile in L1 „Offene Punkte" vermerken.

5. **Ausweis (Pflicht, nie weglassen).** Tabelle aller Bausteine mit genau einem Status je Zeile: **✓** konfiguriert · **○** offen (optional) · **–** bewusst übersprungen · **✗** fehlt (Pflicht). Dazu die Plugin-Version (aus der plugin.json des Plugins, falls als Plugin installiert). Ohne diese Tabelle gilt der Lauf als nicht geprüft. Im Check-Modus zusätzlich: Spec-Pointer in der CLAUDE.md verifizieren (Pfad existiert?) — bei totem Pointer neu verankern.

6. **Skill-Inventar (nur Check-Modus, informativ — kein Baustein).** Alle installierten Skills mechanisch erheben und mit Quelle listen: persönliche (`~/.claude/skills/`), projektlokale (`.claude/skills/` im Arbeitsbaum), Plugin-Skills (installierte Plugins samt Version). Dazu ausschließlich mechanische Befunde: **Namens-Kollisionen/Trigger-Überlappungen** zwischen Skills · **tote Symlinks** · **Skill-Referenzen auf nicht existierende Pfade**. Die Arbeitsweise fremder Skills wird nicht bewertet — Befunde benennen nur, was mechanisch bricht oder kollidiert.

## Harte Gates

- Messen vor Handeln; Ausweis-Tabelle ist Pflicht.
- **Secrets nie erfragen, nie entgegennehmen, nie notieren.** Der Begleiter richtet den Ort ein (Vault, Verweis-Muster, infra.md) — den Wert legt der Nutzer selbst im Vault ab; wird trotzdem ein Wert in den Chat eingefügt, wird er nirgends persistiert und der Nutzer auf Rotation hingewiesen.
- Bestehende Dateien (CLAUDE.md, context.md, language.md, brand.yaml, infra.md) nie überschreiben: ergänzen oder flaggen.
- Kein Aufräumen im Bestand — an `/cleanup` delegieren.
- Interview kurz, keine Use-Case-Findung, kein Psychogramm (Personen-Regeln der Spec gelten).
- Skips in L1 „Offene Punkte" festhalten.
- Scope-Ende: keine Domänen anlegen oder befüllen — das ist Kursinhalt.
