---
name: protocol
description: Use when ein rohes Meeting-Transkript (Teams, Gemini, .vtt, .docx oder Inline-Text) in ein strukturiertes, teilbares Protokoll verwandelt werden soll. Triggert auf /protocol, „Protokoll erstellen", „Transkript verarbeiten", „Meeting auswerten", „Call Notes".
---

## Zweck

Rohes Transkript → **Protokoll** als Markdown-Artefakt. Eine reine Transformation: Der Skill liest keine und schreibt in keine Kontext-Artefakte (`context.md`, `memory.md`, `log.md`) — was aus dem Protokoll ins eigene System fließt, entscheidet der Mensch danach.

Das Protokoll wird **besser als das Transkript**, nicht nur kürzer: Implizites wird explizit, Beschlüsse werden von Annahmen getrennt, offene Punkte werden als solche markiert, Transkriptionsfehler werden bereinigt.

Zwei Ebenen, strikt getrennt:

- **Protokoll** — teilbar. Maßstab: Es kann unverändert an alle Teilnehmer (auch Externe) gehen.
- **Interne Beobachtungen** — Beziehungsdynamik, Organisationspolitik, Stimmungen, Geschäftspotenziale. Stehen **nie** im Protokoll; sie werden im Chat-Output berichtet und nur auf Wunsch als eigene Datei geschrieben.

## Input

| Feld | Pflicht | Beschreibung |
|---|---|---|
| Transkript | Ja | Dateipfad (`.md`, `.txt`, `.vtt`, `.docx` → erst mit `markitdown` konvertieren) oder Inline-Text |
| Zielpfad | Nein | Fehlt er → neben das Transkript als `protokoll_JJJJ-MM-TT_<anlass>.md` |
| „mit interner Notiz" | Nein | Schreibt die internen Beobachtungen zusätzlich als `<zielpfad>_intern.md` |

## Workflow

1. **Transkript lesen.** Datum, Teilnehmer, Dauer, Anlass aus Kopf und Inhalt ziehen.

2. **Drei Filterschichten** über jede Aussage:
   - **Protokollrelevant** — Beschlüsse, Informationen, Zahlen, offene Punkte, Aufgaben → ins Protokoll.
   - **Intern** — Beziehungsdynamik, Politik, Stimmungen, beiläufig erwähnte Schmerzpunkte und Potenziale → in die interne Ebene, nie ins Protokoll.
   - **Rauschen** — Smalltalk, Technikprobleme, Wiederholungen → verwerfen.

3. **Transkript-Hygiene, still und konservativ:**
   - Offensichtliche Erkennungsfehler korrigieren, wenn der Kontext sie eindeutig macht (verhörte Tool- und Eigennamen); Namen auf eine Schreibweise normalisieren; Zahlen exakt übernehmen.
   - Nicht Auflösbares neutral-konservativ formulieren oder, wenn unwesentlich, weglassen. Nichts erfinden, keine Editorial-Marker im Artefakt — der Autor liest gegen und hakt inhaltlich ein.

4. **Kategorien scharf halten:**
   - **Beschluss** — wurde explizit bestätigt oder unwidersprochen festgehalten.
   - **Annahme** — geäußert, aber nicht bestätigt („ich gehe davon aus …", offen gelassene Antwort) → als Annahme kennzeichnen, nie zum Beschluss befördern.
   - **Aufgabe** — Wer, Was, Bis wann. Fehlt der Verantwortliche oder der Termin → `offen` eintragen, nicht erfinden.
   - **Offener Punkt** — explizit vertagt oder erkennbar ungelöst; auch Lücken benennen, die im Meeting selbst auffielen.

5. **Protokoll schreiben** (Format unten), Datei am Zielpfad ablegen.

6. **Chat-Output:** der Pfad des Protokolls und — kompakt — die internen Beobachtungen. Keine Prozess-Erzählung (was wann woraus korrigiert oder ergänzt wurde). Auf „mit interner Notiz" die Beobachtungen zusätzlich als Datei (Format unten). Interne Inhalte nie eigenmächtig in teilbare Artefakte übernehmen.

## Output-Format — Protokoll

Das Format folgt dem `doccontent`-Vertrag — wo eine Render-Strecke existiert (z.B. `/pdfrender` mit `doccontent.SCHEMA.md`), ist es damit direkt PDF-renderbar; beides ist optional und nicht Bestandteil dieses Skills.

```markdown
---
title: "Anlass des Meetings"
subtitle: "Kurzeinordnung, eine Zeile"     # optional
doc_type: protokoll
date: T. Monat JJJJ                        # Datum des Meetings
author: Organisation oder Verfasser        # optional
created: JJJJ-MM-TT                        # Dokument-Frontmatter-Konvention
updated: JJJJ-MM-TT
---

**Teilnehmende:** … · **Datum/Dauer:** …

## Zusammenfassung

3–5 Sätze: Anlass, wichtigste Ergebnisse, größter offener Punkt.

## Beschlüsse

1. Beschluss mit Kern und Kontext — nummeriert, zitierfähig.

## [Thematische Sektionen]

Aus dem Inhalt abgeleitet, nicht aus einem Template. Informationen, Positionen,
Zahlen; gekennzeichnete Annahmen. Tragende O-Töne kursiv als Zitat —
1–3 im ganzen Dokument, Priorität: Haltung > strategische Aussage > Kontext,
der sonst verloren geht.

## Offene Punkte

- Punkt mit Stand („vertagt auf …", „Klärung durch …")

## Aufgaben

| Wer | Was | Bis wann |
|---|---|---|
```

Keine leeren Sektionen. Struktur über Überschriften, kein Styling im Markdown.

## Output-Format — interne Notiz (nur auf Anforderung)

Gleiche Frontmatter-Logik (`title: "Interne Notiz — <Anlass>"`, ohne `doc_type`). Sektionen nach Bedarf: **Signale & Potenziale** (Schmerzpunkte, mögliches Folgegeschäft, Budget-Signale), **Beziehungsdynamik** (Spannungen, Allianzen, Stimmungen), **Beobachtungen** (Sonstiges zwischen den Zeilen). Mit Beleg aus dem Gespräch, nicht als Spekulation.

## Prinzipien

1. **Teilbarkeit ist der Filter.** Vor jedem Satz: Kann das unverändert an alle Teilnehmer gehen? Nein → interne Ebene.
2. **Kein Meta im Artefakt.** Das Protokoll liest sich, als hätte es ein Mensch nach dem Termin geschrieben: keine Quellen- und Prozess-Verweise (Transkript, Dateien, Korrekturen), keine Editorial-Marker, keine Erwähnung des Werkzeugs. Was *inhaltlich* offen blieb, steht unter „Offene Punkte" — das ist Substanz, kein Meta.
3. **Kein Informationsverlust über beide Ebenen.** Was nicht ins Protokoll darf, wird nicht verworfen, sondern intern berichtet.
4. **Beschluss ≠ Annahme.** Die häufigste Protokoll-Falle: Eine unbeantwortete Absichtserklärung wird als Entscheidung festgeschrieben.
5. **Nur was im Transkript steht.** Keine erfundenen Ownership-Zuordnungen, Termine oder Auflösungen; Unklares wird neutral-konservativ gefasst statt aufgelöst.
6. **Sektionen aus dem Inhalt** — Zwischenüberschriften spiegeln die tatsächlichen Themen, keine leeren Pflichtsektionen.
7. **Korrekte Umlaute** (ä, ö, ü, ß).

## Abgrenzung

- Verfassen des Protokolls: dieser Skill. Rendern als PDF: ein Render-Skill, falls vorhanden (z.B. `/pdfrender`). Einspeisen von Erkenntnissen in Kontext-Artefakte: `/contextify` bzw. `/remember`, mit dem Protokoll als Input — nie automatisch.
