# Monitoring-Patches: So nutzt du die vorbereitete Änderung

## Umfang

| Datei | Verändert im Repository `live` | Zustand nach Bereitstellung |
|---|---|---|
| `monitoring-test.patch` | `live/test/terragrunt.stack.hcl` | Alarmregel aktiviert. Vorher Logabfrage prüfen und Alex über den Test informieren. |
| `monitoring-dev.patch` | `live/dev/terragrunt.stack.hcl` | Alarmregel aktiviert. Optional, kein Pflichtschritt. |
| `monitoring-prod.patch` | `live/prod/terragrunt.stack.hcl` | Alarmregel deaktiviert. Die spätere Aktivierung ist ein eigener bewusster Schritt. |

Die Patches sind unabhängig. Sie verändern weder Provider-Versionen noch Datenbanken, Netzwerke oder Anwendungscode. Sie nutzen die vorhandenen Eingaben `action_groups` und `scheduled_log_alerts` des bereits referenzierten Katalogs. [B1, B2]

## Vor der Übernahme

Prüfe den aktuellen Branch und den realen Azure-Stand. Gibt es bereits neuere Alarmregeln, ergänzt du nicht blind eine zweite. Die getestete Basis ist der hochgeladene Snapshot, nicht automatisch der heutige Hauptbranch.

Die Log-Abfrage muss im Scope der verwendeten Application-Insights-Ressource funktionieren. Der Patch verwendet **nicht** automatisch den Log-Analytics-Workspace als Alarm-Scope. Im Katalog ist Application Insights fest vorgegeben. Daten, Berechtigungen, Tabellenplan und Regionsunterstützung sind im Tenant zu prüfen. Bei einer fehlgeschlagenen Abfrage nicht `skip_query_validation` als Umgehung einschalten. [B2, W1, W2]

## TEST-Patch lokal anwenden

Öffne dein Terminal im Wurzelverzeichnis des Repositorys `live`. Kopiere aus diesem Paket die Datei `monitoring-test.patch` direkt in dieses Verzeichnis. Dann kannst du den folgenden Block unverändert verwenden:

```bash
set -euo pipefail

test -f live/test/terragrunt.stack.hcl
test -f monitoring-test.patch

git status --short
git apply --check monitoring-test.patch
git apply monitoring-test.patch
git diff -- live/test/terragrunt.stack.hcl
```

Das ändert **nur deine Arbeitskopie**. Es wird nichts bereitgestellt, committed oder gepusht. Prüfe den Diff vor dem Commit. Nimm die kopierte `.patch`-Datei nicht versehentlich mit in den Commit auf.

Wenn `git apply --check` scheitert, nicht mit Gewalt fortfahren. Der aktuelle Stand weicht dann von der geprüften Basis ab. Vergleiche den Monitoring-Block und übertrage nur die fehlenden Werte nach Review.

## Was die Regel genau macht

Sie prüft alle 60 Sekunden ein fünfminütiges Fenster, zählt Logzeilen mit `SeverityText` gleich `error` oder `fatal` unabhängig von Groß-/Kleinschreibung und löst bei mehr als null Treffern aus. Die Gruppierung erfolgt ausschließlich nach `ServiceName`; ein leerer Name wird als `unknown-service` sichtbar. Die Zeitspanne ist keine garantierte Mail-Laufzeit. [W1, W2]

Die Abfrage liefert ausdrücklich **keinen Nachrichtentext, Stacktrace, Benutzernamen oder Request-Inhalt** in den ausgewählten Ergebnisfeldern. Für die Fehlersuche bleibt der normale Zugriff auf die Logs erforderlich.

Das bestehende Modul setzt `auto_mitigation_enabled` nicht. Für die im Upload vorgegebene AzureRM-4.x-Familie wurde die Dokumentation von 4.63.0 geprüft; sie nennt als Default `false`. Der Entwurf setzt daher keinen zustandsbasierten Einmalalarm voraus. Überlappende Fenster und die Azure-Benachrichtigungslogik können zu Wiederholungen führen. Den tatsächlich aufgelösten Provider-Stand und das Verhalten im Plan beziehungsweise Test prüfen. [B1, W2]

**Kein dynamischer Betreff:** Eine Dimension stellt den Dienstnamen im Alarmkontext bereit, aber nicht automatisch im E-Mail-Betreff. Azure unterstützt eine separate `Email.Subject`-Eigenschaft; der vorhandene Modulvertrag und die geprüfte Ressourcendokumentation bilden sie nicht als solches Feld ab. `custom_properties` ist kein gleichwertiger Ersatz. Der Betreffwunsch bleibt separat zu behandeln. [W3]

## Validieren und bereitstellen

Verwende die Projektwerkzeuge und den bestehenden Ablauf für die Monitoring-Unit. Im Upload sind Terraform `1.13.4` und Terragrunt `1.1.6` eingetragen; `root.hcl` enthält die Provider-Vorgaben. Für dieses kleine Ticket keine Versionsmigration nebenbei anfangen. [B1]

Vor einer Bereitstellung müssen mindestens die HCL-/Terraform-Validierung, die echte Logabfrage, der State-Abgleich und der Plan der Zielumgebung erfolgreich sein. Falls eure Pipeline nur Gesamtpläne erzeugt, unerwartete Änderungen außerhalb von Monitoring aussortieren beziehungsweise gesondert klären.

Diese Schritte wurden hier **nicht** gegen eure Infrastruktur durchgeführt. Die konkrete Prüfung im Paket ist im [Prüfprotokoll](../06_Codebefunde_und_Pruefprotokoll.md) aufgeführt.

## Zurücknehmen

Solange du die lokale Änderung noch nicht weiterbearbeitet oder committed hast, kannst du im gleichen Repository den TEST-Patch wieder zurücknehmen:

```bash
set -euo pipefail

git apply --reverse --check monitoring-test.patch
git apply --reverse monitoring-test.patch
git diff -- live/test/terragrunt.stack.hcl
```

Ist die Änderung bereits bereitgestellt, entfernt ein lokales Rückwärts-Patchen keine Azure-Ressource. Bei ungewollten Mails zuerst die betroffene Regel über den vorgesehenen Betriebsweg deaktivieren und diese Änderung im Code nachvollziehen. Kein unüberlegtes Löschen des gesamten Monitoring-Aufbaus.

## Enthaltene Log-Abfragen

`01_logeingang_pruefen.kql` zeigt den vorhandenen Logeingang ohne Inhalte der Fehlermeldungen. `02_alert_abfrage.kql` enthält exakt die fachliche Alarmabfrage. `03_filter_selbsttest.kql` erzeugt nur eine virtuelle Testtabelle und soll zwei Treffer liefern; sie erzeugt selbst keinen echten Logeintrag und löst keinen End-to-End-Test aus.

[Zurück zum Monitoring-Ticket](../02_DBI-6455_Exception_Monitoring.md)
