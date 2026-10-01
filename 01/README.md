# UMP: Weiterarbeiten ohne Urlaubsblocker

**Stand: 01.10.2026.** Arbeitsgrundlage: Jira-Exporte vom 30.09.2026 und dein Repository-Upload. Der aktuelle Zustand in Azure, auf den Servern und in Jira wurde nicht abgefragt.

## Das ändert sich für dich

Du wartest nicht auf Timo Traub, Marcel Sebbin oder Christoph Sievers. Es gibt keine Rückgabe an sie, keine Startnachricht an sie und keine Freigabe durch sie als Voraussetzung dieses Plans. Du hältst die Bearbeitung zusammen und dokumentierst Ergebnisse selbst im Jira.

Wo Code und Ticket die Antwort bereits geben, arbeitest du damit weiter. Andere Personen brauchst du nur für etwas, das du nicht aus dem Code erledigen kannst: einen Zugang auf der Gegenseite, einen gemeinsamen Anwendungstest oder eine verbindliche Schnittstellenentscheidung. Die Verfügbarkeit der übrigen Kontakte ist nicht bestätigt.

## Deine Reihenfolge

| Reihenfolge | Ticket | Dein nächster eigener Schritt | Wen du wirklich brauchst |
|---|---|---|---|
| **1: sofort anstoßen** | [SFTP / DBI-11149](01_DBI-11149_SFTP.md) | Den vorhandenen öffentlichen Schlüssel direkt an Marc schicken; parallel die tatsächliche UMP-Konfiguration und die Verbindung prüfen. | Marc Hintz für Lukas, Alex Köster für den Anwendungstest. |
| **2: erstes Umsetzungsticket** | [Monitoring / DBI-6455](02_DBI-6455_Exception_Monitoring.md) | Vorhandene Logs prüfen und den vorbereiteten TEST-Patch übernehmen. | Alex nur für das Testereignis und die Empfangsbestätigung. |
| **3: kurzer Integrationscheck** | [Temporärer Login / PB2B-27849](03_PB2B-27849_SAML_Login.md) | Auf der richtigen Instanz die aktive Login-Konfiguration prüfen. | Oliver Ehli für die Zuordnung seiner temporären App. |
| **4: technische Verbindung vorbereiten** | [Event Grid / DBI-7161](04_DBI-7161_Event_Grid.md) | Den vorgeschlagenen Aufbau konkretisieren, vorhandene Ressourcen prüfen und Schnittstellen-Fakten bei Sinan/OSC einholen. | Sinan Balcin und die Anwendungskontakte für Sender und Empfänger. |

**Praktisch:** Nachricht an Marc abschicken, dann am Monitoring arbeiten. Die Login-Zuordnung und die drei offenen Event-Grid-Fakten kannst du gleichzeitig anfragen. Blockiert der Login tatsächlich die Migrationstests, ziehst du seinen Integrationscheck vor.

## Sind die Tickets standalone?

**Zwischen diesen vier Tickets ist keine zwingende technische Abhängigkeit belegt.** Ihre Priorität ist keine Bau-Reihenfolge. Du musst SFTP nicht abschließen, bevor du Monitoring beginnst. Der temporäre Browser-Login ist auch keine Voraussetzung der technischen Event-Grid-Anmeldung.

Innerhalb der Tickets gilt dagegen:

| Ticket | Wirkliche Abfolge |
|---|---|
| SFTP | Richtigen Schlüssel und Laufzeitkonfiguration prüfen → Gegenstelle/SSH-Anmeldung funktionieren → freigegebener Transfer → dokumentierte Abnahme. |
| Monitoring | Logs im richtigen Bereich sichtbar → Alarmregel bereitstellen → Anwendung erzeugt Testfehler → E-Mail nachgewiesen. |
| Login | Richtige Instanz identifizieren → temporäre App und Metadaten zuordnen → gezielte Änderung → echter Login-Test. |
| Event Grid | Schnittstellen und Zugriffsweg festlegen → Infrastruktur und Anwendungsteile können parallel entstehen → gemeinsame Zustell- und Fehlertests. |

## Was im Paket schon vorbereitet ist

Die vier Ticketdateien enthalten einfache Schritte und fertige freundliche Nachrichten. Im [Technik-Anhang](05_Technik_zum_Nachschlagen.md) stehen kopierbare Prüfungen. Die [Codebefunde und das Prüfprotokoll](06_Codebefunde_und_Pruefprotokoll.md) erklären, was tatsächlich geprüft wurde.

Zusätzlich liegen unter `code/` drei unabhängige Monitoring-Patches und drei Log-Abfragen. Die Patches ergänzen nur den vorhandenen Monitoring-Block im Repository `live`. Es ist dafür kein neuer `catalog`-Baustein nötig. TEST und DEV werden im jeweiligen Patch aktiviert; PROD wird ausdrücklich **deaktiviert vorbereitet**. DEV ist optional und kein neuer Pflichtschritt. [B1, B2]

Unter `anhaenge/` liegt ausschließlich der bereits im Jira veröffentlichte **öffentliche** Lukas-Schlüssel. Du brauchst dafür keinen alten Mailverlauf. Private Schlüssel, Passwörter und vollständige Repository-Exporte sind nicht Bestandteil dieses Pakets.

## So verschickst du die Nachrichten

Kopiere jeweils nur die Nachricht, deren Voraussetzung erfüllt ist. Die Startnachrichten kannst du sofort verwenden. Erfolgs- oder Abschlussnachrichten erst nach dem tatsächlichen Test. Interne Kontakte erreichst du über Teams oder einen Jira-Kommentar mit persönlicher Markierung. E-Mail-Adressen werden nicht geraten.

Ein Auftrag an Marc ist noch kein funktionierender Transfer. Ein Terraform-Patch ist noch keine bereitgestellte Alarmierung. Ein angelegter Login ist noch kein erfolgreicher Login-Test.

**Keine Nachricht wurde verschickt, kein Jira geändert und nichts in Azure oder auf einem Server ausgerollt.** Die mitgelieferten Änderungen sind lokal geprüfte Arbeitsvorschläge, keine behauptete Produktionslösung.

Quellenkennungen wie **B1** findest du im [Prüfprotokoll](06_Codebefunde_und_Pruefprotokoll.md).
