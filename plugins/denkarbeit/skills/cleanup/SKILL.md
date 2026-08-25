---
name: cleanup
description: Use when bringing a grown, messy workspace area into the context-system logic — sweeping a folder tree to create or refresh the standard context artifacts, sort loose files into the right folders, and reconcile pointers. Triggers on /cleanup.
---

## Zweck

`cleanup` ist die **Kompositions-Schicht über `contextify`**. Wo `contextify` *eine* `context.md` pflegt, überführt `cleanup` einen ganzen *gewachsenen* Teilbaum in die Kontext-System-Logik: eigenständige Vorhaben erkennen, ihnen eine `context.md` geben, lose Dateien einsortieren, die übergeordnete `context.md` zum Index reconcilen, Quer-Liegendes flaggen.

`cleanup` erfindet das Backbone **nicht neu** — es liest die Spezifikation aus `design.md` und `template_context.md` (Single Source of Truth, geteilt mit `contextify`). **Spec-Auflösung:** der in der globalen CLAUDE.md verankerte Pfad (Sektion Referenzen bzw. Workspace-Konvention); Fallback: vom Skill-Verzeichnis aufwärts nach `context_system/design.md` suchen (liegt in der Plugin- bzw. Repo-Wurzel oberhalb von `skills/`). Die beiden Skills sind Geschwister über derselben Spec, keine harte Skill-zu-Skill-Kopplung.

## Scope bestimmen (Pflicht-Input)

Der **Pfad bestimmt die Ebene** (wie bei `contextify`):

- `/cleanup` ohne Pfad → der ganze Workspace ab Root. Geht **Ordner für Ordner**, mit Scope-Bestätigung vorab.
- `/cleanup {pfad}` → dieser Ordner und alles darunter.

Kein Pfad angegeben → nachfragen (oder CWD vorschlagen und bestätigen lassen). Kein stilles Raten des Scopes.

Die Tiefe sagt nichts (Pfad-Mechanik, `design.md` §1): Ein **Kontext-Ordner** ist jeder Ordner mit `context.md`; Sammel- und Zwischenordner (`inaktiv/`, `Archiv/`, Programm-Ebenen) haben keine und bekommen auch keine. Welche Ordner unter dem Scope Kontext-Ordner werden sollen, ist Teil des Dispositions-Plans.

## Safety-Gates (hart)

- **Spec erreichbar?** `context_system/design.md` + `template_context.md` müssen lesbar sein — das Backbone, an dem `cleanup` hängt. Fehlen sie → abbrechen mit klarem Hinweis (nicht raten, nicht selbst ein Backbone erfinden).
- **Plan-dann-Bestätigen.** Nie blind ausführen. Erst die **Dispositions-Tabelle** (jede Datei / jeder Ordner → geplante Aktion) vorlegen, dann auf Freigabe ausführen. Besonders bei breitem Scope (ganzer Workspace).
- **Verschieben, nicht löschen.** Default ist `mv` (reversibel). **Löschen nur auf explizite Autorisierung** des Users (z.B. verkümmerte Logs). Im Zweifel → `archive/`.
- **Varianten nicht auto-entscheiden.** Gleicher Dateiname + abweichender Inhalt/Größe = **Variante**, nicht Dup → nach `archive/` mit `.VARIANT`-Marker + flaggen. Nie die kanonische Version überschreiben.
- **Schreiben/Verschieben macht der Hauptlauf.** Read-only Subagents dürfen lesen und Kontext-Entwürfe zurückgeben; die tatsächlichen Datei-Operationen führt der Controller aus.
- **Repo = atomare Einheit.** Ein Ordner mit `.git` wird als Ganzes klassifiziert, nie intern umsortiert: kein `mv` hinein oder heraus, kein `archive/` im Repo, keine Löschungen — die innere Struktur gehört dem Repo (eigene Konventionen, `git mv`, Commits). Kontext-Artefakte leben außerhalb des Repos auf der Ebenen-Wurzel; Ausnahme ist das Master-Muster (Repo hält den Master, der Workspace Symlinks). Ändert der Kontext-Schritt doch eine Datei im Repo (z.B. Pointer-Reconcile): dirty Zustand im Bestätigungsblock ausweisen, committen bleibt beim User.
- **Symlink-Schutz.** Symlink-Ziele nie verschieben (bricht den Link) → flaggen. Einen Symlink selbst nur verschieben, wenn der Plan das Nachziehen des Links enthält.

## Workflow

1. **Anker lesen.** Die `context.md` im Workspace-Root + die des Sweep-Ordners (bzw. anlegen, falls sie fehlt — via `contextify`-Logik). Das ist der Rahmen, der nach unten injiziert wird, damit neue Contexts nicht duplizieren.

2. **Inventarisieren.** Vollständiges Listing des Subtrees — Ordner **und** lose Dateien, alle Typen. **Mess-Pflicht Repos/Symlinks:** `.git`-Verzeichnisse und Symlinks mechanisch erheben (`find {pfad} -name .git`, `find {pfad} -type l`) — der Befund (auch „keine") ist Pflichtbestandteil des Dispositions-Plans; eine Prüfung ohne erhobenen Befund hat nicht stattgefunden. **Mess-Pflicht Secrets:** den Subtree mechanisch auf Secret-Muster scannen — Passwort-/Token-Zuweisungen in Textdateien (`grep -riE 'passwor[dt]|passwd|secret|api[_-]?key|token' --include='*.md' --include='*.txt' --include='*.env*'`, Treffer einzeln sichten — 1Password-`op://`-Verweise sind konform, Klartext-Werte nicht), `BEGIN.*PRIVATE KEY`, sowie Schlüssel-Dateitypen (`find {pfad} -name '*.pfx' -o -name '*.pem' -o -name '*.p12' -o -name '*.key'`). Der Befund — auch „keiner" — ist Pflichtbestandteil des Dispositions-Plans; Fundstücke werden als **Secrets-Verstoß** geflaggt (Ziel: Vault + Verweis, Datei löschen nur mit Freigabe). Spec: design.md §3, Secrets-Grenze.

3. **Klassifizieren** (der harte Kern — bei Ambiguität fragen). Je Ordner/Datei genau eine Kategorie:
   - **Eigener Gegenstand** — Vorhaben oder Bereich mit Substanz → bekommt `context.md` und wird damit Kontext-Ordner.
   - **Repo** — Ordner mit `.git`: atomare Einheit, als Ganzes zuordnen (oder selbst der Gegenstand); Innenleben nicht anfassen (Safety-Gate).
   - **temp.md** — flüchtiges Arbeitsblatt (Spec §3): bleibt liegen — nie archivieren, verschieben, umbenennen oder als Dup/Variante werten (Instanzen in verschiedenen Ordnern sind unabhängig). Offensichtlich Erntenswertes als Content-Punkt vorlegen (Schritt 7), nicht selbst umrouten.
   - **Input/Support** (`transkripte/`, `customer_inputs/`, …) → behalten, **kein** Kontext.
   - **Assets/Binärmaterial** (Bilder, Videos, Fonts, große Medien) → in einen benannten Ordner der Ebene (`bildmaterial/`, `assets/`) mit `README.md` für die Bau-Regeln; lose Binärdateien auf Ebenen-Wurzeln sind ein Flag. Schlüssel-/Zertifikatsdateien sind **nie** Assets → Secrets-Verstoß (s. Inventar-Schritt).
   - **Archiv / Superseded** — ersetzte Entwürfe, alte Stände → `archive/`.
   - **Dup** — gleicher Inhalt, existiert woanders → `archive/`.
   - **Variante** — gleicher Name, abweichender Inhalt → `archive/` + `.VARIANT` + flaggen.
   - **Quer-liegend / fehlplatziert** — gehört erkennbar woandershin (eingebettetes Fremd-System, Material aus einer anderen Domäne) → **flaggen, nicht zwingen**.

4. **Dispositions-Plan vorlegen** — Tabelle aller geplanten Aktionen → Freigabe abwarten.

5. **Ausführen.** Fehlende Ordner anlegen; Dateien verschieben; (autorisierte) Löschungen; pro neuem Kontext-Ordner die **`contextify`-Logik** (`context.md` gegen das Backbone, mit dem Kontext von oben injiziert, Maxime „referenziere nach oben, dupliziere nicht").

6. **Nach oben reconcilen.** In der übergeordneten `context.md` einen **Pointer-Index** auf die neuen Kontext-Ordner setzen. **Alle Pointer verschobener Dateien in `context.md`/`memory.md`/`log.md` nachziehen** (Move erzeugt eine Pointer-Kaskade — der fehleranfälligste Schritt). Gedächtnis bleibt liegen, wo es liegt; wo `log.md`/`memory.md` entstehen sollen, entscheidet der User (`design.md` §6).

7. **Content-Punkte einsammeln.** Fakten, die der Sweep ans Licht bringt (z.B. „Ansatz X wurde gekippt"), in das passende `context.md`/`memory.md` routen — nicht verlieren.

8. **Bestätigung.** Kompakt: was angelegt / verschoben / archiviert / geflaggt wurde; **offene Flags explizit auflisten** (Varianten zum Prüfen, Quer-Liegendes, deep-doc-interne Verweise, die nicht nachgezogen wurden).

## Verhältnis zu den anderen Skills

- **`contextify`** = das Atom (eine `context.md` gegen das Backbone). `cleanup` ruft dessen Logik je Kontext-Ordner auf. Fehlt `contextify`, ist aber die Spec da: `cleanup` baut die Contexts selbst gegen das Backbone. Echte Degradation nur, wenn die Spec fehlt — dann Abbruch des Kontext-Schritts (Tidy + Klassifikation + Flags laufen trotzdem; Hinweis „contextify/Spec nachziehen").
- **`remember`** = pflegt `memory.md`/`log.md` **inhaltlich** (Übergänge, Urteile, Verdichtung, Task-Closure). `cleanup` fasst sie nur **strukturell** an (verkümmerte Logs nach Autorisierung) — die inhaltliche Verdichtung bleibt `remember`.

## Harte Gates (Kurzfassung)

- Spec (`design.md` / `template_context.md`) erreichbar — sonst Abbruch.
- Secrets-Scan im Inventar — Befund (auch „keiner") im Dispositions-Plan.
- Scope aus dem Pfad, sonst nachfragen; breiter Scope → Plan zuerst.
- Plan-dann-Bestätigen, nie blind ausführen.
- `mv` statt `rm`; löschen nur explizit autorisiert; Zweifel → `archive/`.
- Varianten flaggen, nicht entscheiden.
- Pointer-Reconcile ist Pflicht, kein Bonus.
- Repos atomar behandeln (nichts hinein/heraus/löschen), Symlink-Ziele nie verschieben; Repo-/Symlink-Befund im Plan ausweisen.
- Kontext-Ordner sind die mit `context.md`; Sammelordner bekommen keine.
- Bei Klassifikations-Ambiguität: fragen.
