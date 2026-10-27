# GitLab Duo in VS Code: Selbsttest

**Prüfdatum:** 27.10.2026  
**Ziel:** Herausfinden, ob GitLab Duo Chat, Agentic Chat und die Modellauswahl funktionieren und ob das Problem am Projekt, Namespace, Server oder der Verbindung liegt.  
**Dauer:** etwa 10 bis 20 Minuten  
**Arbeitsumgebung:** Windows, VS Code, firmeninternes GitLab (Self-Managed)

## 0. Ausgangslage aus den Logs vom 22.10.2026

| Bereich | Bisheriger Befund |
|---|---|
| VS Code | 1.106.2 |
| GitLab Extension | 7.2.5 |
| GitLab Language Server | 9.29.0 |
| GitLab Server | 19.4.2-ee |
| Anmeldung / Lizenz | Diagnose erfolgreich |
| Code Suggestions | `On` laut Diagnose, Funktion noch praktisch testen |
| Non-Agentic (klassischer) Chat | `On` laut Diagnose, Funktion noch praktisch testen |
| Agentic Chat | `Off`: `Agentic Chat is supported for the current project (false)` |
| Agent Platform | In den lokalen Einstellungen aktiviert |
| Sandbox | Unter Windows nicht unterstützt; für diesen Chat-Test nicht der erste Ansatz |
| MCP | In den bisher geprüften Befunden **nicht** als Ursache nachgewiesen |

**Wichtig:** Ein grüner Diagnosecheck bestätigt die Berechtigung bzw. Konfiguration, nicht automatisch eine erfolgreiche Antwort eines Sprachmodells.

## 1. Schnelltest: klassischer GitLab Duo Chat

- [ ] In VS Code **ein einziges GitLab-Repository** öffnen. Nicht zuerst den großen Multi-Root-Workspace mit vielen Ordnern verwenden.
- [ ] Eine Quellcodedatei öffnen und Code markieren.
- [ ] `Strg + Shift + P` drücken und **`GitLab Duo Chat: Open Quick Chat`** ausführen. Alternativ Rechtsklick auf markierten Code > **GitLab Duo Chat** > **Open Quick Chat**.
- [ ] Folgende Frage senden:

```text
Erkläre den markierten Code in zwei kurzen Sätzen.
```

**Ergebnis:**  
- [ ] Antwort erhalten  
- [ ] Fehlermeldung erhalten: __________________________  
- [ ] Quick Chat gar nicht verfügbar

**Bewertung:** Wenn der klassische Chat antwortet, funktioniert mindestens eine Modellanfrage. Dann liegt das Agentic-Problem getrennt davon. Scheitert bereits der klassische Chat, bitte GitLab-/Language-Server-Logs zum exakten Zeitpunkt prüfen.

## 2. Schnelltest: Agentic Chat und Modell

- [ ] Das Panel **GitLab Duo Agent Platform** in VS Code öffnen. In Version 7.2.5 ist das neue Layout mit Agent-/Modellauswahl normal.
- [ ] Auf **Pick Model** klicken und prüfen, ob Modelle angeboten werden.
- [ ] Falls ein Modell angeboten wird: Ein freigegebenes Modell auswählen und eine neue Unterhaltung starten.
- [ ] Diese harmlose Frage stellen:

```text
Antworte bitte nur mit: GitLab Duo Test erfolgreich.
```

**Ergebnis:**  
- [ ] Model-Liste vorhanden  
- [ ] Model-Liste leer / nicht auswählbar  
- [ ] Modell gewählt und Antwort erhalten  
- [ ] Meldung `Language model unavailable`  
- [ ] Andere Fehlermeldung: __________________________

**Bewertung:** Eine leere Modellliste oder `Language model unavailable` kann auf fehlende Agentic-Freigabe, fehlende Credits/Modellberechtigung oder eine Verbindungsstörung hindeuten. **Noch keinen MCP-Server konfigurieren:** Agentic Chat muss erst ohne zusätzliche Tools eine einfache Textfrage beantworten.

## 3. GitLab Default Namespace prüfen

- [ ] Im Browser die firmeninterne GitLab-Instanz öffnen.
- [ ] Profilbild > **Preferences** öffnen.
- [ ] Bereich **AI assistance** > **GitLab Duo** > **Default group** suchen. Bei älteren Oberflächen steht die Einstellung ggf. direkt auf der Preferences-Seite.
- [ ] Kontrollieren, ob eine **berechtigte Gruppe** gewählt ist. Nur bei Bedarf ändern und mit **Save changes** speichern.
- [ ] In VS Code `Strg + Shift + P` > **`Developer: Reload Window`** ausführen.
- [ ] Danach **`GitLab: Diagnostics`** ausführen und **Agentic Chat** erneut prüfen.

**Gruppe (nur Gruppenname, keine Tokens):** __________________________  
**Agentic Chat danach:** `On` / `Off`  
**Hinweis:** Bei einem GitLab-Remote wird zunächst dessen Projekt-Namespace verwendet. Der Default Namespace ersetzt keinen fehlenden Agentic-Zugriff der Projektgruppe.

## 4. Projektzuordnung eingrenzen

In einem **PowerShell-Terminal** von VS Code testen. Der folgende Pfad ist ein Beispiel für ein GitLab-Repository aus deinen bisherigen Logs:

```powershell
Set-Location 'C:\Develop\repos\web-marktbearbeitung-truuco-api'
git rev-parse --show-toplevel
git remote get-url origin
```

- [ ] `origin` zeigt auf `gitlab.apps.union-investment.de`.
- [ ] **`GitLab: Diagnostics`** erneut starten, während nur dieses Repository geöffnet ist.
- [ ] Anschließend Agentic Chat erneut testen.

**Ergebnis für GitLab-Repository:** __________________________

Optional mit einem **Azure-DevOps-Repository** vergleichen, z. B. `C:\Develop\ui-mb-infrastructure`:

```powershell
Set-Location 'C:\Develop\ui-mb-infrastructure'
git remote get-url origin
```

**Ergebnis für Azure-DevOps-Repository:** __________________________

**Bewertung:**

- **Nur GitLab-Projekt funktioniert:** Default Namespace und Zuordnung für Nicht-GitLab-Projekte prüfen.
- **Kein Projekt funktioniert:** Agentic-Zugriff auf Instanz-/Gruppenebene oder Verbindung prüfen lassen.
- **Nur einzelne GitLab-Projekte funktionieren:** Projektgruppe und Feature-Berechtigung vergleichen.

## 5. Verbindung prüfen (nur Diagnose, keine Konfiguration ändern)

PowerShell:

```powershell
Test-NetConnection gitlab.apps.union-investment.de -Port 443
```

- [ ] `TcpTestSucceeded : True`
- [ ] `TcpTestSucceeded : False`

**Achtung:** `True` bestätigt nur eine TCP-Verbindung zu Port 443. Es bestätigt **nicht**, dass WebSockets oder der KI-Modellzugriff funktionieren. Wenn **Pick Model** nicht funktioniert, laut GitLab-Dokumentation den IDE-Verbindungstyp **WebSocket** und ggf. Freigabe von WebSocket-Verbindungen durch das Netzwerkteam prüfen.

## 6. Diagnostics und Logs direkt nach dem Fehler sichern

- [ ] VS Code: `Strg + Shift + P` > **`GitLab: Diagnostics`**.
- [ ] Den Abschnitt **GitLab Duo Agentic Chat** kopieren.
- [ ] **View > Output** öffnen.
- [ ] Im Dropdown nacheinander **GitLab** und **GitLab Language Server** auswählen.
- [ ] Zeitstempel der fehlgeschlagenen Chat-Anfrage notieren. Nach `error`, `403`, `401`, `WebSocket`, `model`, `namespace` oder `AgenticChatSupportCheck` suchen.

**Zeitstempel:** __________________________  
**Exakte Fehlermeldung:** __________________________  
**Agentic-Status:** __________________________

**Nicht weitergeben:** Personal Access Tokens, Session-Cookies, Authorization-Header oder ganze Debug-Logs ohne Prüfung.

## 7. Wenn weiterhin `Agentic Chat: Off` angezeigt wird

An das GitLab-Administrations-Team geben:

> Meine GitLab-Duo-Pro-Lizenz wird erkannt. Die Diagnostics zeigen Non-Agentic Chat und Code Suggestions als `On`, aber Agentic Chat als `Off` mit `Agentic Chat is supported for the current project (false)`. Ich habe den klassischen Chat, den Default Duo Namespace, ein einzelnes GitLab-Repository, die Modellliste sowie die Verbindung geprüft. Könnt ihr bitte die Freigabe der GitLab Duo Agent Platform auf unserer Self-Managed-Instanz, die Gruppen-/Projektberechtigung, verfügbare GitLab Credits und den Modellzugriff prüfen? Bei einer leeren Modellliste bitte zusätzlich den WebSocket-Pfad prüfen.

## 8. Abschließendes Testprotokoll

| Test | Status (`OK`, `Fehler`, `Offen`) | Beobachtung |
|---|---|---|
| Klassischer Quick Chat | | |
| Agentic Chat | | |
| Modell auswählen | | |
| Default Namespace | | |
| Einzelnes GitLab-Repo | | |
| Vergleich mit Azure-DevOps-Repo | | |
| TCP 443 | | |
| GitLab: Diagnostics erneut | | |

## Offizielle GitLab-Dokumentation

- [GitLab Duo Non-Agentic Chat](https://docs.gitlab.com/user/gitlab_duo_chat/)
- [GitLab Duo Agentic Chat (inklusive neuem VS-Code-Layout)](https://docs.gitlab.com/user/gitlab_duo_chat/agentic_chat/)
- [Default GitLab Duo Namespace in Preferences](https://docs.gitlab.com/user/profile/preferences/)
- [GitLab for VS Code: Troubleshooting](https://docs.gitlab.com/editor_extensions/visual_studio_code/troubleshooting/)
- [Agent Platform für Self-Managed aktivieren](https://docs.gitlab.com/user/duo_agent_platform/turn_on_off/)
- [Agent Platform: Model Selection und WebSocket](https://docs.gitlab.com/user/duo_agent_platform/model_selection/)

*Stand: 27.10.2026. Grundlage sind deine Diagnostics/Logs vom 22.10.2026 und die GitLab-Dokumentation. Die Prüfpunkte sind noch nicht ausgeführt.*
