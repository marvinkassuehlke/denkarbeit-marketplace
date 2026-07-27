---
created: 2026-06-15
updated: 2026-07-27
---

# Kontext-System — Design

Kanonische Spezifikation des Kontext-Systems im `/workspace`: die drei Standardartefakte (`context.md` / `memory.md` / `log.md`), ihre Hierarchie, ihr Zusammenspiel und die Konsequenzen für `CLAUDE.md` und die Skills `contextify` / `cleanup` / `remember`. Dieses Dokument ist die Single Source of Truth — `CLAUDE.md` und die Skills verweisen hierher, statt die Konvention zu duplizieren.

## Zweck & Geltung

Das System war gewachsen, nicht spezifiziert: 19 `context.md` mit fünf verschiedenen Personen-Erfassungs-Mustern, leckender Rollentrennung (Chronologien und Urteile in `context.md`), `contextify` ohne Schema-Bewusstsein. Es funktioniert gut — dies ist Optimierung, kein Rebuild. Ziel: Wildwuchs auf oberster Ebene beseitigen, ohne die Tiefe zu verengen.

Leitprinzip: **`context.md` ist und bleibt die *eine* Wissensdatei pro Ordner.** Kein Zerlegen in `stakeholder.md` / `objective.md` o.ä. Die Grundstruktur ist bewusst weit abstrahiert, damit „alles reinpasst"; dass Einzelfälle das Konstrukt strapazieren, ist akzeptiert.

## 1. Architektur — rekursive context-Hierarchie

Ein Artefakttyp, vier Ebenen, semantisch klar:

```
L1  /workspace/context.md          ICH            — wer bin ich, was tue ich
L2  {Stakeholder}/context.md       DIE ANDEREN    — wer ist das Gegenüber (oder eigener Bereich)
L3  {Projekt}/context.md           WAS MIT IHNEN  — was tue ich mit/für sie
L4  {Arbeitspaket}/context.md      TEILSTÜCK      — rein hierarchischer Split, wenn L3 zu groß
```

- **L2 ist nicht zwingend „extern".** Auch eigene Domänen sind L2: `strgaGmbH/` (das Unternehmen als ein Bereich), `dev/` (Coding), `trading/` (eigenes Nebenprojekt). „Stakeholder" meint Gegenüber *oder* eigener Bereich.
- **L1 bleibt schlank.** Identität + Domänen-Orientierung (welche *Typen* von Domänen es gibt) + „Motivation" auf höchster Ebene. **Keine gepflegte Domänen-Aufzählung** — die lebt in den L2-Ordnern; L1 verweist dorthin und dupliziert nicht (insb. nicht `strgaGmbH/`).
- **L4 ist optional und rein strukturell** — Überlauf-Ventil, wenn ein L3-Projekt zu groß wird. Das Schema *erlaubt* L4, *erzwingt* es nie.
- **State komponiert abwärts:** Jede Ebene erbt den Kontext der Elternebene und verengt ihn. Jede `context.md` ist eigenständig lesbar.

### Begriffs- und Zählkonvention

- **Primärvokabular sind die semantischen Namen** (Stakeholder → Projekt → Arbeitspaket). Die **L-Nummern sind internes Spec-Kürzel** — sie laufen in Spec und Skills mit, werden aber nicht in die Didaktik exportiert (Kurs-Videos, Slides, Sprechskripte kennen nur die Namen).
- **Zählweise:** Gezählt werden die Ebenen *unter* dem Workspace-Root — drei. Der Root (Ich, L1) ist Träger, keine Stufe. „Dreistufig" (Kurs-Sprech) und „vier Ebenen L1–L4" (Spec-Sprech) bezeichnen dieselbe Struktur.
- **„Stakeholder"** bleibt der kanonische Name der ersten Unterebene; die Ebene ist benannt nach ihrem häufigsten Bewohner. Definition im Kurs-Sprech (Oberbegriff „Domäne"): *für wen oder was du arbeitest — Arbeitgeber, Kunde oder eigener Bereich.*
- **Ebene ≠ Pfadtiefe.** Die Tiefe ist die Default-Kodierung der Ebene, keine Semantik: Zwischenordner (Programm-Ebenen, Sammlungen, Verwaltungs-Ordner wie `inaktiv/`) verschieben die Tiefe, nicht die Ebene — `/inaktiv/Weleda` ist genauso L2 wie `/Weleda`. **Erkennungskette** (für alle Skills verbindlich): (1) semantische Ansprache des Users · (2) Artefakt-Bestand — Ebenen-Träger ist der Ordner, der die Standardartefakte trägt oder tragen sollte (`context.md`, ggf. `memory.md`/`log.md`); reine Sammelordner sind **ebenen-transparent** und bekommen nie eigene Kontext-Artefakte · (3) Pfadtiefe als Default. Im Zweifel nachfragen.

## 2. context.md — Backbone

**Fixer Kopf, freie Tiefe.** Die Wildwuchs-Ursache war nie die inhaltliche Vielfalt unter den Überschriften — sie war der uneinheitliche *oberste* Aufbau. Lösung: Top-Level wird fix, die Substruktur bleibt frei.

| # | Sektion | Frage | Inhalt | Stabilität |
|---|---|---|---|---|
| — | **Kopf / Steckbrief** | Was/wer ist das? | 1 Framing-Satz unter der H1 + `Stand: JJJJ-MM-TT`. Dann ebenen-spezifische Felder (s.u.) | hoch |
| 1 | **Motivation** | Warum existiert das für mich, was ist mein Einsatz? | Motivation, Einsatz, Verortung — als *Kontext/Orientierung*, nicht als Verhaltenssteuerung, kein Psychogramm | hoch |
| 2 | **Lage** | Wie steht es? | Der Wissenskörper. **Frei untergliederbar** — hier wohnen Org-Bäume, Marktbilder, Tech-Stacks, Beziehungsstände | lebendig |
| 3 | **Richtung** | Wohin? | Zielbild, woran gearbeitet wird — als *Zustand*, nicht als To-do-Liste | lebendig |
| 4 | **Offene Punkte** | Was ist unerledigt? | Offene Tasks **und** offene Fragen / Wissenslücken — das PMO-Hauptbuch der ungelösten Dinge | lebendig |

- **Quasi-Pflicht:** Kopf + „Motivation". **Nach Bedarf:** Lage (der Wissenskörper). **Optional:** Richtung / Offene Punkte.
- **Stabilitäts-Gefälle statt statisch-vs-lebendig:** `context.md` ist nicht einheitlich das eine oder andere. Kopf + Motivation sind quasi-statisch (ändern sich selten), Lage/Richtung/Offene-Punkte sind lebendig. Es braucht kein zweites Artefakt für „das Statische".

### Steckbrief-Felder je Ebene

- **L1 (Ich):** Eckdaten, Rollen, Angebot/Tätigkeit, Arbeitsweise-Grundzüge, Domänen-Orientierung (Typen, kein gepflegtes Verzeichnis).
- **L2 (Stakeholder):** Org-Steckbrief (Firma/Entität, Eckdaten) **+ Personen-Block** (s.u.).
- **L3 (Projekt):** Auftrag, Scope, Stand, Beteiligte (Verweis auf L2-Personen statt Duplikat).
- **L4 (Arbeitspaket):** konkretes Ergebnis, Abgrenzung zum Projekt.

### Personen-Erfassung (löst den Fünf-Muster-Wildwuchs)

- **Organisation = Ordner (L2).** Personen = Einträge im Steckbrief der Ebene, auf der die Beziehung lebt.
- Org-weite Person (z.B. Andrea bei Weleda) → **L2**. Projekt-spezifischer Kontakt → **L3** (mit Rückverweis auf L2, falls dort ebenfalls geführt).
- **Ein Feld-Muster für alle:** `Name · Rolle/Funktion · Kontakt (optional) · Hinweis (optional)`. Der Hinweis trägt nur echte sachliche Information (Zuständigkeit, Entscheidungsmacht, Kommunikationsweg) — **kein** Psychogramm.

### Disziplin-Schnitt

**Raus aus `context.md`** (gehört woanders hin oder nirgends):
- Chronologien / Zeitachsen → `log.md` / `memory.md`
- Urteile, „Erkenntnisse", strategische Einordnungen → `memory.md`
- Erledigt-Marker, abgehakte/abgeschlossene Tasks → `log.md` (als Faktum) bzw. ersatzlos
- To-do-Mechanik mit Status-Tracking → nur „offene Punkte" bleiben, kein Status-Pingpong

**Rein in `context.md`** (Reifegrad-Marker der guten Dateien):
- Sicherheitsgrade `[BELEGT]` / `[ANNAHME]` / `[WEB]` / `[PITCH]`
- Widerspruch-Marker `<!-- Widerspruch: Quelle A sagt X, Quelle B sagt Y -->`

### Größen-Heuristik (weiche Regel)

Eine `context.md` jenseits **~20 KB** (Faustregel, justierbar; Empirie 07/2026: gesundes Feld endet bei ~20 KB, der entgleiste Referenzfall lag bei 40 KB) ist ein Prüfsignal, kein Urteil — meist ist dann etwas als „Kontext" festgehalten, was ein anderes Artefakt sein will. Diagnose-Fächer: eingesickerte Chronologien/Urteile (→ Disziplin-Schnitt, memory/log) · Register-Fall (→ operatives Register, §5) · eigenständiger Fach-Block (→ Fach-Artefakt + Pointer, Muster `bewerbungsformate.md`) · legitim groß (→ belassen). **Hart messen, weich handeln:** Gemessen wird mechanisch im `remember`-Bottom-up-Schritt; bei Überschreitung wird der Befund vorgelegt, nie autonom umgebaut. Geduldete Genre-Fremdkörper (Dossiers wie `johan`, `babymama`, `tsenso`) und Register-Artefakte sind ausgenommen.

**Auslagerungs-Protokoll (Vollzug des Falls „Fach-Block"):** Bestätigt der User den Befund, ist der Vollzug `contextify`-Arbeit (context.md-Pflege) — im selben Durchlauf, egal aus welchem Skill der Befund kam; ein Befund ohne definierten Vollzugsweg verpufft im Chat. Regeln:
1. **Verlustfreier Schnitt** — der Block wandert vollständig in ein Fach-Artefakt neben der `context.md` derselben Ebene (Frontmatter, H1, ein Framing-Satz); beim Umzug wird nicht mitverdichtet — Verdichtung wäre ein eigener, sichtbarer Schritt.
2. **Pointer-Rückstand** — in der `context.md` bleibt ein Pointer mit einem Satz Substanz (was dort liegt, ggf. Stand); sie bleibt eigenständig lesbar. Referenzmuster: `bewerbungsformate.md`, `duesseldry/datenschutz.md`.
3. **Ausweis** — der Umzug steht im contextify-log (chars vorher → nachher belegen die Entlastung) und im Bestätigungsblock.

## 3. Die drei Artefakte — Rollen & Asymmetrie

Die heutige Formel „context = Zustände / memory = Übergänge" leckt, weil sie zwei Hälften *derselben* Daten suggeriert. Korrekt sind **drei verschiedene Fragen**:

- **`context.md` = „was gilt jetzt".** Das integrierte, aktuelle Bild. **Vernichtet** Historie bewusst (alte Rolle wird überschrieben). Lebendig, kontinuierlich an die Wahrheit abgeglichen.
- **`memory.md` = „was sich geändert hat und warum".** Der Urteils- und Begründungspfad, den man aus dem aktuellen Bild *nicht* rekonstruieren kann. **Bewahrt** Historie mit Wertung.
- **`log.md` = Ereignisstrom.** Roh, session-nah, append-only. Speist `memory.md`.

**Die Asymmetrie ist der Kern:**
- **State komponiert abwärts** (context): Fakten verengen sich von L1 → L4.
- **Judgment integriert aufwärts** (memory): Urteile entstehen aus dem Zusammenführen *über* Projekte hinweg und kristallisieren eine Ebene höher. Sie bauen *nicht* additiv aufeinander auf wie context — sie verschmelzen.

Empirie (Weleda): Die „Strategische Positionierung" der Stakeholder-`memory.md` verwebt KI-Effizienz + AI-levers + Platform-vs-Coding + Rollen-Diskussion + Budgetkürzung zu *einem* Narrativ; das Effectiveness-Mandat starb an einer Group-Digital-Budgetkürzung (Stakeholder-Fakt killt Projekt); das L3a/L3b-Framework „trägt über Weleda hinaus". Das Urteil lebt an den Nähten *zwischen* Projekten — Projekt-Silos zerschnitten genau dieses Bindegewebe.

### temp.md — das flüchtige Arbeitsblatt

Viertes Standard-Muster neben den drei Wissensartefakten — bewusst **kein** Wissensartefakt: `temp.md` hält Arbeitszustand, kein Wissen. Zweck: Zwischenablage und Dialog-Arbeitsfläche — Copy-Paste-Eingang, gemeinsames Arbeiten und Kommentieren an einem Markdown im Session-Dialog. Entsteht ad hoc dort, wo die Session arbeitet (jede Ebene; per Anweisung auch anderswo); der eine etablierte Name beugt Scratch-Wildwuchs vor.

- **Flüchtig per Definition:** Der Inhalt wird beim nächsten Bedarf überschrieben — ein `temp.md` pro Ebene genügt. Instanzen auf verschiedenen Ebenen sind unabhängig (keine Varianten, keine Dups).
- **Ernte vor Überschreiben:** Inhalt, der bleiben soll, wird vorher in ein dauerhaftes Artefakt geroutet (context/memory/log/Fach-Artefakt). Die Ernte ist Session-Arbeit im Dialog, kein Skill-Automatismus.
- **Nie stabiles Pointer-Ziel:** `context.md`/`memory.md` verweisen nie auf `temp.md`; in `log.md` allenfalls als historischer Vermerk mit Flüchtigkeits-Kennzeichnung.
- **Skills lassen es liegen:** `cleanup` archiviert, verschiebt oder benennt `temp.md` nie um; offensichtlich Erntenswertes wird als Content-Punkt vorgelegt. Frontmatter wie üblich (`updated` zeigt die Frische des Inhalts).
- **Namens-Konvergenz:** genau `temp.md`; Abweichler (`input_temp.md` u.ä.) werden opportunistisch beim Anfassen umbenannt — kein Massen-Retrofit.

## 4. Artefakt-Verteilung über die Ebenen

- **`context.md`: auf jeder Ebene (L1–L4).** Additiv.
- **`memory.md` + `log.md`: per Default auf Stakeholder-Ebene (L2).** Projekt-Ebene (L3) ist eine **Earn-it-Ausnahme** (Kriterien s. §6). log und memory splitten **gemeinsam** — dieselbe Integrations-Grenze.
- L1 trägt allenfalls eine schlanke memory; L4 nie.

Begründung aus den Daten: Weledas per-Projekt-`log.md` (KI-Effizienz, CIP, trAIlblazer) sind winzig (538 B – 1,5 KB) und eingefroren am 21.03., während Stakeholder-`memory.md` (14 KB) und `log.md` (11 KB) bis 10.05. leben. Die Schwerkraft zieht nach oben — und das ist gesund.

**Tokenökonomie:** Der Spar-Einwand gegen große Stakeholder-memory ist real, aber (a) das „irrelevante" Drittel ist oft der strategische Rahmen für das aktuelle Projekt, und (b) die Lösung bei echtem Überlauf ist **Sektionierung + Verdichtung innerhalb** der Stakeholder-memory (nach Thema), nicht Datei-Fragmentierung nach Projekt.

## 5. Task-Lebenslauf

context.md ist der Kern des PMO. Ein Task wandert über die drei Artefakte, statt als totes Gewicht zu verrotten:

```
Task entsteht        → context.md §4 "Offene Punkte"      [offene Verpflichtung = aktueller Zustand]
Task läuft           → bleibt in context.md, wird fortgeschrieben  [lebendiges Scaffolding]
Task erledigt        → (a) raus aus context.md            [Scaffolding gelöscht — kein Verlust]
                       (b) log.md: "TT.MM. X getan/übergeben"   [das Faktum bleibt auffindbar]
                       (c) falls Urteil dranhängt: memory.md
```

Drei Dinge, die nicht zu verschmelzen sind: das **Arbeits-Scaffolding** (lebt in context solange offen, wird beim Schließen gelöscht — sein einziger Zweck war die Erledigung), das **Faktum** (→ log, die 6-Monats-Erinnerung), das **Urteil** (→ memory).

**Auslöser + Triage** (in `remember` kodiert): Task-Schluss feuert „prune aus context + (falls relevant) log-Zeile". Triviale Erledigungen werden nur geprunt, ersatzlos. Triage-Schwelle = „will ich das in 6 Monaten noch erinnern?".

**Ausnahme — operatives Register:** Vorgänge, die abgeschlossen, aber betrieblich referenzpflichtig bleiben (erledigt ≠ irrelevant — z.B. Bewerbungen unter Portal-Doppel-Erkennung), folgen dem Task-Lebenslauf nicht. Sie leben in einem eigenen **Register-Artefakt** neben der `context.md` (Dossier-Genre, darf append-only wachsen); die `context.md` hält nur den Pointer, `memory.md` die Muster. Referenzfall: `consulting_portale/bewerbungen.md` — ohne dieses Genre wuchs der Tracker in der context.md auf 40 KB, weil abgeschlossene Vorgänge dedup-relevant blieben und der Prune-Mechanismus nie greifen durfte.

## 6. Split-Entscheidung (memory/log L2 → L3)

**Wer/wann:** `remember`, im Bottom-up-Schritt — fester Prüfpunkt, nicht ad hoc.

**Drei Bedingungen, alle drei nötig:**
1. **Volumen** (mechanisch): Stakeholder-memory ist so groß, dass Mitlesen bei projektfremden Prompts spürbar Ballast ist, UND der Projekt-Strang ist ein großer, klar abgrenzbarer Block.
2. **Severabilität** (Urteil): memory-Einträge des Projekts zeigen nach *innen*, nicht seitwärts. Wenn ich beim Schreiben ständig Schwester-Projekte / Stakeholder-weite Fakten referenzieren muss → verschränkt → **kein** Split.
3. **Eigenleben:** genug eigene Historie, dass eine separate Verdichtung Substanz hätte — keine *präventiven* Splits frischer Projekte.

**Protokoll:**
- **Default = nicht splitten.** Asymmetrische Kosten: Aufschieben billig, Zusammenführen teuer, verfrühter Split zerschneidet Bindegewebe.
- **Klarer Fall** (alle drei deutlich, keine Quer-Referenzen) → autonom ausführen + im `/remember`-Bestätigungsblock melden.
- **Grenzfall** (groß aber verschränkt / unabhängig aber dünn) → vorlegen.

## 7. Schicht-Modell: CLAUDE.md / L1 / Referenz

Aus dem ursprünglichen Aufschlag (imperativ vs. deskriptiv). Prinzip entschieden, Detail-Umsetzung nachgelagert:

- **`CLAUDE.md` = imperative Regeln + Pointer.** Schlank. Behält: Rolle, Don'ts, Trust, Sprach-/Dokument-Konventionen, **Session-Start-Lade-Regel**. Verweist für Artefakt-Spezifikation hierher.
- **Deskriptive Infra raus** (MCP-Tabelle, rclone, IPv6, Google-Workspace-Specifics) → eigene Referenz-Datei, on-demand gezogen statt in jeder Session geladen.
- **Persona raus** → `/workspace/context.md` (L1). `CLAUDE.md` *pointet* darauf, hält sie nicht.
- **Session-Start-Regel erweitern:** L1-`context.md` (Persona) wird relevant, sobald im Workspace gearbeitet wird; L2/L3-Laden wie gehabt nach CWD.

## 8. Skill-Konsequenzen

### contextify — Umbau

Von „transformiere beliebige Inputs in *ein* ideales Dokument" zu **„lege/pflege die `context.md` *dieser Ebene* gegen das Backbone-Schema an"**.

- **Behält** seine Prinzipien als das *Wie* der Befüllung: semantische Kompression, Implizites explizieren, Widerspruch-Marker, Quellen-Vollständigkeitsprüfung, kein Informationsverlust.
- **Neu:** kennt das Backbone (§2), erkennt die Ebene aus dem Pfad (L1–L4), instanziiert die ebenen-spezifischen Steckbrief-Felder, erzwingt den Personen-Block und den Disziplin-Schnitt (schiebt Chronologien/Urteile raus statt sie aufzunehmen).
- Inputs bleiben beliebig (Calls, PDFs, Notizen) — das *Ziel* ist immer eine schema-konforme `context.md`.

### remember — Erweiterung

Kern (Drei-Stufen-Verdichtung, Bottom-up-Abgleich) bleibt; vier Ergänzungen:

1. **Task-Closure modellieren:** beim Erledigen prune aus `context.md` §4 + (falls relevant) log-Zeile + ggf. memory-Urteil.
2. **Split-Kriterien kodieren** (§6) als fester Prüfpunkt im Bottom-up-Schritt.
3. **Neues `context.md`-Schema kennen**, damit der Bottom-up-Abgleich (Korrektur veralteter context-Stellen) das Backbone respektiert.
4. **Default memory/log auf L2** (statt der bisherigen impliziten Projekt-Ebene); Projekt-Ebene nur per Earn-it.

### cleanup — Komposition

`cleanup` ist die **Kompositions-Schicht über `contextify`**: es überführt einen ganzen *gewachsenen* Teilbaum in die Logik — Projekte erkennen, L3-Contexts anlegen, lose Dateien einsortieren, die übergeordnete `context.md` zum Index reconcilen, Quer-Liegendes flaggen.

- **Scope aus dem Pfad** (wie `contextify`): `/cleanup {pfad}` → L1 (ganzer Workspace, Stakeholder-für-Stakeholder) / L2 (Stakeholder) / L3 (ein Projekt). Kein Pfad → nachfragen.
- **Geschwister über geteilter Spec, keine harte Kopplung:** `cleanup` und `contextify` lesen das Backbone beide aus diesem Dokument + `template_context.md`. `cleanup` bettet das Backbone **nicht** neu ein (sonst Re-Duplikation). Safety-Check = *Spec erreichbar*, nicht „`contextify` installiert". Fehlt die Spec → Abbruch des Kontext-Schritts (Tidy/Flags laufen trotzdem).
- **Sicherheits-Prinzipien:** Plan-dann-Bestätigen (nie blind); `mv` statt `rm`, löschen nur autorisiert, Zweifel → `archive/`; Varianten flaggen statt entscheiden; **Pointer-Reconcile ist Pflicht** (Moves erzeugen eine Pointer-Kaskade in context/memory/log).
- **Repos & Symlinks:** `.git`-Verzeichnisse und Symlinks werden im Inventar mechanisch erhoben (Mess-Pflicht), der Befund — auch „keine" — im Dispositions-Plan ausgewiesen. Ein Repo ist eine **atomare Einheit**: als Ganzes klassifizieren (i.d.R. Bestandteil eines Projekts), nie intern umsortieren — kein `mv` hinein oder heraus, kein `archive/` im Repo, keine Löschungen; die innere Struktur gehört dem Repo (eigene Konventionen, `git mv`, Commits). Kontext-Artefakte leben per Default außerhalb des Repos auf der Ebenen-Wurzel; Ausnahme ist das Master-Muster (Repo hält den Master, der Workspace Symlinks — Vorbild Kurs-Repo). Symlink-Ziele nie verschieben (bricht den Link) → flaggen. Ändert der Kontext-Schritt doch eine Datei innerhalb eines Repos (z.B. Pointer-Reconcile), wird der entstandene dirty Zustand im Bestätigungsblock ausgewiesen; das Committen bleibt beim User.
- **Arbeitsteilung:** `cleanup` fasst memory/log nur *strukturell* an (Split-Ebene, verkümmerte Logs nach Autorisierung); die inhaltliche Verdichtung bleibt `remember`.

## 9. Stand der Umsetzung & Restpunkte

**Umgesetzt (Stand 2026-06-16):**
- `CLAUDE.md` geschlankt (imperativ + Pointer); Infra → `~/.claude/reference/infra.md`; Session-Start liest L1; imperativer Block entdoppelt.
- L1-`context.md` (Persona) angelegt.
- `contextify` schema-aware umgebaut · `remember` erweitert (Task-Closure, Split-Kriterien, L2-Default) · `cleanup` neu.
- Volumen-Schwelle für den Split: ~15 KB Faustregel (in `remember` fixiert, justierbar).
- **Referenz-Implementierung `/Weleda`** vollständig durchgezogen: L2 + 6 L3-Contexts, Dateien aufgeräumt, Pointer reconciled.

**Restpunkte:**
- Migration der übrigen Bestands-`context.md`: kein Massen-Retrofit — via `cleanup`/`contextify`, sobald ein Bereich ohnehin angefasst wird. Genre-Fremdkörper (`johan`, `babymama`, `tsenso`) bleiben geduldete Dossier-Ausnahmen.
- Weleda-Flags (User-Entscheidung): ROOT-VARIANTEN in `Weleda/archive/` prüfen · `ai-levers/`-Zuschnitt bestätigen.
- Schulungs-Delivery: `cleanup` + `contextify` + Spec + `template_context.md` als Bündel ausliefern.
