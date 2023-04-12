---
title: 'Portainer auf der Synology DiskStation'
#subtitle: 'Das Fundament für unser Smarthome mit Docker'
description: 'Ein Tutorial zur Einrichtung von Portainer auf der Synology DiskStation'
summary: 'Nachdem ich in meinem letzten Beitrag schon Einiges allgemein über Portainer geschrieben habe, möchte ich dir in diesem Tutorial zeigen, wie du Portainer speziell auf deine Synology DiskStation bekommst....'
date: 2023-02-12T21:12:44+01:00
lastmod: ''
draft: false
featuredImage: /img/posts/2023-02-12/header.png

tags: [portainer, docker, container, synology, diskstation]
categories: [Tutorials, DiskStation, Portainer]

author: 'André (buanet)'

seo:
  image: ''
---
## Einleitung

Nachdem ich in meinem [letzten Beitrag](/posts/2023/02/10_portainer_zur_verwaltung_des_docker_dienstes) schon Einiges allgemein über Portainer geschrieben habe, möchte ich dir in diesem Tutorial zeigen, wie du Portainer speziell auf deine Synology DiskStation bekommst. Dabei werden wir sogar komplett auf die Kommandozeile verzichten. :) Zunächst aber wie üblich die...

## Voraussetzungen

Natürlich benötigst du eine Synology DiskStation, oder etwas allgemeiner, ein NAS auf dem der Synology DiskStation Manager (DSM) ausgeführt wird. Dabei ist es unerheblich ob dein System noch auf DSM 6, oder bereits mit DSM 7 arbeitet. Hauptsache ist, dass dein NAS das Docker Paket von Synology unterstützt (siehe [Liste der Unterstützten Modelle](https://www.synology.com/de-de/dsm/packages/Docker)).

Da die Installation des Docker Pakets in der Regel keine große Hürde darstellen sollte, gehe ich auch davon aus, dass du dieses bereits über die Paketverwaltung Installiert hast und die Docker-Anwendung auf deinem Synology Desktop verfügbar ist.

Zu guter Letzt brauchen wir noch ein Verzeichnis, welches wir als Datenverzeichnis in den Portainer Container mounten, um eine persistente Speicherung der Konfigurationsdaten zu ermöglichen. Hierzu können wir einfach über die Systemsteuerung einen gemeinsamen Order und die FileStation einen Unterordner anlegen. Ich habe mir dazu einen gemeinsamen Ordner `docker` auf Volume1 erstellt und einen Unterordner `portainer-data` angelegt. Daraus ergibt sich in meinem Fall der folgende Pfad für die Einbindung im Portainer Contaier:

`/volume1/docker/portainer-data`

{{< image src="/img/posts/2023-02-12/portainer_synology1.png" alt="Portainer Datenverzeichnis" width="100%" >}}

Kurz zusammengefasst also nochmal die Voraussetzungen:
* Vorhandensein einer DiskStation mit DSM 6 oder 7
* Installiertes Docker Paket aus dem Paket-Zentrum
* Verzeichnis für die Speicherung der Portainer-Konfigurationsdaten

## Portainer Container starten

Im Prinzip ist es nun bereits Zeit den Portainer Container mit dem `docker run`-Befehl zu starten. Den Befehl habe ich in meinem [letzten Beitrag](/posts/2023/02/10_portainer_zur_verwaltung_des_docker_dienstes) schon ausführlich erläutert. Er kann genau so auch auf der DiskStation verwendet werden. Einziger Unterschied ist, dass wir den Dateipfad für die Portainer Konfigurationsdaten anpassen müssen. Am Ende steht dabei dieser Befehl:
```bash
docker run -d -p 9443:9443 \
  --name portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /volume1/docker/portainer-data:/data \
  portainer/portainer-ce:latest
```

Statt diesen Befehl jetzt über die Kommandozeile der DS einzutippen, nutzen wir zur Ausführung den Aufgabenplaner in der Systemsteuerung. Hier erstellen wir zunächst eine neue "Geplante Aufgabe" vom Typ "Benutzerdefiniertes Skript".

{{< image src="/img/posts/2023-02-12/portainer_synology2.png" alt="DSM Aufgabenplaner - Allgemein" width="100%" >}}

Wir vergeben einen Namen und ändern den (ausführenden) Benutzer auf `root`. Das ist notwendig, da zur Bedienung des Docker Dienstes erweiterte Berechtigungen erforderlich sind. Zuletzt können wir das Häkchen bei "Aktiviert" noch entfernen, da wir die Aufgabe ja nur manuell ausführen werden.

{{< image src="/img/posts/2023-02-12/portainer_synology3.png" alt="DSM Aufgabenplaner - Zeitplan" width="100%" >}}

Den Zeitplan können wir konfigurieren wie wir wollen, da die Aufgabe ja nicht aktiviert sein wird. Ich mache es trotzdem wie im Screenshot dargestellt. So bin ich sicher, dass auch falls die Aufgabe mal aus Versehen aktiviert wurde keine zeitgesteuerte Ausführung statt findet.

{{< image src="/img/posts/2023-02-12/portainer_synology4.png" alt="DSM Aufgabenplaner - Aufgabeneinstellungen" width="100%" >}}

In der letzten Registerkarte "Aufgabeneinstellungen" konfigurieren wir dann letztendlich unter "Benutzerdefiniertes Skript" unseren `docker run`-Befehl.
Damit ist die Konfiguration der Aufgabe abgeschlossen. Über die Schaltfläche "Ausführen" oder per Rechtsklick > "Ausführen" auf die Aufgabe, können wir die Erstellung des Portainer Docker Containers anstoßen.

{{< image src="/img/posts/2023-02-12/portainer_synology5.png" alt="DSM Aufgabenplaner - Aufgabe Starten" width="100%" >}}

Die Abfrage bestätigen wir natürlich mit "Ja".

Die Erstellung kann nun einen kurzen Moment dauern. Den Erfolg können wir über die im DSM eingebaute Administrationsoberfläche für Docker überprüfen.

{{< image src="/img/posts/2023-02-12/portainer_synology6.png" alt="DSM Docker - Container" width="100%" >}}

Wenn es so ausschaut, dann läuft unser Portainer bereits. Wir können nun auf die Weboberfläche des Portainer zugreifen, indem wir die URL nach folgendem Schema aufrufen: 

`https://[IP oder Hostname]:9443`

Die folgende Seite sollte erscheinen:

{{< image src="/img/posts/2023-02-12/portainer_synology7.png" alt="Portainer - Start" width="100%" >}}

Hier können wir Portainer aus einem Backup wiederherstellen oder einfach mit einer frischen Portainer Installation starten, indem wir einen neuen Benutzer für die Administration anlegen.
Nach einem Klick auf "Create user" landen wir im "Quick Setup".  

{{< image src="/img/posts/2023-02-12/portainer_synology8.png" alt="Portainer - Quick Setup" width="100%" >}}

Mit einem Klick auf "Get Started" ist das "Quick Setup" auch schon erledigt, und wir landen in der Übersicht unseres Portainers. 

{{< image src="/img/posts/2023-02-12/portainer_synology9.png" alt="Portainer - Home" width="100%" >}}

Unser Portainer ist nun startklar. 

Da sich mit einer Portainer Weboberfläche mehrere Docker Hosts (Environments) verwalten lassen, müssen wir zum Anlegen unseres ersten Containers noch unser Environment auswählen. Dies tun wir mit einem Klick auf "local" oder auf den Button "Live connect". 

## Bonus: Portainer Aktualisieren

Von Zeit zu Zeit bekommt natürlich auch Portainer Updates. Wenn ein neues Update zur Verfügung steht, informiert euch Portainer über die Weboberfläche. Da Portainer ein Docker Container ist, lässt sich das Update aber leider nicht über die Weboberfläche einspielen. Stattdessen müssen wir den Portainer Container und das Portainer Docker Image löschen und den Container neu erstellen. Und das geht so:

Zunächst rufen wir die Docker Web UI im Synology DSM auf und stoppen den Portainer Container.

{{< image src="/img/posts/2023-02-12/portainer_synology10.png" alt="DSM Docker - Container entfernen" width="100%" >}}

Über das Menü "Aktion" > "Löschen" oder "Rechtsklick" > "Löschen" entfernen wir den Portainer Container.
Anschließend wechseln wir auf der linken Seite in den Bereich "Image".

{{< image src="/img/posts/2023-02-12/portainer_synology11.png" alt="DSM Docker - Image entfernen" width="100%" >}}

Dort entfernen wir auch das Portainer Docker Image. Anschließend können wie über den Aufgabenplaner, wie oben beschrieben, den Portainer Container neu erstellen. Dabei wird automatisch das neuste Docker Image geladen und verwendet. Sobald der Container erstellt ist, sollte auch der Zugriff auf die Weboberfläche wieder wie gewohnt funktionieren. 

{{< admonition type=note title="Hinweis" open=true >}}
Für den Fall, dass beim Update etwas schief geht, bietet es sich an vor dem Update eure Portainer Konfigurationsdaten zu sichern. Das kannst du entweder über das weg Kopieren des `portainer-data`-Ordners machen oder über den Export der Konfiguration im Portainer unter "Settings" > "Backup Portainer".
{{< /admonition >}}

---

Ich hoffe ich konnte dir mit diesem Beitrag einen kleinen Überblick über die Einrichtung von Portainer auf der Synology DiskStation geben.  
&nbsp;
Für Fragen und Feedback nutze gerne die Kommentarfunktion zu diesem Beitrag. 
&nbsp;
MfG,
André
