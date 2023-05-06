---
title: Verwenden von Stacks (aka docker-compose) in Portainer
subtitle: ''
description: 'Ein Tutorial zum Arbeiten mit Stacks (docker-compose) in Portainer'
summary: 'Stacks nennt sich eine beliebte Funktion in Portainer, mit der es möglich ist über eine einzige "Konfigurationsdatei" (Stackfile) einen oder mehrere Container, samt Netzwerken und Volumes, erstellen und entsprechend konfigurieren zu lassen. Das Ganze basiert dabei auf "docker-compose"...'
date: 2023-02-15T22:21:55+01:00
lastmod: ''
draft: false
featuredImage: /img/posts/2023-02-15/header.png

tags: []
categories: [Tutorials, Portainer]

author: 'André (buanet)'

seo:
  image: ''
---

## Einleitung

Stacks nennt sich eine beliebte Funktion in Portainer, mit der es möglich ist über eine einzige "Konfigurationsdatei" (Stackfile) einen oder mehrere Container, samt Netzwerken und Volumes, erstellen und entsprechend konfigurieren zu lassen. Das Ganze basiert dabei auf ["docker-compose"](https://docs.docker.com/compose/), ist aber durch die vollständige Integration in die Portainer Weboberfläche für den Durchschnittsuser deutlich komfortabler zu bedienen. 

In diesem Tutorial möchte ich dir ein paar Grundlagen zur Verwendung von Portainer Stacks vermitteln und dir damit, anhand eines Beispiels, ein Tool an die Hand geben, dass du sicher gut in deinem Smarthome mit Docker gebrauchen kannst.

## Voraussetzungen

Fangen wir wie üblich mit den Voraussetzungen für dieses Tutorial an. 

* Vorhandensein eines auf Linux basierenden [Docker Hosts oder NAS mit Docker](/posts/2023/02/01_grundlagen/)
* [Portainer zur Verwaltung des Docker Dienstes](/posts/2023/02/10_portainer_zur_verwaltung_des_docker_dienstes)
* Grundverständnis zum Thema [Docker](https://docs.docker.com/get-started/)

## Grundlegende Funktionsweise

Wie eingangs bereits erwähnt, bieten uns Stacks unter Portainer die Möglichkeit einen oder mehrere Container samt Netzwerken und Volumes in einer Konfigurationsdatei, dem Stackfile, zu definieren und anschließend durch den Docker Dienst, voll automatisiert, erstellen zu lassen. Das Stackfile wird dabei in [YAML (Yet Another Markup Language)](https://de.wikipedia.org/wiki/YAML) verfasst. YAML ist eine einfache [Auszeichnungssprache](https://de.wikipedia.org/wiki/Auszeichnungssprache) mit deren Hilfe wir in unserem Fall dem Docker Dienst allerlei Konfigurationsparameter bereitstellen können, aus denen dieser dann unsere Docker Container, Netzwerke oder Volumes erstellt.

Für ein funktionierendes Stackfile ist es wichtig, dass es bestimmten Regeln im Rahmen der YAML Formatierung folgt. Dies sind zum Beispiel Einrückungen, Trennzeichen und Anführungszeichen. Aber keine Angst, der in der Weboberfläche von Portainer eingebettete Webeditor hilft uns dabei, die Formatierungsregeln einzuhalten.

Darüber hinaus ist es natürlich wichtig, dass wir in unserem Stackfile nur Parameter definieren, die der Docker Dienst auch verstehen und umsetzen kann. Hierzu stellt Docker mit der ["compose file reference"](https://docs.docker.com/compose/compose-file/compose-file-v3/) eine ausführlichen Leitfaden zur Verfügung, welcher die verfügbaren Parameter auflistet und erläutert.

Ein sehr einfaches Beispiel für ein Stackfile könnte zum Beispiel so aussehen:

```YAML
version: "3"            # Version der compose file reference
services:               # Hier beginnt die definition der Services
  hello_world:          # Name des ersten Service
    image: hello-world  # Verwendetes Docker Image
```

Dabei wird dann allerdings lediglich ein einzelner Container aus dem "hello-world"-Docker Image erstellt, dessen einzige Aufgabe darin besteht, die Funktion des Docker Dienstes zu testen und eine entsprechende Ausgabe im Log zu erzeugen. 
Weitere Infos zum Testimage findet ihr im Docker Hub: https://hub.docker.com/_/hello-world

## Stacks in der Weboberfläche

Werfen wir nun zunächst einem Blick auf die Weboberfläche von Portainer. Unter dem Menüpunkt "Stacks" finden wir die Übersicht der von uns definierten Stacks.

{{< image src="/img/posts/2023-02-15/portainer_stacks1.png" alt="Portainer Stacks" width="100%" >}}

Mit einem Klick auf "+ Add Stack" gelangen wir in den Editor zum erstellen eines neuen Stackfiles.

{{< image src="/img/posts/2023-02-15/portainer_stacks2.png" alt="Portainer Stack Editor" width="100%" >}}

Hier können wir einen Namen festlegen und die "Build method" auswählen. Ich werde mich hier zunächst nur auf den Webeditor beschränken. Es ist aber natürlich auch möglich fertige Stackfiles hoch zu laden bzw. direkt aus einem Repository zu laden. Alternativ kann auch ein zuvor angelegtes Custom Template verwendet werden.

Ausserdem befinden sich unterhalb des Editors noch weitere Optionen. Diese lassen wir fürs erste allerdings ebenfalls unbeachtet. Einzig der Button "Deploy the stack" ist für uns relevant. Ein Klick hierauf startet mit der Erstellung des Stacks und seiner Container.

## Stacks am Beispiel von ioBroker und Redis

Zeit für ein echtes (aber etwas vereinfachtes) Beispiel. Wir wollen einen ersten Stack, besehend aus zwei Containern und einem gemeinsamen Bridge Netzwerk erstellen. Dabei werden wir das Bridge Netzwerk natürlich nicht manuell anlegen, sondern dies ebenfalls im Stackfile definieren und automatisch erstellen lassen. Außerdem speichern wir die Daten der Container in zwei, ebenfalls im Stack definierten, Volumes. Dazu habe ich das folgende Stackfile vorbereitet und mit kurzen Kommentaren versehen. 

```YAML
version: "3"                            # Version der compose file reference
services:                               # Hier beginnt die Definition der Services (Container)
  ####### ioBroker #######
  iobroker:                             # Name des ersten Service
    container_name: iobroker            # Container Name
    image: buanet/iobroker:latest-v8    # Docker Image und Tag
    hostname: iobroker                  # Hostname (als Instanz in ioBroker)
    restart: always                     # Restart policy
    networks:                           # Verbundene Netzwerke
      bridge:                           # Name des Netzwerks wie weiter unten definiert
    ports:                              # Ports
      - "8081:8081"                     # Durchgereichter Port für Admin
    environment:                        # Umgebungsvariablen
      - IOB_STATESDB_HOST=redis         # Host für die Datenbankverbindung der States DB
      - IOB_STATESDB_PORT=6379          # Port für die Datenbankverbindung der States DB
      - IOB_STATESDB_TYPE=redis         # Typ für die Datenbankverbindung der States DB
    volumes:                            # Verbundene Volumes
      - iobroker-data:/opt/iobroker     # Volume wue weiter unten definiert
    depends_on:                         # Abhängigkeit des Containers
      - redis
  ####### redis #######
  redis:                                # Name des zweiten Service
    container_name: redis               # Container Name
    image: redis:latest                 # Docker Image und Tag
    restart: always                     # Restart policy
    networks:                           # Verbundene Netzwerke
      bridge:                           # Name des Netzwerks wie weiter unten definiert
    volumes:                            # Verbundene Volumes
      - redis-data:/data                # Volume wue weiter unten definiert

####### Netzwerke #######
networks:                               # Definition der zu erstellenden Netzwerke
  bridge:                               # Name des Netzwerks
    driver: bridge                      # Typ/ Treiber des Netzwerks

####### Volumes #######
volumes:                                # Definition der zu erstellenden Volumes
  iobroker-data:                        # Name des Volumes 
  redis-data:                           # Name des Volumes
```

Wir erstellen nun also einen neuen Stack (ich habe als Namen iobroker gewählt) und kopieren das vorbereitete Stackfile in den Editor. Sofern der Editor kein Syntaxfehler feststellt, starten wir die Erstellung des Stacks mit dem Button "Deploy the stack".

{{< admonition type=info title="Habe etwas Geduld..." open=false >}}
Da beim Erstellen des Stacks in der Regel noch die Docker Images aus dem Internet herunter geladen werden müssen, kann es schonmal sein, dass dieser Prozess ein wenig Zeit in Anspruch nimmt. habe also ein wenig Geduld. Solange keine Fehlermeldung erscheint ist alles in Ordnung. 
{{< /admonition >}}

Nach Einiger Zeit, wenn der Stack erstellt ist, landen wir automatisch in der Übersicht der Stacks.

{{< image src="/img/posts/2023-02-15/portainer_stacks3.png" alt="Portainer ioBroker Stack" width="100%" >}}

Mit einem Klick auf unseren Stack bekommen wir eine Übersicht des Stacks und seiner Container.

{{< image src="/img/posts/2023-02-15/portainer_stacks4.png" alt="Portainer ioBroker Stack Übersicht" width="100%" >}}

Unter "Networks" sollten wir außerdem ein neues Netzwerk namens "iobroker_bridge" finden. Der Name resultiert aus dem Namen des Stacks und dem im Stack definierten Netzwerknamens. 

{{< image src="/img/posts/2023-02-15/portainer_stacks5.png" alt="Portainer Networks" width="100%" >}}

Gleiches gilt natürlich auch für die von uns definierten Volumes.

{{< image src="/img/posts/2023-02-15/portainer_stacks6.png" alt="Portainer Volumes" width="100%" >}}

Damit ist unser Stack erstellt. Schaue dir auch gerne mal die Details der erstellten Elemente wie Container, Networks und Volumes an um zu verstehen wie diese Elemente zusammen hängen. Außerdem kannst du in den Details eine ganze Reihe an Informationen auslesen. Bei den Volumes z.B. den lokalen "Mount path", oder bei den "Networks" die "Containers in network". Unter Images findest du auch die beim Erstellen herunter geladenen Docker Images. Nutze einfach die Gelegenheit dich ein wenig mit Portainer vertraut zu machen. Der ein oder andere Menüpunkt wird dir in meinen weiteren Tutorials sicherlich noch über den Weg laufen.

## Konfiguration eines Containers über Stack ändern

Docker Container können ja bekanntlich im laufenden Betrieb nicht einfach umkonfiguriert werden. Stattdessen muss man bei Änderungen den alten Container meist löschen und einen Neuen mit geänderter Konfiguration anlegen. Auch wenn es dazu im Portainer in den "Container details" mittlerweile die Funktion "Duplicate/Edit" gibt, ist das nicht selten ein reges geklicke. Und wehe es geht was schief, dann heißt es den Container komplett neu anlegen.

Hier bieten die Portainer Stacks einen großen Vorteil. So besteht die Möglichkeit einfach nachträglich Änderungen am Stack durchzuführen und den Stack neu zu deployen. Dabei kümmert sich dann Portainer selbstständig um die Neuerstellung der geänderten Container, währen die anderen Container des Stacks in der Regel einfach weiter laufen.

Um das Ganze ein wenig bildlicher zu machen, werden wir im Folgenden eine Änderung am ioBroker Container durchführen. Über das Hinzufügen der Umgebungsvariable `PACKAGES` wollen wir beim Start im Container automatisch das Paket `nano` installieren lassen. Dafür müssen wir in die Konfiguration unter `environment:` die folgende Zeile hinzufügen:

```YAML
      - PACKAGES=nano
```

Um dies zu erledigen öffnen wir die "Stack details" und wechseln in die Registerkarte "Editor". Anschließend fügen wir die Zeile in der Konfiguration ein.

{{< image src="/img/posts/2023-02-15/portainer_stacks7.png" alt="Portainer Stacks Editor" width="100%" >}}

Ein Klick auf "Update the stack" startet die Aktualisierung. In der Übersicht sollte jetzt lediglich der ioBroker Container neustarten, während unsere Redis Datenbank einfach weiter läuft. Im Log des Containers sollte zudem ersichtlich werden, dass bei Start nun das Paket `nano` installiert wird.

{{< image src="/img/posts/2023-02-15/portainer_stacks8.png" alt="Portainer Stacks Editor" width="100%" >}}

## Kurze Zusammenfassung

Fassen wir also kurz zusammen. Mit Stacks lassen sich schnell und einfach Container erstellen und ändern. Außerdem können in Stacks mehrere Container zusammen mit Netzwerken und Volumes definiert werden. Portainer übernimmt dabei das Management der Container und Ressourcen.
Das Stackfile wird in [YAML](https://de.wikipedia.org/wiki/YAML) geschrieben. Der Webeditor hilft uns bei der Syntax. Die verfügbaren Parameter sind in der ["compose file reference"](https://docs.docker.com/compose/compose-file/compose-file-v3/) definiert. Soweit verstanden? Dann viel Spaß beim Ausprobieren. Du wirst nicht mehr ohne arbeiten wollen.

## Bonus: Stackfile Snippets

Als kleinen Bonus noch ein paar Ausschnitte aus meiner Stackfile Sammlung.

#### Bridge Netzwerk mit benutzerdefiniertem Adressraum

```YAML
networks:
  bridge:                             # Name des Netzwerks innerhalb des Stacks
    driver: bridge
    ipam:
      config:
        - subnet: 172.18.0.0/16       # Subnetz
          gateway: 172.18.0.1         # Gateway
          ip_range: 172.18.0.1/24     # Adressbereich
```
Mehr Infos zum [benutzerdefinierten Bridge Netzwerk](/posts/2023/02/23_einrichtung_eines_benutzerdefinierten_bridge_netzwerks_mit_portainer/)

#### MACVLAN Netzwerk mit benutzerdefiniertem Adressraum

```YAML
networks:
  my_macvlan:                             # Name des Netzwerks innerhalb des Stacks
    driver: macvlan
    driver_opts:
      parent: eth0                        # Parent network card
    ipam:
      config:
        - subnet: 192.168.11.0/24         # Subnetz
          gateway: 192.168.11.1           # Gateway
          ip_range: 192.168.11.120/29     # Adressbereich
```
Mehr Infos zum [benutzerdefinierten MACVLAN Netzwerk](/posts/2023/02/23_einrichtung_eines_macvlan_netzwerks_mit_portainer/).

#### Existierendes Netzwerk einbinden

```YAML
networks:
  my_bridge:              # Name des Netzwerks innerhalb des Stacks
    external:
      name: my_bridge     # Name des bereits existierenden Netzwerks
```

#### Container im Host Modus ausführen

```YAML
version: "3"
services:
  iobroker:
    network_mode: host    # Netzwerkmodus Host
```

---

Ich hoffe ich konnte dir mit diesem Beitrag einen kleinen Überblick über das Arbeiten mit Portainer Stacks geben.  
&nbsp;
Für Fragen und Feedback nutze gerne die Kommentarfunktion zu diesem Beitrag. 
&nbsp;
MfG,
André