# 4. DBI-7161: Die Verbindung zwischen ProductHub und UMP konkret vorbereiten

**Priorität:** nach den laufenden Migrationsthemen. Rückfragen zu den Schnittstellen kannst du sofort losschicken. **Standalone:** ja; die anderen drei Tickets sind keine Voraussetzung.

## Worum geht es?

Der ProductHub soll Änderungen als Nachrichten an die UMP senden. Azure Event Grid vermittelt die Zustellung. Nachrichten, die nicht zugestellt werden können, sollen in einer vorgesehenen Fehlerablage landen. [J4]

Du brauchst keine persönliche Erklärung zu Marcels altem Codeverweis, um weiterzuarbeiten. Der Verweis heißt `eventhub`, das Ticket fordert jedoch Event Grid. Im Upload liegt diese externe Vorlage nicht. Außerdem habe ich im geprüften Anwendungscode keine fertige Event-Grid-Sendefunktion und keinen zugehörigen UMP-Empfänger nachgewiesen. Ein Slack-Webhook in einer Logging-Konfiguration ist dafür kein Beleg. [B6]

## 1. Mit einem konkreten Arbeitsvorschlag starten

**Das ist der technische Vorschlag, keine bereits genehmigte Architektur:**

| Teil | Was du vorbereitest |
|---|---|
| ProductHub sendet | Eigene technische Identität mit gezielter Sendeberechtigung für das Event-Grid-Topic. |
| Azure nimmt an | Ein Event-Grid-Custom-Topic in der vereinbarten Umgebung. Kein Austausch gegen Event Hubs allein wegen des alten Links. |
| Azure stellt zu | Eine Event Subscription für die vereinbarte UMP-HTTPS-Schnittstelle. |
| UMP nimmt entgegen | Eine authentifizierte Schnittstelle mit Eingangsprüfung und Schutz vor mehrfacher fachlicher Verarbeitung. |
| Zustellung scheitert | Ein tatsächlich angebundener Blob-Container für nicht zustellbare Nachrichten sowie ein klarer Bearbeitungsweg. |

Die technische Anmeldung muss für die beiden Verbindungen getrennt werden. Für natives Event-Grid-Publishing ist ein Entra-Zugriff mit passender Azure-Rolle vorgesehen; für einen geschützten UMP-Webhook gelten andere Rollen und Token-Prüfungen. Die im Jira genannten eigenen B2C-Scopes sind deshalb **nicht einfach die fertige native Event-Grid-Konfiguration**. Sollte davor ein eigener Vermittlungsdienst geplant sein, kann das eine andere Architektur sein. Diese Abweichung konkret mit Sinan entscheiden, nicht stillschweigend umsetzen. [J4, W4, W5]

## 2. Nur die wirklich fehlenden Fakten bei Sinan anfordern

**An:** Sinan Balcin. **Kanal:** Teams oder Jira.

```text
Hallo Sinan,

ich nehme DBI-7161 weiter auf und bereite die Azure-Seite direkt aus dem vorhandenen Code und den Ticketanforderungen vor.

Als Arbeitsvorschlag setze ich auf ein Event-Grid-Topic, eine Zustellung an die UMP und einen angebundenen Blob-Container für nicht zustellbare Nachrichten. Den alten Event-Hub-Codeverweis brauche ich dafür nicht als Startvoraussetzung.

Für die konkrete Anbindung fehlen mir noch drei Dinge: Welche Umgebung und welche UMP-Empfängeradresse sind vorgesehen? Wie sieht eine echte Beispielnachricht aus und was soll die UMP damit tun? Wer übernimmt bei OSC das Senden aus dem ProductHub und den Empfang in der UMP?

Einen Punkt möchte ich gezielt mit dir entscheiden: Das Jira nennt B2C-Scopes. Für eine direkte Event-Grid-Anbindung würde ich den dokumentierten Entra-/Rollenweg verwenden und Sender- sowie Empfängeranmeldung getrennt einrichten. Falls im Konzept stattdessen ein eigener Vermittlungsdienst vorgesehen ist, brauche ich diese Information vor dem Aufbau.

Bitte gib mir außerdem Zugriff auf die beiden im Jira genannten Konzeptanhänge. Die Bestandsprüfung und die technische Vorbereitung führe ich parallel weiter.

Danke dir!
```

- [ ] Vorhandene Azure-Ressourcen und den aktuellen Infrastruktur-Branch prüfen. Nicht allein aus dem ZIP folgern, dass live noch nichts existiert.
- [ ] Den vorgesehenen Testfall schriftlich festhalten: Änderung im ProductHub → Nachricht → erwartete Reaktion der UMP.
- [ ] Die Authentifizierungsabweichung und die nötigen Entscheidungen im Jira dokumentieren.

**Falls Sinan kurzfristig nicht antwortet:** Die Bestandsaufnahme und ein lokaler Infrastrukturentwurf können weitergehen. Keine unbestätigte Empfängeradresse und keine geratenen Anwendungsberechtigungen produktiv bereitstellen. Das ist eine fehlende Schnittstellenvorgabe, kein Urlaubsblocker der drei genannten Personen.

## 3. Die Anwendungsarbeit direkt mit OSC abgrenzen

**An:** Alex Köster. **Kanal:** Teams oder Jira. Falls Sinan bereits andere Anwendungskontakte genannt hat, nutze diese Zuordnung und spare die doppelte Anfrage.

```text
Hallo Alex,

ich bereite DBI-7161 für den Nachrichtenweg vom ProductHub zur UMP vor. Die Azure-Ressourcen und deren Berechtigungen übernehme ich auf unserer Seite.

Im hochgeladenen Anwendungsstand habe ich noch keine fertige Event-Grid-Sendefunktion und keinen dazugehörigen UMP-Empfänger gefunden. Kannst du mir bitte sagen, ob diese Umsetzung in einem anderen Branch liegt oder wer sie bei euch übernimmt?

Für den gemeinsamen Test brauche ich eine Beispielnachricht und die vorgesehene Empfängeradresse. Wichtig ist außerdem, dass die UMP wiederholte Zustellungen nicht als neue fachliche Aktion verarbeitet und die erstmalige Einrichtung der Event Subscription unterstützt.

Dann können wir Infrastruktur und Anwendungsteile parallel vorbereiten, statt nacheinander aufeinander zu warten. Danke dir!
```

## 4. Den Azure-Aufbau selbst umsetzen

- [ ] Nach der Schnittstellenentscheidung einen passenden neuen Baustein in `catalog` und die konkrete Nutzung in `live` vorbereiten. Vorhandene Namens-, Tagging- und Umgebungsregeln wiederverwenden.
- [ ] Das Topic, die Event Subscription und den Fehler-Container als zusammenhängenden Aufbau abbilden.
- [ ] Nur die notwendige Sendeberechtigung für den ProductHub vergeben. Ein technischer Sender benötigt keinen allgemeinen Administratorzugriff auf die Subscription.
- [ ] Die erlaubte Herkunft des extern betriebenen ProductHub anhand seiner echten ausgehenden Netzwerkadressen festlegen. Keine Adressen raten.
- [ ] Prüfen, ob Event Grid den UMP-Empfänger tatsächlich erreichen kann. Ein privater UMP-Endpunkt ist nicht automatisch vom Zustelldienst erreichbar. Den unterstützten Zustellweg konkret testen. [W5, W7]
- [ ] Die Fehlerablage an die Event Subscription anschließen und den erforderlichen Schreibzugriff einrichten. Ein nur angelegter Storage Account genügt nicht. [W6]
- [ ] Erst in der bestätigten Testumgebung bereitstellen und den Plan auf unerwartete Veränderungen prüfen lassen.

Ein vollständiger ausführbarer Infrastruktur-Patch ist hier noch nicht beigefügt: Zieladresse, Netzwerkfreigaben und der gültige Authentifizierungsvertrag fehlen. Diese Werte zu erfinden würde ein anderes System bauen. Der obige Vorschlag begrenzt die offenen Entscheidungen, statt das gesamte Ticket bis zur Rückkehr eines Kollegen liegenzulassen.

## 5. Erfolg und Fehlerfall gemeinsam testen

- [ ] Eine freigegebene Nachricht aus dem tatsächlichen ProductHub senden und ihre erwartete Verarbeitung in der UMP nachweisen.
- [ ] Einen kontrollierten vorübergehenden Empfängerfehler testen und die Wiederholung prüfen.
- [ ] Einen dauerhaften Zustellfehler nach dem vereinbarten Wiederholungs-/Ablaufverhalten prüfen: Die Nachricht muss nachvollziehbar in der Fehlerablage ankommen. Nicht davon ausgehen, dass jeder Fehler sofort dort landet. [W6]
- [ ] Dieselbe Nachricht erneut zustellen. Sie darf die fachliche Aktion nicht ungewollt verdoppeln.
- [ ] Einen unberechtigten Zugriff zurückweisen lassen.
- [ ] Dokumentieren, wer Fehlernachrichten prüft und wie eine freigegebene erneute Verarbeitung erfolgt.

### Nach vorhandenem technischem Testfall, falls die fachliche Abnahme noch fehlt: an Maike Schmillen

```text
Hallo Maike,

für DBI-7161 bereiten wir die Übertragung von ProductHub-Änderungen an die UMP vor. Den technischen Test stimmen wir mit den Anwendungskontakten ab.

Kannst du bitte bestätigen, woran du beim vereinbarten Beispiel erkennst, dass die Änderung in der UMP fachlich richtig verarbeitet wurde, und wer diese Prüfung abnimmt? So können wir neben der Zustellung auch das tatsächliche Ergebnis dokumentieren.

Danke dir!
```

## Fertig, wenn …

- [ ] Der tatsächliche ProductHub erreicht Event Grid und die UMP verarbeitet die Nachricht korrekt.
- [ ] Der Fehlerfall einschließlich Ablage und erneuter Verarbeitung ist geprüft.
- [ ] Zuständigkeiten, Berechtigungen, Netzwerkweg und Abnahme sind dokumentiert.

**Echte Abhängigkeit:** Für den Gesamttest müssen Sender und Empfänger bereit sein. Keines der anderen drei Tickets ersetzt diese Arbeit.

Quellen: **J4, B6, W4 bis W7** im [Prüfprotokoll](06_Codebefunde_und_Pruefprotokoll.md). [Zurück](README.md)
