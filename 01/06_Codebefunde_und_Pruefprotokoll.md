# Codebefunde, Quellen und tatsächliche Prüfungen

**Stand: 01.10.2026.** Dieses Paket ersetzt die bisherigen Arbeitspläne. Die Abwesenheit von Timo Traub, Marcel Sebbin und Christoph Sievers stammt aus deiner aktuellen Nachricht. Andere Personen wurden nicht kontaktiert; ihre aktuelle Verfügbarkeit ist nicht bestätigt.

## Die Befunde, auf denen der neue Plan beruht

**B1: Monitoring ist schon Bestandteil der Umgebungen.** DEV, TEST und PROD referenzieren den Infrastrukturkatalog `v0.13.0`. In ihren Monitoring-Blöcken fehlen im Upload die Eingaben für E-Mail-Gruppen und Log-Alarmregeln. Das belegt eine Lücke im Snapshot, nicht automatisch im Azure-Tenant. Der Patch ergänzt genau diese Eingaben.

**B2: Dafür gibt es bereits einen passenden Modulvertrag.** Der Katalog unterstützt E-Mail-Empfängergruppen, zeitgesteuerte Log-Abfragen und Dimensionen. Die vier relevanten Modul-/Unit-Dateien wurden zusätzlich mit dem tatsächlich im Upload enthaltenen Git-Tag `v0.13.0` verglichen. Sie stimmen nach Normalisierung der Zeilenenden überein. Die Alarmregel ist im Modul auf Application Insights begrenzt. Die OTEL-DCR ist separat mit Workspace und Application-Insights-Referenz konfiguriert. Daraus folgt der Prüfweg: erst Abfrage im Ziel-Scope, dann Bereitstellung. Kein neuer Katalog-Release ist für den mitgelieferten Patch erforderlich.

**B3: Lukas verwendet im MBO-Export den alten Shell-Connector.** Der Export lädt Host, Port, Benutzer, Schlüsselpfad und Zielverzeichnis über konfigurierte Parameter und ruft `Comm\File\sftp_connector` auf. Dieser startet `sftp` über `proc_open`. Der Exportbefehl erzeugt XML-Dateien, aktualisiert Bestellinformationen und überträgt Dateien; beim Versand wird auch `.done` angehängt. Deshalb sind OpenSSH-Diagnose und vollständiger Anwendungstest getrennte Schritte. Ein durchlaufender Diagnosetest beweist noch keinen fehlerfreien Export.

**B4: SAML ist über Ansible nachvollziehbar, aber der temporäre Laufzeitstand fehlt.** `host_vars/ump-app` benennt die Mellon-Dateien und deren stageabhängigen Upload. Das PROD-Inventar verwendet `mellon_file_stage: prod`; die enthaltene IdP-Datei referenziert die reguläre Policy ohne `_tmp`. Die beiden öffentlichen Metadatendateien wurden lokal als XML eingelesen. Es wurde weder ihre Signatur kryptografisch verifiziert noch ein Login getestet. Die aktuelle temporäre Zuordnung muss Oliver beziehungsweise das laufende System bestätigen.

**B5: Die Anwendung überträgt bereits passende Logmerkmale.** `OpenTelemetryService` setzt `service.name`; `SentryEventConverter` setzt die Fehlerstufe über `setSeverityText`. Der vorhandene `BeforeSendHandler` enthält eine Prüfung auf doppelte Meldungen. Das stützt die Auswahl der Logfelder und die Empfehlung für unterschiedliche Testkennungen. Es beweist keinen heutigen Ende-zu-Ende-Datenfluss.

**B6: Event Grid ist im geprüften Upload nicht als fertige Integration nachgewiesen.** In den untersuchten Verzeichnissen `ui-producthub/app`, `ui-producthub/routes`, `ui-producthub/config`, `unioninvestment/htdocs/src`, `unioninvestment/etc/config` und `catalog/modules` wurden passende Event-Grid-, CloudEvent- und Subscription-Validation-Einstiegspunkte gesucht. Der gefundene Slack-Webhook gehört nur zum Logging. Dieses Suchergebnis beweist nicht, dass eine Implementierung in einem anderen Branch, unter anderen Namen oder außerhalb des Uploads fehlt. Das Konzept-PDF, die Zeichnung und das verlinkte externe Event-Hub-Repository sind nicht als entsprechende Dateien im gelieferten Paket enthalten.

## Nachprüfbare Code-Einstiegspunkte

Alle Pfade beginnen unter `UMP_repos/UMP_legacy/` im ZIP. Die folgenden Zeilennummern beziehen sich auf genau diesen Upload, nicht auf einen späteren Branch.

| Befund | Originalpfad im Repository-Upload | Einstieg |
|---|---|---|
| B1 | `live/live/test/terragrunt.stack.hcl` | `unit "monitoring"`: Zeile 498; `tg_catalog_version`: Zeile 11, 95, 139, 162, 194 |
| B1 | `live/live/prod/terragrunt.stack.hcl` | `unit "monitoring"`: Zeile 393 |
| B1 | `live/live/dev/terragrunt.stack.hcl` | `unit "monitoring"`: Zeile 228 |
| B1 | `live/live/root.hcl` | `required_version`: Zeile 24; `version = "~>4.63"`: Zeile 32 |
| B2 | `catalog/modules/log-and-monitor/main.tf` | `azurerm_monitor_action_group`: Zeile 34 |
| B2 | `catalog/modules/log-and-monitor/main.alerts.tf` | `scopes`: Zeile 10, 79; `azurerm_monitor_scheduled_query_rules_alert_v2`: Zeile 4 |
| B2 | `catalog/modules/log-and-monitor/variables.logs.tf` | `variable "action_groups"`: Zeile 12; `variable "scheduled_log_alerts"`: Zeile 85 |
| B2 | `catalog/units/log-and-monitor/terragrunt.hcl` | `action_groups`: Zeile 27; `scheduled_log_alerts`: Zeile 34 |
| B2 | `catalog/modules/log-collector-open-telemetry/constants.tf` | `Microsoft-OTel-Logs`: Zeile 30, 61, 132; `replaceResourceIdWithReference`: Zeile 55, 96, 130, 163 |
| B3 | `unioninvestment/etc/config/packages/global.yml` | `mboExportServerHost`: Zeile 20; `mboExportServerRemoteDir`: Zeile 24 |
| B3 | `unioninvestment/htdocs/application_includes/W2P/Export/mbo_export_controller.php` | `mboExportServerAuthKeyPath`: Zeile 180; `function copy_export_files`: Zeile 580; `function connect_and_check`: Zeile 674; `$filename . '.done'`: Zeile 616 |
| B3 | `unioninvestment/htdocs/includes/Comm/File/class.sftp_connector.php` | `$sftp_cmd =`: Zeile 223; `proc_open`: Zeile 224 |
| B3 | `unioninvestment/htdocs/src/Console/Command/Export/MboExportCommand.php` | `name: 'app:export:mbo'`: Zeile 27; `create_export_files`: Zeile 130; `updateExportedBasketOrders`: Zeile 132; `copy_export_files`: Zeile 135 |
| B3 | `ansible-ump/playbooks/configuration/ssh-client.yml` | `osc.system.sshclient`: Zeile 15 |
| B3 | `ansible-ump/host_vars/ump-app` | `ssh_client_config:`: Zeile 843 |
| B4 | `ansible-ump/host_vars/ump-app` | `MellonIdPMetadataFile`: Zeile 230; `MellonSPMetadataFile`: Zeile 231; `files/app/mellon/{{ mellon_file_stage }}/idp_metadata_azure.xml`: Zeile 740 |
| B4 | `ansible-ump/inventories/prod.yml` | `host_app_fqdn`: Zeile 46; `mellon_file_stage`: Zeile 55 |
| B5 | `unioninvestment/htdocs/src/Service/OpenTelemetry/OpenTelemetryService.php` | `SERVICE_NAME`: Zeile 30 |
| B5 | `unioninvestment/htdocs/src/Service/OpenTelemetry/SentryEventConverter.php` | `setSeverityText`: Zeile 27 |
| B5 | `unioninvestment/htdocs/src/Service/Sentry/BeforeSendHandler.php` | `canSendDuplicate`: Zeile 102, 122 |

Zusätzlich für B4: `ansible-ump/playbooks/configuration/files/app/mellon/prod/idp_metadata_azure.xml` und `ump-app_metadata.xml`. Aus Sicherheitsgründen werden weder ganze Zertifikatsblöcke noch private Schlüssel in diesem Paket dupliziert.

## Jira-Quellen

**J1: `ump_jiras/DBI-11149.doc`.** Anforderungen und Zusatzinformation zu MOVEit; Firewall-Change `UI-CHG-0094446`; Kontakt Marc Hintz mit `hintz@msu.biz`; Kommentare vom 14./15.09. zum SSH-/Schlüsselproblem; öffentlicher RSA-Schlüssel von Alex am 17.09.; Auftrag zur zusätzlichen Hinterlegung am 21.09.; Prio-1-Kommentar vom 29.09.; Hinweis vom 20.08. auf Norman und die weitere Wertanlage-Nutzung. Die vorherige Rückgabe an Christoph wurde im neuen Plan aufgrund deiner Abwesenheitsinformation ausdrücklich nicht als Arbeitsschritt übernommen.

**J2: `ump_jiras/DBI-6455.doc`.** Alex' Anforderungen vom 27.08. nennen `SeverityText` `error`/`fatal`, `ui-team@os-cillation.de` und den Dienstnamen im Betreff „wenn möglich“. Die Collector-Datei ist im Kommentar vom 31.08. genannt. Am 11.09. bestätigt Alex die getesteten OpenTelemetry-Instanzen und den fehlenden E-Mail-Versand. Aus diesen Kommentaren folgt keine neue Sentry-Installation.

**J3: `ump_jiras/PB2B-27849.doc`.** Oliver ist im Export Bearbeiter. Sein Kommentar vom 22.09. bestätigt die temporäre App und nennt die Metadaten der Policy `B2C_1A_signin_saml_ump_tmp`.

**J4: `ump_jiras/DBI-7161.doc`.** Event Grid, Fehlerablage, externer ProductHub, B2C-Audiences/Scopes, Sinan als IT-Kontakt, Maike als Fachkontakt, zwei Konzeptanhänge und der externe Event-Hub-Codeverweis vom 19.08. Die Entra-/RBAC-Variante im Arbeitsplan ist ein offengelegter technischer Vorschlag, keine aus dem Ticket erfundene Freigabe.

## Geprüfte öffentliche Primärquellen

Die folgenden Quellen wurden für die technischen Empfehlungen am 01.10.2026 herangezogen. Sie ersetzen nicht eure interne Freigabe oder die Prüfung eurer Umgebung.

**W1: Microsoft Learn, Azure Monitor Logs reference: OTelLogs.** Tabellenname, `SeverityText`, `ServiceName` und Zeit-/Ressourcenfelder.

```text
https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/otellogs
```

**W2: Microsoft Learn zur Erstellung von Log-Alarmregeln und HashiCorp-Dokumentation der im Projekt passenden Provider-Version 4.63.0.** Aggregation, Dimensionen, Voraussetzungen für einminütige Auswertung und Default für `auto_mitigation_enabled`. Der Patch verspricht keine exakt einmalige Benachrichtigung und keine feste E-Mail-Laufzeit.

```text
https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-create-log-alert-rule
https://raw.githubusercontent.com/hashicorp/terraform-provider-azurerm/v4.63.0/website/docs/r/monitor_scheduled_query_rules_alert_v2.html.markdown
```

**W3: Microsoft Learn, Customize log search alert email subjects.** Dynamische Betreffzeilen sind grundsätzlich möglich; sie verwenden eine eigene `actionProperties`-Eigenschaft `Email.Subject` und einen geeigneten API-Stand. Das ist nicht identisch mit gewöhnlichen Payload-Zusatzfeldern.

```text
https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-customize-email-subject-how-to
```

**W4: Microsoft Learn, Authenticate Event Grid publishing clients using Microsoft Entra ID.** Nativer Publishing-Zugriff und gezielte Azure-RBAC-Rollen.

```text
https://learn.microsoft.com/en-us/azure/event-grid/authenticate-with-microsoft-entra-id
```

**W5: Microsoft Learn zur gesicherten Webhook-Zustellung und zu den verschiedenen Zustell-Authentifizierungen.** Publishing, Webhook-Authentifizierung und Ablageberechtigung werden nicht als derselbe Mechanismus behandelt.

```text
https://learn.microsoft.com/en-us/azure/event-grid/secure-webhook-delivery
https://learn.microsoft.com/en-us/azure/event-grid/security-authentication
```

**W6: Microsoft Learn zu Dead-Letter-Ziel, Wiederholungen und Zustellverhalten.** Die Event Subscription braucht ein tatsächlich konfiguriertes Fehlerziel und den passenden Schreibzugriff. Wiederholungs-/Ablaufbedingungen werden mitgetestet.

```text
https://learn.microsoft.com/en-us/azure/event-grid/manage-event-delivery
https://learn.microsoft.com/en-us/azure/event-grid/delivery-and-retry
```

**W7: Microsoft Learn zu privaten Endpunkten für Event-Grid-Topics.** Der beschriebene Private Endpoint ist der Eingang zum Topic, nicht pauschal eine Lösung für den ausgehenden Weg zum UMP-Webhook.

```text
https://learn.microsoft.com/en-us/azure/event-grid/configure-private-endpoints
```

**W8: OpenSSH, Legacy Options und Release Notes.** Getrennte Host-/Benutzerauthentifizierung, hostbezogene Übergangsausnahmen und der Unterschied zwischen RSA-Schlüsseln und alten RSA/SHA-1-Signaturen.

```text
https://www.openssh.org/legacy.html
https://www.openssh.org/releasenotes.html
```

## Tatsächlich durchgeführte lokale Prüfungen

| Prüfung | Ergebnis |
|---|---|
| Alle drei Patches mit `git apply --check` gegen unveränderte temporäre Kopien des Uploads | Erfolgreich für DEV, TEST und PROD. |
| Patches angewendet und Ergebnis bytegenau mit dem erwarteten Inhalt verglichen | Erfolgreich für alle drei Umgebungen. |
| Patches rückwärts angewendet und Originalbytes wiederhergestellt | Erfolgreich für alle drei Umgebungen. |
| Vier relevante Katalogdateien gegen `v0.13.0` verglichen | Inhaltlich identisch nach Normalisierung der Zeilenenden. |
| Öffentlichen Jira-Schlüssel exakt extrahiert, als RSA-Schlüssel geparst, Bitlänge und SHA-256-Fingerabdruck berechnet | RSA 4096; Fingerabdruck im SFTP-Ticket. Keine private Schlüsseldatei übernommen. |
| Öffentliche PROD-SAML-Metadatendateien als XML geparst | Beide strukturell lesbar; keine Signatur-/Login-Prüfung behauptet. |
| Bash-Codeblöcke in diesem Paket mit `bash -n` geprüft | Syntaxprüfung erfolgreich; Serverbefehle nicht ausgeführt. |
| Dokumentierte Code-Einstiegspunkte | Gegen die tatsächlichen Quelldateien geprüft. |

Die maschinenlesbaren Ergebnisse liegen in `code/testergebnisse.json`.

**Nicht durchgeführt:** Terraform-/Terragrunt-Validierung mit den echten Providern, Terraform-Plan gegen euren State, Azure-KQL-Ausführung, E-Mail-Test, SFTP-Verbindung, Dateitransfer, Live-Metadatenabruf, SAML-Login, Event-Grid-Bereitstellung oder eine Codeänderung in euren entfernten Repositories. Terraform und Terragrunt waren in der lokalen Analyseumgebung nicht vorhanden. Daher ist der Patch ein geprüfter Diff mit offener Infrastrukturvalidierung, kein nachgewiesen deploybares Endergebnis.

## Separater Hinweis zu Schlüsseldateien im Originalupload

Unter den Mellon-Konfigurationen des Originaluploads liegen auch Dateien mit privaten SAML-Schlüsseln. Ob diese heute aktiv sind, wurde nicht geprüft. Dieses Arbeitsplan-Paket enthält sie nicht. Den Repository-Upload deshalb nicht als allgemeinen Mailanhang weiterreichen. Den aktiven Schlüsselbestand über den vorgesehenen Betriebskanal prüfen; bei unzulässiger Weitergabe aktiver Schlüssel eine abgestimmte Rotation planen. Nicht während der Login-Diagnose auf Verdacht alle Schlüssel ersetzen.

[Zurück zur Übersicht](README.md)
