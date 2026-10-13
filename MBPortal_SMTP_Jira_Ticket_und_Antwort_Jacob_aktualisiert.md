# Jira-Ticket und Antwort an Jacob

> Entwürfe, nicht angelegt oder versendet. Die Mail erst nach Ticketanlage verwenden und `DBI-XXX` ersetzen. Der nächste Sprint ist das Umsetzungsziel, keine bestätigte Abschlusszusage.

## 1. Jira-Ticket

**Projekt:** DBI | **Typ:** Story | **Komponente:** DBI_Team_MAN

**Zusammenfassung:**

```text
MBPortal: SMTP-Transportverschlüsselung prüfen und absichern
```

**Beschreibung in Jira-Wiki-Syntax:**

```jira
h3. Ziel
Als Anwendungsverantwortlicher möchte ich, dass E-Mails vom MBPortal zum SMTP-Relay verschlüsselt übertragen werden und der Nachweis für die Data-in-Transit-Abfrage vorliegt.

h3. Technische Hinweise
* Repository: {{mbportal-backend}}
* Datei: {{service/src/main/resources/application.properties}}, Zeilen 63–64 im gelieferten Stand:
{code:language=properties}
quarkus.mailer.tls=false
quarkus.mailer.start-tls=DISABLED
{code}
Ohne wirksamen Override verwendet der Mailer kein TLS. Produktiver Klartextversand ist damit noch nicht nachgewiesen.
Overrides über {{ci/stage.deploy.yml}} (Zeilen 49–72, {{FNCENV_}}-Variablen) und die tatsächlichen Function-App-Settings prüfen.

h3. Anforderungen
* Effektive Mailer-Konfiguration in DEV/TEST/PROD prüfen.
* Mit dem Mailbetrieb verpflichtendes STARTTLS oder direktes TLS abstimmen und erforderliche Konfigurationsänderungen umsetzen. Unsichere allgemeine Defaults bereinigen; kein Klartext-Fallback.
* Zertifikatsprüfung und Verfahren gemäß SecSt. 003 berücksichtigen. Ziel: Abschluss im nächsten Sprint.

h3. Akzeptanzkriterien
* Wirksame Einstellungen und verschlüsselter Versand sind für DEV/TEST/PROD dokumentiert.
* Zustellung funktioniert; bei fehlender erforderlicher TLS-Unterstützung oder ungültigem Serverzertifikat erfolgt kein Versand.
* Erforderliche Änderungen sind bereitgestellt und der Produktionsnachweis liegt Daniel für die Rückmeldung an Jacob vor.
```

## 2. E-Mail an Jacob

```text
Betreff: AW: Abfrage Data-in-Transit-Verschlüsselung

Hallo Jacob,

ich habe die vorliegenden Repository- und Infrastrukturkonfigurationen der vier genannten Anwendungen geprüft.

Bereits vorhanden sind in den jeweils untersuchten Konfigurationen HTTPS für Web-/API-Zugriffe, TLS mindestens 1.2 am Service Bus sowie HTTPS-only und TLS mindestens 1.2 für Azure Storage. Dafür ist derzeit kein zusätzlicher Implementierungsbedarf erkennbar. Der Abgleich mit dem tatsächlichen Produktivbetrieb steht noch aus.

Zum Stand der einzelnen Anwendungen:

• Marktbearbeitungsplattform: Die genannten Schutzvorgaben sind in den geprüften Komponenten vorhanden. Datenbank-, SMB- und Betriebsverbindungen sind noch abschließend nachzuweisen.

• MB-Portal: Konkreter Prüfpunkt ist der E-Mail-Versand zum SMTP-Relay. TLS und STARTTLS sind in der Grundkonfiguration deaktiviert; produktive Overrides werden geprüft. Dafür habe ich DBI-XXX beim MBPortal-Team angelegt. Ziel ist, Prüfung, gegebenenfalls Korrektur und Nachweis im nächsten Sprint abzuschließen. Datenbank- und Anbieteranbindungen bleiben separat zu bestätigen.

• Investment Welt Marktbearbeitung: HTTPS ist in den untersuchten API-Verträgen und für DMS-Downloads vorgesehen. Die produktiven Verbindungen vom BFF zu DMS und ProductHub müssen noch bestätigt werden.

• Verbundschnittstelle Truuco: Der Truuco-Vertrag sieht HTTPS vor. Für die tatsächlichen Truuco-/SOLID-Endpunkte sowie PostgreSQL und Oracle-vDWH fehlen noch die vollständigen Verschlüsselungsnachweise.

Die Bestandsaufnahme ist damit abgeschlossen, eine vollständige Erfüllungsbestätigung für alle Anwendungen ist noch nicht möglich. Die übrigen offenen Punkte sind Nachweislücken, nicht bereits belegte unverschlüsselte Verbindungen.

Viele Grüße
Daniel
```

Quellengrundlage: Jacobs Mail vom 15.09.2026; Vorlage `DBI-8123.doc`; [Verbindungsmatrix, V01–V21 und Q01](Data_in_Transit_Verbindungsmatrix_und_Nachweise.md); [bisherige Arbeitsgrundlage, SMTP-Fundstelle und Deployment-Overrides](MBPortal_SMTP_Jira_Ticket_und_Antwort_Jacob.md).
