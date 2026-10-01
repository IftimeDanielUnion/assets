# Technik zum Nachschlagen

Die einfachen Arbeitsabläufe und Nachrichten stehen in den vier Ticketdateien. Dieser Anhang enthält nur die konkreten Einstiegspunkte und Prüfungen. Serverbefehle sind für eure berechtigte Linux-Umgebung gedacht, nicht für einen beliebigen eigenen Rechner. Es wurden hier keine Verbindungen zu euren Systemen aufgebaut.

<a id="sftp"></a>
## SFTP: Konfiguration und harmlose Erstprüfung

### 1. Welche Werte verwendet die Anwendung?

Diese fünf Parameter werden im UMP-Code auf Umgebungsvariablen abgebildet:

```text
mboExportServerHost        ← MBO_EXPORT_SERVER_HOST
mboExportServerPort        ← MBO_EXPORT_SERVER_PORT
mboExportServerUser        ← MBO_EXPORT_SERVER_USER
mboExportServerAuthKeyPath ← MBO_EXPORT_SERVER_AUTH_KEY_PATH
mboExportServerRemoteDir   ← MBO_EXPORT_SERVER_REMOTE_DIR
```

Quelle: `unioninvestment/etc/config/packages/global.yml`, Zeilen 20 bis 24. Sie werden im Konstruktor von `mbo_export_controller` verwendet. Lies die wirksame Deployment-/Laufzeitkonfiguration, nicht nur `.env.example`. Eine `.env`-Datei allein beweist den effektiven Wert nicht, wenn zum Beispiel andere Umgebungswerte oder ein kompiliertes Environment verwendet werden. Den danebenliegenden `MBO_PARTNER_HASH_SECRET_WORD` brauchst du für diese Diagnose nicht auszugeben. [B3]

### 2. Schlüssel und wirksame SSH-Einstellungen lokal prüfen

**Nur verwenden, wenn die tatsächliche UMP-Konfiguration die hier aus dem Jira übernommenen Werte bestätigt.** Bei einer Abweichung gelten die realen freigegebenen Zielwerte. Dieser Block wird auf der vorgesehenen UMP-Instanz ausgeführt. Er verbindet sich noch nicht zur Gegenstelle.

```bash
sudo -u www-data -H bash <<'BASH'
set -euo pipefail

key=/var/www/.ssh/id_rsa_lukas_mbo
test -r "$key"

printf '\nFingerabdruck des tatsächlich nutzbaren Schlüssels:\n'
ssh-keygen -y -P '' -f "$key" | ssh-keygen -lf - -E sha256

printf '\nWirksame SSH-Konfiguration für den dokumentierten Lukas-Zugang:\n'
ssh -G -i "$key" -l UI_marketing lukaspartner.msu.biz |
  awk '$1 ~ /^(hostname|user|port|identityfile|hostkeyalgorithms|pubkeyacceptedalgorithms|stricthostkeychecking|userknownhostsfile)$/'
BASH
```

Es wird kein privater Schlüssel ausgegeben. Bei einem verschlüsselten privaten Schlüssel scheitert die Fingerabdruckableitung mit leerer Passphrase bewusst; nicht zur Vereinfachung die Verschlüsselung entfernen. Ein Dateiname mit `rsa` beweist den Schlüsseltyp nicht.

Erwarteter Fingerabdruck des öffentlichen Jira-Schlüssels:

```text
SHA256:dnNfJ61wYm8HJzH+x+DLgIvcREFPbmZUXdMVjJPvXV8
RSA, 4096 Bit, Kommentar azure-lukas-sftp-prod
```

### 3. Nur Anmeldung und aktuelles Verzeichnis testen

**Voraussetzungen:** Ziel/Benutzer/Schlüsselpfad sind bestätigt, der richtige Server-Host-Key ist geprüft und im freigegebenen Known-Hosts-Bestand vorhanden. Der folgende Block verbindet sich, schreibt aber keine Datei auf die Gegenstelle. Serverseitige Zugriffsprotokolle können dabei natürlich entstehen.

```bash
sudo -u www-data -H timeout 30s sftp \
  -v \
  -b - \
  -i /var/www/.ssh/id_rsa_lukas_mbo \
  -o BatchMode=yes \
  -o PasswordAuthentication=no \
  -o IdentitiesOnly=yes \
  -o StrictHostKeyChecking=yes \
  -o UpdateHostKeys=no \
  -o ConnectTimeout=10 \
  -o ConnectionAttempts=1 \
  UI_marketing@lukaspartner.msu.biz <<'SFTP'
pwd
bye
SFTP
```

Ein unbekannter oder abweichender Host-Key stoppt den Test. Das ist kein Grund, `StrictHostKeyChecking=no` zu setzen. Den Server-Fingerabdruck unabhängig mit Marc abgleichen. Alte Algorithmen sind in diesem Standardtest **nicht zusätzlich freigeschaltet**. [W8]

Der Test ist ein direkter OpenSSH-Test unter dem relevanten Benutzer, aber noch kein Aufruf des PHP-Connectors und kein vollständiger Anwendungstest. Er nutzt zusätzlich explizite Batch-/Sicherheitsoptionen. Nach seinem Erfolg muss der freigegebene Anwendungspfad weiter geprüft werden. [B3]

### 4. Nur bei belegtem Legacy-Problem: eng begrenzte Ausnahme

Der Jira-Verlauf enthält einen Fehler bei der Host-Key-Aushandlung mit einer alten Gegenstelle. Falls dieser Fehler weiterhin vorliegt, der Server-Fingerabdruck geprüft ist und der Betrieb die Übergangslösung freigibt, kann eine auf genau diesen Host begrenzte Ergänzung relevant sein:

```sshconfig
Host lukaspartner.msu.biz
    HostKeyAlgorithms +ssh-rsa
    PubkeyAcceptedAlgorithms +ssh-rsa
```

Das ist eine **bedingte Konfiguration**, kein automatisch auszuführender Fix. Die zweite Zeile zur Benutzeranmeldung ist nur nötig, wenn auch dieser Teil das alte Signaturverfahren verlangt. RSA-Schlüssel und die alte `ssh-rsa`-Signatur sind nicht dasselbe. Keinesfalls einen globalen `Host *`-Block abschwächen. Vorhandene frühere Host-Blöcke und deren wirksame Werte mit `ssh -G` prüfen; ein spätes Anhängen überschreibt nicht automatisch bereits gesetzte Werte. Die Ausnahme bei Modernisierung der Gegenseite wieder entfernen. [W8]

Im Ansible-Upload führt `ssh-client.yml` die Rolle `osc.system.sshclient` aus. `host_vars/ump-app` enthält bisher einen SSH-Client-Eintrag für Filme, aber keinen entsprechenden Lukas-Eintrag. Eine nötige dauerhafte Änderung deshalb in den bestehenden Konfigurationsweg übernehmen und nicht nur manuell auf einem Server belassen. Die passende Rollen-Schnittstelle vor der Änderung lesen. [B3]

<a id="monitoring"></a>
## Monitoring: konkrete Fundstellen

| Frage | Stelle |
|---|---|
| Wo wird Monitoring je Umgebung verwendet? | `live/live/dev/terragrunt.stack.hcl`, `live/live/test/terragrunt.stack.hcl`, `live/live/prod/terragrunt.stack.hcl`, jeweils `unit "monitoring"`. |
| Wo werden E-Mail-Empfänger erzeugt? | `catalog/modules/log-and-monitor/main.tf`, `azurerm_monitor_action_group.main`. |
| Wo entsteht die Alarmregel? | `catalog/modules/log-and-monitor/main.alerts.tf`, `azurerm_monitor_scheduled_query_rules_alert_v2.main`. |
| Welche Eingaben sind zulässig? | `catalog/modules/log-and-monitor/variables.logs.tf`. |
| Wie kommen die Eingaben im Modul an? | `catalog/units/log-and-monitor/terragrunt.hcl`. |
| Wie werden die OTEL-Daten zugeordnet? | `catalog/modules/log-collector-open-telemetry/constants.tf`, insbesondere `dataFlows`, `directDataSources` und `references`. |
| Wo werden Fehlerstufe und Dienstname gesetzt? | `unioninvestment/htdocs/src/Service/OpenTelemetry/SentryEventConverter.php` und `OpenTelemetryService.php`. |

Die DCR-Konfiguration routet den Logstream in den Workspace und versieht die Datensätze mit der Application-Insights-Referenz. Das ist der Code-Grund für die vorhandene Kombination, **kein Beweis für den aktuellen Logeingang**. Die tatsächliche Abfrage und der Alarm-Scope müssen zusammenpassen. [B2]

Für den Collector zunächst den Dienstzustand ansehen, zum Beispiel über den vorhandenen Dienstmanager. Den genauen Dienstnamen aus der Installation verwenden. Die ganze Konfigurationsdatei oder ungefilterte Logs nicht an einen Verteiler schicken; Fehlerdaten können Anwendungs- und Benutzerinformationen enthalten.

Die vollständigen Änderungen und die Bedienung liegen in [code/README.md](code/README.md). Die KQL-Abfrage verwendet die Tabelle `OTelLogs`, nicht auf Verdacht `exceptions` oder `AppExceptions`. Der Tabellenname und die Felder `SeverityText` und `ServiceName` sind für diesen OTEL-Weg in der offiziellen Tabellenreferenz dokumentiert. [W1]

<a id="saml"></a>
## SAML: aktive Dateien prüfen, nicht den Login umstellen

### 1. Aktive Webseiten und Mellon-Dateipfade ansehen

```bash
sudo apache2ctl -S
sudo grep -R -nE \
  '^[[:space:]]*(ServerName|ServerAlias|MellonIdPMetadataFile|MellonSPMetadataFile|MellonEndpointPath)[[:space:]]' \
  /etc/apache2/sites-enabled /etc/apache2/conf-enabled
```

Die Datei-Ausgabe ist ein Einstieg. Prüfe die wirklich aktiven Includes anhand der Apache-Konfiguration; ein Textfund in einer ungenutzten Datei ist noch kein Laufzeitnachweis.

### 2. Nur Kennungen und öffentliche Endpunkte auslesen

Der folgende Block liest ausschließlich die beiden im Upload verwendeten XML-Pfade. Verwende sie erst, nachdem Schritt 1 bestätigt hat, dass diese Dateien tatsächlich zur Zielinstanz gehören.

```bash
sudo python3 - <<'PYCODE'
from pathlib import Path
import sys
import xml.etree.ElementTree as ET

paths = (
    Path('/etc/apache2/mellon/idp_metadata_azure.xml'),
    Path('/etc/apache2/mellon/ump-app_metadata.xml'),
)
namespace = '{urn:oasis:names:tc:SAML:2.0:metadata}'
failed = False
for path in paths:
    print(f'\nDatei: {path}')
    try:
        root = ET.parse(path).getroot()
        entities = [root] if root.tag == namespace + 'EntityDescriptor' else []
        entities.extend(root.findall('.//' + namespace + 'EntityDescriptor'))
        if not entities:
            raise ValueError('Keine SAML-EntityDescriptor-Elemente gefunden.')
        for entity in entities:
            print('entityID:', entity.get('entityID', '(fehlt)'))
        for name in ('SingleSignOnService', 'AssertionConsumerService'):
            for element in root.findall('.//' + namespace + name):
                print(name + ':', element.get('Location', '(fehlt)'))
    except (OSError, ET.ParseError, ValueError) as exc:
        print(f'Prüfung fehlgeschlagen: {exc}', file=sys.stderr)
        failed = True
raise SystemExit(1 if failed else 0)
PYCODE
```

Der Block zeigt weder Zertifikatsinhalte noch private Schlüssel. Er prüft XML-Struktur und ausgewählte Werte, **nicht** die kryptografische XML-Signatur und **nicht** die erfolgreiche Anmeldung.

Olivers im Jira genannte Metadatenadresse für die temporäre App lautet:

```text
https://auth.user.union-investment.de/b2cunioninvestmentprod.onmicrosoft.com/B2C_1A_signin_saml_ump_tmp/samlp/metadata
```

Für eine Änderung diese Metadaten vollständig über den vorgesehenen vertrauenswürdigen Weg beziehen und die Zielzuordnung mit Oliver abgleichen. Signiertes XML nicht durch Suchen/Ersetzen bearbeiten. [J3, B4]

<a id="event-grid"></a>
## Event Grid: entscheidende Schnittstellenfragen

Der vorgeschlagene direkte Aufbau unterscheidet drei Berechtigungen:

| Verbindung | Vorgeschlagener Ansatz nach Bestätigung der Architektur |
|---|---|
| ProductHub → Event Grid | Entra-Anwendungsidentität mit `EventGrid Data Sender` am konkreten Topic. Keine selbst erfundene B2C-Audience als Ersatz für die native Dienstberechtigung. |
| Event Grid → UMP-Webhook | Entra-geschützte HTTPS-Schnittstelle; den dokumentierten Event-Grid-Webhook-Rollenweg und die Token-Prüfung der UMP passend einrichten. Eine Topic-Identity allein löst das nicht automatisch. |
| Event Grid → Fehler-Container | Dead-Letter-Ziel in der Event Subscription konfigurieren und dem verwendeten Zustellweg die notwendige Blob-Schreibberechtigung geben. Bei Managed Identity die passende Blob-Datenrolle verwenden. |

Quellen: **W4, W5, W6**. Die konkrete Identität, URL und Ressourcen-ID werden aus dem freigegebenen Zielaufbau übernommen, nicht geraten.

Das Event-Schema und die Erstvalidierung der Subscription müssen zum Empfänger passen. Außerdem braucht der Empfänger eine nachvollziehbare Behandlung wiederholter Zustellung. Fachliche Aktionen dürfen nicht allein deshalb doppelt ausgeführt werden, weil Azure erneut zustellt. [W5, W6]

Der Private Endpoint eines Topics ist ein **Eingangsweg zum Topic**. Er beweist keine private Erreichbarkeit des UMP-Webhooks aus Event Grid. Senderzugriff und ausgehende Zustellung separat planen. [W7]

[Zurück zur Übersicht](README.md)
