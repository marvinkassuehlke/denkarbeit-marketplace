---
name: research
description: Use when researching a topic from scratch via web search to build a structured knowledge document. Triggers on /research.
---

## Zweck

Erforsche ein Thema, zu dem keine oder unzureichende Unterlagen vorliegen, und baue ein strukturiertes Wissensdokument auf. Der Output ist keine Quellensammlung, sondern kohärent gemachtes Wissen — synthetisiert, strukturiert, mit expliziten Zusammenhängen und benannter Heterogenität.

## Workflow

1. **Prompt parsen.** Extrahiere aus der Usereingabe:
   - **Rahmung** — Was genau soll erforscht werden? Welcher Aspekt, welche Tiefe, welche Perspektive?
   - **Zielpfad** — wohin das Output-Dokument geschrieben wird
   - **Ephemeres filtern** — Relative Zeitangaben (nächste Woche, gestern, gerade, bald), situative Verweise und anderes verderbliches Material aus dem Prompt identifizieren. Diese Informationen entweder in absolute Angaben umwandeln (wenn das Datum bestimmbar ist) oder nicht in den Output übernehmen.

2. **Rückfragen-Safeguard.** Wenn die Rahmung unklar ist (Thema zu breit, Erkenntnisinteresse nicht erkennbar): frage nach, bevor du recherchierst. Wenn kein Zielpfad angegeben: frage nach. Kein stilles Raten.

3. **Erhebungsstufe wählen.** Default ist die eigene Recherche (Schritte 4–8). Bei breiter, kontroverser oder faktenkritischer Rahmung (viele Teilthemen, strittige Behauptungen, hohe Fehlerkosten einer Falschangabe) den nativen `/deep-research` als vorgelagerte Erhebungsstufe nutzen — die in Schritt 2 geklärte Rahmung wird als fertige Forschungsfrage übergeben; dessen Rückfragen-Gate entfällt dadurch (die Klärung ist bereits passiert).

   Mit Erhebungsstufe schalten die Recherche-Schritte von Voll- auf **Lücken-Recherche**:
   - Der Report ist Arbeitsmaterial, keine Quelle: Seine **Originalquellen** wandern ins Quellenverzeichnis; der Report selbst wird dort nie geführt. Synthese-Schlüsse des Reports sind Sekundär-Urteile und werden als solche attribuiert (Prinzip 3), nie als Quellen-Fakt übernommen. Inline-Zitate des Reports nicht übernehmen — es gilt der eigene Format-Vertrag.
   - **Bleibt eigenständig:** der Aktualitäts-Check (Schritt 6 — der Report hat keinen Versions-Vertrag) und die Heterogenitäts-Prüfung (Schritt 8): Verifikation und Heterogenität sind verschiedene Dinge — der Report prüft, *ob Behauptungen stimmen*; dieser Skill kartiert zusätzlich, *welche Positionen es gibt*. Reports tendieren zur einen Antwort — prüfen, ob legitime Positionsvielfalt plattgezogen wurde, und sie gezielt nachrecherchieren.
   - Eigene Suchen (Schritte 4 und 7) nur noch gezielt für Rahmungs-Schwerpunkte, die der Report nicht deckt.
   - Der research-log vermerkt die Erhebungsstufe (siehe Log-Konvention).

4. **Erste Recherche (Landkarte).** Breit suchen, um die Struktur des Themas zu verstehen:
   - Welche Teilthemen gibt es?
   - Wie hängen sie zusammen?
   - Wo liegt der natürliche Schwerpunkt relativ zur Rahmung?

   Ziel dieser Runde: eine Gliederung im Kopf, nicht schon den Output.

5. **Schwerpunkt identifizieren.** Aus der Landkarte ableiten, welche Abschnitte den Kern des Themas ausmachen — also die Abschnitte, die bei oberflächlicher Recherche zu dünn bleiben würden. Dem User kurz mitteilen, welche Schwerpunkte du vertiefst.

6. **Aktualitäts-Check.** Bei Themen mit Versionshistorie (Standards, Normen, Gesetze, Frameworks): gezielt nach der aktuellsten Fassung und deren Änderungen suchen. Ältere, SEO-stärkere Quellen dominieren Suchergebnisse — daher explizit mit Jahreszahl und Begriffen wie „Neufassung", „Änderungen", „aktuelle Fassung" suchen. Wenn die Landkarte eine ältere Version als aktuell behandelt: korrigieren, bevor die Tiefenrecherche beginnt.

7. **Zweitrecherche (Tiefe).** Jeden identifizierten Schwerpunktabschnitt mit eigenen, spezifischen Suchbegriffen vertiefen. Nicht „IDW S6 Gliederung" erneut suchen, sondern „Integrierte Sanierungsplanung Ertrags- Bilanz- Liquiditätsplanung Anforderungen". Die Verbindung zum Gesamtthema in den Suchbegriffen beibehalten.

8. **Heterogenitäts-Recherche.** Gezielt nach der Vielfalt innerhalb des Themas suchen:
   - Gibt es verschiedene Schulen, Ansätze, Interpretationen?
   - Wo herrscht Uneinigkeit in der Praxis?
   - Welche Varianten oder Sonderfälle existieren?

   Nicht messen, wie heterogen ein Thema ist — sondern benennen, worin die Heterogenität besteht. Wenn das Thema homogen ist (ein klarer Standard, eine herrschende Meinung), ist das auch ein Ergebnis — dann diesen Schritt kurz halten.

9. **Synthese und Strukturierung.** Aus allen Recherche-Runden ein kohärentes Dokument bauen. Nicht Quellen nacheinander referieren, sondern thematisch strukturieren. Wende die Prinzipien an.

10. **Quellen-Vollständigkeitsprüfung.** Prüfe: Gibt es Teilthemen, die in der Recherche aufgetaucht sind, aber im Output fehlen? Besonders bei Themen mit mehreren Ebenen (z.B. Struktur + Inhalt + Praxis + Recht) — jede Ebene muss im Output vertreten sein.

11. **Output schreiben.** Schreibe das Research-Dokument als `.md` an den Zielpfad. Setze ganz oben die Frontmatter `created`/`updated` (beide = heutiges Datum, ISO 8601) gemäß Dokument-Frontmatter-Konvention. Bei einer Überarbeitung eines bestehenden Research-Dokuments nur `updated` hochsetzen.

12. **Log-Eintrag anhängen.** Füge einen research-log-Eintrag am Dateiende an (siehe Log-Konvention). Die Frontmatter bildet den Dokument-Lebenszyklus ab, der research-log die Recherche-Statistik — beide bleiben bestehen.

## Prinzipien

1. **Kohärenz vor Vollständigkeit** — lieber ein zusammenhängendes Bild mit benannten Lücken als eine lose Faktensammlung ohne roten Faden.
2. **Heterogenität benennen** — wo verschiedene Ansätze, Interpretationen oder Praxisvarianten existieren, diese explizit darstellen. Nicht eine Perspektive als „die Wahrheit" präsentieren, wenn es mehrere gibt.
3. **Wertungen attribuieren** — Meinungen, Einordnungen und Handlungsempfehlungen sind wertvoller Bestandteil einer Research. Sie müssen aber korrekt eingeordnet sein: Wer sagt das, auf welcher Basis, in welchem Kontext? Eigene Synthese-Schlüsse als solche kennzeichnen. Nie eine fremde Wertung als objektiven Fakt präsentieren, nie eine eigene Schlussfolgerung einer Quelle zuschreiben.
4. **Schwerpunkt respektieren** — jedes Thema hat einen natürlichen Kern. Diesem Kern gebührt Tiefe, den Randthemen Kürze. Nicht alles gleich gewichten.
5. **Implizites explizieren** — Zusammenhänge zwischen Teilthemen sichtbar machen, die in den Einzelquellen nicht verbunden werden.
6. **Widersprüche markieren** — wo Quellen sich widersprechen: `<!-- Widerspruch: Quelle A sagt X, Quelle B sagt Y -->` markieren. Nicht still eine Seite wählen.
7. **Redundanz eliminieren** — gleiche Information nur einmal, am thematisch richtigen Ort.
8. **Struktur dient dem Inhalt** — Dokumentaufbau für jeden Output durchdenken. Flache Hierarchie bevorzugt (max. 2–3 Heading-Ebenen), klare Sektionen mit sprechenden Überschriften.
9. **Jeder Token verdient seinen Platz** — keine Füllsätze, keine Wiederholungen. Vollständige knappe Sätze schlagen reine Stichworte.
10. **Output altert ohne Kontext** — Das Dokument muss ohne Kenntnis des Erstellungsdatums, des Prompts und der Session lesbar bleiben. Keine relativen Zeitangaben („nächste Woche", „gestern"), keine unaufgelösten Pronomen auf Prompt-Kontext, keine Annahmen über den Wissensstand des Lesers, die nur in der Erstellungssession galten.

### Harte Gates

- Bei unklarer Rahmung: Rückfrage stellen, nicht raten.
- Log-Kommentar am Dateiende — immer.
- Erfinde keine Informationen, die nicht in den Quellen stehen. Wenn du etwas nicht verifizieren kannst, kennzeichne es als unsicher.
- Keine stille Verkürzung: Wenn die Recherche ein Thema als relevant identifiziert hat, darf es nicht im Output fehlen, weil „der Abschnitt schon lang genug ist".

## Output-Format

Reines Markdown. Sektionen mit `#`/`##`-Headings, maximal 3 Ebenen tief. YAML-Frontmatter ausschließlich für `created`/`updated` (Dokument-Konvention, siehe Schritt 11) — keine weitere Frontmatter als Inhaltsstruktur, keine XML-Tags als Inhaltsstruktur. HTML-Kommentare sind erlaubt für Meta-Annotationen (Log, Widerspruch-Marker, offene Fragen). Korrekte deutsche Umlaute verwenden (ä, ö, ü, ß) — niemals ae, oe, ue, ss als Ersatz.

Wenn ein Fakt in mehreren Quellen identisch vorkommt, steht er genau einmal im Output — an der thematisch passenden Stelle.

Quellen werden nicht inline zitiert, sondern als Quellenverzeichnis am Ende des Dokuments aufgeführt (vor dem Log). Format: nummerierte Liste mit Titel und URL. Bei Nutzung der Erhebungsstufe: die Originalquellen des Reports, nie den Report selbst.

## Log-Konvention

Füge am Ende jedes Output-Dokuments einen Log-Eintrag als HTML-Kommentar an:

```html
<!-- research-log
YYYY-MM-DD | rahmung: [kurze Beschreibung] | erhebung: [eigene | deep-research] | searches: [Anzahl eigener Suchen] | sources: [Anzahl verwerteter Quellen] | chars: [Zeichenanzahl Output]
-->
```

Felder pro Eintrag: Datum, Rahmung (Kurzform), Erhebungsstufe, Anzahl eigener Suchen (bei Erhebungsstufe: nur die Nachrecherchen), Anzahl tatsächlich verwerteter Quellen, Zeichenanzahl des Outputs.
