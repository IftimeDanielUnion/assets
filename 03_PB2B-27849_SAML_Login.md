# PB2B-27849: Temporären SAML-Login integrieren und abnehmen

| Feld | Wert |
|---|---|
| Arbeitsreihenfolge | **3.** Vorziehen, sobald ein aktueller Blocker für Migrationstests bestätigt ist. |
| Jira-Status / Priorität im Export | `In progress` / `Prio 4` |
| Zuständig laut Export | **Oliver Ehli**, nicht Daniel Iftime. |
| Planstatus | Offen; keine Schritte dieser Checkliste ausgeführt. |
| Standalone im Viererpaket | **Ja.** Keine belegte harte Voraussetzung aus SFTP, Monitoring oder Event Grid. |
| Basiert extern auf | Bereits angelegter temporärer App, passender IdP-Konfiguration, OSC-/Oliver-Abstimmung und zugelassenem Testbenutzer. |
| Hauptansprechpartner laut Verlauf | Oliver Ehli; Integration mit OSC koordinieren. |
| Datenstand | Jira-Export vom 30.09.2026; Code-Snapshot aus dem Upload. |

## Ziel

Der SAML-Login der temporären UMP-Instanz für Tests in Prod funktioniert mit der dafür vorgesehenen App. Der reguläre Produktionslogin bleibt unverändert funktionsfähig.

## Belegter Ausgangsstand

- Das Jira fordert eine temporäre UMP-App für den SAML-Login während der Migrationstests in Prod. Vorgeschlagener Name: `UnionMarketingPlattform Azure Temp`. [J1]
- Oliver meldet am 22.09.2026 die Umsetzung und bittet darum, für das Mellon-Modul die Metadaten der temporären Policy zu hinterlegen. Daher zuerst Integration und Abnahme prüfen, nicht erneut eine App anlegen. [J1]
- Der Kommentar nennt `B2C_1A_signin_saml_ump_tmp`. Die hochgeladene Prod-Metadatendatei verwendet dagegen `B2C_1A_signin_saml_ump`. Das ist ein konkreter Prüfpunkt, aber **kein Beweis für eine fehlerhafte Laufzeitkonfiguration**: Die Datei kann zum regulären Login gehören. [J1, C1]

Metadatenquelle aus dem Jira-Kommentar, vor Verwendung von Oliver bestätigen lassen:

```text
https://auth.user.union-investment.de/b2cunioninvestmentprod.onmicrosoft.com/B2C_1A_signin_saml_ump_tmp/samlp/metadata
```

## Externe Voraussetzungen und mögliche Blocker

| Voraussetzung | Benötigt für | Umgang bei Fehlen |
|---|---|---|
| Oliver bestätigt temporäre App, Policy und Zuständigkeit | Richtige Integrationszuordnung | Keine zweite App auf Verdacht erstellen. |
| Eindeutige temporäre Zielinstanz und Zugriff auf deren Konfiguration | Sichere Mellon-Anpassung | Erst Instanz-/Virtual-Host-Zuordnung klären. |
| Zugelassener Testbenutzer und vereinbarte Attribute | Erfolgreicher Anwendungslogin | Test mit Oliver/OSC vorbereiten. |
| Freigabe für Änderungen im Prod-Testkontext | Deployment | Nur den abgestimmten Änderungsweg verwenden. |

## To-dos in Reihenfolge

### SAML-01: Zuständigkeit, Zielinstanz und aktuellen Blocker klären

**Basiert auf:** keiner anderen Aufgabe. Sofort startbar.

- [ ] Mit Oliver klären, welcher Teil bereits umgesetzt und welcher Integrations-/Testschritt noch offen ist. Übernahme oder gemeinsame Bearbeitung ausdrücklich abstimmen.
- [ ] Bestätigen, ob aktuell Migrationstests blockiert sind. Falls ja, diese Arbeit vor das Monitoring ziehen; sonst reguläre Reihenfolge beibehalten.
- [ ] Temporäre URL, Zielserver/Virtual Host, zugehörige App/Policy, Service-Provider-Konfiguration und tatsächliche Mellon-Dateipfade zuordnen.
- [ ] Prüfen, ob die temporäre Instanz eigene Konfiguration besitzt oder gemeinsame Dateien mit dem regulären Produktionslogin nutzt.
- [ ] Von Oliver/OSC die erwartete Login-Identität und erforderlichen Attribute für den Testbenutzer bestätigen lassen.

### SAML-02: Metadaten und Konfiguration gezielt integrieren

**Basiert auf:** SAML-01.

- [ ] Metadatenquelle der temporären Policy bestätigen und die tatsächlich geladenen IdP-Metadaten damit vergleichen.
- [ ] Zusammenpassende IdP-/SP-Kennungen, Assertion-Consumer-Adresse, Bindings, Signaturzertifikate und erwartete Attribute prüfen.
- [ ] Nur die Konfiguration der bestätigten temporären Instanz ändern. Die reguläre Prod-Metadatendatei nicht pauschal durch die temporäre Variante ersetzen.
- [ ] Prüfen, über welchen Ansible-/Deployment-Pfad die Datei ausgerollt wird; notwendige Zuordnung zum richtigen Host beziehungsweise Stage nachvollziehbar im verwalteten Code pflegen.
- [ ] Vorherigen Stand sichern, Konfiguration prüfen und einen gezielten Reload/Deployment-Schritt mit OSC abstimmen. Keine neue App erzeugen, solange die vorhandene nutzbar ist.

### SAML-03: Temporären Login und regulären Login testen

**Basiert auf:** SAML-02, Testbenutzer und freigegebenem Test.

- [ ] Login von der temporären UMP-Instanz aus durchführen und prüfen, dass die erwartete temporäre App/Policy verwendet wird.
- [ ] Erfolgreiche Rückkehr zur richtigen UMP-Instanz, Benutzerzuordnung und erforderliche Attribute prüfen.
- [ ] Login in einer frischen Sitzung wiederholen, damit eine bestehende Sitzung keinen Fehler verdeckt.
- [ ] Einen abgestimmten negativen Zugriffsfall testen, ohne reale Konten zu sperren; unberechtigter Zugriff darf nicht durch die temporäre Konfiguration ermöglicht werden.
- [ ] Den regulären Produktionslogin als Regressionstest prüfen.
- [ ] Testzeitpunkt, Zielinstanz und bereinigte Diagnosebelege dokumentieren. Keine vollständigen SAML-Assertions oder Sitzungsdaten ins Ticket kopieren.

### SAML-04: Abnahme und temporären Betrieb dokumentieren

**Basiert auf:** SAML-03.

- [ ] Oliver und OSC bestätigen die Integration und den erfolgreichen Test.
- [ ] Dokumentieren, welche App/Policy und welche verwaltete Konfiguration zur temporären Instanz gehören.
- [ ] Verantwortlichen und Auslöser für den späteren Rückbau festlegen. Rückbau nicht während laufender Migrationstests durchführen.
- [ ] Ergebnis an Oliver zur Aktualisierung beziehungsweise Schließung des bestehenden Jiras übergeben oder die abgestimmte Bearbeitung selbst dokumentieren.

## Repository-Einstieg

| Datei im Repository `ansible-ump` | Relevanz |
|---|---|
| `playbooks/configuration/files/app/mellon/prod/idp_metadata_azure.xml` | Enthält im Snapshot die reguläre Policy ohne `_tmp`; besonders Entity-ID und SSO-Einträge in Zeilen 1 und 10-14 prüfen. [C1] |
| `inventories/prod.yml`, Zeile 55 | `mellon_file_stage: "prod"` als Einstieg in die Stage-Zuordnung. Kein Nachweis für die Konfiguration einer gesonderten temporären Instanz. [C2] |
| `playbooks/configuration/files/app/mellon/` | Stage-bezogene Metadaten und SP-Material. Private Schlüssel weder vervielfältigen noch in Testbelege aufnehmen. [C1] |

## Abnahmekriterien

- [ ] Temporäre UMP-Instanz verwendet die bestätigte temporäre App-/Policy-Konfiguration.
- [ ] Berechtigter Testbenutzer gelangt mit korrekter Zuordnung in die Anwendung.
- [ ] Der reguläre Produktionslogin funktioniert weiterhin.
- [ ] Konfigurationsweg, Testergebnisse und spätere Rückbauzuständigkeit sind dokumentiert.
- [ ] Oliver/OSC haben die Abnahme bestätigt.

## Nicht im Umfang / Rückfall

Keine allgemeine SSO-Neugestaltung, keine ungeprüfte Änderung am regulären Prod-Login und keine zweite temporäre App ohne belegten Bedarf.

Bei Regression den vorher gesicherten Stand der betroffenen temporären Konfiguration wiederherstellen und regulären Login erneut prüfen. Eine gemeinsam genutzte Datei darf erst geändert werden, nachdem ihre Auswirkungen auf beide Instanzen geklärt sind.

## Quellen

- **[J1]** `ump_jiras.zip` → `ump_jiras/PB2B-27849.doc`: Beschreibung, Zuständigkeit und Kommentare vom 22.09.2026.
- **[C1]** `UMP_repos.zip` → `UMP_repos/UMP_legacy/ansible-ump/playbooks/configuration/files/app/mellon/`, besonders die Prod-IdP-Metadatendatei.
- **[C2]** Gleiches Repository: `inventories/prod.yml`, Zeile 55.

[Zur Übersicht](README.md)
