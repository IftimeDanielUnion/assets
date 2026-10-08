# DBI-12045 | NEUER CORE-FIREWALL-Change TEST (ServiceNow Copy & Paste)

> **Soll-Freigabe, eindeutig und minimal:** `10.250.223.128/27 → 172.28.59.9/32 | TCP | Zielport 443 | Allow | fw-hub-test-01`.
> **Umfang:** nur APIM TEST → der eine Key-Vault-Private-Endpoint TEST. Keine pauschale Workload-Freigabe.

**Technischer Stand:** Ziel-IP aus dem vom Antragsteller übermittelten Azure-Private-Endpoint-Export (`customDnsConfigs`), Private-Link-Verbindung `Approved`, Provisioning `Succeeded`. Quell-CIDR und Quell-EPG aus dem gelieferten Repository `Firewall-Rules` (Live-Abgleich durch IBA).

**EPG-Hinweis:** Der unten genannte *Ziel-EPG-Name ist ein eindeutiger Neuanlage-Vorschlag nach DEV-Muster*, **kein als bereits vorhanden bestätigtes Objekt**. IBA prüft die ServiceNow-EPGs, legt falls notwendig das Einzelziel an und verwendet im Firewall-Repository diese EPG oder eine gleichwertige exakt auf **eine IP** begrenzte EPG.

## A. Freischaltung (für Felder Quelle / Ziel / Ports)

| Feld | Exakter Wert |
|---|---|
| **Quelle, APIM** | `uiapim125ce218d1e3` |
| **Quelladressbereich** | `10.250.223.128/27` |
| **Quelle EPG (bestehend laut Repo)** | `IPS-AZR-TSTABN-S2-APIMANAGEMENT-GATEWAY` |
| **Ziel, Key Vault** | `kv-250fc9d46bacfd08c783` |
| **Ziel-FQDN** | `kv-250fc9d46bacfd08c783.vault.azure.net` |
| **Zieladresse** | `172.28.59.9/32` (ein einzelner Host) |
| **Ziel EPG (neu, vorgeschlagener Name)** | `IPS-AZR-TSTABN-S2-MARKTBEARBEITUNG-KEYVAULT`; `value: 172.28.59.9` |
| **Richtung** | `APIM TEST -> Key Vault TEST` |
| **Aktion / Regeltyp** | `Allow / Network Rule` |
| **Protokoll** | `TCP` |
| **Zielport** | `443` |
| **Azure-Firewall-Instanz** | `fw-hub-test-01` |
| **Firewall-Stage** | `nonprd` |

## B. Change-Anforderung, Feld für Feld

### Titel

```text
[AZURE][CORE-FIREWALL][Anlage][MARKTBEARBEITUNG][TEST][APIM-Zugriff auf IM-Action-Hub-Key-Vault]
```

### Angefordert von

```text
Iftime, Daniel (UIS)
```

### Typ / Modell / Kategorie / Subkategorie

```text
Typ: Normal
Modell: Normal
Change Kategorie: Netz
Subkategorie: Firewall
```

### Technical Service Offering

```text
TSO tAB MARKTBEARBEITUNG Test
```

*Aus `Global-Settings` übernommen; Auswahl mit den in ServiceNow tatsächlich angebotenen CIs abgleichen.*

### Application Service

```text
AS Marktbearbeitung Test
```

### Zuweisungsgruppe Haupt-Change

```text
ChK-FMD-Marktbearbeitung
```

### IBA-Implementierungsaufgabe (vom Workflow anzulegen / zuzuweisen)

```text
Typ: Implementierung
Zuweisungsgruppe: ChK-IIM-IBA
Betroffenes CI: TSO Azure Firewall Rules Produktion
Application Service: AS Azure Core Platform Produktion
```

*Wie bei der DEV-Implementierung. Die vom ServiceNow-Workflow erzeugten Aufgaben und ihre tatsächliche CI-Zuordnung kontrollieren. DEV-CTASK0110524 nicht wiederverwenden.*

### Quelle(n) EPG

```text
IPS-AZR-TSTABN-S2-APIMANAGEMENT-GATEWAY
```

### Ziel(e) EPG

```text
IPS-AZR-TSTABN-S2-MARKTBEARBEITUNG-KEYVAULT
```

*Die Ziel-EPG muss bei Neuanlage **ausschließlich `172.28.59.9`** enthalten. Wenn der vorgeschlagene Name bei IBA/ServiceNow abweicht, den tatsächlich angelegten exakten EPG-Namen im Change nachführen. Keine vorhandene breite `#WL_MARKTBEARBEITUNG_TEST` als Ersatz ohne auf /32 begrenzten Scope.*

### Ziel-Port(s)

```text
443
```

### Protokoll(e)

```text
TCP
```

### Begründung

```text
Im Rahmen von DBI-12045 / DBI-8123 benötigt die APIM-Instanz uiapim125ce218d1e3 (TEST) Zugriff auf den privaten Endpunkt des IM-Action-Hub-Key-Vaults kv-250fc9d46bacfd08c783, um das Client-Zertifikat für die Anbindung an die Impulsmanager-API bei Atruvia abrufen zu können.

Es wird ausschließlich folgende gerichtete Firewall-Freigabe beantragt:
QUELLE: APIM TEST, EPG IPS-AZR-TSTABN-S2-APIMANAGEMENT-GATEWAY, IP-Bereich 10.250.223.128/27
ZIEL: Key Vault kv-250fc9d46bacfd08c783, FQDN kv-250fc9d46bacfd08c783.vault.azure.net, Private-Endpoint-IP 172.28.59.9/32
AKTION: Allow
REGELTYP: Azure Firewall Network Rule
PROTOKOLL: TCP
ZIELPORT: 443
FIREWALL: fw-hub-test-01 (nonprd)

Der Ziel-EPG soll nur die einzelne private IP 172.28.59.9 abdecken; vorgeschlagener Name: IPS-AZR-TSTABN-S2-MARKTBEARBEITUNG-KEYVAULT. Keine Freigabe auf gesamte Key-Vault-/Marktbearbeitung-Netze und keine weiteren Zielports, Protokolle, Quellen oder Umgebungen.

IBA soll vor der Neuanlage die effektive bestehende Regel-/Tag-Abdeckung auf exakt diese Verbindung prüfen. Bei bereits vollständig vorhandener Freigabe bitte den Nachweis dokumentieren und keine Doppelregel erzeugen. Andernfalls genau diese Einzelziel-Verbindung durch eine dedizierte Netzwerkregel und gegebenenfalls neue /32-Ziel-EPG ermöglichen.

Die RBAC-Leseberechtigung für Key Vault ist Bestandteil eines getrennten CIS-Vorgangs und gehört nicht zu diesem CORE-FIREWALL-Change.
```

### Zusätzliche Informationen

```text
Jira: DBI-12045; Parent: DBI-8123
Vorbild DEV: UI-CHG-0097506, IBA-Implementierung CTASK0110524, PR 95435, Merge-Commit 22d462b4e181df9e5d4f447afe2530dfff75cff6.

Quell-Subscription: S_APIMANAGEMENT_TEST
Quell-Subscription-ID: 14557421-c8d7-430d-8fc3-62e217d47033
Quellressource: APIM uiapim125ce218d1e3
Quell-EPG: IPS-AZR-TSTABN-S2-APIMANAGEMENT-GATEWAY
Quellnetz: 10.250.223.128/27

Ziel-Subscription: S_MARKTBEARBEITUNG_TEST
Ziel-Subscription-ID: d98221c3-b24d-4770-a9a3-3716667d7353
Zielressource: Key Vault kv-250fc9d46bacfd08c783
Ziel-FQDN: kv-250fc9d46bacfd08c783.vault.azure.net
Ziel-IP: 172.28.59.9/32
Private Endpoint: pe-im-action-hub-test-secrets-internal-kv
Private-Endpoint-Ressourcen-ID: /subscriptions/d98221c3-b24d-4770-a9a3-3716667d7353/resourceGroups/rg-im-action-hub-test-secrets-internal/providers/Microsoft.Network/privateEndpoints/pe-im-action-hub-test-secrets-internal-kv
VNet/Subnetz Private Endpoint: vnet-marktbearbeitung-test-swc-01/privateendpoints
Ziel-Subnetz-Ressourcen-ID: /subscriptions/d98221c3-b24d-4770-a9a3-3716667d7353/resourceGroups/rg-marktbearbeitung-test-networking/providers/Microsoft.Network/virtualNetworks/vnet-marktbearbeitung-test-swc-01/subnets/privateendpoints
Private Endpoint: Approved / Succeeded (aus bereitgestelltem Export)

Gewünschte Freigabe: 10.250.223.128/27 -> 172.28.59.9/32, TCP/443, Allow, Firewall fw-hub-test-01; ausschließlich TEST.
Quell-EPG und IP-Bereich stammen aus dem Repo und sind vor Ausrollung im Azure-/Firewall-Live-Zustand zu verifizieren.
Ziel-EPG-Vorschlag: IPS-AZR-TSTABN-S2-MARKTBEARBEITUNG-KEYVAULT; EPG-Wert exakt 172.28.59.9.

Bestehenden Firewall-Bestand prüfen:
Im Repository ist bereits die allgemeine Netzwerkregel „API Managment Test to Marktverarbeitung Test“ mit #WL_APIMANAGEMENT_TEST → #WL_MARKTBEARBEITUNG_TEST, TCP 443, dokumentiert (UI-CHG-0041935). Das belegt allein weder deren aktuelle Ausrollung noch die Abdeckung der Ziel-IP 172.28.59.9. Zuerst effektive Abdeckung prüfen.

Ansprechpartner Change: Iftime, Daniel (UIS)
Fachlicher Ansprechpartner: Kurrat, Christoph (EXT)
Umsetzung ausschließlich durch ChK-IIM-IBA im Repository Firewall-Rules.
```

## C. Details / Planungsfelder

### Umgebung

```text
Fachlicher Scope: Test (APIM TEST -> Key Vault TEST); keine Freigabe für andere Umgebungen.
```

*ServiceNow-Umgebungsfeld anhand des tatsächlich verwendeten Azure-Core-Firewall-CI setzen. Bei einer zentralen Plattform kann als technischer CI-Kontext „Produktion“ erscheinen, auch wenn der fachliche Scope TEST ist. Entscheidend ist die obige Quell-/Ziel-Eingrenzung.*

### Implementierungsplan (nur IBA-Aufgaben)

```text
1. Im aktuell ausgerollten Azure-Firewall-Regelwerk fw-hub-test-01 prüfen, ob 10.250.223.128/27 -> 172.28.59.9/32, TCP Zielport 443, bereits wirksam freigegeben ist. Quell-EPG IPS-AZR-TSTABN-S2-APIMANAGEMENT-GATEWAY und die EPG-/Tag-Auflösung auf ihre tatsächlichen IPs verifizieren; private Ziel-IP mit Azure-Private-Endpoint- und DNS-Konfiguration abgleichen.

2. Falls die Verbindung bereits vollständig erlaubt ist: effektive Regel / EPG-Auflösung / Deploymentstand nachvollziehbar im Change dokumentieren. Keine zweite Regel anlegen.

3. Falls die Verbindung nicht vollständig erlaubt ist: nur die unter A/B definierte gerichtete Network-Rule-Allow-Freigabe implementieren. Als Quelle IPS-AZR-TSTABN-S2-APIMANAGEMENT-GATEWAY = 10.250.223.128/27, als Ziel eine Einzelhost-EPG mit ausschließlich 172.28.59.9 (vorgeschlagener Name IPS-AZR-TSTABN-S2-MARKTBEARBEITUNG-KEYVAULT), Protokoll TCP und Zielport 443. Quell-/Zielnetz oder Ports nicht ausweiten.

4. Änderungen ausschließlich im Repository Firewall-Rules vornehmen. Vorgesehene Netzwerkregel-Collection: Workload-APIMANAGEMENT-Marktbearbeitung in rules/workloads/apimanagement/apimanagement-marktbearbeitung.yml; neue oder angepasste Ziel-EPG in tags/epgs/epg-azr.yml. Den neuen Change-Identifier in die Regelbeschreibung übernehmen. Branch/PR nach internem Schema erstellen.

5. Pre-Commit, YAML-/Repository-Validierung, Pull-Request-Review sowie Prüfung des Diffs und des Firewall-Deploymentplans auf unbeabsichtigte Änderungen durchführen. Genehmigungen des jeweiligen Changes einholen.

6. Die genehmigte Regel-/EPG-Änderung über die vorgesehene Firewall-Pipeline für fw-hub-test-01 ausrollen. Pull Request, Commit und CI/CD-/Deployment-Run samt Ergebnis im Change dokumentieren.

7. Nachweis der technisch ausgerollten Freigabe 10.250.223.128/27 -> 172.28.59.9/32, TCP/443 und der finalen EPG-Werte erbringen. Die Funktionsprüfung des APIM-Zertifikatsabrufs und Key-Vault-RBAC werden separat durch Antragsteller/APIM-Team bzw. CIS behandelt.
```

### Risiko- und Auswirkungsanalyse

```text
Keine Betriebsunterbrechung geplant. Es handelt sich um eine eng begrenzte Ergänzung einer Azure-Firewall-Freigabe von APIM TEST (10.250.223.128/27) zu genau einer privaten Key-Vault-IP (172.28.59.9/32) über TCP 443.

Risiko: Falsche EPG-Auflösung, falsche Ziel-IP oder versehentlich zu breite Regel könnten ungewollte Freigaben erzeugen. Ein fehlerhaftes Policy-Deployment könnte bestehende Verbindungen beeinträchtigen. Auch bei TEST können falsch definierte Quell-EPGs oder eine zu breite Ziel-EPG zusätzliche Verbindungen ermöglichen; Verbindungen anderer DEV-/TEST-Workloads dürfen nicht verändert werden.

Risiko wird begrenzt durch /32-Ziel, eine vorab geprüfte Quell-EPG, Diff-/Deploymentplan-Review, genehmigte Umsetzung und überprüfbaren Rollback. Keine Änderung am Key Vault, an Private Endpoints, an Zertifikaten oder an Azure-RBAC.
```

### Backout-Plan / Rollback-Plan

```text
Wenn die bereits ausgerollte Firewall-Policy die exakt angeforderte Verbindung erlaubt und kein Deployment erforderlich ist: keine neue Konfiguration zurückzurollen.

Bei Problemen ausschließlich die durch diesen Change neu eingeführte Regel bzw. Einzelziel-EPG im Repository Firewall-Rules gezielt zurücknehmen. Den Revert über Pull Request, Freigabe und denselben vorgesehenen Azure-Firewall-Deploymentprozess ausrollen. Vorherige Regeln und EPGs nicht pauschal entfernen. Anschließend Rückkehr zum dokumentierten Ausgangszustand in der Firewall-Policy belegen.
```

### Qualitätssicherung vor dem Change

```text
Quell-EPG IPS-AZR-TSTABN-S2-APIMANAGEMENT-GATEWAY und IP-Bereich 10.250.223.128/27 anhand aktueller APIM-/EPG-Daten abgleichen. Ziel-FQDN kv-250fc9d46bacfd08c783.vault.azure.net, Private Endpoint pe-im-action-hub-test-secrets-internal-kv und private Ziel-IP 172.28.59.9/32 verifizieren. Vorhandene effektive Regeln und aufgelöste Tags auf der Firewall fw-hub-test-01 prüfen. Freigabeumfang auf TCP-Zielport 443, Richtung 10.250.223.128/27 -> 172.28.59.9/32, und Umgebung TEST beschränken. Neue Ziel-EPG nur mit exakt einem Zielhost vorsehen. PR-Diff, notwendige Freigaben und Deploymentfenster prüfen.
```

### Qualitätssicherung nach dem Change

```text
Kontrollieren und im Change nachweisen: Azure-Firewall-Policy auf fw-hub-test-01 wurde erfolgreich mit dem vorgesehenen Stand ausgerollt; Quell-EPG IPS-AZR-TSTABN-S2-APIMANAGEMENT-GATEWAY enthält den freigegebenen Quellbereich 10.250.223.128/27; finale Ziel-EPG enthält ausschließlich 172.28.59.9; effektiv erlaubt ist nur die beantragte Richtung 10.250.223.128/27 -> 172.28.59.9/32 über TCP Zielport 443. PR, Commit, Pipeline-/Deployment-Lauf und finalen EPG-Namen verlinken. Keine unbeabsichtigten zusätzlichen Regel-/Tag-Änderungen.
```

## D. Nur aus dem realen ServiceNow-Workflow übernehmen (keine erfundenen Angaben)

- **Change-Nummer / CTASK-Nummer:** automatisch bei Neuanlage vergeben; weder DEV- noch alte Entwurfsnummer wiederverwenden.
- **Risiko / Auswirkung:** nach Change-Risikomatrix durch zuständige Stelle einstufen; PROD nicht ohne Freigabe übernehmen.
- **Start-/Enddatum:** genehmigtes neues Changefenster; die DEV-Vorlage nannte ein Standardfenster (Start in drei Tagen, Ende sieben Tage später), das ist **kein automatisch genehmigter neuer Termin**.
- **Changekoordinator, Genehmigung, Status, betroffene CIs, ServiceNow-Umgebungswert:** im aktuellen Formular / CMDB kontrollieren. Für die IBA-Aufgabe ist als Referenz aus DEV `ChK-IIM-IBA` bekannt.

## E. Technische Umsetzungsskizze für IBA (nicht im Antragsteller-Terraform-Repo)

Ziel-EPG in `tags/epgs/epg-azr.yml` (sofern aktuell keine äquivalente reine Einzelziel-EPG existiert):

```yaml
- name: IPS-AZR-TSTABN-S2-MARKTBEARBEITUNG-KEYVAULT
  desc: Private Endpoint des IM-Action-Hub-Key-Vaults TEST
  value:
    - 172.28.59.9
```

Netzwerkregel innerhalb der bestehenden Collection `Workload-APIMANAGEMENT-Marktbearbeitung` (Regelbeschreibung um die neue Change-Nummer ergänzen):

```yaml
- name: 'Access APIM-TEST to IM-Action-Hub-Key-Vault-TEST'
  desc: 'DBI-12045'
  src:
    - '#IPS-AZR-TSTABN-S2-APIMANAGEMENT-GATEWAY'
  dst:
    - '#IPS-AZR-TSTABN-S2-MARKTBEARBEITUNG-KEYVAULT'
  ports:
    - '443'
  protocols:
    - 'TCP'
```

**Wichtig:** Diese Skizze definiert die gewünschte Einzelhost-Freigabe. IBA prüft und integriert sie in das tatsächliche Firewall-Rules-Schema samt Collection und Change-Referenz. Das Modell ist auf die **Richtung APIM TEST → genau einen Key Vault TEST** begrenzt; keine Freigabe aller Marktbearbeitung-Adressen.

## F. Abgrenzung

Die Freigabe ist ausschließlich eine Netzänderung. Kein manuelles Anpassen des Key Vaults, keine neue Access Policy, keine Änderung des Berechtigungsmodells, kein Zertifikatsimport. Die CIS-RBAC-Bestellung und der anschließende tatsächliche Zertifikatsabruf aus APIM liegen außerhalb dieses Changes. Für TEST nur die Netzwerkregel in der Nonprod-Firewall fw-hub-test-01 erweitern.
