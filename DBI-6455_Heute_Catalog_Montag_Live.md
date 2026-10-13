# DBI-6455: Catalog erledigt, Live-MR fertigstellen

**Stand: 12.10.2026.** Die Schritte gelten für deine nächste Arbeitssitzung (falls erst nächsten Montag: 19.10.2026).

## 1. Wo stehen wir?

- **Catalog:** MR ist laut Timos Nachricht gemerged. Timo hat **Tag `v0.13.1`** erstellt. **Hier nichts mehr taggen oder ändern.**
- **Live:** Dein bestehender MR **!56** (`feat/DBI-6455-test-law-alerts` nach `main`) soll fertiggestellt werden. **Keinen neuen Branch und keinen neuen MR erstellen.**
- **TEST:** Muss die gemeinsame `tg_catalog_version` `v0.13.1` verwenden. Die Monitoring-Unit darf nicht mehr separat auf den alten Catalog-Commit zeigen.
- **Deployment und Alarmmail-Test:** Noch nicht als erfolgreich belegt. **Nicht behaupten, dass das Ticket schon abgeschlossen ist.**

**Warum?** `catalog` definiert die wiederverwendbaren Azure-Monitor-Bausteine. Erst wenn `live/test` die neue Version referenziert und der TEST-Apply läuft, werden die Änderungen in Azure wirksam.

## 2. In WSL den bestehenden Live-Branch aktualisieren

Öffne **das originale `live`-Repository in VS Code (WSL-Terminal)**. Im Repository-Ordner:

```bash
pwd
git remote -v
git status --short
git fetch origin
git switch feat/DBI-6455-test-law-alerts
git pull --ff-only origin feat/DBI-6455-test-law-alerts
git status --short
```

**Prüfen:** `origin` muss auf `ump_legacy/infrastructure/live` zeigen. Wenn `git status --short` vor dem Kopieren offene Änderungen zeigt, **erst klären und nichts überschreiben**. Falls MR !56 bereits gemerged ist, statt dieser Anleitung zuerst den tatsächlichen Stand prüfen.

## 3. Fertige Dateien per Drag & Drop übernehmen

Im ZIP unter `DBI-6455/KOPIEREN/live/` liegt eine kleine **Repository-Root-Struktur**. Der Inhalt enthält **zwei vollständige Dateien**, keine Code-Schnipsel:

```text
KOPIEREN/live/                    <- Inhalt in den originalen live-Repo-Root ziehen
└── live/
    └── test/
        ├── terragrunt.stack.hcl   (vollständige Datei, für v0.13.1 angepasst)
        └── exception-alerts.hcl   (vollständige TEST-Alert-Konfiguration)
```

Führe im Terminal aus:

```bash
explorer.exe .
```

Ziehe **den Inhalt von `KOPIEREN/live/`** in den geöffneten Repository-Root und bestätige das Ersetzen der beiden Dateien. **Nicht den äußeren `live`-Ordner als zusätzliche Ebene hineinziehen!** Der Git-Pfad muss `live/test/terragrunt.stack.hcl` heißen, niemals `live/live/test/terragrunt.stack.hcl`.

**Die Änderungen gegenüber deinem bisherigen Live-MR sind nur:**

1. `tg_catalog_version = "v0.13.1"` statt `v0.13.0`.
2. `unit "monitoring"` referenziert `${local.tg_catalog_version}` statt eines einzelnen Commit-Hashes.
3. Der Kommentar zur Monitoring-Referenz wurde aktualisiert.

`exception-alerts.hcl` bleibt gegenüber dem letzten vorbereiteten MR-Stand unverändert. Andere Umgebungen werden nicht angefasst.

**Achtung:** Die Datei basiert auf deinem zuletzt hochgeladenen Repo-Snapshot. Wenn am bestehenden Live-Branch inzwischen weitere Änderungen in derselben TEST-Stackdatei gemacht wurden, **nicht blind ersetzen**. Das zeigt Schritt 4.

## 4. Vor Commit und Push den Diff prüfen

```bash
git status --short
git diff --check
git diff --stat
git diff -- live/test/terragrunt.stack.hcl
```

Erwartet: **nur die TEST-Stackdatei** mit den oben beschriebenen wenigen Zeilen. Die neue `exception-alerts.hcl` ist im MR bereits enthalten und sollte keinen zusätzlichen Diff erzeugen. **Wenn hunderte neue Zeilen, andere Dateien oder `live/live/...` auftauchen: STOPP, nicht committen.** Dann ist der Branch weitergelaufen oder der Kopierpfad falsch.

Wenn alles passt:

```bash
git add live/test/terragrunt.stack.hcl live/test/exception-alerts.hcl
git diff --cached --check
git diff --cached --stat
git commit -m "DBI-6455: use catalog v0.13.1 for TEST"
git push origin feat/DBI-6455-test-law-alerts
```

Wenn `git status --short` bereits leer ist und die Version `v0.13.1` stimmt, ist kein weiterer Commit nötig. Dann mit Schritt 5 weitermachen.

## 5. Bestehenden Live-MR !56 prüfen und mergen

Im GitLab-Projekt `ump_legacy/infrastructure/live` den **bestehenden MR !56** öffnen.

- Neuesten Commit und ggf. erforderliche neue Review-Freigabe prüfen.
- `checkTestEnvironment` (Featurebranch) und `planTestEnvironment` (MR) prüfen. Sie laufen laut der hochgeladenen CI-Konfiguration automatisch, sofern die Regeln greifen.
- Den **gesamten TEST-Plan** lesen: `v0.13.1` betrifft alle TEST-Units, die `local.tg_catalog_version` nutzen, nicht nur Monitoring.
- Erwartet sind im Monitoring die benötigte **Action Group** und **Log-Alert-Regel** im vorhandenen LAW. Keine neue komplette Monitoring-Landschaft.
- Bei unerwarteten Destroy-/Replace-Vorgängen an VM, DB, Gateway, Storage oder anderen Services **nicht mergen**, sondern mit Timo klären.

Wenn Plan und Freigaben passen: **MR !56 nach `main` mergen**.

**Wichtig:** Der Merge allein stellt noch nichts in TEST bereit und löst kein PROD-Deployment aus.

## 6. Manuellen TEST-Apply starten

Danach in GitLab im **`live`-Projekt**:

1. **Build → Pipelines** öffnen.
2. Die **neue `main`-Pipeline nach dem Merge** auswählen.
3. `planTestEnvironment` erfolgreich durchlaufen lassen und **diesen neuen Plan** prüfen, nicht den alten MR-Plan.
4. In derselben Pipeline **`applyTestEnvironment`** per Play/Run manuell starten. `usePlan` auf `true` lassen.
5. Erfolgreichen Jobabschluss prüfen. **Kein DEV-/PROD-Apply.**

Der Plan ist laut hochgeladener CI als Artifact für einen Tag konfiguriert. Falls der Plan abgelaufen ist, neuen Plan erzeugen, prüfen und dann erst anwenden. **Kein planloser Apply als Abkürzung.**

## 7. Funktionstest und Ticketabschluss

Nach erfolgreichem TEST-Apply mit Alex/OSC einen **kontrollierten echten Fehler** erzeugen und diese Kette prüfen:

```text
Anwendung → OpenTelemetry → OTelLogs (error/fatal)
→ Azure Monitor Alert (ServiceName) → Action Group
→ E-Mail an ui-team@os-cillation.de
```

**Nachricht an Alex (erst wenn TEST deployed ist):**

> Hi Alex, die error/fatal-Alarmierung ist in TEST ausgerollt. Können wir einen kontrollierten Fehler erzeugen und gemeinsam prüfen, ob er in OTelLogs ankommt, den Azure-Alert auslöst und die Mail bei ui-team@os-cillation.de eingeht? Danke dir!

**Jira-Zwischenkommentar (nach Merge und Apply, vor Mail-Test):**

> DBI-6455: Der Catalog-Baustein wurde als v0.13.1 veröffentlicht, die TEST-Konfiguration im Live-Repo nachgezogen und die Azure-Monitor-Alarmierung in TEST ausgerollt. Als nächstes folgt der End-to-End-Test mit einem kontrollierten error/fatal-Logeintrag und Bestätigung des E-Mail-Eingangs durch OSC.

**Jira-Kommentar nach erfolgreichem TEST-End-to-End-Test (Gesamtticket nur schließen, wenn TEST die vereinbarte Abnahme ist):**

> DBI-6455: Die error/fatal-Alarmierung in TEST wurde erfolgreich end-to-end geprüft. Ein kontrollierter Anwendungsfehler wurde in OTelLogs erfasst, der Azure Monitor Alert hat ausgelöst und die Benachrichtigung wurde bei ui-team@os-cillation.de empfangen. Das TEST-Ergebnis ist dokumentiert. Ein gegebenenfalls erforderlicher Rollout auf weitere Umgebungen bleibt separat offen.

## Kurz-Checkliste

- [ ] Catalog-Merge/Tag `v0.13.1` bestätigt (laut Timo erledigt)
- [ ] Bestehenden Live-Branch aktualisiert und die beiden TEST-Dateien geprüft
- [ ] `tg_catalog_version` auf `v0.13.1`; Monitoring nutzt `${local.tg_catalog_version}`
- [ ] Commit gepusht; MR !56 hat sauberen TEST-Plan und nötige Freigaben
- [ ] Live-MR gemerged
- [ ] Neuer `main`-Plan geprüft und `applyTestEnvironment` manuell erfolgreich
- [ ] Echte Alarmmail bestätigt, Jira aktualisiert

**Ticketabschluss:** Ein erfolgreicher TEST-Test allein beweist keine Abnahme für PROD. Das gesamte Jira nur schließen, wenn TEST laut abgestimmtem Scope ausreicht oder der Rest ausdrücklich separat beauftragt ist.

**Prüfstand:** Die fertigen Dateien wurden aus dem letzten im Chat hochgeladenen `live`-Snapshot abgeleitet und lokal auf die oben genannten Änderungen geprüft. Keine aktuelle GitLab-/Azure-Abfrage, kein tatsächliches Deployment und kein Mailtest durchgeführt.
