---
name: cleanup
description: Use when bringing a grown, messy workspace area into the context-system logic — sweeping a folder tree to create or refresh the standard context artifacts, sort loose files into the right folders, and reconcile pointers. Triggers on /denkarbeit:cleanup.
---

## Zweck

`cleanup` ist die **Kompositions-Schicht über `contextify`**. Wo `contextify` *eine* `context.md` pflegt, überführt `cleanup` einen ganzen *gewachsenen* Teilbaum in die Kontext-System-Logik: eigenständige Vorhaben erkennen, ihnen eine `context.md` geben, lose Dateien einsortieren, die übergeordnete `context.md` zum Index reconcilen, Quer-Liegendes flaggen.

`cleanup` erfindet das Backbone **nicht neu**: Es ist dasselbe wie in `contextify` — fünf Top-Level-Sektionen (**Steckbrief · Motivation · Lage · Richtung · Offene Punkte**), fixer Kopf, freie Tiefe unter „Lage". Die beiden Skills sind Geschwister, keine harte Skill-zu-Skill-Kopplung: Wo `contextify` eine einzelne Datei schreibt, disponiert `cleanup` den Baum und ruft die Schreibarbeit auf.

## Scope bestimmen (Pflicht-Input)

Der **Pfad bestimmt den Scope**:

- `/denkarbeit:cleanup` ohne Pfad → der ganze Workspace ab Root. Inventar und Klassifikation laufen einmal über das Ganze, der **Dispositions-Plan wird je Top-Level-Ordner vorgelegt und freigegeben** — ein Plan über alles auf einmal ist zu groß zum Prüfen.
- `/denkarbeit:cleanup {pfad}` → dieser Ordner und alles darunter.

Kein Pfad angegeben → nachfragen (oder CWD vorschlagen und bestätigen lassen). Kein stilles Raten des Scopes.

Die Tiefe sagt nichts (Pfad-Mechanik): Ein **Kontext-Ordner** ist jeder Ordner mit `context.md`; Sammel- und Zwischenordner (`inaktiv/`, `Archiv/`, Programm-Ebenen) haben keine und bekommen auch keine. Welche Ordner unter dem Scope Kontext-Ordner werden sollen, ist Teil des Dispositions-Plans.

## Safety-Gates (hart)

- **Plan-dann-Bestätigen.** Nie blind ausführen. Erst die **Dispositions-Tabelle** (jede Datei / jeder Ordner → geplante Aktion) vorlegen, dann auf Freigabe ausführen. Besonders bei breitem Scope (ganzer Workspace).
- **Verschieben, nicht löschen.** Default ist `mv` (reversibel). **Löschen nur auf explizite Autorisierung** des Users (z.B. verkümmerte Logs). Im Zweifel → `archive/`.
- **Varianten nicht auto-entscheiden.** Gleicher Dateiname + abweichender Inhalt/Größe = **Variante**, nicht Dup → nach `archive/` mit `.VARIANT`-Marker + flaggen. Nie die kanonische Version überschreiben.
- **Schreiben/Verschieben macht der Hauptlauf.** Read-only Subagents dürfen lesen und Kontext-Entwürfe zurückgeben; die tatsächlichen Datei-Operationen führt der Controller aus.
- **Repo = atomare Einheit.** Ein Ordner mit `.git` wird als Ganzes klassifiziert, nie intern umsortiert: kein `mv` hinein oder heraus, kein `archive/` im Repo, keine Löschungen — die innere Struktur gehört dem Repo (eigene Konventionen, `git mv`, Commits). Kontext-Artefakte leben in der Wurzel des Kontext-Ordners. Ändert der Kontext-Schritt doch eine Datei im Repo (z.B. Pointer-Reconcile): dirty Zustand im Bestätigungsblock ausweisen, committen bleibt beim User. **Ein Secret-Fund im Repo ist die eine Ausnahme, die das Gate nicht deckt:** Der Scan läuft auch dort, der Fund wird geflaggt — behoben wird er vom User im Repo selbst, weil ein Klartext-Wert außerdem in der Git-Historie steht und ein Dateiedit ihn nicht entfernt.
- **Symlink-Schutz.** Symlink-Ziele nie verschieben (bricht den Link) → flaggen. Einen Symlink selbst nur verschieben, wenn der Plan das Nachziehen des Links enthält.

## Workflow

1. **Anker lesen.** Die `context.md` im Workspace-Root + die des Sweep-Ordners, soweit vorhanden. Das ist der Rahmen, der nach unten injiziert wird, damit neue Contexts nicht duplizieren. **Hier wird nichts geschrieben** — fehlende `context.md` werden im Dispositions-Plan vorgeschlagen und erst in Schritt 5 angelegt. Fehlt auch der Rahmen oben, arbeitet der Sweep ohne ihn und vermerkt das.

2. **Inventarisieren.** Vollständiges Listing des Subtrees — Ordner **und** lose Dateien, alle Typen. **Mess-Pflicht Repos/Symlinks:** `.git`-Verzeichnisse und Symlinks mechanisch erheben (`find {pfad} -name .git`, `find {pfad} -type l`) — der Befund (auch „keine") ist Pflichtbestandteil des Dispositions-Plans; eine Prüfung ohne erhobenen Befund hat nicht stattgefunden. **Mess-Pflicht Secrets:** den Subtree mechanisch auf Secret-Muster scannen — Passwort-/Token-Zuweisungen (`grep -riE 'passwor[dt]|passwd|secret|api[_-]?key|token' {pfad}` — **ohne `--include`-Filter, Code-Dateien eingeschlossen**: hardcodete Keys liegen typischerweise gerade dort. Treffer einzeln sichten; Vault-Verweise wie `op://` sind konform, Klartext-Werte nicht), `BEGIN.*PRIVATE KEY`, sowie Schlüssel-Dateitypen (`find {pfad} -name '*.pfx' -o -name '*.pem' -o -name '*.p12' -o -name '*.key'`). Der Befund — auch „keiner" — ist Pflichtbestandteil des Dispositions-Plans; Fundstücke werden als **Secrets-Verstoß** geflaggt (Ziel: Vault + Verweis, Datei löschen nur mit Freigabe).

3. **Klassifizieren** (der harte Kern — bei Ambiguität fragen). Je Ordner/Datei genau eine Kategorie:
   - **Eigener Gegenstand** — Vorhaben oder Bereich mit Substanz → bekommt `context.md` und wird damit Kontext-Ordner.
   - **Repo** — Ordner mit `.git`: atomare Einheit, als Ganzes zuordnen (oder selbst der Gegenstand); Innenleben nicht anfassen (Safety-Gate).
   - **temp.md** — flüchtiges Arbeitsblatt: bleibt liegen — nie archivieren, verschieben, umbenennen oder als Dup/Variante werten (Instanzen in verschiedenen Ordnern sind unabhängig). Offensichtlich Erntenswertes als Content-Punkt vorlegen (Schritt 7), nicht selbst umrouten.
   - **Input/Support** (`transkripte/`, `customer_inputs/`, …) → behalten, **kein** Kontext.
   - **Assets/Binärmaterial** (Bilder, Videos, Fonts, große Medien) → in einen benannten Unterordner (`bildmaterial/`, `assets/`) mit `README.md` für die Bau-Regeln; lose Binärdateien in einer Ordner-Wurzel sind ein Flag. Schlüssel-/Zertifikatsdateien sind **nie** Assets → Secrets-Verstoß (s. Inventar-Schritt).
   - **Archiv / Superseded** — ersetzte Entwürfe, alte Stände → `archive/` **im selben Kontext-Ordner**, nie ein zentrales Archiv am Root (der Bezug zum Gegenstand geht sonst verloren). Ein bestehender Alt-Archivordner unter anderem Namen (`archiv_alt/`, `alt/`) wird als Umbenennungs-Vorschlag in den Plan aufgenommen, nicht still umbenannt.
   - **Dup** — gleicher Inhalt, existiert woanders → `archive/`.
   - **Variante** — gleicher Name, abweichender Inhalt → `archive/` + `.VARIANT` + flaggen. **Vorrang vor „Superseded":** Trifft beides zu, gilt Variante — der Marker kostet nichts, sein Fehlen kann eine Fassung unbemerkt verdrängen.
   - **Symlink** — eigene Kategorie, nie ein Dup oder eine Variante: Ziel und Link getrennt beurteilen (Safety-Gate). Zeigt der Link ins Leere → flaggen.
   - **Leer** (Ordner ohne Inhalt, Datei ohne Substanz) → **belassen und flaggen**; Entfernen nur auf Autorisierung.
   - **Quer-liegend / fehlplatziert** — gehört erkennbar woandershin (eingebettetes Fremd-System, Material aus einer anderen Domäne) → **flaggen, nicht zwingen**.

4. **Dispositions-Plan vorlegen** — Tabelle aller geplanten Aktionen → Freigabe abwarten.

5. **Ausführen.** Fehlende Ordner anlegen; Dateien verschieben; (autorisierte) Löschungen; pro neuem Kontext-Ordner **und** für jede bestehende `context.md`, der Backbone-Sektionen fehlen, die **`contextify`-Logik** (`context.md` gegen das Backbone, mit dem Kontext von oben injiziert, Maxime „referenziere nach oben, dupliziere nicht").

6. **Nach oben reconcilen.** In der übergeordneten `context.md` einen **Pointer-Index** auf die neuen Kontext-Ordner setzen. **Alle Pointer verschobener Dateien in `context.md`/`memory.md`/`log.md` nachziehen** (Move erzeugt eine Pointer-Kaskade — der fehleranfälligste Schritt). Gedächtnis bleibt liegen, wo es liegt; wo `log.md`/`memory.md` entstehen sollen, entscheidet der User.

7. **Content-Punkte einsammeln.** Fakten, die der Sweep ans Licht bringt (z.B. „Ansatz X wurde gekippt"), in das passende `context.md`/`memory.md` routen — nicht verlieren. Existiert keine `memory.md` (und `cleanup` legt keine an): die Fakten im Bestätigungsblock als Liste ausweisen und auf `remember` verweisen, statt sie in eine `context.md` zu drücken, in die sie nicht gehören.

8. **Bestätigung.** Kompakt: was angelegt / verschoben / archiviert / geflaggt wurde; **offene Flags explizit auflisten** (Varianten zum Prüfen, Quer-Liegendes, deep-doc-interne Verweise, die nicht nachgezogen wurden).

## Verhältnis zu den anderen Skills

- **`contextify`** = das Atom (eine `context.md` gegen das Backbone). `cleanup` ruft dessen Logik je Kontext-Ordner auf. Ist `contextify` nicht verfügbar, baut `cleanup` die Contexts gegen das oben genannte Backbone selbst; Tidy, Klassifikation und Flags laufen davon unabhängig.
- **`remember`** = pflegt `memory.md`/`log.md` **inhaltlich** (Übergänge, Urteile, Verdichtung, Task-Closure). `cleanup` fasst sie nur **strukturell** an (verkümmerte Logs nach Autorisierung) — die inhaltliche Verdichtung bleibt `remember`.
