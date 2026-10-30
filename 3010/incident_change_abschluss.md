# ServiceNow: Incident und Change-Abschluss

## Incident UI-INC-0466999

**Titel:** BFF PROD: Key-Vault-Referenzen nach Managed-Identity-Umstellung nicht aufgelöst

**Beschreibung:**
Im Rahmen des Changes UI-CHG-0100007 zur Einrichtung eines Deployment Slots für die BFF in PROD traten Probleme bei der Auflösung der Key-Vault-Referenzen auf. Nach dem Löschen und erneuten Anlegen der User Assigned Managed Identity erhielt diese eine neue Principal-ID. Ein gespeicherter Terraform-Plan verwendete weiterhin die alte Principal-ID, sodass die Key-Vault-Rollenzuweisung fehlschlug. Zusätzlich traten im Deployment Berechtigungsfehler auf. Die Function App war bereits auf die User Assigned Identity konfiguriert.

**Abschlusscode:** Lösung bereitgestellt

**Lösungsbeschreibung (Rückmeldung an Kunden):**
Die Berechtigung `Key Vault Secrets User` wurde erfolgreich für die aktuell gültige Managed Identity eingerichtet. Anschließend wurde die `keyVaultReferenceIdentity` der Function App per Azure CLI kurz auf `SystemAssigned` und wieder auf die vorgesehene User Assigned Identity gesetzt. Die anschließende Prüfung ergab für alle sechs Key-Vault-Referenzen den Status `Resolved`. Die ursprüngliche Identity-Konfiguration ist wiederhergestellt.

## Change UI-CHG-0100007

**Abschlusscode:** Erfolgreich mit Problemen

**Abschlussnotizen / Abschlussprüfung:**
Bei der Umsetzung des Changes zur Erstellung eines BFF-Deployment-Slots in PROD kam es zu Problemen mit der Managed Identity und der Key-Vault-Berechtigungszuweisung. Nach der Neuanlage der Identity wurde in einem gespeicherten Terraform-Plan noch die alte Principal-ID verwendet. Die Berechtigungszuweisung wurde anschließend erfolgreich für die neue Principal-ID vorgenommen. Durch erneutes Setzen der Key-Vault-Referenz-Identity auf die vorgesehene User Assigned Identity konnten alle sechs Key-Vault-Referenzen erfolgreich aufgelöst werden (`Resolved`). Die technische Prüfung der Key-Vault-Referenzen war erfolgreich; die Störung wurde behoben und im Incident UI-INC-0466999 dokumentiert.

**Hinweis zur vollständigen Change-Abnahme:**
Der vorliegende Nachweis bestätigt die Auflösung der sechs Key-Vault-Referenzen. Die Erstellung des Deployment Slots und der fachliche Funktionstest der Anwendung sollten vor dem endgültigen Change-Abschluss separat anhand der Deployment- und Testergebnisse bestätigt werden.
