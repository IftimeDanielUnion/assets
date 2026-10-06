# ServiceNow Firewall Change - SFTP Freischaltung ftp81.msu.biz

## Angefordert von

```text
Iftime, Daniel (UIS)
```

## Change Kategorie

```text
Netz
```

## Subkategorie

```text
Firewall
```

## Technical Service Offering

```text
TSO Marktbearbeitung Produktion
```

## Application Service

```text
AS Marktbearbeitung Produktion
```

## Risiko

```text
2 - Mittel
```

## Auswirkung

```text
2 - Mittel
```

## Typ

```text
Normal
```

## Modell

```text
Normal
```

## Geplantes Startdatum

```text
08.10.2026 17:00:00
```

## Geplantes Enddatum

```text
15.10.2026 17:00:00
```

## Changekoordinatorgruppe

```text
ChK-FMD-DVOS
```

## Changekoordinator

```text
Iftime, Daniel (UIS)
```

## Zuweisungsgruppe

```text
ChK-FMD-DVOS
```

## Zugewiesen an

```text
Iftime, Daniel (UIS)
```

## Titel

```text
[AZURE][CORE-FIREWALL][Anlage][MARKTBEARBEITUNG][PROD][SFTP Zugriff auf ftp81.msu.biz]
```

## Begründung / Beschreibung

```text
!! BITTE BEACHTEN !!

Changes werden standardmäßig im IIM/PWS/FST Internen CAB: Do. 15:30 freigegeben.
Startdatum = Kommender Do. ab 17 Uhr
Enddatum = Startdatum + 7 Tage

Dringliche Azure Firewall Changes müssen mit IIM-IBA abgesprochen werden!

!! BITTE BEACHTEN !!

Begründung:

Für die Anwendung Marktbearbeitung muss aus der produktiven Azure-Umgebung eine SFTP-Verbindung zum externen Lukas-SFTP-System aufgebaut werden.

Der bisher verwendete SFTP-Endpunkt "lukaspartner.msu.biz" wurde durch einen neuen Endpunkt "ftp81.msu.biz" ersetzt bzw. ergänzt.

Beim Verbindungsaufbau vom aufrufenden System "umplapp01" zu "ftp81.msu.biz" auf TCP Port 22 kommt es aktuell zu einem Connection Timeout:

ssh: connect to host ftp81.msu.biz port 22: Connection timed out

Die bestehende Firewall-Freischaltung wurde nach aktuellem Kenntnisstand für "lukaspartner.msu.biz" eingerichtet. Da nun "ftp81.msu.biz" verwendet wird, greift diese Freischaltung für den neuen Ziel-FQDN nicht.

Die bestehende Source-Konfiguration der Firewall-Regel für "lukaspartner.msu.biz" soll unverändert verwendet und um das neue Ziel "ftp81.msu.biz" erweitert werden.

Die bestehende Freischaltung für "lukaspartner.msu.biz" soll zunächst bestehen bleiben. Eine Entfernung erfolgt erst nach bestätigter Außerbetriebnahme des alten Endpunkts.

Referenz:
Jira DBI-11149 - SFTP Verbindung

Metadaten

Subscription:
S_MARKTBEARBEITUNG_PROD

Ansprechpartner:
Iftime, Daniel

## Hinweis: Bitte möglichst EPGs als Quelle + Ziel verwenden. Diese sind zu finden in ServiceNow unter "Endpoint Group"! ##

Quelle

Aufrufendes System:
umplapp01

Quelle(n) EPG:
Identisch zur bestehenden Firewall-Freischaltung für "lukaspartner.msu.biz".
Es soll keine neue oder breitere Source-Freigabe eingerichtet werden.

Ziel

Name:
Lukas SFTP

Ziel(e):
ftp81.msu.biz

Bestehendes Ziel:
lukaspartner.msu.biz

Ziel-Port(s):
22

Protokoll(e):
TCP

Verbindungsrichtung:
Ausgehend aus der produktiven Azure-Umgebung zum externen SFTP-System.

Zusätzliche Informationen:

Die neue Freischaltung soll ausschließlich für dieselbe Quelle gelten, die bereits für "lukaspartner.msu.biz" freigeschaltet ist.

Es handelt sich ausschließlich um SFTP/SSH über TCP Port 22.

Es ist keine eingehende Freischaltung aus dem externen Netz in die Azure-Umgebung erforderlich.

Der bestehende Endpunkt "lukaspartner.msu.biz" soll mit diesem Change noch nicht entfernt werden.

Umgebung:
Produktion
```

## Implementierungsplan

```text
1. Neuen Branch nach dem bestehenden Branch-Schema für den neuen UI-CHG im Repository "Firewall-Rules" erstellen.

2. Im Firewall-Regelwerk die bestehende Regel für das Ziel "lukaspartner.msu.biz" ermitteln.

3. Die bestehende Source-/EPG-Konfiguration dieser Regel unverändert übernehmen.

4. Den neuen Ziel-FQDN "ftp81.msu.biz" für TCP Port 22 zusätzlich freischalten.

5. Die bestehende Freischaltung für "lukaspartner.msu.biz" unverändert bestehen lassen.

6. Change-Nummer des neuen ServiceNow Changes in der Description der Firewall-Regel ergänzen.

7. Anpassungen im Repository mittels Pre-Commit und den vorgesehenen Validierungen prüfen.

8. Definierten Pull-Request-Flow durchlaufen.

9. Ergebnis des CI/CD-Runs des Repositories "Firewall-Rules" prüfen.

10. Nach erfolgreichem Deployment Netzwerkverbindung von "umplapp01" nach "ftp81.msu.biz" auf TCP Port 22 testen.

11. Anschließend SFTP-Verbindungsaufbau mit dem vorgesehenen technischen Benutzer testen.

12. Testergebnis in Jira DBI-11149 und im Change dokumentieren.
```

## Risiko- und Auswirkungsanalyse

```text
Geringes Risiko.

Es wird ausschließlich ein zusätzlicher externer SFTP-Ziel-FQDN auf TCP Port 22 für eine bereits bestehende und definierte Quelle freigeschaltet.

Die Source-Konfiguration sowie bestehende Firewall-Freischaltungen werden nicht erweitert oder entfernt.

Es wird kein eingehender Zugriff auf Systeme der produktiven Azure-Umgebung ermöglicht.

Bestehende Verbindungen und Anwendungen werden durch die additive Änderung nicht beeinflusst.
```

## Backout-Plan

```text
Die Änderung wird über Infrastructure as Code zurückgerollt.

Hierzu wird der neu hinzugefügte Ziel-FQDN "ftp81.msu.biz" aus der Firewall-Regel entfernt und der vorherige Stand über den definierten CI/CD-Prozess erneut deployed.

Die bestehende Freischaltung für "lukaspartner.msu.biz" bleibt dabei unverändert bestehen.
```

## Rollback-Plan

```text
Über Infrastructure as Code zurückzurollen.

Der neu hinzugefügte Ziel-FQDN "ftp81.msu.biz" wird aus der Firewall-Regel entfernt und der vorherige Stand erneut deployed.
```

## Backout-Test

```text
Wird im Rahmen des Infrastructure-as-Code Deployments und der CI/CD-Pipeline automatisiert validiert.

Nach einem Rollback wird zusätzlich geprüft, dass der Firewall-Regelstand dem Stand vor der Änderung entspricht.
```

## Backout- und Rollback-Plan identisch

```text
wahr
```

## Vorschlag für Standard Change

```text
falsch
```

## Qualitätssicherung vor dem Change

```text
Überprüfung der technischen und funktionalen Anforderung.

Abgleich des neuen Ziel-FQDN "ftp81.msu.biz" mit Jira DBI-11149.

Überprüfung, dass für die Source exakt dieselbe EPG-/Netzwerkdefinition wie bei der bestehenden Freischaltung für "lukaspartner.msu.biz" verwendet wird.

Überprüfung, dass ausschließlich TCP Port 22 freigeschaltet wird.
```

## Qualitätssicherung nach dem Change

```text
Ergebniskontrolle des CI/CD-Merge-Laufs und Validierung der Firewall-Anpassungen.

Prüfung der Netzwerkverbindung von "umplapp01" zu "ftp81.msu.biz" auf TCP Port 22.

Anschließend Prüfung des SFTP-Verbindungsaufbaus.
```

## Abnahmetest geplant/durchgeführt

```text
Ja
```

## Testdokumentation hinterlegt

```text
Nach Durchführung des Changes: wahr
```

## Auto-Deployment

```text
falsch
```

## Service-Impact

```text
falsch
```

## Change übergreifend

```text
falsch
```

## CAB erforderlich

```text
falsch
```

## Quell AS

```text
AS Marktbearbeitung Produktion
```

## Ziel AS

```text
Externes Lukas SFTP-System / ftp81.msu.biz
```

## Beobachtungsliste

```text

```

## Arbeitsnotizenliste

```text

```

## Kommentare

```text
Neue SFTP-Zieladresse "ftp81.msu.biz" soll zusätzlich zur bestehenden Freischaltung für "lukaspartner.msu.biz" freigeschaltet werden. Bestehende Source-/EPG-Definition unverändert verwenden. Port 22/TCP.
```

## Arbeitsnotizen

```text
Nach Umsetzung CI/CD-Ergebnis prüfen und Verbindungstest von umplapp01 nach ftp81.msu.biz:22 dokumentieren.
```

## Abschlussnotizen

```text
Nach erfolgreicher Umsetzung:
Firewall-Regel für ftp81.msu.biz auf TCP Port 22 wurde für die bestehende Source der Lukas-SFTP-Verbindung ergänzt. CI/CD erfolgreich. Verbindungstest erfolgreich durchgeführt und in Jira DBI-11149 dokumentiert.
```

## Abschlusscode

```text
Erfolgreich
```

## Wurde der Zeitplan eingehalten?

```text
Ja
```

## Wurden Mängel in Bezug auf Planung und/oder Deployment identifiziert?

```text
Nein
```

## Waren die Angaben zu Auswirkung und Risiko-/Auswirkungsanalyse zutreffend?

```text
Ja
```

## Hat der Change alle gesetzten Ziele erreicht?

```text
Ja
```

## Wurden ungeplante Maßnahmen beim Deployment umgesetzt?

```text
Nein
```

## Wurden geplante Maßnahmen nicht umgesetzt?

```text
Nein
```

## Entspricht die Doku-Güte der Beschreibung den formalen Anforderungen?

```text
Ja
```

## Entspricht die Kategorisierung den formalen Anforderungen?

```text
Ja
```
