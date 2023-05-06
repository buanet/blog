---
title: 'Einrichtung eines benutzerdefinierten Bridge Netzwerks mit Portainer'
subtitle: ''
description: 'Ein Tutorial zur Einrichtung eines benutzerdefinierten Docker Bridge Netzwerks mit Portainer'
summary: 'Wer mehrere Docker Container über das Standard-Netzwerk "bridge" verbindet, wird früher oder später die Einschränkungen dieses Netzwerks kennenlernen. Doch es gibt eine Lösung. Ein benutzerdefiniertes Bridge Netzwerk muss her. Wie du dieses einrichtest zeige ich dir in diesem Tutorial...'
date: 2023-02-23T20:49:17+01:00
lastmod: ''
draft: false
featuredImage: /img/posts/2023-02-23/header.png

tags: []
categories: [Tutorials, Portainer]

author: 'André (buanet)'

seo:
  image: ''
---

## Einleitung

Wer mehrere Docker Container über das Standard-Netzwerk "bridge" verbindet, wird wohl früher oder später darüber stolpern: Im Standard-Bridge-Netzwerk von Docker gibt es unter anderem nicht nur keine Namensauflösung (DNS) sondern auch keine Möglichkeit den Containern feste IP-Adressen im virtuellen Netz zu vergeben (alle Einschränkungen zum Nachlesen in den [Docker Docs](https://docs.docker.com/network/bridge/#differences-between-user-defined-bridges-and-the-default-bridge)). Doch es gibt eine Lösung. Legt man sich nämlich sein eigenes Bridge-Netzwerk an umgeht man die Beschränkungen.

Wie man ein benutzerdefiniertes Bridge Netzwerk über Portainer anlegt, das zeige ich dir in diesem Tutorial.

## Voraussetzungen

Selbstverständlich gibt es auch hier ein paar Voraussetzungen die du schon erfüllt haben solltest. Hier eine kurze Liste:

* Vorhandensein eines auf Linux basierenden [Docker Hosts oder NAS mit Docker](/posts/2023/02/01_grundlagen/)
* [Portainer zur Verwaltung des Docker Dienstes](/posts/2023/02/10_portainer_zur_verwaltung_des_docker_dienstes)
* Grundverständnis zum Thema [(Docker) Netzwerk](https://docs.docker.com/network/)
* optional: Zugriff auf die Kommandozeile des Docker Hosts (nur für Bonus)

## Bridge Netzwerk über Portainer anlegen

Erste Anlaufstelle für Docker Netzwerke ist der Menüpunkt "networks" in der Portainer Weboberfläche. 

{{< image src="/img/posts/2023-02-23/portainer_bridge1.png" alt="Portainer - Networks" width="100%" >}}

Hier sehen wir eine Übersicht über die bisher angelegten Docker Netzwerke. In meinem Fall sind das lediglich die Docker Standardkonfigurationen für "bridge", "host" und "none". Über den Button "+ Add network" können wir nun ein neues Netzwerk definieren. 

{{< image src="/img/posts/2023-02-23/portainer_bridge2.png" alt="Portainer - Networks" width="100%" >}}

Zunächst vergeben wir einen aussagekräftigen Namen. In meinem Fall nenne ich das Netzwerk "my_bridge". Im Feld "Driver" wählen wir "bridge", denn wir wollen ja ein benutzerdefiniertes Bridge Netzwerk erstellen. 

Nun müssen wir die IPv4 bzw. IPv6 Konfiguration für unser Netzwerk festlegen. Da mir für meinen Anwendungszweck IPv4 ausreicht, konfiguriere ich lediglich die IPv4 Optionen.
Analog zum Subnetz des Docker Default Bridge Netzwerks wähle ich für mein Netzwerk das Subnetz `172.20.0.0/16` und lege das Gateway auf `172.20.0.1` fest. Für die "IP range" (den Adressbereich) wähle ich `172.20.0.0/24`. Dies beschränkt die IP-Adressen meines Netzes auf den Bereich von `172.20.0.1` bis `172.20.0.254`. 

{{< admonition type=info title="Info" open=true >}}
Da dieses Tutorial kein Netzwerkkurs sein soll, gehe ich nicht weiter darauf ein, warum ich den IP-Adressbereich so gewählt habe. Hier aber ein nützlicher [Link zum Thema "IP-Adressen"](https://de.wikipedia.org/wiki/IP-Adresse) sowie zum [Netzwerkrechner von Heise](https://www.heise.de/netze/tools/netzwerkrechner/). Außerdem findest du reichlich [Videos zum Thema "Netzwerk/ Subnetting" bei Youtube](https://www.youtube.com/results?search_query=netzwerk+subnetting).  
{{< /admonition >}}

Die weiteren Optionen unter "Advanced configuration" benötigen wir nicht. Mit einem Klick auf "Create the network" wird das Netzwerk erstellt und erscheint in unserer Netzwerkliste:

{{< image src="/img/posts/2023-02-23/portainer_bridge3.png" alt="Portainer - Networkslist" width="100%" >}}

## Bridge Netzwerk über stack/ docker-compose definieren

Natürlich lässt sich solch ein benutzerdefiniertes Netzwerk auch in [docker-compose](https://docs.docker.com/compose/compose-file/compose-file-v3/) definieren. In der Portainer Web UI nennt sich das Ganze "Stacks". Die Definition des oben manuell angelegten Netzwerks sieht dabei so aus:

```yaml
networks:
  my_bridge:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
          gateway: 172.20.0.1
          ip_range: 172.20.0.0/24
```

## Bonus: Bridge Netzwerk über die Konsole anlegen

Selbstverständlich können wir Netzwerke auch über die Kommandozeile unseres Docker Hosts anlegen. Dafür benötigen wir einen entsprechenden Zugang per SSH oder direkt auf die Konsole des Systems. 

Mit einem `docker network ls` (je nach Systemkonfiguration sind ggf. erweiterte Rechte über `sudo` erforderlich), lassen wir uns eine Liste der bereits vorhandenen Netzwerke anzeigen. 

```shell
$ docker network ls
NETWORK ID     NAME        DRIVER    SCOPE
9335f6cb157e   bridge      bridge    local
62d546b7d844   host        host      local
6735f2dd8b6d   my_bridge   bridge    local
8b720f638bc1   none        null      local
```

Details zu unserem bereits über Portainer erstellten Netzwerks `my_bridge` können wir per `docker network inspect my_bridge` anzeigen lassen.

```shell
$ docker network inspect my_bridge
[
    {
        "Name": "my_bridge",
        "Id": "6735f2dd8b6d27fab59382cf919b1a88e1eeeff6bcc24517074277329886a106",
        "Created": "2023-03-12T21:11:52.358474863+01:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.20.0.0/16",
                    "IPRange": "172.20.0.0/24",
                    "Gateway": "172.20.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

Ein einfaches, benutzerdefiniertes Bridge Netzwerk können wir jetzt einfach per `docker network create [Netzwerkname]` erstellen. Dabei wählt Docker die Parameter für Subnet, Gateway und Adressbereich (IP range) selbst. 

```shell
docker network create my_other_bridge
```

```shell
$ docker network ls
NETWORK ID     NAME              DRIVER    SCOPE
9335f6cb157e   bridge            bridge    local
62d546b7d844   host              host      local
6735f2dd8b6d   my_bridge         bridge    local
e0f4f0715651   my_other_bridge   bridge    local
8b720f638bc1   none              null      local

```

```shell
$ docker network inspect my_other_bridge
[
    {
        "Name": "my_other_bridge",
        "Id": "e0f4f071565141986e111a0b511149e7361c79ed78f6e80e8abd6ce7d91aef20",
        "Created": "2023-03-13T08:59:53.863603535+01:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.21.0.0/16",
                    "Gateway": "172.21.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```
Natürlich können wir die Parameter auch selbst mit geben. Hier ein Beispiel zum Erstellen unsere `my_bridge`-Netzwerks von oben:

```shell
docker network create \
  --subnet=172.20.0.0/16 \
  --ip-range=172.20.0.0/24 \
  --gateway=172.20.0.1 my_bridge
```
Das soll es zur Kommandozeile auch gewesen sein. In den [Docker Docs zu docker network create](https://docs.docker.com/engine/reference/commandline/network_create/) findest du weiterführende Informationen und Beispiele. 

---

Ich hoffe ich konnte dir mit diesem Beitrag einen kleinen Überblick über die Einrichtung eines benutzerdefinierten Docker Bridge Netzwerks mit Portainer geben.  
&nbsp;
Für Fragen und Feedback nutze gerne die Kommentarfunktion zu diesem Beitrag. 
&nbsp;
MfG,
André