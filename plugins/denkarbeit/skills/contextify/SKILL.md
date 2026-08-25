---
name: contextify
description: Use when creating or maintaining a `context.md` — the workspace standard context artifact — from raw inputs (calls, PDFs, notes, briefings). Enforces the context-system backbone. Triggers on /contextify.
---

## Zweck

Lege die `context.md` einer Workspace-Ebene an oder pflege sie — gegen das feste Backbone des Kontext-Systems. **Spec-Auflösung** (gilt für alle Verweise auf `design.md`/`template_context.md` in diesem Skill): der in der globalen CLAUDE.md verankerte Pfad (Sektion Referenzen bzw. Workspace-Konvention); Fallback: vom Skill-Verzeichnis aufwärts nach `context_system/design.md` suchen (liegt in der Plugin- bzw. Repo-Wurzel oberhalb von `skills/`). Inputs sind beliebig (Calls, PDFs, Notizen, Briefings, Inline-Text); das *Ziel* ist immer eine schema-konforme `context.md` im Zielordner. Nicht kürzen — in die Idealform der jeweiligen Ebene überführen: semantische Kompression mit Strukturierung. Jeder Token verdient seinen Platz, aber Telegrammstil ist kein Ideal — vollständige, knappe Sätze schlagen reine Stichworte.

`context.md` hält **was jetzt gilt** (Zustand). Übergänge, Urteile, Chronologien gehören nicht hierher (siehe Disziplin-Schnitt) — dafür sind `memory.md`/`log.md` da (`/remember`).

## Backbone (Pflichtstruktur)

Jede `context.md` hat fünf Top-Level-Sektionen in dieser Reihenfolge. Skelett: `template_context.md` neben der design.md (Spec-Auflösung s.o.). (Spiegelt design.md §2 — Änderungen dort zuerst.)

1. **Steckbrief** (Pflicht) — Kopf: ein Framing-Satz unter der H1, dann ebenen-spezifische Felder (s.u.). Datierung ausschließlich über die Frontmatter — keine `Stand:`-Zeile; Bestands-Zeilen beim Anfassen entfernen.
2. **Motivation** (Pflicht) — warum existiert das für mich, mein Einsatz, Verortung. Orientierung („worum geht es hier"), **nicht** Verhaltenssteuerung, **kein** Psychogramm.
3. **Lage** (nach Bedarf) — der Wissenskörper. **Frei untergliederbar** (eigene `##`/`###`). Hier lebt die inhaltliche Vielfalt: Org, Markt, Technik, Beziehungsstand.
4. **Richtung** (optional) — Zielbild / woran gearbeitet wird, als *Zustand*, nicht als To-do-Liste.
5. **Offene Punkte** (optional) — offene Tasks UND offene Fragen/Wissenslücken. Der aktuelle Stand eines offenen Punkts ist Zustand und gehört hierher; Erledigtes nicht (siehe Disziplin-Schnitt).

Fixer Kopf, freie Tiefe: Die fünf Überschriften liegen fest; unter „Lage" ist die Substruktur frei.

### Steckbrief-Felder — nach Inhalt des Ordners

Die Felder ergeben sich daraus, **was der Ordner beschreibt**, nicht aus seiner Pfadtiefe (Master: design.md §2):

- **Mich selbst** (die `context.md` im Workspace-Root): Eckdaten, Rollen, Angebot/Tätigkeit, Arbeitsweise; Domänen nur als Typ-Orientierung, kein gepflegtes Verzeichnis — point, don't duplicate.
- **Ein Gegenüber oder eigener Bereich** (Kunde, Arbeitgeber, Firma, privates Feld): Org-Eckdaten + **Personen-Block**. Muster: `**{Name}** — {Rolle/Funktion} · {Kontakt, optional} · {Hinweis, nur wenn sachlich}`. Org-weite Personen hier, vorhabenspezifische Kontakte im jeweiligen Vorhaben.
- **Ein Vorhaben** (Projekt, Arbeitspaket): Auftrag, Scope, Stand, Beteiligte (Verweis auf die Personen darüber statt Duplikat); bei Teilstücken die Abgrenzung zum übergeordneten Vorhaben.

Ein Ordner kann beides sein (ein Gegenüber mit genau einem Vorhaben) — dann trägt eine `context.md` beide Feldgruppen. Benennt der User den Gegenstand, gilt seine Ansprache. Bei Unklarheit nachfragen.

**Beobachtung, kein Vollzug:** Beschreibt eine bestehende `context.md` erkennbar mehrere getrennte Vorhaben, lege das als Befund vor (Spec §1, Strukturwechsel) — die Teilung ist Nutzerarbeit.

## Workflow

1. **Prompt parsen.** Extrahiere aus der Usereingabe:
   - **Rahmung** — Zweck und Scope der Transformation
   - **Inputs** — Dateipfade oder Inline-Text
   - **Zielpfad** — wohin die `context.md` geschrieben wird

2. **Gegenstand bestimmen.** Was beschreibt dieser Ordner (siehe oben)? Das bestimmt die Steckbrief-Felder.

3. **Rückfragen-Safeguard.** Wenn die Rahmung unklar ist (kein Zweck erkennbar, Scope zu vage), kein Zielpfad angegeben oder der Gegenstand nicht eindeutig: frage nach, bevor du transformierst. Kein stilles Raten.

4. **Modus bestimmen:**
   - **Neu** — keine Datei am Zielpfad → erzeuge gegen das Backbone von Grund auf.
   - **Erweitern** — bestehende Datei + neue Inputs → zusammenführen, Backbone wahren. Neue Information aktualisiert überlappende Sektionen, genuin Neues bekommt seinen Platz in der passenden Backbone-Sektion (Substanz unter „Lage"). Konflikte werden als Widerspruch markiert (Prinzip 3).
   - **Re-Run** — bestehende Datei, keine neuen Inputs → erneut gegen das Backbone formen. Der User muss die Rahmung mitgeben oder explizit „gleiche Rahmung" sagen — rate nicht.

5. **Inputs lesen.** Alle referenzierten Dateien und Inline-Texte einlesen.

6. **Context-Window-Gate.** Wenn die Inputs sehr umfangreich sind (geschätzt >80% des Context Windows): informiere den User, frage nach Priorisierung. Schneide niemals still ab.

7. **In das Backbone überführen.** Wende die Prinzipien an; ordne den Inhalt den fünf Sektionen zu; gliedere die Substanz unter „Lage" frei nach Inhalt. Orientiere dich an den Beispielen als Leitbild für das *Wie*.

8. **Quellen-Vollständigkeitsprüfung.** Gehe jede Input-Quelle einzeln durch und prüfe: Gibt es in dieser Quelle eine Informationsschicht, die im transformierten Output nicht vertreten ist? Besonders bei Dokumenten mit mehreren semantischen Ebenen (z.B. Zielgrößen + Arbeitspakete, Strategie + Maßnahmen) — jede Ebene muss eigenständig im Output auftauchen, auch wenn sie thematisch nahe an bereits übernommener Information liegt.

9. **Output schreiben.** Schreibe die `context.md` an den Zielpfad, inklusive YAML-Frontmatter (`created`/`updated`) gemäß Dokument-Frontmatter-Konvention.

10. **Bestätigungs-Ausweis.** Nenne im Bestätigungs-Output Modus, Quellen und Zeichenumfang vorher → nachher (kein Log-Kommentar in der Datei — die contextify-log-Telemetrie ist abgeschafft; Bestands-Blöcke beim Anfassen entfernen). Liegt der Output über ~20 KB: Größen-Heuristik-Befund vorlegen (Diagnose-Fächer in Spec §2 — eingesickerte Chronologien/Urteile, Register-Fall, Fach-Block oder legitim groß), nicht autonom auslagern. Bestätigt der User eine Auslagerung, vollziehe sie nach dem **Auslagerungs-Protokoll** (Spec §2): verlustfreier Schnitt in ein Fach-Artefakt neben der `context.md` (Frontmatter, H1, Framing-Satz; keine Mitverdichtung), Pointer-Rückstand mit einem Satz Substanz, Ausweis im Bestätigungs-Output.

## Prinzipien

Diese Prinzipien sind das *Wie* der Befüllung des Backbones — sie ersetzen die Pflichtstruktur nicht, sie füllen sie.

1. **Kein Informationsverlust** — alles, was laut Rahmung relevant ist, muss im Output sein.
2. **Implizites explizieren** — Zusammenhänge, Abhängigkeiten und Implikationen, die nur zwischen den Zeilen stehen, sichtbar machen. Sparsam, aber gezielt.
3. **Widerspruchsfreiheit** — Widersprüche auflösen. Wo das nicht eindeutig möglich ist: `<!-- Widerspruch: Quelle A sagt X, Quelle B sagt Y -->` markieren.
4. **Redundanz eliminieren** — gleiche Information nur einmal, am richtigen Ort.
5. **Positiv formulieren** — "wir machen kein X" in positive Handlungsanweisungen umformulieren, wo es die Klarheit erhöht. Nicht dogmatisch.
6. **Struktur dient dem Inhalt** — innerhalb des Backbones (vor allem unter „Lage") den Aufbau für jeden Output durchdenken. Flache Hierarchie bevorzugt (max. 2–3 Heading-Ebenen), klare Sektionen mit sprechenden Überschriften.
7. **Jeder Token verdient seinen Platz** — keine Füllsätze, keine Wiederholungen, keine schmückenden Formulierungen. Vollständige knappe Sätze schlagen reine Stichworte.

### Harte Gates

- Bei unklarer Rahmung oder unklarem Gegenstand: Rückfrage stellen, nicht raten.
- Erfinde keine Informationen, die nicht im Input stehen.
- Bei Inputs, die das Context Window übersteigen: User informieren, Priorisierung erfragen, nie still abschneiden.
- **Disziplin-Schnitt context.md:** Nicht hierher gehören — Chronologien/Zeitachsen, Urteile/Erkenntnisse, Betriebs-/Bedienungswissen über Systeme und Werkzeuge (→ `infra.md` der Ebene, Spec §3), sowie *erledigte* Tasks samt ihrer Verlaufshistorie (Erledigt-Marker, offen→done-Ketten). Der aktuelle Stand eines *offenen* Punkts („wartet auf X", „Entwurf liegt vor") ist Zustand und bleibt. Begegnet Verlaufs-, Urteils- oder Erledigt-Material im Input → nach `memory.md`/`log.md` verweisen, nicht hier aufnehmen, knapp im Bestätigungs-Output vermerken. **Ausnahme operatives Register:** Abgeschlossenes, das betrieblich referenzpflichtig bleibt (z.B. ein Dedup-Register), gehört weder in die `context.md` noch ins Log, sondern in ein eigenes Register-Artefakt neben der `context.md` — die `context.md` hält nur den Pointer (Spec §5).
- **Opportunistische Migration:** Triffst du eine Bestands-`context.md` ohne Backbone, bring sie beim Anfassen auf die Struktur (kein separater Massen-Retrofit). Genre-Fremdkörper (reine Dossiers/Fallakten) nicht in die Form pressen.
- **Personen-Block:** kein Psychogramm — nur sachliche Rolle, Zuständigkeit, Kommunikationsweg.

## Output-Format

Reines Markdown. Top-Level die fünf Backbone-Sektionen mit `##`, Substruktur (vor allem unter „Lage") mit `###`, maximal 3 Ebenen tief. **YAML-Frontmatter** (`created`/`updated`) gehört an den Dateianfang — `context.md` fällt unter die Dokument-Frontmatter-Konvention. HTML-Kommentare sind erlaubt für Meta-Annotationen (Widerspruch-Marker, offene Fragen). Korrekte deutsche Umlaute verwenden (ä, ö, ü, ß) — niemals ae, oe, ue, ss als Ersatz.

Wenn ein Fakt oder KPI in mehreren Quellen identisch vorkommt, steht er genau einmal im Output — an der thematisch passenden Stelle. Nicht pro Quelle wiederholen, nicht in mehreren Sektionen duplizieren.

## Beispiele

Die folgenden Beispiele illustrieren das *Wie* (die Prinzipien): die transformierten Sektionen leben typischerweise unter „Lage", das Backbone (Steckbrief, Motivation, …) liegt verbindlich darum herum.

--- Beispiel 1 — Prosa-Briefing (Kompression) ---

**Rahmung:** Entscheidungsgrundlage für den Vorstand zur Marktexpansion NordLog GmbH nach Polen.

**Input:**

> Die NordLog GmbH hat in den letzten Jahren eine starke Position im norddeutschen Logistikmarkt aufgebaut. Vor dem Hintergrund der strategischen Wachstumsziele für die kommenden Jahre prüft die Geschäftsführung nun intensiv eine mögliche Expansion in den polnischen Markt, der mit einem Volumen von ca. 48 Mrd. EUR als einer der dynamischsten Logistikmärkte in Mittelosteuropa gilt. Dabei ist zu berücksichtigen, dass bereits drei große deutsche Wettbewerber — Schenker, Hellmann und Dachser — in Polen aktiv sind und zusammen rund 12% Marktanteil halten. Regulatorisch ist vor allem die polnische Kabotage-Regelung relevant, die seit 2022 verschärft wurde und den Einsatz ausländischer Fahrzeuge für Inlandstransporte begrenzt. Was in den bisherigen Analysen noch nicht ausreichend berücksichtigt wurde: Der polnische Arbeitsmarkt für LKW-Fahrer ist extrem angespannt — die Branche meldet aktuell ca. 15.000 unbesetzte Stellen landesweit.

**Output:**

## Markt

Polnischer Logistikmarkt: ~48 Mrd. EUR Volumen, einer der dynamischsten in Mittelosteuropa.

## Wettbewerb

Schenker, Hellmann und Dachser sind bereits in Polen aktiv (zusammen ~12% Marktanteil).

## Regulatorik

Verschärfte Kabotage-Regelung seit 2022: Einsatz ausländischer Fahrzeuge für polnische Inlandstransporte ist begrenzt.

## Risiken

Fahrermangel: ~15.000 unbesetzte LKW-Fahrer-Stellen in Polen. Betrifft Personalaufbau und Betriebskontinuität direkt.

**Warum:** Narrativer Fließtext auf vier thematische Sektionen verdichtet. Der Fahrermangel war im Original als Nebenbemerkung vergraben — hier als explizites Risiko mit Auswirkung auf den Markteintritt herausgestellt. Redundante Floskeln ("in den letzten Jahren", "prüft nun intensiv") entfernt. Alle Fakten erhalten.

--- Beispiel 2 — Stichpunktartige Notizen (Implizites explizieren) ---

**Rahmung:** Kontextdokument zur Team-Restrukturierung DataPuls AG für den neuen CTO.

**Input:**

> - ML übernimmt Ops + Dev (ab April)
> - SK bleibt Teamlead Data, aber nur noch 60%
> - Budget: offen, FK will Freigabe bis Q3
> - Schnittstelle Vertrieb: unklar, bisher lief das über TH
> - TH geht Ende März
> - 2 offene Stellen Dev, 1 Ops — Ausschreibung läuft
> - Retro zeigte: Team will klarere Zuständigkeiten
> - Nächstes Steering: 14.04.

**Output:**

## Rollen und Zuständigkeiten

ML übernimmt ab April die Leitung von Ops und Dev (Doppelrolle). SK bleibt Teamlead Data mit reduziertem Pensum (60%).

Die Vertriebsschnittstelle ist ungeklärt. TH hat sie bisher verantwortet und verlässt das Unternehmen Ende März — es gibt keinen designierten Nachfolger für diese Funktion.

## Offene Stellen

Drei Positionen ausgeschrieben: zwei Dev, eine Ops. Ausschreibung läuft.

## Budget

Budgetfreigabe steht aus. FK strebt Freigabe bis Q3 an.

## Team-Feedback

Retro ergab: Team wünscht klarere Zuständigkeiten. Relevant im Kontext der anstehenden Umstrukturierung.

## Offene Punkte

Steering-Termin 14.04. — offen: Vertriebsschnittstelle, Budgetfreigabe und Zuständigkeitsklärung.

<!-- Offen: Wer übernimmt die Vertriebsschnittstelle nach THs Weggang? -->
<!-- Offen: Ist MLs Doppelrolle Ops+Dev dauerhaft oder Übergang? -->

**Warum:** Abkürzungen aufgelöst und AKV-Zuordnung (Aufgaben, Kompetenzen, Verantwortung) explizit gemacht. Die implizite Lücke — THs Abgang hinterlässt die Vertriebsschnittstelle unbesetzt — war im Original über zwei getrennte Stichpunkte verteilt und nirgends als Problem benannt. Offene Fragen als `<!-- Offen -->` markiert statt erfunden.

--- Beispiel 3 — Widersprüchliche Quellen (Synthese) ---

**Rahmung:** Statusübersicht Projekt Aurora für das Management-Review.

**Input:**

> **Report A (Projektleitung, 08.03.):**
> Projekt Aurora liegt im Plan. Go-Live ist weiterhin für Q2 vorgesehen, Gesamtbudget 340k EUR. Team ist vollständig besetzt, Entwicklung zu 70% abgeschlossen. Kritischer Pfad: API-Integration mit dem Partnersystem.
>
> **Report B (Controlling, 10.03.):**
> Projekt Aurora: Budget aktuell bei 280k EUR zzgl. externe Beratungskosten (noch nicht final beziffert). Go-Live-Termin wurde in der letzten Steering-Sitzung auf Q3 verschoben. Entwicklungsfortschritt laut Ticketsystem: 58%.

**Output:**

## Status Entwicklung

Entwicklung ist mehrheitlich abgeschlossen. Kritischer Pfad: API-Integration Partnersystem.

<!-- Widerspruch: Projektleitung meldet 70% Fortschritt (08.03.), Controlling misst 58% via Ticketsystem (10.03.). Unterschied möglicherweise durch verschiedene Messmethoden. -->

## Zeitplan

<!-- Widerspruch: Report A (08.03.) nennt Go-Live Q2, Report B (10.03.) nennt Q3 nach Steering-Entscheidung. Report B ist aktueller — Q3 wahrscheinlich der gültige Termin, aber Bestätigung einholen. -->

## Budget

Kernbudget: 280–340k EUR. Externe Beratungskosten kommen hinzu (noch nicht final beziffert).

<!-- Widerspruch: Projektleitung nennt 340k Gesamtbudget, Controlling 280k zzgl. externe Kosten. Unklar ob 340k die externen Kosten bereits enthält. -->

## Team

Team ist vollständig besetzt (laut Projektleitung).

**Warum:** Fakten, die in beiden Reports übereinstimmen (Team besetzt, API-Integration kritisch), stehen je einmal — nicht pro Quelle wiederholt. Alle drei Widersprüche (Fortschritt, Go-Live, Budget) sind als `<!-- Widerspruch -->` transparent markiert mit Quellenangabe und Datumskontext. Keine eigenmächtige Auflösung, aber Hinweis wo ein Report aktueller ist.

## Typischer Fehler: Subsumption

Ein Dokument enthält zwei Informationsschichten — z.B. ein OKR-Dokument mit Zielgrößen (Objective + Key Results) und Arbeitspaketen (Working Packages). Die Arbeitspakete sind detailliert und konkret, die Key Results wirken wie "die Klammer drum herum". Ergebnis: Nur die WPs werden übernommen, die KRs mit ihren messbaren Schwellenwerten, Methodik und Milestones gehen verloren.

**Das ist falsch.** Thematische Nähe ist nicht Redundanz. Zwei Informationsschichten im selben Dokument sind zwei Informationsschichten im Output. Die Quellen-Vollständigkeitsprüfung (Workflow Schritt 8) existiert genau für diesen Fall.

