# DBI-12139: Dateien für Landingpages bereitstellen

**Eigenständiges Arbeitspaket. Stand: 01.10.2026.** Grundlage sind deine Ticketfotos und der hochgeladene Repository-Snapshot. Es wurde nichts versendet oder in Azure verändert.

## Nachricht an Zeljko

> Hi Zeljko, ja, ich kümmere mich darum und prüfe den vorhandenen Aufbau für dev, test und prod. Ich gebe dir danach ein Update.

## Die wichtigste Entdeckung

**Die geforderten Speicher sind im Code bereits für dev, test und prod definiert.** Deshalb nicht noch einmal drei neue Accounts anlegen. Zuerst feststellen, ob der vorhandene Aufbau tatsächlich ausgerollt ist und die Dateien ohne Anmeldung erreichbar sind. [Codebelege](04_Codebefunde_und_Quellen.md)

Der alte Container `assets` wurde laut Git-Historie bewusst gesperrt. Vorgesehen sind inzwischen eigene Container je Landingpage und Release. Ihn einfach wieder öffentlich zu schalten wäre keine begründete Lösung dieses Tickets.

## Womit du anfängst

1. [Aufgabe und Reihenfolge](01_Aufgabe_einfach.md) lesen und die reine Bestandsprüfung starten.
2. Anhand des Ergebnisses nur die tatsächlich fehlenden Teile bearbeiten. [Anleitung und Prüfwerkzeug](03_Anleitung_und_Tests.md)
3. Den Zugriff testen und [Abnahme dokumentieren](05_Abnahme.md). Weitere [fertige Nachrichten](02_Nachrichten.md) sind nach Anlass getrennt.

**Priorität:** Prio 1 laut Ticketfoto. Wegen der direkten neuen Zuweisung würde ich diese Bestandsprüfung vor dem Monitoring beginnen. Die bereits angestoßene SFTP-Rückmeldung kann parallel laufen. Das ist eine Arbeitsreihenfolge, keine technische Abhängigkeit.

**Standalone gegenüber den bisherigen vier Tickets:** Ja. DBI-11149, DBI-6455, PB2B-27849 und DBI-7161 müssen dafür nicht abgeschlossen sein. Dieses Ticket baut aber auf den vorhandenen Storage-, Gateway-, DNS- und Zugriffsbausteinen auf. Es setzt keine Rückmeldung von Timo, Marcel oder Christoph voraus.

## Was im Paket liegt

| Datei | Wofür du sie brauchst |
|---|---|
| `01_Aufgabe_einfach.md` | Verständliche Schritte und Entscheidungen nach dem jeweiligen Ergebnis. |
| `02_Nachrichten.md` | Kopierfertige Nachrichten an Zeljko, Alex und bei Bedarf den zuständigen Webzugangs-Kontakt. |
| `03_Anleitung_und_Tests.md` | Konkrete Befehle für Bestandsprüfung, Testdatei, anonymen Zugriff und Bereinigung. |
| `04_Codebefunde_und_Quellen.md` | Nachprüfbare Codefunde, Git-Historie, Quellen und tatsächlicher Prüfstand. |
| `05_Abnahme.md` | Checkliste für alle drei Umgebungen. |
| `code/lp_assets.py` | Eigenständiges Python-Werkzeug ohne zusätzliche Python-Pakete. |
| `code/umgebungen.json` | Aus dem Snapshot übernommene Zielnamen und vorhandene Pfadzuordnungen. |
| `tests/test_lp_assets.py` | Lokale Tests mit simulierten Azure- und HTTP-Antworten. |
| `quellen/` | Auszüge aus den Originaldateien und lokale Prüfergebnisse. |

**Bewusst kein pauschaler Terraform-Patch:** Der vorhandene Code enthält bereits die benötigten Ressourcen. Ohne Azure-Abgleich wäre ein Neuanlage- oder Öffnungs-Patch spekulativ. Das Paket enthält stattdessen einen konkreten Weg zur Prüfung, gezielten Nachlieferung und Abnahme.
