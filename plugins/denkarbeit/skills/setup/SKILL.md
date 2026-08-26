---
name: setup
description: Der Begleiter — geführtes Ersteinrichten des Denkarbeit-Setups (Umgebung, CLAUDE.md, Root-Kontext, Sprache, CI, Systeme & Secrets) und Gesamtcheck. Triggers on /denkarbeit:setup, „Setup starten", „richte mein Setup ein", „hab ich alles konfiguriert?".
---

## Zweck

Der Begleiter richtet das Denkarbeit-Setup ein und prüft es. Ein Mechanismus, zwei Aufrufe:

- `/denkarbeit:setup` — geführtes Einrichten: erst messen, dann fehlende Bausteine im Dialog schließen.
- `/denkarbeit:setup check` — nur messen und ausweisen, nichts verändern.

**Setup = Check + Lücken schließen.** Dadurch ist der Skill resumierbar: Jeder Aufruf beginnt mit derselben Messung; erledigte Bausteine werden erkannt und übersprungen. Es gibt kein Status-File — der Zustand wird aus den Artefakten selbst abgeleitet (Kein-Status-Prinzip). Einzige Ausnahme: bewusste Skips stehen als Zeile in der `context.md` des Roots unter „Offene Punkte" — sonst wären sie von „vergessen" nicht unterscheidbar.

**Scope-Linie:** Der Begleiter endet bei „System steht leer und korrekt konfiguriert". Den ersten eigenen Kontext-Ordner anzulegen und zu befüllen ist die Arbeit danach, nicht Teil des Setups.

Die Templates dieses Skills liegen unter `templates/` relativ zum Skill-Basisverzeichnis. Das Backbone der `context.md` steht in `denkarbeit:contextify`; dieser Skill wiederholt es nicht, sondern wendet dessen Backbone in B2 an — ein eigener Skill-Aufruf ist nicht nötig, weil Gegenstand und Inputs dort schon feststehen. Ein externes Spezifikationsdokument gibt es nicht und wird nirgends nachgeladen.

## Bausteine & Messung

**Mess-Pflicht:** Vor jedem Handeln alle Bausteine mechanisch erheben — nicht schätzen, nicht aus dem Gedächtnis. Eine Prüfung ohne erhobenen Befund hat nicht stattgefunden. Lässt sich eine Messung nicht durchführen (Kommando nicht erlaubt, Ort nicht lesbar), belegt ein gleichwertiger Ersatzbefund den Baustein genauso (`which git` statt `git --version`) — dann ✓ mit Vermerk. Bleibt die Sache offen, ist das **kein** ✓ und kein ✗, sondern `?`, und die ungeprüfte Stelle wird wörtlich benannt.

| # | Baustein | Messung | Status |
|---|---|---|---|
| 0 | Umgebung | `git --version` läuft · Workspace-Ordner existiert | Pflicht · Obsidian-Vault (`{workspace}/.obsidian/`) empfohlen, nicht erforderlich |
| 1 | CLAUDE.md | `~/.claude/CLAUDE.md` existiert und enthält Workspace-Konvention + Session-Start-Regel + Sprachregeln + Secrets-Block | Pflicht |
| 2 | Root-Kontext | `{workspace}/context.md` existiert mit Backbone-Kopf (Steckbrief, Motivation) | Pflicht |
| 3 | Sprache | Sprachregeln in der CLAUDE.md vorhanden (Baustein 1); `{workspace}/language.md` **optional**, nur mit geeignetem Referenzmaterial | Pflicht (Regeln) · optional (Referenz) |
| 4 | Bestand | Liegt im Workspace Material, das **nicht** aus diesem Setup stammt (alles außer `context.md`, `CLAUDE.md`, `language.md` im Root)? | Situativ |
| 5 | Systeme & Secrets | Nutzer arbeitet gegen APIs/CLIs/MCP-Server: `infra.md` des betroffenen Ordners existiert · ein Vault ist benannt (Passwort-Manager-CLI oder OS-Schlüsselbund) · CLAUDE.md trägt den Secrets-Block | Situativ — Pflicht, sobald zutreffend |

Workspace-Pfad finden: aus der CLAUDE.md (Baustein 1 verankert ihn). Fehlt sie **oder trägt sie keinen Workspace-Anker** (der Regelfall bei bestehender Fremd-Konfiguration): den User fragen, bei Neueinrichtung `~/workspace` vorschlagen. Alle Skills arbeiten danach gegen diesen Anker; `{workspace}` in den Skills meint immer ihn, nie einen festen Pfad.

## Workflow

1. **Modus bestimmen.** `check` im Aufruf → nur die Schritte 2, 5 und 6. Sonst geführtes Setup (Schritte 2–5).

2. **Messen.** Alle Bausteine der Tabelle mechanisch erheben. Skips erkennen: Einträge „… übersprungen" unter „Offene Punkte" im Root-Kontext.

3. **Lücken schließen** (nur fehlende Bausteine, in Reihenfolge):

   **B0 — Umgebung.**
   - git fehlt: macOS → `git --version` löst den Ein-Klick-Installationsdialog aus; Windows → `winget install Git.Git`.
   - Kommando nicht gefunden trotz Installation: Der Regelfall-Fix ist die PATH-Zeile im Shell-Profil — wörtlich anleiten, nicht diagnostizieren lassen.
   - Workspace-Ordner anlegen (Vorschlag `~/workspace`, User bestätigt).
   - Obsidian: Vault auf den Workspace-Ordner öffnen („Open folder as vault") — prüfbar an `{workspace}/.obsidian/`.

   **B1 — CLAUDE.md.** Aus `templates/CLAUDE_template.md` erzeugen; Platzhalter füllen: `{{NAME}}`, `{{WORKSPACE}}` (absoluter Pfad). Es wird **kein Pfad in dieses Plugin** geschrieben: Ein versionierter Plugin-Pfad stirbt beim ersten Update, und die Regeln stehen ohnehin in den Skills selbst. Ist `~/.claude/` nicht beschreibbar, wird **nicht** still auf den Workspace ausgewichen: Eine `CLAUDE.md` dort gilt nur für Sessions in diesem Baum, die Session-Start-Regel greift außerhalb nicht mehr. Den Ort dem User vorlegen, das Ausweichen kenntlich machen. **Existiert bereits eine CLAUDE.md: nie überschreiben** — fehlende Blöcke ergänzen vorschlagen, Konflikte flaggen. Ergänzungen landen als **ein** zusammenhängender, markierter Block (`<!-- denkarbeit:begin -->` … `<!-- denkarbeit:end -->`), damit sie neben einer bestehenden Fremd-Konfiguration identifizierbar und rückbaubar bleiben; in fremde Blöcke wird nie hineingeschrieben.

   **B2 — Root-Kontext (Mini-Interview).** Das Interview ist der Primärweg, Dokumente sind Beschleuniger — es gibt keinen „keine Quellen"-Fall. 5–7 Fragen: Rolle und Organisation · für wen/was gearbeitet wird (Domänen, Stakeholder) · Kern-Arbeitsinhalte · laufende Vorhaben · Ziel mit dem Setup. Quellen, falls vorhanden: CV-PDF · LinkedIn-Export (im Profil „Als PDF speichern" — URLs scheitern oft an der Login-Wall) · **ein selbstverfasstes Arbeitsdokument** (die wertvollste Quelle: verrät Rolle und Sprache, füttert auch B3). Der Root-Kontext braucht Gegenwart, nicht Werdegang — CV-Vergangenheit nur als Steckbrief-Hintergrund. Die Zielgruppe weiß, was sie will: Interview kurz halten, keine Use-Case-Findung. Dann `{workspace}/context.md` nach der contextify-Logik erzeugen — das Backbone und die Steckbrief-Felder stehen dort, nicht hier.

   **B3 — Sprach-Referenz (optional, bewusst kein Default-Artefakt).** Die Sprachregeln stehen in der CLAUDE.md (Baustein 1) und tragen für sich. Eine zusätzliche `{workspace}/language.md` lohnt nur mit **gutem Referenzmaterial** — ohne das entsteht ein Artefakt, das den Ton verschlechtert statt ihn zu schärfen. Deshalb: Angebot machen, Kriterien nennen, Nein akzeptieren (kein Skip-Vermerk nötig).

   Frage: „Hast du längere Texte, die du **selbst geschrieben** hast und für gut hältst?" Dann die Eignung klären:
   - **Geeignet:** eigene Autorenschaft (ein LLM darf geglättet, nicht formuliert haben — sonst imitiert das Modell sich selbst) · erklärende Sachprosa für ein Publikum · ein Autor, konsistente Handschrift · redigierte oder veröffentlichte Fassungen · Gesamtumfang etwa 1.500 bis 8.000 Wörter (mehr kostet Kontext ohne Zusatznutzen).
   - **Nicht geeignet:** Mails, Chat-Verläufe, Notizen (zu kurz und situativ, keine Satzarchitektur) · LLM-Entwürfe mit Politur · Sammlungen mehrerer Autoren (mitteln sich zum Durchschnitt, und der Durchschnitt ist genau der Maschinen-Ton) · reine Fachdokumentation · Übersetzungen.
   - **Nichts davon vorhanden:** kein Artefakt anlegen. Die CLAUDE.md-Regeln gelten ohnehin; die Referenz kann jederzeit später entstehen.

   Bei Zusage: Die stärksten Absätze **kuratieren**, nicht die Dateien kopieren, und mit einer Fallenliste (beobachtete Drifts mit Gegenbeispiel) in `language.md` ablegen. Aufbau: Referenzmaterial zum Imitieren, Fallenliste, Pflegehinweis — **keine Stilvorschriften.** Positive Stilregeln werden von Modellen als Checkliste abgearbeitet und übererfüllt; was im Original zweimal trägt, steht dann in jedem Absatz.

   **B4 — Bestand.** War der Workspace nicht leer: Befund erheben (Ordnerzahl, lose Dateien, erkennbare Kontext-Ordner; Repos/Symlinks mechanisch: `find -name .git`, `find -type l`) und **an `/denkarbeit:cleanup` übergeben** — der hat Plan-Gate und Safety-Gates. Der Begleiter räumt nie selbst auf (keine Duplikation der cleanup-Logik).

   **B5 — Systeme & Secrets (situativ).** Eine Interview-Frage: „Wird dein Claude mit Systemen arbeiten — APIs, Datenbanken, eigene Dienste, MCP-Server?" Wenn nein: kein Baustein, kein Skip-Vermerk. Wenn ja:
   1. **Vault klären** (Staffel, kein Tool-Zwang): vorhandener Passwort-Manager mit CLI (1Password `op` · Bitwarden `bw` · KeePassXC `keepassxc-cli`) → sonst der **OS-Schlüsselbund** (macOS Keychain via `security` · Windows Credential Manager/DPAPI via PowerShell) — der ist immer da und ist der Default-Fallback. Eine Klartext-Datei ist **kein** Fallback. Laufzeit-Muster einmal zeigen: Secret aus dem Vault in die Umgebung der Session laden, nie in eine Datei.
   2. **`infra.md`-Gerüst anlegen** — im Ordner, zu dem das System gehört (Dienst-Infra). Existiert dieser Ordner noch nicht (der Regelfall beim Ersteinrichten, weil die Scope-Linie hier endet): die `infra.md` **im Workspace-Root** anlegen und als Zeile in den Offenen Punkten vermerken, dass sie mitwandert, sobald der zugehörige Ordner entsteht. Sektionen: *System & Zugang* (Verweis-Muster: Vault + Eintragsname + Zugriffs-Kommando — nie der Wert), *Eigenheiten*, *Fehlerbilder & Fixes*. Maschinen-Infra (Tools, MCP-Konfig) → `~/.claude/reference/infra.md`.
   3. **Secrets-Block der CLAUDE.md** verifizieren (Baustein 1 legt ihn aus dem Template an; bei Alt-Installationen fehlt er → ergänzen vorschlagen).
   Liegen bereits Zugriffs-Praktiken in flüchtigen Notizen (Scratchpad, temp.md, Chat-Verläufe), werden sie in die `infra.md` **geerntet** — das ist der häufigste Alt-Fall.

4. **Skips festhalten.** Einen bewusst übersprungenen Baustein als Zeile unter „Offene Punkte" im Root-Kontext vermerken — **außer** wo der Baustein selbst darauf verzichtet (B3 und B5: ein Nein ist dort eine Antwort, kein Skip).

5. **Ausweis (Pflicht, nie weglassen).** Tabelle aller Bausteine mit genau einem Status je Zeile: **✓** konfiguriert · **○** offen (optional) · **–** bewusst übersprungen · **✗** fehlt (Pflicht) · **n/a** trifft nicht zu (situativer Baustein ohne Anwendungsfall) · **?** ungeprüft (Messung war nicht durchführbar — die Stelle in der Befund-Spalte wörtlich benennen). Bündelt ein Baustein Pflicht und Optionales (0 und 3), richtet sich der Status nach dem **Pflichtteil**; der offene optionale Rest steht in der Befund-Spalte. Dazu die Plugin-Version aus `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`, falls als Plugin installiert; ist der Pfad nicht lesbar, die Version aus dem geladenen Plugin-Verzeichnis dieses Skills nehmen, sonst „Version nicht ermittelbar" ausweisen. Ohne diese Tabelle gilt der Lauf als nicht geprüft.

6. **Skill-Inventar (nur Check-Modus, informativ — kein Baustein).** Alle installierten Skills mechanisch erheben und mit Quelle listen: persönliche (`~/.claude/skills/`), projektlokale (`.claude/skills/` im Arbeitsbaum), Plugin-Skills (installierte Plugins samt Version). Dazu ausschließlich mechanische Befunde: **Namens-Kollisionen/Trigger-Überlappungen** zwischen Skills · **tote Symlinks** · **Skill-Referenzen auf nicht existierende Pfade**. Die Arbeitsweise fremder Skills wird nicht bewertet — Befunde benennen nur, was mechanisch bricht oder kollidiert.

## Harte Gates

- Messen vor Handeln; Ausweis-Tabelle ist Pflicht.
- **Secrets nie erfragen, nie entgegennehmen, nie notieren.** Der Begleiter richtet den Ort ein (Vault, Verweis-Muster, infra.md) — den Wert legt der Nutzer selbst im Vault ab; wird trotzdem ein Wert in den Chat eingefügt, wird er nirgends persistiert und der Nutzer auf Rotation hingewiesen.
- Bestehende Dateien (CLAUDE.md, context.md, language.md, infra.md) nie überschreiben: ergänzen oder flaggen.
- Kein Aufräumen im Bestand — an `/denkarbeit:cleanup` delegieren.
- Interview kurz, keine Use-Case-Findung, kein Psychogramm.
- Skips unter „Offene Punkte" im Root-Kontext festhalten, soweit der Baustein sie verlangt (B3/B5 ausgenommen).
- Scope-Ende: keine Kontext-Ordner für die eigentliche Arbeit anlegen oder befüllen.
