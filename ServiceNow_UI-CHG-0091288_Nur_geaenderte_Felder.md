# ServiceNow UI-CHG-0091288 | Nur geänderte Felder

*Gilt für UI-CHG-0091288 und CTASK0111253. Alle anderen Felder bleiben unverändert.*

## CHG Titel

```text
[AZURE][CORE-FIREWALL][Anlage][UNIONMARKETINGPLATTFORM][PROD][SFTP Zugriff auf ftp81.msu.biz]
```

## CTASK Titel

```text
[AZURE][CORE-FIREWALL][Anlage][UNIONMARKETINGPLATTFORM][PROD][SFTP Zugriff auf ftp81.msu.biz]
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

Für die UNIONMARKETINGPLATTFORM muss aus der produktiven Azure-Umgebung eine SFTP-Verbindung zum externen Lukas-SFTP-System aufgebaut werden.

Der bisher verwendete SFTP-Endpunkt "lukaspartner.msu.biz" wurde durch den neuen Endpunkt "ftp81.msu.biz" ersetzt bzw. ergänzt.

Beim Verbindungsaufbau vom aufrufenden System "umplapp01" zu "ftp81.msu.biz" auf TCP Port 22 kommt es aktuell zu einem Connection Timeout:

ssh: connect to host ftp81.msu.biz port 22: Connection timed out

Für den neuen SFTP-Endpunkt soll deshalb die bestehende UNIONMARKETINGPLATTFORM-PROD-Source verwendet und der ausgehende Zugriff auf "ftp81.msu.biz" über TCP Port 22 freigeschaltet werden.

Bestehende Firewall-Freischaltungen, insbesondere für den bisherigen Endpunkt, sollen durch diesen Change nicht entfernt werden.

Referenzen:
Jira DBI-11149 - SFTP Verbindung
Bestehende Anforderung: CTASK0105899 - [AZURE][CORE-FIREWALL][Anlage][UNIONMARKETINGPLATTFORM][TEST & PROD][Verbindung zu Lukas]
Aktueller Change: UI-CHG-0091288
Aktueller Task: CTASK0111253

Metadaten

Subscription:
S_UNIONMARKETINGPLATTFORM_PROD

Ansprechpartner:
Iftime, Daniel

## Hinweis: Bitte möglichst EPGs als Quelle + Ziel verwenden. Diese sind zu finden in ServiceNow unter "Endpoint Group"! ##

Quelle

Name:
UNIONMARKETINGPLATTFORM PROD VM

Aufrufendes System:
umplapp01

Quelle(n) EPG:
#IPS-AZR-PROD-S02-UNIONMARKETINGPLT-VM

Quellnetz:
172.28.63.32/27

Ziel

Name:
Lukas SFTP

Ziel(e):
ftp81.msu.biz

Ziel-Port(s):
22

Protokoll(e):
TCP

Verbindungsrichtung:
Ausgehend aus der produktiven UNIONMARKETINGPLATTFORM-Azure-Umgebung zum externen SFTP-System.

Zusätzliche Informationen:

Die Freischaltung soll ausschließlich für die bestehende PROD-VM-Source der UNIONMARKETINGPLATTFORM erfolgen:

#IPS-AZR-PROD-S02-UNIONMARKETINGPLT-VM
172.28.63.32/27

Freizuschalten ist ausschließlich:

Quelle:
#IPS-AZR-PROD-S02-UNIONMARKETINGPLT-VM / 172.28.63.32/27

Ziel:
ftp81.msu.biz

Port:
22

Protokoll:
TCP

Es handelt sich ausschließlich um ausgehende SFTP-/SSH-Kommunikation.

Es ist keine eingehende Freischaltung aus dem externen Netz in die Azure-Umgebung erforderlich.

Bestehende Firewall-Regeln und bestehende Ziel-FQDNs werden mit diesem Change nicht entfernt.

Umgebung:
Produktion
```

