---
title: ioBroker unter Docker auf der Synology DiskStation
subtitle: ''
description: 'Ein Tutorial zur Einrichtung des ioBroker Docker Images auf der Synology DiskStation'
summary: 'In diesem Tutorial zeige ich dir, wie du mit Hilfe von Docker und Portainer das ioBroker Docker Image auf deiner Synology DiskStation installierst...'
date: 2023-02-27T21:50:05+01:00
lastmod: ''
draft: false
featuredImage: /img/posts/2023-02-27/header.png

tags: []
categories: [Tutorials, ioBroker, DiskStation]

author: 'André (buanet)'

seo:
  image: ''

---

## Einleitung

In diesem Tutorial zeige ich dir, wie du mit Hilfe von Docker und Portainer das ioBroker Docker Image auf deiner Synology DiskStation installierst.
Neben den verlinkten Informationen zu Docker und Portainer empfehle ich dir vorab einen Blick in meinen [letzten Beitrag](/posts/2023/02/26_iobroker_im_smarthome_mit_docker/) zu werfen. Hier habe ich bereits einmal die Grundlagen zum ioBroker Docker Image erläutert.

## Voraussetzungen

Wie immer starte ich mit einer kurzen Liste der Voraussetzungen die zum Gelingen des Tutorials erfüllt sein sollten. 

* [Synology DiskStation mit installiertem Docker Paket](https://www.synology.com/de-de/dsm/packages/Docker)
* [Synology DiskStation Manager (DSM) in Version 6 oder 7](https://www.synology.com/de-de/dsm)
* [Portainer zur Verwaltung des Docker Dienstes](/posts/2023/02/12_portainer_auf_der_synology_diskstation/)
* Internetzugang

## Vorbereitungen

### Persistente Datenspeicherung

Bevor wir den ioBroker Docker Container anlegen, sollten wir uns Gedanken um den Speicherort der ioBroker Konfigurationsdaten machen. Um später einen einfachen Zugriff auf die Daten zu haben, empfehle ich auf der DiskStation das Anlegen eines gemeinsamen Ordners `docker` auf den wir dann über die File Station in der Web UI unserer DiskStation zugreifen können. Solltest du bereits den Portainer nach meiner Anleitung installiert haben, ist dieses Verzeichnis vermutlich schon vorhanden und wir brauchen nur noch ein separates Unterverzeichnis für den ioBroker anlegen. Bei der Benennung wäre mein Vorschlag hier (analog zum Portainer Ordner) `iobroker-data`.

{{< image src="/img/posts/2023-02-27/iobroker_synology1.png" alt="ioBroker Synology Folder" width="100%" >}}

### Überlegungen zum Netzwerk

Das Thema Docker und Netzwerk ist immer wieder ein Knackpunkt im Smarthome mit Docker. Prinzipiell reden wir hier über 3 mögliche Kandidaten: Bridge, Host und MACVLAN. Dabei hat jedes Netzwerk so seine Vor- und Nachteile. Im folgenden gebe ich einen wirklich kurzen, stark vereinfachten Überblick in Bezug auf ioBroker als kleine Entscheidungshilfe.

#### Bridge

Beim Bridge Netzwerk befindet sich der ioBroker Docker Container in einem abgeschotteten, virtuellen Netzwerk. Damit er dennoch mit der Außenwelt (deinem Heimnetzwerk) kommunizieren kann, müssen über das Bridge Netzwerk manuell alle Ports freigegeben werden, die für die Kommunikation benötigt werden. Je nach verwendeten ioBroker Adaptern kann das eine relativ lange Liste sein. Außerdem gibt es Einschränkungen bei Adaptern die z.B. eine Discovery-Funktion bereitstellen bzw. voraussetzen. Da die Kommunikation mit dem ioBroker nur über die freigegeben Ports möglich ist, gelangen [Broad-](https://de.wikipedia.org/wiki/Broadcast) oder [Multicast](https://de.wikipedia.org/wiki/Multicast)-Pakete nicht zum ioBroker und der betreffende Adapter kann ggf. nicht benutzt werden.

#### Host

Beim Host Netzwerk erhält der ioBroker Docker Container vollen Zugriff auf die Netzwerkschittstelle(n) der DiskStation. Er kümmert sich selbstständig um die zu öffnenden Ports und lauscht so auch auf alle anderen Pakete die die Netzwerkschnittstelle der DiskStation erreichen. Allerdings kann es hierbei zu Portkonflikten kommen. Dies geschieht, wenn der ioBroker versucht für sich oder einen seiner Adapter einen Port zu öffnen, den vielleicht schon der DiskStation Manager oder einer seiner Services verwendet. 

#### MACVLAN

Beim MACVLAN erhält der ioBroker Docker Container eine virtuelle IP-Adresse, z.B. aus dem Netzwerk Bereich in dem sich auch deine DiskStation befindet. Dadurch erscheint der Container im Netzwerk wie ein eigenständiges Gerät mit IP-Adresse und Hostnamen. Er kümmert sich selbstständig um die Freigabe von Ports. Durch die eigene IP-Adresse besteht keine Gefahr von Portkonflikten mit der DiskStation. Allerdings setzt die Konfiguration eines MACVLAN ein gewisses Maß an Wissen zum Thema Netzwerke voraus. 

#### Kurzes Fazit

Zusammengefasst lässt sich dazu also sagen:

Mit Bridge macht man nichts kaputt, es ist allerdings aufwändiger zu konfigurieren und beinhaltet teils unangenehme Einschränkungen.

Das Host Netzwerk ist einfach konfiguriert, gilt aber, aufgrund des weitreichenden Zugriffs auf die Netzwerkschnittstelle, als die potentiell unsicherste Lösung.

MACVLAN kommt bei all dem noch am Besten weg, ist aber sicher nicht für jeden "mal eben so" zu konfigurieren.

Kurzum: Für Einsteiger empfehle ich Host, der fortgeschrittene(re) User wagt sich an das MACVLAN. Zum "Ausprobieren" von ioBroker oder für die Kommunikation zu anderen Containern eignet sich das Bridge Netzwerk.

Weiterführende Infos zum Thema Netzwerke in Docker findest du in den [Docker Docs](https://docs.docker.com/network/). Eine Anleitung zum Einrichten eines benutzerdefinierten Bridge Netzwerks in meinem Tutorial [Einrichtung eines benutzerdefinierten Bridge Netzwerks mit Portainer](/posts/2023/02/23_einrichtung_eines_benutzerdefinierten_bridge_netzwerks_mit_portainer/). Eine Anleitung zur Einrichtung eines MCVLAN in meinem Tutorial [Einrichtung eines MACVLAN Netzwerks mit Portainer](/posts/2023/02/23_einrichtung_eines_macvlan_netzwerks_mit_portainer/).  

### Übernahme von Konfigurationsdaten

Solltest du ioBroker bereits auf einer anderen Platform betreiben, so kannst du per Backup & Restore deine Daten vom Altsystem mitnehmen. Da der Restore erst nach dem Erstellen des Containers erfolgt, kannst du das Tutorial einfach bis zum Ende durcharbeiten und dich erst wenn der ioBroker in der Grundinstallation läuft um den Restore kümmern. Von einem kompletten Umzug des ioBroker Datenverzeichnisses (`/opt/iobroker`) zu Docker würde ich jedoch abraten. Das ist zwar theoretisch möglich, birgt aber viele kleine mögliche Fehlerquellen. Außerdem würdest du so auch alle Altlasten, wie temporäre Dateien, mit schleppen. 
Weitere Details zu Backup & Restore findest du in der [offiziellen ioBroker Dokumentation](https://www.iobroker.net/#de/documentation) sowie in der [Doku zum Docker Image](https://docs.buanet.de/de/iobroker-docker-image/docs/).

## Container mit Portainer erstellen

Nachdem alles Vorbereitet ist, können wir mit dem Erstellen des Docker Containers über Portainer beginnen. Dazu rufen wir mit einem Klick auf den Button "+ Add container" im Menüpunkt "Containers" den Dialog für die Erstellung eines neuen Containers auf.

{{< image src="/img/posts/2023-02-27/iobroker_synology2.png" alt="ioBroker Synology Portainer Create" width="100%" >}}

Wir vergeben einen Namen für unseren Container, in meinem Fall `iobroker` und füllen das Feld "Image" mit dem Namen und Tag des Images, welches wir verwenden wollen. In diesem Fall verwende ich `buanet/iobroker:latest`.

Wenn wir vor hätten den Container mit einem Bridge Netzwerk auszuführen, dann bestünde an dieser Stelle, unter "Manual network port publishing", die Möglichkeit das entsprechende Port Mapping durchzuführen. Da ich meinen Container allerdings mit der Host Netzwerk Option erstellen werde, kann ich diese Einstellung ignorieren. 

{{< admonition type=info title="Hinweis" open=true >}}
Warum der Tag `latest` eigentlich keine gute Wahl für ein Produktivsystem ist, kannst du [hier](https://docs.buanet.de/de/iobroker-docker-image/docs/#vermeide-latest-tag) nachlesen. 
{{< /admonition >}}

Am unteren Ende des Dialogs finden wir die "Advanced container settings". In der Registerkarte "Volumes" mounten wir nun unser Verzeichnis, welches wir in der File Station angelegt haben. Zunächst tragen wir den Pfad innerhalb des Containers (`/opt/iobroker`) ein und wählen "Bind". Daraufhin können wir den Pfad unseres Ordners auf der DiskStation ausfüllen. In meinem Beispiel ist dies `/volume1/docker/iobroker-data`. 

{{< image src="/img/posts/2023-02-27/iobroker_synology3.png" alt="ioBroker Synology Portainer Volumes" width="100%" >}}

{{< admonition type=info title="Hinweis" open=true >}}
Wenn du in deiner DiskStation mehrere Volumes eingerichtet hast und dir sich nicht sicher bist, ob dein Verzeichnis auch unter `/volume1` liegt, kannst du den Pfad auch aus den Eigenschaften des Ordners (Rechtsklick > Eigenschaften) in der File Station kopieren.
{{< /admonition >}}

In der Registerkarte "Network" wählen wir als Netzwerk `host`.

{{< image src="/img/posts/2023-02-27/iobroker_synology4.png" alt="ioBroker Synology Portainer Network" width="100%" >}}

Die Angabe eines Hostnamen können wir uns sparen, da durch die verwendete Netzwerkoption automatisch der Hostname der DiskStation verwendet wird. Für den Fall dass du hier ein bereits vorbereitetes [MACVLAN](/posts/2023/02/23_einrichtung_eines_macvlan_netzwerks_mit_portainer/) oder [Bridge Netzwerk](/posts/2023/02/23_einrichtung_eines_benutzerdefinierten_bridge_netzwerks_mit_portainer/) auswählst empfiehlt sich die Angabe eines Hostnamens, da dieser in ioBroker auch für die Instanz verwendet wird. 

Weiter geht es in der Registerkarte ENV. Hier haben wir die Möglichkeit Konfigurationsoptionen (Umgebungsvariablen) für unseren ioBroker Docker Container hinzuzufügen. Eine Lister der verfügbaren Optionen findest du in der [Dokumentation zum ioBroker Docker Image](https://docs.buanet.de/de/iobroker-docker-image/docs/#umgebungsvariablen-env).

{{< image src="/img/posts/2023-02-27/iobroker_synology5.png" alt="ioBroker Synology Portainer ENV" width="100%" >}}

Exemplarisch habe ich hier einmal die Umgebungsvariable `PACKAGES` hinzugefügt und mit dem Wert `nano` belegt. Diese sorgt nun dafür, dass beim Start des Containers das Paket `nano`, welches nicht im Docker Image enthalten ist, installiert wird. Diese Option ist vor allem für ioBroker Adapter interessant, die zusätzliche Pakete erfordern. Infos dazu, welche Pakete ggf. benötigt werden, findest du in der jeweiligen Adapterdokumentation.

Weiter geht es in der Registerkarte "Restart policy". 

{{< image src="/img/posts/2023-02-27/iobroker_synology6.png" alt="ioBroker Synology Portainer Restart Policy" width="100%" >}}

Hier wählen wir zu guter Letzt die Option `Always`. Diese sorgt dafür, dass der Container im Fehlerfall automatisch neu gestartet wird. 

Damit haben wir alle notwendigen Konfigurationen getätigt und können mit einem Klick auf "Deploy the container" oberhalb der "Advanced container settings" die Erstellung anstossen. 

{{< admonition type=info title="Hinweis" open=true >}}
Die Erstellung des Containers kann mitunter ein wenig Zeit in Anspruch nehmen, da Portainer im Hintergrund zunächst das ioBroker Docker Image herunter laden muss. Hab einfach ein wenig Geduld. Solange keine Fehlermeldung erscheint ist alles in Ordnung.
{{< /admonition >}}

Im Anschluss landen wir wieder in der Container Übersicht ("Container list"). Hier sollte nun unser iobroker auftauchen. 

{{< image src="/img/posts/2023-02-27/iobroker_synology7.png" alt="ioBroker Synology Portainer Container List" width="100%" >}}

Ein Klick auf das erste Symbol in der Spalte "Quick Actions" öffnet die Logausgabe. Hier können wir den Startvorgang des Containers beobachten. Die Ausgabe sollte in etwa so aussehen. 

```bash
--------------------------------------------------------------------------------
-------------------------     2023-03-28 21:19:46      -------------------------
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
-----                    hostname:            vm-dsm7-test                 -----
-----                                                                      -----
-----                          Version Information                         -----
-----                    image:               v8.0.0                       -----
-----                    build:               2023-03-20T21:14:35+00:00    -----
-----                    node:                v18.15.0                     -----
-----                    npm:                 9.5.0                        -----
-----                                                                      -----
-----                        Environment Variables                         -----
-----                    PACKAGES:            nano                         -----
-----                    SETGID:              1000                         -----
-----                    SETUID:              1000                         -----
--------------------------------------------------------------------------------
 
--------------------------------------------------------------------------------
-----                   Step 1 of 5: Preparing container                   -----
--------------------------------------------------------------------------------
 
Updating Linux packages on first run... Done.
 
PACKAGES is set. Installing the following additional Linux packages: nano
 
nano is not installed. Installing... Done.
Registering maintenance script as command... Done.
 
--------------------------------------------------------------------------------
-----             Step 2 of 5: Detecting ioBroker installation             -----
--------------------------------------------------------------------------------
 
There is no data detected in /opt/iobroker.
Restoring initial ioBroker installation... Done.
 
--------------------------------------------------------------------------------
-----             Step 3 of 5: Checking ioBroker installation              -----
--------------------------------------------------------------------------------
 
(Re)setting permissions (This might take a while! Please be patient!)... Done.
 
Fixing "sudo-bug" by replacing sudo with gosu... Done.
 
Initializing a fresh installation of ioBroker... Done.
 
Hostname in ioBroker does not match the hostname of this container.
Updating hostname to "vm-dsm7-test"... The host for instance "system.adapter.admin.0" was changed from "buildkitsandbox" to "vm-dsm7-test".
The host for instance "system.adapter.discovery.0" was changed from "buildkitsandbox" to "vm-dsm7-test".
The host for instance "system.adapter.backitup.0" was changed from "buildkitsandbox" to "vm-dsm7-test".
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
host.vm-dsm7-test check instance "system.adapter.admin.0" for host "vm-dsm7-test"
host.vm-dsm7-test check instance "system.adapter.discovery.0" for host "vm-dsm7-test"
host.vm-dsm7-test check instance "system.adapter.backitup.0" for host "vm-dsm7-test"
```

Ist ioBroker erfolgreich gestartet, so können wir über die URL nach folgendem Schema auf die ioBroker Admin UI zugreifen:

`http://[Hostname_oder_IP_des_Docker_Host]:8081`

Es sollte der ioBroker Setup Assistent für die Ersteinrichtung erscheinen. 

{{< image src="/img/posts/2023-02-26/iobroker1.png" alt="ioBroker Setup Assistent" width="100%" >}}

Unser ioBroker Docker Container ist damit startklar. 

## Bonus: Container mit Portainer Stacks erstellen

Zum [Verwenden von Stacks in Portainer](/posts/2023/02/15_verwenden_von_stacks_in_portainer/) habe ich ja bereits ein Tutorial gemacht. Als Stackfile würde das Beispiel von oben dann so aussehen:

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
    volumes:
      - /volume1/docker/iobroker-data:/opt/iobroker
```

---

Für Fragen und Feedback nutze gerne die Kommentarfunktion zu diesem Beitrag. 
&nbsp;
MfG,
André