# UMP: Dein Arbeitsplan mit fertigen Nachrichten

**Stand: 01.10.2026.** Grundlage sind die Jira-Exporte vom 30.09.2026 und die hochgeladenen Repositories. Spätere Änderungen in Jira oder Azure sind hier nicht geprüft.

## Damit fängst du an

| Reihenfolge | Ticket | Was du erreichen sollst | Deine erste Nachricht |
|---|---|---|---|
| **1** | [DBI-11149: Dateiübertragung](01_DBI-11149_SFTP.md) | Den offenen Zugang bei Lukas klären und die Übertragung testen lassen. | An **Marc Hintz**. Ohne Zugriff auf den bisherigen Mailverlauf zuerst an **Christoph Sievers**. |
| **2** | [DBI-6455: Fehlermails](02_DBI-6455_Exception_Monitoring.md) | Das Team bekommt eine E-Mail, wenn die Anwendung einen relevanten Fehler meldet. | An **Alex Köster** zu den Anforderungen; parallel an **Marcel Sebbin** zum vorhandenen Azure-Aufbau. |
| **3** | [PB2B-27849: Temporärer Login](03_PB2B-27849_SAML_Login.md) | Prüfen, ob der zusätzliche Login für die Migrationstests funktioniert. | An **Oliver Ehli**. Das Ticket ist im Export ihm zugewiesen. |
| **4** | [DBI-7161: Nachrichten zwischen Anwendungen](04_DBI-7161_Event_Grid.md) | Den Nachrichtenweg vom ProductHub zur UMP aufbauen und testen. | An **Sinan Balcin** für Konzept, aktuellen Stand und Ansprechpartner. |

Die Prioritäten im Export sind 1, 2, 4 und 4. Die konkrete Arbeitsreihenfolge ist eine Empfehlung. Die Ansprechpartner ergeben sich aus den jeweiligen Jira-Verläufen; Quellen stehen am Ende jeder Datei.

**Praktisch:** Schicke zuerst die SFTP-Anfrage ab. Während du auf die Antwort wartest, kannst du das Monitoring bearbeiten. Den Login-Stand und die Event-Grid-Unterlagen kannst du ebenfalls früh anfragen. Du musst dafür nicht erst Ticket 1 abschließen.

**Ausnahme:** Bestätigt Oliver, dass der Login gerade Migrationstests verhindert, ziehst du diesen Punkt vor.

## Bauen die Tickets aufeinander auf?

**Zwischen diesen vier Tickets ist keine zwingende technische Reihenfolge belegt.** Alle vier sind in diesem Sinn *standalone*: Du kannst sie beginnen, ohne ein anderes aus dem Paket abgeschlossen zu haben.

Trotzdem brauchst du zum Fertigstellen Hilfe:

| Ticket | Was zuerst vorhanden sein muss |
|---|---|
| SFTP | Die Gegenstelle bestätigt den Zugang. Danach kann der vereinbarte Transfer getestet werden. |
| Monitoring | Die Fehlerdaten kommen bereits in Azure an. Dann kannst du darauf eine E-Mail-Regel aufbauen. |
| Temporärer Login | Die temporäre App und die richtige UMP-Testinstanz sind bekannt. Dann lassen sich Konfiguration und Login prüfen. |
| Event Grid | Der geplante Nachrichtenweg ist abgestimmt. Für den Gesamttest müssen außerdem ProductHub und UMP vorbereitet sein. |

**Merksatz:** Auf eine Rückmeldung warten ist nicht dasselbe wie auf ein anderes Jira warten.

## So nutzt du die Nachrichten

In jeder Ticketdatei steht direkt beim Arbeitsschritt, **an wen**, **wann** und **mit welchem Text** du schreibst. Die Textblöcke kannst du kopieren. Sende zuerst nur die Startnachricht; spätere Vorlagen sind ausdrücklich an einen bestätigten Zwischenstand gebunden.

Für interne Kontakte nutzt du den bisherigen Teams-Chat oder einen Jira-Kommentar mit persönlicher Markierung. Im Export fehlen deren E-Mail-Adressen. Es werden deshalb keine Adressen geraten. Für Marc Hintz ist `hintz@msu.biz` im SFTP-Verlauf angegeben. Die Monitoring-Adresse `ui-team@os-cillation.de` ist der gewünschte **Alarmempfänger**, nicht automatisch der richtige Verteiler für sämtliche Projektfragen.

Die Vorlagen unterschreiben nicht mit einem geratenen Absendernamen. Deine bestehende E-Mail-Signatur kannst du verwenden. Es wurden keine Nachrichten verschickt und keine Jiras verändert.

## Wenn eine Antwort ausbleibt

Antworte im gleichen Verlauf, statt eine neue Runde an alle Beteiligten zu eröffnen. Diese Nachfrage passt ohne Namens- oder Datumsplatzhalter direkt unter deine ursprüngliche Nachricht:

```text
Ich wollte noch einmal freundlich nachfragen, ob du schon auf meine Nachricht schauen konntest. Mir hilft auch ein kurzer Zwischenstand, was noch offen ist und wann du voraussichtlich eine Rückmeldung geben kannst. Falls ich dafür bei jemand anderem richtig bin, nenn mir gern die passende Ansprechperson. Danke dir!
```

## Wann du einen Haken setzen kannst

Eine Anfrage ist erledigt, sobald sie versandt ist. Der zugrunde liegende Arbeitsschritt ist erst erledigt, wenn die benötigte Antwort oder der Test vorliegt. Halte Antworten, Testergebnisse und offene Punkte im jeweiligen Jira fest. Schließe ein Ticket nicht allein deshalb, weil eine Azure-Ressource angelegt wurde oder jemand einen Schlüssel hinterlegt hat.
