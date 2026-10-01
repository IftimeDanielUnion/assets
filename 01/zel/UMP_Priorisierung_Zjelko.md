# UMP Tickets – Priorisierung & Nachricht an Zeljko

## Mein Vorschlag zur Reihenfolge

1. **DBI-11149 – SFTP**
   - Wegen hoher Priorität direkt anstoßen.
   - Benötigte externe Rückmeldung / Key-Hinterlegung anfragen.
   - Währenddessen nicht warten, sondern mit DBI-12139 weitermachen.

2. **DBI-12139 – Blob Storage für Landingpage Assets**
   - Als erstes aktiv bearbeiten.
   - Ist-Stand in Azure prüfen.
   - IaC mit Azure vergleichen.
   - Fehlende oder abweichende Konfiguration korrigieren.
   - Mit Test-Asset prüfen, ob der öffentliche Zugriff wie gefordert funktioniert.

3. **DBI-6455 – Exception Monitoring**
   - Danach umsetzen.
   - Bestehendes Monitoring prüfen.
   - E-Mail-Alarmierung für `error` / `fatal` ergänzen und testen.

4. **PB2B-27849 – Temporärer SAML Login**
   - Danach prüfen.
   - Vorziehen, falls dadurch aktuell Migrationstests blockiert sind.

5. **DBI-7161 – Event Grid**
   - Zuletzt angehen.
   - Vor Umsetzung erst offene Architektur-, Authentifizierungs- und Schnittstellenfragen klären.

## Kurz gesagt

```text
DBI-11149 anstoßen
        ↓
DBI-12139 bearbeiten
        ↓
DBI-6455
        ↓
PB2B-27849
        ↓
DBI-7161
```

## Nachricht an Zeljko

> Hi Zeljko, ich hab mir die Tickets und den aktuellen Code-Stand etwas genauer angeschaut.  
> Ich würde DBI-11149 wegen der hohen Prio direkt anstoßen und während ich dort auf Rückmeldung warte DBI-12139 abarbeiten. Das Blob-Storage-Ticket wirkt aktuell recht überschaubar, da die grundlegenden Storage-Bausteine für dev, test und prod schon im IaC vorhanden sind. Danach würde ich DBI-6455 angehen. PB2B-27849 und DBI-7161 würde ich anschließend bearbeiten, sofern davon aktuell nichts blockiert.  
> Passt die Reihenfolge für dich oder soll ich etwas davon vorziehen?

## Falls Zeljko DBI-12139 direkt bestätigt

Dann antworte:

> Super, dann nehme ich mir DBI-12139 direkt vor. Ich prüfe zuerst den aktuellen Stand in Azure gegen das IaC und korrigiere nur die Punkte, die tatsächlich noch fehlen. Danach teste ich den Zugriff mit einem Asset und gebe dir ein kurzes Update.
