# MBPortal: SMTP-Transportverschlüsselung
## Jira-Ticket, Antwort an Jacob und technische Arbeitsgrundlage

**Stand:** 13.10.2026  
**Verantwortung und Koordination:** Daniel Iftime  
**Umsetzung:** MBPortal-Team / DBI_Team_MAN, mit Unterstützung durch Infrastruktur und Mailbetrieb

> **Verwendung:** Abschnitt 1 enthält die Felder und die Beschreibung für ein neues Jira-Ticket. Abschnitt 2 ist die kopierbare E-Mail an Jacob **nach tatsächlicher Ticketanlage**. Die übrigen Abschnitte dienen als technische Arbeitsgrundlage. Durch die Erstellung dieser Datei wurden weder ein Jira-Ticket angelegt noch eine E-Mail versendet oder Systeme verändert. Der nächste Sprint ist das gewünschte Umsetzungsziel; eine bestätigte Sprintzusage liegt nicht vor.

**Bewertungsgrenze:** Grundlage sind die gelieferten Repository-Snapshots, die bisherige Verbindungsmatrix, Jacobs abgebildete Anfrage und die Struktur der Vorlage `DBI-8123.doc`. Es gibt keinen produktiven Konfigurations- oder Verbindungsnachweis. „Im Code bereits vorhanden“ bedeutet deshalb nicht „produktiv abschließend freigegeben“. Die Umsetzungsvorschläge und Abnahmekriterien unten sind neu formulierte Arbeitsaufträge, keine bereits durchgeführten Maßnahmen.

---

# 1. Neues Jira-Ticket für das MBPortal-Team

## 1.1 Felder für die Anlage

| Feld | Inhalt |
|---|---|
| Projekt | Domain Business Intelligence / DBI |
| Vorgangstyp | Story, entsprechend der bereitgestellten Vorlage |
| Zusammenfassung | **MBPortal: SMTP-Transportverschlüsselung prüfen und verbindlich absichern** |
| Komponente | DBI_Team_MAN |
| Umsetzungsteam | MBPortal-Team; individuelle Zuweisung im Refinement |
| Koordination / Ansprechperson | Daniel Iftime |
| Abstimmung | Julian Schmidt, Christoph Kurrat sowie zuständiger Mailbetrieb/CIS |
| Priorisierung | Produktionsrelevanz kurzfristig klären; Priorität danach mit dem Team festlegen. Die Prio 4 der Beispielstory nicht ungeprüft übernehmen. |
| Zielplanung | Prüfung, gegebenenfalls Korrektur, Tests und Bereitstellung möglichst im nächsten Sprint abschließen. Sprintzuordnung und Abhängigkeiten im Planning bestätigen. |
| Labels | `SD_MB_DevOps`, sofern für das Team weiterhin passend |
| Primäres Repository | `mbportal-backend` |
| Anlass | Data-in-Transit-Abfrage von Jacob Oehler zu den betreuten Anwendungen |

`DBI-8123` ist ausschließlich die Strukturvorlage. Deren fachlichen Inhalt, Epic „Aufbau Impulsmanager HUB“, Fix-Version und Sprintnummern nicht übernehmen. Es ist ein neuer Vorgang anzulegen; dessen Schlüssel wird von Jira vergeben.

## 1.2 Beschreibung für Jira

### Als Verantwortlicher für die Anwendungssicherheit möchte ich, dass das MBPortal E-Mails zum SMTP-Relay ausschließlich über einen verbindlich abgesicherten Transport übermittelt, damit die E-Mail-Inhalte auf diesem Übertragungsweg geschützt sind und ein belastbarer Data-in-Transit-Nachweis vorliegt.

**Hintergrund und verständliche Einordnung**

Jacob fragt für die Marktbearbeitungsplattform, das MB-Portal, Investment Welt Marktbearbeitung und die Verbundschnittstelle Truuco nach Verbindungen, Protokollen und Verschlüsselungsstatus. Seine Anfrage nennt RL-5069 und SecSt. 16 als Anforderung sowie SecSt. 003 für zugelassene TLS-Verfahren. Der aktuelle Wortlaut dieser internen Vorgaben liegt noch nicht vollständig vor.

Im MBPortal wurde ein konkreter Prüfpunkt gefunden: Das Backend erstellt E-Mails und übergibt sie an einen Mailserver beziehungsweise SMTP-Relay. In der allgemeinen Backend-Konfiguration sind beide vorgesehenen TLS-Wege deaktiviert. Wenn diese Werte zur Laufzeit nicht überschrieben werden, sichert der Mailer die Verbindung zum Relay nicht mit TLS ab. Das ist ein Quellbefund, noch kein nachgewiesener produktiver Klartextversand. [S01, S02, E01]

**Betroffene Verbindung:** `MBPortal-Backend -> SMTP-Relay`. Nicht gemeint ist, dass das gesamte Portal unverschlüsselt wäre. Die weiterführende Zustellung durch die zentrale Mailinfrastruktur ist mit dem Mailbetrieb gesondert abzugrenzen. Eine Ende-zu-Ende-Verschlüsselung der E-Mail beim Empfänger ist nicht Gegenstand dieser Story.

### Offene Punkte:

1. Welche Mailer-Einstellungen sind in DEV, TEST und PROD tatsächlich wirksam? Überschreiben Azure-App-Settings, Pipelinevariablen oder andere Konfigurationsquellen die Werte aus `application.properties`?
2. Welches Relay mit welchem Port wird verwendet? Unterstützt der freigegebene Zugang verbindliches STARTTLS oder direktes TLS? Welche Zertifikatskette und Namensprüfung sind dafür erforderlich?
3. Welche Anforderungen ergeben sich aus der aktuellen SecSt. 003? Ist verpflichtendes STARTTLS zulässig oder muss die Verbindung bereits ab dem Aufbau TLS verwenden?
4. Wer liefert die Relay-/Zertifikatsinformationen und bestätigt die Erreichbarkeit? Sind diese Voraussetzungen rechtzeitig für das gewünschte Sprintziel verfügbar?

### Vorbedingungen:

- Aktueller Release-/Commitstand des Backends und effektive Konfiguration je Umgebung sind zugänglich. Der Archivstand allein ist kein Live-Nachweis.
- Mailbetrieb/CIS bestätigt Relay, Port, unterstützten Sicherheitsmodus und Vertrauenskette.
- Sichere Testempfänger und eine isolierte Testmöglichkeit für erfolgreiche sowie absichtlich fehlschlagende TLS-Verbindungen stehen bereit.
- Aktuelle interne Sicherheitsvorgaben werden für die Abnahme bereitgestellt. Zugangsdaten und E-Mail-Inhalte werden nicht in Jira abgelegt.

### Anforderungen:

**1. Produktiven Istzustand feststellen und dokumentieren**

Die wirksamen Mailer-Einstellungen mit ihrer Herkunft je Umgebung erfassen. Mindestens Host, Port, aktives Profil, `tls`, `start-tls` und gegebenenfalls eine benannte TLS-Konfiguration prüfen. Die nachstehend beschriebenen Azure-DevOps-Variablengruppen und die tatsächlichen Function-App-Settings abgleichen. Falls die Anwendung inzwischen aus GitLab ausgeliefert wird, auch die aktuell verwendete GitLab-Konfiguration prüfen. [S03, S04]

**2. Sichere und nachvollziehbare Konfiguration herstellen**

Falls der produktive Versand nicht ausreichend abgesichert ist, auf den mit Mailbetrieb und Security abgestimmten TLS-Modus umstellen. Kein stiller Rückfall auf unverschlüsselte Übertragung der E-Mail-Nutzdaten. Unsichere globale Defaults bereinigen oder wirksam auf isolierte lokale Tests begrenzen.

Falls eine sichere produktive Konfiguration bereits existiert, deren Wirkung belegen und die widersprüchliche Quell-/Deploymentkonfiguration bereinigen. Nicht ohne Bedarf einen funktionierenden Relay-Zugang austauschen.

**3. Zertifikatsprüfung und Fehlerverhalten nachweisen**

Der erwartete Relay-Server muss anhand der vereinbarten Vertrauenskette und Serveridentität überprüft werden. Keine pauschale Abschaltung der Zertifikatsprüfung als Fehlerbehebung. Bei fehlender TLS-Unterstützung, nicht vertrauenswürdigem Zertifikat oder fehlgeschlagener Namensprüfung muss der Versand kontrolliert fehlschlagen.

**4. Bestehende E-Mail-Funktionen und Tests erhalten**

Die vorhandene Versandfunktion, Empfänger, Vorlagen und Anhänge dürfen durch die Transportumstellung nicht fachlich verändert werden. Reale Transporttests ergänzen; ein Mock-Test des Mailer-Aufrufs allein reicht für den Verschlüsselungsnachweis nicht aus. Lokale Mailpit-Tests dürfen nicht unbeabsichtigt produktive Einstellungen beeinflussen. [S05, S06, S07]

**5. Bereitstellung und Abschlussnachweis liefern**

Änderung, sofern erforderlich, über DEV und TEST nach PROD bereitstellen. Je Umgebung die tatsächlich wirksame Konfiguration und eine Testverbindung dokumentieren. Abschlussnachweise an Daniel für die Rückmeldung an Jacob übergeben. Ein gemergter Codechange ohne Bereitstellung und Betriebsnachweis schließt den SMTP-Punkt nicht ab.

### Fehlerhandling:

- Eine nicht abgesicherte Verbindung darf nicht als erfolgreicher Mailversand behandelt werden.
- TLS-, Zertifikats- und Verbindungsfehler müssen erkennbar bleiben. Keine automatische Umschaltung auf Klartext.
- Die bestehende Wiederholungs-/Fehlerbehandlung prüfen und erhalten. Eine bisher nicht nachgewiesene Retry-Funktion nicht voraussetzen; das Betriebsverfahren für fehlgeschlagene Sendungen dokumentieren.
- Ein Rückfallverfahren darf nicht stillschweigend die unsichere Konfiguration wieder aktivieren. Bei Problemen gegebenenfalls Sendungen kontrolliert zurückhalten und Mailbetrieb einbeziehen.

### Server-Log / Audit-Log / Business-Log:

- Für die technische Nachvollziehbarkeit Umgebung, Zeitpunkt, Release, Zielsystem und Fehlerkategorie dokumentieren.
- Für den Abnahmenachweis TLS-Verfahren, ausgehandelte Parameter und Ergebnis der Serverprüfung aus geeigneten Client-/Relay-Nachweisen erfassen.
- Keine Passwörter, Tokens, vollständigen E-Mail-Inhalte oder unnötigen personenbezogenen Daten protokollieren. Kein unbereinigter Export aller Function-App-Settings in das Ticket.
- TLS-Details müssen nicht bei jeder E-Mail im Business-Log stehen. Ein zuordenbarer technischer Nachweis genügt als Umsetzungsvorschlag für die Abnahme.

### UX/UI:

Keine Änderung an der Oberfläche vorgesehen. Fachliche E-Mail-Inhalte und Empfänger bleiben unverändert. Bestehende Hinweise bei Versandfehlern dürfen keinen erfolgreichen Versand vortäuschen.

### Technische Hinweise:

#### A. Konkrete Fundstelle

**Repository:** `mbportal-backend`  
**Datei:** `service/src/main/resources/application.properties`  
**Zeilen im gelieferten Stand:** 61 bis 64, entscheidend 63 und 64.

```properties
quarkus.mailer.mock=false
quarkus.mailer.auth-methods=NONE
quarkus.mailer.tls=false
quarkus.mailer.start-tls=DISABLED
```

Diese Einträge stehen in der allgemeinen Konfiguration, nicht ausschließlich in `application-local.properties`. `mock=false` steht für tatsächlichen Versand statt Mockbetrieb. Der Verschlüsselungsbefund ergibt sich aus der Kombination von `tls=false` und `start-tls=DISABLED`; `auth-methods=NONE` ist nicht der Nachweis eines TLS-Fehlers. [S01, E01]

**Versionskorrektur:** Die erneut geprüfte `mbportal-backend/pom.xml`, Zeile 25, nennt **Quarkus 3.33.1**. Für die Umsetzung ist deshalb die Mailer-Dokumentation der Linie 3.33 relevant. Der frühere Bezug auf 3.20 wird hier korrigiert. Der tatsächlich ausgelieferte Release ist dennoch zu prüfen. [S02, E01]

#### B. Wo produktive Overrides herkommen können

In `mbportal-backend/ci/azure_deploy.yml` wird `discoveryKey=func-sales-portal` gesetzt. Die Datei `ci/stage.deploy.yml` bindet die Gruppe `${workloadEnvironment}.${discoveryKey}` ein. Daraus ergeben sich im gelieferten Pipelinepfad:

- `DEV.func-sales-portal`
- `TEST.func-sales-portal`
- `PROD.func-sales-portal`

`ci/stage.deploy.yml`, Zeilen 49 bis 72, liest Variablen mit dem Präfix `FNCENV_`, entfernt dieses Präfix und überträgt die Werte als Function-App-Settings. Deren konkrete Inhalte fehlen im Archiv. [S03, S04]

Für die Prüfung sind daher insbesondere diese Namen relevant; die Tabelle behauptet nicht, dass alle bereits gesetzt sind:

| Zu prüfender Pipelinevariablenname | Daraus erzeugtes Function-App-Setting |
|---|---|
| `FNCENV_QUARKUS_MAILER_HOST` | `QUARKUS_MAILER_HOST` |
| `FNCENV_QUARKUS_MAILER_PORT` | `QUARKUS_MAILER_PORT` |
| `FNCENV_QUARKUS_MAILER_TLS` | `QUARKUS_MAILER_TLS` |
| `FNCENV_QUARKUS_MAILER_START_TLS` | `QUARKUS_MAILER_START_TLS` |
| `FNCENV_QUARKUS_MAILER_TLS_CONFIGURATION_NAME` | `QUARKUS_MAILER_TLS_CONFIGURATION_NAME` |

Auch mögliche JVM-Systemparameter, benannte Mailer und weitere Konfigurationsquellen berücksichtigen. Nicht aus einer leeren oder fehlenden Variablenliste auf die endgültige Laufzeitkonfiguration schließen.

#### C. Geeigneten Sicherheitsmodus auswählen

**Wenn das freigegebene Relay verpflichtendes STARTTLS unterstützt**, sieht das relevante Quarkus-Konfigurationspaar so aus:

```properties
quarkus.mailer.tls=false
quarkus.mailer.start-tls=REQUIRED
```

`tls=false` ist in dieser Variante beabsichtigt: Nach einer zunächst unverschlüsselten SMTP-Begrüßung wird auf TLS umgestellt, bevor E-Mail-Nutzdaten übertragen werden. `OPTIONAL` ist für das geforderte verbindliche Verhalten nicht als Ziel vorgesehen. [E01]

**Wenn direktes TLS verlangt wird**, muss der entsprechende Relay-Zugang mit `quarkus.mailer.tls=true` verwendet werden. Host, Port, Zertifikatsvertrauen und weitere TLS-Einstellungen passend zu diesem Zugang festlegen. Die beiden Varianten nicht ungeprüft kombinieren. [E01]

Dies sind Auswahlmöglichkeiten, kein freigegebener Produktionspatch. Es wird bewusst kein unbekannter Relay-Hostname oder pauschaler Port vorgegeben. Die Zulässigkeit des gewählten Verfahrens und seine Parameter sind intern abzustimmen.

#### D. Tests und Infrastrukturzuständigkeit

- Der eigentliche Versand liegt in `service/src/main/java/com/ui/b2b/mbportal/backend/email/sender/EmailSender.java`, Zeilen 18 bis 26. Dort wird `Mailer.send(...)` aufgerufen. Es ist zunächst ein Konfigurations- und Nachweisthema, nicht automatisch ein Fehler dieser Java-Methode. [S05]
- `service/src/test/java/com/ui/b2b/mbportal/backend/email/sender/EmailSenderTest.java` verwendet einen gemockten Mailer. Diese Tests prüfen die Übergabe der Nachricht, nicht den TLS-Verbindungsaufbau. [S06]
- `application-local.properties`, Zeilen 24 bis 30, konfiguriert einen lokalen Mailserver; `application-integrationtest.properties`, Zeile 28, verweist auf `localhost`. Bei geänderten Defaults die isolierten Testprofile gezielt nachziehen. Lokale Ausnahmen sind kein Nachweis produktiven Klartextverkehrs. [S07]
- Im Repository `ui-mb-infrastructure`, Datei `modules/functionapp/main.tf`, Zeilen 198 bis 203, sind `app_settings` unter `ignore_changes` aufgeführt. Daher Quellkonfiguration und wirksame Anwendungsauslieferung gemeinsam korrigieren, statt eine Terraform-Änderung an Mailer-Settings als hinreichend anzusehen. [S08]
- Diese Story hängt nicht zwingend von der vollständigen GitLab-Migration ab. Die aktuelle Korrektur muss anschließend in den neuen Auslieferungsweg übernommen werden.

### Feature Toggle:

Kein fachlicher Feature-Schalter vorgesehen. Die Produktionsverschlüsselung darf nicht durch einen allgemeinen Komfortschalter deaktivierbar werden. Isolierte lokale Testausnahmen müssen explizit auf diese Profile begrenzt bleiben.

### Akzeptanzkriterien:

- [ ] Der effektive Istzustand für DEV, TEST und PROD einschließlich Herkunft der relevanten Konfigurationswerte ist dokumentiert.
- [ ] Relay, Port, Sicherheitsmodus, Zertifikatsanforderungen und zulässige Verfahren wurden mit den zuständigen Stellen abgestimmt.
- [ ] Der Versand von E-Mail-Nutzdaten erfolgt ausschließlich über den vereinbarten verschlüsselten Transport. Es gibt keinen stillen Klartext-Fallback.
- [ ] Globale unsichere Defaults sind bereinigt oder technisch nachvollziehbar auf isolierte lokale Testprofile begrenzt. Bestehende sichere Overrides sind dauerhaft im verwendeten Deploymentpfad berücksichtigt.
- [ ] Erfolgreicher TLS-Versand mit funktionierender Zustellung wurde nachgewiesen.
- [ ] Negativtests belegen den Abbruch bei fehlender erforderlicher TLS-Unterstützung sowie unzulässigem Serverzertifikat oder falscher Serveridentität.
- [ ] Repräsentative bestehende E-Mail-Funktionen, einschließlich eines vorgesehenen Anhangsfalls, funktionieren weiterhin. Lokale und Integrations-Testprofile sind geprüft.
- [ ] Änderungen wurden in den betroffenen Umgebungen bereitgestellt. Für den Produktionsabschluss liegen ein Nachweis der tatsächlich eingesetzten Version und eine zuordenbare Testverbindung vor.
- [ ] Fehler- und Wiederanlaufverfahren sind dokumentiert; die Nachweise enthalten keine Geheimnisse oder unnötigen personenbezogenen Daten.
- [ ] Daniel hat den Abschlussnachweis für den SMTP-Hop erhalten. Nicht zum SMTP-Hop gehörende offene Data-in-Transit-Nachweise werden dadurch nicht automatisch geschlossen.

## 1.3 Vorgeschlagene Unteraufgaben und Zuständigkeiten

| Unteraufgabe | Federführung | Ergebnis |
|---|---|---|
| Effektive Mailer-Konfiguration je Umgebung prüfen | MBPortal-Team und Daniel | Bereinigter Istnachweis, inklusive möglicher Overrides |
| Relay-TLS und Zertifikatsanforderungen abstimmen | Mailbetrieb/CIS, koordiniert durch Daniel | Freigegebener Zugang und Testvoraussetzungen |
| Defaults und Deploymentkonfiguration absichern | MBPortal-Team | Reviewfähige Änderung oder belastbare Bereinigung bereits sicherer Overrides |
| Positive und negative Transporttests ergänzen | MBPortal-Team / QA | Belegbarer Versand und Ablehnung unsicherer Verbindungen |
| Bereitstellen und Nachweise übergeben | MBPortal-Team und Daniel | DEV-/TEST-/PROD-Nachweise und Rückmeldung für Jacob |

---

# 2. E-Mail an Jacob

> **Versandbedingung:** Der folgende Text meldet die Ticketanlage und darf deshalb erst nach tatsächlichem Anlegen der Story verwendet werden. Das Sprintziel ist bewusst als Ziel formuliert, nicht als bereits bestätigter Abschluss. Nach der Anlage kann der Ticket-Titel im Mailtext mit dem tatsächlichen Jira-Vorgang verlinkt werden; es wird hier keine Vorgangsnummer erfunden.

**Betreff:** AW: Abfrage Data-in-Transit-Verschlüsselung: Marktbearbeitungsplattform, MB-Portal, Investment Welt Marktbearbeitung und Verbundschnittstelle Truuco

Hallo Jacob,

ich habe die vorliegenden Repository-, Infrastruktur- und Schnittstellenkonfigurationen der genannten Anwendungen ausgewertet und die erkennbaren Verbindungen, Protokolle und Verschlüsselungsvorgaben zusammengestellt. Die quellenbasierte Bestandsaufnahme und die Zuordnung des konkret auffälligen SMTP-Punkts zum MBPortal-Team sind damit abgeschlossen.

**Bereits vorhandene Maßnahmen**

In den untersuchten Konfigurationen sind folgende Schutzmaßnahmen bereits vorgesehen. Für diese konkreten Einstellungen ergibt sich aus dem Review derzeit kein zusätzlicher Implementierungsauftrag:

| Bereich / Verbindung | Protokoll und bereits vorhandene Maßnahme |
|---|---|
| Webzugriffe über das Application Gateway | HTTPS und eine TLS-Policy sind konfiguriert. Auch die untersuchten Verbindungen vom Gateway zu Web- und Storage-Backends verwenden HTTPS. |
| API-Zugriffe und Azure-Functions-Backends | Die untersuchten API-Verträge sehen HTTPS vor. Das geprüfte Function-App-Modul setzt HTTPS-only und mindestens TLS 1.2. |
| Anwendungen zum Azure Service Bus | Messaging über TLS; der Namespace ist mit einer TLS-Mindestversion von 1.2 konfiguriert. |
| Anwendungen zu den untersuchten Azure-Storage-Ressourcen | HTTPS-only und mindestens TLS 1.2 sind konfiguriert. Die untersuchte DMS-Downloadfunktion beschränkt ihre SAS-Zugriffe auf HTTPS. |
| Anmeldung und Tokenabruf bei Entra ID/B2C | Die sichtbaren Authentifizierungsadressen verwenden HTTPS. |

Diese Maßnahmen müssen nicht erneut grundsätzlich implementiert werden. Für den endgültigen Erfüllungsnachweis bleibt jedoch abzugleichen, ob die vorgesehenen Einstellungen an sämtlichen tatsächlich genutzten produktiven Ressourcen und Verbindungspfaden wirksam sind. Das ist eine verbleibende Nachweisaufgabe, keine bereits festgestellte Fehlkonfiguration dieser Bereiche. Neue technische Änderungen oder produktive Tests sind durch die bisherige Auswertung noch nicht erfolgt.

**MBPortal: Ticket für den E-Mail-Transport angelegt**

Für den konkreten Prüfpunkt habe ich beim MBPortal-Team die Story **„MBPortal: SMTP-Transportverschlüsselung prüfen und verbindlich absichern“** angelegt. Ziel ist, die Prüfung und die gegebenenfalls erforderliche Korrektur einschließlich Tests und Bereitstellung im nächsten Sprint abzuschließen. Die dafür erforderlichen Abstimmungen mit dem Mailbetrieb und dem Team koordiniere ich.

Betroffen ist ausschließlich die Verbindung **vom MBPortal-Backend zum SMTP-Relay**. Im Repository `mbportal-backend`, Datei `service/src/main/resources/application.properties`, Zeilen 63 und 64, stehen `quarkus.mailer.tls=false` und `quarkus.mailer.start-tls=DISABLED`. Das bedeutet nicht, dass das gesamte MBPortal unverschlüsselt kommuniziert. Zunächst wird geprüft, ob produktive Einstellungen diese Werte bereits sicher überschreiben. Falls nicht, wird der E-Mail-Transport verbindlich abgesichert und mit einem tatsächlichen Verbindungsnachweis getestet.

**Noch getrennt nachzuweisen**

Zusätzlich bleiben die vollständigen produktiven PostgreSQL-Verbindungen und die Oracle-vDWH-Verbindung, einzelne extern gesetzte API-Ziele einschließlich SOLID/Truuco sowie Anbieteranbindungen zu prüfen. Beim Action Hub ist für PostgreSQL bereits `sslmode=require` hinterlegt; die vollständige Serveridentitätsprüfung ist damit noch nicht belegt.

Auch die wirksame SMB-Verschlüsselung der Azure-Files-Anbindung sowie die vollständigen Konfigurationen der Betriebsverbindungen zu Key Vault, App Configuration und Monitoring/Reporting müssen abschließend zugeordnet beziehungsweise nachgewiesen werden. Die geplante externe UMP-Service-Bus-Anbindung wird erst entsprechend ihrem tatsächlichen Inbetriebnahmestand bewertet. Fehlende Nachweise bedeuten hier nicht automatisch unverschlüsselte Übertragung.

Bitte den Stand daher als **abgeschlossene quellenbasierte Bestandsaufnahme mit bereits vorhandenen Schutzvorgaben, einer angelegten SMTP-Story und noch offenen Betriebsnachweisen** aufnehmen, noch nicht als vollständige Erfüllungsbestätigung für alle Anwendungen. Auch der Abschluss der SMTP-Story allein schließt die übrigen Nachweise nicht ab.

Bitte stelle mir für den finalen Abgleich noch die aktuelle SecSt. 003 und gegebenenfalls die gewünschte Reporting-Vorlage zur Verfügung. Den Abschluss des SMTP-Punkts melde ich nach Bereitstellung und erfolgreichem Nachweis zurück.

Viele Grüße  
Daniel

## 2.1 Ersatzabsatz, falls die E-Mail vor der Ticketanlage verschickt wird

Den Absatz beginnend mit „Für den konkreten Prüfpunkt habe ich …“ durch folgenden Text ersetzen und die Zwischenüberschrift in **„MBPortal: Ticket für den E-Mail-Transport vorbereitet“** ändern. Im Schlussabsatz entsprechend „vorbereiteter SMTP-Story“ statt „angelegter SMTP-Story“ verwenden.

> Für den konkreten Prüfpunkt habe ich die Story „MBPortal: SMTP-Transportverschlüsselung prüfen und verbindlich absichern“ für das MBPortal-Team vorbereitet. Die Anlage und Einplanung stimme ich mit dem Team ab. Ziel ist, die Prüfung und die gegebenenfalls erforderliche Korrektur einschließlich Tests und Bereitstellung im nächsten Sprint abzuschließen.

---

# 3. Was ist erledigt, was muss tatsächlich noch bearbeitet werden?

## 3.1 Erledigt ist die Analyse, nicht eine neue technische Umsetzung

Die quellenbasierte Verbindungsmatrix wurde erstellt, vorhandene Verschlüsselungsvorgaben wurden erfasst und der SMTP-Prüfpunkt wurde auf konkrete Dateien sowie den möglichen Override-Pfad eingegrenzt. Es wurden keine Repositories, Azure-Ressourcen oder Mailserver geändert und keine produktiven Handshakes durchgeführt.

## 3.2 Bereits vorhanden: kein konkreter zusätzlicher Umbau aus diesem Review abgeleitet

Die Einstufung gilt nur für die jeweils genannte Konfiguration. Ein vollständiges Repository oder eine ganze Anwendung ist damit nicht pauschal „erledigt“.

| Bereich | Repository und Datei | Bereits vorhandene Einstellung | Was noch bleibt |
|---|---|---|---|
| Gateway-TLS | `ui-mb-infrastructure` / `modules/app-gateway/main.tf`, Z. 31-36 | `AppGwSslPolicy20220101S` | Aktive Listener, Zertifikate und Policy mit PROD abgleichen. |
| Verschlüsselter Gateway-Backend-Hop | `ui-mb-infrastructure` / `environment/prod/applications/plattform/app-gateway/terragrunt.hcl`, Z. 404-470 | Untersuchte Backend-Settings: `protocol="Https"`, Port 443 | Tatsächlich aktive Backends und Serverprüfung nachweisen. |
| Azure-Functions-Zugänge | `ui-mb-infrastructure` / `modules/functionapp/main.tf`, Z. 109 und 140-142 | `https_only=true`, TLS-Mindestversion 1.2 für Anwendung und SCM | Nutzung des Moduls, Ausnahmen und effektive Ressourcenwerte zuordnen. |
| Service Bus | `ui-mb-infrastructure` / `modules/servicebus/main.tf`, Z. 17 | `minimum_tls_version="1.2"` | Aktive Namespaces und Client-/Triggertransporte prüfen. Queue-RBAC ist ein separates Thema. |
| Blob-/Function-Storage | `ui-mb-infrastructure` / `modules/blob-storage/main.tf`, Z. 33-34; `modules/functionapp/main.tf`, Z. 50-51 | HTTPS-only, mindestens TLS 1.2 | Alle tatsächlich genutzten Storage-Konten mit diesen Vorgaben abgleichen. |
| DMS-Download | `iw-dms-api` / `src/main/java/com/ui/b2b/mb/dms/util/AzureStorageUtil.java`, Z. 124-125 | `SasProtocol.HTTPS_ONLY` | Aktiven Stand und Zielkonto bestätigen. Dies bewertet nicht den gesonderten DMS-Virenscanprozess. |
| Portal-APIM-Vertrag | `API-Definitions` / `api-packages/web-marktbearbeitung-mbportal-api/1.0.0/config.yml`, Z. 15-33 | HTTPS auf API-Seite und HTTPS-Backend-URLs | Der Vertragsstand ist ein Snapshot; aktive Version und komplette Route bestätigen. |
| OIDC im Portal | `mbportal-backend` / `service/src/main/resources/application.properties`, Z. 43-44 | HTTPS für Authentifizierungsadresse und Issuer | Produktive Konfigurationswerte und tatsächliche Authentifizierungspfade zuordnen. |

Quellen: ursprüngliche Nachweisdatei Q02-Q08, Q18-Q19 und Q27. Die Infrastrukturdateien und der Portal-APIM-/OIDC-Auszug wurden für dieses Dokument erneut aus dem Originalarchiv gegengeprüft.

## 3.3 Weiter offen: Nachweise oder konkret zu klärende Abweichungen

| Punkt | Repository / genaue Fundstelle | Art des Punkts und nächster Schritt |
|---|---|---|
| **MBPortal-SMTP** | `mbportal-backend` / `service/src/main/resources/application.properties`, Z. 63-64 | **Konkreter abweichender Quellbefund.** Overrides und tatsächlichen Versand prüfen, danach absichern beziehungsweise sichere bestehende Konfiguration nachweisen. Gegenstand der neuen Story. |
| MBPortal-/Admin-/Datenprodukt-Datenbanken | `ui-mb-infrastructure` / `modules/postgresql/main.tf`, Z. 48-54; `environment/_apps/mashub/database.hcl`, Z. 41-44; die jeweiligen `service/src/main/resources/application.properties` | **Nachweislücke.** Fehlende explizite TLS-Parameter im Ausschnitt beweisen keinen unverschlüsselten Datenbankbetrieb. Alle effektiven Datasources und Server-/Sitzungseinstellungen erfassen. |
| Action-Hub-Datenbank | `im-actionhub-backend` / `deployment/appsettings_development.json`, `deployment/appsettings_test.json`, `deployment/appsettings_prod.json`, jeweils Z. 35-36 | `sslmode=require` ist vorhanden. Serveridentitätsprüfung und tatsächlich ausgehandelter Transport bleiben zu bewerten. Keine pauschale Freigabe allein anhand dieses Werts. |
| VBI-Datenbanken | `verbundintegration-backend` / `src/main/resources/application.properties`, Z. 44-45 und 64-65; `.gitlab-ci.yml`, Z. 79-95 | **Nachweislücke.** `JDBC_URL` und `JDBC_ORACLE_URL` werden extern geliefert. Mit Betreiber/DBA tatsächliche PostgreSQL- und Oracle-Transportabsicherung prüfen. |
| VBI zu SOLID und Truuco | `verbundintegration-backend` / `src/main/resources/application.properties`, Z. 84-97; `.gitlab-ci.yml`, Z. 84 und 95 | Externe Werte `TRUUCO_BASE_URL` und `SOLID_SERVER_URL` sowie beide Gateway-Seiten nachweisen. |
| Investmentwelt-BFF zu DMS/ProductHub | `iw-bff-api` / `src/main/java/com/ui/b2b/mb/bff/config/di/AppConfig.java`, Z. 20-27; `.gitlab-ci.yml`, Z. 75-77 | Effektive Zieladressen und APIM-Backends abgleichen. REST-Verträge allein belegen nicht den kompletten Live-Pfad. |
| FINCO-/PBM-/UMP-Anbieterzugänge | Beispielhafte **konkrete Abweichung**: `web-marktbearbeitung-finco-api` unter `api/`, Datei `api_config/OpenAPI/1.0.0/api_definition.yml`, Z. 29-34. UMP-Fachvertrag: `ump-integration/service/src/main/resources/openapi_ump.yml`, Z. 5-9. | FINCO-TEST/PROD enthalten im Snapshot Wetter-Demo-Adressen. Keine Aussage über aktive Fehlleitung; zunächst tatsächlichen Deploymentstand bestimmen. Alle produktiven Anbieterziele und Rückmeldungen zuordnen. |
| Azure Files / Transaktionsablage | `ui-mb-infrastructure` / `modules/functionapp/main.tf`, Z. 65-75; `environment/_apps/mashub/sales-portal-api.hcl`, Z. 68-73 | SMB 3.1.1 und AES-GCM sind vorgesehen. Die wirksame SMB-Verschlüsselung und interne Zulässigkeit bleiben nachzuweisen. SMB-Verschlüsselung nicht als TLS bezeichnen. |
| VBI-Storage-Queues | `verbundintegration-backend` / `src/main/java/de/unioninvestment/services/infrastructure/queue/QueueHandler.java`, Z. 64-70 | `AzureWebJobsStorage` kommt aus der Laufzeit. Zielkonto und Verbindungsparameter ohne Schlüssel prüfen. Das ist Azure Storage Queue, nicht Service Bus. |
| Key Vault / App Configuration | `verbundintegration-backend` / `src/main/java/de/unioninvestment/vaults/VbstKeyVault.java`, Z. 35-37; `mbportal-shared` / `config/src/main/java/com/ui/b2b/mbportal/config/AzureConfigurationClientFactory.java`, Z. 20-24 | HTTPS ist für den sichtbaren Vault-Aufruf belegt; weitere Endpunkte sind variabel. Vollständige produktive Verbindungen ergänzen. |
| Monitoring / Reporting | `im-actionhub-backend` / `src/main/resources/application.properties`, Z. 50-58, als Beispiel für extern konfigurierte Telemetrie | Effektive Exporter-, Monitoring- und Reportingziele fehlen teilweise. Nicht aus einem aktivierten SDK auf einen vollständigen Transportnachweis schließen. |
| Neue UMP-Service-Bus-Anbindung | DBI-10553 und zugehöriger Action-Hub-Vertrag | Geplanten und tatsächlich betriebenen Stand getrennt führen. Keine Vorabfreigabe eines noch nicht nachgewiesenen externen Clients. |

Quellen: ursprüngliche Nachweisdatei Q01, Q08-Q17 und Q21-Q29 sowie der gelieferte DBI-10553-Kontext. Bei den erneut gelieferten VBI-/Action-Hub-Repositories ist der neuere Stand aus `_2.7z` gemeint. Sämtliche Zeilennummern beziehen sich auf die gelieferten Snapshots und können im aktuellen Branch abweichen.

**Zuständigkeitsvorschlag:** Die neue MBPortal-Story bearbeitet den SMTP-Hop. Daniel koordiniert die anwendungsübergreifenden Betriebsnachweise mit den jeweiligen Teams, Mailbetrieb und DBA. Die Story nicht mit allen noch offenen Verbindungen der gesamten Plattform überladen; diese Punkte separat verfolgen und nicht bei SMTP-Abschluss automatisch als erledigt melden.

---

# 4. Quellennachweise zum SMTP-Ticket

Die folgenden Fundstellen wurden für dieses Dokument direkt aus `marktbearbeitung.zip` geprüft. Archivpfade beginnen jeweils mit `marktbearbeitung/`. Die Referenzen benennen den gelieferten Stand, nicht zwingend den aktuellen produktiven Stand.

| Referenz | Repository / Datei | Beleg |
|---|---|---|
| S01 | `mbportal-backend/service/src/main/resources/application.properties`, Z. 61-64 | Allgemeine Mailer-Konfiguration mit deaktiviertem TLS/STARTTLS. |
| S02 | `mbportal-backend/pom.xml`, Z. 25; `mbportal-backend/service/pom.xml`, Z. 155 | Quarkus 3.33.1; Einbindung von `quarkus-mailer`. |
| S03 | `mbportal-backend/ci/azure_deploy.yml`, Z. 19-23, 84-115 | Discovery-Key `func-sales-portal` und Deploymentstufen DEV/TEST/PROD. |
| S04 | `mbportal-backend/ci/stage.deploy.yml`, Z. 21-24, 49-72 | Variablengruppen und Übertragung der `FNCENV_`-Werte in App Settings. |
| S05 | `mbportal-backend/service/src/main/java/com/ui/b2b/mbportal/backend/email/sender/EmailSender.java`, Z. 18-26 | Tatsächlicher Aufruf des injizierten Quarkus-Mailers. |
| S06 | `mbportal-backend/service/src/test/java/com/ui/b2b/mbportal/backend/email/sender/EmailSenderTest.java`, Z. 18-45 | Mockbasierter Test der Nachricht, kein TLS-Verbindungstest. |
| S07 | `mbportal-backend/service/src/main/resources/application-local.properties`, Z. 24-30; `application-integrationtest.properties` im selben Verzeichnis, Z. 28 | Lokale Mailserver-/Testkonfiguration. |
| S08 | `ui-mb-infrastructure/modules/functionapp/main.tf`, Z. 198-203 | Terraform ignoriert Änderungen an `app_settings`. |
| S09 | `DBI-8123.doc`, Beschreibung von „Als Entwickler …“ bis „Akzeptanzkriterien“ | Ausschließlich Strukturvorlage für die neue Story. Nicht als Nachweis einer SMTP-Umsetzung verwendet. |
| S10 | `Data_in_Transit_Verbindungsmatrix_und_Nachweise.md`, Stand 12.10.2026 | Vorangegangene Arbeitsmatrix mit Q01-Q30, Originalauszügen und Bewertungsgrenzen. |
| S11 | Abgebildete E-Mail von Jacob, Betreff „Abfrage Data-in-Transit Verschlüsselung“ | Anlass, betroffene Anwendungen und Verweise auf interne Sicherheitsvorgaben. |

**Zusätzlich geprüfte Herstellerdokumentation, keine Aussage über eure Live-Umgebung:**

- **E01:** Quarkus, *Mailer reference guide*, Version 3.33, Abschnitte „Configuring TLS“ und „Mailer Configuration Reference“, geprüft am 13.10.2026. `https://quarkus.io/version/3.33/guides/mailer-reference/`

Die aktuelle SecSt. 003 wurde nicht bereitgestellt und wird nicht durch die Herstellerdokumentation ersetzt. Neue Prüf- und Umsetzungsvorschläge sind in Abschnitt 1 als Anforderungen und Akzeptanzkriterien formuliert.

## Integrität der entscheidenden Originaldateien

- **S01:** `marktbearbeitung/mbportal-backend/service/src/main/resources/application.properties`
  - SHA-256: `c4abf75f9510aeac821d4f2ade43c8fec42835df8d22373cb31e9359f17a3cbf`
- **S02:** `marktbearbeitung/mbportal-backend/pom.xml`
  - SHA-256: `0a6d4ce82f22c2113a3767fcd7d28038b2bbfe56f82ff4eb58b1dcb564b3bbb0`
- **S04:** `marktbearbeitung/mbportal-backend/ci/stage.deploy.yml`
  - SHA-256: `7b879a1b720a574e2aedd10e82b93ac0d7197fa81c4113d00dce7ed16c0214af`
- **S08:** `marktbearbeitung/ui-mb-infrastructure/modules/functionapp/main.tf`
  - SHA-256: `0de6f7de519fde613eba1ba82ea1568d3ecb1b8d995810e477ea198d61801d0d`


---

# 5. Ablauf für die Übergabe

1. Die Story aus Abschnitt 1 in Jira anlegen und den tatsächlichen Ticket-Schlüssel notieren. Zuweisung und Sprintziel mit dem MBPortal-Team bestätigen.
2. Sobald die Story angelegt ist, die E-Mail aus Abschnitt 2 verwenden. Vor der Anlage den Ersatzabsatz und die dazu genannten Anpassungen verwenden. Eine feste Sprintzusage nur dann als bestätigt kommunizieren, wenn das Team sie tatsächlich gegeben hat.
3. Zuerst wirksame SMTP-Einstellungen und Relay-Fähigkeiten prüfen. Nur belegte notwendige Änderungen umsetzen; einen bereits sicheren Live-Zugang nicht auf Verdacht umbauen.
4. Bei Abschluss der Story Bereitstellung und echte Transportnachweise prüfen. Danach den SMTP-Hop an Jacob als abgeschlossen melden; übrige Datenbank-, Endpunkt- und Betriebsnachweise separat weiterführen.
