---
title: 'Einrichtung eines MACVLAN Netzwerks über Portainer'
subtitle: ''
description: ''
date: 2023-02-23T22:31:15+01:00
lastmod: ''
draft: false
featuredImage: /img/posts/2023-02-23/header.png

comments: false

tags: [portainer, docker, container, netzwerk, macvlan]
categories: [Tutorials, Portainer]

seo:
  image: ''
---

## Einleitung

In machen Fällen benötigt ein Docker Container mehr Zugriff auf das heimische Netzwerk, als über ein Bridge Netzwerk möglich ist. Hier geht es zum Beispiel um [Broadcast](https://de.wikipedia.org/wiki/Broadcast) oder [Multicast Traffic](https://de.wikipedia.org/wiki/Multicast), der den Container über das Bridge Netzwerk schlicht nicht erreichen kann, aber von vielen Tools und Protokollen benötigt wird. In diesem Fall bietet sich die Verwendung eines MACVLAN Netzwerks an.
Hierbei bekommt der Container eine eigene (virtuelle) IP-Adresse im Netzwerk und ist damit von anderen Geräten vollständig erreichbar. Eine Freigabe einzelner Ports ist nicht mehr erforderlich. 

In diesem zeige ich dir, wie du über Portainer ein MACVLAN Netzwerk erstellst und worauf du achten musst. Am Beispiel meines Testnetzes werden wir zusammen ein Subnetz für das MACVLAN finden und entsprechend konfigurieren. 

## Voraussetzungen

Auch dieses Mal gilt es wieder Voraussetzungen zu erfüllen um das Tutorial erfolgreich umsetzen zu können. Hier eine kurze Liste: 

* Vorhandensein eines Docker Hosts (Linux) mit eingerichtetem Portainer Docker Container
* Grundverständnis zum Thema IP-Netzwerk im Allgemeinen (IP-Adressen und Bereiche, Subnetze, DHCP, Gateway)
* Grundverständnis zum Thema [Docker Netzwerk](https://docs.docker.com/network/)
* Kenntnisse des eigenen Netzwerks (DHCP-Bereiche, Gateways, usw.)
* Zugriff auf die Kommandozeile des Docker Hosts

## Adressbereich (IP range) ermitteln

Starten wir mit dem Ermitteln eines passenden Adressbereichs für unser MACVLAN. Dazu muss man wissen, dass Docker, im für das MACVLAN verwendeten Adressbereich, selbst IP-Adressen für Container vergeben kann, wenn diese mit dem Netzwerk verbunden werden. Ganz ähnlich wie es auch ein DHCP-Server tut wenn sich ein neues Gerät im Netzwerk meldet. Um hier später IP-Adresskonflikte im Heimnetz zu vermeiden, sollte sich daher der Adressbereich für das MACVLAN nicht mit dem Adressbereich deines DHCP überschneiden. 

Nehmen wir mein Test-Netzwerk als praktisches Beispiel. Ich verwende das Subnetz `192.168.11.0/24`. Ein Netzwerkbereich der 254 Adressen (`192.168.11.0 bis 192.168.11.254`) umfasst. Das Gateway ist die `192.168.11.1`. Mein DHCP vergibt IP-Adressen im Bereich von `192.168.11.200 bis 192.168.11.250`.
Vergleichbar ist dieses Netz von der Beschaffenheit her zum Beispiel mit den Standard Netzwerken die auch Heimnetz-Router wie die Fritzbox verwenden.

Um nun innerhalb dieses Subnetzes einen passenden Adressbereich zu finden, welches sich für das MACVLAN eignet, nutze ich immer wieder gerne den [Netzwerkrechner von Heise](https://www.heise.de/netze/tools/netzwerkrechner/). Dort gebe ich zunächst eine IP-Adresse aus dem Adressbereich ein, von dem ich denke, dass er sich für mein MACVLAN Netzwerk eignet und nicht mit dem DHCP kollidiert. In meinem Fall ist das zum Beispiel der Bereich um `192.168.11.120`.
Weiter geht es mit dem CIDR-Suffix (Definiert die Größe der Netzmaske). Für mein komplettes Netz ist das CIDR-Suffix `/24`. Je größer das Suffix wird, desto kleiner wird mein Netzwerkbereich. Dabei halbiert sich mit jedem Schritt die Anzahl der nutzbaren IP-Adressen im Netzwerkbereich (-2 für Netzadresse und Broadcast). Bedeutet für die Adressbereiche: 

```
/24 = 254 nutzbare IP-Adressen im Bereich
/25 = 126 nutzbare IP-Adressen im Bereich
/26 = 62 nutzbare IP-Adressen im Bereich
/27 = 30 nutzbare IP-Adressen im Bereich
/28 = 14 nutzbare IP-Adressen im Bereich
/29 = 6 nutzbare IP-Adressen im Bereich
/30 = 2 nutzbare IP-Adressen im Bereich
```
Mein Tip: Einfach mal verschiedene Werte im Rechner eingeben und schauen wie sich die Ergebnisse verändern. 
{{< admonition type=warning title="Info" open=false >}}
Kleine, ergänzende Info zum CIDR-Suffix und Spezialfall `/31`. Eigentlich wäre hier die Anzahl der nutzbaren Adressen = 0. Docker nutzt allerdings auch die "Netzadresse", sodass wir das MACVLAN auch mit dem Suffix `/31` definieren könnten. In diesem Fall wäre im Netz genau eine Adresse (nämlich die Netzadresse) für Container verfügbar.
{{< /admonition >}}

Da mir ein Netzbereich mit 6 IP-Adressen ausreicht, verwende ich das CIDR-Suffix `/29`und erhalte über den Rechner folgendes Ergebnis:

{{< image src="/img/posts/2023-02-23/portainer_macvlan1.png" alt="Netzwerkrechner" width="100%" >}}

Dieser Adressbereich umfasst nun insgesamt die IP-Adressen von `192.168.11..120 bis 192.168.11..127`, während ich die Adressen von `192.168.11..121 bis 192.168.11.126` für Docker Container verwenden könnte. 

{{< admonition type=warning title="Hinweis" open=true >}}
Dieses Tutorial ist kein Netzwerkkurs. :nerd_face: Die Auswahl des Netzbereiches ist sehr oberflächlich gehalten, funktioniert aber für diesen Anwendungsfall. Für weiteren Infos hier ein [Link zum Thema "IP-Adressen"](https://de.wikipedia.org/wiki/IP-Adresse). Außerdem findest du reichlich [Videos zum Thema "Netzwerk/ Subnetting" bei Youtube](https://www.youtube.com/results?search_query=netzwerk+subnetting).
{{< /admonition >}}

Damit haben wir unseren Adressbereich ermittelt und können die gewonnenen Informationen in unserer MACVLAN-Konfiguration verwenden.

## Parent network card ermitteln

Um ein MACVLAN anlegen zu können, muss dieses an einen Netzwerkadapter im Docker Host gebunden werden. Damit wir dies bei der Konfiguration über die Portainer erledigen können, müssen wir zunächst die Bezeichnung des Adapters ermitteln. Dies geht am einfachsten über die Kommandozeile.
Über `ifconfig` oder `ip addr show` können wir uns eine Auflistung der Netzwerkkonfiguration anzeigen lassen.

```shell
$ ifconfig
docker0   Link encap:Ethernet  HWaddr 02:42:32:39:3E:55
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
          inet6 addr: fe80::42:32ff:fe39:3e55/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:2391 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2657 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:3308725 (3.1 MiB)  TX bytes:872522 (852.0 KiB)

docker-67 Link encap:Ethernet  HWaddr 02:42:33:D2:74:92
          inet addr:172.20.0.1  Bcast:172.20.255.255  Mask:255.255.0.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

docker66f Link encap:Ethernet  HWaddr 0E:1B:75:00:0A:ED
          inet6 addr: fe80::c1b:75ff:fe00:aed/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:2391 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2662 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:3342199 (3.1 MiB)  TX bytes:872960 (852.5 KiB)

eth0      Link encap:Ethernet  HWaddr EE:10:EC:B0:CB:20
          inet addr:192.168.11.200  Bcast:192.168.11.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:10802 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6910 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:2018717 (1.9 MiB)  TX bytes:15025239 (14.3 MiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:68 errors:0 dropped:0 overruns:0 frame:0
          TX packets:68 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:7279 (7.1 KiB)  TX bytes:7279 (7.1 KiB)

```

```shell
$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether ee:10:ec:b0:cb:20 brd ff:ff:ff:ff:ff:ff
    inet 192.168.11.200/24 brd 192.168.11.255 scope global eth0
       valid_lft forever preferred_lft forever
3: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1
    link/sit 0.0.0.0 brd 0.0.0.0
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 02:42:32:39:3e:55 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:32ff:fe39:3e55/64 scope link
       valid_lft forever preferred_lft forever
5: docker-6735f2dd: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 02:42:33:d2:74:92 brd ff:ff:ff:ff:ff:ff
    inet 172.20.0.1/16 brd 172.20.255.255 scope global docker-6735f2dd
       valid_lft forever preferred_lft forever
7: docker66fe1a9@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 0e:1b:75:00:0a:ed brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::c1b:75ff:fe00:aed/64 scope link
       valid_lft forever preferred_lft forever
```

In beiden Ausgaben finde ich die IP-Adresse `192.168.11.200` meines Docker Hosts. Zugewiesen ist diese der Netzwerkinterface `eth0`. Mit dieser Info im Gepäck können wir nun das MACVLAN anlegen. 

## MACVLAN über UI anlegen

Das Anlegen eines MACVLAN über Portainer erfolgt in zwei Schritten. Zunächst müssen wir eine MACVLAN "Configuration" anlegen, bevor wir in einem weiteren Schritt eine MACVLAN "Creation" erstellen können. Fangen wir also mit der "Configuration" an.  

### "Configuration" erstellen

Los geht es natürlich wieder im Bereich "Networks" in der Portainer Weboberfläche mit einem Klick auf "+ Add network". 

{{< image src="/img/posts/2023-02-23/portainer_macvlan2.png" alt="Portainer - Network list" width="100%" >}}

Wir vergeben einen aussagekräftigen Namen, in meinem Fall wähle ich `my_macvlan_conf`. Als Driver wählen wir `macvlan`. An dieser Stelle erscheint dann auch der Hinweis auf "Configuration" und "Creation". Wir wählen den Button für "Contiguration". 

{{< image src="/img/posts/2023-02-23/portainer_macvlan3.png" alt="Portainer - Create network" width="100%" >}}

Unter "Parent network card" geben wir das ermittelte Netzwerkinterface `eth0` an.
Die "IPv4 Network configuration" füllen wir mit den Daten zu unserem Subnetz und dem ausgewählten Adressbereich aus. In meinem Fall ist das das Subnetz (Heimnetz) `192.168.11.0/24`. Das Gateway (mein Router) darin hat die `192.168.0.1`. Als IP-Adressbereich habe ich `192.168.11.120/29`ermittelt.  
Weitere Optionen müssen wir in diesem Fall nicht konfigurieren. 

{{< admonition type=tip title="Tip" open=true >}}
Sollte sich im ausgewählten Adressbereich bereits ein Netzwerkgerät befinden, also eine oder mehrere Adressen nicht mehr verfügbar sein, so kannst du bei der Erstellung über "+ Add exclude IP" Adressen definieren, die Docker aus dem Bereich nicht verwenden darf.
{{< /admonition >}}

Ein Klick auf "Create the network" erstellt die Konfiguration für unser neues MACVLAN Netzwerk. 

### "Creation" erstellen

Wir befinden uns wieder im Bereich "Networks" des Portainer Web interfaces. Um die "Creation" zu erstellen, klicken wir nochmal auf "+ Add network". 

{{< image src="/img/posts/2023-02-23/portainer_macvlan4.png" alt="Portainer - Network list" width="100%" >}}

Wir vergeben einen aussagekräftigen Namen für die Creation, in meinem Fall wähle ich `my_macvlan`. Als Driver wählen wir `macvlan`. Anders als zuvor wählen wir nun aber den Button "Creation".

{{< image src="/img/posts/2023-02-23/portainer_macvlan5.png" alt="Portainer - Create network" width="100%" >}}

Aus dem Dropdown "Configuration" wählen wir unsere Configuration. In meinem Fall `my_macvlan_conf`.
Weitere Einstellungen müssen wir nicht konfigurieren. Ein Klick auf "Create the network" erstellt unser MACVLAN.  

## MACVLAN über stack/ docker-compose definieren

Natürlich lässt sich ein MACVLAN auch in [docker-compose](https://docs.docker.com/compose/compose-file/compose-file-v3/) definieren. In der Portainer Web UI nennt sich das Ganze "Stacks". Die Definition des oben manuell angelegten MACVLAN Netzwerks sieht dabei so aus:

```yaml
networks:
  my_macvlan:
    driver: macvlan
    driver_opts:
      parent: eth0
    ipam:
      config:
        - subnet: 192.168.11.0/24
          gateway: 192.168.11.1
          ip_range: 192.168.11.120/29
```

## Bonus: MACVLAN über die Konsole anlegen

Das MACVLAN kann selbstverständlich auch über die Kommandozeile des Docker Hosts angelegt werden. Dafür benötigen wir einen entsprechenden Zugang per SSH oder direkt auf die Konsole des Systems. 

Mit einem `docker network ls` (je nach Systemkonfiguration sind ggf. erweiterte Rechte über `sudo` erforderlich), lassen wir uns eine Liste der bereits vorhandenen Netzwerke anzeigen. 

```shell
$ docker network ls
NETWORK ID     NAME        DRIVER    SCOPE
9335f6cb157e   bridge      bridge    local
62d546b7d844   host        host      local
6735f2dd8b6d   my_bridge   bridge    local
8b720f638bc1   none        null      local
```

Um ein MACVLAN Netzwerk wie oben beschrieben zu erstellen, reicht ein einziges Kommando in dem wir alle nötigen Parameter einfach mitgeben.

```shell
 docker network create -d macvlan \
  --subnet=192.168.11.0/24 \
  --ip-range=192.168.11.120/29 \
  --gateway=192.168.11.1 \
  -o parent=eth0 my_macvlan
```

Die Details zum Netzwerk lassen sich anschließend per `docker network inspect my_macvlan` auslesen. 

```shell
$ docker network inspect my_macvlan
[
    {
        "Name": "my_macvlan",
        "Id": "77dc5f025355da453a3e6a781700bda078bdef596637a415344db99e578df56b",
        "Created": "2023-03-13T14:54:21.688669799+01:00",
        "Scope": "local",
        "Driver": "macvlan",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.11.0/24",
                    "IPRange": "192.168.11.120/29",
                    "Gateway": "192.168.11.1"
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
        "Options": {
            "parent": "eth0"
        },
        "Labels": {}
    }
]
```

---

MfG,
André
