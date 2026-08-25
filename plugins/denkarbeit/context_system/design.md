---
created: 2026-06-15
updated: 2026-08-25
---

# Kontext-System — Design

Kanonische Spezifikation des Kontext-Systems im `/workspace`: die Standardartefakte (`context.md` / `memory.md` / `log.md` / `temp.md` / `infra.md`), ihre Hierarchie, ihr Zusammenspiel und die Konsequenzen für `CLAUDE.md` und die Skills `contextify` / `cleanup` / `remember`. **SSOT-Modell:** Diese Spec trägt das Modell — Genres, Orte, Kreisläufe, Kriterien. Die Skills tragen die Ausführung und dürfen Kernregeln bewusst spiegeln, damit sie eigenständig lauffähig sind; gespiegelte Passagen tragen einen Vermerk (`Master: design.md §X`), und Änderungen laufen zuerst hier, dann im Skill nach. Formatkonventionen (Log-/Memory-Form) sind hier normativ definiert (§3) — der Skill führt sie aus.

## Zweck & Geltung

Das System war gewachsen, nicht spezifiziert; es funktioniert gut — dies ist Optimierung, kein Rebuild. Leitprinzip: **`context.md` ist und bleibt die *eine* Wissensdatei pro Ordner.** Kein Zerlegen in `stakeholder.md` / `objective.md` o.ä. Die Grundstruktur ist bewusst weit abstrahiert, damit „alles reinpasst"; dass Einzelfälle das Konstrukt strapazieren, ist akzeptiert.

**Enforcement-Doktrin** (gilt für jede Regel dieser Spec): *Eine Regel, deren Bedingung der Normallauf nicht misst, existiert nicht.* Mess-Pflicht (mechanisch erheben, nicht schätzen) + Ausweis-Pflicht (Ergebnis als Pflichtzeile im Output) + harte Gates nur, wo groß immer falsch ist. Jedes Artefakt-Genre braucht einen definierten **Lese-Moment**, **Schreib-Moment** und **Hygiene-Moment** — ein Genre ohne alle drei verwaist (Empirie: `infra.md` 08/2026). Maschinelle Härtung (Hooks) darf diese Momente **härten, nie tragen**: Das System muss rein prompt-basiert funktionieren, weil nicht jede Harness Hooks kennt (Cowork).

## 1. Architektur — rekursive Hierarchie, Rollen statt Tiefen

Die Regeln hängen an drei **Rollen**, nicht an Pfadtiefen:

```
Wurzel   /workspace/context.md      ICH             — wer bin ich, was tue ich
Anker    {Domäne}/context.md        DIE ANDEREN     — Gegenüber oder eigener Bereich;
                                                      Default-Heimat von log.md + memory.md
Knoten   {…}/context.md             ARBEITSGEGENSTAND — Projekt, Arbeitspaket, beliebig tief;
                                                      context.md ja, log/memory nur per Split (§6)
```

- **Das Normalbild sind drei benannte Ebenen** — Ich → Stakeholder → Projekt („dreistufig" im Kurs-Sprech). Mehr Tiefe ist erlaubt und **ändert keine Regel**: Ein Knoten unter einem Knoten ist wieder ein Knoten. Wer fünf Ebenen einzieht, bekommt fünf `context.md`-Knoten — die Split-Kriterien halten log/memory automatisch knapp. Die historischen L-Kürzel (L1=Wurzel, L2=Anker, L3/L4=Knoten) bleiben als Kurzschreibweise zulässig.
- **Anker ist nicht zwingend „extern".** Auch eigene Domänen sind Anker: `strgaGmbH/` (das Unternehmen), `trading/`, `johan/`. „Stakeholder" bleibt der kanonische Name (benannt nach dem häufigsten Bewohner); Kurs-Definition: *für wen oder was du arbeitest — Arbeitgeber, Kunde oder eigener Bereich.*
- **Die Wurzel bleibt schlank.** Identität + Domänen-Orientierung (welche *Typen* es gibt) + Motivation auf höchster Ebene. **Keine gepflegte Domänen-Aufzählung** und keine duplizierten Anker-Fakten — die Wurzel verweist (insb. nicht `strgaGmbH/` spiegeln).
- **State komponiert abwärts:** Jede Ebene erbt den Kontext der Elternebene und verengt ihn. Jede `context.md` ist eigenständig lesbar.

### Rollen-Erkennung

- **Rolle ≠ Pfadtiefe.** Die Tiefe ist Default-Kodierung, keine Semantik. Sammel- und Zwischenordner sind **ebenen-transparent** und bekommen nie eigene Kontext-Artefakte — das gilt für thematische Zwischenordner (Programm-Ebenen) wie für Verwaltungs-Sammelordner am Root (`inaktiv/`, `Archiv/` — beide Vokabeln existieren, beide gleichbehandeln): `/Archiv/Altkunde` ist genauso Anker wie `/Altkunde`. Davon zu unterscheiden ist das ebenen-lokale **`archive/`** (cleanup-Ablage für ersetzte Dateien innerhalb einer Ebene) — es ist keine Ebene, sondern ein Ordner-Genre.
- **Erkennungskette** (für alle Skills verbindlich): (1) semantische Ansprache des Users · (2) Artefakt-Bestand — Rollen-Träger ist der Ordner, der die Standardartefakte trägt oder tragen sollte · (3) Pfadtiefe als Default. Im Zweifel nachfragen.
- **Träger = Repo:** Ist der Rollen-Träger selbst ein Git-Repo (der Ordner enthält `.git`), leben die Kontext-Artefakte trotzdem in seiner Wurzel — die Repo-Eigenschaft ändert die Rolle nicht; ob sie committet werden, entscheidet der User (ggf. `.gitignore`). Das Master-Muster (Repo hält den Master, der Workspace Symlinks) bleibt der Sonderfall für geteilte Artefakte.

## 2. context.md — Backbone

**Fixer Kopf, freie Tiefe.** Top-Level ist fix, die Substruktur bleibt frei.

| # | Sektion | Frage | Inhalt | Stabilität |
|---|---|---|---|---|
| 1 | **Steckbrief** | Was/wer ist das? | 1 Framing-Satz unter der H1, dann rollen-spezifische Felder (s.u.) | hoch |
| 2 | **Motivation** | Warum existiert das für mich, was ist mein Einsatz? | Motivation, Einsatz, Verortung — Orientierung, keine Verhaltenssteuerung, kein Psychogramm | hoch |
| 3 | **Lage** | Wie steht es? | Der Wissenskörper. **Frei untergliederbar** — Org-Bäume, Marktbilder, Tech-Stacks, Beziehungsstände | lebendig |
| 4 | **Richtung** | Wohin? | Zielbild, woran gearbeitet wird — als *Zustand*, nicht als To-do-Liste | lebendig |
| 5 | **Offene Punkte** | Was ist unerledigt? | Offene Tasks **und** offene Fragen / Wissenslücken | lebendig |

- **Fünf Top-Level-Sektionen**, Steckbrief als eigene `## Steckbrief` (gelebte Konvention, hier geadelt). **Quasi-Pflicht:** Steckbrief + Motivation. **Nach Bedarf:** Lage. **Optional:** Richtung / Offene Punkte.
- **Datierung:** ausschließlich über die Frontmatter (`created`/`updated`). Eine `Stand:`-Zeile im Kopf gibt es nicht mehr — zwei Datierungen driften (Empirie 08/2026); Bestands-Zeilen werden beim Anfassen entfernt.
- **Stabilitäts-Gefälle statt statisch-vs-lebendig:** Steckbrief + Motivation quasi-statisch, der Rest lebendig. Es braucht kein zweites Artefakt für „das Statische".

### Steckbrief-Felder je Rolle

- **Wurzel (Ich):** Eckdaten, Rollen, Angebot/Tätigkeit, Arbeitsweise-Grundzüge, Domänen-Orientierung (Typen, kein Verzeichnis).
- **Anker (Stakeholder/Bereich):** Org-Steckbrief (Firma/Entität, Eckdaten) **+ Personen-Block** (s.u.) + Pointer auf `infra.md`, falls vorhanden.
- **Knoten (Projekt/Arbeitspaket):** Auftrag, Scope, Stand, Beteiligte (Verweis auf Anker-Personen statt Duplikat); bei tieferen Knoten zusätzlich die Abgrenzung zum Eltern-Knoten.

### Personen-Erfassung

- **Organisation = Anker-Ordner.** Personen = Einträge im Steckbrief der Ebene, auf der die Beziehung lebt: org-weite Person → Anker; projekt-spezifischer Kontakt → Knoten (mit Rückverweis).
- **Ein Feld-Muster:** `Name · Rolle/Funktion · Kontakt (optional) · Hinweis (optional)`. Der Hinweis trägt nur sachliche Information (Zuständigkeit, Entscheidungsmacht, Kommunikationsweg).
- **Grenze zum Personenwissen:** Der Steckbrief hält kein Psychogramm. **Arbeitsrelevantes Beziehungswissen** (wie jemand entscheidet, was ihn überzeugt, welche Themen heikel sind) ist legitim — sein Ort ist die `memory.md` des Ankers, als Urteil aus der Zusammenarbeit. Nicht legitim, in keinem Artefakt: Charakter-Dossiers und Bewertungen der Person jenseits der Arbeitsbeziehung.

### Disziplin-Schnitt

**Raus aus `context.md`:** Chronologien/Zeitachsen und Urteile/Erkenntnisse → `memory.md` (Fakten-Splitter gelangen über `/remember` ins Log — contextify schreibt nie ins Log, §3) · Erledigt-Marker → ersatzlos bzw. als Faktum über die Task-Closure (§5) · To-do-Mechanik mit Status-Tracking → nur offene Punkte bleiben · Betriebs-/Bedienungswissen → `infra.md` (§3).

**Rein in `context.md`:** Sicherheitsgrade **`[BELEGT]` / `[ANNAHME]`** (genau diese zwei; freie Begründungszusätze wie `[ANNAHME — Quelle X]` sind erlaubt) · Widerspruch-Marker `<!-- Widerspruch: Quelle A sagt X, Quelle B sagt Y -->` · offene Fragen als `<!-- Offen: … -->`.

### Größen-Heuristik (weiche Regel)

Eine `context.md` jenseits **~20 KB** ist ein Prüfsignal, kein Urteil. Diagnose-Fächer: eingesickerte Chronologien/Urteile (→ Disziplin-Schnitt) · Register-Fall (→ §5) · eigenständiger Fach-Block (→ Fach-Artefakt + Pointer) · legitim groß (→ belassen). **Hart messen, weich handeln:** Gemessen wird mechanisch im `remember`-Bottom-up-Schritt; bei Überschreitung wird der Befund vorgelegt, nie autonom umgebaut. Geduldete Genre-Fremdkörper (private Dossiers) und Register-Artefakte sind ausgenommen.

**Auslagerungs-Protokoll (Fach-Block):** Bestätigt der User den Befund, ist der Vollzug `contextify`-Arbeit im selben Durchlauf: (1) **Verlustfreier Schnitt** in ein Fach-Artefakt neben der `context.md` derselben Ebene (Frontmatter, H1, Framing-Satz; keine Mitverdichtung) · (2) **Pointer-Rückstand** mit einem Satz Substanz · (3) **Ausweis** im Bestätigungsblock (chars vorher → nachher). Referenzmuster: `bewerbungsformate.md`, `duesseldry/datenschutz.md`.

## 3. Die Artefakte — Rollen, Formate, Kreisläufe

Drei Wissensartefakte, drei verschiedene Fragen:

- **`context.md` = „was gilt jetzt".** Das integrierte, aktuelle Bild. **Vernichtet** Historie bewusst.
- **`memory.md` = „was sich geändert hat und warum".** Der Urteils- und Begründungspfad. **Bewahrt** Historie mit Wertung.
- **`log.md` = Ereignisstrom.** Session-nah, speist `memory.md`.

**Die Asymmetrie ist der Kern:** State komponiert abwärts (context verengt von der Wurzel zu den Knoten); **Judgment integriert aufwärts** (memory: Urteile entstehen aus dem Zusammenführen *über* Projekte hinweg und kristallisieren am Anker — sie verschmelzen, statt additiv zu stapeln).

### Formate (normativ)

- **`log.md`:** Header `# Log — {Name}`, dann ein YAML-Block mit Einträgen `- date: / event: / body: / artifacts:` — **neueste oben**. Keine Datei-Frontmatter (per Eintrag datiert). **Einziger Schreiber ist `/remember`**: Sessions editieren das Log nie direkt; die Verdichtung überführt ältere Einträge in die memory und entfernt sie aus dem Log. („Append-only" meint genau das — kein manuelles Editieren; es verbietet nicht die Verdichtung.) Führen mehrere Personen Einträge in denselben Ast, trägt der Eintrag ein `autor:`-Feld; bei einer Person entfällt es.
- **`memory.md`:** Prosa-Markdown mit Frontmatter, **thematische Abschnitte** — die Struktur folgt dem Inhalt, nicht dem Kalender. **Verschmelzungsregel:** Die Verdichtung integriert in bestehende Abschnitte, statt datierte Abschnitte anzuhängen — eine memory, die wie ein zweites Log aussieht (chronologische Anhänge), ist das Signal, dass die zweite Verdichtungsstufe fällig ist. Leserichtung folgt der Kausalität (Grundlegendes zuerst); memory wird ganz gelesen, nicht vom Ende.
- **Zwei Verdichtungsstufen:** Session → Log (Eintrag) · Log → Memory (Verdichtung, entfernt Quell-Einträge). Die dritte Bewegung ist die Verschmelzung *innerhalb* der memory (Abschnitte konsolidieren) — sie ist Teil der Verdichtung, keine eigene Stufe.

### temp.md — das flüchtige Arbeitsblatt

Bewusst **kein** Wissensartefakt: hält Arbeitszustand. Zwischenablage und Dialog-Arbeitsfläche; entsteht ad hoc auf jeder Ebene. **Flüchtig per Definition** (Inhalt wird überschrieben; ein `temp.md` pro Ebene; Instanzen unabhängig) · **Ernte vor Überschreiben** (Erhaltenswertes vorher in ein dauerhaftes Artefakt routen — Session-Arbeit im Dialog, kein Skill-Automatismus; der `/remember`-Durchlauf *fragt* nach ungeernteter temp, §8) · **nie Pointer-Ziel** · **Skills lassen es liegen** · Frontmatter wie üblich. Namens-Konvergenz: genau `temp.md`; Abweichler beim Anfassen umbenennen.

### infra.md — die Betriebs-Referenz

Das benannte Muster für Wissen über die *Bedienung von Systemen und Werkzeugen* — APIs, CLIs, Zugriffswege, Eigenheiten, Fehlerbilder samt Fix. Abgrenzung: `context.md` hält den Zustand der **Arbeit**, `infra.md` die Bedienung der **Werkzeuge**, `memory.md` die Urteile.

- **Zwei Sorten, zwei Orte:** *Dienst-Infra* (Systeme einer Domäne — deren ERP-API, die Firmen-Postfächer, ein eigener Bot) → `infra.md` neben der `context.md` der Ebene, zu der das System gehört. *Maschinen-Infra* (der Rechner: Tools, MCP, Pfade) → **`~/.claude/reference/infra.md`**, außerhalb des Workspace — gerätegebunden, wandert nicht über Sync.
- **Kreislauf** (die drei Momente): *Lesen* — on-demand, nie vorladen; der Session-Start **registriert** beim Pfad-Scan, welche `infra.md` existieren (CLAUDE.md-Regel), und Anker-Steckbriefe halten den Pointer. *Schreiben* — die **`/remember`-Weiche**: Betriebslektionen einer Session gehören in die infra der betroffenen Ebene, nicht in log/memory (§8). *Hygiene* — Einträge mit Verfallsbedingung („Flag kann raus, sobald …") werden im `/remember`-Durchlauf der Ebene bei Gelegenheit geprüft.
- **Secrets-Grenze (hart):** Schlüssel, Tokens, Passwörter, Schlüsseldateien (.pfx/.pem/.p12) stehen **nie** in `infra.md` oder anderen Workspace-Dateien. Einziger Ort ist ein Vault (Passwort-Manager mit CLI, sonst OS-Schlüsselbund); `infra.md` hält den *Verweis* (Vault, Eintragsname, Zugriffs-Kommando). Zur Laufzeit aus dem Vault in die Session-Umgebung, nie in Dateien materialisiert; Klartext-`.env` ist kein Fallback. **Mess-Moment:** Der `cleanup`-Inventar-Schritt scannt auf Secret-Muster (§8) — Empirie 08/2026: Ohne Scan lagen Klartext-Passwörter monatelang im Sync-Pfad.

### Werkzeuge — ausführbare Artefakte einer Ebene

Wiederverwendbarer eigener Code, der zu einer Domäne gehört (Bau-Skripte, ein CI-Layout-Gerüst, Automatisierungen), lebt **neben der `context.md` der Ebene, zu der er gehört** — bei Nutzung über mehrere Projekte auf dem untersten gemeinsamen Vorfahren (Anker). Die `infra.md` derselben Ebene hält je Werkzeug die Bedien-Zeile (wann benutzen, was man leicht falsch macht); der Code selbst dokumentiert das Wie. Werkzeuge sind keine Kontext-Artefakte — sie werden nie vorgeladen und folgen keiner Frontmatter-Pflicht. Ein Designsystem entsteht aus dem zweiten Anwendungsfall, nicht aus dem ersten: erst beim zweiten Bau wird getrennt, was Ebene (Marke) und was Anlass (das eine Deck) war.

### Binär- und Bildmaterial

Rohmaterial (Bilder, Videos, große Assets) ist kein Kontext-Artefakt: Es liegt in einem benannten Ordner der Ebene (z.B. `bildmaterial/`, `assets/`) — **außerhalb von Repos**, wenn nur Ergebnisse geteilt werden (Empirie: Kurs-Bildmaterial); ein `README.md` im Ordner trägt die Bau-Regeln. Schlüssel- und Zertifikatsdateien sind nie „Assets" — Secrets-Grenze.

## 4. Artefakt-Verteilung über die Rollen

- **`context.md`: auf jedem Rollen-Träger.** Additiv, beliebig tief.
- **`memory.md` + `log.md`: per Default am Anker.** Tiefere Knoten sind eine **Earn-it-Ausnahme** (§6); log und memory splitten **gemeinsam**. Die Wurzel führt keine memory — Urteile kristallisieren an den Ankern.
- Begründung aus den Daten: per-Projekt-Logs sind typischerweise winzig und früh eingefroren, während Anker-memory und -log monatelang weiterleben. Die Schwerkraft zieht nach oben — das ist gesund.
- **Tokenökonomie:** Der Spar-Einwand gegen große Anker-memory ist real, aber (a) das „irrelevante" Drittel ist oft der strategische Rahmen des aktuellen Projekts, und (b) die Lösung bei echtem Überlauf ist die Verschmelzung innerhalb (§3) oder — wenn die Split-Kriterien tragen — der Split, nicht Datei-Fragmentierung auf Verdacht.

## 5. Task-Lebenslauf

`context.md` ist der Kern des PMO. Ein Task wandert:

```
Task entsteht   → context.md §5 „Offene Punkte"   [offene Verpflichtung = Zustand]
Task läuft      → bleibt dort, wird fortgeschrieben
Task erledigt   → (a) raus aus context.md          [Scaffolding gelöscht — kein Verlust]
                  (b) Faktum → log.md               [als Satz im Body des Session-Eintrags,
                                                     kein eigener Log-Eintrag]
                  (c) falls Urteil dranhängt → memory.md
                  (d) falls betrieblich referenzpflichtig → Register-Artefakt
```

**Triage** (in `remember` kodiert, vier Ausgänge): prunen (trivial, ersatzlos) · Faktum-Zeile im Session-Eintrag (6-Monats-Frage) · memory-Urteil · **Register**. Das Register-Genre: Vorgänge, die abgeschlossen, aber referenzpflichtig bleiben (erledigt ≠ irrelevant — z.B. Bewerbungen unter Dedup-Pflicht), leben in einem eigenen Register-Artefakt neben der `context.md` (darf append-only wachsen); die `context.md` hält den Pointer, `memory.md` die Muster. Referenzfall: `consulting_portale/bewerbungen.md`.

## 6. Split-Entscheidung (memory/log Anker → Knoten)

**Wer/wann:** `remember`, im Bottom-up-Schritt — fester Prüfpunkt, nicht ad hoc.

**Drei Bedingungen, alle nötig:** (1) **Volumen** — Anker-memory so groß, dass Mitlesen bei fremden Prompts spürbar Ballast ist, UND der Projekt-Strang ist ein großer, klar abgrenzbarer Block (Faustregel-Auslöser: memory > ~15 KB und ein Projekt ≳ ⅓) · (2) **Severabilität** — die Projekt-Einträge zeigen nach innen, nicht seitwärts · (3) **Eigenleben** — genug eigene Historie für separate Verdichtung; keine präventiven Splits.

**Protokoll:** Default = nicht splitten (Aufschieben billig, Zusammenführen teuer). Klarer Fall → autonom ausführen + melden; Grenzfall → vorlegen. **Marker-Pflicht:** Der Vollzug hinterlässt im Kopf des Anker-Logs die Zeile „*Split-Hinweis: `{Projekt}` führt eigenes Log + Memory (Split TT.MM.) — Projekt-Einträge gehören dorthin.*" — ein Split, der nur Dateisystem-Zustand ist, wird von späteren Sessions überschrieben. **Lese-Folge:** Die Session-Start-Regel lädt bei Split-Projekten deren memory zusätzlich zur Anker-memory (CLAUDE.md).

## 7. Schicht-Modell: CLAUDE.md / Wurzel / Referenzen

- **`CLAUDE.md` = imperative Regeln + Pointer.** Schlank: Rolle, Don'ts, Trust, Sprach-/Dokument-Konventionen, **Session-Start-Lade-Regel** (context-Pfad + memory inkl. Split-Zweig + Log-Kopf + infra-Registrierung). Verweist für die Artefakt-Spezifikation hierher. CLAUDE.md ist das einzige Artefakt mit Ladegarantie — was zuverlässig passieren soll, muss dort oder in einem Skill-Flow verankert sein.
- **Deskriptive Maschinen-Infra** → `~/.claude/reference/infra.md`, on-demand.
- **Persona** → Wurzel-`context.md`; CLAUDE.md pointet darauf.
- **Maschinen-Status (explizite Annahme):** Das System läuft derzeit auf **einer** Maschine; der frühere Workspace-Sync ist stillgelegt (08/2026). Multi-Maschinen-Betrieb braucht vor Reaktivierung eine Merge-Strategie — das Neueste-oben-Log kollidiert bei parallelem Schreiben strukturell an der obersten Zeile.

## 8. Skill-Konsequenzen

### contextify

„Lege/pflege die `context.md` *dieser Ebene* gegen das Backbone-Schema an." Kennt Backbone (§2) und Rollen (§1), instanziiert die rollen-spezifischen Steckbrief-Felder, erzwingt Personen-Block und Disziplin-Schnitt. Inputs beliebig; das Ziel ist immer eine schema-konforme `context.md`. **Kein contextify-log mehr:** Der Nachweis eines Laufs ist der Bestätigungsblock (chars vorher → nachher bei Auslagerungen); die HTML-Kommentar-Telemetrie am Dateiende ist abgeschafft — Bestands-Blöcke werden beim Anfassen entfernt. Das eingebettete Backbone im Skill trägt den Spiegel-Vermerk (Master: §2).

### remember

Zwei Verdichtungsstufen + Bottom-up-Abgleich. Die Prüf- und Schreibpunkte:

1. **Split-Probe** (mechanisch, Pflicht) und **Mess-Pflicht** (Log-Stand erheben, Pflichtzeile im Bestätigungsblock).
2. **Task-Closure** mit Vier-Wege-Triage (§5, inkl. Register-Zweig).
3. **infra-Weiche:** Betriebslektionen der Session (Werkzeug-Eigenheit, Fehlerbild + Fix, System-Bedienung) werden in die `infra.md` der betroffenen Ebene geschrieben, nicht in den Log-Eintrag; der Log-Eintrag referenziert allenfalls. Ausweis im Bestätigungsblock. Bei der Gelegenheit: infra-Verfallsbedingungen der Ebene prüfen.
4. **Verschmelzungsregel memory** (§3): in bestehende thematische Abschnitte integrieren; chronologische Anhänge sind das Signal zur Konsolidierung.
5. **Split-Kriterien** (§6) als fester Prüfpunkt; **Größen-Heuristik context.md** (§2) messen und ggf. vorlegen. Der Bottom-up-Schritt korrigiert offensichtlich überholte `context.md`-Stellen **direkt** (Backbone wahrend) — ein eigener contextify-Lauf ist nur für Neuanlage oder strukturellen Umbau nötig. Reihenfolge am Session-Ende damit: `/remember` genügt im Regelfall; `/contextify` auf Ansage.
6. **temp-Erinnerung:** Führt die Ebene eine `temp.md` mit ungeerntetem Inhalt (updated jünger als der letzte Log-Eintrag), wird im Bestätigungsblock daran erinnert — geerntet wird im Dialog.
7. **Pointer-Pflege:** Entsteht im Durchlauf ein neues Artefakt (infra, Register, Fach-Artefakt), wird der Pointer der Ebene (Steckbrief bzw. Lage) im selben Durchlauf gesetzt.

### cleanup

Kompositions-Schicht über `contextify`: gewachsene Teilbäume in die Logik überführen. Sicherheits-Prinzipien: Plan-dann-Bestätigen · `mv` statt `rm`, löschen nur autorisiert · Varianten flaggen · Pointer-Reconcile ist Pflicht · Repos atomar, Symlink-Ziele nie verschieben. Zwei Pflicht-Erweiterungen: **Secrets-Scan im Inventar-Schritt** (Muster: Passwort-/Token-Zuweisungen, `BEGIN … PRIVATE KEY`, Dateitypen .pfx/.pem/.p12/.key; Befund — auch „keiner" — ist Pflichtbestandteil des Dispositions-Plans) und die Kategorie **Assets/Binärmaterial** (benannter Ordner + README statt lose Dateien; Secret-Träger nie als Asset klassifizieren).

### Geteilte Regeln

- Skills lesen Backbone und Kriterien aus dieser Spec + `template_context.md`; gespiegelte Passagen tragen den Spiegel-Vermerk.
- Frontmatter-Konvention (`created`/`updated`) setzen alle Skills um; `log.md` ist ausgenommen (per Eintrag datiert).
