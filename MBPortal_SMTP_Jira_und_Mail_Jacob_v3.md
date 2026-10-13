# MBPortal: Jira-Ticket und ausführlichere Antwort an Jacob

**Verwendung:** Die folgenden Blöcke sind zum Kopieren vorbereitet. Es wurde kein Ticket angelegt und keine E-Mail versendet. Die Mailfassung erst nach tatsächlicher Ticketanlage verwenden und `DBI-XXX` durch den vergebenen Schlüssel ersetzen. Der nächste Sprint ist das vorgeschlagene Umsetzungsziel, keine bereits bestätigte Abschlusszusage.

**Grundlage:** Vorliegende Repository-Snapshots, Verbindungsmatrix und Jacobs Anfrage. „Vorhanden“ bezeichnet die untersuchte Konfiguration, nicht einen bereits geprüften Produktivbetrieb. Die Ticketanforderungen sind vorgeschlagene Arbeiten; die Mail berichtet den belegbaren Analysestand.

## 1. Jira-Ticket

**Projekt:** DBI | **Typ:** Story | **Komponente:** DBI_Team_MAN  
**Umsetzung:** MBPortal-Team, unterstützt durch Daniel und Mailbetrieb/CIS

### Zusammenfassung

```text
MBPortal: SMTP-Transportverschlüsselung prüfen und verbindlich absichern
```

### Beschreibung in Jira-Wiki-Syntax

```jira
h3. Ziel
Als Anwendungsverantwortlicher möchte ich, dass E-Mails vom MBPortal-Backend zum SMTP-Relay ausschließlich über einen abgesicherten Transport übertragen werden und ein nachvollziehbarer Data-in-Transit-Nachweis vorliegt.

Anlass ist Jacobs Abfrage zu Verbindungen, Protokollen und Verschlüsselung nach RL-5069 und SecSt. 16. Die zulässigen Verfahren sind anhand der aktuellen SecSt. 003 abzugleichen.

h3. Befund und technische Hinweise
* Repository: {{mbportal-backend}}
* Datei: {{service/src/main/resources/application.properties}}
* Fundstelle: Zeilen 63–64 im gelieferten Stand, allgemeine Konfiguration und nicht ausschließlich ein lokales Testprofil:

{code:language=properties}
quarkus.mailer.tls=false
quarkus.mailer.start-tls=DISABLED
{code}

Damit sind beide TLS-Wege des Mailers deaktiviert. Ohne wirksamen Override wird die Verbindung zum SMTP-Relay nicht durch den Mailer mit TLS abgesichert. Ein tatsächlich unverschlüsselter Produktivversand ist aus dem Quellstand allein noch nicht nachgewiesen.

*Betroffen ist der SMTP-Hop vom Backend zum Relay, nicht das gesamte MBPortal.* Die weitere Zustellung durch die zentrale Mailinfrastruktur und eine Ende-zu-Ende-Verschlüsselung der Nachricht sind nicht Gegenstand dieser Story.

*Weitere Prüffundstellen:*
* {{mbportal-backend/ci/stage.deploy.yml}}, Z. 49–72: {{FNCENV_}}-Variablen werden als Function-App-Settings übernommen. Die tatsächlichen Werte sind im Archiv nicht enthalten.
* {{mbportal-backend/ci/azure_deploy.yml}} und {{ci/stage.deploy.yml}}: Im gelieferten Auslieferungsweg werden die Gruppen {{DEV.func-sales-portal}}, {{TEST.func-sales-portal}} und {{PROD.func-sales-portal}} verwendet. Den aktuell aktiven Deploymentweg einschließlich möglicher GitLab-Umstellung prüfen.
* {{ui-mb-infrastructure/modules/functionapp/main.tf}}, Z. 198–203: {{app_settings}} stehen unter {{ignore_changes}}. Die wirksame Anwendungsauslieferung muss daher mitbetrachtet werden; eine Terraform-Änderung allein genügt nicht als Nachweis.

h3. Offene Punkte / Vorbedingungen
* Welche Mailer-Werte und Profile sind in DEV, TEST und PROD tatsächlich wirksam? Insbesondere Host, Port, {{QUARKUS_MAILER_TLS}}, {{QUARKUS_MAILER_START_TLS}} und gegebenenfalls benannte TLS-Konfiguration prüfen.
* Welches Relay und welcher Port sind freigegeben? Mailbetrieb/CIS muss unterstützten TLS-Modus, Zertifikatskette und Erreichbarkeit bestätigen.
* Aktuelle SecSt. 003 und sichere Testmöglichkeiten müssen für die Abnahme vorliegen. Keine Zugangsdaten in Jira hinterlegen.

h3. Anforderungen
# Istzustand einschließlich Konfigurationsherkunft je Umgebung dokumentieren. Produktive Overrides nicht übersehen und nicht allein aus den allgemeinen Defaults auf Klartextverkehr schließen.
# Mit Mailbetrieb/Security direktes TLS oder verpflichtendes STARTTLS abstimmen. Falls erforderlich, Quell- und Deploymentkonfiguration korrigieren. Kein optionaler oder stiller Klartext-Fallback für E-Mail-Nutzdaten.
# Besteht bereits ein sicherer Override, dessen Wirkung nachweisen und die widersprüchlichen allgemeinen Defaults bereinigen. Lokale Testausnahmen müssen isoliert bleiben.
# Zertifikats- und Serveridentitätsprüfung sicherstellen. TLS-Probleme nicht durch pauschales Abschalten der Prüfung umgehen.
# Erfolgreiche Zustellung und kontrolliertes Scheitern unsicherer Verbindungen testen. Vorhandene Mailvorlagen, Empfänger und Anhänge fachlich unverändert lassen.
# Erforderliche Änderungen bereitstellen und Daniel den Abschlussnachweis zur Rückmeldung an Jacob übergeben. Ziel ist der Abschluss im nächsten Sprint; Einplanung und Abhängigkeiten sind im Planning zu bestätigen.

h3. Fehlerhandling / Server-Log / Audit-Log
* Bei fehlender erforderlicher TLS-Unterstützung oder unzulässigem Serverzertifikat keine E-Mail-Nutzdaten übertragen; Fehler erkennbar behandeln.
* Bestehendes Fehler-/Wiederanlaufverhalten prüfen. Ein Rollback darf nicht stillschweigend Klartextversand reaktivieren.
* Technische Nachweise mit Umgebung, Release, Zeitpunkt, Ziel, Sicherheitsmodus und Ergebnis dokumentieren. Keine Secrets oder E-Mail-Inhalte protokollieren.

h3. UX/UI und Feature Toggle
Keine fachliche Oberflächenänderung und kein neuer Feature-Schalter vorgesehen. Produktive Transportverschlüsselung darf nicht durch einen Komfortschalter umgangen werden.

h3. Akzeptanzkriterien
# Effektive Einstellungen und deren Herkunft sind für DEV, TEST und PROD dokumentiert; mögliche Overrides wurden berücksichtigt.
# Der freigegebene Transportmodus und die Zertifikatsanforderungen sind umgesetzt beziehungsweise bei bereits sicherem Betrieb nachgewiesen. Kein Klartext-Fallback für Nutzdaten.
# Ein realer Versandtest bestätigt abgesicherten Transport und erfolgreiche Zustellung. Ein Mailer-Mocktest allein genügt nicht.
# Negative Transporttests in einer kontrollierten Testumgebung belegen den Abbruch bei fehlendem erforderlichem TLS sowie ungültiger Vertrauenskette oder falscher Serveridentität.
# Erforderliche Änderungen sind ausgerollt. Für PROD liegen die wirksame Konfiguration und ein zuordenbarer Verbindungsnachweis vor.
# Daniel hat den Abschlussnachweis erhalten. Andere offene Datenbank-, Anbieter- und Betriebsverbindungen werden separat verfolgt und durch den Abschluss dieser Story nicht automatisch geschlossen.
```

## 2. E-Mail an Jacob

**Versandbedingung:** Ticket tatsächlich anlegen und `DBI-XXX` ersetzen. Die Aussagen zum nächsten Sprint beschreiben ein Ziel, keine unbestätigte Fertigstellungszusage.

```text
Betreff: AW: Abfrage Data-in-Transit-Verschlüsselung – Status der vier Anwendungen / DBI-XXX

Hallo Jacob,

ich habe die vorliegenden Repository-, Infrastruktur- und Schnittstellenkonfigurationen zu deinen vier genannten Anwendungen ausgewertet. Die erkennbaren Verbindungen, Protokolle und Verschlüsselungsvorgaben sind in einer Verbindungsmatrix erfasst.

Bereits vorhandene Schutzmaßnahmen müssen nicht erneut grundsätzlich implementiert werden. Offen ist teilweise noch der Nachweis, dass sie an den tatsächlich genutzten produktiven Ressourcen und über den gesamten Verbindungsweg wirksam sind. Neue technische Änderungen oder produktive Verbindungstests sind durch diesen Review noch nicht erfolgt.

1. Marktbearbeitungsplattform / gemeinsame Plattformkomponenten

Vorhanden sind Application Gateway, API Management und Azure-Functions-Backends. Am Gateway sind eine TLS-Policy sowie HTTPS für die untersuchten Web- und Backend-Verbindungen konfiguriert. Die geprüften Function Apps sind auf HTTPS-only und mindestens TLS 1.2 eingestellt; die untersuchten API-Verträge sehen ebenfalls HTTPS vor.

Beim betrachteten Action Hub wird für PostgreSQL in DEV, TEST und PROD mit „sslmode=require“ eine verschlüsselte Datenbankverbindung angefordert. Die vollständige Prüfung von Serverzertifikat und Hostname ist damit noch nicht belegt.

Noch zu prüfen sind die aktiven Gateway-/APIM-Ziele und Zertifikate, insbesondere auch die Verbindung vom Gateway zum Backend, sowie die wirksamen Datenbank- und Betriebsverbindungen. Die Zuordnung der gemeinsam genutzten Komponenten zu den jeweiligen Applikationseinträgen ist abschließend abzugleichen.

2. MB-Portal

Für Portal- und Administrationszugriffe sind HTTPS-Verbindungen vorgesehen. Für die Nachrichtenübertragung über Azure Service Bus ist mindestens TLS 1.2 konfiguriert; die untersuchten Blob-Storage-Ressourcen sind auf HTTPS-only und mindestens TLS 1.2 eingestellt. Diese Vorgaben sind bereits im Infrastrukturcode vorhanden.

Ein konkreter Prüfpunkt besteht beim E-Mail-Versand vom Backend zum SMTP-Relay: Im Repository „mbportal-backend“, Datei „service/src/main/resources/application.properties“, Zeilen 63–64, sind TLS und STARTTLS in der allgemeinen Konfiguration deaktiviert. Ob produktive Einstellungen dies bereits sicher überschreiben, ist noch zu prüfen. Das betrifft diesen Übertragungsweg, nicht das gesamte Portal.

Dafür habe ich DBI-XXX beim MBPortal-Team angelegt. Ziel ist, die Prüfung, gegebenenfalls Korrektur und den Betriebsnachweis im nächsten Sprint abzuschließen. Die erforderliche Abstimmung mit dem Mailbetrieb und die Einplanung koordiniere ich.

Separat offen bleiben die PostgreSQL-Verbindungen von Portal, Administration und Datenprodukt sowie die produktiven FINCO-, PBM- und UMP-Schnittstellen einschließlich Rückmeldungen. Die Anbieter-Verträge sehen HTTPS vor, belegen aber noch nicht alle aktiven Endpunkte. Für die Azure-Files-Transaktionsablage sind SMB 3.1.1 und AES-GCM-Verschlüsselung vorgesehen; die tatsächlich verschlüsselte Sitzung und die interne Zulässigkeit sind noch nachzuweisen. Hier handelt es sich um SMB-Verschlüsselung, nicht TLS.

3. Investment Welt Marktbearbeitung

Betrachtet wurden das Frontend-Backend (BFF), der Dokumentenservice (DMS), ProductHub und die zugehörigen Storage-Zugriffe. Die untersuchten API-Verträge sehen HTTPS vor. Die DMS-Funktion beschränkt ihre erzeugten Downloadlinks auf HTTPS.

Noch zu prüfen sind die tatsächlich eingesetzten Zieladressen vom BFF zu DMS und ProductHub einschließlich der APIM-Backend-Verbindungen. Diese Adressen werden über Deploymentvariablen gesetzt und liegen nicht vollständig als produktive Werte vor. Ebenso müssen die verwendeten Storage-Konten den vorgesehenen HTTPS-/TLS-Einstellungen zugeordnet werden. Ein zusätzlicher Implementierungsbedarf ist für die genannten vorhandenen Schutzvorgaben derzeit nicht erkennbar; der vollständige Betriebsnachweis steht aus.

4. Verbundschnittstelle Truuco

Die Verbundintegration verbindet sich mit Truuco/Atruvia, SOLID, PostgreSQL und dem Oracle-vDWH sowie mit Azure Blob Storage und Storage Queues. Für Truuco ist im untersuchten API-Vertrag HTTPS vorgesehen. Die betrachteten Storage-Infrastrukturvorgaben sehen ebenfalls HTTPS vor; der Queue-Pfad verwendet hier Azure Storage Queues, nicht Service Bus.

Noch nachzuweisen sind die tatsächlichen Truuco-/SOLID-Endpunkte und Gateway-Verbindungen, die PostgreSQL-TLS-Einstellungen und die ausgehandelte Verschlüsselung der Oracle-vDWH-Verbindung. Auch das zur Laufzeit verwendete Storage-Konto ist abzugleichen. Die erforderlichen URLs und Datenbankparameter werden teilweise extern gesetzt. Daraus lässt sich weder eine vollständige Erfüllung noch ein bereits belegter Klartextbetrieb ableiten.

Anwendungsübergreifend

Für die sichtbaren Entra-ID-/B2C-Anmeldeadressen und einen untersuchten Key-Vault-Zugriff ist HTTPS belegt. Für die vollständigen Betriebsverbindungen zu Key Vault, App Configuration sowie Monitoring/Reporting fehlen noch teilweise die produktiven Ziel- und Transportnachweise. Die neue externe UMP-Service-Bus-Anbindung wird entsprechend ihrem tatsächlichen Inbetriebnahmestand bewertet, nicht vorab als umgesetzt gemeldet.

Damit liegt die quellenbasierte Übersicht vor. Für die oben genannten vorhandenen HTTPS-/TLS-Vorgaben ist aktuell keine erneute Grundimplementierung erkennbar. Der SMTP-Punkt wird über DBI-XXX bearbeitet; die übrigen offenen Punkte sind zunächst Nachweisaufgaben und keine bereits bestätigten unverschlüsselten Verbindungen.

Eine vollständige Erfüllungsbestätigung für alle vier Anwendungen ist deshalb noch nicht möglich. Auch der Abschluss von DBI-XXX allein ersetzt die übrigen Nachweise nicht. Bitte stelle mir für den abschließenden Abgleich noch die aktuelle SecSt. 003 zur Verfügung. Den aktualisierten Stand melde ich nach Vorliegen der Nachweise zurück.

Viele Grüße
Daniel
```

## 3. Quellengrundlage zur internen Nachverfolgung

Nicht Bestandteil der Kopierblöcke. Die genannten Zeilennummern beziehen sich auf die gelieferten Snapshots und können im aktuellen Branch abweichen.

| Inhalt | Grundlage |
|---|---|
| Umfang der vier Anwendungen und Abfrage zu Verbindungen, Protokollen und Verschlüsselung | Jacobs abgebildete E-Mail vom 15.09.2026. |
| Jira-Struktur | `DBI-8123.doc`, Abschnitt „Description“. Fachinhalt, Epic, Fix-Version und Sprintangaben der Beispielstory wurden nicht übernommen. |
| SMTP-Fundstelle, mögliche Overrides und Infrastrukturzuständigkeit | `Data_in_Transit_Verbindungsmatrix_und_Nachweise.md`, Q01; `MBPortal_SMTP_Jira_Ticket_und_Antwort_Jacob.md`, technische Hinweise A, B und D. |
| Gemeinsame Plattform und Action Hub | Verbindungsmatrix V01–V03 und V12; Originalauszüge Q02–Q04 und Q10–Q11. |
| MB-Portal, Messaging, Storage und Anbieter | Verbindungsmatrix V06–V09, V11, V13 und V18–V19. |
| Investment Welt Marktbearbeitung | Verbindungsmatrix V04 und V08; Originalauszüge Q16–Q18 und Q20. |
| Verbundschnittstelle Truuco | Verbindungsmatrix V10 und V14–V17; Originalauszüge Q14–Q15, Q21, Q24 und Q29. |
| Übergreifende Betriebsverbindungen und geplante UMP-Anbindung | Verbindungsmatrix V05, V07 und V20–V21. |

Die Zuordnung zu verbindlichen Applikations-/CMDB-IDs sowie die vollständige interne Konformität sind durch diese Quellen nicht abschließend belegt. Die geplanten Arbeitsschritte und Abnahmekriterien sind keine Meldung bereits durchgeführter Maßnahmen.
