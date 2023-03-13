---
title: 'Grundlagen und Voraussetzungen'
#subtitle: 'Das Fundament für unser Smarthome mit Docker'
description: ''
date: 2023-02-01T00:00:01+01:00
lastmod: ''
draft: false
featuredImage: /img/posts/2023-02-01_header1.png

tags: []
categories: [Sonstiges]

seo:
  image: ''
---

Bevor wir unser Smarthome mit Hilfe von Docker zum Leben erwecken können, sollten wir unbedingt kurz über die Grundlagen und Voraussetzungen sprechen, die für das Verständnis und die erfolgreiche Umsetzung meiner Tutorials erforderlich sind. Dazu habe ich ein paar Fragen zusammen gestellt über die ihr euch vorab einmal Gedanken gemacht haben solltet. 

## Was ist Docker?

Wer in seinem Smarthome die Vorzüge von Docker nutzen möchte, der wird nicht umhin kommen sich tiefer mit diesem Thema zu beschäftigen und sich ggf. selbst Wissen dazu anzueignen. 

Wenn es um die Definition von Docker geht, sagt Wikipedia dazu: [„Docker ist eine Freie Software zur Isolierung von Anwendungen mit Containervirtualisierung.“](https://de.wikipedia.org/wiki/Docker_(Software)).

Zugegeben, so richtig schlau gemacht (in praktischer Hinsicht) hat mich der Artikel nicht. Daher hier ein kleiner Tipp: Schaut mal bei [Youtube](https://www.youtube.com/results?search_query=docker) vorbei. Dort gibt es eine Menge Informationen zu Docker. Das geht los bei "Docker in 5 Minuten" und endet bei mehrteiligen Video-Tutorials oder zweistündigen Online-Video-Kursen. Da ist wirklich für jeden was dabei.

Für alle weiteren Infos zu Docker würde ich dann unbedingt auf die offiziellen [Docker Docs](https://docs.docker.com/) verweisen. Hier findet ihr wirklich alles was es zum Thema zu wissen gibt.

## Welche Hardware benötige ich? 

Ein weiteres Thema ist die Frage nach der verwendeten Hardwareplattform für den Docker Dienst. Mittlerweile ist Docker weit verbreitet, sodass man den Dienst eigentlich in allen gängigen NAS Systemen finden kann. Unter Linux gibt es Distributionen die entweder bereits Docker enthalten oder die Möglichkeit bieten den Dienst bei der Einrichtung automatisch installieren zu lassen. Und auch unter Windows hält der Dienst dank WSL Einzug.

Im folgenden habe ich einmal die gängigsten Plattformen aufgelistet und eine kurze Einschätzung, basierend auf meinen eigenen Erfahrungen, formuliert. Vielleicht hilft euch dies bei der Entscheidung wie ihr Docker in eurer Smarthomeumgebung integrienen könnt. 

### NAS Systeme

Der einfachste Weg Docker zu nutzen führt wahrscheinlich über die fertigen NAS Lösungen der gängigen Anbieter wie Synology und QNAP oder NAS-Betriebssysteme wie Unraid, OpenMediaVault oder TreuNAS. Der Vorteil liegt auf der Hand. Docker ist auf den meisten Systemen über Zusatzpakete oder Apps aus der Weboberfläche heraus installierbar. Das Erstellen und Verwalten von Containern erfolgt dann ebenfalls über eigens entwickelte Weboberflächen. Leider können diese herstellerspezifischen Oberflächen meist nicht den kompletten Funktionsumfang von Docker bereitstellen. Abhilfe schaffen hier aber herstellerunabhängige Oberflächen wie z.B. Portainer. Ein Tool auf das ich hier im Blog demnächst noch genauer eingehen werde. 

Ein weiterer Aspekt den es bei NAS Systemen zu berücksichtigen gilt sind die Systemressourcen. Gerade die kleineren, günstigen Geräte kommen oftmals mit wenig Rechenleistung und Arbeitsspeicher. Das reicht zwar für die Verwaltung von Netzwerkspeicher, bei größeren Aufgaben wie z.B. der Steuerung eines Smarthomes oder einer Videoaufzeichnung stoßen die Geräte schnell an ihre Grenzen. Man sollte also stets die Auslastung der Ressourcen im Blick behalten und, sofern möglich, z.B. den Arbeitsspeicher entsprechend aufrüsten.

Für jeden der bereits ein NAS sein Eigen nennt ist dies eine gute Option um in das Thema Smarthome mit Docker einzutauchen. Wer kein NAS besitzt, der sollte in Anbetracht des in der Regel relativ hohen Anschaffungspreises abwägen ob ein NAS die richtige Entscheidung ist. Andere Lösungen wie SBCs oder Mini PCs bieten oftmals mehr Leistung bei weniger Investition.  

Meine Erfahrung: Ich habe mit einer Synology DiskStation 1515+ (Prozessor: Intel Atom C2538 Quad Core 2,4 GHz) und 6GB Arbeitsspeicher begonnen. Nachdem ich aber einen Plex Mediaserver und die Survelliance Station (Videoüberwachung) mit 6 Kameras aktiviert hatte, war das dann doch ein bisschen zu wenig Leistung und ich musste mit meinem Smarthome auf einen anderen Docker Host umziehen.  

### Einplatinencomputer (RaspberryPi & Co)

Gerade auch wegen ihres geringen Stromverbrauchs sind Einplatinen Computer, kurz SBCs, wie der Raspberry Pi sehr beliebt wenn es um die Übernahme von Aufgaben im Smarthome geht. Allerdings gilt auch hier, dass die Ressourcen schnell erschöpft sein können. Ich persönlich bin ein Fan vom Raspberry Pi 4 mit 4 oder 8 GB RAM. Hier bekommt man schon eine Menge geboten. Zudem ist der Raspberry wegen seiner Verbreitung gut dokumentiert und es finden sich zahllose Anleitungen und Tutorials im Netz.

Ich persönlich halte den Raspberry Pi und seine SBC-Kollegen für die beste Möglichkeit mit relativ wenig Investition den Einstieg in Smarthome mit Docker zu wagen. Zumindest wenn man nicht schon ein Docker taugliches NAS Zuhause hat.

Trotzdem birgt der Raspberry Pi natürlich seine Tücken. So hört man immer wieder von SD Karten die durch zu viele Lese- und Schreibzyklen das Zeitliche segnen. Aber auch hierfür gibt es Lösungen. Diese reichen von Konfigurationsänderungen zur Minimierung der Last, über die Verwendung externer Speichermedien, bis hin zum Umstieg auf eine Raspberry Pi Alternative mit z.B. einer SATA-Schnittstelle für Festplatten.

Meine Empfehlung: Schmeißt die Stichworte dazu einfach mal in eine Suchmaschine. Ihr werdet reichlich Erfahrungsberichte und Lösungen finden. Versprochen.

### Mini PCs

Mini PCs, wie z.B. der Intel NUC, sind wohl sowas wie die nächste Evolutionsstufe der SBCs. Geringer Stromverbrauch, genügend Leistung und manchmal sogar die Möglichkeit zwei SSDs zu verbauen erfüllen in der Regel alle Wünsche wenn es um den eigenen Homeserver für Docker geht. Egal ob als Docker Host mit Ubuntu Betriebssystem oder gar als komplette Virtualisierungsplattform mit Proxmox oder Vmware. Viele User (mich eingeschlossen) setzen auf die kleinen PCs. Allerdings sind hier die technischen Hürden zum Aufsetzen eines solchen Systems auch geringfügig höher als das flashen einer SD Karte für einen Einplatinen Computer. Dank ausführlicher Anleitungen im Internet für fortgeschrittene Benutzer aber kein Problem.

Meine Erfahrung: Für mein Smarthome habe ich eine Zeit lang auf einen Intel NUC (Intel Quad-Core J3455 Prozessor 4x 1,50 GHz mit Turbo-Boost bis zu 4x 2,30 GHz) mit 16 GB RAM gesetzt. Meines Erachtens für ein mittelgroßes Smarthome völlig ausreichend. Umgestiegen bin ich letztendlich nur, weil ich mir für die Softwareentwicklung, Desktop Virtualisierung und meine Testumgebung einen eigenen Proxmox Server aufgesetzt habe. Da dieser genügend Ressourcen hat, habe ich schlicht den NUC eingespart und das komplette Smarthome Setup auf den Server umgezogen. 

### Proxmox, VMware, Hyper-V oder Virtual Box

Die letzte Evolutionsstufe für das Smarthome ist dann wohl die Verwendung eines Hypervisors (Virtualisierungshost). Letztendlich handelt es sich dabei um ein Server Betriebssystem (läuft aber in der Regel auch auf einem Mini PC) über welches man virtuelle Maschinen erstellen kann, in denen dann z.B. ein Docker Dienst seine Arbeit verrichtet. Zum Aufsetzen einer solchen Lösung ist dann allerdings schon etwas mehr technisches Verständnis und Wissen erforderlich. Zudem sind entsprechende Server und Lösungen meist für das eigene Smarthome recht überdimensioniert.

Eine solche virtuelle Infrastruktur hat natürlich aber auch Vorteile. So ist es in der Regel dank Snapshot Funktionalitäten und eingebauten Backup Tools recht einfach möglich Sicherungen der virtuellen Maschinen anzulegen und bei Bedarf auf Knopfdruck wiederherzustellen.

Meine Erfahrung: Ich setze seit einiger Zeit auf einen Proxmox Virtualisierungshost. Darüber laufen verschiedene virtuelle Maschinen und LXC Container. Darin dann wiederum mein Docker Dienst und die verschiedenen Container für meine Smarthome-Dienste. Ich nutze den Server allerdings nicht nur als Zuhause für mein Smarthome, sondern auch für meine Test- und Spielumgebung sowie zur Softwareentwicklung und Desktop-Virtualisierung. 

## Habe ich mein Netzwerk im Griff?

Nach dem kleinen Ausflug bezüglich der Hardwareplattformen auf denen wir Docker einsetzen können, nun zurück zu einem nicht zwingend entscheidendem, aber sehr wohl wichtigem Thema: Eurem Heimnetzwerk. Je größer euer Smarthome wird, um so wichtiger ist meines Erachtens, dass ihr etwas mit Begriffen wie DHCP, DNS oder Gateway anfangen könnt und wisst was z.B. IP-Adressen sind. Bei größeren Smarthomes oder wenn euch bestimmte Sicherheitsthemen wichtig sind wird dann vermutlich irgendwann auch so etwas wie VLANs interessant. Also genau die richtige Zeit euch hier etwas weiter zu bilden, wenn es hier noch Wissenslücken gibt. 

Denn, genauso wie das Heimnetzwerk eurer Fritzbox die physischen Netzwerkgeräte verbindet, so verbindet der Docker Dienst mit seiner eigenen Netzwerkkonfiguration später auch die Docker Container untereinander oder macht sie aus dem Heimnetzwerk erreichbar. Wenn ihr euch also in eurem Heimnetzwerk auskennt, wird es euch leichter fallen die virtuelle Netzwerkstruktur von Docker zu verstehen und für eure Zwecke einzusetzen. Dazu aber zu gegebener Zeit mehr. 

## Habe ich Angst vor der Kommandozeile?

Zugegeben, diese Frage ist nicht so richtig ernst gemeint. Obwohl ich mir nicht sicher bin ob es das vielleicht nicht doch als Krankheitsbild gibt... 

Wie dem auch sei, ein Docker-Dienst fühlt sich am wohlsten in einer Linux Umgebung. Wer bereits erste Erfahrungen im Umgang mit Linux sammeln durfte weiß wie schnell man dort den Weg auf die Kommandozeile findet. Und auch in einem Smarthome mit Docker wird man hin und wieder die Kommandozeile bemühen müssen. In meinen Tutorials versuche ich dies zwar immer so gut es geht zu umschiffen, aber es ist vielleicht keine schlechte Idee wenn man grundsätzlich weiß wie man auf die Kommandozeile zugreifen und etwas mit Befehlen wie `cd`, `mkdir`, `ls` oder `sudo` anfangen kann.

-----

Soviel dann ersteinmal zu den Grundlagen für's Smarthome. Ich hoffe meine Erfahrungen und Tips helfen euch, euren ganz eigenen Weg ins Smarthome mit Docker zu finden. 

MfG,
André