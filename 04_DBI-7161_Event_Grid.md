# 4. DBI-7161: Nachrichten vom ProductHub zur UMP ermöglichen

**Priorität:** Unterlagen früh anfragen, die Umsetzung nach der Klärung beginnen. **Standalone:** ja, keines der anderen drei Tickets muss dafür abgeschlossen sein.

**Deine Ansprechpartner:** Sinan Balcin ist als IT-Kontakt und Maike Schmillen als Fachkontakt genannt. Marcel Sebbin hat eine mögliche Codevorlage verlinkt. Wer die Anwendungsseite bei OSC übernimmt, muss noch konkret benannt werden. [1]

## Worum geht es?

Der ProductHub soll Änderungen als Nachrichten an die UMP weitergeben können. Event Grid ist der dafür im Ticket vorgesehene Azure-Dienst. Wenn eine Nachricht nicht zugestellt werden kann, soll sie in einer vorgesehenen Fehlerablage landen. Im Ticket heißt diese Ablage **Dead Letter**. [1]

Das besteht aus drei Teilen: Der ProductHub sendet, Azure stellt zu und die UMP nimmt entgegen. Nur Azure-Ressourcen anzulegen reicht daher nicht als Gesamtnachweis.

**Der bisherige Export enthält noch keine ausreichende Bauanleitung:** Das Konzept-PDF und die Zeichnung werden zwar im Jira genannt, sind im Upload aber nicht enthalten. Außerdem verweist Marcel auf eine Datei namens `eventhub.tf`, während das Ticket Event Grid fordert. Der Name allein beweist nicht, ob diese Vorlage passt. [1, 2]

## Schritt 1: Sinan nach dem aktuellen Konzept fragen

### Nachricht A: An Sinan Balcin

**Kanal:** Teams oder Kommentar in DBI-7161 mit Markierung von Sinan.

```text
Hallo Sinan,

ich bereite DBI-7161 zur Verbindung von ProductHub und UMP vor und möchte vor dem Aufbau den aktuellen Stand verstehen.

Im Jira sind das Konzept 20251205_Konzept_UMP_ProductHub_IV_v2.1.pdf und azure-schaubild.png genannt. Kannst du mir bitte die aktuell gültigen Unterlagen zugänglich machen und kurz sagen, ob Teile davon inzwischen umgesetzt sind?

Hilfreich wäre außerdem, welche Umgebungen wir aufbauen sollen und wer bei OSC für das Senden aus dem ProductHub sowie den Empfang in der UMP zuständig ist. Dann kann ich die Azure-Arbeiten passend abgrenzen.

Danke dir!
```

- [ ] Das aktuelle Konzept und die Zeichnung lesen.
- [ ] Von Sinan bestätigen lassen, welcher Stand gilt und welche Umgebungen dazugehören.
- [ ] Die Ansprechpartner für ProductHub und UMP im Jira festhalten.
- [ ] Prüfen, ob bereits Azure-Ressourcen oder Code für dieses Vorhaben existieren.

**Wenn die Unterlagen fehlen:** Noch keine endgültige technische Umsetzung festlegen. Die Beschaffung weiterverfolgen und währenddessen die anderen Tickets bearbeiten.

## Schritt 2: Marcels Codeverweis einordnen

**Diese Nachfrage kann parallel zu Schritt 1 laufen.**

### Nachricht B: An Marcel Sebbin

**Kanal:** Teams oder Jira-Kommentar mit Markierung von Marcel.

```text
Hallo Marcel,

in DBI-7161 hast du am 19.08. eine Terraform-Datei aus dem eventhub-Repository verlinkt. Im Ticket selbst ist Event Grid vorgesehen.

Kannst du mir bitte kurz sagen, ob der Link als allgemeine Vorlage gedacht war oder ob es dazu schon eine abgestimmte Lösung für dieses Ticket gibt? Falls inzwischen eine andere Vorlage oder bereits eine Umsetzung existiert, wäre der passende Codeverweis hilfreich.

Ich möchte auf dem richtigen Stand aufsetzen und nichts doppelt bauen. Danke dir!
```

- [ ] Die tatsächlich gemeinte Vorlage lesen.
- [ ] Festhalten, welche Teile verwendbar sind und welche für dieses Ticket noch fehlen.
- [ ] Nicht allein aus dem Dateinamen ableiten, dass die Vorlage falsch oder bereits ausreichend ist.

## Schritt 3: Einen einfachen Ablauf und die Zuständigkeiten festlegen

Bevor du implementierst, müssen diese Fragen beantwortet sein:

- [ ] **Was wird gesendet?** Ein konkretes Beispiel für eine ProductHub-Änderung und die erwartete Reaktion der UMP sind beschrieben.
- [ ] **Wie kommen die Systeme zusammen?** Die Ansprechpartner haben für beide Übertragungswege die Zugänge, Berechtigungen und Netzwerkfreigaben bestätigt.
- [ ] **Was passiert bei einem Fehler?** Wiederholungen, Fehlerablage und Zuständigkeit für eine spätere erneute Verarbeitung sind vereinbart.

Im Ticket stehen bereits technische Vorgaben zur Anmeldung der beteiligten Systeme. Lass sie mit den Verantwortlichen für beide Verbindungen bestätigen: **ProductHub zu Event Grid** und **Event Grid zur UMP**. Das ist nicht dieselbe Aufgabe wie der Browser-Login aus PB2B-27849. [1]

### Nachricht C: An Sinan Balcin zur gemeinsamen Abstimmung

**Wann:** nachdem die aktuellen Unterlagen vorliegen und du sie gelesen hast; nur für noch offene Punkte.

```text
Hallo Sinan,

ich habe mir die Unterlagen zu DBI-7161 angesehen. Bevor ich die Azure-Seite aufbaue, würde ich die noch offenen Übergänge gern mit dir und den benannten Ansprechpartnern durchgehen.

Mir ist wichtig, dass wir für ProductHub zu Event Grid und für Event Grid zur UMP jeweils festhalten, wie Anmeldung und Netzwerkzugriff funktionieren und wer den jeweiligen Anwendungsteil bereitstellt.

Außerdem brauchen wir einen gemeinsamen Testfall und eine klare Zuständigkeit für Nachrichten, die in der Fehlerablage landen. Können wir die offenen Punkte gemeinsam im Ticket festhalten?

Danke dir!
```

### Nachricht D: Nur bei fehlendem fachlichem Testfall an Maike Schmillen

**Wann:** falls das gültige Konzept und die Abstimmung mit Sinan noch nicht erklären, woran der Fachbereich eine erfolgreiche Verarbeitung erkennt.

```text
Hallo Maike,

ich unterstütze bei DBI-7161, damit der ProductHub Änderungen an die UMP weitergeben kann. Für den gemeinsamen Test möchte ich sicherstellen, dass wir den richtigen fachlichen Ablauf prüfen.

Kannst du mir bitte ein konkretes Beispiel nennen, welche Änderung im ProductHub welche Reaktion in der UMP auslösen soll? Hilfreich wäre auch, woran du erkennen würdest, dass der Ablauf korrekt funktioniert hat, und wer das fachlich bestätigen kann.

Dann können wir beim Test mehr nachweisen als nur eine technisch zugestellte Nachricht. Danke dir!
```

## Schritt 4: Den abgestimmten Azure-Teil umsetzen

**Ab hier beginnt dein eigener Umsetzungsteil. Voraussetzung sind die Entscheidungen aus Schritt 3.**

- [ ] Event Grid, die vereinbarte Zustellung und die Fehlerablage im bestehenden Infrastrukturcode abbilden. Einen vorhandenen passenden Baustein wiederverwenden.
- [ ] Die vereinbarten Berechtigungen und Netzwerkwege ergänzen.
- [ ] Die Fehlerablage tatsächlich mit der Zustellung verbinden. Ein leerer Storage Account allein erfüllt das Ticket nicht.
- [ ] Änderungen prüfen lassen und zunächst in die vereinbarte Testumgebung ausrollen.
- [ ] Mit den benannten Anwendungskontakten abgleichen, wann der ProductHub senden und die UMP empfangen kann.

Für Infrastrukturänderungen kommen voraussichtlich `catalog` und `live` infrage. Wo die nötigen Anwendungsänderungen entstehen, musst du mit den benannten Kontakten klären. Im geprüften Upload ist damit noch keine fertige Event-Grid-Implementierung nachgewiesen. [2]

**Die tatsächliche Abhängigkeit liegt hier:** Du kannst die Azure-Arbeiten vorbereiten, aber den Gesamttest nicht abschließen, solange der vereinbarte Sender oder Empfänger fehlt. Das macht die drei anderen Jiras nicht zu Vorgängern.

## Schritt 5: Zustellung und Fehlerfall gemeinsam testen

### Nachricht E: An Sinan Balcin zur Testkoordination

**Nur senden, wenn der vereinbarte Azure-Aufbau in der Testumgebung bereitsteht.**

```text
Hallo Sinan,

der vereinbarte Azure-Aufbau für DBI-7161 ist in der Testumgebung bereit. Für den Gesamttest brauche ich jetzt die Unterstützung der benannten ProductHub- und UMP-Ansprechpartner.

Kannst du bitte mit ihnen abstimmen, ob Sender und Empfänger testbereit sind und wir den gemeinsamen Test durchführen können?

Wir sollten sowohl den erfolgreichen Ablauf vom ProductHub bis zur Verarbeitung in der UMP als auch einen kontrollierten Zustellfehler prüfen. Im Fehlerfall muss nachvollziehbar sein, wo die Nachricht abgelegt wird und wer sie anschließend bearbeitet.

Danke dir!
```

- [ ] Eine abgestimmte Testnachricht aus dem tatsächlichen ProductHub-System senden und ihre Verarbeitung in der UMP bestätigen lassen.
- [ ] Einen vereinbarten Zustellfehler auslösen und nach dem festgelegten Wiederholungsablauf prüfen, ob die Nachricht in der Fehlerablage liegt.
- [ ] Prüfen, wie eine Nachricht erneut verarbeitet wird und dass eine doppelte Zustellung nicht unbeabsichtigt doppelte fachliche Aktionen auslöst.
- [ ] Gemeinsam prüfen, dass nur berechtigte Systeme senden beziehungsweise die UMP-Funktion aufrufen können.
- [ ] Ergebnisse im Jira dokumentieren und die technische sowie fachliche Bestätigung einholen.

Wenn ein Test fehlschlägt, die vereinbarte Verarbeitung kontrolliert anhalten und den Fehler eingrenzen. Bereits eingegangene Nachrichten und die Fehlerablage nicht unüberlegt löschen.

## Fertig, wenn …

- [ ] Der vereinbarte Nachrichtenweg vom ProductHub bis zur UMP nachweislich funktioniert.
- [ ] Nicht zustellbare Nachrichten wie vereinbart in der Fehlerablage landen.
- [ ] Klar ist, wer Fehler prüft und eine erneute Verarbeitung anstößt.
- [ ] Code, Testergebnisse und technische sowie fachliche Abnahme dokumentiert sind.

## Quellen

[1] `ump_jiras.zip` → `ump_jiras/DBI-7161.doc`: Beschreibung und Abnahmekriterien, IT-Kontakt Sinan Balcin, Fachkontakt Maike Schmillen, Anhangsliste und Marcels Kommentar vom 19.08.2026. [2] Inhaltsverzeichnisse der hochgeladenen Archive und Repository-Übersicht: Das Konzept, die Zeichnung und das verlinkte externe Terraform-Repository wurden nicht als entsprechende Dateien mitgeliefert. Das sagt nichts darüber aus, ob sie im aktuellen Jira beziehungsweise GitLab zugänglich sind.

[Zurück zur Übersicht](README.md)
