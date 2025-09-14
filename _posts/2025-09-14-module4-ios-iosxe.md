---
layout: post
title: "Inter-VLAN Routing: Essentiële Methoden voor Netwerksegmentatie"
date: 2023-11-20 10:00:00 +0100
categories: netwerken vlan routing cisco
tags: inter-vlan router-on-a-stick layer3-switch svi cisco-ios netwerkbeheer
---

Netwerksegmentatie via VLANs (Virtual Local Area Networks) is een hoeksteen van moderne netwerkarchitectuur, essentieel voor het verbeteren van beveiliging, prestaties en beheerbaarheid. Echter, zonder een mechanisme om verkeer tussen deze gesegmenteerde netwerken te leiden, zouden hosts in verschillende VLANs geïsoleerd blijven. Dit is waar **Inter-VLAN Routing** onmisbaar wordt: het is het proces van het doorsturen van netwerkverkeer van het ene VLAN naar het andere.

In deze blogpost duiken we dieper in de drie belangrijkste methoden van Inter-VLAN Routing, met een focus op moderne implementaties en gedetailleerde configuratievoorbeelden voor Cisco IOS.

## De Evolutie van Inter-VLAN Routing

De behoefte aan communicatie tussen VLANs heeft geleid tot verschillende implementatiemethoden, elk met hun eigen schaalbaarheid en complexiteit.

### 1. Legacy Inter-VLAN Routing (Verouderd)

Oorspronkelijk werd Inter-VLAN routing geïmplementeerd met behulp van een router met meerdere fysieke Ethernet-interfaces. Elke interface was verbonden met een switchpoort in een apart VLAN en diende als de standaardgateway voor de hosts in dat specifieke VLAN.

**Nadeel:** Deze methode is **niet schaalbaar**. Routers hebben een beperkt aantal fysieke interfaces, wat snel een knelpunt wordt in netwerken met veel VLANs. Vanwege deze beperkingen wordt deze methode **niet langer geïmplementeerd in moderne geschakelde netwerken**.

### 2. Router-on-a-Stick (Kleine tot Middelgrote Netwerken)

De "Router-on-a-Stick" methode overwint de beperkingen van legacy Inter-VLAN routing door **slechts één fysieke Ethernet-interface** op de router te gebruiken om verkeer tussen meerdere VLANs te routeren. Deze interface wordt geconfigureerd als een **802.1Q trunk** en verbonden met een trunkpoort op een Layer 2 switch.

De magie zit in de **subinterfaces**: de fysieke routerinterface wordt logisch verdeeld in meerdere softwarematige virtuele interfaces. Elk subinterface is gekoppeld aan een specifiek VLAN (via 802.1Q tagging) en krijgt een uniek IP-adres, dat fungeert als de **standaardgateway** voor de hosts in dat VLAN. Wanneer getagd verkeer de router binnenkomt, wordt het naar de juiste subinterface gestuurd, waar de router een routeringsbeslissing neemt en het verkeer eventueel getagd terugstuurt via dezelfde fysieke interface met de tag van het nieuwe VLAN.

**Voordelen:**
*   Elimineert de noodzaak voor meerdere fysieke routerinterfaces.
*   Een acceptabele oplossing voor kleine tot middelgrote netwerken.

**Nadelen:**
*   Schaalt niet verder dan **ongeveer 50 VLANs**.
*   Kan een **single point of failure** zijn, aangezien één fysieke link alle inter-VLAN verkeer afhandelt.
*   Hogere latentie vergeleken met Layer 3 switches, omdat verkeer de switch moet verlaten en de router moet passeren.

#### Configuratievoorbeeld: Router-on-a-Stick (Cisco IOS)

De configuratie voor Router-on-a-Stick omvat stappen op zowel de Layer 2 switch als de router.

**1. Switch Configuratie (Layer 2)**
*   Maak de benodigde VLANs aan en geef ze namen.
*   Wijs toegangspoorten toe aan de respectievelijke VLANs.
*   Configureer de poort die naar de router gaat als een 802.1Q trunkpoort.

{% highlight bash %}
Switch(config)# vlan 10
Switch(config-vlan)# name DATA_VLAN
Switch(config-vlan)# exit
Switch(config)# vlan 20
Switch(config-vlan)# name VOICE_VLAN
Switch(config-vlan)# exit
Switch(config)# interface FastEthernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# exit
Switch(config)# interface FastEthernet0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
Switch(config-if)# exit
Switch(config)# interface GigabitEthernet0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# exit
{% endhighlight %}

**2. Router Configuratie (Cisco IOS)**
*   Maak voor elk te routeren VLAN een subinterface aan op de fysieke interface die met de switch is verbonden. Het is gebruikelijk om het subinterfacenummer te laten overeenkomen met het VLAN-nummer.
*   Configureer encapsulatie (`encapsulation dot1q vlan_id`) op elke subinterface om te reageren op 802.1Q getagd verkeer van het gespecificeerde VLAN ID. De `native` optie wordt alleen gebruikt om het native VLAN in te stellen op iets anders dan VLAN 1.
*   Wijs aan elke subinterface een uniek IPv4-adres en subnetmasker toe; dit adres fungeert als de standaardgateway voor het betreffende VLAN.
*   Schakel tot slot de fysieke hoofdinterface in met de `no shutdown` opdracht. Als de fysieke interface uit staat, zijn alle subinterfaces ook uitgeschakeld.

{% highlight bash %}
Router(config)# interface GigabitEthernet0/0/1.10
Router(config-subif)# encapsulation dot1q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# exit
Router(config)# interface GigabitEthernet0/0/1.20
Router(config-subif)# encapsulation dot1q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
Router(config-subif)# exit
Router(config)# interface GigabitEthernet0/0/1
Router(config-if)# no shutdown
Router(config-if)# exit
{% endhighlight %}

**Noot over Cisco XE en andere vendors:** De beschikbare bronnen zijn primair gericht op Cisco IOS. Specifieke configuraties voor Cisco XE kunnen variëren, hoewel de onderliggende concepten van subinterfaces en 802.1Q trunking vergelijkbaar zijn. Voor andere vendors zoals HPE of Juniper, beschrijven de bronnen geen specifieke implementatiedetails. De kernprincipes van Inter-VLAN routing (VLANs, trunking, Layer 3 gateway functionaliteit) blijven echter universeel.

### 3. Layer 3 Switch met Switched Virtual Interfaces (SVIs) (Schaalbaar, Enterprise)

De meest schaalbare en moderne oplossing voor Inter-VLAN routing is het gebruik van **Layer 3 switches**, ook wel **multilayer switches** genoemd. Deze switches kunnen zowel Layer 2- als Layer 3-bewerkingen uitvoeren, waardoor ze zowel kunnen schakelen als routeren.

Layer 3 switches maken gebruik van **Switched Virtual Interfaces (SVIs)**. Een SVI is een virtuele interface die op de Layer 3 switch is geconfigureerd en die dezelfde Layer 3-verwerkingsfuncties voor een VLAN uitvoert als een routerinterface. Het toegewezen IP-adres van de SVI dient als de standaardgateway voor de hosts in dat VLAN.

**Voordelen van Layer 3 Switches:**
*   **Hogere snelheid:** Gebruikt hardware-gebaseerd schakelen en routeren, wat resulteert in veel hogere pakketverwerkingssnelheden dan routers.
*   **Geen externe links nodig:** Eliminatie van externe links van de switch naar de router vermindert de bekabelingscomplexiteit en een potentieel single point of failure.
*   **Lagere latentie:** Data hoeft de switch niet te verlaten om gerouteerd te worden, wat de latentie aanzienlijk verlaagt.
*   **Hogere bandbreedte:** Mogelijkheid om Layer 2 EtherChannels te gebruiken als trunklinks tussen switches om de bandbreedte te vergroten, wat niet mogelijk is met een enkele Router-on-a-Stick verbinding.
*   **Gangbaarder in campus LANs:** Vaker ingezet in campus LANs dan routers, wat de architectuur vereenvoudigt.

**Nadeel:**
*   Layer 3 switches zijn doorgaans **duurder** dan Layer 2 switches en routers.

#### Configuratievoorbeeld: Inter-VLAN Routing met Layer 3 Switch (Cisco IOS)

Het configureren van een Layer 3 switch voor Inter-VLAN routing is relatief eenvoudig.

**1. VLANs en SVIs aanmaken**
*   Maak de benodigde VLANs aan op de Layer 3 switch.
*   Voor elk VLAN dat moet worden gerouteerd, wordt een Switched Virtual Interface (SVI) aangemaakt. Het IP-adres dat aan deze SVI wordt toegewezen, zal dienen als de standaardgateway voor hosts in dat VLAN.

{% highlight bash %}
Switch(config)# vlan 10
Switch(config-vlan)# name DATA_VLAN
Switch(config-vlan)# exit
Switch(config)# vlan 20
Switch(config-vlan)# name VOICE_VLAN
Switch(config-vlan)# exit
Switch(config)# interface vlan 10
Switch(config-if)# ip address 192.168.10.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit
Switch(config)# interface vlan 20
Switch(config-if)# ip address 192.168.20.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit
{% endhighlight %}

**2. Access poorten configureren**
*   Wijs de juiste switchpoorten toe aan de respectievelijke VLANs als access poorten.

{% highlight bash %}
Switch(config)# interface GigabitEthernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# exit
Switch(config)# interface GigabitEthernet0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
Switch(config-if)# exit
{% endhighlight %}

**3. IP-routing inschakelen**
*   Geef de globale configuratieopdracht `ip routing` om Layer 3 routing op de switch in te schakelen. Dit is essentieel voor het uitwisselen van verkeer tussen VLANs.

{% highlight bash %}
Switch(config)# ip routing
{% endhighlight %}

#### Geleide Poorten (Routed Ports)

Voor communicatie met andere Layer 3-apparaten, zoals een externe router, kunnen **geleide poorten (routed ports)** worden geconfigureerd op een Layer 3 switch. Een geleide poort is een Layer 2 switchpoort die wordt geconverteerd naar een Layer 3 interface door de `no switchport` opdracht te gebruiken. Vervolgens kan aan deze poort een IP-adres worden toegewezen en kan deze deelnemen aan routeringsprotocollen.

{% highlight bash %}
Switch(config)# interface GigabitEthernet0/3
Switch(config-if)# no switchport
Switch(config-if)# ip address 10.0.0.1 255.255.255.252
Switch(config-if)# no shutdown
Switch(config-if)# exit
{% endhighlight %}

**Noot over Cisco XE en andere vendors:** Net als bij Router-on-a-Stick, zijn de bronnen sterk gericht op Cisco IOS voor Layer 3 switches. Voor Cisco XE zijn de commando's voor SVI's en `ip routing` over het algemeen vergelijkbaar. Implementaties bij andere vendors zoals HPE (bijv. met VLAN interfaces) of Juniper (met IRB - Integrated Routing and Bridging interfaces) volgen vergelijkbare logische principes voor het creëren van Layer 3 gateway-functionaliteit per VLAN, maar gebruiken hun eigen specifieke CLI-syntax. De bronnen bieden hiervoor geen concrete configuratievoorbeelden.

## Probleemoplossing van Inter-VLAN Routing

Inter-VLAN routingproblemen zijn meestal gerelateerd aan connectiviteitskwesties. Hieronder de meest voorkomende oorzaken en verificatiecommando's:

*   **Ontbrekende VLANs:** Het VLAN is niet aangemaakt, per ongeluk verwijderd, of wordt niet toegestaan op de trunklink. Dit kan ertoe leiden dat poorten inactief worden.
    *   **Verificatie:** `show vlan brief`, `show interfaces switchport`.
*   **Switch Trunk Poort Problemen:** Bij "Router-on-a-Stick" is een verkeerd geconfigureerde trunkpoort (de poort die de switch met de router verbindt) een veelvoorkomende oorzaak. De poort is mogelijk niet geconfigureerd als een trunk of is uitgeschakeld.
    *   **Verificatie:** `show interface trunk`, `show running-config interface X`.
*   **Switch Access Poort Problemen:** Een hostpoort is onjuist toegewezen aan het verkeerde VLAN, of de host zelf heeft een incorrecte IP-configuratie (IP-adres, subnetmasker, standaardgateway). Een host kan zijn standaardgateway niet pingen.
    *   **Verificatie:** `show vlan brief`, `show interface X switchport`, `show running-config interface X`. Op de host: `ipconfig` (Windows).
*   **Router Configuratie Problemen:** Bij "Router-on-a-Stick" zijn dit meestal problemen met de subinterface-configuratie, zoals een onjuist IPv4-adres, een verkeerde VLAN-ID-toewijzing, of de fysieke interface van de router is niet ingeschakeld.
    *   **Verificatie:** `show ip interface brief`, `show interfaces` (voor gedetailleerde encapsulatie-informatie).

**Algemene verificatie:** Het `ping`-commando is essentieel om end-to-end connectiviteit tussen apparaten in verschillende VLANs te testen. De `show ip route` opdracht kan ook worden gebruikt om de routeringstabel te controleren en te zien of de router de netwerken van de verschillende VLANs kent.

## Conclusie

Inter-VLAN routing is een fundamenteel aspect van moderne netwerken, essentieel voor het laten communiceren van logisch gesegmenteerde VLANs. Hoewel **Legacy Inter-VLAN Routing** historisch relevant is, wordt het niet langer gebruikt vanwege de schaalbaarheidsbeperkingen.

De **Router-on-a-Stick** methode biedt een kosteneffectieve oplossing voor kleine tot middelgrote netwerken door slim gebruik te maken van subinterfaces en 802.1Q trunking. Voor grotere en complexere bedrijfsnetwerken is de **Layer 3 switch met SVIs** de superieure keuze, vanwege de hogere prestaties, lagere latentie en inherente schaalbaarheid die hardware-gebaseerd schakelen en routeren met zich meebrengen.

Het kiezen van de juiste methode en een nauwkeurige configuratie zijn cruciaal voor een functioneel en efficiënt netwerk. Een grondig begrip van de configuratiestappen en een proactieve benadering van probleemoplossing zijn essentieel voor elke netwerkbeheerder.