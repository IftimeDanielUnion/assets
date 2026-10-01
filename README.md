# DBI-11149 - SFTP-Verbindungen

**Status:** Noch nicht begonnen  
**Priorität:** Hoch, deshalb externe Rückfrage sofort anstoßen  
**Standalone:** Gegenüber den anderen Tickets ja  
**Externe Abhängigkeit:** Marc/Lukas muss bestätigen, dass der neue Public Key hinterlegt ist.

## Worum geht es einfach erklärt?

Die UMP überträgt Dateien per SFTP. Beim Lukas-PROD-Ziel gab es zuletzt Probleme.

Wichtig aus dem bisherigen Verlauf:

- Die Netzwerkverbindung kam bereits zustande.
- Danach gab es Probleme in der SSH-/Schlüsselstrecke.
- Ein neuer RSA-Public-Key wurde bereitgestellt.
- Der neue Schlüssel soll **zusätzlich** hinterlegt werden, nicht den alten ersetzen.

Der Public Key liegt direkt in diesem Ordner: `azure-lukas-sftp-prod.pub`.

## Schritt 1 - Nachricht jetzt an Marc

**An:** Marc Hintz  
**Adresse:** `hintz@msu.biz`  
**Anhang:** `azure-lukas-sftp-prod.pub`

**Betreff:** `DBI-11149 - Lukas Prod SFTP Zugang`

> Hallo Marc,
>
> ich übernehme aktuell DBI-11149 und würde den noch offenen SFTP-Zugang zu Lukas Prod direkt mit dir fertig klären.
>
> Im Anhang ist der neue öffentliche RSA-Schlüssel. Kannst du bitte prüfen, ob dieser für den vorgesehenen Benutzer bereits hinterlegt ist? Falls nicht, bitte zusätzlich hinterlegen, der bisherige Schlüssel soll erhalten bleiben.
>
> Im bisherigen Stand wurde `UI_marketing` auf `lukaspartner.msu.biz` verwendet. Kannst du mir bitte kurz bestätigen, ob das weiterhin die richtige Kombination ist und welches Verzeichnis wir für einen sicheren Verbindungstest verwenden können?
>
> Wenn möglich, schick mir bitte auch den aktuellen SSH-Host-Key-Fingerabdruck zum Gegenprüfen.
>
> Danke dir!

Danach nicht auf die Antwort warten. Zurück zu DBI-12139.

## Schritt 2 - UMP-Konfiguration selbst prüfen

Auf der tatsächlichen UMP-Instanz brauchst du:

- Host
- Port
- Benutzer
- privaten Schlüsselpfad
- Zielverzeichnis

Der erwartete neue Public-Key-Fingerprint aus dem bisherigen Stand ist:

```text
SHA256:dnNfJ61wYm8HJzH+x+DLgIvcREFPbmZUXdMVjJPvXV8
```

## Schritt 3 - Nur Verbindung testen

Wenn du auf dem UMP-Server bist, teste zuerst nur SSH/SFTP, nicht den kompletten Export.

Beispiel:

```bash
sudo -u www-data ssh -vvv \
  -i /PFAD/ZUM/PRIVATE_KEY \
  -p 22 \
  UI_marketing@lukaspartner.msu.biz
```

oder:

```bash
sudo -u www-data sftp \
  -i /PFAD/ZUM/PRIVATE_KEY \
  -P 22 \
  UI_marketing@lukaspartner.msu.biz
```

Den echten Schlüsselpfad und Port aus der UMP-Konfiguration nehmen.

**Nicht einfach `app:export:mbo` als Verbindungstest starten.** Der Befehl kann echte Auftragsdaten verarbeiten und Dateien übertragen.

## Typische Ergebnisse

| Meldung | Bedeutung |
|---|---|
| Timeout | Netzwerk/DNS/Port prüfen |
| `no matching host key type` | SSH-Host-Key-Algorithmus passt nicht |
| `Permission denied (publickey)` | Benutzer oder Client-Key passt nicht |
| Login klappt, Verzeichnis nicht | Zielpfad/Berechtigung klären |
| Login und Verzeichnis klappen | Erst dann sicheren Anwendungstest abstimmen |

## Nachricht an Alex für den Anwendungstest

Erst wenn die technische Anmeldung klappt:

> Hi Alex, ich habe bei DBI-11149 die Lukas-SFTP-Strecke technisch bis zur Anmeldung geprüft. Bevor ich einen echten UMP-Export starte, möchte ich keinen produktiven Auftrag als Test verwenden. Kannst du mir bitte einen sicheren Testfall bzw. freigegebenen Testauftrag nennen, mit dem wir den vollständigen Ablauf abnehmen können? Danke dir!

## Jira-Kommentar bei technischem Zwischenstand

```text
DBI-11149 Zwischenstand:

- verwendeten Host/Port/User aus der UMP-Konfiguration geprüft
- verwendeten Schlüssel geprüft
- reine SSH/SFTP-Anmeldung getestet
- Ergebnis: <eintragen>

Offen:
<z.B. Public-Key-Hinterlegung bei Lukas / sicherer Anwendungstest>
```

## Fertig, wenn

- [ ] Public Key bei Lukas bestätigt
- [ ] SFTP-Anmeldung aus dem echten UMP-Kontext funktioniert
- [ ] Zielverzeichnis erreichbar
- [ ] sicherer Anwendungstest erfolgreich
- [ ] Ergebnis im Jira dokumentiert
