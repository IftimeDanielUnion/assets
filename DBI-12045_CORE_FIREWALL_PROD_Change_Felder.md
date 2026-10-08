# DBI-12045: CORE-FIREWALL-Change PROD | Felder für ServiceNow

## Zweck und Stand

Vorlage für einen **eigenständigen PROD-Change**, angelehnt an den erfolgreich abgeschlossenen DEV-Change **UI-CHG-0097506** (IBA-Aufgabe **CTASK0110524**, PR **95435**, Commit `22d462b4e181df9e5d4f447afe2530dfff75cff6`). Die DEV-Change-Nummer und DEV-CTASK **nicht** als Nummern des neuen PROD-Changes verwenden.

**Stand:** Das Repository bestätigt APIM-Instanz, Gateway-Quell-EPG/-CIDR, Key-Vault-Namen sowie die Ziel-Subscription und den vorgesehenen `privateendpoints`-Subnetzpfad. **Die tatsächliche private IP des PROD-Key-Vault-Private-Endpoints fehlt in den hochgeladenen Live-Exports.** Daher ist dies ein vollständiger **Change-Entwurf**, aber noch kein fertig verifizierter Implementierungsauftrag. IP/EPG vor Freigabe ergänzen.

**Zuständigkeit:** Anforderung und Koordination durch den Change-Antragsteller, technische Firewall-Bestandsprüfung und Umsetzung durch **ChK-IIM-IBA**. Kein separater Direktkontakt zu IBA erforderlich, sofern eine reguläre Implementierungsaufgabe zugewiesen wird.

**Abgrenzung:** Dieser Change umfasst nur die Netzwerkfreigabe. CIS-RBAC, Secret-Zugriff und APIM-Funktionstests stehen in separaten Vorgängen.

## Change-Anforderung: Felder zum Kopieren

### Angefordert von

**Hinweis:** Nur falls derselbe Antragsteller wie beim DEV-Change. ServiceNow kann den Wert automatisch setzen.

```text
Iftime, Daniel (UIS)
```

### Change Kategorie

```text
Netz
```

### Subkategorie / Release Kategorie

```text
Firewall
```

### Typ

```text
Normal
```

### Modell

```text
Normal
```

### Titel

**Hinweis:** „Anlage“ analog DEV. Falls sich nur eine Bestandsbestätigung oder Regelerweiterung ergibt, Aktionsbezeichnung vor Abschluss mit dem Change-Koordinator abstimmen.

```text
[AZURE][CORE-FIREWALL][Anlage][MARKTBEARBEITUNG][PROD][APIM-Zugriff auf IM-Action-Hub-Key-Vault]
```

### Technical Service Offering

**Hinweis:** Bezeichnung aus dem Workloads-Repo, nur wählen, wenn die aktuelle ServiceNow-CMDB diesen Eintrag tatsächlich führt.

```text
TSO tAB MARKTBEARBEITUNG Produktion
```

### Application Service

**Hinweis:** Bezeichnung aus dem Workloads-Repo; Anzeigename im ServiceNow-Katalog bestätigen.

```text
AS Marktbearbeitung Produktion
```

### Zuweisungsgruppe (Haupt-Change)

**Hinweis:** Wie beim DEV-Change vorgesehen. Für den neuen Change anhand des tatsächlich gewählten Services/Workflows kontrollieren; die IBA-Implementierungsaufgabe bleibt separat.

```text
ChK-FMD-Marktbearbeitung
```

### Zugewiesen an

**Hinweis:** Über den bestehenden Teamworkflow zuweisen lassen, keine Person aus der DEV-Implementierung als automatische Bearbeitung übernehmen.

### Risiko

**Hinweis:** Nicht blind den DEV-Wert 2 - Mittel übernehmen. Änderung für diesen Change, besonders PROD, mit dem Change-Koordinator nach Risikomatrix einstufen.

### Auswirkung

**Hinweis:** Nach aktueller ServiceNow-Risikomatrix bewerten und dokumentieren, keine bereits genehmigte Einstufung vortäuschen.

### Geplantes Startdatum

**Hinweis:** Change-Fenster mit Koordinator abstimmen. In der DEV-Vorlage gilt als Standard „in 3 Tagen“. Kein ausgedachtes Datum eintragen.

### Geplantes Enddatum

**Hinweis:** Change-Fenster abstimmen. In der DEV-Vorlage steht „Start + 7 Tage“, kein Nachweis einer siebentägigen Betriebsunterbrechung.

### Changekoordinatorgruppe / Changekoordinator

**Hinweis:** Den tatsächlich zuständigen Koordinator und seine Gruppe in ServiceNow auswählen lassen. Kein DEV-Automatismus.

### Genehmigung

**Hinweis:** Workflowfeld. Nicht als genehmigt markieren, solange Ziel-IP/EPG, Fenster und Risikoeinstufung fehlen.

### Status

**Hinweis:** Systemseitigen Status beim Anlegen verwenden; erst nach Prüfung dem regulären Freigabeprozess folgen.

### Korrelations-ID

**Hinweis:** Leer, falls keine externe Korrelations-ID vom Workflow verlangt wird. DBI-12045 als Referenz in die Begründung aufnehmen.

### Warten / Grund für Warten

**Hinweis:** Nur bei tatsächlichem Wartezustand setzen, beispielsweise wenn die Portalbestätigung der Private-Endpoint-IP noch fehlt.

### Begründung

```text
Begründung:
Im Rahmen von DBI-12045 / DBI-8123 benötigt APIM PROD (uiapim2b80e30deb7e) eine HTTPS-Verbindung über TCP 443 zum privaten Endpunkt des Key Vaults kv-17cdd303c9bb2e255af9, um die vorgesehene Client-Zertifikatsreferenz für die Anbindung an die Impulsmanager-API bei Atruvia nutzen zu können.

Vor einer Regelanlage bitte den aktuellen Bestand auf tatsächliche Abdeckung, ausgerollte Tag-/EPG-Auflösung und Routing prüfen. Der bestehende PRD-Netzwerkbereich enthält eine TCP-443-Regel zum Marktbearbeitung-Container-App-Ziel (UI-CHG-0090309), nicht den nachgewiesenen Private Endpoint dieses Vaults. Die vorhandene PRD-Application-Regel nennt mehrere FQDN-Ziele einschließlich eines anderen Key Vaults, aber nicht kv-17cdd303c9bb2e255af9. Vor Neuanlage dennoch aktuelle Policy, Netzwerk- und Applikationsregeln sowie deren tatsächlichen Deploymentstand prüfen.

Bei ausreichender bestehender Netzfreigabe bitte den Nachweis dokumentieren und keine Doppelregel anlegen. Andernfalls nur eine gezielte Freigabe vom genannten APIM-Gateway-Subnetz zur noch zu verifizierenden privaten IP des genannten Key-Vault-Private-Endpoints über TCP 443 umsetzen. Der DEV-Change UI-CHG-0097506, PR 95435 und Commit 22d462b4e181df9e5d4f447afe2530dfff75cff6 dienen als technisches Muster, nicht als Freigabe weiterer Umgebungen.

Metadaten:
Quell-Subscription: S_APIMANAGEMENT_PROD
Quell-Subscription-ID: 6ae0ab8b-523e-450a-86b4-b253b4cfb40a
Ziel-Subscription: S_MARKTBEARBEITUNG_PROD
Ziel-Subscription-ID: a6c520e0-a1af-4c4c-82f0-8482ff82ecd8
Ansprechpartner für Change: Iftime, Daniel (UIS)
Fachlicher Ansprechpartner für Zertifikat und APIM-Referenz: Kurrat, Christoph (EXT)

Quelle:
APIM-Instanz: uiapim2b80e30deb7e
Quell-EPG laut Firewall-Repo: IPS-AZR-PRD-S2-APIMANAGEMENT-GATEWAY
Quelladressbereich laut EPG-Repo: 10.250.222.160/27
Die Live-Quell-IP-/Subnetz-Zuordnung vor Umsetzung verifizieren.

Ziel:
Key Vault: kv-17cdd303c9bb2e255af9
FQDN: kv-17cdd303c9bb2e255af9.vault.azure.net
Ziel-VNet: vnet-marktbearbeitung-prod-swc-01
Ziel-Subnetz: privateendpoints
Ziel-Private-Endpoint-IP: noch im Azure Portal/über IBA für PROD zu bestätigen, vor Freigabe zwingend nachzutragen
Ziel-EPG: anhand dieser tatsächlichen privaten IP zu prüfen oder bedarfsgerecht neu anzulegen, nicht aus DEV übernehmen
Protokoll: TCP
Ziel-Port: 443
Richtung: APIM PROD -> Key Vault PROD

Abgrenzung:
Ausschließlich PROD. Kein Key-Vault-Public-Access, keine Änderung des RBAC-Modells, keine Access Policy, kein Zertifikatsimport und keine Regel für andere Umgebungen. Die Key-Vault-RBAC-Zuweisung läuft separat über CIS. Die nachgelagerte APIM-Funktionsabnahme ist nicht Gegenstand der IBA-Implementierung.
```

### Zusätzliche Informationen

```text
Referenzen: DBI-12045 / DBI-8123. Vorbild: DEV-Change UI-CHG-0097506, IBA-Implementierung CTASK0110524, PR 95435, Commit 22d462b4e181df9e5d4f447afe2530dfff75cff6.

Vorhandene Regel/Referenz für PROD: UI-CHG-0090309 (rules/workloads/apimanagement/apimanagement-marktbearbeitung.yml). Im gelieferten Firewall-Repo ist keine spezifische PROD-Freigabe zum Key Vault kv-17cdd303c9bb2e255af9 nachgewiesen. Das ist kein Beweis, dass keine andere aktuell ausgerollte Policy existiert. IBA prüft vor Neuanlage.

Ziel-VNet/Subnetz laut Workloads-Repo:
/subscriptions/a6c520e0-a1af-4c4c-82f0-8482ff82ecd8/resourceGroups/rg-marktbearbeitung-prod-networking/providers/Microsoft.Network/virtualNetworks/vnet-marktbearbeitung-prod-swc-01/subnets/privateendpoints

Noch nicht durch Live-Export bestätigt: private IP und Status des konkreten Key-Vault-Private-Endpoints, Ziel-EPG, Live-Netz-/Routing-Abdeckung, Quell-Subnetzzuordnung sowie genehmigtes Zeitfenster. Die Change-Freigabe zur Implementierung erst nach Ergänzung dieser Angaben einholen.

PROD ist der fachliche Umfang. Die Änderung erst nach abgeschlossenem DEV-Test und abgestimmter TEST-Abnahme in dem separat genehmigten PROD-Fenster umsetzen. Die bestehenden DEV-/TEST-Regeln nicht ändern.

Firewall-Repository: Firewall-Rules. Bei Bedarf auf der Basis des gemergten DEV-PR eine dedizierte Netzwerkregel und Ziel-EPG durch ChK-IIM-IBA umsetzen lassen. Das Workload-Terraform-Repo ui-mb-infrastructure gehört nicht zur Firewall-Implementierung.
```

## Details / Implementierung

### Umgebung

**Hinweis:** Fachlicher Scope ist nur diese Umgebung. Wenn das Firewall-CI in der CMDB als zentrale Produktionsplattform geführt wird, die nötige Formularkonvention mit dem Koordinator klären; das erweitert den fachlichen Scope nicht.

```text
Produktion
```

### Implementierungsplan

```text
1. IBA prüft im neuen PROD-Change den ausgerollten Firewall-Bestand, einschließlich EPGs, aufgelöster Tags und des vorhandenen Eintrags UI-CHG-0090309. Der Ziel-Key-Vault kv-17cdd303c9bb2e255af9 wird hinsichtlich tatsächlicher Private-Endpoint-IP, Erreichbarkeit und Routing abgeglichen.
2. IBA gleicht IPS-AZR-PRD-S2-APIMANAGEMENT-GATEWAY (10.250.222.160/27 laut Repository) mit dem aktuellen APIM-Gateway-Subnetz ab. Ziel-IP und gültige Ziel-EPG werden vor Implementierung im Change dokumentiert und freigegeben.
3. Besteht bereits eine nachgewiesene passende Freigabe, wird das Ergebnis mit Regel-/Deploymentnachweis dokumentiert. Es erfolgt keine Doppelregel.
4. Bei nachgewiesener Lücke wird ausschließlich die genehmigte PROD-Verbindung zum privaten Key-Vault-Endpunkt über TCP 443 als gezielte Netzwerkregel im Repository Firewall-Rules ergänzt. DEV UI-CHG-0097506 dient als Umsetzungsbeispiel. Keine Änderung am Workload-Terraform-Repo und keine Erweiterung auf andere Umgebungen.
5. IBA führt Repository-/Pre-Commit-Prüfungen, Review/Pull Request und die erforderlichen Freigaben durch. Den konkreten Deploymentplan auf Seiteneffekte prüfen.
6. Die freigegebene Firewall-Konfiguration wird im vereinbarten PROD-Zeitfenster über die zuständige Pipeline ausgerollt. Commit, PR, Pipeline-Lauf und Resultat im Change dokumentieren.
7. IBA kontrolliert die tatsächlich wirksame Netzfreigabe (Quell-EPG IPS-AZR-PRD-S2-APIMANAGEMENT-GATEWAY, konkrete bestätigte Ziel-IP/EPG, TCP 443) und dokumentiert den Netz-/Deploymentnachweis. APIM-Zertifikatsabruf und CIS-RBAC werden separat abgenommen.
```

### Risiko- und Auswirkungsanalyse

```text
Produktiver Zugriff; auch bei erwarteter fehlender Betriebsunterbrechung ist die Änderung an gemeinsam genutzter Firewall-Policy auf Seiteneffekte zu prüfen. Risiko und Auswirkung sind für PROD separat durch den Change-Koordinator zu bewerten.

Falsch aufgelöste EPGs, eine zu weit gefasste Zieldefinition oder eine Änderung an gemeinsam genutzten Firewall-Policies können ungewollte Netzwerkfreigaben und Seiteneffekte verursachen. Deshalb ausschließlich das geprüfte APIM-Gateway-Quellsubnetz und die tatsächliche private Key-Vault-Ziel-IP zulassen.

Die Quell-EPG ist nur als Repository-Wert belegt. Die Ziel-IP ist aktuell noch nicht belegt. Ohne diese Verifikation keine genehmigte Regel implementieren.

Die separate CIS-RBAC-Berechtigung gehört nicht zum Firewall-Change. Ein erfolgreicher Firewall-Deploymentlauf beweist noch keinen Zertifikatsabruf.
```

### Backout-Plan

```text
Wenn der Bestand bereits genügt und keine Anpassung ausgerollt wurde, ist kein technischer Rollback notwendig.

Bei einer fehlerhaften Umsetzung nur die durch diesen neuen PROD-Change eingeführte Firewall-Regel/-EPG-Änderung im Repository Firewall-Rules gezielt zurücknehmen. Den Revert über Review/Freigabe und die zuständige Firewall-Pipeline deployen. Danach den zuvor gültigen Regelzustand und die ausgerollte Konfiguration dokumentieren.

Nicht pauschal vorhandene PROD-Bestandsregeln, DEV-Change UI-CHG-0097506 oder Änderungen anderer Teams entfernen. Key Vault, Private Endpoint, Zertifikate und CIS-RBAC-Zuweisungen nicht über den Firewall-Rollback verändern.
```

### Rollback-Plan

**Hinweis:** Identisch zum Backout-Plan, sofern dies den ServiceNow-Vorgaben entspricht.

```text
Wenn der Bestand bereits genügt und keine Anpassung ausgerollt wurde, ist kein technischer Rollback notwendig.

Bei einer fehlerhaften Umsetzung nur die durch diesen neuen PROD-Change eingeführte Firewall-Regel/-EPG-Änderung im Repository Firewall-Rules gezielt zurücknehmen. Den Revert über Review/Freigabe und die zuständige Firewall-Pipeline deployen. Danach den zuvor gültigen Regelzustand und die ausgerollte Konfiguration dokumentieren.

Nicht pauschal vorhandene PROD-Bestandsregeln, DEV-Change UI-CHG-0097506 oder Änderungen anderer Teams entfernen. Key Vault, Private Endpoint, Zertifikate und CIS-RBAC-Zuweisungen nicht über den Firewall-Rollback verändern.
```

### Backout-Test

```text
Ein praktischer Backout-Test wurde für diesen neuen Change bislang nicht durchgeführt. Vor Umsetzung den gezielten Revert der ausschließlich in diesem Change eingeführten Regel-/EPG-Änderung und den Wiederherstellungsnachweis prüfen. Ein erfolgreicher Pipeline-Lauf allein ist kein ausgeführter Rollback-Test. Bei unverändertem Firewall-Bestand ist kein Rollback erforderlich.
```

### Backout- und Rollback-Plan identisch

**Hinweis:** Nur wenn in beiden Feldern derselbe Inhalt steht und die Auswahl so vorgesehen ist.

```text
wahr
```

### Vorschlag für Standard Change

**Hinweis:** Ein Normal Change wird nicht durch diese Vorlage zu einem Standard Change.

```text
falsch
```

## Zeitplan, Konfliktprüfung und Freigaben

### Service-Impact

**Hinweis:** Auswahl gemäß abgestimmter Risikobewertung und tatsächlicher Betriebswirkung treffen; keine ungetestete Aussage „keine Auswirkungen“.

### Service-Impact Von / Bis

**Hinweis:** Nur bei tatsächlich erwarteter Unterbrechung mit genehmigtem Zeitfenster befüllen.

### CAB-Empfehlung / CAB erforderlich / CAB-Datum

**Hinweis:** Nach dem regulären Workflow durch Change-Koordination bestimmen lassen, besonders für PROD.

### Nicht autorisiert

**Hinweis:** Systemfeld nicht eigenmächtig ändern.

### Change übergreifend

**Hinweis:** Wenn der ServiceNow-Prozess hier eine verknüpfte CIS-Bestellung berücksichtigt, Wert mit Change-Koordination klären.

### Konfliktstatus / Letzter Lauf des Konflikts

**Hinweis:** Konfliktprüfung nach Festlegung des Zeitfensters durch das vorgesehene System/Workflow ausführen lassen; nichts als geprüft markieren, bevor sie gelaufen ist.

### Testdokumentation hinterlegt

**Hinweis:** Erst nach Vorlage eines tatsächlichen IBA-Firewall-/Deploymentnachweises als hinterlegt kennzeichnen.

### Betriebsübernahmetest ausschließlich durch Provider

**Hinweis:** Vorgabe des ServiceNow-Workflows prüfen; IBA prüft die Netzwerkumsetzung, Anwendungsabnahme ist separat.

### Abnahmetest ausschließlich beim Provider

**Hinweis:** Vorgabe des ServiceNow-Workflows prüfen; die APIM-/Atruvia-Abnahme läuft separat.

### Qualitätssicherung vor dem Change

```text
Vor Freigabe die reale APIM-Quellsubnetzzuordnung (IPS-AZR-PRD-S2-APIMANAGEMENT-GATEWAY, 10.250.222.160/27 laut Repo) sowie die private IP und den Genehmigungs-/Provisioningstatus des Key-Vault-Private-Endpoints kv-17cdd303c9bb2e255af9 im Azure Portal verifizieren.

Konkrete Ziel-IP, Ziel-EPG, genehmigte Zeitfenster und betroffene CIs dokumentieren. Bestand UI-CHG-0090309 inklusive Deployment/Tag-Auflösung, Netzrouting und eventuell bereits vorhandenem passendem Regelnachweis prüfen. Bei neuem Firewall-Code Pre-Commit, PR, Review und Pipeline-Plan prüfen. Änderung auf PROD beschränken.
```

### Qualitätssicherung nach dem Change

```text
IBA bestätigt entweder die tatsächlich bereits ausgerollte Regelabdeckung oder die erfolgreich ausgerollte neue PROD-Firewall-Anpassung, jeweils mit Quelle 10.250.222.160/27 / IPS-AZR-PRD-S2-APIMANAGEMENT-GATEWAY, verifizierter Ziel-Private-IP/-EPG und TCP 443.

PR-/Commit-ID, Pipeline-Lauf, Regel-/Deploymentstatus, Zeitpunkt und Ergebnis dokumentieren. Die Anwendungstests (APIM-Zertifikatsreferenz, Key-Vault-RBAC, Zertifikatsabruf und Kommunikation mit Atruvia) erfolgen außerhalb dieser IBA-Aufgabe und werden separat erfasst.
```

### Auto-Deployment

**Hinweis:** ServiceNow-Systemwert unverändert lassen, sofern der Workflow nichts anderes fordert. Firewall-Pipeline-Ausführung separat dokumentieren.

### SAC-Liste

**Hinweis:** Nur eine tatsächlich vorhandene, relevante Liste zuordnen.

### Pipeline Check Sum

**Hinweis:** Nur echten Wert aus dem Deployment eintragen; nicht schätzen.

### Abnahmetest geplant/durchgeführt

**Hinweis:** Netzwerkergebnis kann erst nach IBA-Prüfung dokumentiert werden. Nicht vorzeitig als durchgeführt markieren.

## Formularbereich: Firewall

### Quell AS

**Hinweis:** APIM PROD in ServiceNow auswählen: uiapim2b80e30deb7e. Der konkrete CMDB-Anzeigename des APIM-Application-Service ist in den Unterlagen nicht nachgewiesen.

### Ziel AS

**Hinweis:** Als Zielservice wählen, falls im aktuellen ServiceNow-Katalog vorhanden.

```text
AS Marktbearbeitung Produktion
```

## Notizen und Kommunikation

### Kommentare / Arbeitsnotizen

```text
DBI-12045: CORE-FIREWALL PROD, Orientierung am erfolgreich abgeschlossenen DEV-Change UI-CHG-0097506 (PR 95435, Commit 22d462b4e181df9e5d4f447afe2530dfff75cff6).

Bitte den vorhandenen Stand zuerst auf die Verbindung APIM uiapim2b80e30deb7e (IPS-AZR-PRD-S2-APIMANAGEMENT-GATEWAY, 10.250.222.160/27) zum Key Vault kv-17cdd303c9bb2e255af9 (Private-Endpoint-IP noch nicht bestätigt), TCP 443, prüfen. Die private Ziel-IP und Ziel-EPG bitte vor Genehmigung ermitteln/dokumentieren. Nur eine nachweislich fehlende Freigabe gezielt ergänzen. Keine Doppelregel, keine DEV-Änderung. Die CIS-RBAC-Bestellung erfolgt separat.
```

### Beobachtungsliste / Arbeitsnotizenliste

**Hinweis:** Nur die bei euch tatsächlich erforderlichen Rollen/Personen hinzufügen. IBA wird über die Implementierungsaufgabe beauftragt.

## Umsetzungstask (nach automatischer Erstellung)

### Nummer

**Hinweis:** Die neue CTASK-Nummer wird durch ServiceNow erzeugt. DEV-CTASK0110524 nicht übernehmen.

### Typ

```text
Implementierung
```

### Zuweisungsgruppe

**Hinweis:** Analog zur erfolgreich abgeschlossenen DEV-Aufgabe.

```text
ChK-IIM-IBA
```

### Titel

```text
[AZURE][CORE-FIREWALL][Anlage][MARKTBEARBEITUNG][PROD][APIM-Zugriff auf IM-Action-Hub-Key-Vault]
```

### Begründung / Implementierungsplan

**Hinweis:** Begründung, technische Werte und Implementierungsplan aus dem Haupt-Change übernehmen. Vor Ausführung fehlende Private-Endpoint-IP und Ziel-EPG ergänzen.

### Betroffenes CI

**Hinweis:** Wird entsprechend der zentralen Firewall-Service-CMDB und der Vorgaben von IBA ausgewählt. Bei DEV war in der Task TSO Azure Firewall Rules Produktion eingetragen; dies nicht als PROD-Ziel dieser fachlichen Änderung interpretieren.

### Zugewiesen an

**Hinweis:** Durch IBA zuweisen lassen, keinen Bearbeiter aus DEV automatisch übernehmen.

### Status / Bestätigungsstatus

**Hinweis:** Durch regulären Task-Workflow und tatsächliches Ergebnis bestimmen lassen, nicht als erledigt vorbelegen.

## Abschluss und offene Angaben

### Abschlusscode / Abschlussnotizen

**Hinweis:** Erst nach abgeschlossener Implementierung bzw. dokumentierter Bestandsabdeckung ausfüllen. Hier die echte PR-/Commit-/Deploymentreferenz des neuen Changes eintragen, nicht den DEV-Commit als eigenen Abschluss.

### Tatsächlicher Start / Tatsächliches Ende

**Hinweis:** Durch tatsächlich erfolgte Task-/Change-Durchführung automatisch bzw. anhand realer Zeiten erfassen.

### Pflichtangaben vor Freigabe

```text
1. Aktuelle Private-Endpoint-IP für kv-17cdd303c9bb2e255af9.vault.azure.net über Azure Portal / IBA verifizieren.
2. Bestätigen, dass der Private Endpoint zu genau diesem Key Vault gehört und die Verbindung Approved / Succeeded ist.
3. Aktuelles APIM-Quellsubnetz und tatsächliche Quell-EPG prüfen.
4. Ziel-EPG oder gezieltes Einzelziel (Private-IP/32) dokumentieren; breit gefasste EPG nur mit begründeter Genehmigung verwenden.
5. Bestandsfreigaben und Routing auf Live-Deployment prüfen.
6. Reale ServiceNow-CMDB-Felder, Risiko-/Auswirkung, Change-Fenster, Genehmigungen und CTASK-Zuweisung bestätigen.
7. Separate CIS-RBAC-Bestellung referenzieren, nicht im Firewall-Change umsetzen.
```

## Quellen und Grenzen

- DEV-Vorbild: hochgeladene `change_task.pdf` vom 08.10.2026 mit erfolgreichem Abschluss zu `UI-CHG-0097506` und `CTASK0110524`.
- DEV-Regel: hochgeladener Screenshot des gemergten PR 95435, Commit `22d462b4e181df9e5d4f447afe2530dfff75cff6`.
- Source EPG/CIDR: hochgeladener Repo-Snapshot `Firewall-Rules/tags/epgs/epg-azr.yml`. Live-Werte vor Ausführung prüfen.
- Bestandsregeln: hochgeladener Repo-Snapshot `rules/workloads/apimanagement/apimanagement-marktbearbeitung.yml`.
- Ziel-Key-Vault: hochgeladene Repos `ui-mb-infrastructure` und `Global-Settings`.
- Ziel-Subscription-ID/privates Subnetz: `Global-Settings/Workloads/workloads.d/marktbearbeitung.yml`.
- APIM-Subscription-ID: `Global-Settings/service_principals/apimanagement/` (historische Repo-Referenz; Live-Zuordnung bestätigen).
- Dies ist **kein** nachträglich verifizierter Live-Export der PROD-Private-Endpoint-IP und kein Nachweis eines bereits freigegebenen neuen Changes.
