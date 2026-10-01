# 3. PB2B-27849: Prüfen, ob der zusätzliche Test-Login funktioniert

**Priorität:** nach dem Monitoring-Start. Vorziehen, falls damit gerade Migrationstests blockiert sind. **Standalone:** ja, keines der anderen drei Tickets ist dafür ein belegter Vorgänger.

**Dein erster Ansprechpartner:** Oliver Ehli. **Das Ticket ist im Export ihm zugewiesen.** Stimme deine Unterstützung deshalb mit ihm ab, statt die Aufgabe ungefragt zu übernehmen. [1]

## Worum geht es?

Für die Migrationstests wird eine zusätzliche UMP-Anmeldung gebraucht. Die reguläre Produktionsanmeldung soll dabei weiter funktionieren. [1]

Oliver hat am 22.09.2026 geschrieben, dass die temporäre App bereits angelegt ist. Er bittet noch darum, ihre Anmelde-Konfigurationsdaten im Mellon-Modul zu hinterlegen. Mellon ist der beteiligte Login-Baustein auf der UMP-Seite. [1, 2]

**Dein Auftrag ist zunächst ein Abgleich:** Ist die bestehende App schon mit der richtigen Testinstanz verbunden und funktioniert der Login? Du legst nicht vorsorglich eine zweite App an.

## Schritt 1: Oliver nach dem aktuellen Stand fragen

### Nachricht A: An Oliver Ehli

**Kanal:** Teams oder Kommentar in PB2B-27849 mit Markierung von Oliver.

```text
Hallo Oliver,

ich arbeite mich gerade in die offenen UMP-Themen ein und habe mir PB2B-27849 angesehen. Du hattest am 22.09. geschrieben, dass die temporäre App angelegt ist und noch die passenden Metadaten für Mellon hinterlegt werden sollen.

Ist das inzwischen eingebunden und der Login getestet? Falls noch etwas offen ist, unterstütze ich gern. Dafür wären die temporäre UMP-URL und der Ansprechpartner für die Konfiguration auf der UMP-Seite hilfreich.

Kannst du mir außerdem sagen, ob dadurch gerade Migrationstests blockiert sind? Dann würde ich die Unterstützung entsprechend vorziehen.

Danke dir!
```

**Je nach Antwort:**

- [ ] **Bereits fertig:** Oliver um den Testnachweis im Jira bitten. Kein zweites Mal konfigurieren.
- [ ] **Noch offen:** Mit Oliver klären, welchen konkreten Teil du übernimmst und wer auf der UMP-Seite unterstützt.
- [ ] **Aktuell ein Testblocker:** Den Integrationscheck vorziehen und die geänderte Reihenfolge kurz mit den Beteiligten abstimmen.

Für die Serverkonfiguration nennt dieser Jira-Export keine weitere Person. Lass Oliver den richtigen Kontakt benennen; Alex oder Marcel werden nicht allein deshalb zuständig, weil sie in anderen UMP-Tickets vorkommen.

## Schritt 2: Nur die zusätzliche Testinstanz richtig verbinden

**Dieser Schritt beginnt erst, wenn Zielinstanz und Zuständigkeit feststehen.**

- [ ] Die konkrete temporäre UMP-Adresse und die dazugehörige App festhalten.
- [ ] Gemeinsam mit dem benannten UMP-Ansprechpartner prüfen, welche Anmelde-Konfiguration diese Instanz tatsächlich verwendet.
- [ ] Die von Oliver bestätigten Daten nur dort einbinden, wo sie für die temporäre Instanz gebraucht werden.
- [ ] Vor einer Änderung den bisherigen Stand sichern und den normalen Freigabeweg verwenden.

**Die wichtigste Falle:** Im Jira steht die temporäre Konfiguration mit dem Namen `B2C_1A_signin_saml_ump_tmp`. Die hochgeladene reguläre Prod-Datei nennt dagegen `B2C_1A_signin_saml_ump`. Das ist nicht automatisch ein Fehler. Es können schlicht zwei unterschiedliche Anmeldungen sein. **Ersetze deshalb nicht pauschal die normale Prod-Konfiguration durch die temporäre.** [1, 2]

Wenn du die Zuordnung nicht sicher feststellen kannst, klärst du sie mit Oliver und dem benannten UMP-Ansprechpartner, bevor du etwas änderst.

## Schritt 3: Einen echten Login testen

### Nachricht B: An Oliver Ehli zur Testabstimmung

**Nur senden, wenn die Zuordnung geklärt und die temporäre Instanz für den Login-Test vorbereitet ist.**

```text
Hallo Oliver,

die temporäre UMP-Instanz ist für den Login-Test vorbereitet. Können wir jetzt gemeinsam prüfen, ob die Anmeldung über die dafür angelegte App bis in die richtige UMP-Instanz funktioniert?

Dafür brauchen wir einen freigegebenen Testbenutzer und die Unterstützung auf der UMP-Seite. Kannst du den Test begleiten oder die passende Ansprechperson dazunehmen?

Ich würde außerdem den regulären Produktionslogin kurz gegenprüfen, damit wir sicherstellen, dass die zusätzliche Konfiguration dort nichts verändert hat.

Danke dir!
```

- [ ] Mit einem berechtigten Testbenutzer an der temporären UMP anmelden.
- [ ] Prüfen, ob du danach in der richtigen UMP-Instanz landest und als der richtige Benutzer erkannt wirst.
- [ ] Den Test in einer frischen Browsersitzung wiederholen, damit ein bereits bestehender Login das Ergebnis nicht verfälscht.
- [ ] Gegenprüfen, dass der normale Produktionslogin weiterhin funktioniert.
- [ ] Die Ergebnisse im Jira festhalten. Keine Zugangsdaten oder vollständigen Anmeldetokens hineinkopieren.

**Wenn der Test scheitert:** Die genaue Stelle notieren, an der es hängt, und mit Oliver sowie dem UMP-Ansprechpartner prüfen. Bei einem durch deine Änderung verursachten Fehler den gesicherten vorherigen Stand gezielt wiederherstellen.

## Schritt 4: Oliver das Ergebnis übergeben

### Nachricht C: An Oliver Ehli zum Abschluss

**Nur senden, wenn die beschriebenen Tests erfolgreich waren und die Ergebnisse im Jira stehen.**

```text
Hallo Oliver,

der Login an der temporären UMP funktioniert im vereinbarten Test. Den regulären Produktionslogin haben wir ebenfalls geprüft; er funktioniert weiterhin. Die Ergebnisse sind in PB2B-27849 dokumentiert.

Kannst du bitte bestätigen, ob der Nachweis für die Migrationstests ausreicht und du das Ticket damit abschließen kannst?

Lass uns bitte noch festhalten, wer die temporäre App und ihre Konfiguration nach Abschluss der Tests wieder zurückbaut und wer dafür das Signal gibt.

Danke dir!
```

- [ ] Oliver die Abnahme beziehungsweise Aktualisierung seines Tickets überlassen, sofern keine andere Zuständigkeit vereinbart wurde.
- [ ] Verantwortlichen und Anlass für den späteren Rückbau dokumentieren. Nicht schon während laufender Migrationstests entfernen.

## Fertig, wenn …

- [ ] Der temporäre Login mit einem berechtigten Benutzer nachweislich funktioniert.
- [ ] Der reguläre Produktionslogin weiterhin funktioniert.
- [ ] Oliver die Abnahme bestätigt hat.
- [ ] Klar ist, wer die temporäre Lösung später wieder entfernt.

## Nur zum Nachschlagen

Im Repository `ansible-ump` ist `playbooks/configuration/files/app/mellon/prod/idp_metadata_azure.xml` die geprüfte reguläre Prod-Metadatendatei. Der Pfad allein sagt nicht, dass auch die temporäre Instanz diese Datei verwendet. [2]

**Quellen:** [1] `ump_jiras.zip` → `ump_jiras/PB2B-27849.doc`, Beschreibung, Zuweisung an Oliver und Kommentare vom 22.09.2026. [2] `UMP_repos.zip` → `UMP_repos/UMP_legacy/ansible-ump/playbooks/configuration/files/app/mellon/prod/idp_metadata_azure.xml`.

[Zurück zur Übersicht](README.md)
