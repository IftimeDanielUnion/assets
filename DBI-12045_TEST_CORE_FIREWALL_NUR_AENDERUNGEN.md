# DBI-12045: TEST CORE-FIREWALL | Nur Änderungen am bereits angelegten Change

Diese Ergänzungen beziehen sich auf die zuvor erstellte TEST-Vorlage. **Nicht den gesamten Change neu ausfüllen.** Alle anderen Felder bleiben unverändert.

## 1. Feld: Begründung

In der vorhandenen Begründung **nur** die bisherige Angabe zur fehlenden Private-Endpoint-IP ersetzen:

**Alter Satzteil:** `zur noch zu verifizierenden privaten IP des genannten Key-Vault-Private-Endpoints`

**Neuer Satzteil (kopieren):**

```text
zur im Azure-Private-Endpoint-Export ausgewiesenen Ziel-IP 172.28.59.9/32
```

**Alte Zeile:** `Ziel-Private-Endpoint-IP: noch im Azure Portal/über IBA für TEST zu bestätigen, vor Freigabe zwingend nachzutragen`

**Neue Zeile (kopieren):**

```text
Ziel-Private-Endpoint-IP: 172.28.59.9/32 (aus customDnsConfigs des Azure-Private-Endpoint-Exports)
```

Die übrige Begründung einschließlich Bestandsprüfung `UI-CHG-0041935` unverändert lassen.

## 2. Feld: Zusätzliche Informationen

Den bisherigen Satz, der **private IP und Status als unbestätigt** bezeichnet, durch diesen Absatz ersetzen:

```text
Azure-Private-Endpoint-Daten für TEST sind nun vorhanden:
Private Endpoint: pe-im-action-hub-test-secrets-internal-kv
Key Vault: kv-250fc9d46bacfd08c783
FQDN: kv-250fc9d46bacfd08c783.vault.azure.net
Private Ziel-IP: 172.28.59.9/32 (customDnsConfigs des Private Endpoints)
Private-Link-Verbindung: Approved
Provisioning: Succeeded
Subscription: d98221c3-b24d-4770-a9a3-3716667d7353
Ziel-VNet/Subnetz: vnet-marktbearbeitung-test-swc-01 / privateendpoints

Weiterhin durch IBA zu prüfen: passende Ziel-EPG beziehungsweise neue Einzelziel-EPG, tatsächliche Bestandsregel-Abdeckung einschließlich UI-CHG-0041935, Firewall-Deployment und Routing. Die APIM-Quell-EPG sowie Change-Fenster und Freigaben sind ebenfalls im regulären Prozess zu bestätigen. Keine Doppelregel erstellen.
```

## 3. Feld: Risiko- und Auswirkungsanalyse

**Nur** den bisherigen Satzteil `Die Ziel-IP ist aktuell noch nicht belegt. Ohne diese Verifikation keine genehmigte Regel implementieren.` ersetzen durch:

```text
Die Ziel-IP 172.28.59.9 ist im Azure-Private-Endpoint-Export ausgewiesen; die korrekte Ziel-EPG, ausgerollte Firewall-Abdeckung und Live-Quellnetz-Zuordnung sind weiterhin durch IBA zu prüfen.
```

## Falls die bereits gespeicherten Felder nicht mehr editierbar sind

Diese **eine Arbeitsnotiz** genügt als technischer Nachtrag:

```text
DBI-12045, CORE-FIREWALL TEST: Private-Endpoint-Daten ergänzt. Für kv-250fc9d46bacfd08c783.vault.azure.net ist im Azure-Export die private Ziel-IP 172.28.59.9/32 angegeben (PE pe-im-action-hub-test-secrets-internal-kv, Verbindung Approved, Provisioning Succeeded). Bitte bei der Regel-/Ziel-EPG-Prüfung berücksichtigen. Quell-EPG und -Subnetz bleiben gemäß ursprünglichem Change; Bestandsregel UI-CHG-0041935 vor neuer Regelanlage prüfen. CIS-RBAC erfolgt separat.
```

**Nicht geändert:** Titel, Quelle `10.250.223.128/27`, Port TCP 443, Zuständigkeit, Implementierungsplan, Qualitätssicherung, Rollback, CIS-Abgrenzung. Die Private-Endpoint-IP ist nun belegt, eine erfolgreiche Firewall-Verbindung damit aber noch nicht nachgewiesen.