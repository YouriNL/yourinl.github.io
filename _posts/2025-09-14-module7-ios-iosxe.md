---
layout: post
title: "DHCPv4 Implementatie: Een Essentiële Gids voor Cisco Netwerken"
date: 2023-10-26 10:00:00 +0100
categories: [Netwerken, DHCP, Cisco, Configuratie]
tags: [DHCPv4, Cisco IOS, IP-beheer, Netwerkconfiguratie, Routers]
author: NetwerkExpert
---

## Dynamisch IP-Beheer met DHCPv4: Een Diepgaande Blik op Cisco Implementaties

### Inleiding

In het hedendaagse netwerklandschap is efficiënt IP-adresbeheer van cruciaal belang. Het handmatig toewijzen van IP-adressen in netwerken met een groot aantal apparaten is tijdrovend en foutgevoelig. Hier komt **Dynamic Host Configuration Protocol v4 (DHCPv4)** om de hoek kijken. DHCPv4 is een essentieel netwerkprotocol dat het dynamisch toewijzen van IPv4-adressen en andere netwerkconfiguratie-informatie aan clientapparaten automatiseert en vereenvoudigt. Dit bespaart netwerkbeheerders aanzienlijk veel tijd.

In kleine kantoren (SOHO-omgevingen) of bijkantoren kan een Cisco-router zelfs worden geconfigureerd om DHCPv4-services te leveren, zonder de noodzaak van een dedicated server. In deze blogpost duiken we dieper in de werking en configuratie van DHCPv4, specifiek gericht op Cisco IOS-apparaten, en bespreken we de verschillende rollen die een router kan aannemen binnen dit protocol.

### De Fundamenten van DHCPv4: Lease Mechanisme en Werking

DHCPv4 opereert volgens een **client/server-model**, waarbij de server IPv4-adressen 'least' aan clients voor een beperkte periode. Dit lease-mechanisme is essentieel, omdat het ervoor zorgt dat IP-adressen efficiënt worden hergebruikt wanneer clients de verbinding verbreken of uitschakelen, waardoor ze geen adressen vasthouden die ze niet langer nodig hebben. De leasetijd kan variëren van 24 uur tot een week of langer.

#### Lease Verkrijgingsproces (4 stappen)

Wanneer een client opstart en een lease wil verkrijgen, doorloopt deze een vierstappenproces:

1.  **DHCP Discover (DHCPDISCOVER)**: De client stuurt een broadcastbericht om een DHCPv4-server in het netwerk te vinden.
2.  **DHCP Offer (DHCPOFFER)**: Een DHCPv4-server die het Discover-bericht ontvangt, reageert met een aanbod van een beschikbaar IP-adres en andere configuratie-informatie.
3.  **DHCP Request (DHCPREQUEST)**: De client selecteert een aanbod en stuurt een broadcastbericht om de acceptatie van het aangeboden adres te bevestigen aan alle servers.
4.  **DHCP Acknowledgment (DHCPACK)**: De geselecteerde DHCPv4-server stuurt een bevestiging van de lease, waarmee het adres definitief wordt toegewezen aan de client.

#### Lease Vernieuwingsproces (2 stappen)

Voordat de lease verloopt, probeert de client deze te vernieuwen via een tweestappenproces:

1.  **DHCP Request (DHCPREQUEST)**: De client stuurt een unicast DHCPREQUEST-bericht rechtstreeks naar de server die oorspronkelijk het adres heeft toegekend. Als geen bevestiging (ACK) wordt ontvangen, stuurt de client een broadcast DHCPREQUEST.
2.  **DHCP Acknowledgment (DHCPACK)**: De server bevestigt de leaseverlenging.

### Cisco Routers als DHCPv4-server

Een Cisco-router met Cisco IOS-software kan volledig functioneren als een DHCPv4-server. Dit is een handige oplossing voor kleinere netwerkomgevingen waar een dedicated server niet praktisch of noodzakelijk is.

#### Configuratievoorbeelden voor Cisco IOS

De configuratie van een Cisco IOS DHCPv4-server omvat doorgaans drie stappen:

1.  **Sluit IPv4-adressen uit:** Het is belangrijk om adressen uit te sluiten die statisch zijn toegewezen aan apparaten zoals routers, servers en printers. Dit voorkomt IP-adresconflicten.

    {% highlight bash %}
    Router(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.9
    Router(config)# ip dhcp excluded-address 192.168.10.254
    {% endhighlight %}

2.  **Definieer een DHCPv4-poolnaam:** Gebruik het commando `ip dhcp pool` gevolgd door een naam om een adrespool te creëren. Dit brengt de router in de DHCPv4-configuratiemodus (`Router(dhcp-config)#`).

    {% highlight bash %}
    Router(config)# ip dhcp pool LAN_POOL_10
    Router(dhcp-config)#
    {% endhighlight %}

3.  **Configureer de DHCPv4-pool:** Binnen de poolmodus configureert u het netwerkbereik, de standaardgateway en eventueel DNS-servers, domeinnamen en leasetijden.

    {% highlight bash %}
    Router(dhcp-config)# network 192.168.10.0 255.255.255.0
    Router(dhcp-config)# default-router 192.168.10.1
    Router(dhcp-config)# dns-server 8.8.8.8 8.8.4.4
    Router(dhcp-config)# domain-name example.com
    Router(dhcp-config)# lease 2 0 0  # Lease van 2 dagen, 0 uur, 0 minuten
    {% endhighlight %}

#### Verificatie van de DHCPv4-server

Om de correcte werking van de DHCPv4-server te verifiëren, kunt u de volgende commando's gebruiken:

*   `show running-config | section dhcp`: Toont de geconfigureerde DHCPv4-commando's.
*   `show ip dhcp binding`: Toont een lijst met toegewezen IPv4-adressen aan MAC-adressen.
*   `show ip dhcp server statistics`: Toont statistieken over verzonden en ontvangen DHCPv4-berichten.

Op een client-pc kan `ipconfig /all` worden gebruikt om te controleren of de client IP-adresinformatie heeft ontvangen.

#### DHCPv4-service Uitschakelen

De DHCPv4-service is standaard ingeschakeld. Om deze uit te schakelen, gebruikt u het globale configuratiecommando `no service dhcp`. Om de service opnieuw in te schakelen, gebruikt u `service dhcp`.

    {% highlight bash %}
    Router(config)# no service dhcp
    Router(config)# service dhcp
    {% endhighlight %}

#### Cisco IOS vs. Cisco XE voor DHCPv4 Server

De beschikbare bronnen richten zich specifiek op **Cisco IOS** software. **Cisco IOS XE** is een ander besturingssysteem van Cisco dat gebruikt wordt op nieuwere routerplatforms (zoals de ISR G2, ASR 1000). Hoewel de Command Line Interface (CLI) en de logica achter DHCPv4-configuratie grotendeels vergelijkbaar zijn tussen IOS en IOS XE, kunnen er subtiele verschillen bestaan in syntax, functionaliteit of de beschikbaarheid van bepaalde opties.

De kerncommando's zoals `ip dhcp excluded-address`, `ip dhcp pool`, `network`, `default-router`, `dns-server`, en de `show` commando's zijn doorgaans identiek of zeer vergelijkbaar in IOS XE. Voor exacte details en eventuele platformspecifieke configuraties is het altijd aan te raden de officiële Cisco documentatie voor uw specifieke IOS XE-versie te raadplegen, aangezien de bronnen geen specifieke IOS XE-voorbeelden bevatten.

### DHCPv4 Relay Agent: Communicatie Over Subnetten Heen

In complexe netwerken bevinden clients en DHCP-servers zich vaak op verschillende subnetten. Aangezien DHCPDISCOVER-berichten broadcasts zijn, kunnen ze niet standaard over routergrenzen heen worden gestuurd. Een router kan echter worden geconfigureerd als een **DHCPv4 relay agent** om deze broadcasts door te sturen.

#### Configuratievoorbeelden voor Cisco IOS

De `ip helper-address` interface configuratiecommando zorgt ervoor dat de router DHCPv4-broadcasts (en andere UDP-services) doorstuurt als een unicast-bericht naar een gespecificeerde DHCPv4-server op een ander netwerk.

    {% highlight bash %}
    Router(config)# interface GigabitEthernet0/1
    Router(config-if)# ip helper-address 192.168.11.6
    {% endhighlight %}
    (Hierbij is `192.168.11.6` het IP-adres van de DHCPv4-server.)

#### Standaard Doorgestuurde UDP-services

Het `ip helper-address` commando stuurt standaard niet alleen DHCP/BOOTP-berichten door (poorten 67 en 68), maar ook andere UDP-services:

*   Poort 37: Time
*   Poort 49: TACACS
*   Poort 53: DNS
*   Poort 67: DHCP/BOOTP server
*   Poort 68: DHCP/BOOTP client
*   Poort 69: TFTP
*   Poort 137: NetBIOS name service
*   Poort 138: NetBIOS datagram service

#### Cisco IOS vs. Cisco XE voor DHCPv4 Relay

Ook hier geldt dat de basisfunctionaliteit en het `ip helper-address` commando identiek zijn in Cisco IOS en IOS XE. De configuratie wordt uitgevoerd in de interface-configuratiemodus, net zoals bij IOS. De bronnen bieden geen specifieke verschillen, wat suggereert dat de implementatie in dit opzicht zeer consistent is tussen de twee besturingssystemen. Desalniettemin, raadpleeg altijd de documentatie van uw specifieke IOS XE-platform voor de meest nauwkeurige informatie.

### Cisco Routers als DHCPv4-client

In bepaalde scenario's, zoals in SOHO-omgevingen waar een router verbinding maakt met een ISP, kan een Cisco IOS-router fungeren als een DHCPv4-client. Dit betekent dat de router automatisch een IPv4-adres en andere netwerkinstellingen verkrijgt van de DHCPv4-server van de ISP.

#### Configuratievoorbeeld voor Cisco IOS

Om een Ethernet-interface op een Cisco-router als DHCP-client te configureren, gebruikt u het `ip address dhcp` commando in de interface-configuratiemodus.

    {% highlight bash %}
    Router(config)# interface GigabitEthernet0/0/1
    Router(config-if)# ip address dhcp
    Router(config-if)# no shutdown
    {% endhighlight %}

#### Thuisrouters als DHCPv4-client

Veel thuisrouters zijn standaard al geconfigureerd om IPv4-adresinformatie automatisch te ontvangen van de ISP. De internetverbindingstype is dan vaak ingesteld op "Automatic Configuration - DHCP". Dit maakt de installatie voor de eindgebruiker eenvoudig.

#### Verificatie

Na configuratie als DHCPv4-client kunt u met `show ip interface [interface-type interface-number]` verifiëren dat de interface een adres heeft ontvangen en operationeel is.

    {% highlight bash %}
    Router# show ip interface GigabitEthernet0/0/1
    {% endhighlight %}

#### Cisco IOS vs. Cisco XE voor DHCPv4 Client

De configuratie van een interface als DHCPv4-client met het `ip address dhcp` commando is een standaardfunctie en is consistent over Cisco IOS en IOS XE platforms. De bronnen vermelden geen afwijkende syntax of gedrag voor IOS XE in deze context, wat de universaliteit van dit commando benadrukt.

### Andere Vendors: HPE en Juniper

De verstrekte bronnen concentreren zich uitsluitend op de implementatie en configuratie van DHCPv4 binnen het Cisco-ecosysteem, met name op apparaten die Cisco IOS draaien. Informatie over hoe andere netwerkleveranciers zoals **HPE** of **Juniper** DHCPv4 implementeren of configureren, wordt niet behandeld in deze materialen. Hoewel de basisprincipes van DHCPv4 universeel zijn, zullen de specifieke commandosyntax en de configuratiehiërarchie per leverancier verschillen. Raadpleeg voor configuratiedetails van HPE of Juniper de respectievelijke productdocumentatie van die leveranciers.

### Conclusie

DHCPv4 is een onmisbaar protocol voor het automatiseren van IP-adresbeheer in moderne netwerken. Cisco-routers bieden de flexibiliteit om te fungeren als DHCPv4-server, -client of -relay agent, waardoor ze in diverse netwerkscenario's inzetbaar zijn. Door de juiste configuratie kunnen netwerkbeheerders de complexiteit van IP-adresmanagement aanzienlijk verminderen en zorgen voor een robuuste en efficiënte netwerkinfrastructuur. Hoewel de basiscommando's voor Cisco IOS in dit document zijn behandeld, is het altijd raadzaam de meest actuele documentatie te raadplegen voor platformspecifieke details en best practices, zeker bij het werken met Cisco IOS XE of andere netwerkleveranciers.