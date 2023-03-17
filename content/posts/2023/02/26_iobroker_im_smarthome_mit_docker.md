---
title: ioBroker im Smarthome mit Docker
subtitle: ''
description: ''
summary: ''
date: 2023-02-26T22:25:17+01:00
lastmod: ''
draft: true
featuredImage: /img/posts/2023-02-26_header.png

tags: []
categories: [Docker Images, ioBroker]

seo:
  image: ''

#comment:
#  enable: false
---

## Einleitung

Eine zentrale Rolle in meinem Smarthome mit Docker spielt ioBroker. In diesem Artikel will ich euch einen kurzen Überblick über ioBroker geben und zeigen, wie einfach und schnell ihr einen ioBroker Docker Container für einen ersten Test aufsetzen könnt.  

## Was ist ioBroker?

IoBroker ist eine in JavaScript geschriebene Open-Source-IoT-Plattform die Smarthome-Komponenten und Services verschiedener Hersteller verbindet. Mit Hilfe von Plugins (genannt: "Adapter") ist ioBroker in der Lage mit einer Vielzahl von IoT-Hardware und -Diensten über verschiedene Protokolle und APIs zu kommunizieren. Alle Daten werden in einer zentralen Datenbank gespeichert auf die alle Adapter zugreifen können. Damit ist es sehr einfach logische Verbindungen, Automatisierungsskripte und schöne Visualisierungen herstellerübergreifend aufzubauen.
Weitere Informationen zu ioBroker findest du unter [iobroker.net](https://www.iobroker.net).

## Was ist das ioBroker Docker Image

IoBroker ist nicht speziell für den Betrieb unter Docker entwickelt worden. Daher habe ich bereits 2017 damit begonnen ioBroker in ein Docker Image zu portieren. Seitdem hat sich das [Projekt](https://github.com/buanet/ioBroker.docker) stetig weiter entwickelt und trägt mittlerweile auch den Titel "Offizielles ioBroker Docker Image".
Weitere Informationen zum ioBroker Docker Image findest du auf meiner [Docs-Webseite](https://docs.buanet.de/de/iobroker-docker-image/).

## Wo finde ich weitere Infos? 

Bevor ich dir zeige, wie wir einfach und schnell einen ioBroker unter Docker starten hier noch einen kleine Liste mit Links zu weiterführenden Informationen, Bezugsquellen und Dokumentationen:

* [Informationen zu ioBroker](https://www.iobroker.net)
* [Offizielle ioBroker Dokumentation](https://www.iobroker.net/#de/documentation)
* [Informationen zum ioBroker Docker Image](https://docs.buanet.de/de/iobroker-docker-image/)
* [Offizielle ioBroker Docker Image Dokumentation](https://docs.buanet.de/iobroker-docker-image/docs/)
* [IoBroker Docker Image Source Code & Issues](https://github.com/buanet/ioBroker.docker)
* [IoBroker Docker Image im Docker Hub](https://hub.docker.com/r/buanet/iobroker)
* IoBroker Communities:
  * [ioBroker Forum](https://forum.iobroker.net/)
  * [Discord Channel](https://discord.gg/HwUCwsH)
  * [Telegram Group](https://t.me/+Xfjuou6-LztkOTBi)
  * [Facebook Group](https://www.facebook.com/groups/440499112958264)

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

Der ioBroker im Docker Container speichert alle Konfigurationsdaten unter `/opt/iobroker`. Damit diese Daten beim Löschen des Containers nicht verloren gehen, legt das Docker Image (ab v8.0.0) beim Erstellen des Containers ein [Standard-Docker-Volume](https://docs.docker.com/storage/volumes/) mit dem Namen `iobroker-data` an. Dieses Verhalten kannst du überschreiben indem du ein eigenes Volume oder einen Dateipfad an den Pfad `/opt/iobroker` in den Container mountest. Möchtest du ein Docker Volume nutzen, solltest du es vor dem Start des Container bereits via `docker volume create iobroker-data` erstellt haben.

Beim Start des Containers mountest du dein Volume oder Verzeichnis dann mit dieser Option: 

```bash
-v [dein_volume_oder_verzeichnis]:/opt/iobroker
```

In docker-compose sieht das dann so aus:

```bash
    volumes:
      - [dein_volume_oder_verzeichnis]:/opt/iobroker
```

### Per Kommandozeile erstellen



### Bonus: Mit Portainer erstellen

---

