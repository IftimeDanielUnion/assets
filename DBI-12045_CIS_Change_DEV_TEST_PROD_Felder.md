# DBI-12045: CIS-Change / RBAC-Zuweisungen DEV, TEST und PROD

**Stand:** 08.10.2026  
**Zweck:** Kopiervorlage für den **CIS-Anteil**. In diesem Dokument stehen ausschließlich die von CIS beantragten, auszuführenden und zu prüfenden **Azure-RBAC-Arbeiten**. Netzwerk-, APIM-Zertifikats- und Anwendungstests durch den Antragsteller gehören **nicht** in diesen Change; sie stehen in `DBI-12045_Nach_CIS_Eigene_Pruefungen.md`.

**Vorgang:** DBI-12045 (Parent: DBI-8123). **Leistungsweg:** CIS-Servicekatalog **RBAC assignment** (entsprechend dem bestehenden Cross-Subscription-Vorgehen, Referenz DBI-9172).

> **Wichtig:** Es sind drei **getrennte** Rollenzuweisungen an drei verschiedenen Vaults, keine pauschale Berechtigung auf Subscription- oder Resource-Group-Ebene. Falls der CIS-Katalog nur eine Principal-/Scope-Kombination pro Bestellposition erlaubt: drei Positionen bzw. Bestellungen unter DBI-12045 erfassen. Wenn ein gemeinsamer Change mit drei Umgebungen nicht erlaubt ist, die Zuordnungen in drei Changes aufteilen.

## 1. Felder für den gemeinsamen CIS-Change (Copy & Paste)

### Katalogeintrag

```text
RBAC assignment
```

### Kurzbeschreibung / Titel

```text
[AZURE][RBAC][APIM][KEYVAULT][DEV/TEST/PROD] Key Vault Secrets User für APIM-Identitäten (DBI-12045)
```

### Beschreibung / Was soll CIS durchführen?

```text
Im Rahmen von DBI-12045 sollen für DEV, TEST und PROD insgesamt drei voneinander getrennte Azure-RBAC-Zuweisungen erstellt werden. Ziel ist der lesende Secret-Zugriff der jeweiligen Azure API Management (APIM)-Managed Identity auf den zugehörigen IM-Action-Hub-Key-Vault.

Rolle: Key Vault Secrets User
Role Definition ID: 4633458b-17de-408a-b874-0445c86b69e6
Principal-Typ: ServicePrincipal / System-assigned Managed Identity
Tenant ID: 2eb07b91-f9f8-437b-a89e-59f55e565c11
Scope: ausschließlich der jeweilige Key Vault (Vault-Ressourcenebene).

DEV:
APIM: uiapimf5c41c8714ee
APIM-Principal/Object ID: d2f189cb-814b-4d25-ae88-1b6552f464a0
APIM-Subscription: S_APIMANAGEMENT_DEV / 1da614d7-0e79-4508-83be-95ee7b55f02b
Ziel-Vault: kv-21d79b9b0ff404b6dcba
Vault-Subscription: S_MARKTBEARBEITUNG_DEV / 6289fbfa-0b84-4f9e-8250-684455e9bb1a

TEST:
APIM: uiapim125ce218d1e3
APIM-Principal/Object ID laut Repository: 44fd1bdb-45fe-4fc8-9351-0b05bd28f794
APIM-Subscription: S_APIMANAGEMENT_TEST / 14557421-c8d7-430d-8fc3-62e217d47033
Ziel-Vault: kv-250fc9d46bacfd08c783
Vault-Subscription: S_MARKTBEARBEITUNG_TEST / d98221c3-b24d-4770-a9a3-3716667d7353

PROD:
APIM: uiapim2b80e30deb7e
APIM-Principal/Object ID laut Repository: 53c9b7c3-50e1-4a8d-b45b-aa10ed1882b9
APIM-Subscription: S_APIMANAGEMENT_PROD / 6ae0ab8b-523e-450a-86b4-b253b4cfb40a
Ziel-Vault: kv-17cdd303c9bb2e255af9
Vault-Subscription: S_MARKTBEARBEITUNG_PROD / a6c520e0-a1af-4c4c-82f0-8482ff82ecd8

Bitte die aktuellen Principal IDs und die vollständigen Key-Vault-Resource-IDs vor Umsetzung gegen Azure abgleichen, bestehende effektive Berechtigungen berücksichtigen und ausschließlich fehlende Zuweisungen vornehmen. DEV ist durch Azure-Exporte belegt; die TEST-/PROD-Principal IDs sind bislang nur im Repository nachgewiesen.

Keine Access Policies anlegen, das Vault-Berechtigungsmodell nicht ändern und keine höherliegenden RBAC-Scopes verwenden. Netzwerk-/Firewall-Änderungen sind nicht Bestandteil dieses CIS-Auftrags.
```

### Begründung / Business Justification

```text
Die APIM-Instanzen benötigen lesenden Zugriff auf die Secrets der jeweiligen IM-Action-Hub-Key-Vaults, damit Client-Zertifikate für die Impulsmanager-Anbindung verwendet werden können. Der Zugriff erfolgt per Azure Managed Identity und wird je Umgebung auf den zugehörigen Key Vault beschränkt.

Die Berechtigung soll über das bestehende CIS-Verfahren für Cross-Subscription-RBAC vergeben werden. Beantragt wird Key Vault Secrets User am jeweiligen Vault. Der Umfang beinhaltet damit Lesezugriff auf alle Secrets des jeweiligen Vaults, nicht ausschließlich auf ein einzelnes Zertifikat. Dieser Berechtigungsumfang ist Bestandteil der fachlichen Freigabe.
```

### Implementierungsplan (nur CIS)

```text
1. Für DEV, TEST und PROD die beantragte Principal-/Vault-Kombination sowie die erforderliche Freigabe für den Vault-weiten Secrets-Lesezugriff prüfen.
2. In Azure je Umgebung prüfen, ob die angegebene Managed-Identity-Principal-ID aktuell existiert und zur vorgesehenen APIM-Instanz gehört. Insbesondere die TEST- und PROD-IDs gegen den Live-Stand abgleichen. Bei Abweichungen nicht zuweisen, sondern Rückfrage an den Antragsteller stellen.
3. Pro Umgebung die vollständige Key-Vault-Resource-ID, Subscription und das RBAC-Berechtigungsmodell prüfen. Bei Unklarheit oder Abweichung nicht umsetzen.
4. Vorhandene direkte, geerbte und soweit nachvollziehbar gruppenbasierte gleichwertige Berechtigungen prüfen; keine unnötigen Doppelzuweisungen erstellen.
5. Für jede noch fehlende und genehmigte Kombination die Rolle Key Vault Secrets User ausschließlich am jeweiligen Key-Vault-Scope an die bestätigte Principal-ID zuweisen.
6. Je Umgebung die tatsächlich erstellte Role-Assignment-ID, den Scope, die Principal-ID, die Rolle und den Umsetzungszeitpunkt dokumentieren. Falls bereits ausreichende Berechtigungen bestehen, den Befund anstelle einer neuen Zuweisung dokumentieren.
```

### Prüfplan / Testplan durch CIS (nur RBAC)

```text
CIS kontrolliert für DEV, TEST und PROD jeweils in Azure Access control (IAM) bzw. über die Azure-RBAC-Role-Assignments:
1. Die korrekte Principal/Object ID ist der richtigen APIM-Managed Identity zugeordnet.
2. Die Rolle Key Vault Secrets User ist am richtigen Key-Vault-Resource-Scope vorhanden (neu vergeben oder bereits wirksam).
3. Es wurde keine Role Assignment auf Resource-Group- oder Subscription-Ebene erzeugt.
4. Es wurden keine bestehenden Berechtigungen unbeabsichtigt verändert und keine zusätzlichen Access Policies angelegt.
5. Die Role-Assignment-ID und das Ergebnis der RBAC-Prüfung sind pro Umgebung im CIS-Vorgang dokumentiert.

Die Abnahme dieses CIS-Auftrags beschränkt sich auf die korrekte RBAC-Zuweisung. Es sind keine APIM-, Netzwerk-, Zertifikatsabruf- oder API-Funktionstests Bestandteil des CIS-Auftrags.
```

### Risiko / Auswirkungen

```text
Die Änderung vergibt lesende Secret-Berechtigungen an drei definierte APIM-Managed Identities. Durch die reine RBAC-Zuweisung ist keine Dienstunterbrechung geplant.

Risiken sind eine veraltete Principal-ID, ein falscher Ziel-Key-Vault und ein zu großer Berechtigungs-Scope. Auf Vault-Ebene erlaubt Key Vault Secrets User das Lesen aller Secrets des jeweiligen Vaults. CIS prüft deshalb vor Ausführung Identität, Zielressource, Scope, Bestandsberechtigungen und erforderliche Freigaben. Die formale Risikoeinstufung erfolgt nach CIS-/Change-Vorgabe.
```

### Backout / Rollback (nur CIS)

```text
Bei notwendigem Rollback entfernt CIS ausschließlich die im Rahmen dieses Auftrags neu angelegten RBAC-Role-Assignments anhand der dokumentierten Role-Assignment-IDs und nach erforderlicher Freigabe. Bereits bestehende Rollen, Access Policies, Key Vaults, Secrets und Zertifikate bleiben unverändert.

Der Rollback erfolgt je Umgebung unabhängig. War keine neue Rollenzuweisung erforderlich, besteht für diese Umgebung kein RBAC-Rollback.
```

### Backout-Prüfung (nur CIS)

```text
CIS verifiziert im Azure-IAM, dass ausschließlich die im Rahmen dieses Auftrags neu erstellten Role-Assignments entfernt wurden und die zuvor bestehenden Berechtigungen unverändert sind. Ergebnis mit Umgebung, Scope und Assignment-ID dokumentieren.
```

### Betroffene Systeme / CIs

```text
APIM DEV: uiapimf5c41c8714ee
Key Vault DEV: kv-21d79b9b0ff404b6dcba
APIM TEST: uiapim125ce218d1e3
Key Vault TEST: kv-250fc9d46bacfd08c783
APIM PROD: uiapim2b80e30deb7e
Key Vault PROD: kv-17cdd303c9bb2e255af9
```

### Referenzen

```text
DBI-12045: Key-Vault-Zugriffsfreigaben für APIM
DBI-8123: Übergeordnete Impulsmanager-Anbindung
DBI-9172: Internes Referenzverfahren für CIS-Cross-Subscription-RBAC
```

## 2. Werte für einzelne CIS-RBAC-Bestellpositionen

Die folgenden **drei Paare** sind unabhängig voneinander zu erfassen, auch wenn sie in einem Change dokumentiert werden.

| Bestellposition | APIM-Principal/Object ID | Vault | Ziel-Subscription |
|---|---|---|---|
| DEV | `d2f189cb-814b-4d25-ae88-1b6552f464a0` | `kv-21d79b9b0ff404b6dcba` | `6289fbfa-0b84-4f9e-8250-684455e9bb1a` |
| TEST | `44fd1bdb-45fe-4fc8-9351-0b05bd28f794` | `kv-250fc9d46bacfd08c783` | `d98221c3-b24d-4770-a9a3-3716667d7353` |
| PROD | `53c9b7c3-50e1-4a8d-b45b-aa10ed1882b9` | `kv-17cdd303c9bb2e255af9` | `a6c520e0-a1af-4c4c-82f0-8482ff82ecd8` |

### Azure-RBAC-Scope DEV: im bereitgestellten Azure-Export bestätigt

```text
/subscriptions/6289fbfa-0b84-4f9e-8250-684455e9bb1a/resourceGroups/rg-im-action-hub-dev-secrets-internal/providers/Microsoft.KeyVault/vaults/kv-21d79b9b0ff404b6dcba
```

### Azure-RBAC-Scope TEST: Resource Group bisher nur aus Namensschema abgeleitet

```text
/subscriptions/d98221c3-b24d-4770-a9a3-3716667d7353/resourceGroups/rg-im-action-hub-test-secrets-internal/providers/Microsoft.KeyVault/vaults/kv-250fc9d46bacfd08c783
```

**Vor Zuweisung die tatsächliche Resource ID in Azure bestätigen.**

### Azure-RBAC-Scope PROD: Resource Group bisher nur aus Namensschema abgeleitet

```text
/subscriptions/a6c520e0-a1af-4c4c-82f0-8482ff82ecd8/resourceGroups/rg-im-action-hub-prod-secrets-internal/providers/Microsoft.KeyVault/vaults/kv-17cdd303c9bb2e255af9
```

**Vor Zuweisung die tatsächliche Resource ID in Azure bestätigen.**

## 3. Formale Felder, die von der CIS-Maske abhängen

Diese Werte sind **nicht belastbar aus den technischen Daten ableitbar** und müssen anhand eurer CIS-Vorlage bzw. ServiceNow-Auswahl gesetzt werden:

- Change-Typ, Kategorie / Subkategorie
- Zuständige CIS-Zuweisungs- und Koordinatorengruppe
- Technical Service Offering / Application Service für einen umgebungsübergreifenden Change
- Risikoklasse und Genehmigungen
- Ausführungszeitfenster und Reihenfolge, soweit durch CIS-Freigaben vorgegeben

**Nicht** die IBA-Firewall-Zuweisungsgruppe oder die Auswahlfelder des bestehenden Firewall-Changes ungeprüft übernehmen.

## 4. Herkunft und Verifikationsstand der Daten

- Key-Vault-Namen: Jira DBI-12045 und bereitgestellte Repo-Dateien.
- APIM-Namen und Principal IDs: `ui-mb-infrastructure` / `Global-Settings`; DEV zusätzlich durch Azure-Export bestätigt. **TEST und PROD vor CIS-Umsetzung live abgleichen.**
- DEV-Vault-Ressourcen-ID: Azure-Export. TEST-/PROD-Resource-Groups: aktuell nur aus Repo-Namensschema abgeleitet, **nicht live bestätigt**.
- CIS-Weg: dokumentierter Referenzfall aus `environment/_apps/verbundintegration/keyvault-apim.hcl`.
