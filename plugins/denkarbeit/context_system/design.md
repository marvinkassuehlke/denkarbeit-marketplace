---
created: 2026-06-15
updated: 2026-08-25
---

# Kontext-System — Design

Kanonische Spezifikation des Kontext-Systems im `/workspace`: die Standardartefakte (`context.md` / `memory.md` / `log.md` / `temp.md` / `infra.md`), ihre Hierarchie, ihr Zusammenspiel und die Konsequenzen für `CLAUDE.md` und die Skills `contextify` / `cleanup` / `remember`. **SSOT-Modell:** Diese Spec trägt das Modell — Genres, Orte, Kreisläufe, Kriterien. Die Skills tragen die Ausführung und dürfen Kernregeln bewusst spiegeln, damit sie eigenständig lauffähig sind; gespiegelte Passagen tragen einen Vermerk (`Master: design.md §X`), und Änderungen laufen zuerst hier, dann im Skill nach. Formatkonventionen (Log-/Memory-Form) sind hier normativ definiert (§3) — der Skill führt sie aus.

## Zweck & Geltung

Leitprinzip: **`context.md` ist und bleibt die *eine* Wissensdatei pro Ordner.** Kein Zerlegen in `stakeholder.md` / `objective.md` o.ä. Die Grundstruktur ist bewusst weit abstrahiert, damit „alles reinpasst"; dass Einzelfälle das Konstrukt strapazieren, ist akzeptiert.

**Enforcement-Doktrin** (gilt für jede Regel dieser Spec): *Eine Regel, deren Bedingung der Normallauf nicht misst, existiert nicht.* Mess-Pflicht (mechanisch erheben, nicht schätzen) + Ausweis-Pflicht (Ergebnis als Pflichtzeile im Output) + harte Gates nur, wo groß immer falsch ist. Jedes Artefakt-Genre braucht einen definierten **Lese-Moment**, **Schreib-Moment** und **Hygiene-Moment** — ein Genre ohne alle drei verwaist (Empirie: `infra.md` 08/2026). Maschinelle Härtung (Hooks) darf diese Momente **härten, nie tragen**: Das System muss rein prompt-basiert funktionieren, weil nicht jede Harness Hooks kennt (Cowork).

## 1. Architektur — Pfad-Mechanik

Das System kennt **keine Ebenen-Semantik**. Es kennt zwei Dinge: den **Workspace-Root** (in der `CLAUDE.md` verankert, dort liegt die oberste `context.md`) und den **Session-Ordner** (das aktuelle Arbeitsverzeichnis). Alles andere ergibt sich aus dem Pfad dazwischen.

**Ein Ordner mit `context.md` ist ein Kontext-Ordner.** Das ist die einzige Definition, die es braucht. Die Datei macht aus einem Verzeichnis einen Ort mit Identität; ein Ordner ohne sie ist bloße Ablage und wird von der Mechanik ignoriert. Damit bestimmt der User die Struktur durch einen normalen Arbeitsakt: Er legt eine `context.md` an, wo ein eigener Gegenstand entsteht.

**Lesen (vom Session-Ordner aufwärts bis zum Root):**

| Artefakt | Regel | Grund |
|---|---|---|
| `context.md` | **alle** auf dem Pfad | Zustand komponiert sich: Organisation + Vorhaben + Arbeitspaket ergeben zusammen das Bild |
| `memory.md` | **alle** auf dem Pfad | Urteile ergänzen sich; was oben liegt, gilt auch unten |
| `log.md` | **nur das nächste** | Zwei Ereignisströme mischen sich nicht |
| `infra.md` | keins vorladen, Existenz registrieren | on-demand-Genre (§3) |

**Schreiben:** In die nächste vorhandene Datei aufwärts. **Neu angelegt** wird nur in einem Kontext-Ordner, nie in bloßer Ablage; ohne vorhandene Datei wird gefragt (kein Auto-Create).

**Warum das Gedächtnis nach oben zieht.** Die Ablage-Regel folgt der Sichtbarkeit: Weil aufwärts gelesen wird, sieht ein höher liegendes Artefakt mehr Sessions. Daraus die Leitregel — **so hoch wie es gilt, so tief wie möglich** (dieselbe Logik wie Variablen-Scope: der engste Bereich, der alle Verwendungen umfasst). Der Aufwärts-Default ist zugleich der Failsafe für neue Vorhaben: Ein frisch angelegter Ordner erbt automatisch, was darüber liegt, statt bei null zu beginnen. Und die Fehlerkosten sind asymmetrisch — zu hoch abgelegt kostet Token, zu tief abgelegt kostet Wissen, und der zweite Fehler bleibt unsichtbar.

**Das Denkmodell bleibt, als Empfehlung statt als Regel.** Ich → für wen ich arbeite → was ich mit ihnen tue ist die Gliederung, die sich bewährt hat und die der Kurs lehrt. Die Mechanik kennt sie nicht: Sie liest stumpf aufwärts und funktioniert bei zwei wie bei sechs Ordnern auf dem Pfad. Wer anders gliedert, bekommt kein kaputtes System, sondern eine andere Verteilung.

### Konsequenzen, die keine eigene Regel mehr brauchen

- **Sammel- und Zwischenordner** (`inaktiv/`, `Archiv/`, Programm-Ebenen) haben keine `context.md` und werden übersprungen. Davon zu unterscheiden ist das lokale **`archive/`** als cleanup-Ablage innerhalb eines Kontext-Ordners.
- **Beliebige Tiefe** funktioniert ohne Sonderfall. Wer tief gliedert, lädt mehr `context.md`; das ist sichtbar und selbstverschuldet.
- **Kontext-Ordner = Repo:** Enthält der Ordner `.git`, leben die Artefakte trotzdem in seiner Wurzel; ob sie committet werden, entscheidet der User (ggf. `.gitignore`). Sonderfall bleibt das Master-Muster (Repo hält den Master, der Workspace Symlinks).
- **Strukturwechsel ist Nutzerarbeit.** Wird aus einem Ordner, der Gegenüber und Vorhaben in einem war, ein zweites Vorhaben herausgelöst, teilt der User die `context.md`; `memory.md` und `log.md` bleiben liegen, weil ihr Inhalt weiter gilt. Skills erkennen den Anlass und legen ihn vor (§8), vollziehen ihn nie selbst.

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

### Steckbrief-Felder — nach Inhalt, nicht nach Position

Welche Felder ein Steckbrief trägt, ergibt sich daraus, **was der Ordner beschreibt**, nicht daraus, wie tief er liegt:

- **Mich selbst** (der Workspace-Root): Eckdaten, Rollen, Angebot/Tätigkeit, Arbeitsweise-Grundzüge, Orientierung über die Domänen-Typen (kein gepflegtes Verzeichnis, keine Duplikate aus den Ordnern darunter).
- **Ein Gegenüber oder einen eigenen Bereich** (Kunde, Arbeitgeber, Firma, privates Feld): Org-Steckbrief (Entität, Eckdaten) **+ Personen-Block** (s.u.) + Pointer auf `infra.md`, falls vorhanden.
- **Ein Vorhaben** (Projekt, Arbeitspaket, Teilstück): Auftrag, Scope, Stand, Beteiligte (Verweis auf die Personen darüber statt Duplikat); bei Teilstücken zusätzlich die Abgrenzung zum übergeordneten Vorhaben.

Im Zweifel fragen. Ein Ordner kann auch beides sein (ein Gegenüber mit genau einem Vorhaben) — dann trägt eine `context.md` beide Feldgruppen, bis der User sie teilt.

### Personen-Erfassung

- **Organisation = eigener Ordner.** Personen sind Einträge im Steckbrief dort, wo die Beziehung lebt: org-weite Person beim Gegenüber, projekt-spezifischer Kontakt beim Vorhaben (mit Rückverweis).
- **Ein Feld-Muster:** `Name · Rolle/Funktion · Kontakt (optional) · Hinweis (optional)`. Der Hinweis trägt nur sachliche Information (Zuständigkeit, Entscheidungsmacht, Kommunikationsweg).
- **Grenze zum Personenwissen:** Der Steckbrief hält kein Psychogramm. **Arbeitsrelevantes Beziehungswissen** (wie jemand entscheidet, was ihn überzeugt, welche Themen heikel sind) ist legitim — sein Ort ist die zuständige `memory.md` für dieses Gegenüber, als Urteil aus der Zusammenarbeit. Nicht legitim, in keinem Artefakt: Charakter-Dossiers und Bewertungen der Person jenseits der Arbeitsbeziehung.

### Disziplin-Schnitt

**Raus aus `context.md`:** Chronologien/Zeitachsen und Urteile/Erkenntnisse → `memory.md` (Fakten-Splitter gelangen über `/remember` ins Log — contextify schreibt nie ins Log, §3) · Erledigt-Marker → ersatzlos bzw. als Faktum über die Task-Closure (§5) · To-do-Mechanik mit Status-Tracking → nur offene Punkte bleiben · Betriebs-/Bedienungswissen → `infra.md` (§3).

**Rein in `context.md`:** Sicherheitsgrade **`[BELEGT]` / `[ANNAHME]`** (genau diese zwei; freie Begründungszusätze wie `[ANNAHME — Quelle X]` sind erlaubt) · Widerspruch-Marker `<!-- Widerspruch: Quelle A sagt X, Quelle B sagt Y -->` · offene Fragen als `<!-- Offen: … -->`.

### Größen-Heuristik (weiche Regel)

Eine `context.md` jenseits **~20 KB** ist ein Prüfsignal, kein Urteil. Diagnose-Fächer: eingesickerte Chronologien/Urteile (→ Disziplin-Schnitt) · Register-Fall (→ §5) · eigenständiger Fach-Block (→ Fach-Artefakt + Pointer) · legitim groß (→ belassen). **Hart messen, weich handeln:** Gemessen wird mechanisch im `remember`-Bottom-up-Schritt; bei Überschreitung wird der Befund vorgelegt, nie autonom umgebaut. Geduldete Genre-Fremdkörper (private Dossiers) und Register-Artefakte sind ausgenommen.

**Auslagerungs-Protokoll (Fach-Block):** Bestätigt der User den Befund, ist der Vollzug `contextify`-Arbeit im selben Durchlauf: (1) **Verlustfreier Schnitt** in ein Fach-Artefakt neben der `context.md` desselben Ordners (Frontmatter, H1, Framing-Satz; keine Mitverdichtung) · (2) **Pointer-Rückstand** mit einem Satz Substanz · (3) **Ausweis** im Bestätigungsblock (chars vorher → nachher). Referenzmuster: `bewerbungsformate.md`, `duesseldry/datenschutz.md`.

## 3. Die Artefakte — Aufgaben, Formate, Kreisläufe

Drei Wissensartefakte, drei verschiedene Fragen:

- **`context.md` = „was gilt jetzt".** Das integrierte, aktuelle Bild. **Vernichtet** Historie bewusst.
- **`memory.md` = „was sich geändert hat und warum".** Der Urteils- und Begründungspfad. **Bewahrt** Historie mit Wertung.
- **`log.md` = Ereignisstrom.** Session-nah, speist `memory.md`.

**Die Asymmetrie ist der Kern:** State komponiert abwärts (context verengt sich mit jedem Schritt nach unten); **Judgment integriert aufwärts** (memory: Urteile entstehen aus dem Zusammenführen *über* Vorhaben hinweg und kristallisieren oberhalb davon; sie verschmelzen, statt additiv zu stapeln).

### Formate (normativ)

- **`log.md`:** Header `# Log — {Name}`, dann ein YAML-Block mit Einträgen `- date: / event: / body: / artifacts:` — **neueste oben**. Keine Datei-Frontmatter (per Eintrag datiert). **Einziger Schreiber ist `/remember`**: Sessions editieren das Log nie direkt; die Verdichtung überführt ältere Einträge in die memory und entfernt sie aus dem Log. („Append-only" meint genau das — kein manuelles Editieren; es verbietet nicht die Verdichtung.)
- **`memory.md`:** Prosa-Markdown mit Frontmatter, **thematische Abschnitte** — die Struktur folgt dem Inhalt, nicht dem Kalender. **Verschmelzungsregel:** Die Verdichtung integriert in bestehende Abschnitte, statt datierte Abschnitte anzuhängen — eine memory, die wie ein zweites Log aussieht (chronologische Anhänge), ist das Signal, dass die zweite Verdichtungsstufe fällig ist. Leserichtung folgt der Kausalität (Grundlegendes zuerst); memory wird ganz gelesen, nicht vom Ende.
- **Zwei Verdichtungsstufen:** Session → Log (Eintrag) · Log → Memory (Verdichtung, entfernt Quell-Einträge). Die dritte Bewegung ist die Verschmelzung *innerhalb* der memory (Abschnitte konsolidieren) — sie ist Teil der Verdichtung, keine eigene Stufe.

### temp.md — das flüchtige Arbeitsblatt

Bewusst **kein** Wissensartefakt: hält Arbeitszustand. Zwischenablage und Dialog-Arbeitsfläche; entsteht ad hoc in jedem Ordner. **Flüchtig per Definition** (Inhalt wird überschrieben; ein `temp.md` pro Ordner; Instanzen unabhängig) · **Ernte vor Überschreiben** (Erhaltenswertes vorher in ein dauerhaftes Artefakt routen — Session-Arbeit im Dialog, kein Skill-Automatismus; der `/remember`-Durchlauf *fragt* nach ungeernteter temp, §8) · **nie Pointer-Ziel** · **Skills lassen es liegen** · Frontmatter wie üblich. Namens-Konvergenz: genau `temp.md`; Abweichler beim Anfassen umbenennen.

### infra.md — die Betriebs-Referenz

Das benannte Muster für Wissen über die *Bedienung von Systemen und Werkzeugen* — APIs, CLIs, Zugriffswege, Eigenheiten, Fehlerbilder samt Fix. Abgrenzung: `context.md` hält den Zustand der **Arbeit**, `infra.md` die Bedienung der **Werkzeuge**, `memory.md` die Urteile.

- **Zwei Sorten, zwei Orte:** *Dienst-Infra* (Systeme einer Domäne — deren ERP-API, die Firmen-Postfächer, ein eigener Bot) → `infra.md` neben der `context.md` des Ordners, zu dem das System gehört. *Maschinen-Infra* (der Rechner: Tools, MCP, Pfade) → **`~/.claude/reference/infra.md`**, außerhalb des Workspace — gerätegebunden, wandert nicht über Sync.
- **`infra.md` ist der Default-Name, kein Zwang:** Trägt ein Ordner sein Betriebswissen bereits in benannten Fach-Artefakten (z.B. ein Übergabe-Handbuch, API-Notes im Projekt-Repo), erfüllen diese das Genre — dann dorthin routen statt eine redundante `infra.md` daneben zu stellen. Entscheidend ist der benannte Ort, nicht der Dateiname.
- **Kreislauf** (die drei Momente): *Lesen* — on-demand, nie vorladen; der Session-Start **registriert** beim Pfad-Scan, welche `infra.md` existieren (CLAUDE.md-Regel), und die Steckbriefe halten den Pointer. *Schreiben* — die **`/remember`-Weiche**: Betriebslektionen gehören in die infra des Ordners, für den sie gelten, nicht in log/memory (§8). *Hygiene* — Einträge mit Verfallsbedingung („Flag kann raus, sobald …") werden im `/remember`-Durchlauf bei Gelegenheit geprüft.
- **Secrets-Grenze (hart):** Schlüssel, Tokens, Passwörter, Schlüsseldateien (.pfx/.pem/.p12) stehen **nie** in `infra.md` oder anderen Workspace-Dateien. Einziger Ort ist ein Vault (Passwort-Manager mit CLI, sonst OS-Schlüsselbund); `infra.md` hält den *Verweis* (Vault, Eintragsname, Zugriffs-Kommando). Zur Laufzeit aus dem Vault in die Session-Umgebung, nie in Dateien materialisiert; Klartext-`.env` ist kein Fallback. **Mess-Moment:** Der `cleanup`-Inventar-Schritt scannt auf Secret-Muster (§8) — Empirie 08/2026: Ohne Scan lagen Klartext-Passwörter monatelang im Sync-Pfad.

### Werkzeuge — ausführbare Artefakte eines Ordners

Wiederverwendbarer eigener Code, der zu einer Domäne gehört (Bau-Skripte, ein CI-Layout-Gerüst, Automatisierungen), lebt **neben der `context.md` des Ordners, für den er gilt** — bei Nutzung über mehrere Vorhaben im untersten gemeinsamen Ordner (dieselbe Scope-Regel wie beim Gedächtnis, §1). Die `infra.md` daneben hält je Werkzeug die Bedien-Zeile (wann benutzen, was man leicht falsch macht); der Code selbst dokumentiert das Wie. Werkzeuge sind keine Kontext-Artefakte — sie werden nie vorgeladen und folgen keiner Frontmatter-Pflicht. Ein Designsystem entsteht aus dem zweiten Anwendungsfall: erst beim zweiten Bau wird sichtbar, was der Marke gehört und was dem einen Anlass.

### language.md — die Sprach-Referenz (optional)

Die Sprachregeln selbst leben in der `CLAUDE.md` und gelten immer, weil sie kurz sind und jede Session sie trägt. Eine `language.md` im Workspace-Root ergänzt sie um **Referenzmaterial** für längere Texte an Dritte (ab etwa einer halben Seite): kuratierte Absätze aus eigenen, selbst verfassten Texten plus eine Fallenliste beobachteter Drifts.

- **Optional, kein Default-Artefakt.** Ohne geeignetes Material verschlechtert eine Sprach-Datei den Ton, statt ihn zu schärfen — dann bleibt es bei den CLAUDE.md-Regeln. `setup` bietet sie an und nennt die Eignungskriterien (§8).
- **Beispiel statt Vorschrift.** Positive Stilregeln („nutze Gedankenstriche", „variiere den Rhythmus") werden als Checkliste abgearbeitet und übererfüllt: Was im Original zweimal trägt, steht dann in jedem Absatz und fällt als Manier auf. Wirksam sind Referenztexte (sie prägen Satzbau und Register) und **negative** Listen (sie sind prüfbar). Empirie 08/2026: Die Vorgänger-Datei schrieb Gedankenstrich-Einschübe, „Nicht X, sondern Y" und Kurzsatz-Kaskaden ausdrücklich vor — genau die drei Muster, die als maschinell auffielen.
- **Kein Fremdmaterial im Auslieferungspaket.** Ein mitgelieferter Referenzkorpus prägt alle Nutzer auf eine fremde Stimme und widerspricht dem Kurs-Kernsatz, dass Kontext kuratiert wird. Jeder Workspace baut seine eigene.

### Binär- und Bildmaterial

Rohmaterial (Bilder, Videos, große Assets) ist kein Kontext-Artefakt: Es liegt in einem benannten Unterordner (z.B. `bildmaterial/`, `assets/`) — **außerhalb von Repos**, wenn nur Ergebnisse geteilt werden (Empirie: Kurs-Bildmaterial); ein `README.md` im Ordner trägt die Bau-Regeln. Schlüssel- und Zertifikatsdateien sind nie „Assets" — Secrets-Grenze.

## 4. Wo das Gedächtnis liegt

`context.md` entsteht dort, wo ein eigener Gegenstand entsteht. Für `memory.md` und `log.md` gilt die Scope-Regel aus §1: **so hoch wie es gilt, so tief wie möglich.**

- **Empfehlung, keine Regel:** Gedächtnis lohnt sich dort, wo Urteile über mehrere Vorhaben zusammenlaufen — in der bewährten Gliederung also beim Gegenüber, nicht im einzelnen Projekt. Ein Vorhaben bekommt eigenes Gedächtnis, wenn es genug eigene Historie hat, dass eine separate Verdichtung Substanz hätte, und wenn seine Einträge nach innen zeigen statt ständig auf Nachbarn zu verweisen. Beides ist Erfahrung, kein Prüfpunkt: Wer es anders will, legt die Datei an, wo er sie haben will.
- **`log.md` und `memory.md` gehören zusammen.** Wo eins entsteht, entsteht das andere; ein Log ohne Verdichtungsziel läuft voll, eine Memory ohne Zufluss verhungert.
- **Warum es sich lohnt, nicht vorschnell zu vertiefen:** Erfahrungsgemäß sind Projekt-Logs winzig und früh eingefroren, während Gedächtnis eine Stufe höher jahrelang weiterlebt — Urteile entstehen an den Nähten *zwischen* Vorhaben. Und ein neu angelegter Ordner erbt automatisch, was über ihm liegt (§1, Failsafe).
- **Tokenökonomie:** Der Spar-Einwand gegen große Gedächtnisdateien ist real, aber das „irrelevante" Drittel ist oft der Rahmen für das aktuelle Vorhaben. Die Antwort bei echtem Überlauf ist Verschmelzung innerhalb der memory (§3), nicht Fragmentierung auf Verdacht.

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

## 6. Ein eigenes Gedächtnis anlegen

Kein Vorgang mit Kriterien, sondern ein Handgriff: Wer für ein Vorhaben eigenes Gedächtnis will, legt dort `log.md` und `memory.md` an. Die Mechanik findet sie ab dem nächsten Lauf, weil immer die nächste Datei aufwärts gilt (§1). Ein Marker im übergeordneten Log ist nicht mehr nötig — niemand kann an einer tiefer liegenden Datei vorbeischreiben.

**Wann es sich lohnt** (Erfahrung, siehe §4): genug eigene Historie für eine sinnvolle Verdichtung, Einträge, die nach innen zeigen, und ein Umfang, bei dem Mitlesen in fremden Sessions spürbar Ballast wird. **Wann nicht:** vorsorglich bei frischen Vorhaben. Aufschieben ist billig, Zusammenführen ist teuer, und ein verfrühter Schnitt zerschneidet das Bindegewebe zwischen den Urteilen.

`/remember` legt so etwas nie von sich aus an. Fehlt auf dem ganzen Pfad ein Log, wird gefragt (§8).

## 7. Schicht-Modell: CLAUDE.md / Root / Referenzen

- **`CLAUDE.md` = imperative Regeln + Pointer.** Schlank: Rolle, Don'ts, Trust, Sprach-/Dokument-Konventionen, **Session-Start-Lade-Regel** (Pfad-Scan, alle context/memory aufwärts, nächstes Log, infra-Registrierung — §1). Verweist für die Artefakt-Spezifikation hierher. CLAUDE.md ist das einzige Artefakt mit Ladegarantie — was zuverlässig passieren soll, muss dort oder in einem Skill-Flow verankert sein.
- **Deskriptive Maschinen-Infra** → `~/.claude/reference/infra.md`, on-demand.
- **Persona** → `context.md` im Root; CLAUDE.md pointet darauf und verankert den Root-Pfad. Der Root ist die einzige Konfiguration, die eine Installation braucht.

## 8. Skill-Konsequenzen

### contextify

„Lege/pflege die `context.md` *dieses Ordners* gegen das Backbone-Schema an." Kennt Backbone (§2) und Pfad-Mechanik (§1), wählt die Steckbrief-Felder nach dem Inhalt des Ordners, erzwingt Personen-Block und Disziplin-Schnitt. Inputs beliebig; das Ziel ist immer eine schema-konforme `context.md`. **Kein contextify-log mehr:** Der Nachweis eines Laufs ist der Bestätigungsblock (chars vorher → nachher bei Auslagerungen); die HTML-Kommentar-Telemetrie am Dateiende ist abgeschafft — Bestands-Blöcke werden beim Anfassen entfernt. Das eingebettete Backbone im Skill trägt den Spiegel-Vermerk (Master: §2).

### remember

Zwei Verdichtungsstufen + Bottom-up-Abgleich. Die Prüf- und Schreibpunkte:

1. **Ort-Probe** (mechanisch, Pflicht): Pfad vom Session-Ordner aufwärts scannen, das nächste `log.md` bestimmen, Befund im Bestätigungsblock ausweisen. Dazu die **Mess-Pflicht** (Log-Stand erheben, Pflichtzeile im Bestätigungsblock).
2. **Task-Closure** mit Vier-Wege-Triage (§5, inkl. Register-Zweig). Vor dem Schreiben eines Urteils die Scope-Frage (§1): Gilt es auch außerhalb dieses Ordners? Dann gehört es weiter nach oben.
3. **infra-Weiche:** Betriebslektionen der Session (Werkzeug-Eigenheit, Fehlerbild + Fix, System-Bedienung) werden in die `infra.md` des Ordners geschrieben, für den sie gelten (Scope-Regel §1), nicht in den Log-Eintrag; der Log-Eintrag referenziert allenfalls. Ausweis im Bestätigungsblock. Bei der Gelegenheit: infra-Verfallsbedingungen der berührten Dateien prüfen.
4. **Verschmelzungsregel memory** (§3): in bestehende thematische Abschnitte integrieren; chronologische Anhänge sind das Signal zur Konsolidierung.
5. **Größen-Heuristik context.md** (§2) messen und ggf. vorlegen. **Struktur-Beobachtung:** Beschreibt eine `context.md` erkennbar mehrere getrennte Vorhaben, oder sammelt ein Unterordner ohne eigene `context.md` Substanz, wird das als Befund vorgelegt (§1, Strukturwechsel) — nie selbst vollzogen. Der Bottom-up-Schritt korrigiert offensichtlich überholte `context.md`-Stellen **direkt** (Backbone wahrend) — ein eigener contextify-Lauf ist nur für Neuanlage oder strukturellen Umbau nötig. Reihenfolge am Session-Ende damit: `/remember` genügt im Regelfall; `/contextify` auf Ansage.
6. **temp-Erinnerung:** Führt der Ordner eine `temp.md` mit ungeerntetem Inhalt (updated jünger als der letzte Log-Eintrag), wird im Bestätigungsblock daran erinnert — geerntet wird im Dialog.
7. **Pointer-Pflege:** Entsteht im Durchlauf ein neues Artefakt (infra, Register, Fach-Artefakt), wird der Pointer in der `context.md` desselben Ordners (Steckbrief bzw. Lage) im selben Durchlauf gesetzt.

### cleanup

Kompositions-Schicht über `contextify`: gewachsene Teilbäume in die Logik überführen. Sicherheits-Prinzipien: Plan-dann-Bestätigen · `mv` statt `rm`, löschen nur autorisiert · Varianten flaggen · Pointer-Reconcile ist Pflicht · Repos atomar, Symlink-Ziele nie verschieben. Zwei Pflicht-Erweiterungen: **Secrets-Scan im Inventar-Schritt** (Muster: Passwort-/Token-Zuweisungen, `BEGIN … PRIVATE KEY`, Dateitypen .pfx/.pem/.p12/.key; Befund — auch „keiner" — ist Pflichtbestandteil des Dispositions-Plans) und die Kategorie **Assets/Binärmaterial** (benannter Ordner + README statt lose Dateien; Secret-Träger nie als Asset klassifizieren).

### Geteilte Regeln

- Skills lesen Backbone und Kriterien aus dieser Spec + `template_context.md`; gespiegelte Passagen tragen den Spiegel-Vermerk.
- Frontmatter-Konvention (`created`/`updated`) setzen alle Skills um; `log.md` ist ausgenommen (per Eintrag datiert).
