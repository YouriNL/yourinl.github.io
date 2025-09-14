---
layout: post
title: "VLAN's en Trunks: Essentiële Netwerksegmentatie voor Moderne Infrastructuur"
date: 2023-10-27 10:00:00 +0100
categories: [Netwerken, VLAN, Trunks, Cisco]
tags: [VLAN, Trunking, 802.1Q, DTP, Cisco IOS, Cisco XE, Netwerkbeveiliging, Netwerkbeheer]
---

## **VLAN's en Trunks: Essentiële Netwerksegmentatie voor Moderne Infrastructuur**

### Introductie tot VLAN's

**VLAN's (Virtual Local Area Networks)** vormen de ruggengraat van moderne, efficiënte netwerkinfrastructuren. In plaats van netwerken fysiek te segmenteren, bieden VLAN's een logische methode om apparaten met vergelijkbare behoeften te groeperen, ongeacht hun fysieke locatie op een switch. Dit is te vergelijken met het creëren van aparte klaslokalen binnen één grote gemeenschappelijke ruimte, waarbij VLAN's fungeren als de muren tussen deze groepen op Layer 2. Op Layer 2 kunnen echter geen 'deuren' worden gecreëerd, wat betekent dat een Layer 3-apparaat nodig is voor communicatie tussen verschillende VLAN's.

De implementatie van VLAN's biedt aanzienlijke voordelen:
*   **Kleinere Broadcast-domeinen**: Door het LAN op te splitsen, wordt het aantal broadcast-domeinen verminderd, wat de netwerkprestaties verbetert door minder onnodig verkeer.
*   **Verbeterde beveiliging**: Verkeer blijft geïsoleerd binnen het VLAN, waardoor alleen gebruikers binnen hetzelfde VLAN standaard kunnen communiceren. Dit beperkt de toegang tot gevoelige informatie.
*   **Verbeterde IT-efficiëntie en eenvoudiger beheer**: Apparaten met vergelijkbare vereisten (bijvoorbeeld docenten versus studenten) kunnen gemakkelijk worden gegroepeerd, wat het beheer stroomlijnt.
*   **Lagere kosten**: Eén enkele switch kan meerdere VLAN's ondersteunen, waardoor de behoefte aan extra fysieke switches afneemt.
*   **Unieke IP-adressering**: Elk VLAN krijgt zijn eigen unieke IP-adresbereik.
*   **Isolatie**: Broadcasts, multicasts en unicasts blijven geïsoleerd in hun individuele VLAN.

### Soorten VLAN's

Binnen een netwerkomgeving zijn er diverse soorten VLAN's, elk met een specifiek doel:

*   **Default VLAN (VLAN 1)**: Dit is het standaard VLAN voor alles, inclusief de standaard Native VLAN en Management VLAN. Het kan niet worden verwijderd of hernoemd. Hoewel de switch standaard hierdoor functioneert, raadt Cisco aan om deze standaardfuncties toe te wijzen aan andere, specifieke VLAN's voor betere praktijken.
*   **Data VLAN**: Specifiek bedoeld voor door gebruikers gegenereerd verkeer, zoals e-mail en webactiviteiten. VLAN 1 is standaard het data VLAN omdat alle interfaces hieraan zijn toegewezen.
*   **Native VLAN**: Uitsluitend gebruikt voor **trunk-verbindingen**. Op een 802.1Q trunk-link worden alle frames getagd, behalve die van het Native VLAN. Dit is ontworpen voor legacy-gebruik. Beide zijden van een trunk-link moeten hetzelfde Native VLAN geconfigureerd hebben om problemen te voorkomen.
*   **Management VLAN**: Wordt gebruikt voor beheerdoeleinden, zoals SSH/Telnet VTY-verkeer, en moet gescheiden blijven van eindgebruikersverkeer. Dit is meestal de Switched Virtual Interface (SVI) voor de Layer 2-switch.
*   **Voice VLAN**: Een afzonderlijk VLAN is essentieel voor spraakverkeer (VoIP). Spraakverkeer heeft specifieke en kritische vereisten, waaronder gegarandeerde bandbreedte, hoge **QoS (Quality of Service)**-prioriteit, vermijding van congestie, en een vertraging van minder dan 150 ms van bron tot bestemming. Het hele netwerk moet ontworpen zijn om spraak te ondersteunen. Scheiding van spraak- en dataverkeer voorkomt problemen zoals "TCP starvation", waarbij TCP-verkeer wordt gedropt wanneer buffers vol raken, wat ten goede komt aan UDP-verkeer zoals spraak.

### VLAN's in een Multi-Switch Omgeving: Trunks en 802.1Q Tagging

In netwerken met meerdere switches zijn **VLAN-trunks** onmisbaar om VLAN's over het gehele netwerk te kunnen uitbreiden. Een trunk is een punt-naar-punt verbinding tussen twee netwerkapparaten die verkeer voor meerdere VLAN's over één fysieke link transporteert. Zonder trunks zou men voor elk VLAN een aparte fysieke verbinding tussen switches nodig hebben, wat veel poorten zou verbruiken. Trunks ondersteunen standaard alle VLAN's.

De industriestandaard voor **VLAN-tagging** op moderne netwerken is **IEEE 802.1Q**. Dit protocol voegt een 4-byte header (tag) toe aan Ethernet-frames. De tag bevat belangrijke velden:
*   **Tag Protocol ID (TPID)**: Een 2-byte veld met hexadecimale waarde 0x8100. Dit identificeert het frame als een getagd 802.1Q VLAN-frame.
*   **User Priority**: Een 3-bit waarde die Layer 2 QoS-prioriteit ondersteunt (CoS - Class of Service). Dit kan door VoIP-telefoons worden ingesteld.
*   **Canonical Format Identifier (CFI)**: Een 1-bit waarde die token ring-frames op Ethernet kan ondersteunen.
*   **VLAN ID (VID)**: Een 12-bit VLAN-identificatie die tot 4096 VLAN's kan ondersteunen.

Bij het aanmaken van een tag moet de Frame Check Sequence (FCS) opnieuw worden berekend. Voordat het frame naar eindapparaten wordt verzonden, wordt de tag verwijderd en de FCS teruggebracht naar het originele nummer.

Frames van het **Native VLAN** worden als enige niet getagd op een 802.1Q trunk-link. Cisco ondersteunde vroeger ISL (Inter-Switch Link) trunking, maar 802.1Q is de voorkeur op hedendaagse netwerken, mede omdat het QoS ondersteunt, wat ISL niet doet.

VoIP-telefoons fungeren als driepoorts-switches en gebruiken CDP (Cisco Discovery Protocol) om door de switch geïnformeerd te worden over het Voice VLAN. De telefoon tagt vervolgens zijn eigen spraakverkeer en kan Layer 2 CoS prioriteit instellen. Ook ontvangt de telefoon via DHCP optie 150 voor de TFTP-server, waar de firmware vandaan komt.

### VLAN Configuratie op Cisco Switches (IOS & XE)

Cisco Catalyst switches ondersteunen meer dan 4000 VLAN's, verdeeld in twee bereiken:

*   **Normal Range VLANs (1 – 1005)**: Deze worden gebruikt in kleine tot middelgrote bedrijven. VLAN's 1, 1002-1005 worden automatisch aangemaakt en kunnen niet worden verwijderd. De configuratie van deze VLAN's wordt opgeslagen in het `vlan.dat`-bestand in flash. VTP (VLAN Trunking Protocol) kan deze VLAN's synchroniseren tussen switches.
*   **Extended Range VLANs (1006 – 4095)**: Deze worden voornamelijk gebruikt door serviceproviders. Ze zijn zichtbaar in de running-config en ondersteunen minder VLAN-functies dan normal range VLAN's. Voor het aanmaken van extended VLAN's kan VTP in transparante modus nodig zijn.

#### VLAN Aanmaken

Het aanmaken van een VLAN gebeurt in de globale configuratiemodus. Als er geen naam wordt opgegeven, genereert Cisco IOS/XE een standaardnaam zoals `vlan0020` voor VLAN 20.

**Cisco IOS & Cisco XE:**
{% highlight bash %}
Switch# configure terminal
Switch(config)# vlan 20
Switch(config-vlan)# name STUDENTEN
Switch(config-vlan)# end
{% endhighlight %}
*Bovenstaande commando's zijn identiek voor Cisco IOS en Cisco XE bij het aanmaken van VLAN's.*

#### Poorten Toewijzen aan een Data VLAN (Access Mode)

Na het aanmaken van een VLAN, wordt een switchpoort toegewezen aan dit VLAN door de poort in access-modus te plaatsen. Een access-poort kan slechts aan één data VLAN tegelijk worden toegewezen.

**Cisco IOS & Cisco XE:**
{% highlight bash %}
Switch# configure terminal
Switch(config)# interface FastEthernet0/18
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
Switch(config-if)# end
{% endhighlight %}
*Ook deze commando's zijn identiek voor Cisco IOS en Cisco XE.*

#### Data- en Voice VLAN's op één Poort

Een access poort kan één data VLAN en één Voice VLAN toegewezen krijgen, bijvoorbeeld wanneer een IP-telefoon en een pc op dezelfde switchpoort zijn aangesloten. De switch kan automatisch de VLAN aanmaken als deze nog niet bestaat, wanneer het aan een interface wordt toegewezen.

**Cisco IOS & Cisco XE:**
{% highlight bash %}
Switch# configure terminal
Switch(config)# vlan 30
Switch(config-vlan)# name STEM
Switch(config-vlan)# exit
Switch(config)# interface FastEthernet0/18
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
Switch(config-if)# switchport voice vlan 30
Switch(config-if)# mls qos trust cos
Switch(config-if)# end
{% endhighlight %}
*De `mls qos trust cos` opdracht is een voorbeeld van het configureren van QoS voor spraakverkeer. Dit is doorgaans identiek in IOS en XE.*

#### VLAN Informatie Verifiëren

Controleer de VLAN-configuratie en poorttoewijzingen met de volgende commando's:

**Cisco IOS & Cisco XE:**
{% highlight bash %}
Switch# show vlan brief
Switch# show interfaces FastEthernet0/18 switchport
{% endhighlight %}

#### VLAN's Verwijderen

Verwijder een individueel VLAN met `no vlan vlan-id`. Zorg ervoor dat alle poorten die lid zijn van het te verwijderen VLAN, eerst aan een ander VLAN zijn toegewezen. Om alle VLAN's te verwijderen, kan het `vlan.dat`-bestand worden gewist, waarna de switch moet worden herladen.

**Cisco IOS & Cisco XE:**
{% highlight bash %}
Switch# configure terminal
Switch(config)# no vlan 20
Switch(config)# end
Switch# delete flash:vlan.dat
Switch# reload
{% endhighlight %}

### VLAN Trunk Configuratie op Cisco Switches (IOS & XE)

Het configureren van trunkpoorten is essentieel om VLAN's over meerdere switches te laten communiceren. Trunks zijn Layer 2-links die verkeer voor alle VLAN's dragen.

#### Trunkpoort Configureren

**Cisco IOS (Layer 2-switches zoals Catalyst 2960):**
{% highlight bash %}
Switch# configure terminal
Switch(config)# interface GigabitEthernet0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 99
Switch(config-if)# switchport trunk allowed vlan 10,20,30,99
Switch(config-if)# end
{% endhighlight %}

**Cisco XE (Layer 3-switches zoals Catalyst 3650 of nieuwer):**
Op Layer 3-switches moet de encapsulatiemethode expliciet worden geconfigureerd *voordat* de poort in trunk-modus wordt gezet.

{% highlight bash %}
Switch# configure terminal
Switch(config)# interface GigabitEthernet0/1
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 99
Switch(config-if)# switchport trunk allowed vlan 10,20,30,99
Switch(config-if)# end
{% endhighlight %}

#### Trunk Configuratie Verifiëren

Controleer de trunk-status met de volgende commando's:

**Cisco IOS & Cisco XE:**
{% highlight bash %}
Switch# show interfaces GigabitEthernet0/1 switchport
Switch# show interfaces trunk
{% endhighlight %}

#### Trunk naar Standaardinstellingen Terugzetten

Om een trunk terug te zetten naar de standaardinstellingen (alle VLAN's toegestaan, Native VLAN = 1), gebruikt u de `no`-commando's:

**Cisco IOS & Cisco XE:**
{% highlight bash %}
Switch# configure terminal
Switch(config)# interface GigabitEthernet0/1
Switch(config-if)# no switchport trunk native vlan
Switch(config-if)# no switchport trunk allowed vlan
Switch(config-if)# end
{% endhighlight %}

U kunt ook een trunkpoort terugzetten naar een access-modus met `switchport mode access`.

### Dynamic Trunking Protocol (DTP)

**DTP (Dynamic Trunking Protocol)** is een Cisco-propriëtair protocol dat beheert of een interface een trunk-link wordt en welke modus de trunk-link aanneemt. Op Catalyst 2960 en 2950 switches is DTP standaard ingeschakeld, met de modus `dynamic auto`. DTP kan worden uitgeschakeld met de `nonegotiate` opdracht.

DTP biedt verschillende onderhandelingsmodi:
*   **access**: Permanente access-modus. Deze onderhandelt om de naburige link om te zetten naar een access-link.
*   **dynamic auto**: De interface wordt een trunk als de buurpoort is ingesteld op `trunk` of `desirable`-modus.
*   **dynamic desirable**: De interface probeert actief een trunk te worden door te onderhandelen met andere `auto` of `desirable` interfaces.
*   **trunk**: Permanente trunking-modus. Deze onderhandelt om de naburige link om te zetten naar een trunk-link.

**Cisco's best practice is om DTP uit te schakelen en trunk- of access-interfaces statisch te configureren**. Dit voorkomt onverwachte onderhandelingen, verhoogt de beveiliging en zorgt voor een voorspelbare netwerkconfiguratie. Onbedoelde trunk-verbindingen kunnen beveiligingsrisico's opleveren door ongeoorloofde toegang tot verschillende VLAN's.

#### DTP Uitschakelen of Statisch Instellen

**Cisco IOS & Cisco XE:**
{% highlight bash %}
Switch# configure terminal
Switch(config)# interface GigabitEthernet0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport nonegotiate
Switch(config-if)# end
{% endhighlight %}
*Met `switchport nonegotiate` wordt DTP uitgeschakeld voor die interface. De `switchport mode trunk` opdracht zet de interface in permanente trunking modus. Het statisch instellen van de modus voorkomt onderhandelingsproblemen.*

#### DTP-modus Verifiëren

**Cisco IOS & Cisco XE:**
{% highlight bash %}
Switch# show dtp interface GigabitEthernet0/1
{% endhighlight %}

### Andere Leveranciers (HPE, Juniper)

De beschikbare bronnen zijn uitsluitend gericht op Cisco-technologieën en bevatten geen specifieke configuratievoorbeelden of details over de implementatie van VLAN's en trunks op apparatuur van andere leveranciers zoals HPE of Juniper. Hoewel de onderliggende standaard (IEEE 802.1Q) universeel is, zullen de commando's, syntax en specifieke protocollen (zoals DTP, wat een Cisco-propriëtair protocol is) verschillen per vendor. Raadpleeg de documentatie van de desbetreffende leverancier voor details over hun specifieke implementatie.
*Deze informatie over andere leveranciers is niet afkomstig uit de gegeven bronnen en kan onafhankelijke verificatie vereisen.*

### Conclusie

**VLAN's en trunks** zijn fundamentele concepten in de moderne netwerkinfrastructuur die aanzienlijke voordelen bieden op het gebied van beveiliging, prestaties en beheer. Door een goed begrip van de verschillende VLAN-typen, de werking van 802.1Q-tagging en de juiste configuratie van access- en trunkpoorten – inclusief de best practices rondom DTP – kunnen netwerkbeheerders robuuste en efficiënte netwerken ontwerpen en onderhouden. Het consequent toepassen van deze principes draagt bij aan een stabiel en goed presterend netwerk.