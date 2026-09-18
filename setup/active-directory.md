<div class="lang-en" markdown="1">

# Setting up Active Directory access

## On a domain member or a domain controller

Nothing to configure. The scanner uses the logged-on account through SSPI:

```ini
AD_AUTH_MODE=windows_current_user
AD_LDAP_URL=ldap://dc01.contoso.com:389
AD_BASE_DN=DC=contoso,DC=com
AD_LDAP_STARTTLS=true
```

## Which account to use

The scanner reads. It never writes to the directory, and nothing it generates
deletes anything — there is a test asserting that no generated fix command does.

**A privileged account sees more, and the difference is not cosmetic.** Security
descriptors, replication metadata, LAPS passwords and the Deleted Objects container
are readable only with sufficient rights. Bind as a standard user and several
checks return less, or nothing.

The report is explicit about which it was: the **Info** tab names the bound
account, whether it was privileged, and its group memberships. That matters when
comparing two reports — a drop in findings between them can mean the estate
improved or that the second scan ran with less access, and the two must not look
alike.

## Running from macOS or Linux

For development, or to scan from outside the domain:

```ini
AD_LDAP_URL=ldaps://contoso.com
AD_BIND_DN=CN=svc-scan,OU=Accounts,DC=contoso,DC=com
AD_BASE_DN=DC=contoso,DC=com
AD_AUTH_MODE=simple_bind
AD_BIND_PASSWORD=<password>
```

On macOS, keep the password out of the file with the keychain:

```ini
AD_AUTH_MODE=keychain
AD_KEYCHAIN_SERVICE=mcp-ad-ldap
```

```bash
security add-generic-password    -s "mcp-ad-ldap" -a "<bind dn>" -w   # first time
security add-generic-password -U -s "mcp-ad-ldap" -a "<bind dn>" -w   # after a change
security find-generic-password   -s "mcp-ad-ldap" -a "<bind dn>" -w   # verify
```

The password is read at startup, so restart after changing it.

## Group Policy and SYSVOL

Several checks read `Registry.pol` and `GptTmpl.inf` over SYSVOL to see what a GPO
actually sets, rather than only that it exists. On Windows this needs no setup. On
macOS or Linux, mount the share and point the scanner at it:

```ini
AD_SYSVOL_PATH=/Volumes/SYSVOL
```

Where SYSVOL is unreachable the affected checks say how many GPO paths they could
not read, instead of reporting the policies they could not see as absent.

## Verify

```powershell
.\ITdSecScan.exe -testauth ad
```

This prints the endpoint, the transport, the bound account and whether it is
privileged. Exit code 21 means the connection failed, 22 that it succeeded and the
account was refused.

</div>

<div class="lang-de" markdown="1">

# Active-Directory-Zugriff einrichten

## Auf einem Domänenmitglied oder einem Domänencontroller

Nichts zu konfigurieren. Der Scanner nutzt das angemeldete Konto über SSPI:

```ini
AD_AUTH_MODE=windows_current_user
AD_LDAP_URL=ldap://dc01.contoso.com:389
AD_BASE_DN=DC=contoso,DC=com
AD_LDAP_STARTTLS=true
```

## Welches Konto verwendet werden soll

Der Scanner liest. Er schreibt nie in das Verzeichnis, und nichts, was er
generiert, löscht etwas — ein Test stellt sicher, dass kein generierter
Fix-Befehl das tut.

**Ein privilegiertes Konto sieht mehr, und der Unterschied ist nicht
kosmetisch.** Security Descriptors, Replikationsmetadaten, LAPS-Passwörter und
der Deleted-Objects-Container sind nur mit ausreichenden Rechten lesbar. Bindet
man als Standardbenutzer, liefern mehrere Checks weniger oder gar nichts.

Der Report macht explizit, welches Konto es war: Der **Info**-Tab nennt das
gebundene Konto, ob es privilegiert war, und seine Gruppenmitgliedschaften. Das
ist beim Vergleich zweier Reports wichtig — ein Rückgang der Funde zwischen ihnen
kann bedeuten, dass sich der Bestand verbessert hat, oder dass der zweite Scan
mit weniger Zugriff lief, und die beiden Fälle dürfen nicht gleich aussehen.

## Ausführung von macOS oder Linux

Für die Entwicklung, oder um von außerhalb der Domäne zu scannen:

```ini
AD_LDAP_URL=ldaps://contoso.com
AD_BIND_DN=CN=svc-scan,OU=Accounts,DC=contoso,DC=com
AD_BASE_DN=DC=contoso,DC=com
AD_AUTH_MODE=simple_bind
AD_BIND_PASSWORD=<password>
```

Auf macOS das Passwort mit der Keychain aus der Datei heraushalten:

```ini
AD_AUTH_MODE=keychain
AD_KEYCHAIN_SERVICE=mcp-ad-ldap
```

```bash
security add-generic-password    -s "mcp-ad-ldap" -a "<bind dn>" -w   # erstmalig
security add-generic-password -U -s "mcp-ad-ldap" -a "<bind dn>" -w   # nach einer Änderung
security find-generic-password   -s "mcp-ad-ldap" -a "<bind dn>" -w   # prüfen
```

Das Passwort wird beim Start gelesen, nach einer Änderung also neu starten.

## Gruppenrichtlinien und SYSVOL

Mehrere Checks lesen `Registry.pol` und `GptTmpl.inf` über SYSVOL, um zu sehen,
was eine GPO tatsächlich einstellt, statt nur, dass sie existiert. Unter Windows
braucht das keine Einrichtung. Unter macOS oder Linux die Freigabe einbinden und
den Scanner darauf verweisen:

```ini
AD_SYSVOL_PATH=/Volumes/SYSVOL
```

Ist SYSVOL nicht erreichbar, sagen die betroffenen Checks, wie viele GPO-Pfade sie
nicht lesen konnten, statt die nicht einsehbaren Richtlinien als nicht vorhanden
zu melden.

## Verifizieren

```powershell
.\ITdSecScan.exe -testauth ad
```

Das gibt den Endpunkt, das Transportprotokoll, das gebundene Konto sowie aus, ob
es privilegiert ist. Exit-Code 21 bedeutet, die Verbindung ist fehlgeschlagen,
22, dass sie erfolgreich war und das Konto abgelehnt wurde.

</div>
