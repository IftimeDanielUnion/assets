# DBI-6455: Exception Monitoring mit E-Mail-Alarmierung abschließen

| Feld | Wert |
|---|---|
| Arbeitsreihenfolge | **2: Erstes Umsetzungsticket.** Während externer SFTP-Wartezeit bearbeiten. |
| Jira-Status / Priorität im Export | `In progress` / `Prio 2` |
| Zuständig laut Export | Daniel Iftime |
| Planstatus | Offen; keine Schritte dieser Checkliste ausgeführt. |
| Standalone im Viererpaket | **Ja.** SFTP, temporärer SAML-Login und Event Grid sind keine belegten harten Vorgänger. |
| Basiert extern auf | Verfügbaren OTEL-Logs, Azure-/IaC-Zugriff, abgestimmten Alarmkriterien und OSC-Abnahme. |
| Hauptansprechpartner laut Verlauf | Alex Köster / OSC, Marcel Sebbin, Timo Traub; finale Lösungsfreigabe laut Akzeptanzkriterien mit Zeljko. |
| Datenstand | Jira-Export vom 30.09.2026; Code-Snapshot aus dem Upload. |

## Ziel

Fehlermeldungen erreichen über die vorhandene OpenTelemetry-/Azure-Monitor-Anbindung den vereinbarten E-Mail-Verteiler. Die Konfiguration ist in IaC gepflegt und von einem Testereignis bis zur empfangenen E-Mail nachgewiesen.

## Belegter Ausgangsstand

- Der ursprüngliche Sentry-Container-Auftrag wurde durch den Weg über Azure Insights und OpenTelemetry konkretisiert. Am 11.09.2026 bestätigt OSC erfolgreiche Tests der OTEL-Instanzen; offen ist der E-Mail-Versand für `error` und `fatal`. [J1]
- Der Kommentar vom 27.08.2026 nennt `SeverityText = error/fatal`, sofortige oder kurz aggregierte Benachrichtigung an `ui-team@os-cillation.de` und möglichst `ServiceName` im Betreff. Der Betreff ist ein Wunsch, keine im Code nachgewiesene Fähigkeit. [J1]
- Die Quellen nennen unterschiedliche Umfänge: ältere Beschreibung „Nur Testumgebung“, spätere Kommentare Dev, Abnahme und Prod. Die tatsächlich abzunehmenden Umgebungen müssen deshalb bestätigt werden. [J1]
- `catalog` unterstützt E-Mail-Action-Groups und Scheduled Query Alerts. Die Monitoring-Blöcke in `live` enthalten im Snapshot keine `action_groups` oder `scheduled_log_alerts`. Das beweist nicht, dass in Azure keine manuell angelegten Alerts existieren. [C1-C3]
- Der Alert-Scope im Modul ist auf die Application-Insights-Ressource festgelegt. Die drei Umgebungsstacks referenzieren `catalog` über `v0.13.0`; den Funktionsumfang dieser konkreten Version vor einer Nutzung oder Erweiterung prüfen. [C2-C3]

## Externe Voraussetzungen und mögliche Blocker

| Voraussetzung | Benötigt für | Umgang bei Fehlen |
|---|---|---|
| Reale, aktuelle Logdaten mit nachvollziehbarer Herkunft | Korrekte Abfrage und End-to-End-Test | Logaufnahme mit OSC eingrenzen; keine Tabellen oder Feldtypen erfinden. |
| Zugriffe auf Logs, Alerts, Terraform-State und reguläre Pipeline | Zustandsprüfung und kontrollierter Rollout | Berechtigungen gezielt anfordern. |
| Bestätigte Umgebungen, Empfänger und Alarmierungssemantik | Abnahmefähige Konfiguration | Mit OSC und Verantwortlichen festlegen. |
| OSC bestätigt tatsächlichen E-Mail-Eingang | Fachliche Abnahme | Testfall und Ansprechpartner vor dem Rollout vereinbaren. |

## To-dos in Reihenfolge

### MON-01: Ist-Zustand und genaue Alarmkriterien feststellen

**Basiert auf:** keiner anderen Aufgabe. Sofort startbar.

- [ ] Aktuellen Jira-Stand und vorhandene Azure-Alerts, Action Groups, Unterdrückungen und IaC-Verwaltung prüfen; keine Doppelkonfiguration erzeugen.
- [ ] Je vereinbarter Umgebung feststellen, wo die Logs tatsächlich ankommen: Ressource/Workspace, Tabelle, `SeverityText`, `ServiceName`, Zeitstempel und Dienstzuordnung.
- [ ] Mit OSC die einzuschließenden Dienste und Umgebungen bestätigen. Dev nur dann als Pilot verwenden, wenn dort geeignete Daten und ein freigegebener Testweg vorhanden sind.
- [ ] Alarmkriterien festhalten: `error`/`fatal`, Umgang mit Groß-/Kleinschreibung, Aggregationsfenster, erwartete Zustellzeit, Wiederholungen und Verhalten nach Entwarnung.
- [ ] Verteiler `ui-team@os-cillation.de` bestätigen. `ServiceName` im Betreff als Wunsch prüfen; falls nicht passend unterstützt, eine akzeptierte alternative Kennzeichnung vereinbaren.
- [ ] Einen reproduzierbaren serverseitigen Testweg mit OSC festlegen. Eine optionale UI-Teststrecke macht das gesamte Ticket nicht vom SAML-Ticket abhängig.

### MON-02: IaC-Änderung und Abfrage vorbereiten

**Basiert auf:** MON-01.

- [ ] Mit realen Testdaten eine KQL-Abfrage erstellen und positive sowie negative Treffer prüfen. Keine Logtabelle allein aus dem Tickettext ableiten.
- [ ] Prüfen, ob diese Abfrage im aktuell festgelegten Application-Insights-Scope funktioniert. Bei abweichendem notwendigen Scope das Modul gezielt erweitern statt den Fehler zu umgehen.
- [ ] Den tatsächlich referenzierten `catalog`-Stand `v0.13.0` mit der benötigten Schnittstelle vergleichen.
- [ ] Zuerst die vorhandenen Eingaben `action_groups` und `scheduled_log_alerts` in den betroffenen `live`-Stacks nutzen. Nur bei nachgewiesenem Bedarf `catalog` ändern.
- [ ] Bei Moduländerung einen geprüften, versionierten Katalogstand bereitstellen und danach die entsprechenden `live`-Referenzen aktualisieren. Bestehende Tags nicht umschreiben.
- [ ] Erkennbaren Dienst-/Umgebungsbezug, sinnvolle Abfrageintervalle, Aggregation und ein mit OSC abgestimmtes Wiederholungsverhalten konfigurieren.
- [ ] Formatierung, Validierung und Plan über den vorhandenen Projektworkflow prüfen. Unbeabsichtigte Löschungen, Ressourcenneuanlagen und Änderungen außerhalb des Monitoring-Umfangs ausschließen.

### MON-03: Pilot und Ende-zu-Ende-Test

**Basiert auf:** MON-02 und freigegebener Pilotumgebung.

- [ ] Änderung zunächst in der vereinbarten Nicht-Produktivumgebung ausrollen.
- [ ] Action-Group-Test auslösen und die Erreichbarkeit des Verteilers prüfen. Diesen Versandtest ausdrücklich vom vollständigen Log-zu-E-Mail-Test unterscheiden.
- [ ] Kontrollierte `error`- und `fatal`-Testereignisse über den vereinbarten Anwendung-/Collector-Pfad erzeugen und anhand einer eindeutigen Kennung nachverfolgen.
- [ ] Logaufnahme, Abfragetreffer, Alert-Auslösung und tatsächlichen E-Mail-Eingang zusammenhängend nachweisen; Zustellzeit messen.
- [ ] Negative Fälle prüfen: `info`/`warning` dürfen diese Fehlerregel nicht auslösen; keine neuen Fehler dürfen nicht dauerhaft neue Benachrichtigungen erzeugen.
- [ ] Wiederholte Fehler, mehrere Dienste, mögliche Doppelmeldungen und das Verhalten nach Entwarnung mit der vereinbarten Semantik vergleichen.
- [ ] OSC bestätigt Verständlichkeit der Meldung, Dienst-/Umgebungszuordnung und akzeptierte Benachrichtigungshäufigkeit.

### MON-04: Ausrollen, dokumentieren und abnehmen

**Basiert auf:** MON-03.

- [ ] Nach Pilotabnahme in die übrigen ausdrücklich vereinbarten Umgebungen ausrollen; Prod über den regulären Freigabeweg.
- [ ] Je Zielumgebung mindestens ein geeignetes End-to-End-Ergebnis mit OSC dokumentieren.
- [ ] Regelname, Scope, Query, Intervall, Empfänger, Verantwortliche und Vorgehen bei Alarmen im Betriebsablageort dokumentieren.
- [ ] Belege für die ursprünglichen Akzeptanzkriterien ergänzen: Beauftragung, freigegebene finale Lösung, OSC-Implementierung und erfolgreicher Test.
- [ ] Abnahme im Jira festhalten. Ein grüner Terraform-Apply oder eine Action-Group-Testmail allein genügt nicht.

## Repository-Einstieg

| Datei im jeweiligen Repository | Relevanz |
|---|---|
| `catalog/modules/log-and-monitor/main.tf`, Zeilen 34-49 | Action Groups und E-Mail-Empfänger. [C1] |
| `catalog/modules/log-and-monitor/main.alerts.tf`, Zeilen 4-47 | Scheduled Query Alerts; Scope in Zeile 10 auf Application Insights festgelegt. [C2] |
| `catalog/modules/log-and-monitor/variables.logs.tf`, ab Zeile 12 und ab Zeile 85 | Eingabeformate für Action Groups und Log-Alerts. [C1-C2] |
| `catalog/units/log-and-monitor/terragrunt.hcl`, Zeilen 21-35 | Weitergabe der Alert-Eingaben. [C2] |
| `live/live/dev/terragrunt.stack.hcl`, Zeilen 228-244 | Monitoring-Block Dev. [C3] |
| `live/live/test/terragrunt.stack.hcl`, Zeilen 498-514 | Monitoring-Block Test. [C3] |
| `live/live/prod/terragrunt.stack.hcl`, Zeilen 393-409 | Monitoring-Block Prod. [C3] |

**Erwarteter erster PR:** `live`. `catalog` nur bei nachgewiesener Schnittstellenlücke. Die Collector-Konfiguration wird im Jira unter `/etc/otelcol-contrib/config.yaml` genannt; ihren aktuellen Inhalt auf dem Zielsystem prüfen, nicht aus dem Pfad ableiten. [J1]

## Abnahmekriterien

- [ ] `error` und `fatal` führen über die echte Aufnahme- und Alarmkette zu empfangenen E-Mails.
- [ ] Alle vereinbarten Umgebungen und Dienste sind abgedeckt.
- [ ] Dienst und Umgebung sind eindeutig erkennbar; Betreffwunsch oder akzeptierte Alternative ist dokumentiert.
- [ ] Negative Tests und Wiederholungsverhalten entsprechen der Vereinbarung.
- [ ] IaC, Testbelege, Betriebsdokumentation und OSC-Abnahme liegen vor.

## Nicht im Umfang / Rückfall

Kein neuer Sentry-Aufbau und keine pauschale Neuimplementierung von OTEL. Zusätzliche Benachrichtigungsdienste erst nach tatsächlichem Bedarf und Freigabe.

Bei Fehlalarmserie die neue Regel gezielt deaktivieren oder auf den vorherigen geprüften Stand zurücksetzen. Logaufnahme und bestehende Überwachung nicht abschalten. Den bekannten Rückfallweg vor dem Prod-Rollout dokumentieren.

## Quellen

- **[J1]** `ump_jiras.zip` → `ump_jiras/DBI-6455.doc`: Beschreibung, Akzeptanzkriterien sowie Kommentare vom 25./27./31.08., 10./11./17.09. und 28.09.2026.
- **[C1]** `UMP_repos.zip` → `UMP_repos/UMP_legacy/catalog/`: Action-Group-Ressource und Eingabevariablen in den oben genannten Dateien.
- **[C2]** Gleiches Repository: Alert-Ressource, Variablen und Terragrunt-Unit in den oben genannten Dateien.
- **[C3]** `UMP_repos.zip` → `UMP_repos/UMP_legacy/live/`: die drei oben genannten Stacks; Katalogversion jeweils in Zeilen 8-11.

[Zur Übersicht](README.md)
