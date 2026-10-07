# DBI-12045 - CIS Change DEV - RBAC für APIM auf Key Vault

## Zweck

Dieser Change vergibt die fehlende Azure-RBAC-Berechtigung für die **System Assigned Managed Identity** der DEV-APIM-Instanz auf dem DEV-Key-Vault.

Die Firewall-Freigabe läuft separat über **UI-CHG-0097506**.

Es ist **keine Terraform-Änderung** und **keine Access Policy** im Key Vault erforderlich.

---

## Change-Titel / Short description

```text
[AZURE][DEV][RBAC] APIM Zugriff auf Key Vault für Impulsmanager Client-Zertifikat
```

Alternative, falls eure Namenskonvention CIS im Titel verlangt:

```text
[CIS][AZURE][DEV][RBAC] APIM Key Vault Secrets User für Impulsmanager
```

---

## Beschreibung / Description

```text
Im Rahmen von DBI-12045 benötigt die DEV-APIM-Instanz uiapimf5c41c8714ee Zugriff auf den DEV-Key-Vault kv-21d79b9b0ff404b6dcba.

APIM muss das Client-Zertifikat impulsmanager-dev-client-cert aus dem Key Vault lesen können, um die Authentifizierung gegenüber der Impulsmanager-Schnittstelle zu ermöglichen.

Der Key Vault verwendet Azure RBAC als Berechtigungsmodell. Daher soll für die System Assigned Managed Identity der APIM-Instanz die Rolle "Key Vault Secrets User" auf Scope des DEV-Key-Vaults vergeben werden.

Die Netzwerkfreigabe wird separat über den bestehenden Firewall-Change UI-CHG-0097506 behandelt.
```

---

## Business Justification / Begründung

```text
Die Berechtigung wird benötigt, damit APIM das für die Impulsmanager-Anbindung erforderliche Client-Zertifikat aus dem DEV-Key-Vault abrufen kann.

Ohne die RBAC-Berechtigung kann die APIM-Instanz nicht auf das im Key Vault hinterlegte Zertifikat zugreifen und die vorgesehene Authentifizierung gegenüber der Impulsmanager-Schnittstelle kann nicht durchgeführt werden.

Referenz: DBI-12045
```

---

## Technische Daten

### Umgebung

```text
DEV
```

### Azure-Ressource / Ziel

```text
Key Vault:
kv-21d79b9b0ff404b6dcba
```

### APIM

```text
APIM:
uiapimf5c41c8714ee
```

### Identität

```text
Identity Type:
System Assigned Managed Identity
```

### Principal / Object ID

```text
d2f189cb-814b-4d25-ae88-1b6552f464a0
```

### Benötigte RBAC-Rolle

```text
Key Vault Secrets User
```

### Scope

```text
Scope auf dem DEV-Key-Vault:
kv-21d79b9b0ff404b6dcba
```

### Verwendetes Zertifikat

```text
impulsmanager-dev-client-cert
```

### Jira / Fachliche Referenz

```text
DBI-12045
```

### Zugehöriger Firewall-Change

```text
UI-CHG-0097506
```

---

## Implementierungsplan / Implementation Plan

```text
1. Prüfen, dass die Zielressource kv-21d79b9b0ff404b6dcba der DEV-Key-Vault für DBI-12045 ist.

2. Prüfen, dass die Principal/Object ID
   d2f189cb-814b-4d25-ae88-1b6552f464a0
   weiterhin zur System Assigned Managed Identity der APIM-Instanz
   uiapimf5c41c8714ee gehört.

3. Auf Scope des Key Vaults kv-21d79b9b0ff404b6dcba folgende Azure-RBAC-Rollenzuweisung erstellen:

   Principal:
   d2f189cb-814b-4d25-ae88-1b6552f464a0

   Role:
   Key Vault Secrets User

4. Keine Änderung am Key-Vault-Berechtigungsmodell vornehmen.

5. Keine zusätzliche Key Vault Access Policy anlegen.

6. Nach erfolgreicher Rollenzuweisung die Umsetzung dokumentieren.

7. Nach Abschluss der separaten Netzwerkfreigabe UI-CHG-0097506 gemeinsam mit dem APIM-Team den Abruf des Zertifikats impulsmanager-dev-client-cert testen.
```

---

## Testplan / Test Plan

```text
1. Prüfen, dass am Key Vault kv-21d79b9b0ff404b6dcba die Rolle
   "Key Vault Secrets User" für die Principal/Object ID
   d2f189cb-814b-4d25-ae88-1b6552f464a0
   vorhanden ist.

2. Sicherstellen, dass UI-CHG-0097506 bzw. die erforderliche Netzwerkfreigabe abgeschlossen ist.

3. In APIM den Zugriff auf die Key-Vault-Zertifikatsreferenz für
   impulsmanager-dev-client-cert auslösen bzw. aktualisieren.

4. Prüfen, dass APIM das Zertifikat erfolgreich aus dem Key Vault lesen kann.

5. Sofern die Impulsmanager-API bereits vollständig konfiguriert ist, einen DEV-Testrequest durchführen und prüfen, dass keine Key-Vault-/Berechtigungsfehler auftreten.
```

---

## Erfolgskriterium / Validation

```text
Die System Assigned Managed Identity der APIM-Instanz uiapimf5c41c8714ee besitzt auf dem DEV-Key-Vault kv-21d79b9b0ff404b6dcba die Rolle "Key Vault Secrets User".

APIM kann das Zertifikat impulsmanager-dev-client-cert erfolgreich aus dem Key Vault abrufen.

Es treten keine Authorization-/Forbidden-Fehler beim Zugriff von APIM auf den Key Vault auf.
```

---

## Rollback Plan

```text
Falls durch die Änderung unerwartete Auswirkungen auftreten, wird die im Rahmen dieses Changes erstellte RBAC-Rollenzuweisung wieder entfernt:

Principal/Object ID:
d2f189cb-814b-4d25-ae88-1b6552f464a0

Role:
Key Vault Secrets User

Scope:
kv-21d79b9b0ff404b6dcba

Weitere Einstellungen des Key Vaults werden nicht verändert.
```

---

## Risiko / Risk

```text
Gering.

Es wird ausschließlich eine zusätzliche RBAC-Leseberechtigung für Secrets auf dem definierten DEV-Key-Vault vergeben.

Es erfolgen keine Änderungen am Key-Vault-Berechtigungsmodell, keine Änderungen an bestehenden Rollen und keine Änderungen an bestehenden Secrets oder Zertifikaten.
```

---

## Auswirkung / Impact

```text
Keine erwartete Serviceunterbrechung.

Die Änderung erweitert ausschließlich die Berechtigung der APIM Managed Identity auf den DEV-Key-Vault.
```

---

## Downtime

```text
Keine Downtime erwartet.
```

---

## Security / Berechtigungsumfang

```text
Die Rolle "Key Vault Secrets User" wird auf Scope des DEV-Key-Vaults vergeben.

Damit erhält die angegebene APIM Managed Identity Leseberechtigung auf Secrets innerhalb dieses Key Vaults.

Es wird keine Key Vault Access Policy angelegt und das vorhandene RBAC-Berechtigungsmodell wird nicht verändert.
```

---

## Change-Abhängigkeit

```text
Für den vollständigen End-to-End-Test wird zusätzlich die Netzwerkverbindung zwischen APIM und dem Private Endpoint des Key Vaults benötigt.

Diese wird separat über UI-CHG-0097506 behandelt.

Der CIS-RBAC-Change und der Firewall-Change sind technisch getrennte Änderungen.
```

---

## Kommunikation / Work Notes

```text
RBAC-Berechtigung für DBI-12045:

APIM:
uiapimf5c41c8714ee

System Assigned Managed Identity / Principal ID:
d2f189cb-814b-4d25-ae88-1b6552f464a0

Ziel:
Key Vault kv-21d79b9b0ff404b6dcba

Rolle:
Key Vault Secrets User

Zertifikat:
impulsmanager-dev-client-cert

Firewall separat:
UI-CHG-0097506
```

---

## Nicht Bestandteil dieses Changes

```text
- Keine Terraform-Änderung im ui-mb-infrastructure Repository
- Keine Firewall-Regel
- Keine Änderung des Key-Vault-Berechtigungsmodells
- Keine neue Key Vault Access Policy
- Keine Änderung oder Erneuerung des Zertifikats
- Keine TEST- oder PROD-Änderung
```

---

## Falls ServiceNow ein Feld "Assignment Group" verlangt

Die konkrete CIS-/Azure-RBAC-Zuweisungsgruppe bitte aus einem bestehenden CIS-RBAC-Change übernehmen oder über den CIS-Servicekatalog automatisch ermitteln lassen.

Nicht raten, wenn keine gültige Vorlage vorliegt.

---

## Falls ServiceNow ein Feld "Configuration Item / CI" verlangt

Bevorzugt die konkrete Azure-/Key-Vault-Zuordnung aus dem CIS-Katalog auswählen.

Technisches Ziel:

```text
kv-21d79b9b0ff404b6dcba
```

---

## Kurzfassung zum Copy & Paste

```text
Für DBI-12045 soll die System Assigned Managed Identity der DEV-APIM-Instanz uiapimf5c41c8714ee auf dem DEV-Key-Vault kv-21d79b9b0ff404b6dcba die Azure-RBAC-Rolle "Key Vault Secrets User" erhalten.

Principal/Object ID:
d2f189cb-814b-4d25-ae88-1b6552f464a0

Benötigtes Zertifikat:
impulsmanager-dev-client-cert

Scope:
DEV-Key-Vault kv-21d79b9b0ff404b6dcba

Die Netzwerkfreigabe wird separat über UI-CHG-0097506 behandelt.

Keine Terraform-Änderung und keine zusätzliche Key Vault Access Policy.
```
