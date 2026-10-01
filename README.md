# UMP: To-do-Tickets und Arbeitsreihenfolge

**Planungsstand:** 01.10.2026  
**Grundlage:** Vier Jira-Exporte vom 30.09.2026 aus `ump_jiras.zip` und der Code-Snapshot aus `UMP_repos.zip`.

Die Dateien sind Arbeitspläne zu bestehenden Jiras. Es wurden keine Jira-Tickets geändert und keine Infrastrukturänderungen ausgeführt. Alle Checklisten sind offen. Angaben zum bisherigen Fortschritt stammen aus den Exporten, nicht aus einer Prüfung der laufenden Systeme.

## 1. Empfohlene Reihenfolge

| Reihenfolge | Ticket und Datei | Jira-Priorität im Export | Erster Schritt | Standalone gegenüber den anderen drei Tickets? |
|---|---|---|---|---|
| **1** | [DBI-11149: SFTP-Verbindungen](01_DBI-11149_SFTP.md) | Prio 1 | Aktuellen Lukas-Stand klären; Hinterlegung des bereits gelieferten Public-Keys bestätigen lassen. | **Ja.** Externe Gegenstelle und OSC werden für die Abnahme benötigt. |
| **2** | [DBI-6455: Exception Monitoring](02_DBI-6455_Exception_Monitoring.md) | Prio 2 | Tatsächliche Logdaten und bestehenden Alert-Stand prüfen; anschließend E-Mail-Alarmierung implementieren. | **Ja.** Funktionierende Logaufnahme und die Abstimmung mit OSC sind Voraussetzungen. |
| **3** | [PB2B-27849: Temporärer SAML-Login](03_PB2B-27849_SAML_Login.md) | Prio 4 | Mit Oliver/OSC prüfen, welche temporäre Instanz welche Mellon-Metadaten verwendet. | **Ja.** Die angelegte temporäre App und passende Zugänge müssen verfügbar sein. |
| **4** | [DBI-7161: Event Grid für ProductHub](04_DBI-7161_Event_Grid.md) | Prio 4 | Fehlende Konzeptunterlagen und referenzierten Code beschaffen; Authentifizierung und Zustellung freigeben lassen. | **Ja.** Umsetzung und Gesamtabnahme benötigen Architekturentscheidungen sowie ProductHub-/UMP-Zuarbeit. |

**Die Nummerierung ist eine Arbeitspriorisierung, keine technische Abhängigkeitskette.** Kein Abschluss eines dieser vier Tickets ist in den vorliegenden Quellen als zwingende Voraussetzung für ein anderes dieser vier Tickets belegt.

**Ausnahme bei der Priorisierung:** Blockiert der temporäre SAML-Login aktuell die Produktionstests der Migration, wird sein Integrationscheck vorgezogen. Das ist eine bedingte Prioritätsänderung, keine nachgewiesene Abhängigkeit der gesamten Monitoring- oder SFTP-Umsetzung.

## 2. Was „standalone“ hier bedeutet

**Standalone im Ticketpaket:** Du kannst das Ticket beginnen, ohne eines der anderen drei Tickets abgeschlossen zu haben.

**Nicht gleichbedeutend mit „ohne Zuarbeit abschließbar“:** Eine Gegenstelle kann einen Schlüssel hinterlegen müssen, OSC muss einen Test bestätigen oder ein Anwendungsteam muss einen Webhook bereitstellen. Diese Voraussetzungen stehen im jeweiligen Ticket ausdrücklich unter „Externe Voraussetzungen“.

### Abhängigkeiten zwischen den Tickets

| Beziehung | Harte Abhängigkeit? | Konsequenz |
|---|---|---|
| SFTP ↔ Monitoring | Nicht belegt. | Monitoring weiterbearbeiten, während eine SFTP-Rückmeldung aussteht. |
| SAML → Monitoring | Nicht für die Infrastrukturumsetzung belegt. | Ein gewählter browserbasierter Test könnte den Login benötigen; ein vereinbarter serverseitiger Test muss deshalb nicht warten. |
| SAML → SFTP | Nicht für Verbindungs- und Transfertests aus dem Laufzeitkontext belegt. | Nur ein zusätzlich gewählter UI-Test könnte vom Login abhängen. |
| Monitoring → Event Grid | Nicht als Jira-Voraussetzung belegt. | Gemeinsame Überwachung ist eine sinnvolle Betriebsoption, aber kein künstlicher Startblocker. |
| SAML → Event Grid | Nicht belegt. | Browser-SAML und die Authentifizierung zwischen Diensten getrennt behandeln. |
| Gemeinsame `live`-/`catalog`-Änderungen | Mögliche Arbeitsüberschneidung, keine fachliche Ticketabhängigkeit. | PRs und Deployments koordinieren; nicht gleichzeitig auf denselben Terraform-State anwenden. |

Im Monitoring-Export ist außerdem ein ausgehender Link „blockiert DBI-480“ enthalten; DBI-480 steht dort auf `Rejected`. Das ist weder ein dokumentierter Vorgänger von DBI-6455 noch eine Abhängigkeit innerhalb dieses Viererpakets. Die aktuelle Gültigkeit dieses Jira-Links muss bei Bedarf geprüft werden. [Quelle: `ump_jiras/DBI-6455.doc`, Abschnitt „Issue Links“.]

## 3. Konkreter Arbeitsablauf

1. **SFTP-01 und SFTP-02 anstoßen.** Aktuellen Stand sichern, Kontakt mit der Gegenstelle koordinieren und bestehende Schlüsselhinterlegung klären.
2. **Während externer Wartezeit MON-01 und MON-02 bearbeiten.** Daten prüfen und den Monitoring-PR vorbereiten. Nicht auf den vollständigen Abschluss von SFTP warten.
3. **SAML-01 mit Oliver/OSC klären.** Bei einem bestätigten Migrationstest-Blocker vorziehen; sonst nach dem Monitoring-Start bearbeiten.
4. **EG-01 früh anstoßen.** Konzept und Verantwortliche können parallel geklärt werden. Die Event-Grid-Implementierung erst nach EG-02 beginnen.
5. **Rückmeldungen gezielt abarbeiten.** SFTP- und SAML-Abnahmen abschließen, Monitoring ausrollen und danach die freigegebene Event-Grid-Integration umsetzen.

Die Bezeichnungen `SFTP-01`, `MON-01`, `SAML-01` und `EG-01` sind lokale Arbeitspaket-IDs innerhalb dieser Markdown-Dateien, keine neu angelegten Jira-Tickets.

## 4. Abhängigkeiten innerhalb der Tickets

```text
DBI-11149
  SFTP-01 → SFTP-02 → SFTP-03 ─┐
          → SFTP-04 ──────────┴→ SFTP-05

DBI-6455
  MON-01 → MON-02 → MON-03 → MON-04

PB2B-27849
  SAML-01 → SAML-02 → SAML-03 → SAML-04

DBI-7161
  EG-01 → EG-02 → EG-03 → EG-04 → EG-05
```

Bei einer notwendigen `catalog`-Erweiterung gilt innerhalb des betroffenen Tickets zusätzlich: Moduländerung prüfen und versionieren, dann die passende Version in `live` referenzieren. Diese technische Reihenfolge ist nicht mit einer Abhängigkeit zu einem anderen Jira zu verwechseln.

## 5. Quellen und Nutzung

Jede Ticketdatei enthält Ziel, bisherigen Stand, externe Voraussetzungen, geordnete Checklisten, Abnahmekriterien, Rückfallhinweise und konkrete Quellen.

Repository-Pfade beginnen beim jeweiligen Repository. Im ZIP liegen sie unter `UMP_repos/UMP_legacy/`. Zeilennummern beziehen sich auf den hochgeladenen Snapshot und können sich nach Updates verschieben.

**Prüfregel vor der ersten Änderung:** Aktuellen Jira-Stand, Branch-/Paketstand und Zielumgebung mit den Angaben dieser Dateien abgleichen. Fehlende Belege nicht als erledigte Arbeit interpretieren.
