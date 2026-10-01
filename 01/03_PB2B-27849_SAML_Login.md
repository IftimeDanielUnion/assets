# 3. PB2B-27849: Den bereits angelegten temporären Login anschließen

**Priorität:** nach dem Monitoring oder früher, wenn Migrationstests daran hängen. **Standalone:** ja. Das Jira ist im Export Oliver Ehli zugewiesen; du unterstützt die Integration, statt es ungefragt neu zuzuweisen.

## Worum geht es?

Oliver hat am 22.09.2026 bestätigt, dass die temporäre App angelegt ist. Er hat auch die Adresse ihrer Login-Metadaten angegeben. Eine neue App zu bestellen wäre daher nicht der richtige erste Schritt. [J3]

Metadaten sind hier die Beschreibung, über welchen Anmeldedienst sich die UMP authentifiziert. Im hochgeladenen PROD-Stand zeigen diese Daten auf die reguläre Policy `B2C_1A_signin_saml_ump`. Die temporäre App verwendet dagegen `B2C_1A_signin_saml_ump_tmp`. Das ist ein Prüfpunkt, aber noch kein Beweis für eine falsche Live-Konfiguration. [B4]

## 1. Die aktive Konfiguration selbst lesen

- [ ] Auf dem für die Migration vorgesehenen UMP-Server die aktiven Apache-Webseiten prüfen. Der reine Dateiname „prod“ reicht zur Zuordnung nicht.
- [ ] Feststellen, welche Domain diese Instanz tatsächlich bedient und ob sie produktiven Verkehr oder nur Migrationstests erhält.
- [ ] In der aktiven Konfiguration die verwendeten Mellon-Metadatenpfade lesen. Im Upload werden `/etc/apache2/mellon/idp_metadata_azure.xml` und `/etc/apache2/mellon/ump-app_metadata.xml` verwendet. [B4]
- [ ] Die Policy der Anmeldeseite, die UMP-Kennung und die Rücksprungadresse aus diesen Dateien prüfen. Dafür stehen reine Lesebefehle im [Technik-Anhang](05_Technik_zum_Nachschlagen.md#saml).

Du brauchst keinen Urlauber, um diese Dateien oder die bestehende Ansible-Zuordnung zu verstehen. Im Code ist auch nachvollziehbar, dass die Dateien abhängig von `mellon_file_stage` ausgerollt werden. [B4]

## 2. Nur die Zuordnung mit Oliver bestätigen

**An:** Oliver Ehli. **Kanal:** Teams oder Kommentar in PB2B-27849. Diese Nachricht behauptet noch keinen Zugriff auf die laufende Instanz.

```text
Hallo Oliver,

ich unterstütze bei der Anbindung des temporären UMP-Logins. Deine Rückmeldung vom 22.09. und die Metadatenadresse für B2C_1A_signin_saml_ump_tmp habe ich vorliegen. Eine weitere App-Anlage brauche ich daher nicht.

Ich prüfe die aktive Mellon-Konfiguration auf der vorgesehenen UMP-Instanz. Im Repository ist für PROD noch die reguläre Policy hinterlegt; ob das auch dem laufenden Stand entspricht, prüfe ich separat.

Kannst du mir bitte die für deine temporäre App hinterlegte UMP-Kennung und Rücksprungadresse bestätigen? Hilfreich ist außerdem, welche konkrete UMP-Adresse wir für den Migrationstest verwenden sollen. Dann kann ich genau diese Zuordnung prüfen, ohne den regulären Login umzubiegen.

Die serverseitige Prüfung und Dokumentation übernehme ich. Danke dir!
```

**Fehlt nur Serverzugriff:** Alex oder Matthias Dietrich als beteiligte OSC-Kontakte gezielt um den vorhandenen Zugriffsweg beziehungsweise eine begleitete Lesesitzung bitten. Matthias ist im Ansible-Code als Mitwirkender und im SFTP-Jira als Beteiligter an Serveranpassungen belegt; seine aktuelle Verfügbarkeit und Freigabeberechtigung sind nicht bestätigt. Nicht stattdessen auf einen Urlauber warten. [B4, J1]

## 3. Genau die betroffene Instanz ändern

- [ ] Die temporären Metadaten von Olivers vollständiger HTTPS-Adresse beziehen. Nicht im regulären, signierten XML lediglich `_tmp` an Zeichenketten anhängen.
- [ ] Die Werte mit Olivers bestätigter Zuordnung vergleichen.
- [ ] Nur für die identifizierte Migrationsinstanz eine getrennte temporäre Metadatenquelle beziehungsweise eine klare Inventar-Zuordnung hinterlegen.
- [ ] Die reguläre Metadatendatei nicht global ersetzen. Das vorhandene Ansible kopiert je Stage feste Dateien; eine direkte Serveränderung allein könnte beim nächsten Lauf überschrieben werden. [B4]
- [ ] Die bisherige wirksame Konfiguration geschützt sichern, die Änderung prüfen lassen, Apache-Konfiguration testen und erst dann gezielt neu laden.

**Hier ist bewusst kein pauschaler SAML-Patch enthalten:** Ohne die aktive Zuordnung wäre nicht geklärt, ob er die Testinstanz oder den regulären Login verändert. Die vorhandenen Quellpfade und Lesebefehle reichen für den Integrationscheck; die konkrete Auswahl muss aus dem laufenden System und Olivers App-Daten kommen.

## 4. Gemeinsam anmelden und Ergebnis festhalten

- [ ] Den Login mit einem dafür freigegebenen Testkonto über die bestätigte Migrationsadresse durchführen.
- [ ] Prüfen: richtige Anmeldeseite → Rückkehr zur richtigen UMP-Instanz → erwarteter Benutzerzugriff.
- [ ] Den normalen UMP-Login separat gegenprüfen, soweit dieser im gleichen Änderungsumfang liegt. Ein erfolgreicher temporärer Login beweist nicht, dass der reguläre unberührt geblieben ist.
- [ ] Keine SAML-Assertions, Cookies oder privaten Schlüssel ins Jira kopieren. Zeitpunkt, beteiligte Instanz und Ergebnis reichen für die erste Dokumentation.

### Erst nach erfolgreichem Test: an Oliver

```text
Hallo Oliver,

der temporäre Login über die abgestimmte UMP-Adresse wurde erfolgreich getestet. Die geprüfte Zuordnung und das Ergebnis habe ich in PB2B-27849 dokumentiert.

Kannst du bitte noch die App-Seite gegen den dokumentierten Test bestätigen? Dann können wir den Integrationspunkt gemeinsam abschließen. Bitte halte auch fest, wann die temporäre App wieder entfernt werden soll, damit sie nach der Migration nicht unbeabsichtigt bestehen bleibt.

Danke dir für die Unterstützung!
```

## Fertig, wenn …

- [ ] Die richtige temporäre App ist an die richtige Migrationsinstanz angebunden.
- [ ] Ein echter Login-Test war erfolgreich; Änderungen sind reproduzierbar dokumentiert.
- [ ] Die reguläre Zuordnung ist nicht versehentlich geändert worden.
- [ ] Für die spätere Entfernung der temporären Lösung ist ein Verantwortlicher benannt.

Quellen: **J3, B4** im [Prüfprotokoll](06_Codebefunde_und_Pruefprotokoll.md). [Zurück](README.md)
