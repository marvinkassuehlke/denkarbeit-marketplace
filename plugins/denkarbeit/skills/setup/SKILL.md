---
name: setup
description: Der Begleiter — geführtes Ersteinrichten des Denkarbeit-Setups (Umgebung, CLAUDE.md, Root-Kontext, Sprache, Bestand, Secrets, Permissions) und Gesamtcheck. Triggers on /denkarbeit:setup, „Setup starten", „richte mein Setup ein", „hab ich alles konfiguriert?".
---

## Zweck

Der Begleiter richtet das Denkarbeit-Setup ein und prüft es. Ein Mechanismus, zwei Aufrufe:

- `/denkarbeit:setup` — geführtes Einrichten: erst messen, dann fehlende Bausteine im Dialog schließen.
- `/denkarbeit:setup check` — nur messen und ausweisen, nichts verändern.

**Setup = Check + Lücken schließen.** Dadurch ist der Skill resumierbar: Jeder Aufruf beginnt mit derselben Messung; erledigte Bausteine werden erkannt und übersprungen. Es gibt kein Status-File — der Zustand wird aus den Artefakten selbst abgeleitet (Kein-Status-Prinzip). Einzige Ausnahme: bewusste Skips stehen als Zeile in der `context.md` des Roots unter „Offene Punkte" — sonst wären sie von „vergessen" nicht unterscheidbar.

**Scope-Linie:** Der Begleiter endet bei „System steht leer und korrekt konfiguriert" — dazu gehört, dass der Nutzer weiß, was Claude ohne Rückfrage darf (B6). Den ersten eigenen Kontext-Ordner anzulegen und zu befüllen ist die Arbeit danach, nicht Teil des Setups.

Die Templates dieses Skills liegen unter `templates/` relativ zum Skill-Basisverzeichnis. Das Backbone der `context.md` steht in `denkarbeit:contextify`; dieser Skill wiederholt es nicht, sondern wendet dessen Backbone in B2 an — ein eigener Skill-Aufruf ist nicht nötig, weil Gegenstand und Inputs dort schon feststehen. Ein externes Spezifikationsdokument gibt es nicht und wird nirgends nachgeladen.

## Bausteine & Messung

**`{konfig}` — das wirksame Konfigurationsverzeichnis.** Überall dort, wo unten `{konfig}` steht, ist der Ort gemeint, aus dem Claude Code tatsächlich liest: `$CLAUDE_CONFIG_DIR`, falls die Variable gesetzt ist, sonst `~/.claude`. Einmal zu Beginn feststellen und für den ganzen Lauf verwenden — eine Datei am falschen Ort hat keine Wirkung, ohne dass es auffällt.

**Mess-Pflicht:** Vor jedem Handeln alle Bausteine mechanisch erheben — nicht schätzen, nicht aus dem Gedächtnis. Eine Prüfung ohne erhobenen Befund hat nicht stattgefunden. Lässt sich eine Messung nicht durchführen (Kommando nicht erlaubt, Ort nicht lesbar), belegt ein gleichwertiger Ersatzbefund den Baustein genauso (`which git` statt `git --version`) — dann ✓ mit Vermerk. Bleibt die Sache offen, ist das **kein** ✓ und kein ✗, sondern `?`, und die ungeprüfte Stelle wird wörtlich benannt.

| # | Baustein | Messung | Status |
|---|---|---|---|
| 0 | Umgebung | `git --version` läuft · Workspace-Ordner existiert | Pflicht · Obsidian-Vault (`{workspace}/.obsidian/`) empfohlen, nicht erforderlich |
| 1 | CLAUDE.md | `{konfig}/CLAUDE.md` existiert und enthält Workspace-Konvention + Session-Start-Regel + Sprachregeln + Secrets-Block | Pflicht |
| 2 | Root-Kontext | `{workspace}/context.md` existiert mit Backbone-Kopf (Steckbrief, Motivation) | Pflicht |
| 3 | Sprache | Sprachregeln in der CLAUDE.md vorhanden (Baustein 1); `{workspace}/language.md` **optional**, nur mit geeignetem Referenzmaterial | Pflicht (Regeln) · optional (Referenz) |
| 4 | Bestand | Liegt im Workspace Material, das **nicht** aus diesem Setup stammt (alles außer `context.md`, `CLAUDE.md`, `language.md` im Root)? | Situativ |
| 5 | Secrets | Ein Vault ist benannt (OS-Schlüsselbund oder aktiv genutzter Passwort-Manager) · CLAUDE.md trägt den Secrets-Block · `infra.md`, sobald gegen ein System gearbeitet wird | Pflicht |
| 6 | Permissions | Der Nutzer weiß, dass Claude im Auslieferungszustand nichts ohne Rückfrage tut, und kennt den Weg, das zu ändern | Pflicht (erklären) · optional (setzen) |

Workspace-Pfad finden: aus der CLAUDE.md (Baustein 1 verankert ihn). Fehlt sie **oder trägt sie keinen Workspace-Anker** (der Regelfall bei bestehender Fremd-Konfiguration): den User fragen, bei Neueinrichtung `~/workspace` vorschlagen. Widersprechen sich Anker und Nutzerangabe, **gilt die Nutzerangabe** — der Anker ist dann veraltet und wird im selben Lauf nachgezogen, nicht stillschweigend übergangen. Alle Skills arbeiten danach gegen diesen Anker; `{workspace}` in den Skills meint immer ihn, nie einen festen Pfad.

## Workflow

1. **Modus bestimmen.** `check` im Aufruf → nur die Schritte 2, 5 und 6. Sonst geführtes Setup (Schritte 2–5).

2. **Messen.** Alle Bausteine der Tabelle mechanisch erheben. Skips erkennen: Zeilen mit dem Präfix `Setup übersprungen:` unter „Offene Punkte" im Root-Kontext.

3. **Lücken schließen** (nur fehlende Bausteine, in Reihenfolge):

   **B0 — Umgebung.**
   - git fehlt: macOS → `git --version` löst den Ein-Klick-Installationsdialog aus; Windows → `winget install Git.Git`.
   - Kommando nicht gefunden trotz Installation: Der Regelfall-Fix ist die PATH-Zeile im Shell-Profil — wörtlich anleiten, nicht diagnostizieren lassen.
   - Workspace-Ordner anlegen (Vorschlag `~/workspace`, User bestätigt).
   - Obsidian: Vault auf den Workspace-Ordner öffnen („Open folder as vault") — prüfbar an `{workspace}/.obsidian/`.

   **B1 — CLAUDE.md.** Aus `templates/CLAUDE_template.md` erzeugen; Platzhalter füllen: `{{NAME}}`, `{{WORKSPACE}}` (absoluter Pfad). Es wird **kein Pfad in dieses Plugin** geschrieben: Ein versionierter Plugin-Pfad stirbt beim ersten Update, und die Regeln stehen ohnehin in den Skills selbst. Ist `{konfig}/` nicht beschreibbar, wird **nicht** still auf den Workspace ausgewichen: Eine `CLAUDE.md` dort gilt nur für Sessions in diesem Baum, die Session-Start-Regel greift außerhalb nicht mehr. Den Ort dem User vorlegen, das Ausweichen kenntlich machen. **Existiert bereits eine CLAUDE.md: nie überschreiben** — fehlende Blöcke ergänzen vorschlagen, Konflikte flaggen. Ein dritter Fall kommt in der Praxis häufiger vor als der zweite: eine **vollständige denkarbeit-CLAUDE.md, die auf eine andere Person oder einen anderen Workspace-Root ausgestellt ist** (Rechnerwechsel, übernommenes Setup, verschobener Workspace). Dann fehlt kein Block, und „nie überschreiben" würde die Korrektur verbieten. Auflösung: die abweichenden Stellen benennen (Name, Workspace-Pfad), die Anpassung genau dieser Stellen vorschlagen und nach Bestätigung ausführen — der Rest der Datei bleibt unberührt. Im Ausweis als ✗ führen, solange sie auf fremde Angaben zeigt. Ergänzungen landen als **ein** zusammenhängender, markierter Block (`<!-- denkarbeit:begin -->` … `<!-- denkarbeit:end -->`), damit sie neben einer bestehenden Fremd-Konfiguration identifizierbar und rückbaubar bleiben; in fremde Blöcke wird nie hineingeschrieben.

   **B2 — Root-Kontext (Mini-Interview).**

   **Zuerst nach Unterlagen fragen, nicht nebenbei.** Zwei Dokumente heben das Ergebnis deutlich, und wer sie hat, spart die Hälfte der Fragen: **eines zur Person** (CV als PDF, LinkedIn-Export über „Als PDF speichern" — URLs scheitern an der Login-Wall) und **eines zum Hauptkontext** (Firmenprofil, Projektbeschreibung, Angebot, ein Papier über den Arbeitgeber). Am wertvollsten ist ein **selbstverfasstes Arbeitsdokument**: Es verrät Rolle und Sprache zugleich. Liegt nichts vor, trägt das Interview allein — aber dann sagen, dass der Steckbrief dünner ausfällt und später nachgeschärft werden kann, statt so zu tun, als sei es gleichwertig.

   **Fünf Fragen, nicht mehr.** Jede muss von jemandem beantwortbar sein, der Claude heute zum ersten Mal einrichtet:
   1. Was machst du, in welchem Rahmen (angestellt, selbständig, eigene Firma)?
   2. Für wen arbeitest du — Kunden, Arbeitgeber, interne Bereiche? Grobe Typen genügen.
   3. Welche zwei, drei Tätigkeiten machen den Großteil deiner Zeit aus?
   4. Woran arbeitest du gerade konkret?
   5. **Was erklärst du immer wieder neu, wenn du mit einem Chat anfängst?**

   Frage 5 ersetzt die frühere Zielfrage („wofür soll Claude dich entlasten?"). Die setzte voraus, dass jemand die Möglichkeiten des Werkzeugs schon kennt — wer gerade installiert hat, kennt sie nicht. Die neue Frage fragt nach einer Erfahrung statt nach einer Absicht und liefert dasselbe Material.

   Der Root-Kontext braucht Gegenwart, nicht Werdegang — CV-Vergangenheit nur als Steckbrief-Hintergrund. Keine Use-Case-Findung. Dann `{workspace}/context.md` nach der contextify-Logik erzeugen — das Backbone und die Steckbrief-Felder stehen dort, nicht hier.

   **B3 — Sprach-Referenz (optional, bewusst kein Default-Artefakt).** Die Sprachregeln stehen in der CLAUDE.md (Baustein 1) und tragen für sich. Eine zusätzliche `{workspace}/language.md` lohnt nur mit **gutem Referenzmaterial** — ohne das entsteht ein Artefakt, das den Ton verschlechtert statt ihn zu schärfen. Deshalb: Angebot machen, Kriterien nennen, Nein akzeptieren (kein Skip-Vermerk nötig).

   Frage: „Hast du längere Texte, die du **selbst geschrieben** hast und für gut hältst?" Dann die Eignung klären:
   - **Geeignet:** eigene Autorenschaft (ein LLM darf geglättet, nicht formuliert haben — sonst imitiert das Modell sich selbst) · erklärende Sachprosa für ein Publikum · ein Autor, konsistente Handschrift · redigierte oder veröffentlichte Fassungen · Gesamtumfang etwa 1.500 bis 8.000 Wörter (mehr kostet Kontext ohne Zusatznutzen).
   - **Nicht geeignet:** Mails, Chat-Verläufe, Notizen (zu kurz und situativ, keine Satzarchitektur) · LLM-Entwürfe mit Politur · Sammlungen mehrerer Autoren (mitteln sich zum Durchschnitt, und der Durchschnitt ist genau der Maschinen-Ton) · reine Fachdokumentation · Übersetzungen.
   - **Nichts davon vorhanden:** kein Artefakt anlegen. Die CLAUDE.md-Regeln gelten ohnehin; die Referenz kann jederzeit später entstehen.

   Bei Zusage: Die stärksten Absätze **kuratieren**, nicht die Dateien kopieren, und mit einer Fallenliste (beobachtete Drifts mit Gegenbeispiel) in `language.md` ablegen. Aufbau: Referenzmaterial zum Imitieren, Fallenliste, Pflegehinweis — **keine Stilvorschriften.** Positive Stilregeln werden von Modellen als Checkliste abgearbeitet und übererfüllt; was im Original zweimal trägt, steht dann in jedem Absatz.

   **B4 — Bestand.** War der Workspace nicht leer: Befund erheben (Ordnerzahl, lose Dateien, erkennbare Kontext-Ordner; Repos/Symlinks mechanisch: `find -name .git`, `find -type l`) und **an `/denkarbeit:cleanup` übergeben** — der hat Plan-Gate und Safety-Gates. Der Begleiter räumt nie selbst auf (keine Duplikation der cleanup-Logik).

   **B5 — Secrets.** **Nicht fragen, ob mit Systemen gearbeitet wird** — die Antwort ist immer ja, und ein Nein heißt nur, dass es noch nicht aufgefallen ist. Der Baustein richtet den *Ort* ein, bevor das erste Secret auftaucht.
   1. **Vault einrichten — der Mechanismus ist Pflicht, das Produkt nicht.** Default ist der **OS-Schlüsselbund** (macOS `security` · Windows Credential Manager/DPAPI via PowerShell): Er ist auf jedem Rechner vorhanden, kostet nichts und muss nicht eingerichtet werden. Nur wer bereits einen Passwort-Manager mit CLI **aktiv nutzt**, bleibt dabei (1Password `op` · Bitwarden `bw` · KeePassXC `keepassxc-cli`) — dann prüfen, ob er auch angemeldet ist (`op account list` und Entsprechungen; eine installierte CLI ohne Konto ist kein Vault). Kein Produkt vorschlagen, das der Nutzer nicht schon hat. Eine Klartext-Datei ist unter keinen Umständen ein Fallback. Laufzeit-Muster einmal zeigen: Secret aus dem Vault in die Umgebung der Session laden, nie in eine Datei.
   2. **`infra.md`-Gerüst anlegen** — im Ordner, zu dem das System gehört (Dienst-Infra). Existiert dieser Ordner noch nicht (der Regelfall beim Ersteinrichten, weil die Scope-Linie hier endet): die `infra.md` **im Workspace-Root** anlegen und als Zeile in den Offenen Punkten vermerken, dass sie mitwandert, sobald der zugehörige Ordner entsteht. Sektionen: *System & Zugang* (Verweis-Muster: Vault + Eintragsname + Zugriffs-Kommando — nie der Wert), *Eigenheiten*, *Fehlerbilder & Fixes*. Maschinen-Infra (Tools, MCP-Konfig) → `{konfig}/reference/infra.md`.
   3. **Secrets-Block der CLAUDE.md** verifizieren (Baustein 1 legt ihn aus dem Template an; bei Alt-Installationen fehlt er → ergänzen vorschlagen).
   Liegen bereits Zugriffs-Praktiken in flüchtigen Notizen (Scratchpad, temp.md, Chat-Verläufe), werden sie in die `infra.md` **geerntet** — das ist der häufigste Alt-Fall.

   **B6 — Permissions (erklären ist Pflicht, setzen ist es nicht).**

   **Erst messen, dann erklären** — die Mess-Pflicht gilt auch hier, ein angenommener Zustand ist keiner. Lies den `permissions`-Block und `defaultMode` aus den Settings-Dateien, die tatsächlich gelten: `{konfig}/settings.json`, daneben `{konfig}/settings.local.json` und, falls im Arbeitsbaum vorhanden, `.claude/settings.json` — die lokale schlägt die globale. Erkläre **den erhobenen Zustand**, nicht den Auslieferungszustand.

   Im Regelfall ist nichts gesetzt: Claude fragt dann vor jeder Datei-Änderung und jedem Kommando. Das in zwei Sätzen erklären und sagen, warum es sinnvoll ist, es zunächst so zu lassen — jede Rückfrage zeigt, was das Werkzeug tun will, und das ist die billigste Art, es kennenzulernen. Dazu der Satz, dass sich das jederzeit ändern lässt („sag mir einfach Bescheid"). Findest du bereits Regeln vor, nenne sie in Klartext statt in Syntax und lass sie unangetastet.

   **Danach das Angebot machen, nicht verschweigen.** Wer jetzt schon weniger Rückfragen will, bekommt genau eine Stufe — keine Profilauswahl, keine Einzelabfrage von Regeln. Die drei Bausteine, jeder mit seinem Grund in einem Satz:
   - `"defaultMode": "acceptEdits"` — Dateien ändern ohne Rückfrage. Vertretbar, weil Änderungen sichtbar und umkehrbar sind.
   - `"ask"`: `Bash(rm:*)`, `Bash(unlink:*)`, `Bash(rmdir:*)` — Löschen ist nicht umkehrbar und wird gefragt, auch wenn sonst nichts gefragt wird.
   - `"deny"`: `Read(./.env*)`, `Read(./secrets/**)` — Dateien mit Zugangsdaten werden nicht einmal gelesen.

   Das gehört unter `permissions` in `{konfig}/settings.json`, nicht in die `CLAUDE.md`. Steht dort schon etwas, wird ergänzt statt ersetzt: fehlende Schlüssel hinzufügen, vorhandene Werte unangetastet lassen, Konflikte flaggen. Liegt eine wirksamere Datei darüber (`settings.local.json`, projektlokale Settings), das dem Nutzer sagen — sonst schreibt er in die Datei, die überstimmt wird.

   **Was nicht angeboten wird:** pauschale Freigaben (`Bash(*)`), `bypassPermissions`, alles mit Außenwirkung (push, senden, deployen). Wer das braucht, weiß nach ein paar Wochen selbst, welche Regel ihm fehlt — und kann sie dann aus den eigenen Sitzungen ableiten lassen, statt sie am ersten Tag zu raten.

4. **Skips festhalten.** Einen bewusst übersprungenen Baustein als Zeile unter „Offene Punkte" im Root-Kontext vermerken, **erkennbar als Skip** (die Zeile beginnt mit `Setup übersprungen:`) — sonst ist er beim nächsten Lauf nicht von einer gewöhnlichen offenen Aufgabe zu unterscheiden, und genau diese Unterscheidung trägt das Kein-Status-Prinzip — **außer** wo der Baustein selbst darauf verzichtet (B3 und B6: ein Nein ist dort eine Antwort, kein Skip).

5. **Ausweis (Pflicht, nie weglassen).** Tabelle aller Bausteine mit genau einem Status je Zeile: **✓** konfiguriert · **○** offen (optional) · **–** bewusst übersprungen · **✗** fehlt (Pflicht) · **n/a** trifft nicht zu (situativer Baustein ohne Anwendungsfall) · **?** ungeprüft (Messung war nicht durchführbar — die Stelle in der Befund-Spalte wörtlich benennen). Bündelt ein Baustein Pflicht und Optionales (0, 3 und 6), richtet sich der Status nach dem **Pflichtteil**; der offene optionale Rest steht in der Befund-Spalte. Dazu die Plugin-Version aus `{konfig}/plugins/installed_plugins.json` — dort steht, was tatsächlich installiert ist, und die Datei liegt im Konfigurationsverzeichnis statt hinter einem versionierten Plugin-Pfad. Ist sie nicht lesbar, ersatzweise `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`, sonst „Version nicht ermittelbar" ausweisen. Ohne diese Tabelle gilt der Lauf als nicht geprüft.

6. **Skill-Inventar (nur Check-Modus, informativ — kein Baustein).** Alle installierten Skills mechanisch erheben und mit Quelle listen: persönliche (`{konfig}/skills/`), projektlokale (`.claude/skills/` im Arbeitsbaum), Plugin-Skills (installierte Plugins samt Version). Dazu ausschließlich mechanische Befunde: **Namens-Kollisionen/Trigger-Überlappungen** zwischen Skills · **tote Symlinks** · **Skill-Referenzen auf nicht existierende Pfade**. Die Arbeitsweise fremder Skills wird nicht bewertet — Befunde benennen nur, was mechanisch bricht oder kollidiert.

## Harte Gates

- Messen vor Handeln; Ausweis-Tabelle ist Pflicht.
- **Bausteinnummern sind interne Gliederung, keine Nutzersprache.** Nie „ich lege B1 an" oder „zu B4 kann ich dir sagen" — im Dialog heißt es, was gemeint ist („deine Arbeits-Regeln", „was in deinem Workspace liegt"). Einzige Ausnahme ist die Ausweis-Tabelle, die eine nummerierte Spalte führt.
- **Secrets nie erfragen, nie entgegennehmen, nie notieren.** Der Begleiter richtet den Ort ein (Vault, Verweis-Muster, infra.md) — den Wert legt der Nutzer selbst im Vault ab; wird trotzdem ein Wert in den Chat eingefügt, wird er nirgends persistiert und der Nutzer auf Rotation hingewiesen.
- Bestehende Dateien (CLAUDE.md, context.md, language.md, infra.md) nie überschreiben: ergänzen oder flaggen.
- Kein Aufräumen im Bestand — an `/denkarbeit:cleanup` delegieren.
- Interview kurz, keine Use-Case-Findung, kein Psychogramm.
- Skips unter „Offene Punkte" im Root-Kontext festhalten, soweit der Baustein sie verlangt (B3/B6 ausgenommen).
- Scope-Ende: keine Kontext-Ordner für die eigentliche Arbeit anlegen oder befüllen.
