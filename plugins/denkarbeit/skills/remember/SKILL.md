---
name: remember
description: Use when the user wants to persist memory from the current session, initialize a new Stakeholder/Projekt memory structure, or compress older memories. Triggers on /remember.
---

## Zweck

Persistiere Wissen, das sonst mit der Session verloren geht — Entscheidungen, Erkenntnisse, Kursänderungen, Urteile. Nicht Fakten cachen, sondern Erfahrung festhalten.

Verdichtungsstufen:

1. **Log** (`log.md`) — Ereignisstrom, session-nah, append-only.
2. **Memory** (`memory.md`) — verdichtet aus dem Log: Urteile, Übergänge, das Warum.

Beide liegen **per Default auf Stakeholder-Ebene (L2)**. Ein Projekt bekommt eigene `log.md`/`memory.md` nur als Earn-it-Ausnahme (siehe „## Split"). „Vergessen" ist gewollte Kompression, kein Datenverlust.

## Ordnerstruktur

```
{workspace}/
├── context.md                  ← L1: Ich (via /contextify)
├── {Stakeholder}/
│   ├── context.md              ← L2: wer (via /contextify)
│   ├── memory.md               ← L2: Urteile/Übergänge (Default-Heimat)
│   ├── log.md                  ← L2: Ereignisstrom (Default-Heimat)
│   └── {Projekt}/
│       ├── context.md          ← L3: was mit ihnen (via /contextify)
│       └── (log.md / memory.md nur bei Split — siehe unten)
```

`context.md` wird über `/contextify` erstellt und gepflegt — nicht Aufgabe dieses Skills. `memory.md`/`log.md` liegen per Default auf Stakeholder-Ebene; Projekt-Ebene nur bei Split.

## Workflow

### 1. Modus bestimmen

Parse die Usereingabe und bestimme den Modus:

- **Log** (default, keine Argumente oder nur Kontext-Hinweise) — schreibe einen Eintrag in `log.md`
- **Init** (`init {Stakeholder}` oder `init {Stakeholder}/{Projekt}`) — lege Ordnerstruktur und leere Dateien an
- **Verdichten** (`verdichten`) — komprimiere ältere Einträge

### 2. Modus: Log (default)

1. **Kontext bestimmen.** Ermittle aus dem Workspace-Pfad den Stakeholder (und ggf. das Projekt). Die Ebene erkennst du semantisch, nicht an der Pfadtiefe: Sammel-/Zwischenordner (z.B. `inaktiv/`) verschieben die Tiefe, nicht die Ebene — Ebenen-Träger ist der Ordner, der die Standardartefakte trägt (Erkennungskette: `design.md` §1). `log.md`/`memory.md` liegen per Default auf Stakeholder-Ebene (L2); nur bei bestehendem Split auf Projekt-Ebene. Wenn nicht eindeutig: frage nach.

2. **`log.md` lokalisieren — Split-Probe ist Pflicht.** Prüfe mechanisch (`ls`), ob auf der Projekt-Ebene des Session-Themas eigene `log.md`/`memory.md` existieren. Existieren sie → das Projekt ist gesplittet, dort wird geschrieben; sonst Stakeholder-Ebene (L2). Nicht aus dem Bestand schließen: Dass ältere projektbezogene Einträge in L2 liegen, ist kein Beleg für L2 — es kann Drift sein. Der geprüfte Pfad-Befund ist Pflichtbestandteil des Bestätigungsblocks; eine Lokalisierung ohne Probe hat nicht stattgefunden. Existiert nirgends eine `log.md`: frage den User, ob du sie anlegen sollst (kein Auto-Create).

3. **Session-Kontext auswerten.** Lies den bisherigen Gesprächsverlauf der aktuellen Session. Identifiziere:
   - Was ist passiert? (Events, Entscheidungen, Ergebnisse)
   - Warum ist es relevant? (Kausalität, strategische Bedeutung)
   - Was hat sich verändert? (Kursänderungen, neue Erkenntnisse)
   - Welche Urteile oder Einschätzungen wurden getroffen?
   - Welche Artefakte wurden erzeugt?

4. **Eintrag schreiben.** Hänge einen neuen Eintrag im Log-Format an `log.md` an. Ein `/remember`-Aufruf erzeugt genau einen Eintrag — auch wenn die Session mehrere Themen berührt hat. Fasse zusammen, splitte nicht. **Ausnahme Misch-Session bei Split:** Berührt die Session ein gesplittetes Projekt UND Stakeholder-Themen, entstehen zwei getrennte Einträge — der projektspezifische Teil ins Projekt-Log, der Rest nach L2 (nie den ganzen Misch-Eintrag auf eine Ebene zwingen).

5. **Task-Closure.** Wurde in der Session ein offener Punkt aus `context.md` §„Offene Punkte" erledigt:
   1. Entferne ihn aus `context.md` (das Arbeits-Scaffolding ist verbraucht — kein Verlust).
   2. Falls „in 6 Monaten erinnernswert": eine Zeile in `log.md` („TT.MM. X getan/übergeben"). Triviales nur prunen, ersatzlos.
   3. Hängt ein Urteil dran (warum, mit welcher Folge): nach `memory.md`.

   Triage-Schwelle = genau diese 6-Monats-Frage.

6. **Bottom-up-Aktualisierung.** Nach dem Log-Eintrag prüfst und aktualisierst du in einem Durchlauf die darüberliegenden Ebenen. Der User soll nicht wissen müssen, welche Datei veraltet ist — das ist dein Job.

   **Messen (Pflicht, vor dem Prüfen):** Erhebe den Log-Stand mechanisch — nicht schätzen, nicht aus dem Gedächtnis:

   ```bash
   wc -c {pfad}/log.md {pfad}/context.md && grep -c "^- date:" {pfad}/log.md && grep "^- date:" {pfad}/log.md | sed 's/- date: //' | sort | head -1
   ```

   Ergebnis festhalten: **N** Einträge gesamt, davon **M** älter als die aktuelle Arbeitswoche, Dateigrößen in KB (log **und** context). Diese Zahlen sind Pflichtbestandteil des Bestätigungsblocks (Schritt 7) — eine Volumen-Prüfung ohne erhobene Zahlen hat nicht stattgefunden.

   **Prüfen:** Lies `memory.md` (L2; Projekt-`memory.md` nur falls Split) und `context.md` (alle relevanten Ebenen, falls vorhanden). Prüfe gegen den neuen Log-Eintrag und die bestehenden Log-Einträge:
   - **Volumen:** M ≥ 3 (Einträge älter als aktuelle Arbeitswoche) → verdichten. **Ab 20 KB Dateigröße ist die Verdichtung nicht mehr optional** — sie läuft im selben Durchlauf (wie Modus Verdichten, Schritte 2–4); Aufschieben nur, wenn der User es in dieser Session explizit abwählt.
   - **Widerspruch:** Log-Eintrag überholt eine Aussage in `memory.md` → korrigieren
   - **Lücke:** `memory.md` leer/nicht vorhanden bei ≥5 Log-Einträgen → erstmalig befüllen
   - **context.md veraltet:** Offensichtlich falsche Fakten (besetzte Rollen als offen markiert, falscher Projektstatus, überholte Strukturen) → direkt fixen, dabei das Backbone respektieren (Steckbrief / Warum / Lage / Richtung / Offene Punkte). Fälschlich in `context.md` stehende Übergänge/Urteile nach `memory.md` verschieben.
   - **Split:** Prüfe, ob ein Projekt eigene `memory.md`/`log.md` verdient (siehe „## Split").
   - **context.md zu groß:** > ~20 KB (Faustregel, justierbar) → **nicht autonom umbauen**, sondern diagnostizieren und dem User als Befund vorlegen: (a) eingesickerte Chronologien/Urteile → memory/log, (b) Register-Fall (abgeschlossen, aber betrieblich referenzpflichtig, z.B. Dedup) → eigenes Register-Artefakt neben der context.md, (c) eigenständiger Fach-Block → Fach-Artefakt + Pointer, (d) legitim groß → belassen. Geduldete Genre-Fremdkörper (Dossiers) und Register-Artefakte sind ausgenommen. **Bestätigt der User eine Auslagerung (Fall b/c), läuft der Vollzug im selben Durchlauf nach dem Auslagerungs-Protokoll** (Spec §2: verlustfreier Schnitt, Pointer-Rückstand mit einem Satz Substanz, Ausweis) — nicht als vertagte Empfehlung enden lassen. Spezifikation: `../../context_system/design.md` §2 Größen-Heuristik.

   **Handeln:** Wenn Signale vorliegen, handle autonom — nicht fragen, sondern tun:
   - Memory verdichten/aktualisieren (Log → Memory, L2)
   - Stakeholder-Memory aktualisieren, wenn sich etwas Wesentliches verändert hat
   - context.md-Stellen korrigieren, die durch die Session offensichtlich überholt sind (Backbone wahren)
   - Verdichtete Log-Einträge aus `log.md` entfernen
   - Bei klar erfülltem Split-Kriterium: Projekt-`memory.md`/`log.md` abspalten

   Nur bei echten Grenzfällen (unklar ob eine Aussage überholt ist, mehrdeutige Signale, Split-Grenzfall) nachfragen.

7. **Bestätigung.** Zeige dem User kompakt, was passiert ist — in einem Block:
   - Den geschriebenen Log-Eintrag
   - **Pflichtzeile Log-Stand** (nie weglassen — sie ist der Nachweis, dass die Volumen-Prüfung gelaufen ist): `Log: N Einträge (X KB), M älter als aktuelle Woche → verdichtet` bzw. `→ nicht verdichtet, weil {Grund}` · dahinter `context.md: Y KB` (ab ~20 KB mit Befund gemäß Diagnose-Fächer). Ohne die context-Zahl gilt auch die Größen-Prüfung der context.md als nicht gelaufen.
   - Was in welchen Dateien aktualisiert/verdichtet/abgespalten wurde (falls etwas passiert ist)
   - Falls darüber hinaus nichts nötig war: Log-Eintrag + Log-Stand-Zeile genügen

### 3. Modus: Init

1. **Pfad parsen.** Erwarte `init {Stakeholder}` oder `init {Stakeholder}/{Projekt}`.

2. **Ordner anlegen.** Erstelle die Ordnerstruktur im Workspace.

3. **Leere Dateien anlegen:**
   - Bei `init {Stakeholder}`: `memory.md` (mit Frontmatter + Header, leer) und `log.md` (mit Header, ohne Frontmatter) auf Stakeholder-Ebene.
   - Bei `init {Stakeholder}/{Projekt}`: nur den Projekt-Ordner. **Keine** Projekt-`memory.md`/`log.md` — die entstehen erst bei einem Split. Die Projekt-`context.md` erstellt der User separat via `/contextify`.
   - `memory.md` erhält oben `created`/`updated` (beide = heutiges Datum) gemäß Dokument-Frontmatter-Konvention. `log.md` ist per Eintrag datiert und bekommt keine Frontmatter.
   - Keine `context.md` — die erstellt der User separat via `/contextify`.

4. **Bestätigung.** Zeige die angelegte Struktur.

### 4. Modus: Verdichten

1. **Ebene bestimmen.** Frage den User oder leite aus dem Kontext ab:
   - **Log → Memory (Default, L2):** Verdichte ältere `log.md`-Einträge in die `memory.md` derselben Ebene. "Älter" = alles was nicht aus der aktuellen Arbeitswoche stammt (Faustregel, User kann abweichen).
   - **Projekt-Memory → Stakeholder-Memory (nur bei Split):** Verdichte eine abgespaltene Projekt-`memory.md` in die Stakeholder-`memory.md`. Typisch bei Projektabschluss oder wenn sich auf Stakeholder-Ebene etwas Wesentliches verändert hat.

2. **Quelle lesen.** Lies die zu verdichtende Datei vollständig.

3. **Verdichten.** Wende die Verdichtungsprinzipien an:
   - Kausalitäten und Urteile bleiben. Details und Zwischenschritte gehen.
   - Was am Ende zählt: Was ist passiert, warum, und was hat es verändert?
   - Bei Log → Memory: mehrere Log-Einträge können zu einem Memory-Eintrag verschmelzen, wenn sie kausal zusammenhängen.
   - Bei Projekt → Stakeholder: nur die Essenz — "CIP-Projekt: 15k, abgeschlossen Dez 2026, Consumer Insights Team ist offen für Automatisierung."

4. **Ergebnis schreiben.** Schreibe/aktualisiere die Ziel-`memory.md`. Bei Log-Verdichtung: entferne die verdichteten Einträge aus `log.md` (die Information lebt jetzt in `memory.md`).

5. **Bestätigung.** Zeige was verdichtet wurde und was in der Ziel-Datei steht.

## Split: memory/log L2 → L3

Prüfpunkt im Bottom-up-Schritt. Ein Projekt bekommt eigene `memory.md`/`log.md` nur, wenn ALLE drei gelten:

1. **Volumen:** Stakeholder-`memory.md` so groß, dass Mitlesen bei projektfremden Prompts spürbar Ballast ist, UND der Projekt-Strang ist ein großer, klar abgrenzbarer Block. Faustregel-Auslöser zur Prüfung: `memory.md` > ~15 KB und ein Projekt ≳ ein Drittel davon (Zahl justierbar).
2. **Severabilität:** Die Projekt-Einträge zeigen nach innen — kein ständiges Referenzieren von Schwester-Projekten oder Stakeholder-weiten Fakten. Sonst: kein Split.
3. **Eigenleben:** genug eigene Historie für eine sinnvolle separate Verdichtung. Keine präventiven Splits frischer Projekte.

Protokoll: Default = nicht splitten (Aufschieben billig, Zusammenführen teuer, ein verfrühter Split zerschneidet das Bindegewebe). Klarer Fall (alle drei deutlich, keine Quer-Referenzen) → autonom ausführen + im Bestätigungsblock melden. Grenzfall → vorlegen. `log` und `memory` splitten gemeinsam. **Der Vollzug hinterlässt einen Marker:** In den Kopf des L2-Logs gehört eine Zeile „*Split-Hinweis: `{Projekt}` führt eigenes Log + Memory (Split TT.MM.) — Projekt-Einträge gehören dorthin.*“ — der Split darf nie nur Dateisystem-Zustand sein, sonst schreiben spätere Sessions daran vorbei.

## Log-Format (`log.md`)

Die Datei beginnt mit einem Header, gefolgt von YAML-Einträgen. Neueste Einträge stehen oben.

```markdown
# Log — {Stakeholder}

```yaml
- date: 2026-03-18
  event: Sandra signalisiert Budget und neue Projektideen
  body: |
    Sandra teilt im Jour fixe mit: PPT Builder als Use Case, Budget
    freigeräumt für mehr LLM-Projekte, Termin mit Jonas Wendt
    (Group Data & AI) wird eingestellt. Strategisch wichtig:
    Sandra öffnet proaktiv die Governance-Tür. Bisher liefen alle
    Projekte bewusst unter IT-Radar — das ändert sich jetzt.
  artifacts:
    - /Herbanea/drafts/ppt_builder_concept.md

- date: 2026-03-16
  event: POC Report Lavendel-Linie aus echten Daten gebaut
  body: |
    Interaktiven HTML-Report aus Excel-Daten generiert. Wird ohne
    CIP-Branding übergeben — soll wirken wie fertiger Analyst-Output,
    nicht wie Pipeline-Demo. Zeigt was CIP kann, ohne "Pipeline"
    zu sagen.
  artifacts:
    - /Herbanea/consumer_insights_pipeline/example_charts/poc_report_lavendel.html
`` `
```

### Eintrags-Prinzipien

- **Ein Eintrag = ein Übergang.** Etwas hat sich verändert. Nicht "wir haben gearbeitet", sondern "wir haben erkannt / entschieden / gelernt".
- **Urteile explizit machen.** "Nicht ganz sauber gegenüber dem Angebot, aber strategisch sinnvoll" — das ist der Wert einer Memory, nicht die Fakten drumherum.
- **Kausalität über Chronologie.** Nicht "erst A, dann B, dann C" sondern "A hat zu B geführt, weil C."
- **Artefakte verlinken, nicht beschreiben.** Was im Artefakt steht, gehört nicht in die Memory.
- **Kein Task-Tracking.** Keine offenen Punkte, keine TODOs, keine Status-Felder. (Tasks und ihr aktueller Stand leben in `context.md` §Offene Punkte — nicht in memory.)
- **Kein Status-Tracking.** "Projekt ist aktiv" gehört in `context.md`. Memory erfasst Übergänge, nicht Zustände.
- **Artefakt-Links zeigen auf Dauerhaftes.** `temp.md` (flüchtiges Arbeitsblatt) ist kein stabiles Ziel — Inhalt, der referenziert werden soll, zuerst in ein dauerhaftes Artefakt ernten; sonst den flüchtigen Charakter im Eintrag vermerken.

## Memory-Format (`memory.md`)

Prosa-Markdown, verdichtet. Im Inhalt keine YAML-Einträge, keine starre Struktur — die Struktur ergibt sich aus dem Inhalt, manchmal chronologisch, manchmal thematisch, manchmal beides. Oben steht die Dokument-Frontmatter (`created`/`updated`): bei jeder Aktualisierung von `memory.md` `updated` auf das heutige Datum hochsetzen. Dasselbe gilt für `context.md`, wenn die Bottom-up-Aktualisierung (Schritt 6) sie korrigiert.

Die Default-Heimat ist die Stakeholder-Ebene (L2). Das Projekt-Memory-Beispiel unten zeigt den Split-Fall.

### Projekt-Memory

```markdown
---
created: 2026-03-01
updated: 2026-03-18
---
# Memory — {Projekt}

## Wie es dazu kam

CIP entstand als Zufallsfund im Effizienz-Projekt: Petra (Consumer Insights)
beschrieb ihren manuellen Auswertungsprozess — 2-3 Tage pro Studie, 90-Slide-Decks,
kein Signifikanztest. Der Automatisierungshebel war offensichtlich.

## Wesentliche Entscheidungen

Three-Container-Architektur gewählt (Data → Stats → LLM), weil die statistische
Analyse deterministisch sein muss — kein LLM für Signifikanztests. POC bewusst ohne
CIP-Branding übergeben, um Pipeline-Diskussion zu vermeiden bevor der Wert sichtbar ist.

## Was wir gelernt haben

Consumer Insights ist eine der datenintensivsten Funktionen bei Herbanea, läuft aber
komplett manuell. Das Team ist offen für Automatisierung, hat aber keine IT-Anbindung
und keinen eigenen Tech-Zugang.
```

### Stakeholder-Memory

```markdown
---
created: 2025-11-15
updated: 2026-03-18
---
# Memory — {Stakeholder}

## Zusammenarbeit

Seit Spätherbst 2025 über eine Empfehlung. Sandra (CDO) wurde früh Test-Kundin,
daraus entwickelte sich die Consulting-Beziehung. Sandra führt ohne direktes Mandat
(75% Dotted-Line) — Kommunikation ist ihr wichtigstes Steuerungsinstrument.

## Projekte

**Effizienz-Projekt (10k, aktiv):** KI-Coaching für Sandras 6 Direct Reports.
Use Cases aus Breakout Sessions waren weniger reif als dargestellt — echte Use Cases
werden über Einzelinterviews identifiziert.

**CIP (12-15k, Concept):** Automatisierte Consumer-Insights-Pipeline. Entstand als
Zufallsfund aus dem Effizienz-Projekt. Adressiert blinden Fleck — datenintensivste
Funktion, komplett manuell.

**AI-Kompass (on hold):** Herbaneas IT baut eigene Lösung. Proposal liegt fertig vor,
ist aktuell nicht platzierbar.

## Was wir über sie wissen

Sandra denkt in Amazon Leadership Principles. "Boring Basics first" — nicht mit
KI-Visionen einsteigen. Budget ist freigeräumt, Governance-Tür zu IT öffnet sich
(Termin mit Jonas Wendt). Momentum ist da.
```

## Harte Gates

- **Kein Auto-Create.** Dateien und Ordner nur auf explizite Anweisung anlegen.
- **Kein Task-Tracking in memory.** Keine TODOs, keine offenen Punkte, keine Checklisten. (Tasks und ihr aktueller Stand leben in `context.md` §Offene Punkte.)
- **Kein Status-Tracking in memory.** Zustände gehören in `context.md`, nicht in Memory.
- **Keine Erfindung.** Nur festhalten, was in der Session tatsächlich passiert oder besprochen wurde.
- **Verdichtung löscht die Quelle.** Was verdichtet wurde, wird aus der Quelldatei entfernt — die Information lebt jetzt eine Ebene höher.
- **Ein `/remember`-Aufruf = ein Eintrag.** Nicht splitten, nicht mehrere Einträge für eine Session.
