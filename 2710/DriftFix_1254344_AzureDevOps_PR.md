# Pull Request in Azure DevOps

**Titel:** `fix(drift): inaktive PROD-Unit keyvault-aadb2c explizit ausschließen`

Den folgenden Markdown-Inhalt in das Beschreibungsfeld kopieren:

```markdown
## Anlass
Drift-Lauf 1254344 auf Commit d60fa0dbca924d974f191e4f890179c417793629 bricht in PROD ab: `keyvault-aadb2c` enthält ausschließlich Kommentare, wird aber als Terraform-Unit ausgeführt. 60 Units erfolgreich, 1 fehlgeschlagen.

## Änderung
- Nur `environment/prod/applications/mashub/keyvault-aadb2c/terragrunt.hcl` angepasst.
- Inaktiven Platzhalter mit `exclude` explizit aus Multi-Unit-Läufen ausschließen; Abhängigkeiten nicht mit ausschließen.
- Kein Key-Vault-Deployment, keine State-Änderung und keine Unterdrückung anderer Fehler.
- Terraform 1.16.7, Terragrunt 1.2.1 und die 5-GiB-Konfiguration bleiben unverändert.

## Prüfung
- [x] Dateivergleich: genau eine Repositorydatei geändert, keine hinzugefügt oder gelöscht.
- [x] Bestehende elf Drift-Parser-Tests erfolgreich.
- [ ] Frischen PROD-Drift-Lauf ausführen: kein Missing-files-Fehler für `keyvault-aadb2c`.
- [ ] Verbleibenden BlobCapacity-/Action-Group-Diff separat prüfen; Exitcode 2 bleibt ein Drift-Hinweis, kein Erfolg ohne Änderungen.

**Jira:** DBI-12755 (Zuordnung laut ursprünglichem Merge-Commit prüfen).
**Vorheriger PR:** 97076.
**Changes:** Bestehende TEST-/PROD-Changes um diese Ergänzung verlinken; tatsächliche Nummern vor Freigabe eintragen. Genehmigungen werden nicht vorausgesetzt.
```

Der vorherige PR 97076 ist laut Checkout-Log bereits gemergt. Für diese Korrektur einen **neuen Branch vom aktuellen `main`** und einen neuen PR verwenden. Die tatsächliche Jira-/Change-Zuordnung ist in Azure DevOps beziehungsweise ServiceNow zu prüfen; es wurde hier nichts angelegt oder geändert.
