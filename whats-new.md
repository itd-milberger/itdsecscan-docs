<div class="lang-en" markdown="1">

# What's new

The changes a user notices, newest first. The full changelog, with every fix, ships
in the `README.md` of each package.

## 0.8.5 — 30 September 2026

- **Configuration reference.** [Configuration file](reference/config.md) lists
  every key `ITdSecScan.config` accepts, with its default and meaning. It is
  generated from the list the scanner loads the file with, so it cannot fall out
  of step.
- **Fixed: three config keys were ignored.** `AD_KEYCHAIN_ACCOUNT`,
  `AD_KEYCHAIN_SERVICE` and `AD_SYSVOL_PATH` were read from the environment but
  dropped when set in the config file, although the example configs set them. They
  now take effect from the file.

## 0.8.4 — 30 September 2026

- **Shorter console help.** `ITdSecScan.exe -h` is the option list, a few examples
  and a link to this site.

## 0.8.3 — 30 September 2026

- **Management summary for management.** `-mgmt-report` works with every run that
  writes a report and looks the same with or without a comparison; a comparison
  adds what changed. German by default, `-lang en` for English. See
  [The management summary](guide/report.md#the-management-summary).
- **PDF.** `-pdf` saves the management summary as PDF (needs Microsoft Edge or
  Google Chrome).
- **The customer in every heading.** `CUSTOMER_NAME` in the config, or `-customer`
  for one run. See [The customer in the heading](guide/report.md#the-customer-in-the-heading).
- **New design** in the company branding: the IT/DESIGN logo, navy and blue, and
  Montserrat — in every report and on this site.

## 0.8.2 — 29 September 2026

- **Apps holding directory roles are reported.** An app registration with Global
  Administrator no longer goes unlisted.
- **Four new Entra checks:** stale enterprise apps, inactive members, unused
  licences and stale devices. Better credential findings for app registrations,
  and permission ratings beyond Microsoft Graph (Exchange Online, SharePoint,
  Defender).

## 0.8.1 — 29 September 2026

- **The setup script prints the config lines to paste** into `ITdSecScan.config`
  at the end of every run.

## 0.8.0 — 29 September 2026

- **A new risk score that sees how much is open.** Every open finding counts by
  severity, so closing findings lowers the score. Older baselines are re-scored when
  loaded, so two scans are always compared on the same scale.

## 0.7.9 — 28 September 2026

- **Compare against an older baseline.** `-compare-against` lists the saved
  baselines and compares with the one you pick.

## 0.7.5 – 0.7.8 — September 2026

- **Password leak check.** `-checkpasswords <file>` checks NTLM hashes against the
  HaveIBeenPwned Pwned Passwords list; only a five-character hash prefix leaves the
  machine.
- **macOS packages** beside the Windows ones.
- **Readable drift overview:** a changed finding shows once, with removed text
  struck through and added text highlighted.
- **DCSync-Eligible rates what a replication right actually allows**, and no longer
  reports rights that apply only to objects below the domain.

## 0.7.0 — 28 August 2026

- **One list file for accepted risks and to-dos**, merged automatically from
  `<baseline>/inbox/`.
- **This documentation site**, with reference pages generated from the scanner.

</div>

<div class="lang-de" markdown="1">

# Neuigkeiten

Die Änderungen, die ein Anwender bemerkt, die neuesten zuerst. Das vollständige
Änderungsprotokoll mit jeder Korrektur liegt in der `README.md` jedes Pakets.

## 0.8.5 — 30. September 2026

- **Referenz der Konfiguration.** [Konfigurationsdatei](reference/config.md) listet
  jeden Schlüssel, den `ITdSecScan.config` akzeptiert, mit Standardwert und
  Bedeutung. Sie wird aus der Liste generiert, mit der der Scanner die Datei lädt,
  und kann deshalb nicht veralten.
- **Behoben: drei Konfigurationsschlüssel wurden ignoriert.** `AD_KEYCHAIN_ACCOUNT`,
  `AD_KEYCHAIN_SERVICE` und `AD_SYSVOL_PATH` wurden aus der Umgebung gelesen, beim
  Setzen in der Konfigurationsdatei aber verworfen, obwohl die Beispielkonfigurationen
  sie setzen. Sie wirken jetzt auch aus der Datei.

## 0.8.4 — 30. September 2026

- **Kürzere Konsolenhilfe.** `ITdSecScan.exe -h` zeigt die Optionen, einige
  Beispiele und einen Link auf diese Seite.

## 0.8.3 — 30. September 2026

- **Management Summary für das Management.** `-mgmt-report` funktioniert mit jedem
  Lauf, der einen Report schreibt, und sieht mit und ohne Vergleich gleich aus; ein
  Vergleich ergänzt, was sich geändert hat. Standardmäßig deutsch, `-lang en` für
  Englisch. Siehe [Die Management Summary](guide/report.md#the-management-summary).
- **PDF.** `-pdf` speichert die Management Summary als PDF (benötigt Microsoft Edge
  oder Google Chrome).
- **Der Kunde in jeder Überschrift.** `CUSTOMER_NAME` in der Konfiguration oder
  `-customer` für einen Lauf. Siehe [Der Kunde in der Überschrift](guide/report.md#the-customer-in-the-heading).
- **Neues Design** im Firmen-Branding: das IT/DESIGN-Logo, Navy und Blau sowie
  Montserrat — in jedem Report und auf dieser Seite.

## 0.8.2 — 29. September 2026

- **Apps mit Verzeichnisrollen werden gemeldet.** Eine App-Registrierung mit
  Global Administrator bleibt nicht mehr ungenannt.
- **Vier neue Entra-Checks:** veraltete Enterprise-Apps, inaktive Mitglieder,
  ungenutzte Lizenzen und veraltete Geräte. Bessere Befunde zu Zugangsdaten von
  App-Registrierungen und Bewertungen von Berechtigungen über Microsoft Graph
  hinaus (Exchange Online, SharePoint, Defender).

## 0.8.1 — 29. September 2026

- **Das Setup-Skript gibt die Zeilen für `ITdSecScan.config` aus**, am Ende jedes
  Laufs, zum Einfügen.

## 0.8.0 — 29. September 2026

- **Ein neuer Risk Score, der sieht, wie viel offen ist.** Jeder offene Befund zählt
  nach Schweregrad, sodass behobene Befunde den Score senken. Ältere Baselines
  werden beim Laden neu bewertet, damit zwei Scans immer auf derselben Skala
  verglichen werden.

## 0.7.9 — 28. September 2026

- **Vergleich mit einer älteren Baseline.** `-compare-against` listet die
  gespeicherten Baselines und vergleicht mit der gewählten.

## 0.7.5 – 0.7.8 — September 2026

- **Passwort-Leak-Check.** `-checkpasswords <Datei>` prüft NTLM-Hashes gegen die
  Pwned-Passwords-Liste von HaveIBeenPwned; nur ein fünfstelliges Hash-Präfix
  verlässt den Rechner.
- **macOS-Pakete** neben den Windows-Paketen.
- **Lesbare Drift-Übersicht:** Ein geänderter Befund erscheint einmal, Entferntes
  durchgestrichen, Hinzugekommenes hervorgehoben.
- **DCSync-Eligible bewertet, was ein Replikationsrecht tatsächlich erlaubt**, und
  meldet keine Rechte mehr, die nur für Objekte unterhalb der Domäne gelten.

## 0.7.0 — 28. August 2026

- **Eine Listendatei für akzeptierte Risiken und To-dos**, automatisch übernommen
  aus `<baseline>/inbox/`.
- **Diese Dokumentationsseite**, mit Referenzseiten, die aus dem Scanner generiert
  werden.

</div>
