---
title: Verwaltung mehrerer Docker Hosts mit Portainer
subtitle: ''
description: 'Wie verwalte ich mehrere Docker Hosts über eine Portainer Instanz?'
summary: 'In diesem Tutorial zeige ich dir, wie du mit einer Portainer Instanz, ohne großen Aufwand, auch mehrere Docker Hosts verwalten kannst...'
date: 2023-03-05T20:51:13+01:00
lastmod: ''
draft: false
featuredImage: /img/posts/2023-03-05/header.png

tags: []
categories: [Tutorials, Portainer]

author: 'André (buanet)'

seo:
  image: ''

#comment:
#  enable: false
---

## Einleitung

In diesem Tutorial zeige ich dir, wie du mit einer Portainer Instanz, ohne großen Aufwand, auch mehrere Docker Hosts verwalten kannst. Dazu werden wir auf einem zweiten Docker Host den Portainer Agent installieren und ihn mit unserer bestehenden Portainer Instanz verbinden.

## Voraussetzungen

Folgendes setze ich voraus:
* Funktionierende Portainer Instanz (siehe [hier](/posts/2023/02/10_portainer_zur_verwaltung_des_docker_dienstes/) und/oder [hier](/posts/2023/02/12_portainer_auf_der_synology_diskstation/))
* Zweiter, Linux basierter Docker Host (Docker up & running)
* Zugriff auf die Kommandozeile des zweiten Docker Hosts

## Portainer Agent auf dem zweiten Docker Host einrichten

Damit wir den Docker Dienst auf unserem zweiten Docker Host aus der Ferne verwalten können benötigen wir einen ["Portainer Agent"](https://hub.docker.com/r/portainer/agent). Dabei handelt es sich, wie beim Portainer selbst, lediglich um einen Docker Container, den wir lokal auf unserem zweiten Docker Host einrichten müssen. Wie schon beim Portainer geschieht dies über die Kommandozeile. 

Mit dem folgenden Befehl starten wir also unseren Portainer Agent:
```bash
docker run -d -p 9001:9001 \
  --name portainer-agent --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /var/lib/docker/volumes:/var/lib/docker/volumes \
  portainer/agent:latest
```

Der Befehl setzt sich dabei aus folgenden Parametern zusammen:
* `docker run` << zum Erstellen eines neuen Containers
* `-d` << für "[detached](https://docs.docker.com/engine/reference/run/#detached--d)" 
* `-p 9001:9001` << Port über den sich Portainer mit dem Agent verbindet
* `--name portainer-agent` << Container Name
* `--restart=always` << Restart Policy = always
* `-v /var/run/docker.sock:/var/run/docker.sock` << Mount des Docker Sockets für die Verwaltung des Docker Dienstes 
* `-v /var/lib/docker/volumes:/var/lib/docker/volumes` << Mount des Verzeichnisses für Docker Volumes
* `portainer/agent:latest` << Verwendetes Image und Tag

{{< admonition type=note title="Hinweis" open=true >}}
Sollte der Benutzer, mit dem du an der Kommandozeile des zweiten Docker Hosts angemeldet bist keine Berechtigung für die Ausführung des `docker run` Befehls besitzen, ist ggf. ein `sudo` erforderlich.

Des Weiteren prüfe bitte ob der verwendete Port `9001` keinen Postkonflikt verursacht (ist z.B. auch der Standardport für die ioBroker Objects-DB). Durch Änderung des Parameters `-p 9001:9001` in z.B. `-p 8889:9001` ließe sich der Port, über den Portainer mit dem Agent Kontakt auf nimmt, auf Port `8889`ändern. 
{{< /admonition >}}

Wie du sicherlich festgestellt hast, ähnelt das Kommando sehr stark dem Kommando mit dem wir auch schon den Portainer gestartet haben. Der größte Unterschied besteht darin, dass wir kein lokales Verzeichnis für die Speicherung von Konfigurationsdaten benötigen, denn diese Daten werden nicht im Agent, sondern im Portainer gespeichert. Also in dem Verzeichnis, dass wir bei der Einrichtung des Portainers auf unserem "Haupt-Host" eingebunden haben.

Mit `/var/lib/docker/volumes` ist allerdings ein neuer Mount hinzugekommen. Hier handelt es sich um das Standardverzeichnis in dem Docker selbst Volumes erstellen kann. Damit später vom Portainer darauf zugegriffen werden kann, mounten wir dieses mit in den Portainer Agent.

Alle Anderen Parameter solltest du bereits von der [Einrichtung des Portainers](/posts/2023/02/10_portainer_zur_verwaltung_des_docker_dienstes/) kennen. Ich spare mir daher hier die Wiederholung.

Die Ausgabe auf der Kommandozeile sollte nun also ungefähr so aussehen:

```bash
andre@vm-docker-test:~$ sudo docker run -d -p 8889:9001 --name portainer-agent --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker/volumes:/var/lib/docker/volumes portainer/agent:latest
[sudo] Passwort für andre:
Unable to find image 'portainer/agent:latest' locally
latest: Pulling from portainer/agent
772227786281: Pull complete
96fd13befc87: Pull complete
a072f93dbf8b: Pull complete
a7de7247ee92: Pull complete
cd4280d0ffc0: Pull complete
32c744610ec2: Pull complete
77bbe6ca9fb5: Pull complete
45b6ddb3fba1: Pull complete
1783048236fb: Pull complete
93fd384a1bda: Pull complete
Digest: sha256:76598c54b29d90636cbe533bf96d5efbd885008b3d5e5ca36a9cc9238ebe47b3
Status: Downloaded newer image for portainer/agent:latest
3bdb68ee29134fe747d8d9a40790b939427d11719170bf226cfeab9b47bfa83c
```

Mit `docker ps` kannst du im Anschluss prüfen, ob der Container ordnungsgemäß erstellt worden ist und welchen Status er hat:

```bash
andre@vm-docker-test:~$ sudo docker ps
CONTAINER ID   IMAGE                    COMMAND     CREATED         STATUS         PORTS                                       NAMES
3bdb68ee2913   portainer/agent:latest   "./agent"   2 minutes ago   Up 2 minutes   0.0.0.0:8889->9001/tcp, :::8889->9001/tcp   portainer-agent
```

## Verbindung zwischen Portainer und Agent herstellen

Weiter geht es nun in unserer Portainer Web UI. Unter "Settings" navigieren wir zum Menüpunkt "Environments" und fügen über den Button "+ Add environment" ein neues Environment hinzu. Im nun startenden "Environment Wizard" wählen wir den Eintrag "Docker Standalone" und bestätigen mit dem Button "Start Wizard".

{{< image src="/img/posts/2023-03-05/portainer_agent1.png" alt="Portainer Environment Wizard" width="100%" >}}

Im nächsten Schritt nutzen wir die vorausgewählte Option "Agent". Wie du sehen kannst gibt es weitere Möglichkeiten Endpunkte zu verbinden. Meiner Meinung nach ist allerdings die Variante über den Agent die einfachste und sicherste Option.

{{< image src="/img/posts/2023-03-05/portainer_agent2.png" alt="Portainer Environment Wizard" width="100%" >}}

Etwas weiter unten müssen wir nun die Informationen zu unserem Environment ergänzen.

{{< image src="/img/posts/2023-03-05/portainer_agent3.png" alt="Portainer Environment Wizard" width="100%" >}}

Wir vergeben einen Namen und füllen die "Endpoint URL" mit der IP-Adresse unseres zweiten Docker Hosts, sowie dem für die Kommunikation vorgesehenen Port, getrennt durch einen Doppelpunkt, aus. Beim Klick auf "Connect" prüft Portainer unsere Konfiguration und fügt in der Liste der Environments (rechts am Bildrand) einen entsprechenden Eintrag hinzu. 

{{< image src="/img/posts/2023-03-05/portainer_agent4.png" alt="Portainer Environment Wizard" width="100%" >}}

Mit einem Klick auf den Button "Close" am ende der Seite schließen wir die Einrichtung ab. Wir landen in der Liste unserer Environments. Unser neues Environment sollte nun in der Liste aufgeführt sein.

{{< image src="/img/posts/2023-03-05/portainer_agent5.png" alt="Portainer Environment List" width="100%" >}}

{{< admonition type=note title="Hinweis" open=true >}}
Die Verbindung vom Portainer zum Agent kann nur einmal hergestellt werden. Damit sich nicht einfach eine zweite Portainer-Instanz mit dem Agent verbinden kann, und damit Zugriff auf den Docker Dienst erhält, tauschen Portainer und Agent bei der ersten Verbindung ein Secret aus. Soll der Agent später an eine andere Portainer Instanz umziehen, so muss der Portainer Agent Container neu erstellt werden. Weitere Infos dazu findest du in der [Portainer Doku](https://docs.portainer.io/). 
{{< /admonition >}}

Wechseln wir nun in den Menüpunkt "Home", sehen wir eine Übersicht der eingerichteten "Environments" zusammen mit Status und einigen Statistik-Werten.

{{< image src="/img/posts/2023-03-05/portainer_agent6.png" alt="Portainer Home" width="100%" >}}

Mit einem Klick auf unser neu eingerichtetes Environment gelangen wir in das Dashboard unseres zweiten Docker Hosts. In der Menüleiste sehen wir den ausgewählten Docker Host und können ihn nun über die Menüpunkte "Stacks", "Containers", "Images" usw. verwalten.

{{< image src="/img/posts/2023-03-05/portainer_agent7.png" alt="Portainer Dashboard" width="100%" >}}

---

Ich hoffe ich konnte dir mit diesem Tutorial zeigen was zu tun ist, um mit einer Portainer Instanz mehrere Docker Hosts verwalten zu können.
&nbsp;
Für Fragen und Feedback nutze gerne die Kommentarfunktion zu diesem Beitrag. 
&nbsp;
MfG,
André