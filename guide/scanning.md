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
.\ITdSecScan.exe -compare                  # scan, compare against the newest snapshot, write the HTML report
.\ITdSecScan.exe -list                     # list all saved baselines
```

`-compare` is the mode, and it always writes the full HTML comparison report, not
only console output. `-mgmt-report` and `-drift-report` are real additions to it
and are refused on their own — `-compare` or `-render` has to be present, since
there is otherwise no comparison for them to write:

| Addition | Effect |
|---|---|
| `-mgmt-report` | Also write a short management summary. |
| `-drift-report` | Also write an overview restricted to material drift, hiding time-only changes. |

`-changed` and `-compare-html` are accepted but have **no effect** — a comparison
run always reports every check and always writes the HTML report. They stay
accepted so existing scripts and scheduled tasks that still pass them keep
running.

A comparison report carries an additional **Changes** tab.

**How the per-check verdict is decided:** from the finding set, weighted by
severity — not from the risk score. The score is normalised to 0–100, so adding
medium findings to a set of high ones can lower it while the estate has got worse.

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
.\ITdSecScan.exe -compare                  # scannen, mit der neuesten Momentaufnahme vergleichen, HTML-Report schreiben
.\ITdSecScan.exe -list                     # alle gespeicherten Baselines auflisten
```

`-compare` ist der Modus, und er schreibt immer den vollständigen HTML-Vergleichsreport,
nicht nur Konsolenausgabe. `-mgmt-report` und `-drift-report` sind echte Ergänzungen
dazu und werden allein verweigert — `-compare` oder `-render` muss vorhanden sein,
da es sonst keinen Vergleich gibt, den sie schreiben könnten:

| Ergänzung | Wirkung |
|---|---|
| `-mgmt-report` | Schreibt zusätzlich eine kurze Management-Zusammenfassung. |
| `-drift-report` | Schreibt zusätzlich eine auf wesentliche Drift beschränkte Übersicht, reine Zeitänderungen ausgeblendet. |

`-changed` und `-compare-html` werden akzeptiert, haben aber **keine Wirkung** —
ein Vergleichslauf meldet immer jeden Check und schreibt immer den HTML-Report.
Sie bleiben akzeptiert, damit bestehende Skripte und geplante Aufgaben, die sie
weiterhin übergeben, funktionsfähig bleiben.

Ein Vergleichsreport trägt einen zusätzlichen **Changes**-Tab.

**Wie das Urteil je Check entschieden wird:** aus der Fundmenge, gewichtet nach
Schweregrad — nicht aus dem Risk Score. Der Score ist auf 0–100 normiert, sodass
das Hinzufügen von Medium-Funden zu einer Menge von High-Funden ihn senken kann,
während sich der Bestand eigentlich verschlechtert hat.

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
