# 2. DBI-6455: Fehlermails auf der vorhandenen Lösung einrichten

**Priorität:** dein erstes eigenes Umsetzungsticket, während die SFTP-Anfrage läuft. **Standalone:** ja.

## Worum geht es?

Die Fehlerdaten werden laut Jira bereits über OpenTelemetry übertragen. Offen ist die Benachrichtigung: Bei `error` oder `fatal` soll eine E-Mail an `ui-team@os-cillation.de` gehen. Ein erkennbarer Dienstname ist erwünscht; im Betreff ausdrücklich nur „wenn möglich“. Diese Anforderungen liegen schon vor. Du musst sie nicht neu bei Timo oder Marcel erfragen. [J2]

## 1. Den vorhandenen Stand selbst prüfen

- [ ] In der richtigen TEST-Subscription die Monitoring-Ressourcengruppe öffnen. Im Upload heißen die Ressourcen `rg-monitoring-test`, `log-ump-test` und `log-ump-test-appi`. Namen und Umgebung im Tenant gegenprüfen. [B1]
- [ ] Prüfen, ob es bereits eine passende Alarmregel und E-Mail-Empfängergruppe gibt. Der Code enthält keine, aber eine manuelle Azure-Konfiguration ist damit nicht ausgeschlossen.
- [ ] In **Application Insights `log-ump-test-appi` → Logs** die Abfrage aus `code/01_logeingang_pruefen.kql` ausführen. Sie zeigt nur Dienstnamen, Fehlerstufen, Zeitpunkte, Ressourcenreferenzen und Anzahlen.
- [ ] Prüfen, ob die erwarteten Dienste aktuelle Daten liefern. Der Code überträgt `service.name` und die Fehlerstufe; die Sammlung ist auf OpenTelemetry-Logs ausgelegt. [B2, B5, W1]

**Wenn Daten fehlen:** Nicht sofort die gesamte Logging-Lösung ersetzen. Zuerst den Collector-Dienst und seine Fehler auf dem betroffenen Server prüfen. Laut Jira liegt seine Konfiguration unter `/etc/otelcol-contrib/config.yaml`. Nur die relevanten Endpunkte und Fehler ansehen, keine komplette Konfigurationsdatei mit möglichen Zugangsdaten ins Jira kopieren. [J2]

**Wenn die Abfrage nur im Workspace, nicht im Application-Insights-Bereich funktioniert:** Das ist ein konkreter Scope-Befund. Der vorhandene Baustein setzt die Alarmregel auf Application Insights. Dann ist der beigefügte Patch allein noch nicht ausreichend. Diesen Befund erst lösen, bevor du die Regel aktivierst. [B2]

## 2. Alex informieren, nicht die Lösung an ihn zurückgeben

**An:** Alex Köster. **Kanal:** Teams oder Jira. **Jetzt senden**, bevor eine Regel aktiviert wird.

```text
Hallo Alex,

ich übernehme bei DBI-6455 die noch fehlende E-Mail-Alarmierung. Deine Anforderungen sind im Ticket bereits klar: error/fatal an ui-team@os-cillation.de. Darauf setze ich direkt auf, ohne die bestehende OpenTelemetry-Anbindung neu aufzubauen.

Ich prüfe zuerst den Logeingang und ergänze dann die vorhandene Azure-Konfiguration in TEST. Als Startpunkt plane ich eine Prüfung pro Minute mit einem Fünf-Minuten-Fenster, getrennt nach Dienst. Die tatsächliche Mail-Laufzeit und mögliche Wiederholungen prüfen wir im Test; das ist keine Zusage für eine Mail pro Fehler oder pro Minute.

Kannst du anschließend mit mir je eine harmlose error- und fatal-Testmeldung aus der Anwendung auslösen und den Eingang am Verteiler bestätigen? Der Dienstname soll zunächst im Alarmkontext sichtbar sein. Den gewünschten Betreff behandeln wir zusätzlich, ohne daran den ersten Mail-Test aufzuhalten.

Ich bereite die Azure-Seite vor und melde mich, sobald wir testen können. Danke dir!
```

## 3. Den vorbereiteten TEST-Patch übernehmen

Unter `code/monitoring-test.patch` liegt eine konkrete Änderung für das Repository **`live`**. Sie ergänzt den bereits vorhandenen Monitoring-Block. **Kein neues Monitoring-System, keine neuen Anwendungspakete, kein neuer Katalog-Baustein.** Der benötigte Baustein ist auch im referenzierten Katalog-Tag `v0.13.0` vorhanden. [B1, B2]

- [ ] Den aktuellen `live`-Stand mit dem Upload vergleichen. Der Patch darf nicht vorhandene neuere Konfiguration verdrängen.
- [ ] Den Patch in einem eigenen Branch prüfen und übernehmen. Die genaue Bedienung steht in `code/README.md`.
- [ ] Den Infrastrukturplan für die **Monitoring-Unit in TEST** erzeugen und lesen, nicht ungeprüft den ganzen Umgebungs-Stack ausrollen.
- [ ] Erwartet sind eine E-Mail-Empfängergruppe und eine Alarmregel. Unerwartete Löschungen, Datenbank- oder Gateway-Änderungen nicht mit ausrollen.
- [ ] Den üblichen Merge-/Deployment-Prozess und einen verfügbaren berechtigten Reviewer nutzen. Die Freigabe ist nicht an einen der drei Urlauber gebunden.

Die Regel zählt passende Logzeilen und unterscheidet nach `ServiceName`. Ein fehlender Dienstname wird sichtbar als `unknown-service`, nicht einfach herausgefiltert. Empfänger und Prüfintervall sind fertig eingetragen. [B2, W1, W2]

**Zum Betreff:** Der mitgelieferte Patch setzt keinen dynamischen Betreff. Azure unterstützt diese Funktion grundsätzlich, aber der vorhandene Terraform-Baustein bildet das passende Feld nicht ab. Der Dienstname in einer Alarm-Dimension ändert den Betreff nicht automatisch. Erst den geforderten Mailversand nachweisen, dann den Betreff als klar benannte Ergänzung behandeln oder die akzeptierte Ausführung im Jira festhalten. [W3]

## 4. Die ganze Strecke testen

- [ ] Einen Test der E-Mail-Empfängergruppe ausführen. Das prüft den Mailweg, noch nicht die Anwendung.
- [ ] Danach mit Alex eine eindeutig erkennbare, harmlose `error`-Meldung aus der tatsächlichen TEST-Anwendung erzeugen.
- [ ] Nachweisen: Meldung in den Logs → ausgelöster Alarm → empfangene Mail am Verteiler.
- [ ] Den Test mit `fatal` wiederholen, ohne absichtlich den Produktivbetrieb zu beschädigen.
- [ ] Eine Kontrollmeldung mit `info` oder `warning` darf von dieser Regel nicht erfasst werden.
- [ ] Wiederholungen und Laufzeit notieren. Bei überlappenden Fenstern und zustandslosen Regeln können weitere Benachrichtigungen entstehen. Keine „genau einmal“-Zustellung versprechen. [W2]

Im Code gibt es bereits eine Unterdrückung doppelter Fehler. Für den Test deshalb eindeutig unterschiedliche Testkennungen verwenden. Nicht aus einer ausbleibenden identischen zweiten Meldung automatisch einen Fehler der Alarmregel ableiten. [B5]

### Nach bereitgestellter TEST-Regel: an Alex

```text
Hallo Alex,

die Alarmregel in TEST ist bereit. Können wir jetzt den vereinbarten Test mit einer klar erkennbaren error-Meldung und danach einer fatal-Meldung durchführen?

Ich prüfe parallel den Eingang in Azure und die Alarmauslösung. Von dir brauche ich die Bestätigung, ob die jeweilige Mail am Verteiler angekommen ist und ob der betroffene Dienst erkennbar ist. Bitte verwende unterschiedliche Testkennungen, damit die vorhandene Unterdrückung doppelter Fehler den Vergleich nicht verfälscht.

Danach halte ich die Ergebnisse und den offenen beziehungsweise akzeptierten Betreff im Ticket fest. Danke dir!
```

## 5. PROD separat abschließen

- [ ] Erst nach dem erfolgreichen TEST-Nachweis den PROD-Plan vorbereiten.
- [ ] `code/monitoring-prod.patch` legt die PROD-Regel zunächst **deaktiviert** an. Dadurch verschickt sie nicht schon bei der Vorbereitung produktive Fehlermails.
- [ ] Den PROD-Logeingang ebenfalls prüfen; TEST beweist ihn nicht.
- [ ] Die Aktivierung mit dem verfügbaren Betrieb/Reviewer abstimmen, `enabled` in PROD bewusst auf `true` ändern und über den regulären Prozess ausrollen.
- [ ] Den End-to-End-Nachweis in PROD mit einem freigegebenen Testereignis wiederholen.

Der DEV-Patch ist eine Option für zusätzliche Tests, **kein Pflicht-Vorgänger** von TEST. Fehlende Rechte auf Azure-Ressourcen oder eine geschützte Pipeline musst du über den bestehenden Team-/Serviceweg lösen. Dafür brauchst du keine private Rückmeldung eines Urlaubers.

## Fertig, wenn …

- [ ] Die im aktuellen Ticketumfang geforderten Umgebungen senden nachweislich Fehlermails.
- [ ] `error` und `fatal` werden erfasst, `info`/`warning` nicht.
- [ ] Empfänger, Dienstzuordnung, Wiederholungsverhalten und der Stand des Betreffwunschs sind dokumentiert.
- [ ] Der Code ist versioniert und die Testbestätigung liegt im Jira.

Quellen: **J2, B1, B2, B5, W1 bis W3** im [Prüfprotokoll](06_Codebefunde_und_Pruefprotokoll.md). [Zurück](README.md)
