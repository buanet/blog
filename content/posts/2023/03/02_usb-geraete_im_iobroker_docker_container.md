---
title: USB-Geräte im ioBroker Docker Container
subtitle: ''
description: 'Wie kann ich USB-Geräte im ioBroker Docker Container nutzen?'
summary: 'In diesem Tutorial zeige ich dir wie du ein USB-Gerät in deinen ioBroker Container durchreichen kannst um zum Beispiel einen Zigbee-Stick in ioBroker verwenden zu können...'
date: 2023-03-03T21:25:42+01:00
lastmod: ''
draft: false
featuredImage: /img/posts/2023-03-02/header.png

tags: []
categories: [Tutorials, ioBroker]

author: 'André (buanet)'

seo:
  image: ''
---

## Einleitung

In diesem Tutorial zeige ich dir wie du ein USB-Gerät in deinen ioBroker Container durchreichen kannst um zum Beispiel einen Zigbee-Stick in ioBroker verwenden zu können.

## Voraussetzungen

Folgendes setze ich voraus:
* Linux basierter Docker Host (NAS Systeme basieren in der Regel auf Linux)
* Zugriff auf die Kommandozeile des Docker Hosts
* Installierter und gestarteter Docker Dienst
* Eingerichteter ioBroker Docker Container

## USB-Gerät auf dem Host identifizieren

Damit wir das USB-Gerät in den Docker Container durchreichen können müssen wir erst einmal identifizieren unter welchem Pfad es auf unserem Docker Host zur Verfügung steht und angesprochen werden kann.
Unter Linux befinden sich die Geräte (devices) in der Regel im Verzeichnis `/dev`. Doch welches Gerät ist jetzt unser USB-Stick? 

Die einfachste Lösung das herauszufinden ist das Ausführen des Befehls `dmesg` unmittelbar nach dem Anstecken des USB-Geräts. Ziemlich am Ende der Ausgabe sollte sich erkennen lassen, dass ein USB-Gerät angesteckt worden ist.

Bei mir auf der DiskStation sieht das dann in etwa so aus:

{{< image src="/img/posts/2023-03-02/01_dmesg.png" alt="Ausgabe Kommandozeile dmesg" width="100%" >}}

An der Ausgabe ist zu erkennen, dass mein USB-Gerät als `ttyACM0` auf dem Host zur Verfügung steht. Der Pfad wäre dann damit `/dev/ttyACM0`.

Falls dieses Vorgehen bei dir nicht zum Erfolg führt, kannst du es auch mit dem Befehl `lsusb` versuchen. Damit bekommst du eine Liste aller USB-Geräte angezeigt.
Auf meiner DiskStation sieht das dann so aus:

{{< image src="/img/posts/2023-03-02/02_lsusb.png" alt="Ausgabe Kommandozeile lsusb" width="100%" >}}

Da ich weiß, das es sich neo meinem Zigbee-Stick um einen Stick von Texas Instruments handelt, kann ich der Ausgabe entnehmen, dass er an den USB-Port `usb2`, genauer sogar an `2-1` gesteckt wurde. Mit dem Befehl `ll /sys/class/tty | grep usb2` oder `ll /sys/class/tty | grep 2-1` kann ich nun ebenfalls ermitteln dass mein Stick unter dem Alias `ttyACM0` zu finden ist:

{{< image src="/img/posts/2023-03-02/03_ll.png" alt="Ausgabe Kommandozeile ll" width="100%" >}}

Der Pfad meines USB-Geräts auf dem Docker Host ist also `/dev/ttyACM0`.

### Best Practice: Nutze /dev/serial/by-id/...

Die beiden bisher genannten Wege zur Ermittlung des Pfades unter dem das USB-Gerät ansprechbar ist, sind Beispiele von meiner DiskStation. Auf anderen Linux Systemen, wie z.B. einem Raspberry Pi, haben wir in der Regel die Möglichkeit ein Gerät nicht nur über den Alias (z. B. `ttyACM0`), sondern auch `by-id` anzusprechen. Dazu kann man mit dem Befehl `ls -al /dev/serial/by-id/` eine Liste der Geräte ausgeben lassen, die sich per eindeutige ID ansprechen lassen. 

In der Ausgabe sieht dies dann so aus:

{{< image src="/img/posts/2023-03-02/03a_ls.png" alt="Ausgabe Kommandozeile ls" width="100%" >}}

Auch hier erkennen wir wieder meinen USB-Stick von Texas Instruments (`usb-Texas_Instruments…`). Zusätzlich sehen wir auch die Verlinkung zum Alias `ttyACM0`.
Wenngleich die Nutzung der ID des Geräts aufgrund der Länge komplizierter zu sein scheint, so hat diese Methode einen entscheidenden Vorteil. Während es nämlich möglich ist, dass bei einem Neustart den verwendeten USB-Geräten andere Aliase zugeordnet werden, so ist die ID eindeutig und bleibt damit immer gleich.
Wenn dein Linux Host also diese Variante unterstützt, empfiehlt es sich, vor Alle bei mehreren USB-Devices, den „by-id“-Pfad zu verwenden und nicht den Alias. 

Der Vollständigkeit halber sei gesagt, dass es natürlich auch noch andere Wege gibt, den Pfad des USB-Geräts zu ermitteln. Welche Möglichkeiten du hast hängt dabei natürlich auch von dem Verwendeten System ab. Google ist hier in der Regel eine gute Anlaufstelle für weitere Informationen.

## ioBroker Docker Container konfigurieren

Nachdem wir den Pfad unseres USB-Geräts ermitteln konnten, ist es nun Zeit das Gerät in den ioBroker Docker Container durchzureichen.

Dabei müssen wir gleich an zwei Stellen die Konfiguration unseres ioBroker Docker Containers anpassen. Zunächst reichen wir den ermittelten Pfad in den Docker Container durch. Dazu werfen wir beim Erstellen bzw, Bearbeiten des Containers über Portainer einen Blick auf die „Advanced container settings“, genauer in den Bereich „Runtime & Resources“. Dort gibt es einen Punkt „Devices“. Über „add device“ können wir ein Gerät hinzufügen

Unter „host“ ist der zuvor ermittelte Pfad des Geräts zu hinterlegen. Bei Verwendung des Alias-Pfads (also z.B. `/dev/ttyACM0`) empfehle ich das Gerät auch innerhalb des Container unter dem gleichen Pfad verfügbar zu machen um nicht durcheinander zu kommen. Das Ganze sieht dann in etwa so aus:

{{< image src="/img/posts/2023-03-02/04_portainer_devices.png" alt="Portainer Devices" width="100%" >}}

Für den Fall, dass wir den eindeutigen, langen „by-id“-Pfad verwenden wollen, empfiehlt es sich zumindest im Container einen Alias zu verwenden. Möglich wäre z.B. `/dev/serial/by-id/zigbee` für einen Zigbee-Stick.

Der zweite Teil der Konfiguration bezieht sich auf die Umgebungsvariable `USBDEVICES`. Um Sicher zu stellen, dass der ioBroker innerhalb des Containers über die notwendigen Rechte für den Zugriff auf das Gerät verfügt, setzen wir unter „Advanced container settings“ im Punkt „Env“ die Umgebungsvariable `USBDEVICES` und füllen den Wert mit dem Pfad des Gerätes innerhalb des Containers, also entweder `/dev/ttyACM0` oder `/dev/serial/by-id/zigbee`. Weitere Infos zur Verwendung der Umgebungsvariable findest du in der [Dokumentation zum Docker Image](https://docs.buanet.de/de/iobroker-docker-image/docs/#umgebungsvariablen-env).

Für mein Beispiel würde die Konfiguration also so aussshen: 

{{< image src="/img/posts/2023-03-02/05_env.png" alt="Portainer Umgebungsvariable" width="100%" >}}

Anschließend starten wir den Container.

## Überprüfung nach Start des Containers

Nach dem Start sollte das USB-Gerät nun innerhalb des Containers zur Verfügung stehen. Prüfen lässt sich das recht einfach über ein `ls -al /dev/*` bzw. `ls -al /dev/serial/by-id/*` auf der Kommandozeile des Containers.
Hier sollte nun das durchgereichte Gerät auftauchen und die folgenden Berechtigungseinstellungen besitzen:

{{< image src="/img/posts/2023-03-02/06_check.png" alt="Ausgabe Kommandozeile ls" width="100%" >}}

Das sollte es dann auch schon gewesen sein. Jetzt müsst ihr nur noch den entsprechenden Adapter installieren und euer Gerät in der Adapter-Konfiguration entsprechend angeben.

## Bonus: Durchreichen des USB-Geräts mit docker-compose (Portainer Stacks)

Selbstverständlich lässt sich die Konfiguration auch in docker-compose bzw. Portainer Stacks abbilden. Hier ein Beispiel des ioBroker Docker Containers aus [diesem Tutorial](/posts/2023/02/27_iobroker_unter_docker_auf_der_synology_diskstation/) mit dem USB-Gerät von oben:

```YAML
version: "3"
services:
  iobroker:
    container_name: iobroker
    image: buanet/iobroker:latest
    network_mode: host
    restart: always
    environment:
      - PACKAGES=nano
      - USBDEVICES=/dev/ttyACM0
    devices:
      - /dev/ttyACM0:/dev/ttyACM0
    volumes:
      - /volume1/docker/iobroker-data:/opt/iobroker
```

---

Für Fragen und Feedback nutze gerne die Kommentarfunktion zu diesem Beitrag. 
&nbsp;
MfG,
André