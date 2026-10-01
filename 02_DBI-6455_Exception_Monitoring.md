# 2. DBI-6455: Bei Anwendungsfehlern eine E-Mail verschicken

**Priorität:** dein erstes Umsetzungsticket, während SFTP-Rückmeldungen ausstehen. **Standalone:** ja, du musst auf keines der anderen drei Tickets warten.

**Du brauchst Hilfe von:** Alex Köster für die gewünschten Meldungen und den Anwendungstest; Marcel Sebbin für den vorhandenen Azure-Aufbau. Zeljko Kovacevic wird in den Abnahmekriterien für die Lösungsfreigabe genannt. [1]

## Worum geht es?

Die Anwendung schreibt Fehler in ihre Protokolle. Diese Daten sollen in Azure ausgewertet werden. Bei bestimmten Fehlern soll das OSC-Team automatisch eine E-Mail bekommen, statt selbst nachsehen zu müssen.

Am 11.09.2026 hat Alex gemeldet, dass die OpenTelemetry-Anbindung bereits erfolgreich getestet wurde. Offen war die E-Mail-Benachrichtigung für `error` und `fatal`. OpenTelemetry ist hier der bereits eingerichtete Weg, über den die Anwendungsdaten nach Azure gelangen. **Du sollst diesen Weg vervollständigen, nicht Sentry neu aufsetzen.** [1]

## Schritt 1: Zwei Dinge klären, bevor du eine Regel baust

Du brauchst die Anforderungen von Alex und den aktuellen Azure-Stand von Marcel. Die beiden Anfragen kannst du parallel schicken.

### Nachricht A: An Alex Köster

**Kanal:** Teams oder Jira-Kommentar mit Markierung von Alex.

```text
Hallo Alex,

ich möchte die noch offene E-Mail-Benachrichtigung aus DBI-6455 fertigstellen. Mein Stand aus deinem Kommentar vom 11.09. ist, dass die Fehlerdaten bereits erfolgreich nach Azure übertragen werden und nur noch die Benachrichtigung fehlt.

Passt weiterhin: E-Mails bei error und fatal an ui-team@os-cillation.de, möglichst mit dem betroffenen Dienst im Betreff?

Kannst du mir bitte noch bestätigen, welche Anwendungen und Umgebungen dazugehören und welche Verzögerung bis zur E-Mail für dich in Ordnung wäre? Für den späteren Test wäre außerdem hilfreich, wie wir einen harmlosen Testfehler gezielt auslösen können.

Danke dir!
```

### Nachricht B: An Marcel Sebbin

**Kanal:** Teams oder Jira-Kommentar mit Markierung von Marcel.

```text
Hallo Marcel,

ich schaue mir die fehlenden E-Mail-Alarme in DBI-6455 an und möchte auf dem vorhandenen Azure-Aufbau aufsetzen.

Kannst du mir bitte die aktuell verwendeten Log-Ressourcen und die zugehörige Terraform-Konfiguration nennen? Gibt es bereits Alarmregeln oder eine laufende Änderung von dir, Timo oder Christoph, die ich berücksichtigen sollte?

Mir hilft auch kurz die Info, welche Testumgebung ich für die erste Umsetzung verwenden soll und über welchen Weg die Änderung ausgerollt wird. Ich möchte nichts doppelt anlegen oder bestehende Einstellungen überschreiben.

Danke dir!
```

**Deine Aufgaben nach den Antworten:**

- [ ] Im Jira festhalten, welche Anwendungen und Umgebungen überwacht werden sollen.
- [ ] Empfänger und gewünschte Reaktionszeit notieren.
- [ ] In Azure prüfen, ob die betreffenden Fehlerdaten tatsächlich sichtbar sind und ob schon passende Regeln existieren.
- [ ] Fehlende Zugriffe gezielt über Marcel beziehungsweise den von ihm genannten Verantwortlichen klären.

**Wenn noch keine passenden Daten ankommen:** Erst diesen Punkt mit Alex und Marcel lösen. Eine E-Mail-Regel kann fehlende Anwendungsdaten nicht ersetzen. Du musst deshalb aber nicht auf das SFTP-, SAML- oder Event-Grid-Ticket warten.

## Schritt 2: Die vorhandene Überwachung ergänzen

Jetzt beginnt dein eigener Umsetzungsteil.

- [ ] Eine Logabfrage vorbereiten, die die vereinbarten `error`- und `fatal`-Meldungen findet. Mit vorhandenen Beispielen prüfen, ob sie den richtigen Dienst und die richtige Umgebung erfasst.
- [ ] Darauf eine E-Mail-Regel mit dem bestätigten Empfänger aufbauen. Vorhandene Alarmregeln berücksichtigen, damit keine doppelten E-Mails entstehen.
- [ ] Änderungen im bestehenden Terraform-/Terragrunt-Code pflegen und den Plan prüfen. Zunächst nur die vereinbarte Testumgebung ändern.
- [ ] Prüfen, ob der Dienstname im Betreff mit dem gewählten Weg möglich ist. Falls nicht, mit Alex eine andere eindeutige Darstellung in der Nachricht abstimmen.

**Wo du anfängst:** Im Repository `live` wird festgelegt, welche Einstellungen je Umgebung gelten. `catalog` enthält bereits Bausteine für E-Mail-Empfänger und Log-Alarmregeln. Im hochgeladenen `live`-Stand sind diese Alarm-Eingaben in den Monitoring-Blöcken noch nicht befüllt. Das ist ein Ansatzpunkt, aber kein Beweis, dass in Azure bisher nichts manuell eingerichtet wurde. [2]

**Wichtig beim Prüfen:** Die vorhandene Alarmregel im `catalog` sucht innerhalb von Application Insights. Kontrolliere, ob deine gewünschten Daten dort erreichbar sind. Übernimm nicht einfach eine Abfrage aus einer anderen Log-Ressource. [2]

### Nachricht C: Nur bei ungeklärter Lösungsfreigabe an Zeljko Kovacevic

**Wann:** vor dem betreffenden Rollout, falls die Freigabe nicht bereits dokumentiert ist. Nicht erneut fragen, wenn die Antwort schon im aktuellen Ticket steht.

```text
Hallo Zeljko,

für DBI-6455 möchte ich die vorhandene Azure-Überwachung um die noch fehlenden E-Mail-Alarme ergänzen. Ein neuer Sentry-Aufbau ist dafür nach meinem Verständnis nicht mehr vorgesehen.

In den Abnahmekriterien ist die Freigabe der finalen Lösung mit dir genannt. Ist dieser Weg bereits freigegeben und dokumentiert? Falls nicht, würde ich die abgestimmte Konfiguration vor dem Rollout kurz mit dir durchgehen.

Danke dir!
```

## Schritt 3: Nicht nur eine Testmail, sondern die ganze Strecke prüfen

Die entscheidende Strecke lautet: **Die Anwendung meldet einen Fehler. Azure erkennt ihn. Die E-Mail kommt tatsächlich beim Team an.**

### Nachricht D: An Alex Köster zum gemeinsamen Test

**Nur senden, wenn die Regel in der vereinbarten Testumgebung eingerichtet und zum Test freigegeben ist.**

```text
Hallo Alex,

die E-Mail-Regel ist in der vereinbarten Testumgebung eingerichtet. Ich würde jetzt gern gemeinsam die vollständige Strecke testen, also vom Anwendungsfehler bis zur tatsächlich empfangenen E-Mail.

Kannst du den abgestimmten Testfehler auslösen oder den Test mit mir begleiten? Ich prüfe parallel, ob der Fehler in Azure ankommt und die Regel reagiert.

Gib mir danach bitte kurz Rückmeldung, ob die E-Mail beim Verteiler angekommen ist, Anwendung und Umgebung erkennbar sind und die Benachrichtigung für dich so passt.

Danke dir!
```

- [ ] Den vereinbarten Fehler kontrolliert auslösen und in Azure wiederfinden.
- [ ] Auslösung der Regel und tatsächlichen E-Mail-Eingang nachweisen.
- [ ] Beide vereinbarten Fehlerstufen prüfen. Normale Informationsmeldungen dürfen diese Fehlerregel nicht auslösen.
- [ ] Prüfen, ob wiederholte Fehler zu einer akzeptablen Anzahl an E-Mails führen.
- [ ] Zeitpunkt, Umgebung, Ergebnis und Alex' Rückmeldung im Jira festhalten.

**Wenn keine E-Mail kommt:** Nacheinander prüfen, ob der Fehler ankam, die Abfrage ihn gefunden hat, die Regel reagierte und der Versand funktionierte. Nicht die gesamte Überwachung auf Verdacht neu aufbauen.

## Schritt 4: Weitere vereinbarte Umgebungen und Abschluss

- [ ] Nach erfolgreichem Test über den üblichen Freigabeweg auf die weiteren ausdrücklich vereinbarten Umgebungen übertragen.
- [ ] Auch dort einen passenden Testnachweis führen. Ein erfolgreicher Test in Dev beweist nicht, dass Prod richtig eingestellt ist.
- [ ] Im Jira dokumentieren, was überwacht wird, wer die E-Mails erhält und wo der Code liegt.
- [ ] Die Bestätigung von OSC und gegebenenfalls noch offene Freigaben ergänzen.

Bei einer Serie unerwünschter Alarme nimmst du gezielt die neue Regel zurück. Die bestehende Logübertragung bleibt an.

## Fertig, wenn …

- [ ] Die vereinbarten Anwendungsfehler tatsächlich zu empfangenen E-Mails führen.
- [ ] Alle vereinbarten Anwendungen und Umgebungen berücksichtigt sind.
- [ ] Die Konfiguration im Code und die Testergebnisse im Jira liegen.
- [ ] OSC die Meldungen akzeptiert und die erforderliche Freigabe dokumentiert ist.

## Nur zum Nachschlagen

`live/live/dev/terragrunt.stack.hcl` ist der Einstieg für Dev; entsprechende Dateien gibt es unter `test` und `prod`. Die vorhandenen Bausteine stehen unter `catalog/modules/log-and-monitor/`, insbesondere in `main.tf` und `main.alerts.tf`. [2]

**Quellen:** [1] `ump_jiras.zip` → `ump_jiras/DBI-6455.doc`, Abnahmekriterien sowie Kommentare vom 25./27./31.08. und 10./11./17.09.2026. Ältere Beschreibung und spätere Kommentare nennen unterschiedliche Umgebungen; deshalb ist der Umfang zu bestätigen. [2] `UMP_repos.zip` → `UMP_repos/UMP_legacy/catalog/modules/log-and-monitor/`, `catalog/units/log-and-monitor/terragrunt.hcl` und die drei genannten `live`-Stacks.

[Zurück zur Übersicht](README.md)
