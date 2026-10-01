# Codebefunde, Quellen und Prüfstand

**Stand: 01.10.2026.** Die Aussagen zum Projekt beziehen sich auf den hochgeladenen Snapshot. Es wurde kein Azure-Tenant, laufender Server oder aktueller entfernter Branch abgefragt. Die öffentliche Microsoft-/Terragrunt-Dokumentation wurde für die technischen Prüfungen separat nachgeschlagen.

## Q1: Das neue Ticket

Die beiden Ticketfotos zeigen DBI-12139 mit Prio 1 und der Anforderung, große digitale Medien in LP-Asset-Blobstorages für dev, test und prod bereitzustellen. Das Akzeptanzkriterium fordert angelegten Storage und öffentlich lesbare Assets. Sie enthalten keine vollständige Beschreibung des Upload-Prozesses, keine neuen Kampagnennamen und keinen benannten Netzwerkverantwortlichen. [Abschrift der sichtbaren Angaben](quellen/code-auszuege.md#q1-ticket)

## Q2: Die Speicher sind schon im Code

In allen drei Umgebungen gibt es `unit "storage-assets"`, `component_name = "lpassets"`, erlaubten anonymen Blobzugriff auf Account-Ebene und aktivierte Versionierung. Der Unit-Baustein nutzt DEV mit LRS, TEST und PROD mit ZRS. Das ist eine Beschreibung der vorhandenen Konfiguration, keine neue Auswahl für dieses Ticket.

| Umgebung | Im Gateway referenzierter Storage Account | Konfigurierte Asset-Domain |
|---|---|---|
| DEV | `stumpdevlpassetsyws91x59` | `landing-assets.dev.apps.union-investment.de` |
| TEST | `stumptestlpassetsjjh3lzn` | `landing-assets.test.apps.union-investment.de` |
| PROD | `stumpprodlpassetsxvhfz3h` | `landing-assets.apps.union-investment.de` |

Die Accounts werden im Modul über das Namensmodul benannt. Die hier genannten konkreten Namen wurden aus den bereits eingetragenen Gateway-Backend-Zielen übernommen. Ob die tatsächlichen Accounts so existieren, muss die Azure-Abfrage bestätigen. [Auszüge](quellen/code-auszuege.md#q2-vorhandene-speicherdefinitionen)

## Q3: Der Storage-Netzzugang bleibt privat

`catalog/modules/blob-storage/main.tf` setzt `public_network_access_enabled = false`, eine standardmäßig sperrende Netzwerkregel und einen Private Endpoint. Das ist bewusst eine andere Einstellung als `allow_nested_items_to_be_public` für anonymes Lesen.

Daraus folgt für dieses Paket: den vorhandenen privaten Backend-Aufbau verwenden und den Kundenweg über die freigegebene Asset-Domain prüfen. Ein direkt im Internet erreichbarer Blob-Service wird nicht als neue Anforderung erfunden. [Auszüge](quellen/code-auszuege.md#q3-privater-netzwerkzugang), W1 und W2.

## Q4: Die richtigen Container und Rechte

Die Container werden über `azapi_resource` angelegt. Das Modul schreibt `publicAccess = "Blob"` nur bei gesetztem `anonymous_access_enabled`; sonst `"None"`. Der Standard für den einzelnen Container ist `false`.

In den Stacks ist `assets` deshalb nicht anonym lesbar. `statuspages`, `uniglobal-release-1`, `ve-release-1` und `zukunft-release-1` sind entsprechend aktiviert. `Blob` ist nicht dieselbe Berechtigung wie die öffentliche Auflistung des gesamten Containers.

Die sichtbaren zusätzlichen Gruppenrollen an den Release-Containern lauten `Storage Blob Data Reader`. Daraus darf kein Schreibrecht abgeleitet werden. Weitere geerbte oder außerhalb dieses Ausschnitts vergebene Rechte sind nicht ausgeschlossen. [Auszüge](quellen/code-auszuege.md#q4-containerrechte-und-katalogeinbindung), W1 und W5.

## Q5: Gateway und Lesetests

Die Asset-Listener sind in DEV, TEST und PROD als `private` definiert. Die drei öffentlichen Pfadnamen werden auf ihre Release-Container abgebildet. Ein zusätzlicher öffentlich vorgeschalteter Dienst ist in den geprüften Asset-Definitionen nicht nachgewiesen.

Die zugehörige WAF-Regel lässt nur GET durch und sperrt Querystrings. Deshalb prüfen die beigefügten HTTP-Tests normale GETs ohne Queryparameter und keine HEAD-Anfragen. Die CORS-Rewrite-Regel des Moduls übernimmt den Origin-Header der Anfrage. Ein vollständiger Browser- oder OPTIONS-Ablauf wird daraus nicht automatisch abgeleitet.

Für die Asset-Backend-Pfade steht `request_timeout = 1`. Ein tatsächliches Timeoutproblem muss gemessen werden; die Zahl beweist weder eine maximale Dateigröße noch für sich allein eine fehlerhafte Übertragung. [Auszüge](quellen/code-auszuege.md#q5-gateway-pfade-und-waf), W7.

## Q6: Der alte Container wurde ausdrücklich stillgelegt

Diese Änderungen sind in der mitgelieferten Git-Historie von `live` enthalten:

| Commit | Datum | Inhalt |
|---|---|---|
| `15f0135` | 10.09.2026 | Verwendung eigener Container je Landingpage-Release in DEV. |
| `e84d685` | 11.09.2026 | Anonymen Zugriff auf den veralteten DEV-Container `assets` ausschalten. |
| `46ec1e1` | 11.09.2026 | Zugriff beziehungsweise bisherige Rollenzuordnung auf dem veralteten `assets`-Container in DEV und TEST entfernen. |

Das ist der konkrete Grund, keinen Patch zur erneuten Öffnung dieses Containers vorzubereiten. In PROD steht der Container im aktuellen Snapshot ebenfalls ohne Freigabeflag. [Git-Auszüge](quellen/code-auszuege.md#q6-git-historie)

## Q7: Die Anwendung hat bereits einen Asset-Adressparameter

`ui-landingpagetool/config/app.php` liest `ASSETS_BASE_URL`. `LandingPageDataListener` gibt den Wert als `assetsBaseUrl` an den Landingpage-Kontext weiter. Für den vorhandenen URL-basierten Ansatz ist damit ein konkreter Konfigurationspunkt vorhanden. Ein funktionierender Zugriff aus sämtlichen Templates ist damit noch nicht bewiesen.

In DEV nennt `ansible-ump/inventories/dev.yml` eine andere Asset-Domain als das aktuelle `live/dev/appgateway.hcl`. TEST und PROD stimmen in den geprüften Hostangaben überein. Die Ansible-Variable wird auch für einen älteren Webserver-VHost verwendet; sie ist kein Beweis für den aktuellen Wert von `ASSETS_BASE_URL`. Deshalb den tatsächlich deployten Wert prüfen, statt einen Domainwechsel blind auszurollen.

`ASSET_URL` und `ASSETS_BASE_URL` sind unterschiedliche Konfigurationseinträge. Nicht versehentlich den falschen ändern. [Auszüge](quellen/code-auszuege.md#q7-anwendung-und-dev-domainabweichung)

## Q8: Bestehender Bereitstellungsweg

Der Snapshot von `live` hat HEAD `dd42ccc` vom 25.09.2026. Die Stacks referenzieren `catalog` `v0.13.0`. Die relevanten fünf Katalogdateien wurden mit diesem Tag verglichen. Sie sind nach Normalisierung der Zeilenenden identisch. Für den bestehenden Storage-/Gateway-Vertrag ist deshalb kein neuer Katalog-Release notwendig.

Die Repo-Dateien nennen Terraform `1.13.4` und Terragrunt `1.1.6`. Das Backend ist ein bestehendes GitLab-HTTP-Backend mit getrennten States je Unit. Der Plan-Abschnitt im Paket verwendet den vorhandenen `storage-assets`-Baustein und erzeugt keinen neuen Infrastrukturentwurf.

Die CI enthält manuelle Apply-Jobs, kann aber den gesamten betroffenen Umgebungsstack verarbeiten. Ein manueller Start allein begrenzt den Änderungsumfang nicht auf dieses Ticket. Den tatsächlichen Plan vollständig prüfen. [Auszüge](quellen/code-auszuege.md#q8-werkzeugstände-und-bereitstellung), [Dateivergleiche](quellen/snapshot-vergleich.json).

## Öffentliche Primärquellen

**W1: Microsoft, anonymer Lesezugriff auf Container und Blobs.** Belegt die Trennung zwischen Account-Freigabe und Containerzugriff sowie zwischen Bloblesen und Containerauflistung. Netzregeln gelten weiterhin.

```text
https://learn.microsoft.com/en-us/azure/storage/blobs/anonymous-read-access-configure
```

**W2: Microsoft, Storage-Netzwerkzugang und Private Endpoints.** Belegt, dass deaktivierter öffentlicher Netzwerkzugang und Private-Endpoint-Zugriff getrennt vom anwendbaren Datenzugriffsrecht betrachtet werden müssen.

```text
https://learn.microsoft.com/en-us/azure/storage/common/storage-network-security-set-default-access
https://learn.microsoft.com/en-us/azure/storage/common/storage-private-endpoints
```

**W3: Microsoft, Container über die Verwaltungsschnittstelle auflisten.** API-Aufbau, Container-Eigenschaft `publicAccess` und offizielle API-Spezifikation für den im Werkzeug verwendeten Stand `2025-01-01`.

```text
https://learn.microsoft.com/en-us/rest/api/storagerp/blob-containers/list
https://raw.githubusercontent.com/Azure/azure-rest-api-specs/main/specification/storage/resource-manager/Microsoft.Storage/stable/2025-01-01/blob.json
```

**W4: Microsoft, Range-Header für Blob-GETs.** Belegt das Teilabruf-Format und die erwartete 206-Antwort. Das Paket prüft eine konkrete Byte-Range, nicht eine allgemeine Streaming-SLA.

```text
https://learn.microsoft.com/en-us/rest/api/storageservices/specifying-the-range-header-for-blob-service-operations
```

**W5: Microsoft, Datenzugriff mit Azure CLI und Microsoft Entra.** Grundlage für `--auth-mode login` statt implizitem Schlüsselabruf und die getrennte Betrachtung von Zugriffsrechten.

```text
https://learn.microsoft.com/en-us/azure/storage/blobs/authorize-data-operations-cli
```

**W6: Microsoft, Azure-CLI-Befehle für Blobs.** Grundlage für Upload, `--overwrite false`, `--if-none-match`, Metadaten, ETag-Prüfung und bedingtes Löschen. Die tatsächlich erforderlichen Azure-Rechte und der Laufzeitweg sind noch offen.

```text
https://learn.microsoft.com/en-us/cli/azure/storage/blob?view=azure-cli-latest
```

**W7: Microsoft, Application-Gateway-Backend-Einstellungen.** Erläutert die Bedeutung des Request-Timeouts und Pfadüberschreibungen. Das Paket leitet daraus keine pauschale Änderung aller Timeouts ab.

```text
https://learn.microsoft.com/en-us/azure/application-gateway/configuration-http-settings
```

**W8: Terragrunt, Stack-Generierung und Run-Kommando.** Generierter `.terragrunt-stack`-Aufbau, Stackfilter und Übergabe von Terraform-Argumenten. Der im Paket gezeigte Plan-Befehl ist nicht gegen euren State ausgeführt worden.

```text
https://docs.terragrunt.com/reference/cli/commands/stack/generate/
https://docs.terragrunt.com/reference/cli/commands/run/
```

## Tatsächlich durchgeführte Prüfungen

| Prüfung | Ergebnis |
|---|---|
| 9 Storage-/Gateway-/WAF-Dateien aus `live` gegen dessen enthaltenen HEAD | Nach Normalisierung von CRLF zu LF identisch. |
| 5 relevante Katalogdateien gegen `v0.13.0` | Nach Normalisierung von CRLF zu LF identisch. |
| Zielnamen und Subscription-IDs in `umgebungen.json` | Aus den tatsächlich gelesenen Konfigurationen abgeleitet, nicht frei erfunden. |
| 29 lokale Tests des Python-Werkzeugs | Bestanden. Azure- und HTTP-Antworten sind in diesen Tests simuliert. |
| Python-Syntax und Kommandohilfe | Lokal geprüft. |
| Bash-Blöcke in den Markdown-Dateien | Mit `bash -n` syntaktisch geprüft, nicht als Azure-/Terraform-Befehle ausgeführt. |

Das ausführliche Testergebnis liegt in `quellen/lokale-tests.txt`; die ergänzenden Paketprüfungen in `quellen/paketpruefung.json`.

**Nicht durchgeführt:** Azure-Bestandsabfrage, DNS-/TLS-Prüfung gegen eure Domains, echter Upload, anonymer HTTP-Abruf, Medientest, Terraform-Validierung mit Providern, Plan gegen euren State, Apply, Änderung entfernter Repositories oder Versand einer Nachricht.

Das Paket enthält keine privaten Schlüssel, keine kopierten Zertifikatsdateien, keine kompletten Repository-Archive und keine Screenshots mit dem zusätzlich sichtbaren Mailinhalt.
