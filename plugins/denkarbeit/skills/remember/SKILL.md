---
name: remember
description: Use when the user wants to persist session knowledge (log entry), compress older entries into memory, or set up memory files for a folder. Triggers on /denkarbeit:remember.
---

## Zweck

Persistiere Wissen, das sonst mit der Session verloren geht — Entscheidungen, Erkenntnisse, Kursänderungen, Urteile. Nicht Fakten cachen, sondern Erfahrung festhalten.

Verdichtungsstufen:

1. **Log** (`log.md`) — Ereignisstrom, session-nah. Einziger Schreiber ist dieser Skill.
2. **Memory** (`memory.md`) — verdichtet aus dem Log: Urteile, Übergänge, das Warum.

Wo sie liegen, bestimmt die **Pfad-Mechanik**: Geschrieben wird in die **nächste vorhandene Datei aufwärts** vom Session-Ordner bis zum Workspace-Root. Neu angelegt wird nur in einem Ordner, der bereits eine `context.md` trägt, und nur auf Ansage. „Vergessen" ist gewollte Kompression, kein Datenverlust.

**Wo Gedächtnis hingehört:** *so hoch wie es gilt, so tief wie möglich.* Ein Urteil, das auch außerhalb des aktuellen Ordners gilt, gehört weiter nach oben — dort sehen es mehr Sessions, und ein neu angelegtes Vorhaben erbt es automatisch. Zu tief abgelegtes Wissen ist unsichtbar verloren; zu hoch abgelegtes kostet nur Token.

**Nicht alles gehört in log/memory** (Genre-Weiche, Schritt 4): Betriebslektionen — Werkzeug-Eigenheiten, Fehlerbilder samt Fix, System-Bedienung — gehören in die `infra.md` des Ordners, für den sie gelten. Beobachtete Sprach-Drift gehört in die Fallenliste der `language.md`, falls es sie gibt. Log und Memory tragen Übergänge und Urteile.

`context.md` wird über `/denkarbeit:contextify` gepflegt, nicht von diesem Skill.

## Workflow

### 1. Modus bestimmen

Parse die Usereingabe und bestimme den Modus:

- **Log** (default) — schreibe einen Eintrag in das nächste `log.md` aufwärts
- **Verdichten** (`verdichten`) — komprimiere ältere Einträge in die memory
- **Anlegen** (`init {pfad}`) — lege `log.md` + `memory.md` in einem Ordner an, der eine `context.md` trägt (auf Ansage, siehe unten)

### 2. Modus: Log (default)

1. **Ort bestimmen — Pfad-Probe ist Pflicht.** Scanne mechanisch (`ls`) den Pfad vom Session-Ordner aufwärts bis zum Workspace-Root und stelle fest: Wo liegt das **nächste `log.md`**? Dort wird geschrieben. Nicht aus dem Bestand schließen, sondern prüfen — dass ältere Einträge zu diesem Thema woanders liegen, ist kein Beleg, es kann Drift sein. Der Pfad-Befund ist Pflichtbestandteil des Bestätigungsblocks; eine Lokalisierung ohne Probe hat nicht stattgefunden. Existiert auf dem ganzen Pfad kein `log.md`: frage den User, wo eins entstehen soll (Vorschlag: der nächste Ordner mit `context.md`) — kein Auto-Create. Trägt der Arbeitsordner selbst keine `context.md`, wandert der Eintrag mechanisch weiter nach oben, als sein Inhalt gilt. Das ist zulässig, aber nennenswert: Betrifft der Eintrag erkennbar nur diesen Ordner, im Bestätigungsblock den Weg dahin nennen — erst `context.md` anlegen, dann `init {ordner}`.

2. **Scope prüfen.** Gilt das, was die Session hervorgebracht hat, auch außerhalb dieses Ordners? Dann gehört es weiter nach oben. Berührt die Session erkennbar **beide** Bereiche — das laufende Vorhaben und den Rahmen darüber —, entstehen zwei Einträge: der spezifische Teil ins nächste Log, der übergreifende in das darüber. Sonst genau ein Eintrag.

3. **Session-Kontext auswerten.** Lies den bisherigen Gesprächsverlauf der aktuellen Session. Identifiziere:
   - Was ist passiert? (Events, Entscheidungen, Ergebnisse)
   - Warum ist es relevant? (Kausalität, strategische Bedeutung)
   - Was hat sich verändert? (Kursänderungen, neue Erkenntnisse)
   - Welche Urteile oder Einschätzungen wurden getroffen?
   - Welche Artefakte wurden erzeugt?

4. **Genre-Weiche, dann Eintrag schreiben.** Erst die Weiche, zwei Ausgänge:
   - **Betriebslektion** (Werkzeug-Eigenheit, Fehlerbild + Fix, System-Bedienung) → `infra.md` des Ordners, für den sie gilt (Scope-Regel); existiert dort keine, schlage das Anlegen vor (kein Auto-Create).
   - **Sprach-Drift** — hat der User in dieser Session eine Formulierung zurückgewiesen, umgeschrieben oder ein wiederkehrendes Muster benannt? → als Falle mit Gegenbeispiel in die Fallenliste der `language.md` im Workspace-Root. Nur bei vorhandener Datei; keine anlegen, keine Stilregeln ergänzen. Das ist der einzige Schreib-Moment dieses Artefakts — ohne ihn altert die Fallenliste vom Tag ihrer Entstehung an.

   Der Log-Eintrag referenziert beides allenfalls in einem Halbsatz, trägt es aber nicht aus. Bei der Gelegenheit: Verfallsbedingungen der berührten `infra.md` prüfen („Flag kann raus, sobald …"). Dann der Eintrag: Setze ihn im Log-Format an den **Kopf** des YAML-Blocks der `log.md` (neueste oben). Ein `/denkarbeit:remember`-Aufruf erzeugt genau einen Eintrag — auch wenn die Session mehrere Themen berührt hat. Fasse zusammen, splitte nicht. Zwei Einträge entstehen nur im Scope-Fall aus Schritt 2.

5. **Task-Closure.** Führt die `context.md` keine Sektion „Offene Punkte" oder ist sie leer, entfällt dieser Schritt ersatzlos. Sonst, für jeden in der Session erledigten Punkt — Triage mit vier Ausgängen:
   1. Entferne ihn aus `context.md` (das Arbeits-Scaffolding ist verbraucht — kein Verlust).
   2. Falls „in 6 Monaten erinnernswert": das Faktum als Satz **im Body des Session-Eintrags** („X getan/übergeben") — kein eigener Log-Eintrag. Triviales nur prunen, ersatzlos.
   3. Hängt ein Urteil dran (warum, mit welcher Folge): nach `memory.md`.
   4. Bleibt der Vorgang betrieblich referenzpflichtig (erledigt ≠ irrelevant, z.B. Dedup-Pflicht): in das **Register-Artefakt** des Ordners — existiert keines, schlage das Anlegen vor; die `context.md` hält den Pointer.

   Triage-Schwelle = die 6-Monats-Frage.

6. **Bottom-up-Aktualisierung.** Nach dem Log-Eintrag prüfst und aktualisierst du in einem Durchlauf die Artefakte auf dem Pfad nach oben. Der User soll nicht wissen müssen, welche Datei veraltet ist — das ist dein Job.

   **Messen (Pflicht, vor dem Prüfen):** Erhebe den Log-Stand mechanisch — nicht schätzen, nicht aus dem Gedächtnis:

   ```bash
   wc -c {pfad}/log.md {pfad}/context.md ; grep -c "^- date:" {pfad}/log.md ; grep "^- date:" {pfad}/log.md | sed 's/- date: //' | sort | head -1
   ```

   Ergebnis festhalten: **N** Einträge gesamt, davon **M** älter als die aktuelle Arbeitswoche, Dateigrößen in KB (log **und** context). Diese Zahlen sind Pflichtbestandteil des Bestätigungsblocks (Schritt 7) — eine Volumen-Prüfung ohne erhobene Zahlen hat nicht stattgefunden.

   **Prüfen:** Lies alle `memory.md` und `context.md` auf dem Pfad. Prüfe gegen den neuen Log-Eintrag und die bestehenden Log-Einträge:
   - **Volumen:** M ≥ 3 (Einträge älter als aktuelle Arbeitswoche) → verdichten. **Ab 20 KB Dateigröße ist die Verdichtung nicht mehr optional** — sie läuft im selben Durchlauf (wie Modus Verdichten, Schritte 2–4); Aufschieben nur, wenn der User es in dieser Session explizit abwählt.
   - **Widerspruch:** Log-Eintrag überholt eine Aussage in `memory.md` → korrigieren
   - **Lücke:** `memory.md` leer/nicht vorhanden bei ≥5 Log-Einträgen → erstmalig befüllen
   - **context.md veraltet:** Offensichtlich falsche Fakten (besetzte Rollen als offen markiert, falscher Projektstatus, überholte Strukturen) → direkt fixen, dabei das Backbone respektieren (Steckbrief / Warum / Lage / Richtung / Offene Punkte). Fälschlich in `context.md` stehende Übergänge/Urteile nach `memory.md` verschieben.
   - **Struktur:** Beschreibt eine `context.md` erkennbar mehrere getrennte Vorhaben, oder sammelt ein Unterordner ohne eigene `context.md` Substanz? Als Befund vorlegen (Strukturwechsel), nie selbst vollziehen.
   - **temp-Erinnerung:** Führt der Ordner eine `temp.md` mit ungeerntetem Inhalt (`updated` jünger als der jüngste Log-Eintrag), im Bestätigungsblock daran erinnern — geerntet wird im Dialog, nicht vom Skill.
   - **Pointer-Probe** (mechanisch, ein `ls` gegen die Pointer der `context.md`): Zeigt ein Pointer auf eine Datei, die es nicht mehr gibt? Liegt neben der `context.md` ein Artefakt, auf das kein Pointer zeigt? Beides als Befund vorlegen, nicht selbst auflösen. Register- und Werkzeuge-Artefakte werden sonst von keinem Schritt je wieder angefasst; ein toter oder fehlender Pointer ist das einzige Signal, das ihre Verwaisung sichtbar macht.
   - **context.md zu groß:** > ~20 KB (Faustregel, justierbar) → **nicht autonom umbauen**, sondern diagnostizieren und dem User als Befund vorlegen: (a) eingesickerte Chronologien/Urteile → memory/log, (b) Register-Fall (abgeschlossen, aber betrieblich referenzpflichtig, z.B. Dedup) → eigenes Register-Artefakt neben der context.md, (c) eigenständiger Fach-Block → Fach-Artefakt + Pointer, (d) legitim groß → belassen. Geduldete Genre-Fremdkörper (Dossiers) und Register-Artefakte sind ausgenommen. **Bestätigt der User eine Auslagerung (Fall b/c), läuft der Vollzug im selben Durchlauf nach dem Auslagerungs-Protokoll** (verlustfreier Schnitt, Pointer-Rückstand mit einem Satz Substanz, Ausweis) — nicht als vertagte Empfehlung enden lassen.

   **Handeln:** Wenn Signale vorliegen, handle autonom — nicht fragen, sondern tun:
   - Memory verdichten/aktualisieren (Log → nächste memory aufwärts) — dabei **verschmelzen statt anhängen**: in bestehende thematische Abschnitte integrieren, keine neuen datierten Abschnitte stapeln. Sieht die memory aus wie ein zweites Log (chronologische Anhänge), ist die Konsolidierung der Abschnitte fällig
   - Stakeholder-Memory aktualisieren, wenn sich etwas Wesentliches verändert hat
   - context.md-Stellen korrigieren, die durch die Session offensichtlich überholt sind (Backbone wahren). **Das geschieht hier, nicht über `/denkarbeit:contextify`** — der Skill ist für Neuanlage aus Rohmaterial und strukturellen Umbau da. Ein zusätzlicher Aufruf am Session-Ende kostet Kontext ohne Zusatznutzen.
   - Verdichtete Log-Einträge aus `log.md` entfernen
   - **Pointer-Pflege:** Entstand im Durchlauf ein neues Artefakt (infra, Register, Fach-Artefakt), den Pointer in der `context.md` desselben Ordners (Steckbrief bzw. Lage) im selben Durchlauf setzen

   Nur bei echten Grenzfällen (unklar ob eine Aussage überholt ist, mehrdeutige Signale) nachfragen.

7. **Bestätigung.** Zeige dem User kompakt, was passiert ist — in einem Block:
   - Den geschriebenen Log-Eintrag
   - **Pflichtzeile Log-Stand** (nie weglassen — sie ist der Nachweis, dass die Volumen-Prüfung gelaufen ist): `Log: N Einträge (X KB), M älter als aktuelle Woche → verdichtet` bzw. `→ nicht verdichtet, weil {Grund}` · dahinter `context.md: Y KB` (ab ~20 KB mit Befund gemäß Diagnose-Fächer). Ohne die context-Zahl gilt auch die Größen-Prüfung der context.md als nicht gelaufen.
   - **Weichen-Ausweis:** wohin Betriebslektionen und Sprach-Drift geroutet wurden — oder „nichts geroutet" (ohne diese Zeile gilt die Genre-Weiche als nicht gelaufen)
   - Was in welchen Dateien aktualisiert/verdichtet/abgespalten wurde (falls etwas passiert ist)
   - Falls darüber hinaus nichts nötig war: Log-Eintrag + Log-Stand-Zeile + Weichen-Ausweis genügen

### 3. Modus: Anlegen

`init {pfad}` legt in diesem Ordner `log.md` (Header, keine Frontmatter — per Eintrag datiert) und `memory.md` (Frontmatter + Header) an. Voraussetzung: Der Ordner trägt eine `context.md`; sonst nachfragen, ob sie zuerst entstehen soll (`/denkarbeit:contextify`). Beide Dateien entstehen gemeinsam — ein Log ohne Verdichtungsziel läuft voll, eine Memory ohne Zufluss verhungert. Ab dem nächsten Lauf greift die Pfad-Mechanik von selbst; ein Marker oder Hinweis anderswo ist nicht nötig.

### 4. Modus: Verdichten

1. **Quelle und Ziel bestimmen.** Verdichtet wird das `log.md`, das die Pfad-Probe liefert, in die **nächste `memory.md` aufwärts** (im Normalfall derselbe Ordner). „Älter" = alles, was nicht aus der aktuellen Arbeitswoche stammt (Faustregel, User kann abweichen). Wird ein Vorhaben abgeschlossen, kann sein Gedächtnis auf Ansage in das darüberliegende verschmolzen werden — nur die Essenz, nicht der Bestand.

2. **Quelle lesen.** Lies die zu verdichtende Datei vollständig.

3. **Verdichten.** Wende die Verdichtungsprinzipien an:
   - Kausalitäten und Urteile bleiben. Details und Zwischenschritte gehen.
   - Was am Ende zählt: Was ist passiert, warum, und was hat es verändert?
   - Bei Log → Memory: mehrere Log-Einträge können zu einem Memory-Eintrag verschmelzen, wenn sie kausal zusammenhängen.
   - **Verschmelzen statt anhängen:** in bestehende thematische Abschnitte der memory integrieren, keine datierten Abschnitte stapeln — die Struktur folgt dem Inhalt, nicht dem Kalender.
   - Betriebslektionen, die im Log stecken, wandern bei der Verdichtung in die `infra.md`, für die sie gelten, nicht in die memory.
   - Bei Vorhaben → Gegenüber: nur die Essenz — was das Vorhaben über die Zusammenarbeit gelehrt hat, nicht sein Verlauf.

4. **Ergebnis schreiben.** Schreibe/aktualisiere die Ziel-`memory.md`. Bei Log-Verdichtung: entferne die verdichteten Einträge aus `log.md` (die Information lebt jetzt in `memory.md`).

5. **Bestätigung.** Zeige was verdichtet wurde und was in der Ziel-Datei steht.

## Ein eigenes Gedächtnis für ein Vorhaben

Kein Vorgang, ein Handgriff: `log.md` und `memory.md` im Ordner des Vorhabens anlegen (Modus „Anlegen"). Die Pfad-Mechanik findet sie ab dem nächsten Lauf, weil immer die nächste Datei aufwärts gilt.

**Wann es sich lohnt** (Erfahrung, kein Prüfpunkt): genug eigene Historie für eine sinnvolle Verdichtung · Einträge, die nach innen zeigen statt ständig auf Nachbarn zu verweisen · ein Umfang, bei dem Mitlesen in fremden Sessions spürbar Ballast wird. **Wann nicht:** vorsorglich bei frischen Vorhaben, denn Aufschieben ist billig und Zusammenführen teuer. Beobachtest du diese Signale, lege sie als Befund vor; angelegt wird auf Ansage.

## Log-Format (`log.md`)

Die Datei beginnt mit einem Header, gefolgt von YAML-Einträgen. **Neueste Einträge stehen oben** — nie vom Dateiende lesen.

````markdown
# Log — {Stakeholder}

```yaml
- date: 2026-03-18
  event: Auftrag dreht sich von der Auswertung auf die Erhebung
  body: |
    Im Jour fixe wurde klar, dass die Auswertungslogik nicht das
    Problem ist — die Daten kommen unvollständig an. Wir bauen
    also keinen besseren Report, sondern setzen eine Stufe früher
    an. Das ist nicht der bestellte Umfang und trotzdem der
    richtige Schnitt; {Kundin} trägt ihn mit.
  artifacts:
    - /{ordner}/erhebung_ist_analyse.md

- date: 2026-03-16
  event: Prototyp bewusst ohne unser Branding übergeben
  body: |
    Der Report geht als fertiges Arbeitsergebnis raus, nicht als
    Werkzeug-Demo. Wer das Ergebnis überzeugend findet, fragt nach
    dem Weg dorthin von selbst.
  artifacts:
    - /{ordner}/report_q1.html
```
````

### Eintrags-Prinzipien

- **Ein Eintrag = ein Übergang.** Etwas hat sich verändert. Nicht "wir haben gearbeitet", sondern "wir haben erkannt / entschieden / gelernt".
- **Urteile explizit machen.** "Nicht ganz sauber gegenüber dem Angebot, aber strategisch sinnvoll" — das ist der Wert einer Memory, nicht die Fakten drumherum.
- **Kausalität über Chronologie.** Nicht "erst A, dann B, dann C" sondern "A hat zu B geführt, weil C."
- **Artefakte verlinken, nicht beschreiben.** Was im Artefakt steht, gehört nicht in die Memory.
- **Kein Task-Tracking.** Keine offenen Punkte, keine TODOs, keine Status-Felder. (Tasks und ihr aktueller Stand leben in `context.md` §Offene Punkte — nicht in memory.)
- **Kein Status-Tracking.** "Projekt ist aktiv" gehört in `context.md`. Memory erfasst Übergänge, nicht Zustände.
- **Artefakt-Links zeigen auf Dauerhaftes.** `temp.md` (flüchtiges Arbeitsblatt) ist kein stabiles Ziel — Inhalt, der referenziert werden soll, zuerst in ein dauerhaftes Artefakt ernten; sonst den flüchtigen Charakter im Eintrag vermerken.

## Memory-Format (`memory.md`)

Prosa-Markdown, verdichtet. Im Inhalt keine YAML-Einträge, keine starre Struktur — aber **thematische Abschnitte, nicht Kalender-Anhänge**: Die Verdichtung integriert in bestehende Abschnitte; die Leserichtung folgt der Kausalität (Grundlegendes zuerst), memory wird ganz gelesen. Oben steht die Dokument-Frontmatter (`created`/`updated`): bei jeder Aktualisierung von `memory.md` `updated` auf das heutige Datum hochsetzen. Dasselbe gilt für `context.md`, wenn die Bottom-up-Aktualisierung (Schritt 6) sie korrigiert.

### Beispiel

```markdown
---
created: 2025-11-15
updated: 2026-03-18
---
# Memory — {Stakeholder}

## Zusammenarbeit

Seit Herbst 2025 über eine Empfehlung. Der Einstieg lief über eine Testnutzung,
daraus wurde der Beratungsauftrag. {Kundin} führt ohne direktes Mandat —
Kommunikation ist ihr wichtigstes Steuerungsinstrument.

## Was die Vorhaben gelehrt haben

Die im Workshop gesammelten Anwendungsfälle waren weniger reif als dargestellt;
belastbar wurden sie erst über Einzelinterviews. Der tragfähigste Strang entstand
als Zufallsfund daneben und traf einen blinden Fleck: die datenintensivste Funktion
des Hauses arbeitet vollständig manuell. Der erste Vorschlag scheiterte nicht
inhaltlich, sondern an einer parallelen Eigenentwicklung der IT — er bleibt gültig,
ist aber nicht platzierbar.

*(Namen, Zahlen und Status stehen in der `context.md` — memory trägt das Warum,
keine Statusliste.)*

## Was wir über die Zusammenarbeit wissen

Entscheidungen fallen bei {Kundin} über Prinzipien, nicht über Einzelfälle. Wer mit
einer Vision einsteigt, verliert sie; wer beim konkreten Ärgernis anfängt, gewinnt
sie. Diese Beobachtungen gehören hierher, weil sie die Arbeit steuern — eine
Charakterbeschreibung wäre etwas anderes und hat in keinem Artefakt Platz.
```

## Harte Gates

- **Kein Auto-Create neuer Genres.** Fehlt auf dem Pfad eine `log.md`, `infra.md` oder ein Register: fragen, nicht anlegen. Ausgenommen ist die erstmalige memory-Befüllung ab 5 Log-Einträgen — dort wird gehandelt und im Bestätigungsblock ausgewiesen.
- **Keine Erfindung.** Nur festhalten, was in der Session tatsächlich passiert oder besprochen wurde.
- **Verdichtung löscht die Quelle.** Was verdichtet wurde, wird aus der Quelldatei entfernt.
