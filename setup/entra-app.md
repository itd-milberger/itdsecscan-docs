<div class="lang-en" markdown="1">

# Setting up the Entra ID app registration

The Entra checks authenticate as an app registration with read-only Graph
permissions. This is a one-time setup per tenant, and it is the step that most
often goes wrong in a way that is hard to see — so read the last section even if
the script succeeds.

## Run the script

`scripts/Setup-AzureAppRegistration.ps1`, as a Global Administrator:

```powershell
pwsh -File Setup-AzureAppRegistration.ps1 `
     -Organization contoso.com `
     -AppName ITdSecScan-Prod
```

It creates the app registration, requests the [Graph permissions](../reference/graph-permissions.md)
the checks need, grants admin consent, issues a self-signed certificate, and writes
`<AppName>.pfx` plus an `<AppName>.env` with the values to copy into your config.

When it creates a certificate, the script **asks for the password** that protects
the exported `.pfx`; the typing is hidden. There is no parameter for it — a
password on the command line would land in the shell history.

Re-running is safe: only missing permissions are added, and existing credentials
are left alone.

Useful variants:

| | |
|---|---|
| `-PermissionsOnly` | Add newly required permissions to an existing app without touching its credentials. This is what you run after a version that adds a check. |
| `-Show` | Read-only: print the app's permissions with their consent status, its certificates with expiry, and the values for your config. Needs no Global Admin. |
| `-Auth secret` \| `both` | A client secret instead of, or in addition to, a certificate. |
| `-RenewCertificate` | Issue a new certificate and export a fresh `.pfx` — for when the old one was lost. Entra stores only the public key, so a previous private key cannot be recovered. |
| `-PruneUnusedPermissions` | Revoke permissions the tool does not use. Opt-in, because revoking is destructive and an administrator may have granted something by hand for another purpose. |

## `-AppName` is required

There is no default. The script finds the app registration by display name, and
Microsoft Graph matches display names case-insensitively — so `ITdSecScan` also
matches an app called `itdsecscan`.

If more than one app registration in the tenant carries the name you pass, the
script lists them with their client IDs and stops without changing anything. Pass
the exact display name of the app you mean.

**The app you configure must be the app the scanner authenticates as.** The script
prints the client ID it configured at the end of its run; that value has to match
`AZURE_CLIENT_ID` in the scanner's configuration. If they differ, the permissions
are granted to one app while the scan reads the tenant as another, and the scan
result does not change.

## Configure the scanner

**Every run of the script ends with a block headed `VALUES FOR ITdSecScan.config`**
— the setup, `-Show`, `-PermissionsOnly` and the maintenance runs alike — holding
the lines to paste. To connect another machine to an app that already exists, run
`-Show` and copy them. The certificate thumbprint is that of the newest valid
certificate, and the block says whether that certificate is already in your
certificate store or still has to be imported. A PFX password or client secret
cannot be read back from Entra ID; the block points to the `.env` the setup wrote
instead.

**The quickest way: use the `.env` file.** The setup run — and
`-RenewCertificate` — writes `<AppName>.env` next to the `.pfx`; the script prints
the folder as `Output dir` when it starts and the full path under `OUTPUT FILES`
when it finishes (set it with `-OutputDir`). It already holds every `AZURE_*` line with the real values, the PFX
password or client secret included, in exactly the format `ITdSecScan.config`
uses. Copy its lines into `ITdSecScan.config` as they are, below your `AD_*`
settings. Because it contains the password in plain text, keep the file somewhere
safe, or delete it once the config works; `-RenewCertificate` writes a new one. `-Show` and `-PermissionsOnly` write no
`.env`; their block shows the values without the password.

On Windows the lines are:

```ini
AZURE_TENANT_ID=<tenant guid>
AZURE_CLIENT_ID=<client id the script printed>
AZURE_AUTH_MODE=cert_store
AZURE_CERT_THUMBPRINT=<thumbprint>
```

`cert_store` reads the certificate from the Windows certificate store, so the
private key never leaves it — this works with non-exportable and TPM-backed keys.
Import the PFX with:

```powershell
certutil -user -importpfx My <AppName>.pfx
```

`AZURE_AUTH_MODE=cert_store` may be left out: a thumbprint on its own selects it.
Alternatives are `AZURE_AUTH_MODE=certificate` with `AZURE_CERT_PATH` and
`AZURE_CERT_PASSWORD` (macOS and Linux), or `secret` with `AZURE_CLIENT_SECRET`.

## Verify it, and read the answer carefully

```powershell
.\ITdSecScan.exe -testauth azure
```

This prints the tenant, the app registration it authenticated as, and every
permission actually granted. Compare the app name and client ID against what the
setup script reported. If they differ, that is the problem — not the consent.

Every scan re-checks this and puts the result in the report's **Info** tab. A
missing permission is named there together with the checks it silences, because a
check that cannot read returns nothing, and nothing in a report reads exactly like
a clean tenant.

</div>

<div class="lang-de" markdown="1">

# Die Entra-ID-App-Registrierung einrichten

Die Entra-Checks authentifizieren sich als App-Registrierung mit nur lesenden
Graph-Berechtigungen. Das ist eine einmalige Einrichtung pro Tenant — und der
Schritt, bei dem am häufigsten etwas auf schwer erkennbare Weise schiefgeht.
Deshalb den letzten Abschnitt lesen, auch wenn das Skript erfolgreich durchläuft.

## Das Skript ausführen

`scripts/Setup-AzureAppRegistration.ps1`, als Globaler Administrator:

```powershell
pwsh -File Setup-AzureAppRegistration.ps1 `
     -Organization contoso.com `
     -AppName ITdSecScan-Prod
```

Es erstellt die App-Registrierung, fordert die von den Checks benötigten
[Graph-Berechtigungen](../reference/graph-permissions.md) an, erteilt den
Admin-Consent, stellt ein selbstsigniertes Zertifikat aus und schreibt
`<AppName>.pfx` sowie eine `<AppName>.env` mit den Werten für die eigene
Konfiguration.

Wenn es ein Zertifikat erstellt, **fragt das Skript nach dem Passwort**, das die
exportierte `.pfx` schützt; die Eingabe ist verdeckt. Einen Parameter dafür gibt es
nicht — ein Passwort auf der Kommandozeile landet im Verlauf der Shell.

Erneutes Ausführen ist unbedenklich: Es werden nur fehlende Berechtigungen
hinzugefügt, bestehende Zugangsdaten bleiben unangetastet.

Nützliche Varianten:

| | |
|---|---|
| `-PermissionsOnly` | Fügt einer bestehenden App neu benötigte Berechtigungen hinzu, ohne ihre Zugangsdaten anzurühren. Das läuft man nach einer Version, die einen Check hinzufügt. |
| `-Show` | Nur lesend: gibt die Berechtigungen der App mit Consent-Status aus, ihre Zertifikate mit Ablaufdatum und die Werte für die eigene Konfiguration. Braucht keinen Globalen Administrator. |
| `-Auth secret` \| `both` | Ein Client-Secret statt, oder zusätzlich zu, einem Zertifikat. |
| `-RenewCertificate` | Stellt ein neues Zertifikat aus und exportiert eine frische `.pfx` — für den Fall, dass das alte verloren ging. Entra speichert nur den öffentlichen Schlüssel, ein vorheriger privater Schlüssel kann also nicht wiederhergestellt werden. |
| `-PruneUnusedPermissions` | Entzieht Berechtigungen, die das Tool nicht nutzt. Opt-in, weil ein Entzug destruktiv ist und ein Administrator etwas per Hand für einen anderen Zweck erteilt haben könnte. |

## `-AppName` ist erforderlich

Es gibt keinen Standardwert. Das Skript findet die App-Registrierung über den
Anzeigenamen, und Microsoft Graph vergleicht Anzeigenamen ohne Berücksichtigung
der Groß-/Kleinschreibung — `ITdSecScan` passt also auch auf eine App namens
`itdsecscan`.

Trägt mehr als eine App-Registrierung im Tenant den übergebenen Namen, listet das
Skript sie mit ihren Client-IDs auf und beendet sich, ohne etwas zu ändern. Den
exakten Anzeigenamen der gemeinten App übergeben.

**Die konfigurierte App muss die App sein, als die sich der Scanner
authentifiziert.** Das Skript gibt am Ende seines Laufs die konfigurierte
Client-ID aus; dieser Wert muss mit `AZURE_CLIENT_ID` in der Konfiguration des
Scanners übereinstimmen. Weichen sie voneinander ab, werden die Berechtigungen
an eine App erteilt, während der Scan den Tenant als eine andere liest — und das
Scan-Ergebnis ändert sich dadurch nicht.

## Den Scanner konfigurieren

**Jeder Lauf des Skripts endet mit einem Block `VALUES FOR ITdSecScan.config`** —
die Einrichtung ebenso wie `-Show`, `-PermissionsOnly` und die Wartungsläufe — mit
den Zeilen zum Einfügen. Um einen weiteren Rechner mit einer bestehenden App zu
verbinden, `-Show` ausführen und sie kopieren. Der Zertifikat-Thumbprint ist der
des neuesten gültigen Zertifikats, und der Block sagt, ob dieses Zertifikat schon
im eigenen Zertifikatspeicher liegt oder noch importiert werden muss. Ein
PFX-Passwort oder Client-Secret lässt sich aus Entra ID nicht zurücklesen; der
Block verweist stattdessen auf die `.env`, die die Einrichtung geschrieben hat.

**Am schnellsten: die `.env`-Datei verwenden.** Die Einrichtung — und
`-RenewCertificate` — schreibt `<AppName>.env` neben die `.pfx`; das Skript nennt
den Ordner beim Start als `Output dir` und den vollständigen Pfad am Ende unter
`OUTPUT FILES` (festlegen mit `-OutputDir`). Sie enthält bereits jede `AZURE_*`-Zeile mit den echten Werten,
einschließlich PFX-Passwort oder Client-Secret, genau im Format der
`ITdSecScan.config`. Ihre Zeilen unverändert in die `ITdSecScan.config` kopieren,
unter die eigenen `AD_*`-Einstellungen. Weil sie das Passwort im Klartext enthält,
die Datei sicher aufbewahren oder löschen, sobald die Konfiguration funktioniert;
`-RenewCertificate` schreibt eine neue. `-Show` und `-PermissionsOnly` schreiben
keine `.env`; ihr Block zeigt die Werte ohne das Passwort.

Unter Windows sind das die Zeilen:

```ini
AZURE_TENANT_ID=<tenant guid>
AZURE_CLIENT_ID=<client id the script printed>
AZURE_AUTH_MODE=cert_store
AZURE_CERT_THUMBPRINT=<thumbprint>
```

`cert_store` liest das Zertifikat aus dem Windows-Zertifikatspeicher, sodass der
private Schlüssel ihn nie verlässt — das funktioniert auch mit nicht
exportierbaren und TPM-gebundenen Schlüsseln. Die PFX importieren mit:

```powershell
certutil -user -importpfx My <AppName>.pfx
```

`AZURE_AUTH_MODE=cert_store` darf fehlen: Ein Thumbprint allein wählt es aus.
Alternativen sind `AZURE_AUTH_MODE=certificate` mit `AZURE_CERT_PATH` und
`AZURE_CERT_PASSWORD` (macOS und Linux), oder `secret` mit `AZURE_CLIENT_SECRET`.

## Verifizieren, und die Antwort sorgfältig lesen

```powershell
.\ITdSecScan.exe -testauth azure
```

Das gibt den Tenant aus, die App-Registrierung, als die authentifiziert wurde,
und jede tatsächlich erteilte Berechtigung. App-Name und Client-ID mit dem
vergleichen, was das Setup-Skript gemeldet hat. Weichen sie ab, liegt dort das
Problem — nicht beim Consent.

Jeder Scan prüft das erneut und trägt das Ergebnis in den **Info**-Tab des
Reports ein. Eine fehlende Berechtigung wird dort zusammen mit den Checks
genannt, die sie zum Schweigen bringt — denn ein Check, der nicht lesen kann,
liefert nichts, und nichts in einem Report liest sich exakt wie ein sauberer
Tenant.

</div>
