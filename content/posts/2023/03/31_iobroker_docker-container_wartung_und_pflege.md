---
title: 'ioBroker Docker Container: Wartung und Pflege'
subtitle: ''
description: 'Hier findest du Infos rund um Wartung und Pflege deines ioBroker Docker Containers'
summary: 'Auch der zuverlässigste ioBroker Docker Container braucht hin und wieder ein bisschen Aufmerksamkeit. Ob Backups oder Updates – die regelmäßige Pflege deines Systems ist ein absolutes Muss.'
date: 2023-03-31T08:07:00+01:00
lastmod: '2023-12-14T20:42:23+01:00'
draft: false
featuredImage: /img/posts/2023-03-31/header.png

tags: []
categories: [ioBroker]

author: 'André (buanet)'

seo:
  image: ''

comment:
  enable: true
---

## Einleitung

Auch der zuverlässigste ioBroker Docker Container braucht hin und wieder ein bisschen Aufmerksamkeit. Ob Backups oder Updates – die regelmäßige Pflege deines Systems ist ein absolutes Muss.

Mit Blick auf den heutigen "Welt-Backup-Tag" habe ich mich entschlossen, einen umfassenden und leicht verständlichen Beitrag zu diesem Thema zu verfassen. Im Mittelpunkt steht dabei, wie du deinen ioBroker am besten sicherst, Updates durchführst und welche praktischen Tools dir dabei unter die Arme greifen können.

Bereit, in die Tiefen der Systemwartung einzutauchen? Lass uns gemeinsam sicherstellen, dass dein ioBroker immer auf der Höhe der Zeit ist und du stets ein gutes Backup in der Hinterhand hast.

Viel Spaß beim Lesen!


## Backups: Die Basis für alles

Die Grundlage dafür, dass du überhaupt gefahrlos mit und an deinem System arbeiten, konfigurieren und aktualisieren kannst, bildet ein zuverlässiges und aktuelles Backup deiner ioBroker Docker Installation. Werfen wir einen Blick auf die Möglichkeiten die du hast und was es ggf. dabei zu beachten gilt.

### Backup auf Dateiebene

Beim Einrichten des ioBroker Docker Containers hast du die Konfigurationsdaten von ioBroker, die sich standardmäßig im Container unter `/opt/iobroker` befinden, in ein Verzeichnis oder Volume auf deinem Docker Host ausgelagert. Das Backup auf Dateiebene bezieht sich darauf, dieses ausgelagerte Verzeichnis zu sichern. Diese Sicherung erfolgt auf der Ebene des Docker Hosts und kann beispielsweise durch die Einrichtung eines Cronjobs in Kombination mit einem kleinen Skript automatisiert werden. Alternativ kannst du auch jedes Backup-Tool verwenden, das in der Lage ist, Daten unter Linux auf Dateiebene zu sichern. Diese Sicherungsstrategie gewährleistet, dass im Falle von Problemen oder Datenverlust eine zuverlässige Wiederherstellung auf Basis der gesicherten Dateien möglich ist.

{{< admonition type=warning title="Achtung" open=true >}}
Um eine Inkonsistenz der Daten beim Datei-Backup zu vermeiden solltest du sicherstellen, dass der ioBroker Docker Container während der Sicherung gestoppt ist!
{{< /admonition >}}

Ergänzend zu diesem Ansatz ergeben sich aus dem Backup auf Dateiebene sowohl Vorteile als auch Nachteile:

#### Vorteile

Durch das Backup auf Dateiebene ergeben sich folgende Vorteile:

* **1:1 Kopie deiner aktuellen ioBroker Installation:** Das Backup sichert exakt den Zustand deiner ioBroker-Umgebung, einschließlich aller Konfigurationen und Daten.

* **Minimale Wiederherstellungszeit:** Im Falle eines Problems ermöglicht das Backup auf Dateiebene eine schnelle Wiederherstellung. Hierzu müssen lediglich der ioBroker-Container gestoppt, die Dateien wiederhergestellt und der Container erneut gestartet werden.

* **Kann mit Bordmitteln des Docker Hosts realisiert werden:** Die Umsetzung dieser Backup-Strategie kann möglicherweise mithilfe der nativen Tools des Docker Hosts erfolgen, wie zum Beispiel bei einer Synology DiskStation durch [Hyper Backup](https://www.synology.com/de-de/dsm/feature/hyper_backup).

* **Einfache Prüfung und Wiederherstellung einzelner Dateien:** Da das Backup auf Dateiebene eine Kopie der einzelnen Dateien und Konfigurationen erstellt, ermöglicht es eine einfache Prüfung und selektive Wiederherstellung bestimmter Dateien, falls erforderlich.

#### Nachteile

Jedoch, wo Licht ist, ist auch Schatten. Hier sind mögliche Nachteile des Backups auf Dateiebene:

* **ioBroker Container sollte beim Backup gestoppt sein:** Um die Gefahr eines defekten Backups zu minimieren, sollte der ioBroker Container während des Backups gestoppt sein.

* **Backup kann sehr groß sein:** Da das gesamte ioBroker-Verzeichnis gesichert wird, kann das Backup beträchtlich groß werden. Dies sollte bei der Speicherplanung berücksichtigt werden.

* **Kann systemressourcenintensiv sein:** Das Erstellen von Backups auf Dateiebene kann systemressourcenintensiv sein, insbesondere wenn große Datenmengen regelmäßig gesichert werden müssen. Dies könnte die Leistung des Systems beeinträchtigen, während das Backup läuft.


### Backup der Konfiguration aus ioBroker heraus

Beim Backup der Konfiguration aus ioBroker heraus erfolgt eine gezielte Sicherung nur der essentiellen Dateien und Konfigurationsdaten. Im Gegensatz zum Backup auf Dateiebene, bei dem sämtliche Dateien der ioBroker-Installation gesichert werden und die Backup-Datei dementsprechend groß sein kann, beschränkt sich dieses Verfahren auf das Notwendigste. Im Falle einer Wiederherstellung lädt ioBroker die Adapter aus dem Internet herunter, installiert sie frisch und stellt dann die Konfiguration wieder her.

Zusätzlich bietet ioBroker mit der integrierten Backup-Lösung über den Backitup-Adapter die Möglichkeit, Backups nach einem festgelegten Zeitplan automatisch zu erstellen. Auf Wunsch kopiert das System die Backups sogar in die Cloud oder auf ein externes Backup-Ziel.

Hier ein Blick auf das ioBroker.backitup User Interface:
{{< image src="/img/posts/2023-03-31/01_backitup_interface.png" alt="ioBroker.backitup Interface" width="100%" >}}

Weiterführende Informationen: 

* [iobroker backup command (ioBroker Doku)](https://www.iobroker.net/docu/index-98.htm?page_id=3971&lang=de#iobroker_backup)
* [ioBroker Backitup-Adapter (GitHub)](https://github.com/simatec/ioBroker.backitup)

###### Vorteile

Diese Methode weist ebenfalls ihre eigenen Vorzüge auf:

* **Backups sind relativ klein:** Da nur die notwendigsten Dateien und Konfigurationsdaten gesichert werden, sind die resultierenden Backups vergleichsweise kompakt.

* **Automatisierung direkt aus ioBroker heraus möglich (Backitup-Adapter):** Durch die Nutzung des [ioBroker.backitup-Adapters](https://www.iobroker.net/docu/index-98.htm?page_id=3971&lang=de#iobroker_backup) kann die Erstellung der Backups direkt aus der ioBroker-Oberfläche automatisiert werden.

* **Erstellung des Backups während des Betriebs möglich:** Die Sicherung der Konfiguration kann während des laufenden Betriebs erfolgen, ohne dass der ioBroker-Container gestoppt werden muss.

###### Nachteile

Natürlich existieren auch potentielle Nachteile:

* **Eventuell sehr lange Wiederherstellungszeit durch Adapterinstallation aus dem Internet:** Die Wiederherstellung könnte unter Umständen lange dauern, insbesondere wenn eine Vielzahl von Adaptern aus dem Internet heruntergeladen und installiert werden muss. Die Geschwindigkeit ist dabei abhängig von der Anzahl der Adapter und der Systemleistung.

* **Bestimmte Voraussetzungen bei der Wiederherstellung gelten (z.B. kompatible js-controller-Version):** Es müssen bestimmte Voraussetzungen erfüllt sein, um eine reibungslose Wiederherstellung sicherzustellen, wie beispielsweise die Kompatibilität mit der verwendeten js-controller-Version.

* **Abhängigkeit von Online-Ressourcen:** Da die Adapter aus dem Internet heruntergeladen werden, besteht eine Abhängigkeit von der Verfügbarkeit dieser Ressourcen. Wenn die Adapter-Downloads aus irgendeinem Grund nicht verfügbar sind, könnte dies die Wiederherstellung beeinträchtigen.

* **Mögliche Inkompatibilitäten nach der Wiederherstellung:** Aufgrund von Aktualisierungen oder Änderungen an den Adaptern könnte es zu Inkompatibilitäten mit der wiederhergestellten Konfiguration kommen. Dem kann allerdings entgegengewirkt werden, indem du dein System stets aktuell hältst. 


### Meine persönliche Backup-Strategie

In meiner individuellen Backup-Strategie verfolge ich einen zweistufigen Ansatz, der sich als äußerst effektiv erwiesen hat. Auf der einen Seite sichere ich die Daten meines ioBroker Docker Containers auf Dateiebene des Docker Hosts mithilfe des Synology Tools ["Active Backup for Business"](https://www.synology.com/en-global/dsm/feature/active-backup-business/overview) (Absoluter Tip für Besitzer einer DiskStation!). Hierbei werden regelmäßig umfassende Dateibackups des gesamten ioBroker Docker Verzeichnisses erstellt. Der "Active Backup for Business Agent" auf meinem Docker Host ermöglicht eine nahtlose Zusammenarbeit mit der Backup-Software auf meiner Synology DiskStation. Im Bedarfsfall führt er das entsprechende Backup vollautomatisch durch. Ein herausragender Vorteil dieser Lösung liegt darin, dass ich über das Wiederherstellungsportal die Freiheit besitze, in die verschiedenen Backups einzusehen und bei Bedarf sogar nur einzelne Dateien gezielt wiederherzustellen.

Zusätzlich setze ich aber auch auf die integrierte Backup-Funktion von ioBroker durch den Backitup-Adapter, um eine zusätzliche Sicherungsebene zu schaffen. Die automatischen Backups innerhalb von ioBroker gewährleisten, dass meine Konfigurationen, Einstellungen und Daten regelmäßig gesichert werden. Somit verfüge ich jederzeit über eine Backup-Datei, die mir eine reibungslose Wiederherstellung meines Systems nach einer Neuinstallation ermöglichen würde. Hierbei verlasse ich mich darauf, dass die täglichen ioBroker-Backups in meinen Backups auf Dateiebene integriert sind und zusätzlich vom Backitup Adapter sicher in der Cloud abgelegt werden.

Durch diese Kombination aus Dateibackup auf Docker-Host-Ebene und den internen Sicherungsfunktionen von ioBroker bin ich zuversichtlich, dass meine ioBroker-Installation zu jeder Zeit optimal geschützt ist – unabhängig von den möglichen Widrigkeiten, die auftreten können.


## Updates: Wartung ist wichtig!

Hast du dein Backup auf dem neuesten Stand? Perfekt. Aber damit du es im Ernstfall gar nicht erst benötigst, ist es entscheidend, deinen ioBroker Docker Container regelmäßig zu warten und idealerweise immer aktuell zu halten. Hierbei gilt die einfache Faustregel: Kleine Schritte sind besser als große. Das bedeutet, wenn du dein System regelmäßig mit Updates versorgst, reduzierst du das Risiko von Komplikationen während des Updates oder möglicherweise notwendigen aufwändigen Zwischenschritten.
Das liegt vor Allem daran, dass sich Software ständig weiterentwickelt und Entwickler nicht immer garantieren können, dass ein Update von Version 1 direkt auf Version 3 genauso reibungslos funktioniert wie ein schrittweises Update von Version 1 auf Version 2 und dann auf Version 3.

Werfen wir einen Blick auf die verschiedenen Ebenen der Updates für deinen ioBroker Docker Container und wie du diese Updates am besten installieren kannst.

### Linux Paket Updates (optional)

Das ioBroker Docker Image umfasst eine Vielzahl von Linux-Paketen, die die Grundlage für die Funktion von ioBroker innerhalb deines Docker Containers bilden. Beispiele für diese Pakete, die dir möglicherweise aus der Admin-Oberfläche von ioBroker bekannt sind, sind unter anderem nodejs oder npm. Um Updates für diese Pakete zu installieren, verfügen die meisten Linux-Systeme, einschließlich des ioBroker Docker Containers, über einen Paketmanager – in unserem Fall ist dies apt.

Mit Hilfe dieses Paketmanagers kannst du problemlos innerhalb deines Docker Containers Linux-Pakete installieren und aktualisieren. Dies geschieht beispielsweise über die Kommandozeile durch den Aufruf von `apt update && apt upgrade`.

Eine alternative, und besonders empfohlene Methode, insbesondere im Kontext von Docker Containern, besteht darin, einfach dein Docker Image zu aktualisieren und den ioBroker Docker Container anschließend neu zu erstellen. Mehr dazu erfährst du im nächsten Abschnitt.

### Docker Image Updates

Neben den Linux-Paketen, die im ioBroker Docker Image integriert sind, wird auch das Image selbst kontinuierlich weiterentwickelt. Bei jeder neuen Version wird automatisch ein aktualisiertes Docker Image erstellt und zum Download bereitgestellt. Dieses Image enthält sowohl die neueste ioBroker-Version als auch die aktuellen Versionen aller enthaltenen Linux-Pakete.

Durch ein Update des Docker Images erreichst du also gleich zwei Ziele auf einmal. Einerseits wird das Docker Image selbst auf den neuesten Stand gebracht, andererseits werden auch die Linux-Pakete aktualisiert. Dieser Ansatz ermöglicht es dir, effizient sicherzustellen, dass nicht nur ioBroker, sondern auch alle erforderlichen Systemkomponenten in ihrer aktuellen Version vorliegen.

### Adapter Updates

Lass uns nun zu den Updates innerhalb der ioBroker-Software übergehen. Da die Adapter deiner ioBroker-Instanz innerhalb des Docker Containers unter `/opt/iobroker` installiert sind und du dieses Verzeichnis bei jedem Update deines Containers wieder einbindest, bleiben die Adapterversionen beim Aktualisieren des Containers selbst unberührt. Stattdessen erfolgt die Aktualisierung deiner Adapter normalerweise über die Admin-Oberfläche deines ioBrokers. Im Abschnitt "Adapter" werden dir neue Versionen vorgeschlagen, die sich durch einen einfachen Klick auf den entsprechenden Button direkt installieren lassen.

{{< image src="/img/posts/2023-03-31/02_adapter_updates.png" alt="ioBroker Adapter Updates" width="100%" >}}

Alternativ gibt es aber natürlich auch die Möglichkeit [Adapter über die Kommandozeile zu installieren](https://www.iobroker.net/docu/index-98.htm?page_id=3971&lang=de#iobroker_add_adapterName).  

### ioBroker Core Updates

Beim "ioBroker Core" handelt es sich um den js-controller, der das Zentrum des ioBrokers bildet. Der js-controller ist ebenfalls unter `/opt/iobroker` installiert und wird daher durch ein Update des Docker Images nicht automatisch aktualisiert. Früher war es notwendig, ein Update des js-controllers über die Kommandozeile durchzuführen. Inzwischen kann jedoch ein js-controller-Update, ähnlich wie ein Adapter-Update, bequem über die Admin-Oberfläche im ioBroker erfolgen. Ein verfügbares Update wird im Abschnitt "Hosts" angezeigt.

{{< image src="/img/posts/2023-03-31/03_js-controller_updates.png" alt="ioBroker.js-controller Updates" width="100%" >}}

Natürlich gibt es auch hier weitere Möglichkeiten für die Installation des Updates. Stets aktuelle Details zu den update-Möglichkeiten des js-controllers im ioBroker Docker Container findest du in der [Offiziellen Doku des ioBroker Docker Images](https://docs.buanet.de/iobroker-docker-image/docs/#iobroker-js-controller-core-updates).

## Fazit

Abschließend möchte ich ein bekanntes Zitat teilen, das nicht nur in der ioBroker-Community, sondern auch in der gesamten digitalen Welt seine Berechtigung hat: "Kein Backup, kein Mitleid!" :smile: Dieser humorvolle Spruch verdeutlicht, wie entscheidend Backup-Strategien sind.

Ich hoffe, mein Beitrag konnte dir einen ausführlichen Überblick über die Wartung und Pflege deines ioBroker Docker Containers verschaffen. Die Bedeutung von Backups sollte nicht unterschätzt werden, insbesondere wenn du viel Zeit und Herzblut in dein Smarthome investiert hast. Es ist meiner Meinung nach unumgänglich, dass du eine eigene Backup-Strategie entwickelst und konsequent verfolgst.

In einer Zeit, in der täglich neue Sicherheitslücken Schlagzeilen machen und die Bedeutung der IT-Sicherheit stetig wächst, sollte es fast selbstverständlich sein, Geräte, die mit dem Internet verbunden sind, auf dem neuesten Softwarestand zu halten. Wenn du dies bereits bei deinem Smartphone und Notebook praktizierst, warum nicht auch im Smarthome? Denn am Ende des Tages gilt: Die Sicherheit deiner digitalen Welt liegt in deinen Händen.

&nbsp;
Für Fragen und Feedback nutze gerne die Kommentarfunktion zu diesem Beitrag. 
&nbsp;
MfG,
André