# UMP Tickets - Kurzfassung für Teams Call

## DBI-6455 - Exception Monitoring

Das Ticket sorgt dafür, dass Fehler aus den UMP-Anwendungen nicht nur in Azure landen, sondern bei `error` oder `fatal` automatisch eine E-Mail an `ui-team@os-cillation.de` auslösen. Der ursprüngliche Ticketansatz mit Sentry wurde inzwischen durch Azure Monitor und OpenTelemetry ersetzt. Die OpenTelemetry-Strecke funktioniert bereits grundsätzlich, die Logs kommen in Azure in `OTelLogs` an. Offen war vor allem die eigentliche Alarmierung.

Dafür habe ich im `live`-Repository einen separaten Terraform-Baustein für eine Azure Monitor Action Group und eine Scheduled Query Alert Rule ergänzt. Die KQL-Abfrage filtert auf `error` und `fatal` und berücksichtigt `ServiceName`, damit nachvollziehbar bleibt, welcher Dienst den Alarm ausgelöst hat. Zusätzlich wurde die Terragrunt-Einbindung für TEST ergänzt und die GitLab-CI so erweitert, dass Änderungen an dem neuen Alert-Modul auch einen TEST-Plan auslösen.

Der Code ist bereits committed und gepusht. Der Merge Request läuft aktuell und wartet auf Review. Nach dem Review und Merge muss `planTestEnvironment` geprüft werden. Danach wird `applyTestEnvironment` manuell gestartet. Anschließend muss ein kontrollierter echter Fehler erzeugt werden und OSC beziehungsweise Alex soll bestätigen, dass die Alarmmail tatsächlich angekommen ist. Erst dann ist das Ticket technisch vollständig abgenommen. PROD ist bewusst noch nicht Teil dieses ersten Rollouts.

## DBI-12139 - Blob Storage für Landingpage Assets

Das Ticket soll größere Landingpage-Dateien über Azure Blob Storage bereitstellen, damit sie von den Landingpages direkt geladen werden können. Die benötigte Infrastruktur war im bestehenden IaC bereits für dev, test und prod definiert, deshalb musste keine neue Storage-Lösung gebaut werden.

Ich habe den Azure-Ist-Stand gegen den vorhandenen Code geprüft. Die Storage Accounts, Container und die relevante Gateway-Konfiguration sind grundsätzlich vorhanden. In TEST wurde zusätzlich ein Test-Asset hochgeladen und erfolgreich ohne Anmeldung abgerufen. Der Abruf inklusive Range Request hat funktioniert. Bei DEV gibt es noch einen Zertifikatsfehler für den verwendeten Hostnamen. Das Ticket wartet deshalb aktuell nicht auf neuen Storage-Code, sondern auf die Klärung beziehungsweise Korrektur dieses Zertifikatsthemas und danach auf einen erneuten Zugriffstest. Ein Jira-Zwischenstand wurde bereits dokumentiert.

## DBI-11149 - SFTP-Verbindungen

Das Ticket soll sicherstellen, dass die UMP ihre Dateien weiterhin per SFTP an externe Partner, insbesondere Lukas, übertragen kann. Im bisherigen Verlauf war die Netzwerkverbindung grundsätzlich erreichbar, danach gab es Probleme im SSH- beziehungsweise Schlüsselbereich.

Ich habe Marc den neuen öffentlichen RSA-Schlüssel geschickt und den aktuellen Stand im Jira dokumentiert. Der neue Schlüssel soll zusätzlich zum bestehenden Schlüssel hinterlegt werden, nicht als Ersatz. Aktuell wartet das Ticket auf die Rückmeldung von Marc beziehungsweise Lukas, dass der Schlüssel korrekt hinterlegt wurde. Danach muss die SFTP-Anmeldung aus dem echten UMP-Laufzeitkontext getestet werden. Erst wenn Login und Zielverzeichnis funktionieren, sollte ein sicherer Anwendungstest mit einem freigegebenen Testauftrag durchgeführt werden.

## PB2B-27849 - Temporärer SAML-Login

Das Ticket betrifft den Login für die temporäre UMP-Instanz im Rahmen der Migration. Die temporäre App wurde laut Jira bereits angelegt und die Mellon-/SAML-Konfiguration war weitgehend vorbereitet.

Ich habe Oliver dazu angeschrieben. Seine Rückmeldung bestätigt, dass der Login über die temporäre App wieder zurück zur UMP führt und die grundlegende Integration damit offenbar funktioniert. Es ist aktuell keine neue SAML-Implementierung geplant. Es fehlt nur noch ein kurzer bestätigter Login-Test beziehungsweise ein sauberer Testbeleg für die temporäre Instanz. Wenn dieser vorhanden ist, kann das Ticket voraussichtlich ohne weiteren Code-Merge abgeschlossen werden.

## DBI-7161 - Event Grid ProductHub zu UMP

Das Ticket soll ProductHub-Änderungen über Azure Event Grid an die UMP übertragen. Geplant ist grob: ProductHub sendet ein Event an Event Grid, Event Grid liefert es an einen UMP-Endpunkt und dauerhaft fehlgeschlagene Zustellungen landen in einem Dead-Letter Blob Storage.

Ein Infrastrukturentwurf ist vorbereitet, aber noch nicht produktiv eingebunden. Das Ticket wartet noch auf fachliche und technische Schnittstellenklärung. Offen sind insbesondere das konkrete Event-Schema, die UMP-Empfänger-URL, die Authentifizierung für beide Richtungen und die Zuständigkeit für ProductHub-Sender und UMP-Empfänger. Deshalb sollte hier noch nichts deployed werden, bevor Sinan beziehungsweise OSC diese Punkte bestätigt haben.

## Aktueller Gesamtstand

DBI-6455 ist aktiv in Umsetzung und wartet gerade auf MR-Review. DBI-12139 ist infrastrukturell weitgehend vorhanden, hat aber noch den DEV-Zertifikatspunkt offen. DBI-11149 wartet extern auf die Bestätigung der Schlüsselhinterlegung. PB2B-27849 ist sehr nah am Abschluss und braucht im Wesentlichen nur noch einen bestätigten Login-Test. DBI-7161 ist noch nicht umsetzungsreif, weil die Schnittstellenfragen noch offen sind.
