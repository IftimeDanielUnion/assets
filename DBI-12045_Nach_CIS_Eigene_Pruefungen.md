# DBI-12045: Meine Prüfungen nach der CIS-Umsetzung

**Stand:** 08.10.2026  
**Zweck:** Deine persönliche Checkliste für die Arbeit **nach** der RBAC-Umsetzung durch CIS. **Diese Punkte gehören nicht in den CIS-Change-Testplan.** Es handelt sich um APIM-, Netzwerk- und Funktionstests mit dem APIM-Team bzw. Christoph.

## 1. Vor dem Test: Voraussetzungen je Umgebung

- [ ] CIS hat für die betreffende Umgebung die RBAC-Berechtigung bestätigt oder bestehende ausreichende Rechte dokumentiert.
- [ ] Die aktuelle APIM-Identität stimmt mit der im CIS-Auftrag verwendeten Principal ID überein.
- [ ] Die erforderliche Firewall-Freigabe bzw. deren Abdeckung ist bestätigt. **DEV:** vorhandener Firewall-Change `UI-CHG-0097506` und Prüfung der Bestandsfreigabe `UI-CHG-0038893`.
- [ ] Das zuständige APIM-Team bestätigt, mit welcher Managed Identity die konkrete Key-Vault-Zertifikatsreferenz arbeitet. Für den Truuco-Referenzfall ist `System assigned` dokumentiert, für Impulsmanager muss die tatsächliche Einstellung geprüft werden.
- [ ] Das Zertifikat bzw. zugehörige Secret ist in dem richtigen Key Vault vorhanden. Keine Secret-Werte oder privaten Schlüssel kopieren oder in Jira einstellen.

## 2. DEV: zuerst durchführen

**APIM:** `uiapimf5c41c8714ee`  
**Key Vault:** `kv-21d79b9b0ff404b6dcba`  
**Zertifikat laut Christoph:** `impulsmanager-dev-client-cert`  
**APIM-System-Principal ID laut Azure-Export:** `d2f189cb-814b-4d25-ae88-1b6552f464a0`

- [ ] In APIM unter **Certificates** die Key-Vault-Referenz auf `impulsmanager-dev-client-cert` prüfen; falls noch nicht vorhanden, Einrichtung mit dem APIM-Team abstimmen.
- [ ] Unter der Zertifikatsreferenz überprüfen, ob **System assigned** oder eine andere Client Identity ausgewählt ist. Die tatsächliche Identität muss zur CIS-Berechtigung passen.
- [ ] Die private DNS-Auflösung und HTTPS-Erreichbarkeit vom **tatsächlichen APIM-Netz** aus bestätigen lassen (nicht nur von deinem Arbeitsplatz). Im bisherigen DEV-Export: APIM-Subnetz `10.250.239.32/27`, Key-Vault-Private-Endpoint-IP `172.28.58.137`, TCP 443.
- [ ] Über APIM einen **neuen Abruf / manuellen Refresh** der Zertifikatsreferenz auslösen und den aktuellen Abrufstatus samt Zeitpunkt prüfen. Die bloße Anzeige eines zuvor gespeicherten Zertifikats reicht nicht.
- [ ] Falls die Impulsmanager-API bereits angebunden ist: vereinbarten End-to-End-Test der Client-Zertifikatsauthentifizierung zusammen mit Christoph / APIM-Team durchführen.
- [ ] Testergebnis und relevante Nachweise im Jira-Ticket `DBI-12045` dokumentieren. Dabei keine Secret-Werte/PFX/privaten Schlüssel anhängen.

## 3. TEST: erst nach erfolgreicher DEV-Prüfung

**APIM:** `uiapim125ce218d1e3`  
**Key Vault:** `kv-250fc9d46bacfd08c783`  
**Principal ID laut Repository:** `44fd1bdb-45fe-4fc8-9351-0b05bd28f794` (Live-Wert noch zu bestätigen)

- [ ] Aktuelle Principal ID sowie Vault-Resource-ID im Azure Portal bestätigen.
- [ ] Korrekte Key-Vault-Zertifikatsreferenz und Secret-/Zertifikatsname mit APIM-Team klären (bisher nicht bestätigt).
- [ ] Identität an der Zertifikatsreferenz mit der genehmigten CIS-Zuweisung abgleichen.
- [ ] Netzweg, Private DNS und ggf. bestehende Firewall-Freigabe prüfen bzw. bestätigen lassen.
- [ ] Einen frischen Zertifikatsabruf/Refresh und ggf. den Impulsmanager-Funktionstest ausführen.
- [ ] Ergebnis in `DBI-12045` dokumentieren.

## 4. PROD: erst nach erfolgreicher TEST-Prüfung und PROD-Freigabe

**APIM:** `uiapim2b80e30deb7e`  
**Key Vault:** `kv-17cdd303c9bb2e255af9`  
**Principal ID laut Repository:** `53c9b7c3-50e1-4a8d-b45b-aa10ed1882b9` (Live-Wert noch zu bestätigen)

- [ ] Aktuelle Principal ID sowie Vault-Resource-ID im Azure Portal bestätigen.
- [ ] Korrekte Key-Vault-Zertifikatsreferenz und Secret-/Zertifikatsname mit APIM-Team klären (bisher nicht bestätigt).
- [ ] Identität an der Zertifikatsreferenz mit der genehmigten CIS-Zuweisung abgleichen.
- [ ] Netzweg, Private DNS und Firewall-Freigabe für PROD bestätigen lassen.
- [ ] Frischen Zertifikatsabruf/Refresh und vereinbarten Impulsmanager-Funktionstest im freigegebenen PROD-Fenster ausführen.
- [ ] Ergebnis in `DBI-12045` dokumentieren und Abschluss mit Christoph abstimmen.

## 5. Kurze Jira-Abschlussnotiz zum späteren Ausfüllen

Den folgenden Text **erst nach den realen Tests** verwenden und je Umgebung nur tatsächliche Ergebnisse ergänzen:

```text
DBI-12045: Die CIS-RBAC-Zuweisung für die betroffene Umgebung wurde bestätigt. Die APIM-Zertifikatsreferenz und die verwendete Managed Identity wurden geprüft. Netzwerk- und DNS-Erreichbarkeit sowie ein aktueller Abruf des Client-Zertifikats wurden separat mit dem APIM-Team getestet. Die konkreten Testergebnisse und Nachweise sind in den jeweiligen Umsetzungsvorgängen dokumentiert.
```

**Wichtig:** Der CIS-Change bestätigt die RBAC-Konfiguration, nicht die erfolgreiche technische Anbindung an Atruvia. Das wird durch die oben genannten separaten Tests festgestellt.
