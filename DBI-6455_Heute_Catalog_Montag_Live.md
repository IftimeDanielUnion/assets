# DBI-6455: Heute Catalog mergen, Montag weiter

**Heute: Freitag, 09.10.2026. Weiterarbeit: Montag, 12.10.2026.**

## Die Antwort auf deine Pipeline-Frage

**Heute nur den freigegebenen Catalog-MR nach `main` mergen. Kein Live-Merge und kein Azure-Apply heute.** Der Catalog enthält den wiederverwendbaren Terraform-Baustein. Die konkrete TEST-Alarmierung wird erst später aus dem `live`-Repository bereitgestellt.

Im aktuellen Catalog-Upload liegt **keine eigene `.gitlab-ci.yml`**. Du musst dort nicht auf Verdacht eine Pipeline manuell starten. Falls GitLab durch eine zentrale Projekteinstellung trotzdem Prüfjobs ausführt, deren Ergebnis beachten. Solche Einstellungen sind im ZIP nicht sichtbar. Im hochgeladenen `live`-Repo sind Plan und manueller Apply dagegen ausdrücklich konfiguriert. [1][2]

Timo hat im Screenshot den Catalog-Merge freigegeben und für den besprochenen Vorgang bestätigt, dass kein separater Change erforderlich ist. Das ist keine allgemeine Freigabe für andere Deployments.

## 1. Heute: Nur Catalog abschließen

Im GitLab-Projekt **`ump_legacy/infrastructure/catalog`** deinen DBI-6455-MR öffnen.

1. Zielbranch `main`, Freigabe und gegebenenfalls vorhandene Prüfungen kontrollieren.
2. **Merge** anklicken. Nicht den Live-MR !56 mergen.
3. Warten, bis wirklich **Merged** angezeigt wird.
4. Die im MR verlinkte **Commit-ID des übernommenen Stands auf `main`** für Montag kopieren. Bei Squash-Merge ist das nicht unbedingt die ursprüngliche Featurebranch-ID.
5. Den Catalog-Quellbranch vorerst behalten: Der offene Live-MR verweist noch auf den bisherigen Catalog-Commit. Nach Umstellung auf den Tag kann der Branch aufgeräumt werden.

**Danach für heute stoppen. Tag, Live-Anpassung, Live-Merge und Deployment bleiben für Montag.**

### Nachricht an Timo, erst nach erfolgreichem Merge

> Hi Timo, danke für die Freigabe! Der Catalog-MR ist gemerged. Den Tag und die Umstellung im Live-MR nehme ich am Montag vor. Danach prüfe ich den TEST-Plan und starte den TEST-Apply manuell. Live bleibt bis dahin offen.

### Jira-Zwischenstand, erst nach erfolgreichem Merge

> Der Catalog-MR für die LAW-basierte Log-Alarmierung ist gemerged. Der Live-MR bleibt vorerst offen. Am Montag folgen der Catalog-Tag, die Umstellung der TEST-Monitoring-Referenz auf diesen Tag sowie Planprüfung, Merge und manueller TEST-Apply. Noch kein Deployment und noch keine End-to-End-Abnahme der Alarmmail durchgeführt.

## 2. Montag: Catalog-Tag anlegen

**Wofür?** Der Tag gibt dem freigegebenen Catalog-Stand einen festen Versionsnamen. `live` soll genau diese Version verwenden, nicht einen vorläufigen Featurebranch-Commit. [3]

In GitLab im **Catalog-Projekt**:

1. **Code > Tags** öffnen. Prüfen, ob Timo inzwischen bereits einen passenden Tag angelegt hat. Dann diesen verwenden, keinen zweiten anlegen.
2. Andernfalls **New tag** wählen und den nächsten freien Tag-Namen nach eurem vorhandenen Versionsschema verwenden. Die aktuelle Tags-Liste ist nicht im Upload enthalten; deshalb ist hier keine erfundene Versionsnummer vorgegeben.
3. Bei **Create from** die am Freitag notierte Commit-ID des gemergten `main`-Stands einfügen. Nicht ungeprüft den möglicherweise inzwischen weitergelaufenen `main` und nicht den alten Featurebranch wählen.
4. Als Nachricht: `DBI-6455: LAW-based scheduled log alerts`.
5. **Create tag** anklicken und den genauen Tag-Namen kopieren.

Bei einem bereits vorhandenen Tag über **Browse files** kontrollieren, dass er die DBI-6455-Änderung enthält: In `modules/log-and-monitor/main.alerts.tf` muss der Scope der Scheduled-Query-Regel auf `azurerm_log_analytics_workspace.main.id` zeigen. Einen vorhandenen Tag nicht nachträglich verschieben.

Falls du keine Tag-Berechtigung hast, Timo genau diese Tag-Erstellung übernehmen lassen. Der Tag selbst ist noch kein TEST-Deployment. [3]

## 3. Montag: Den bestehenden Live-MR auf den Tag umstellen

**Kein neuer Branch, kein neuer MR, kein altes ZIP erneut darüberkopieren.** Du arbeitest im bestehenden Live-MR **!56** weiter. Nur die Versionsreferenz der TEST-Monitoring-Unit wird aktualisiert.

Öffne dein originales `live`-Repository in **VS Code unter WSL**. Im Terminal zuerst:

```bash
cd "$(git rev-parse --show-toplevel)"
git remote -v
git status --short
```

`origin` muss auf `ump_legacy/infrastructure/live` zeigen. Bei offenen lokalen Änderungen erst stoppen und diese sichern, nichts zurücksetzen oder löschen.

Dann nacheinander ausführen; bei einem Fehler nicht mit dem nächsten Befehl fortfahren:

```bash
git fetch origin --prune
git switch feat/DBI-6455-test-law-alerts
git pull --ff-only origin feat/DBI-6455-test-law-alerts
```

In VS Code **`live/test/terragrunt.stack.hcl`** öffnen und nach `unit "monitoring"` suchen. Die vorbereitete Source-Zeile enthält aktuell:

```text
?ref=94343286e555a4e8384b49cfd2b825e6ed9bb79d
```

**Nur die Commit-ID hinter `?ref=` durch den eben kopierten Tag-Namen ersetzen.** Den restlichen Pfad und die Anführungszeichen stehen lassen. Den Kommentar direkt darüber kannst du passend ändern zu:

```hcl
# DBI-6455: pin only monitoring to the released catalog tag with LAW-scoped log alerts.
```

**Nicht die globale `tg_catalog_version` ändern.** Sonst würden weitere Units auf eine neue Catalog-Version wechseln. DEV, PROD und `exception-alerts.hcl` bleiben unverändert.

Speichern und prüfen:

```bash
git diff --check
git diff --stat
git diff -- live/test/terragrunt.stack.hcl
```

Für diesen zusätzlichen Commit darf nur die TEST-Stackdatei geändert sein: die Referenz und gegebenenfalls ihr Kommentar. Wenn dort eine andere Referenz steht oder zusätzliche Änderungen auftauchen, erst prüfen statt blind ersetzen.

Dann:

```bash
git add live/test/terragrunt.stack.hcl
git diff --cached --check
git diff --cached --stat
```

Wenn nur die erwartete Änderung vorgemerkt ist:

```bash
git commit -m "DBI-6455: pin TEST monitoring to released catalog tag"
git push origin feat/DBI-6455-test-law-alerts
```

Damit wird **der vorhandene MR !56** aktualisiert. Auch in dessen Beschreibung die alte Commit-Pinnung durch den tatsächlichen Tag-Namen ersetzen.

## 4. Montag: Plan prüfen, dann Live mergen

Der Push soll nach der hochgeladenen CI-Konfiguration automatisch Prüfungen auslösen. Du startest nicht zuerst per Hand irgendeine zusätzliche Pipeline. [1]

- **Featurebranch-Pipeline:** `checkTestEnvironment`.
- **MR-Pipeline:** `planTestEnvironment`.

Es ist nach dieser Konfiguration normal, dass Check und Plan nicht in derselben Pipeline stehen. Beide Ergebnisse müssen zum neuesten Stand mit Tag passen. Falls die erwarteten Jobs fehlen, zuerst die Pipeline-/Jobansicht prüfen und nicht ungeprüft mergen.

Im MR-Diff bleiben insgesamt die zwei bekannten Dateien:

```text
live/test/terragrunt.stack.hcl
live/test/exception-alerts.hcl
```

Der Plan soll die benötigte **Action Group und Log-Alert-Regel** im bestehenden Monitoring ergänzen. Keine neue Monitoring-Landschaft und kein neuer Workspace. Unerwartete Änderungen an Datenbanken, VMs, Storage oder Gateway vor einem Apply gesondert klären; der TEST-Plan kann auch andere offene Änderungen zeigen.

Passt der Plan und sind die erforderlichen Freigaben noch gültig, **Live-MR !56 nach `main` mergen**. Falls die Tag-Änderung die Freigabe zurückgesetzt hat, Timo erneut bestätigen lassen.

## 5. Montag: Erst jetzt wirklich nach TEST deployen

**Der Live-Merge ist noch kein Deployment.** In der hochgeladenen CI steht beim TEST-Apply `when: manual`. [1][2]

1. Im Projekt `live` **Build > Pipelines** öffnen.
2. Die **neue `main`-Pipeline nach diesem Live-Merge** auswählen.
3. Deren `planTestEnvironment` abwarten und den Plan nochmals lesen. Nicht den alten MR-Plan als Deployment-Freigabe verwenden.
4. In **derselben Pipeline** bei **`applyTestEnvironment`** auf Play/Run klicken und bestätigen.
5. Den gespeicherten Plan verwenden. **`usePlan` nicht auf `false` setzen.** Keinen DEV- oder PROD-Apply starten.
6. Erfolgreichen Jobabschluss und danach die Ressourcen in Azure prüfen.

Die Plan-Artefakte sind in eurer CI mit `expire_in: 1 days` konfiguriert. Deshalb Montag einen frischen Plan verwenden. Falls ein Plan fehlt oder nicht mehr aktuell ist, neu planen und prüfen statt ohne Plan anzuwenden. Ob GitLab einzelne Artefakte länger behält, hängt zusätzlich von dessen Aufbewahrungseinstellungen ab. [1][4]

Eine Pipeline kann wegen des ausstehenden manuellen Apply als **blocked** erscheinen; das ist nicht automatisch ein Fehler. [2]

Erwartete TEST-Ressourcen aus dem vorbereiteten Code:

```text
Action Group: log-ump-test-mag-osc-exceptions
Log-Alert:    ump-test-otel-error-fatal
Workspace:    log-ump-test
Empfänger:    ui-team@os-cillation.de
```

## 6. Montag: Alarmmail abnehmen

Mit Alex/OSC einen kontrollierten Anwendungsfehler erzeugen. Prüfen, ob der Eintrag mit `error` oder `fatal` in `OTelLogs` ankommt, die Regel auslöst und die Mail beim Team eingeht. Auch den ServiceName im Alarmkontext und die spätere Auflösung prüfen. Ein reiner Testversand der Action Group prüft nicht die gesamte Kette.

### Nachricht an Alex, erst nach erfolgreichem TEST-Apply

> Hi Alex, die error/fatal-Alarmierung ist jetzt in TEST ausgerollt. Können wir einen kontrollierten Fehler aus einer Anwendung erzeugen und gemeinsam prüfen, ob er in OTelLogs ankommt, den Alert auslöst und die Mail bei ui-team@os-cillation.de eingeht? Danke dir!

**Erst danach ist TEST abgenommen.** Ein gegebenenfalls noch benötigter Rollout in andere Umgebungen ist ein eigener nächster Schritt. DBI-6455 nicht allein wegen des Merges schließen.

## Grundlage

Repository-Prüfung gegen deinen letzten Upload `test.zip` und den bereits vorbereiteten Live-Kopierstand aus `DBI-6455_02_LIVE_Komplett.zip`. Keine aktuelle Live-Abfrage von GitLab oder Azure durchgeführt. Tag-Name, Merge-Ergebnis und tatsächliche Pipeline-Ergebnisse werden deshalb in GitLab geprüft, nicht vorausgesetzt.

[1] Projektquelle: `test.zip` > `live-main.zip` > `live-main/.gitlab-ci.yml`; die CI im vorbereiteten Live-Paket ist dazu unverändert. Catalog-Export ohne eigene `.gitlab-ci.yml`. Projekt-/Gruppeneinstellungen können zusätzliche CI vorgeben.

[2] GitLab: [Manuelle Jobs](https://docs.gitlab.com/ci/jobs/job_control/) und [abweichende CI-Konfigurationspfade](https://docs.gitlab.com/ci/pipelines/settings/).

[3] GitLab: [Tags anlegen und eine Commit-ID als Ausgangspunkt verwenden](https://docs.gitlab.com/user/project/repository/tags/).

[4] GitLab: [Job-Artefakte und Aufbewahrung](https://docs.gitlab.com/ci/jobs/job_artifacts/).
