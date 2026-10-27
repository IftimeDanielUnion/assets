# Warum eine Volksbank das MBPortal von Union Investment nutzt

## Vom Fondsprodukt zum tatsächlichen Kundengespräch

**Gesprächsvorbereitung für den 29. Oktober 2026.** Grundlage sind die gelieferten Repositories und die unten angegebenen offiziellen Union-Investment-Seiten. Die Alltagsszenen sind ausdrücklich illustrative Beispiele, keine berichteten Kundenfälle oder gemessenen Erfolgsquoten.

## 1. „Wir sind doch Union Investment. Warum arbeiten dann Volksbanken in unserer Anwendung?“

Gerade darin liegt der geschäftliche Zusammenhang: Union Investment bietet Anlagelösungen und Fondsmanagement an. Die genossenschaftlichen Partnerbanken sind wichtige Ansprechpartner für die Menschen, die diese Lösungen nutzen. Union Investment verweist für persönliche Beratung selbst auf Volksbanken und Raiffeisenbanken sowie weitere Partnerbanken. [W01–W03]

Für diesen Zusammenhang kann man die Rollen vereinfacht so beschreiben:

| Beteiligter | Rolle |
|---|---|
| Privatkunde | Hat beispielsweise ein Sparziel oder möchte sich zur Geldanlage informieren. |
| Partnerbank | Betreut den Kunden und unterstützt ihn bei der Auswahl einer passenden Lösung. |
| Union Investment | Stellt Fonds- und Anlagelösungen bereit; die hier betrachtete Anwendungslandschaft unterstützt zusätzlich die Marktbearbeitung. |
| Angebundene Dienstleister | Übernehmen innerhalb definierter Maßnahmen bestimmte Ausführungsaufgaben, etwa Telefonie, Mailing oder digitale Kampagnen. |

Die ersten drei Rollen beschreiben das öffentlich dargestellte Partnerschaftsmodell. Die Dienstleisteranbindung und die Maßnahmenabwicklung sind in den gelieferten Repositories erkennbar. [W01–W03, Q01–Q04]

**Die Bankmitarbeiter nutzen also nicht versehentlich eine Anwendung einer anderen Firma. Sie nutzen ein Werkzeug für eine gemeinsame Vertriebs- und Kundenansprache.** Das MBPortal ist in diesem Ablauf eine Arbeitsoberfläche für Banknutzer, nicht das Onlinebanking der Privatkunden.

## 2. Eine Alltagsszene macht den Nutzen verständlich

Stell dir eine fiktive Volksbank vor. Eine Vertriebsverantwortliche möchte ausgewählte Kunden zu einem Beratungsgespräch einladen. Das Ziel ist klar. Die eigentliche Arbeit beginnt aber erst danach: Wer gehört zur Zielgruppe? Welche Kontaktvoraussetzungen liegen vor? Wer übernimmt die Ansprache? Wie kommen die Ergebnisse zurück? Und woran erkennt die Bank, dass ein Termin tatsächlich zustande gekommen ist?

Ohne eine unterstützende Lösung müsste sie diese Schritte einzeln organisieren. Mit einer standardisierten Leistung möchte sie dagegen sinngemäß beauftragen:

> „Für diese ausgewählten Kunden möchten wir die Terminvereinbarung durchführen lassen. Bitte verarbeitet die Maßnahme und liefert uns die Ergebnisse zurück.“

Genau zu diesem Ablauf passen die Funktionen im gelieferten MBPortal: Leistungspaket auswählen, Zielgruppe vorbereiten, Kundendatei hochladen, Auftrag prüfen und buchen sowie Status und Ergebnisse nachverfolgen. Das ist die fachliche Einordnung des Codes, kein Versprechen, dass jede Kampagne erfolgreich endet. [Q01–Q03]

**Der eigentliche Nutzen ist nicht der Upload. Der Nutzen ist die organisierte Durchführung einer Maßnahme. Der Upload liefert dafür die ausgewählte Zielgruppe.**

## 3. Was „Kunden hochladen“ bei euch konkret bedeutet

Die Formulierung kann missverständlich sein. Es werden keine Personen verschoben und durch den Upload wird auch kein Geld angelegt. Im untersuchten Buchungsablauf wird eine **CSV-Datei mit Kundendaten für ein ausgewähltes Leistungspaket** übergeben. [Q02–Q03]

Das steht nicht nur in einer Architekturbeschreibung. Eure Benutzeroberfläche führt den Banknutzer ausdrücklich durch folgende Arbeitsschritte:

1. Ein bereitgestelltes Template für die Zielgruppenselektion kopieren.
2. Im Analysen-Startportal der Atruvia `agree21IDA` öffnen und das Template in einen Bericht übernehmen.
3. Die Datenfelder und Filter prüfen und dabei die Mindestanforderungen des gewünschten Moduls beachten.
4. Den Bericht als CSV exportieren.
5. Die Datei im MBPortal hochladen. [Q02–Q03]

Für **Terminvereinbarung** erläutert die Oberfläche zusätzlich, dass die Muster-Templates Kunden auswählen sollen, für die eine Einwilligung zur telefonischen Ansprache in `agree21BAP` erfasst ist. [Q02]

Daraus wird der konkrete Zusammenhang deutlich: **Die Bank stellt die Zielgruppe für ihren Auftrag zusammen. Das MBPortal übernimmt die Daten in die Maßnahmenverarbeitung.** Es handelt sich in diesem beschriebenen Weg nicht um einen automatisch vollständigen Abgleich aller Bankkunden.

Wichtig für das Gespräch: Die technische Annahme einer CSV beweist nicht, dass jeder darin enthaltene Datensatz für jeden Kontaktkanal verwendet werden darf. Welche Daten und Verarbeitungen für einen konkreten Auftrag zulässig sind, muss der freigegebene Fach- und Datenschutzprozess festlegen. Diese Freigabe lässt sich aus einer Uploadfunktion allein nicht ableiten. Der sichtbare Hinweis zur Telefonansprache ist eine konkrete Prozessvoraussetzung, keine pauschale Freigabe für sämtliche Maßnahmen.

## 4. Die sechs Leistungspakete, in Alltagssprache erklärt

Im gelieferten Backend sind sechs Pakete definiert. Das ist der belegte Paketumfang dieses Quellstands, nicht das gesamte Produktangebot von Union Investment und auch kein Nachweis, dass heute jede Bank alle Pakete freigeschaltet hat. [Q01]

### Terminvereinbarung: Aus einer Liste sollen Gespräche werden

Eine ausgewählte Kundengruppe ist noch kein voller Beratungskalender. Das Leistungspaket unterstützt den Schritt von der Auswahl zur telefonischen Ansprache und Terminrückmeldung.

Der Code unterscheidet dabei unter anderem: Termin gebucht, kein Termin gebucht, Kontakt nicht möglich und falsche Rufnummer. Die technische Betrachtung zeigt damit mehr als einen bloßen Versandauftrag: Die Bank soll erfahren können, wie die Kontaktversuche ausgegangen sind. [Q01]

**Der beabsichtigte Nutzen:** Die Bank kann die Vorbereitung von Beratungsgesprächen unterstützen lassen, statt nur eine Liste abzuarbeiten. Ein vereinbarter Termin ist allerdings noch kein Produktabschluss.

### Dialogeinwilligung: Erst einen dokumentierten Kontaktweg schaffen

In diesem Paket geht es um eine Einwilligungsmaßnahme, nicht unmittelbar um einen Fondsverkauf. Die UMP-Dokumentation beschreibt einen konkreten Ablauf: Adressen werden einem Auftrag zugeordnet, individuelle Landingpages entstehen, Empfänger erhalten einen Brief mit einem Link, bestätigen dort ihre Daten und durchlaufen anschließend einen E-Mail-Bestätigungsprozess. Die bestätigten Einwilligungen können vom MBPortal per REST abgerufen werden. [Q04]

Das Landingpagetool, kurz LPT, gehört dabei zur beschriebenen UMP-Kette. Es speichert die Eingaben und meldet sie an die UMP zurück. Dort wird der Double-Opt-in-Ablauf angestoßen. [Q04]

**Der beabsichtigte Nutzen:** Statt nur einen Datensatz mit einer Adresse zu haben, kann die Bank den Bearbeitungsstand der Einwilligungsmaßnahme und die Rückmeldungen nachvollziehen. Der dokumentierte technische Ablauf ersetzt dabei nicht die Prüfung der verwendeten Einwilligungstexte und ihres Geltungsbereichs.

### Leadveredelung Sparzielrechner: Aus allgemeinem Interesse einen konkreteren Anlass machen

„Leadveredelung“ klingt abstrakt. Anschaulich geht es darum, aus einem allgemeinen Kontakt ein besser erkennbares Interesse zu machen.

Eine illustrative Kundensituation: Jemand möchte sparen, hat aber noch keine klare Vorstellung von Sparziel oder Sparmöglichkeit. Ein Rechner kann einen Einstieg bieten. Im Paketmodell sind Ereignisse wie erster Rechnerbesuch, ergänztes Sparpotenzial, ergänzte Sparziele sowie Klicks in Richtung Termin oder MeinInvest vorgesehen. [Q01]

**Der beabsichtigte Nutzen:** Eine weitere Ansprache kann an einem konkreteren Interesse anknüpfen. Ein Klick oder Rechnerbesuch bedeutet trotzdem weder eine abgeschlossene Beratung noch einen automatischen Vertragsabschluss.

### Leadveredelung Elternsparrechner: Der Anlass ist das Sparen für Kinder

Dieses Paket greift einen anderen Anlass auf: das Sparen für Kinder. Die technische Grundidee ähnelt dem Sparzielrechner, aber der fachliche Zugang ist eigenständig.

Ein mögliches Gespräch beginnt nicht mit „Hier ist ein Finanzprodukt“, sondern mit „Was möchtest du deinem Kind später ermöglichen?“ Das ist ein Beispiel für die Ansprache, kein aus dem Repository übernommener Werbetext.

Die beiden Rechnerpakete sind im Code getrennte Leistungspakete, nutzen aber dieselbe Administrationsform für Leadveredelung. **Fachlich verschiedene Einstiege können also technische Bausteine teilen.** [Q01]

### UnionDigitalSales UDK: Eine produktspezifische digitale Maßnahme

Im Quellstand heißt dieses Paket `UnionDigitalSales UDK`; die bisherige Repositoryanalyse ordnet UDK UnionDepot Komfort zu. Der Schwerpunkt liegt auf einer digitalen Vertriebsmaßnahme mit eigener Konfiguration und nachgelagerter Verarbeitung. Das Paketmodell enthält beispielsweise die Zustände E-Mail versendet und externer Import erfolgreich. [Q01, H01]

**Der beabsichtigte Nutzen:** Die Bank muss die technische Abwicklung einer passenden digitalen Maßnahme nicht vollständig selbst aufbauen. Wichtig ist die Grenze: Ein erfolgreicher Import oder eine versendete E-Mail belegt noch keine Depoteröffnung.

### UnionDigitalSales JuniorDepot: Ein anderer Produktbezug, eine ähnliche Ablaufidee

Dieses Paket ist auf JuniorDepot ausgerichtet. Es nutzt im betrachteten Modell die gleiche grundsätzliche DigitalSales-Familie, bleibt aber ein eigenes Paket mit eigener Administrationszuordnung. [Q01]

**Der beabsichtigte Nutzen:** Ein Angebot kann gezielt im passenden Kontext ausgespielt werden. In einer beispielhaften Kampagne wäre das die finanzielle Zukunft eines Kindes. Ob die Kampagne tatsächlich zu einem Abschluss führt und welcher Abschlussweg gilt, muss aus dem konkreten Fachprozess hervorgehen.

## 5. Warum Union Investment das bereitstellt

Die wirtschaftliche Logik lässt sich aus dem Partnerschaftsmodell und den belegten Funktionen ableiten: Wenn die Bank passende Kunden besser erreicht und Maßnahmen leichter durchführen kann, unterstützt das zugleich das Geschäft mit den angebotenen Anlagelösungen. [W01–W03, Q01–Q04]

Für die Bank sind mögliche Vorteile weniger Organisationsaufwand, nachvollziehbare Rückmeldungen und zusätzliche Anlässe für die Kundenbetreuung. Für Union Investment entsteht eine skalierbare Unterstützung der Partnerbanken. Für einen Kunden kann daraus ein hilfreicher Kontakt zu einem passenden Thema entstehen.

**Das sind beabsichtigte Vorteile, keine in den Repositories gemessenen Wirkungen.** Konkrete Zeitersparnisse, Abschlussquoten, Paketpreise oder Vergütungsvereinbarungen sind damit nicht belegt.

Die Arbeitsteilung lässt sich so merken:

> Die Bank kennt den konkreten Kundenkontext. Union Investment stellt Anlagelösungen und unterstützende Angebote bereit. Die Marktbearbeitungsanwendungen organisieren einen Teil des Wegs von der Zielgruppe zur tatsächlichen Ansprache und Rückmeldung.

## 6. Jetzt ergeben auch die Anwendungsnamen einen Sinn

Die folgende Einordnung fasst die bisherige Repositoryanalyse zusammen. Sie beschreibt die fachlichen Aufgaben, nicht eine einzige lineare Kette, durch die jeder Auftrag zwingend laufen muss. [H01]

| Anwendung oder Bereich | Einfach erklärt |
|---|---|
| **MBPortal** | Hier werden konkrete Leistungen ausgewählt, Zielgruppen übergeben, Aufträge gebucht und Ergebnisse verfolgt. |
| **MBPortal-Administration** | Hier werden bankbezogene Voraussetzungen, Ansprechpartner, Einstellungen und Paketkonfigurationen gepflegt und veröffentlicht. |
| **FINCO-Integration** | Richtet bankbezogene Rechnerumgebungen und Flows ein beziehungsweise aktualisiert sie. Nicht mit dem Kundenimport in eine Kampagne gleichsetzen. |
| **PBM-Integration** | Verarbeitet Kampagnen- und Kundenimporte sowie deren Status für die passenden Maßnahmen. |
| **UMP-Integration** | Der MBPortal-seitige Adapter für die bestehende UMP-Anbindung, beispielsweise Aufträge, Vorschauen und Einwilligungsdokumente. |
| **UMP / UnionMarketingPlattform** | Eine eigene Marketinglandschaft. Die Kernanwendung `unioninvestment` stellt unter anderem die dokumentierte Permission-Orders-API bereit; das Landingpagetool ergänzt den Einwilligungsablauf. |
| **Investmentwelt** | Eine weitere Oberfläche für Marktbearbeitung und Beratungsunterstützung. Ihr BFF bündelt Daten; DMS und ProductHub liefern Dokumenten- beziehungsweise Materialfunktionen. |
| **IM ActionHub** | Verwaltet Templates und Aufträge für Impulse. Der vorhandene MBPortal-Pfad und der zusätzliche UMP-Zielvertrag sind getrennte Integrationen. |
| **Verbundintegration / VBST** | Überträgt und verarbeitet Vertriebsanlässe und Zielgruppen zwischen den angebundenen Systemen, insbesondere SOLID, vDWH und Truuco. Nicht als Synonym für das MBPortal verwenden. |
| **UBA-Anbindung** | Separater Rücklieferungs- und Dateibereitstellungspfad. |
| **MBPortal-Datenprodukt und Dashboard** | Machen Vorgänge und Kennzahlen für die Auswertung zugänglich. Das Dashboard ist nicht die Oberfläche zum Buchen von Maßnahmen. |

Eine wichtige Grenze für dein Gespräch: **Die vorhandene UMP-REST-Anbindung des MBPortals ist nicht dasselbe wie die zusätzlich geplante UMP-Action-Hub-Anbindung mit vier Service-Bus-Queues.** Ein dokumentierter Zielvertrag ist noch kein Nachweis eines vollständigen produktiven Ablaufs. [H01]

## 7. Was deine technische Verantwortung damit zu tun hat

Aus dieser Sicht sind eure Azure-Komponenten nicht bloß eine Sammlung von Ressourcen. Hinter ihnen stehen konkrete Arbeitsabläufe von Banken und deren Kunden.

Ein nicht verarbeiteter Auftrag kann eine Maßnahme verzögern. Eine nicht zugeordnete Rückmeldung kann einen erfolgreichen Kontakt im Portal unsichtbar machen. Eine falsche bankbezogene Konfiguration kann die falschen Angaben in einen Ablauf bringen. Das sind mögliche Auswirkungen, keine Behauptungen über bereits eingetretene Vorfälle.

Für deine Arbeit heißt das: Bei Deployments und Migrationen nicht nur prüfen, ob eine Function startet. Entscheidend ist, ob der fachliche Ablauf weiter funktioniert: Daten übernehmen, Auftrag zuordnen, richtigen Dienst ansprechen, Ergebnis zurückführen und bei Fehlern kontrolliert wiederanlaufen.

**Das ist auch der Sinn einer vorsichtigen GitLab-Migration: Der Ausführungsweg der Pipelines verändert sich, ohne unbeabsichtigt diese fachlichen Abläufe zu verändern.**

## 8. Sprechtext für deinen Termin

> „Unsere Marktbearbeitungslandschaft unterstützt die Partnerbanken dabei, aus einer ausgewählten Kundengruppe konkrete Maßnahmen zu machen. Union Investment stellt Anlagelösungen bereit, die Banken betreuen die Kunden. Im MBPortal kann eine Bank zum Beispiel eine Terminvereinbarung oder Einwilligungsaktion beauftragen. Dafür wählt sie die passende Zielgruppe in ihren Systemen aus und lädt die benötigten Daten für den Auftrag hoch. Das Portal koordiniert die Verarbeitung mit den angebundenen Diensten und führt Status und Ergebnisse zurück. UMP, Investmentwelt, Action Hub und Verbundintegration haben dabei unterschiedliche Aufgaben. Meine technische Verantwortung ist, dass diese Anwendungen, Schnittstellen und Betriebsabläufe zuverlässig und kontrolliert funktionieren, auch während der Migration nach GitLab.“

## Quellen und Grenzen

### Offizielle Einordnung des Geschäftsmodells, abgerufen am 27.10.2026

- **W01:** Union Investment, „Unser Unternehmen“: https://unternehmen.union-investment.de/ueber-uns/unser-unternehmen
- **W02:** Union Investment, „Union Investment für Privatkunden“: https://www.union-investment.de/ueber-uns/union_investment_fuer_privatkunden
- **W03:** Union Investment, „Filialsuche“, persönliche Beratung bei Partnerbanken: https://www.union-investment.de/unsere-services/kontakte-im-ueberblick/filialsuche

Die öffentliche Unternehmensdarstellung ergänzt die internen Quellen nur um die Frage, warum Union Investment und Partnerbanken zusammenarbeiten. Sie bestätigt nicht den aktuellen Betriebsstand einzelner interner Anwendungen.

### Für diese Erklärung erneut gelesene Originaldateien

- **Q01:** `marktbearbeitung.zip` → `marktbearbeitung/mbportal-backend/service/src/main/java/com/ui/b2b/mbportal/backend/core/domain/servicepackage/ServicePackage.java`
- **Q02:** `marktbearbeitung.zip` → `marktbearbeitung/mbportal-frontend/src/app/modules/booking/components/booking-stepper/step2-upload-step/upload-step.component.html`
- **Q03:** `marktbearbeitung.zip` → `marktbearbeitung/mbportal-frontend/src/app/modules/booking/components/booking-stepper/step2-upload-step/upload-step.component.ts`
- **Q04:** `UMP_repos(1).zip` → `UMP_repos/UMP_legacy/unioninvestment/docs/content/mb-portal-api/introduction.md`

### Ergänzende Zusammenführung der bereits analysierten Repositories

- **H01:** `Marktbearbeitung_Technisches_Gesamthandbuch.pdf`, Arbeitsstand 11.10.2026, insbesondere Seiten 4–15 und 23–24. Eine aus den gelieferten Archiven erstellte Analyse, keine zusätzliche unabhängige Quelle und kein Live-Inventar.

Es wurden keine Bankkundendaten, produktiven Systeme oder Vertragsunterlagen eingesehen. Beispiele illustrieren den Nutzen. Aussagen über tatsächliche Nutzung, messbare Wirkung, rechtliche Freigaben oder heutige Produktivaktivierung werden daraus nicht abgeleitet.
