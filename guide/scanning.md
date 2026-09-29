<div class="lang-en" markdown="1">

# Running a scan

## The short version

```powershell
.\ITdSecScan.exe
```

No options: every check that can run, and an HTML report plus its JSON model
written into `html-reports\`. That is the whole normal case.

## Baselines and comparisons

A baseline is a stored snapshot of every check's results. Later scans compare
against it to report what changed.

```powershell
.\ITdSecScan.exe -baseline                 # scan and save a snapshot
.\ITdSecScan.exe -compare-latest           # scan, compare against the newest snapshot, write the HTML report
.\ITdSecScan.exe -compare-against          # pick the snapshot to compare against from a list, then scan
.\ITdSecScan.exe -list                     # list all saved baselines
```

### Which baseline a comparison runs against

| Mode | Reference |
|---|---|
| `-compare-latest` | The newest saved baseline, without asking. This is what a Scheduled Task runs. |
| `-compare-against` | The one you pick. The scanner lists the saved baselines — first AD, then Entra — numbered, newest first, with the time they were saved, the overall risk score and the number of findings, and you type the number. **Enter** takes the newest, `q` aborts. |
| `-compare` | Same as `-compare-latest`. The old spelling, kept so existing Scheduled Tasks keep running. |

`-compare-against` asks every question **before** anything is scanned, so once
the last number is typed the run needs nobody at the keyboard. A store with only
one baseline is still listed, so you see what the comparison runs against, but
there is nothing to type. With `-profiles` each list names the profile it
belongs to.

It needs an interactive terminal. Run without one — from a Scheduled Task, or with
input redirected — it stops at once with exit code 2 and points at
`-compare-latest`, rather than waiting on a prompt nobody will answer.

The **Changes** tab states the reference baseline with the time it was saved and
whether it was the newest or chosen from the list. Several baselines can be saved
on one day, and a reference chosen months back makes every number larger.

### Additions

Both modes always write the full HTML comparison report, not only console output.
`-mgmt-report` and `-drift-report` are real additions to them and are refused on
their own — `-compare-latest`, `-compare-against` or `-render` has to be present,
since there is otherwise no comparison for them to write:

| Addition | Effect |
|---|---|
| `-mgmt-report` | Also write a short management summary. |
| `-drift-report` | Also write an overview restricted to material drift. Time-only changes and a list merely reordered are hidden; a changed finding is shown once, with what was removed struck through and what was added highlighted. |

`-changed` and `-compare-html` are accepted but have **no effect** — a comparison
run always reports every check and always writes the HTML report. They stay
accepted so existing scripts and scheduled tasks that still pass them keep
running.

A comparison report carries an additional **Changes** tab.

### Comparing two saved baselines

With `-offline` nothing is scanned; two saved baselines are compared with each
other:

```powershell
.\ITdSecScan.exe -offline -compare-latest      # the two newest
.\ITdSecScan.exe -offline -compare-against     # pick both
```

`-offline -compare-against` asks twice per store: first the reference (the older
state), then the baseline to compare it with. The second list offers only
baselines newer than the reference, so a comparison cannot run backwards and
report every fix as a new finding.

**How the per-check verdict is decided:** from the finding set, weighted by
severity — not from the risk score, which is a bounded scale and moves least where
most is open.

**An account enabled again counts as a finding added**, whatever its text says. A
disabled Domain Admin that can log on again produces a membership entry that reads
exactly as before, so it is compared on the account state too: re-enabling a
member of a Tier 0 group, or an account with `adminCount=1`, is a **critical**
change, any other account **high**. An account that leaves the disabled-accounts
list while another check shows it enabled is treated the same way. One that leaves
the list and appears nowhere is most likely deleted — it counts as resolved, and
the management summary asks you to confirm it.

| Verdict | When |
|---|---|
| **degraded** | Findings were added and the added ones outweigh any removed, by severity. |
| **improved** | Findings were removed and the removed ones outweigh any added. |
| **changed** | Findings were exchanged in both directions with equal weight, or their identities changed. |
| **unchanged** | No finding, count or severity moved. |

For a check that reports a single state rather than a list of findings, the count
decides, and the risk score only if the count is equal as well.

## Re-rendering a report without scanning

Every report is written with a JSON model beside it. That model is the report:

```powershell
.\ITdSecScan.exe -render <report>.json
```

No scan, no LDAP, no baseline directory, no configuration. It reproduces the report
byte for byte — the generation time and operator come from the model, not the
clock — which is what makes a report reproducible six months later.

It also means a re-render cannot show findings the original scan did not record. A
newer version's improvements to *presentation* appear; its improvements to
*collection* do not.

## Several environments from one installation

A profile is a configuration file beside the executable, `ITdSecScan.<name>.config`:

```powershell
.\ITdSecScan.exe -list-profiles
.\ITdSecScan.exe -profile customer-a
.\ITdSecScan.exe -profiles customer-a,customer-b
.\ITdSecScan.exe -profiles all
```

**Each profile owns its own baseline store**, under `baselines\profiles\<name>\`,
and that is a correctness property rather than a convenience: two tenants sharing a
store would compare one tenant's snapshot against the other's, and every drift
number after that would be meaningless. The accepted-risk and to-do lists are
per-profile for the same reason.

## Where things are written

```
html-reports\                     every report, HTML plus its JSON model
baselines\
  ad\  azure\                     snapshots, with an index
  whitelist.json                  accepted risks
  todos.json                      to-dos
  inbox\                          exported list files to merge on the next run
  inbox\merged\                   list files already applied, timestamped
  profiles\<name>\                the same layout, per profile
```

Override the root with `BASELINE_DIR`.

The store keeps the two lists in separate files. The **export** out of a report is
a single file holding both, so there is one file to hand on and one file to put
back — see [accepted risks and to-dos](accepted-risks-and-todos.md).

## Exit codes

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | Unknown error |
| 2 | Invalid arguments |
| 20 | Configuration invalid |
| 21 | LDAP connection failed |
| 22 | Access denied |
| 30–40 | Baseline or report error |
| 50 | Timeout |

Every option is listed in the [command line reference](../reference/cli.md).

</div>

<div class="lang-de" markdown="1">

# Einen Scan ausführen

## Die Kurzversion

```powershell
.\ITdSecScan.exe
```

Ohne Optionen: jeder Check, der laufen kann, plus ein HTML-Report samt seinem
JSON-Modell, geschrieben nach `html-reports\`. Das ist der komplette Normalfall.

## Baselines und Vergleiche

Eine Baseline ist eine gespeicherte Momentaufnahme der Ergebnisse jedes Checks.
Spätere Scans vergleichen sich damit und melden, was sich geändert hat.

```powershell
.\ITdSecScan.exe -baseline                 # scannen und eine Momentaufnahme speichern
.\ITdSecScan.exe -compare-latest           # scannen, mit der neuesten Momentaufnahme vergleichen, HTML-Report schreiben
.\ITdSecScan.exe -compare-against          # Momentaufnahme für den Vergleich aus einer Liste wählen, dann scannen
.\ITdSecScan.exe -list                     # alle gespeicherten Baselines auflisten
```

### Gegen welche Baseline verglichen wird

| Modus | Referenz |
|---|---|
| `-compare-latest` | Die neueste gespeicherte Baseline, ohne Rückfrage. Das ist, was ein geplanter Task ausführt. |
| `-compare-against` | Die, die Sie wählen. Der Scanner listet die gespeicherten Baselines auf — zuerst AD, dann Entra —, nummeriert, die neueste oben, mit Speicherzeitpunkt, Gesamt-Risikowert und Anzahl der Befunde, und Sie tippen die Nummer ein. **Enter** nimmt die neueste, `q` bricht ab. |
| `-compare` | Wie `-compare-latest`. Die alte Schreibweise, beibehalten, damit bestehende geplante Tasks weiterlaufen. |

`-compare-against` stellt alle Fragen, **bevor** gescannt wird. Sobald die letzte
Nummer eingetippt ist, braucht der Lauf niemanden mehr an der Tastatur. Ein
Speicher mit nur einer Baseline wird trotzdem aufgelistet, damit Sie sehen, wogegen
verglichen wird, es gibt aber nichts einzutippen. Mit `-profiles` nennt jede Liste
das Profil, zu dem sie gehört.

Es braucht ein interaktives Terminal. Ohne eines — aus einem geplanten Task oder mit
umgeleiteter Eingabe — bricht es sofort mit Exit-Code 2 ab und verweist auf
`-compare-latest`, statt auf eine Eingabe zu warten, die niemand macht.

Der **Changes**-Tab nennt die Referenz-Baseline mit ihrem Speicherzeitpunkt und
ob es die neueste war oder aus der Liste gewählt wurde. An einem Tag können mehrere
Baselines entstehen, und eine Referenz von vor Monaten macht jede Zahl größer.

### Ergänzungen

Beide Modi schreiben immer den vollständigen HTML-Vergleichsreport, nicht nur
Konsolenausgabe. `-mgmt-report` und `-drift-report` sind echte Ergänzungen dazu und
werden allein verweigert — `-compare-latest`, `-compare-against` oder `-render` muss
vorhanden sein, da es sonst keinen Vergleich gibt, den sie schreiben könnten:

| Ergänzung | Wirkung |
|---|---|
| `-mgmt-report` | Schreibt zusätzlich eine kurze Management-Zusammenfassung. |
| `-drift-report` | Schreibt zusätzlich eine auf wesentliche Drift beschränkte Übersicht. Reine Zeitänderungen und bloß umsortierte Listen werden ausgeblendet; ein geänderter Befund erscheint einmal, Entferntes durchgestrichen, Hinzugekommenes hervorgehoben. |

`-changed` und `-compare-html` werden akzeptiert, haben aber **keine Wirkung** —
ein Vergleichslauf meldet immer jeden Check und schreibt immer den HTML-Report.
Sie bleiben akzeptiert, damit bestehende Skripte und geplante Aufgaben, die sie
weiterhin übergeben, funktionsfähig bleiben.

Ein Vergleichsreport trägt einen zusätzlichen **Changes**-Tab.

### Zwei gespeicherte Baselines vergleichen

Mit `-offline` wird nichts gescannt; zwei gespeicherte Baselines werden miteinander
verglichen:

```powershell
.\ITdSecScan.exe -offline -compare-latest      # die zwei neuesten
.\ITdSecScan.exe -offline -compare-against     # beide auswählen
```

`-offline -compare-against` fragt pro Speicher zweimal: zuerst die Referenz (den
älteren Stand), dann die Baseline, mit der sie verglichen wird. Die zweite Liste
bietet nur Baselines an, die neuer als die Referenz sind, damit ein Vergleich nicht
rückwärts läuft und jede Behebung als neuen Befund meldet.

**Wie das Urteil je Check entschieden wird:** aus der Fundmenge, gewichtet nach
Schweregrad — nicht aus dem Risk Score, der eine begrenzte Skala ist und sich dort
am wenigsten bewegt, wo am meisten offen ist.

**Ein wieder aktivierter Account zählt als hinzugefügter Fund**, egal was sein Text
sagt. Ein deaktivierter Domain Admin, der sich wieder anmelden kann, erzeugt einen
Mitgliedschaftseintrag, der genauso lautet wie vorher; deshalb wird auch der
Account-Zustand verglichen: Die Reaktivierung eines Mitglieds einer Tier-0-Gruppe
oder eines Accounts mit `adminCount=1` ist eine **kritische** Änderung, jeder andere
Account **high**. Ein Account, der die Liste der deaktivierten Accounts verlässt,
während ein anderer Check ihn als aktiv zeigt, wird genauso behandelt. Einer, der die
Liste verlässt und nirgends mehr auftaucht, wurde sehr wahrscheinlich gelöscht — er
zählt als behoben, und die Management Summary bittet um Bestätigung.

| Urteil | Wann |
|---|---|
| **degraded** | Funde wurden hinzugefügt, und die hinzugefügten überwiegen nach Schweregrad alle entfernten. |
| **improved** | Funde wurden entfernt, und die entfernten überwiegen nach Schweregrad alle hinzugefügten. |
| **changed** | Funde wurden in beide Richtungen mit gleichem Gewicht getauscht, oder ihre Identitäten haben sich geändert. |
| **unchanged** | Kein Fund, keine Anzahl und kein Schweregrad hat sich bewegt. |

Bei einem Check, der einen einzelnen Zustand statt einer Fundliste meldet,
entscheidet die Anzahl, und der Risk Score nur, wenn auch die Anzahl gleich ist.

## Einen Report neu rendern, ohne zu scannen

Jeder Report wird mit einem JSON-Modell daneben geschrieben. Dieses Modell ist
der Report:

```powershell
.\ITdSecScan.exe -render <report>.json
```

Kein Scan, kein LDAP, kein Baseline-Verzeichnis, keine Konfiguration. Es
reproduziert den Report Byte für Byte — Erstellungszeit und Bearbeiter kommen aus
dem Modell, nicht von der Uhr — und genau das macht einen Report auch nach sechs
Monaten noch reproduzierbar.

Das bedeutet auch: Ein erneutes Rendern kann keine Funde zeigen, die der
ursprüngliche Scan nicht erfasst hat. Verbesserungen einer neueren Version an der
*Darstellung* erscheinen; Verbesserungen an der *Erfassung* nicht.

## Mehrere Umgebungen aus einer Installation

Ein Profil ist eine Konfigurationsdatei neben der ausführbaren Datei,
`ITdSecScan.<name>.config`:

```powershell
.\ITdSecScan.exe -list-profiles
.\ITdSecScan.exe -profile customer-a
.\ITdSecScan.exe -profiles customer-a,customer-b
.\ITdSecScan.exe -profiles all
```

**Jedes Profil besitzt seinen eigenen Baseline-Speicher**, unter
`baselines\profiles\<name>\` — das ist eine Korrektheitseigenschaft und keine
Bequemlichkeit: Würden sich zwei Tenants einen Speicher teilen, verglichen sich
die Momentaufnahme des einen mit der des anderen, und jede Drift-Zahl danach wäre
bedeutungslos. Die Listen für Accepted Risks und To-Dos sind aus demselben Grund
je Profil getrennt.

## Wohin geschrieben wird

```
html-reports\                     jeder Report, HTML plus sein JSON-Modell
baselines\
  ad\  azure\                     Momentaufnahmen, mit einem Index
  whitelist.json                  Accepted Risks
  todos.json                      To-Dos
  inbox\                          exportierte Listendateien, beim nächsten Lauf zusammengeführt
  inbox\merged\                   bereits angewendete Listendateien, mit Zeitstempel
  profiles\<name>\                dasselbe Layout, je Profil
```

Das Wurzelverzeichnis mit `BASELINE_DIR` überschreiben.

Der Speicher hält die beiden Listen in getrennten Dateien. Der **Export** aus
einem Report ist eine einzelne Datei mit beiden Listen, sodass es eine Datei zum
Weitergeben und eine zum Zurücklegen gibt — siehe
[Accepted Risks und To-Dos](accepted-risks-and-todos.md).

## Exit-Codes

| Code | Bedeutung |
|---|---|
| 0 | Erfolg |
| 1 | Unbekannter Fehler |
| 2 | Ungültige Argumente |
| 20 | Konfiguration ungültig |
| 21 | LDAP-Verbindung fehlgeschlagen |
| 22 | Zugriff verweigert |
| 30–40 | Baseline- oder Report-Fehler |
| 50 | Timeout |

Jede Option ist in der [Kommandozeilen-Referenz](../reference/cli.md) aufgeführt.

</div>
