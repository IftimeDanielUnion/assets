# DBI-7161: Event Grid für ProductHub planen, aufbauen und integrieren

| Feld | Wert |
|---|---|
| Arbeitsreihenfolge | **4: Umsetzung nach Architekturklärung.** Unterlagenbeschaffung früh parallel anstoßen. |
| Jira-Status / Priorität im Export | `New` / `Prio 4` |
| Zuständig laut Export | Daniel Iftime |
| Planstatus | Offen; keine Schritte dieser Checkliste ausgeführt. |
| Standalone im Viererpaket | **Ja.** Keines der anderen drei Tickets ist als zwingender Vorgänger belegt. |
| Basiert extern auf | Freigegebenem Integrationskonzept, Identitäts-/Netzwerkentscheidungen und ProductHub-/UMP-Implementierung beziehungsweise Testbereitschaft. |
| Hauptansprechpartner laut Export | Sinan Balcin (IT), Maike Schmillen (Fachbereich); Marcel Sebbin für den Codeverweis, OSC für die Anwendungen. |
| Datenstand | Jira-Export vom 30.09.2026; Code-Snapshot aus dem Upload. |

## Ziel

Eine freigegebene Event-Grid-Integration verbindet den ProductHub mit dem vorgesehenen UMP-Webhook. Berechtigte Ereignisse werden zugestellt; nach den vereinbarten Fehler- und Wiederholungsregeln nicht zustellbare Ereignisse sind im Dead-Letter-Storage nachweisbar.

## Belegter Ausgangsstand

- Das Jira verlangt Event Grid, einen Storage Account mit Blob-Container für Dead-Letter-Events sowie Zugriff für ProductHub und UMP. ProductHub 4 wird laut Beschreibung noch außerhalb der Union-Azure-Cloud betrieben. [J1]
- Die Beschreibung nennt B2C-Audiences und Scopes für Publishing und Webhook-Zustellung. Ob diese Angaben mit dem tatsächlich vorgesehenen Dienstmodell und dessen unterstützter Authentifizierung zusammenpassen, ist vor Umsetzung zu verifizieren; sie sind keine bereits geprüfte Bauanleitung. [J1]
- Der Kommentar vom 19.08.2026 verweist auf `web-portale-templates/terraform/eventhub/.../eventhub.tf`. Der Inhalt dieses verlinkten Repositories wurde im Upload nicht mitgeliefert. Aus seinem Namen allein lässt sich seine Eignung nicht ableiten. [J1, U1]
- Das Konzept-PDF `20251205_Konzept_UMP_ProductHub_IV_v2.1.pdf` und `azure-schaubild.png` sind im Jira aufgeführt, fehlen aber als Dateien in `ump_jiras.zip`. Die Ressourcen- und Integrationsplanung ist deshalb noch nicht abschließend belegbar. [J1, U1]

## Externe Voraussetzungen und mögliche Blocker

| Voraussetzung | Benötigt für | Umgang bei Fehlen |
|---|---|---|
| Konzept, Schaubild und tatsächlicher Inhalt des Codeverweises | Belastbare Architekturentscheidung | EG-01 bearbeiten; keine unbelegte Vorlage deployen. |
| ProductHub-Publisher und empfangender UMP-Webhook mit Ansprechpartnern | Schnittstellenvertrag und End-to-End-Test | Verantwortliche und getrennte Anwendungstasks festlegen. |
| Freigegebene Authentifizierung für beide Übertragungsrichtungen | Sichere, unterstützte Kommunikation | Identitätsteam/Architektur einbeziehen; Scopes nicht ungeprüft anlegen. |
| Netzweg vom externen ProductHub sowie zum Webhook und Storage | Zustellung und Fehlerablage | Firewall-/Netzwerkanforderungen konkret aufnehmen. |
| Berechtigungen auf Azure-Ressourcen und IaC-Pipeline | Provisionierung | Zuständigkeit und Rolloutweg klären. |

**Keine künstliche Abhängigkeit:** Das temporäre Browser-SAML-Ticket ist kein Ersatz für Dienstauthentifizierung. Das Exception-Monitoring-Ticket kann beim Betrieb helfen, ist aber kein belegter harter Vorgänger.

## To-dos in Reihenfolge

### EG-01: Unterlagen und Zuständigkeiten vervollständigen

**Basiert auf:** keiner anderen Aufgabe. Sofort startbar; parallel zu den anderen Tickets möglich.

- [ ] Aktuellen Jira-Stand, Konzept-PDF und Schaubild von Sinan/den Verantwortlichen beschaffen; Konzeptversion und tatsächlichen Sollstand bestätigen.
- [ ] Den von Marcel verlinkten Terraform-Code lesen und klären, ob er als passende Vorlage, nur als Referenz oder gar nicht verwendet werden soll.
- [ ] Zuständigkeiten für Azure-Ressourcen, Identitäten, Netzwerk, ProductHub-Publishing, UMP-Webhook und fachliche Abnahme benennen.
- [ ] Gewünschte Umgebungen, aktuelle beziehungsweise zukünftige Hosting-Orte und tatsächlich benötigte Datenflüsse festhalten.
- [ ] Vorhandene Azure-Ressourcen und aktuelle IaC auf bereits umgesetzte Teilstücke prüfen. Fehlende Dateien im ZIP sind kein Beweis für fehlende Live-Ressourcen.

### EG-02: Dienstmodell und Integrationsvertrag freigeben

**Basiert auf:** EG-01. **Gate:** Kein belastbarer Implementierungsplan ohne diese Entscheidung.

- [ ] Vorgesehenen Event-Grid-Ressourcentyp, Publishing-Endpunkt, Zustellmodell und Ereignisschema anhand des Konzepts und aktueller offizieller Dokumentation bestätigen.
- [ ] Für ProductHub → Event Grid und Event Grid → UMP separat festlegen: Identität, Tokenaussteller, Audience, erforderliche Berechtigungen, Credentials-Verwaltung und technische Unterstützung.
- [ ] Die im Jira genannten B2C-/Scope-Annahmen ausdrücklich prüfen und Abweichungen dokumentieren. Nur die für das bestätigte Modell erforderlichen App-Registrierungen und Berechtigungen vorsehen.
- [ ] Netzwerk-/Firewall-Matrix erstellen: Quelle, Ziel, Richtung, Port, DNS/TLS, Erreichbarkeit des Webhooks und Zugriff auf Dead-Letter-Storage.
- [ ] Mit OSC Eventtypen, Pflichtfelder, Versionierung, Ereigniskennung, Antwortverhalten sowie Umgang mit Wiederholungen und mehrfacher Verarbeitung vereinbaren.
- [ ] Zustellvalidierung, Wiederholungs-/Ablaufregeln und Dead-Letter-Verhalten für das gewählte Modell festlegen. Definieren, wie eine Testnachricht gezielt und nachweisbar in Dead Letter gelangt.
- [ ] Entscheidungsprotokoll freigeben lassen. Notwendige ProductHub-/UMP-Codeänderungen als eigene Arbeitspakete mit Verantwortlichen festhalten; ihre Fertigstellung ist Voraussetzung für den Gesamttest.

### EG-03: IaC und notwendige Berechtigungen implementieren

**Basiert auf:** EG-02 und verfügbaren Zugängen.

- [ ] Passenden vorhandenen IaC-Baustein verwenden oder eine gezielte `catalog`-Erweiterung planen; konkrete Modulpfade erst nach Sichtung festlegen.
- [ ] Event-Grid-Ressourcen, relevante Subscription-/Zustellkonfiguration und explizit konfigurierte Dead-Letter-Ablage in einem Blob-Container abbilden. Ein Storage Account allein erfüllt die Fehlerablage nicht.
- [ ] Ausschließlich die für das freigegebene Modell benötigten Identitäten, Berechtigungen und Secret-Referenzen ergänzen.
- [ ] Netzwerk- und Storage-Zugriffe einschließlich Schreibweg für Dead-Letter-Events umsetzen; keine unkoordinierten öffentlichen Freigaben als Testabkürzung.
- [ ] Bei `catalog`-Änderung geprüfte Version bereitstellen und anschließend die vereinbarten Umgebungen über `live` anbinden.
- [ ] Validierung und Plan im regulären Workflow prüfen; Änderungen zunächst nur in der freigegebenen Testumgebung ausrollen.
- [ ] Falls die endgültige Zustellkonfiguration einen bereits erreichbaren und validierbaren Webhook erfordert, deren Aktivierung bis zur Anwendungsbereitschaft zurückstellen. Kein Dummy-Webhook als unbezeichnete Produktionslösung.

### EG-04: Integration und Fehlerfälle nachweisen

**Basiert auf:** EG-03 sowie betriebsbereitem ProductHub-Publisher und UMP-Webhook.

- [ ] Publishing aus dem tatsächlichen ProductHub-Hostingkontext testen, nicht ausschließlich aus dem Azure-Portal oder vom Entwicklerrechner.
- [ ] Ein eindeutig identifizierbares Ereignis bis zum vorgesehenen UMP-Webhook verfolgen und erfolgreiche Verarbeitung bestätigen lassen.
- [ ] Unberechtigtes Publishing beziehungsweise unberechtigte Webhook-Aufrufe über freigegebene Negativtests prüfen.
- [ ] Webhook-Zustellvalidierung und Wiederholungsfälle entsprechend dem freigegebenen Dienstmodell prüfen.
- [ ] Einen definierten Zustellfehler erzeugen und nach Ablauf der vereinbarten Wiederholungs-/Ablaufregeln den konkreten Dead-Letter-Eintrag im Blob-Container nachweisen.
- [ ] Wiederherstellung und kontrollierte Wiederverarbeitung eines fehlgeschlagenen Ereignisses mit OSC testen, soweit fachlich vorgesehen.
- [ ] Prüfen, dass eine erneute Zustellung nicht zu einer unbeabsichtigten doppelten fachlichen Verarbeitung führt; erwartetes Verhalten dokumentieren.
- [ ] Für alle Tests Ereigniskennung, Umgebung, Zeitpunkte, Sender-, Zustell- und Empfangsbelege ohne Tokens oder Secrets dokumentieren.

### EG-05: Betriebsübergabe und weitere Umgebungen

**Basiert auf:** EG-04.

- [ ] Abnahme mit IT, Fachbereich und OSC dokumentieren; Ausnahmen ausdrücklich entscheiden lassen.
- [ ] Vorgehen bei fehlgeschlagener Zustellung und Dead-Letter-Einträgen, Zuständigkeiten, Aufbewahrung und Wiederverarbeitung dokumentieren.
- [ ] Benötigte Betriebsüberwachung für diese Integration festlegen. Sie kann vorhandene Monitoring-Bausteine nutzen, ohne DBI-6455 künstlich zur Projektvoraussetzung zu machen.
- [ ] Weitere vereinbarte Umgebungen nach Freigabe ausrollen und je Umgebung einen geeigneten Zustell-/Fehlernachweis führen.
- [ ] Ursprüngliche Jira-Kriterien einzeln mit Test- und Ressourcenbelegen abschließen; nicht allein die Existenz von Ressourcen abnehmen.

## Repository-Einstieg

| Bereich | Geplanter Einsatz |
|---|---|
| `catalog` und `live` | Infrastrukturbausteine und Umgebungsintegration; konkreten Baustein erst nach EG-01/EG-02 festlegen. |
| `credentials` | Einbeziehen, wenn der abgestimmte Identitäts-/Secret-Verwaltungsweg dies vorsieht. Keine Pflichtänderung allein wegen des Repository-Namens. |
| `ui-producthub` | Mit OSC klären, ob der hochgeladene Stand dem im Jira genannten ProductHub 4 entspricht und wo der Publisher entsteht. |
| `unioninvestment` | Mit OSC den tatsächlich vorgesehenen UMP-Webhook und dessen Authentifizierung lokalisieren beziehungsweise als Anwendungstask planen. |

Diese Tabelle benennt Arbeitshypothesen für die Repository-Zuordnung, keine bereits nachgewiesenen Event-Grid-Implementierungsstellen.

## Abnahmekriterien

- [ ] Das freigegebene Event-Grid-Modell und die benötigten Azure-Ressourcen sind per IaC bereitgestellt.
- [ ] ProductHub kann aus seinem tatsächlichen Hostingkontext autorisiert publizieren.
- [ ] Event Grid erreicht den UMP-Webhook über den freigegebenen Netz- und Authentifizierungsweg; UMP verarbeitet das Testereignis.
- [ ] Definiert nicht zustellbare Ereignisse sind im richtigen Dead-Letter-Container nachgewiesen.
- [ ] Positive, negative und Wiederholungstests entsprechen dem abgestimmten Vertrag.
- [ ] Betrieb, Verantwortliche, Testbelege und Gesamtabnahme sind dokumentiert.

## Nicht im Umfang / Rückfall

Kein ungeprüftes Ausrollen der verlinkten `eventhub.tf`, keine pauschale Übernahme von Scopes und kein vollständiger Neuaufbau der Identitätsplattform.

Vor dem Rollout mit OSC einen Stopp-/Rückfallweg festlegen: Publisher anhalten oder auf den vorherigen freigegebenen Pfad zurücksetzen, Zustellkonfiguration kontrolliert zurücknehmen und bereits angenommene beziehungsweise fehlgeschlagene Ereignisse sichern. Dead-Letter-Daten nicht als Teil eines pauschalen Infrastruktur-Rollbacks löschen.

## Quellen

- **[J1]** `ump_jiras.zip` → `ump_jiras/DBI-7161.doc`: Beschreibung, Akzeptanzkriterien, Anhangsliste, Ansprechpartner und Kommentar vom 19.08.2026.
- **[U1]** Inhaltsverzeichnisse der hochgeladenen Archive: `ump_jiras.zip` enthält vier Jira-Exportdateien; das verlinkte externe Terraform-Repository und die genannten Konzeptanhänge sind nicht als solche mitgeliefert.

[Zur Übersicht](README.md)
