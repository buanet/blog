---
title: 'Portainer zur Verwaltung des Docker Dienstes'
#subtitle: 'Das Fundament für unser Smarthome mit Docker'
description: 'Ein Überblick über Portainer als grafische Benutzeroberfläche zur Administration des Docker Dienstes'
summary: 'Bei Docker handelt es sich ja bekanntlich um einen Dienst der normalerweise über die Kommandozeile bedient wird. Wer allerdings keine Lust hat Kommandos auswendig zu lernen oder lange Befehlsaufrufe aus verschiedenen Parametern zusammen zu stellen, der greift gerne auf eine grafische Oberfläche zurück...'
date: 2023-02-10T20:23:42+01:00
lastmod: ''
draft: false
featuredImage: /img/posts/2023-02-10/header.png

tags: [portainer, docker, container]
categories: [Docker Images, Portainer]

author: 'André (buanet)'

seo:
  image: ''
---
## Einleitung

Bei Docker handelt es sich ja bekanntlich um einen Dienst der normalerweise über die Kommandozeile bedient wird. Über verschiedene Kommandos lassen sich Images herunterladen, Container erstellen und noch vieles mehr. Wer allerdings keine Lust hat Kommandos auswendig zu lernen oder lange Befehlsaufrufe aus verschiedenen Parametern zusammen zu stellen, der greift gerne auf eine grafische Oberfläche zurück. Bei vielen Systemen, die heutzutage Docker von Haus aus unterstützen, sind diese grafischen Benutzeroberflächen bereits integriert. Bei den gängigen NAS-Systemen wie Synology DSM, TrueNAS oder Unraid zum Beispiel, ist die Bedienung des Docker Dienstes direkt über die eigenen, webbasierten Administrationsoberflächen möglich. Doch nicht immer schöpfen diese Oberflächen die vollen Möglichkeiten, die Docker eigentlich bietet, auch aus. So ist es zum Beispiel im DSM bis heute nicht möglich Docker Containern Systemressourcen oder Devices zuzuteilen.

## Was ist Portainer?

An dieser Stelle kommt Portainer ins Spiel. Portainer ist eine eigens für die Verwaltung von Docker konzipierte, webbasierte Benutzeroberfläche und wird einfach als eigener Docker Container aufgesetzt. Indem der Container Zugriff auf den Docker Socket erhält, ist es dem darin laufenden Portainer möglich andere Container zu erstellen und zu verwalten (und noch vieles mehr).  
Die Community Edition (CE) von Portainer ist ein Open-Source-Projekt und steht allen Usern kostenlos zur Verfügung. Zusätzlich existiert auch eine kostenpflichtige Business Edition mit einigen interessanten Zusatzfeatures. In meinen Tutorials beziehe ich mich aber ausschließlich auf die Community Edition.

{{< admonition type=tip title="Tip" open=true >}}
Die Portainer Business Edition lässt sich für [bis zu 5 Nodes ebenfalls kostenlos](https://www.portainer.io/take-5) nutzen. Erforderlich ist lediglich eine Registrierung. Die entsprechende Lizenz wird per E-Mail zugeschickt. Mit der Lizenz kann dann das `portainer-ee` Docker Image verwendet werden. Ein [Upgrade von der Community Edition](https://docs.portainer.io/start/upgrade) ist dabei problemlos möglich. 
{{< /admonition >}}

## Wo finde ich weitere Infos?

Bevor ich dir an einem kurzen Beispiel zeige, wie du einen Portainer Docker Container auf setzt und die Ersteinrichtung erledigst, hier noch eine kleine Linkliste zu den wichtigsten Informationen rund um Portainer: 

* [Portainer Website >> portainer.io](https://www.portainer.io/)
* [Portainer Documentation >> docs.portainer.io](https://docs.portainer.io/)
* [Portainer Source Code & Issues >> github.com/portainer/portainer](https://github.com/portainer/portainer)
* [Portainer Business Edition - Take 5 for free >> portainer.io/take-5](https://www.portainer.io/take-5)
* [Portainer Comunity Edition im Docker Hub >> hub.docker.com/r/portainer/portainer-ce](https://hub.docker.com/r/portainer/portainer-ce)
* [Portainer Business Edition im Docker Hub >> hub.docker.com/r/portainer/portainer-ee](https://hub.docker.com/r/portainer/portainer-ce)

## Portainer Docker Container erstellen

Im Folgenden werden wir einen frischen Portainer Docker Container der Community Edition erstellen und dafür sorgen, dass unsere Konfiguration in einem lokalen Verzeichnis bzw. Docker Volume gespeichert wird.

### Voraussetzungen

Ich gehe davon aus, dass die folgenden Voraussetzungen bereits erfüllt sind:
* Linux basierter Docker Host (NAS Systeme basieren in der Regel auf Linux)
* Zugriff auf die Kommandozeile des Docker Hosts
* Installierter und gestarteter Docker Dienst
* Internetzugang für den Docker Host

### Persistente Datenspeicherung

Damit unsere Konfigurationsdaten dauerhaft gespeichert werden, benötigen wir entweder ein lokales Verzeichnis oder ein Docker-Volume auf dem Docker Host. Dieses Verzeichnis bzw. Volume mounten wir dann beim Erstellen des Docker Containers in den Pfad `/data`, in dem Portainer alle Konfigurationsdaten speichert. So stellen wir sicher, dass unsere Konfigurationsdaten erhalten bleiben, auch wenn der Portainer Container gelöscht wird. 

Ich persönlich bevorzuge die Speicherung der Daten meiner Container in einem Verzeichnis, statt in einem Docker Volume. Dazu lege ich mir zunächst im Verzeichnis `/mnt` meines Docker Hosts ein Verzeichnis `docker` an. Es tut natürlich auch jeder andere lokale Pfad, wie zum Beispiel das Home Verzeichnis deines Benutzers.
Innerhalb des `docker`-Verzeichnisses erstelle ich mir dann ein Unterverzeichnis `portainer_data`. Der vollständig Pfad für meine Portainer Daten lautet nun also: `/mnt/docker/portainer_data`.

### Portainer Container starten

##### Mit lokalem Verzeichnispfad

Da nun alles vorbereitet ist, erstellen wir den Container mit folgendem Befehl: 

```bash
docker run -d -p 9443:9443 \
  --name portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /mnt/docker/portainer_data:/data \
  portainer/portainer-ce:latest
```

Der Befehl setzt sich dabei aus folgenden Parametern zusammen:
* `docker run` << zum Erstellen eines neuen Containers
* `-d` << für "[detached](https://docs.docker.com/engine/reference/run/#detached--d)" 
* `-p 9443:9443` << Port für den Zugriff auf die Web UI
* `--name portainer` << Container Name
* `--restart=always` << Restart Policy = always
* `-v /var/run/docker.sock:/var/run/docker.sock` << Mount des Docker Sockets
* `-v /mnt/docker/portainer_data:/data` << Mount des Datenverzeichnisses
* `portainer/portainer-ce:latest` << Verwendetes Image und Tag

##### Mit Docker Volume

Falls du statt eines lokalen Verzeichnisses ein Docker Volume verwenden willst, kannst du mit dem folgenden Befehl ein Volume mit dem Namen `portainer-data` erstellen:

```bash
docker volume create portainer-data
```

Den Portainer Container startest du in diesem Fall dann mit:
```bash
docker run -d -p 9443:9443 \
  --name portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer-data:/data \
  portainer/portainer-ce:latest
```

### Zugriff auf die Portainer Web UI

Nachdem der Container gestartet ist, können wir auf die Weboberfläche des Portainer zugreifen, indem wir die URL nach folgendem Schema aufrufen: 

`https://[IP oder Hostname]:9443`

Wenn alles korrekt ausgeführt wurde, erscheint die folgende Seite:

{{< image src="/img/posts/2023-02-10/portainer1.png" alt="Portainer Startseite" width="100%" >}}

Hier können wir Portainer aus einem Backup wiederherstellen oder einfach mit einer frischen Portainer Installation starten, indem wir einen neuen Benutzer für die Administration anlegen.
Nach einem Klick auf "Create user" landen wir im "Quick Setup".  

{{< image src="/img/posts/2023-02-10/portainer2.png" alt="Portainer Quick Setup" width="100%" >}}

Mit einem Klick auf "Get Started" ist das "Quick Setup" auch schon erledigt, und wir landen in der Übersicht unseres Portainers. 

{{< image src="/img/posts/2023-02-10/portainer3.png" alt="Portainer Home" width="100%" >}}

Unser Portainer ist nun startklar. 

Da sich mit einer Portainer Weboberfläche mehrere Docker Hosts (Environments) verwalten lassen, müssen wir zum Anlegen unseres ersten Containers noch unser Environment auswählen. Dies tun wir mit einem Klick auf "local" oder auf den Button "Live connect". 

---

Ich hoffe ich konnte dir mit diesem Beitrag einen kleinen Überblick über Portainer und dessen Inbetriebnahme geben.  
&nbsp;
Für Fragen und Feedback nutze gerne die Kommentarfunktion zu diesem Beitrag. 
&nbsp;
MfG,
André