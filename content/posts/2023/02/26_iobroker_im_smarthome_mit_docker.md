---
title: ioBroker im Smarthome mit Docker
subtitle: ''
description: 'Ein Überblick über die ioBroker Smarthome Software und das ioBroker Docker Image.'
summary: 'Eine zentrale Rolle in meinem Smarthome mit Docker spielt ioBroker. In diesem Artikel will ich dir einen kurzen Überblick über ioBroker geben und zeigen, wie einfach und schnell du einen ioBroker Docker Container für einen ersten Test aufsetzen kannst...'
date: 2023-02-26T22:25:17+01:00
lastmod: ''
draft: false
featuredImage: /img/posts/2023-02-26/header.png

tags: []
categories: [Docker Images, ioBroker]

author: 'André (buanet)'

seo:
  image: ''
---

## Einleitung

Eine zentrale Rolle in meinem Smarthome mit Docker spielt ioBroker. In diesem Artikel will ich dir einen kurzen Überblick über ioBroker geben und zeigen, wie einfach und schnell du einen ioBroker Docker Container für einen ersten Test aufsetzen kannst.

## Was ist ioBroker?

IoBroker ist eine in JavaScript geschriebene Open-Source-IoT-Plattform die Smarthome-Komponenten und Services verschiedener Hersteller verbindet. Mit Hilfe von Plugins (genannt: "Adapter") ist ioBroker in der Lage mit einer Vielzahl von IoT-Hardware und -Diensten über verschiedene Protokolle und APIs zu kommunizieren. Alle Daten werden in einer zentralen Datenbank gespeichert auf die alle Adapter zugreifen können. Damit ist es sehr einfach logische Verbindungen, Automatisierungsskripte und schöne Visualisierungen herstellerübergreifend aufzubauen.
Weitere Informationen zu ioBroker findest du unter [iobroker.net](https://www.iobroker.net).

## Was ist das ioBroker Docker Image

IoBroker ist nicht speziell für den Betrieb unter Docker entwickelt worden. Daher habe ich bereits 2017 damit begonnen ioBroker in ein Docker Image zu portieren. Seitdem hat sich das [Projekt](https://github.com/buanet/ioBroker.docker) stetig weiter entwickelt und trägt mittlerweile auch den Titel "Offizielles ioBroker Docker Image".
Weitere Informationen zum ioBroker Docker Image findest du auf meiner [Docs-Webseite](https://docs.buanet.de/de/iobroker-docker-image/).

## Wo finde ich weitere Infos? 

Bevor wir loslegen hier noch einen kleine Liste mit Links zu weiterführenden Informationen, Bezugsquellen und Dokumentationen:

* [Informationen zu ioBroker](https://www.iobroker.net)
* [Offizielle ioBroker Dokumentation](https://www.iobroker.net/#de/documentation)
* [Informationen zum ioBroker Docker Image](https://docs.buanet.de/de/iobroker-docker-image/)
* [Offizielle ioBroker Docker Image Dokumentation](https://docs.buanet.de/iobroker-docker-image/docs/)
* [IoBroker Docker Image Source Code & Issues](https://github.com/buanet/ioBroker.docker)
* [IoBroker Docker Image im Docker Hub](https://hub.docker.com/r/buanet/iobroker)
* IoBroker Communities: [Forum](https://forum.iobroker.net/), [Discord](https://discord.gg/HwUCwsH), [Telegram](https://t.me/+Xfjuou6-LztkOTBi), [Facebook](https://www.facebook.com/groups/440499112958264)

## ioBroker Docker Container erstellen

Im folgenden zeige ich dir, wie du schnell und einfach einen ioBroker Docker Container für einen ersten Test unter Docker erstellst und worauf du achten musst. Diese Anleitung ist unter Docker allgemein gültig und eignet sich besonders, wenn du schon Erfahrungen mit Docker gesammelt hast.  

### Voraussetzungen

Folgendes setze ich voraus:
* Linux basierter Docker Host (NAS Systeme basieren in der Regel auf Linux)
* Zugriff auf die Kommandozeile des Docker Hosts
* Installierter und gestarteter Docker Dienst
* Internetzugang für den Docker Host

### Persistente Datenspeicherung

Kurz ein paar Worte zur persistenten Speicherung von Konfigurationsdaten.

Der ioBroker im Docker Container speichert alle Konfigurationsdaten unter `/opt/iobroker`. Damit diese Daten beim Löschen des Containers nicht verloren gehen, legt das Docker Image (ab v8.0.0) beim Erstellen des Containers automatisch ein [Standard-Docker-Volume](https://docs.docker.com/storage/volumes/) an. Dieses Verhalten kannst du überschreiben indem du ein eigenes Volume oder einen Dateipfad an den Pfad `/opt/iobroker` in den Container mountest. Möchtest du ein Docker Volume nutzen, solltest du es vor dem Start des Container bereits via `docker volume create [dein_volume_name]` erstellt haben.

Beim Start des Containers mountest du dein Volume oder Verzeichnis dann mit dieser Option: 

```bash
-v [dein_volume_oder_verzeichnis]:/opt/iobroker
```

In docker-compose sieht das Ganze dann so aus:

```bash
    volumes:
      - [dein_volume_oder_verzeichnis]:/opt/iobroker
```

### Per Kommandozeile erstellen

Mit dem folgenden Kommando können wir einen einfachen ioBroker Docker Container erstellen. Ich verzichte dabei auf die Angabe eines Volumes oder Dateipfads für das Verzeichnis `/opt/iobroker`und lasse von Docker ein Standard Volume anlegen.

```bash
docker run -d -p 8081:8081 \
  --name iobroker --hostname iobroker --restart=always \
  buanet/iobroker:latest
```

Der Befehl setzt sich dabei aus folgenden Parametern zusammen:
* `docker run` << zum Erstellen eines neuen Containers
* `-d` << für "[detached](https://docs.docker.com/engine/reference/run/#detached--d)" 
* `-p 8081:8081` << Port für den Zugriff auf die Web UI
* `--name iobroker` << Container Name
* `--hostname iobroker` << Container Hostname (auch Instanzname innerhalb von ioBroker)
* `--restart=always` << Restart Policy
* `buanet/iobroker:latest` << Verwendetes Image und Tag

{{< admonition type=info title="Hinweis" open=true >}}
Dies ist ein einfaches Beispiel zum Starten eines ioBroker Docker Containers. Sehr wahrscheinlich sind weitere Parameter, Ports und Variablen hinzuzufügen, damit der Container später auch produktiv genutzt werden kann. Insbesondere bei der Wahl und Konfiguration des Netzwerks gibt es Einiges zu beachten. Weitere Details zur Konfiguration des Containers findest du in der [offiziellen Doku](https://docs.buanet.de/iobroker-docker-image/docs/).
{{< /admonition >}}

Nachdem der Container angelegt ist, lassen wir uns mit dem Befehl `docker logs --follow iobroker` das Log anzeigen, um den Startprozess verfolgen zu können. 

```bash
$ docker logs --follow iobroker
 
--------------------------------------------------------------------------------
-------------------------     2023-03-17 21:40:55      -------------------------
--------------------------------------------------------------------------------
-----                                                                      -----
----- ██╗  ██████╗  ██████╗  ██████╗   ██████╗  ██╗  ██╗ ███████╗ ██████╗  -----
----- ██║ ██╔═══██╗ ██╔══██╗ ██╔══██╗ ██╔═══██╗ ██║ ██╔╝ ██╔════╝ ██╔══██╗ -----
----- ██║ ██║   ██║ ██████╔╝ ██████╔╝ ██║   ██║ █████╔╝  █████╗   ██████╔╝ -----
----- ██║ ██║   ██║ ██╔══██╗ ██╔══██╗ ██║   ██║ ██╔═██╗  ██╔══╝   ██╔══██╗ -----
----- ██║ ╚██████╔╝ ██████╔╝ ██║  ██║ ╚██████╔╝ ██║  ██╗ ███████╗ ██║  ██║ -----
----- ╚═╝  ╚═════╝  ╚═════╝  ╚═╝  ╚═╝  ╚═════╝  ╚═╝  ╚═╝ ╚══════╝ ╚═╝  ╚═╝ -----
-----                                                                      -----
-----              Welcome to your ioBroker Docker container!              -----
-----                    Startupscript is now running!                     -----
-----                          Please be patient!                          -----
--------------------------------------------------------------------------------
 
--------------------------------------------------------------------------------
-----                          System Information                          -----
-----                    arch:                x86_64                       -----
-----                    hostname:            iobroker                     -----
-----                                                                      -----
-----                          Version Information                         -----
-----                    image:               v8.0.0                       -----
-----                    build:               2023-02-20T17:21:59+00:00    -----
-----                    node:                v18.14.1                     -----
-----                    npm:                 9.3.1                        -----
-----                                                                      -----
-----                        Environment Variables                         -----
-----                    SETGID:              1000                         -----
-----                    SETUID:              1000                         -----
--------------------------------------------------------------------------------
 
--------------------------------------------------------------------------------
-----                   Step 1 of 5: Preparing container                   -----
--------------------------------------------------------------------------------
 
Updating Linux packages on first run... Done.
 
Registering maintenance script as command... Done.
 
--------------------------------------------------------------------------------
-----             Step 2 of 5: Detecting ioBroker installation             -----
--------------------------------------------------------------------------------
 
Existing installation of ioBroker detected in "/opt/iobroker".
 
--------------------------------------------------------------------------------
-----             Step 3 of 5: Checking ioBroker installation              -----
--------------------------------------------------------------------------------
 
(Re)setting permissions (This might take a while! Please be patient!)... Done.
 
Fixing "sudo-bug" by replacing sudo with gosu... Done.
 
Initializing a fresh installation of ioBroker... Done.
 
Hostname in ioBroker does not match the hostname of this container.
Updating hostname to "iobroker"... The host for instance "system.adapter.admin.0" was changed from "buildkitsandbox" to "iobroker".
The host for instance "system.adapter.discovery.0" was changed from "buildkitsandbox" to "iobroker".
The host for instance "system.adapter.backitup.0" was changed from "buildkitsandbox" to "iobroker".
Done.
 
--------------------------------------------------------------------------------
-----                Step 4 of 5: Applying special settings                -----
--------------------------------------------------------------------------------
 
Some adapters have special requirements/ settings which can be activated by the use of environment variables.
For more information see ioBroker Docker Image Docs (https://docs.buanet.de/iobroker-docker-image/docs/).
 
 
--------------------------------------------------------------------------------
-----                    Step 5 of 5: ioBroker startup                     -----
--------------------------------------------------------------------------------
 
Starting ioBroker... 
 
##### #### ### ## # iobroker.js-controller log output # ## ### #### #####
host.iobroker check instance "system.adapter.admin.0" for host "iobroker"
host.iobroker check instance "system.adapter.discovery.0" for host "iobroker"
host.iobroker check instance "system.adapter.backitup.0" for host "iobroker"
```

Wenn das Log ungefähr so aussieht, dann hat alles geklappt und wir können auf die ioBroker Admin UI zugreifen. Dazu verwenden wir die URL nach folgendem Schema: 

`http://[Hostname_oder_IP_des_Docker_Host]:8081`

Es sollte der ioBroker Setup Assistent für die Ersteinrichtung erscheinen. 

{{< image src="/img/posts/2023-02-26/iobroker1.png" alt="ioBroker Setup Assistent" width="100%" >}}

Unser ioBroker Docker Container ist damit startklar. 

### Bonus I: Mit Portainer erstellen

Als kleine Zugabe zeige ich dir jetzt noch, wie du den oben bereits erstellten, einfachen ioBroker Container über die Portainer Weboberfläche erstellen kannst. 

Dazu öffnen wir die Portainer Weboberfläche und rufen den Menüpunkt "Containers" auf.

{{< image src="/img/posts/2023-02-26/iobroker2.png" alt="ioBroker Portainer Container" width="100%" >}}

Über den Button "+ Add container" gelangen wir in die Oberfläche zur Erstellung eines neuen Containers. 

{{< image src="/img/posts/2023-02-26/iobroker3.png" alt="ioBroker Portainer Container Allgemein" width="100%" >}}

Wir vergeben einen Namen, in meinem Fall `iobroker` und tragen unter "Image" das Image + Tag ein, welches wir verwenden wollen. Analog zum obigen Beispiel ist dies `buanet/iobroker:latest`.
Da wir den ioBroker Docker Container mit dem Standardnetzwerk Bridge starten werden, müssen wir unter "Network ports configuration" mit dem Button "+ publish a new network port" eine neue Portfreigabe für den ioBroker Admin Port `8081` erstellen. Wir reichen den Port 1:1 durch und füllen daher die Felder für host und port mit den identischen Werten. 

Weiter geht es mit den "Advanced container settings". unter dem Punkt "Volumes" hätten wir die Möglichkeit unser eigenes Volume oder Verzeichnis für den ioBroker zu mounten. Analog zum obigen Beispiel verzichte ich allerdings darauf und lasse das Volume von Docker automatisch anlegen. 

Im Bereich "Network" überprüfen wir die Auswahl von `bridge` im Feld "Network". Direkt darunter können wir den "Hostname" festlegen. Wie oben nutzen wir hier `iobroker`.

{{< image src="/img/posts/2023-02-26/iobroker4.png" alt="ioBroker Portainer Container Network" width="100%" >}}

Bleibt noch die Einstellung für die "Restart policy", welche dafür sorgt, dass unser Container automatisch neu startet, wenn es ein Problem gibt. Die Einstellung finden wir um gleichnamigen Bereich. Wir wählen hier `always`. 

{{< image src="/img/posts/2023-02-26/iobroker5.png" alt="ioBroker Portainer Container Restart Policy" width="100%" >}}

Das war es auch schon. Mit einem Klick auf den Button "Deploy the container" startet die Erstellung.

Hier kann es jetzt mitunter ein wenig dauern, da das Docker Image beim ersten Mal erst einmal herunter geladen werden muss. Nach Abschluss des Vorgangs landen wir automatisch in der Übersicht der "Containers".

{{< image src="/img/posts/2023-02-26/iobroker6.png" alt="ioBroker Portainer Container Übersicht" width="100%" >}}

Mit einem Klick auf den Namen können wir uns die Details anzeigen lassen. Auch einen Link zu "Logs" finden wir hier.

{{< image src="/img/posts/2023-02-26/iobroker7.png" alt="ioBroker Portainer Container Details" width="100%" >}}

Ab hier geht es dann wie im obigen Beispiel mit der Logausgabe weiter. 

### Bonus II: Mit Stacks unter Portainer erstellen

Wenn dir das manuelle Anlegen des Docker Containers über die Portainer Weboberfläche zu umständlich ist, kannst du dein Ziel auch über die [Stacks](https://docs.portainer.io/user/docker/stacks)-Funktion von Portainer erreichen. Das Ganze funktioniert wie [docker-compose](https://docs.docker.com/compose/), allerdings gestützt über die Portainer Weboberfläche inkl. Editor zum Bearbeiten des Stackfiles.

Für die grundlegenden Instruktionen zum [Verwenden von Stacks in Portainer](/posts/2023/02/15_verwenden_von_stacks_in_portainer/) empfehle ich dir, einen Blick in den verlinkten Artikel zu werfen.

Das YAML-File für den einfachen ioBroker Docker Container ([siehe oben](#per-kommandozeile-erstellen)) würde dabei so aussehen:

```YAML
version: "3"
services:
  iobroker:
    container_name: iobroker
    image: buanet/iobroker:latest
    hostname: iobroker
    restart: always
    ports:
      - "8081:8081"
```
Einen Stack kannst du in der Portainer Weboberfläche unter "Stacks" erstellen. Mit einem Klick auf "+Add stack" gelangst du in den Webeditor. Vergib einen Namen und kopiere das Stackfile in den Editor. 

Ein Klick auf "Deploy the stack" startet die Erstellung des bzw. der Docker Container die im Stackfile definiert worden sind. Im Anschluss gelangst du automatisch in die Übersicht des Stacks und kannst, wie oben beschrieben, mit einem Blick in das Log fortfahren.

---

Ich hoffe ich konnte dir mit diesem Beitrag einen kleinen Überblick über ioBroker unter Docker geben.
&nbsp;
Für Fragen und Feedback nutze gerne die Kommentarfunktion zu diesem Beitrag. 
&nbsp;
MfG,
André