---
layout: post
title: "Dynamische IPv6 Adresallocatie: SLAAC en DHCPv6 op Cisco Platforms"
date: 2023-10-26
author: NetwerkExpert
tags: [IPv6, SLAAC, DHCPv6, Cisco IOS, Netwerkconfiguratie]
---

# Dynamische IPv6 Adresallocatie: SLAAC en DHCPv6 op Cisco Platforms

In de hedendaagse netwerkomgevingen is IPv6 steeds prominenter aanwezig. Het handmatig configureren van IPv6 Global Unicast Adressen (GUA's) op hosts is **tijdrovend en foutgevoelig**. Daarom is de dynamische toewijzing van IPv6-adressen essentieel voor efficiënt netwerkbeheer. Deze blogpost duikt dieper in de twee primaire methoden voor dynamische IPv6-adresallocatie: **Stateless Address Autoconfiguration (SLAAC)** en **Dynamic Host Configuration Protocol for IPv6 (DHCPv6)**, met een focus op configuratie op Cisco IOS-routers.

## De Rol van Router Advertisement (RA) Berichten

Voordat we de details induiken, is het cruciaal om het concept van **ICMPv6 Router Advertisement (RA) berichten** te begrijpen. Een IPv6-router stuurt standaard periodiek RA-berichten uit om hosts te helpen bij het automatisch configureren van hun IPv6-configuratie. Hosts kunnen ook een **Router Solicitation (RS) bericht** sturen om direct een RA te vragen, bijvoorbeeld bij het opstarten.

De wijze waarop een client een IPv6 GUA verkrijgt, wordt beïnvloed door drie cruciale vlaggen in het ICMPv6 RA-bericht:

*   **A-vlag (Address Autoconfiguration flag):** Wanneer deze op 1 is ingesteld, geeft dit aan dat **SLAAC** moet worden gebruikt om een IPv6 GUA aan te maken.
*   **O-vlag (Other Configuration flag):** Wanneer deze op 1 is ingesteld, geeft dit aan dat **aanvullende informatie** beschikbaar is van een stateless DHCPv6-server.
*   **M-vlag (Managed Address Configuration flag):** Wanneer deze op 1 is ingesteld, geeft dit aan dat een **stateful DHCPv6-server** moet worden gebruikt om een IPv6 GUA te verkrijgen.

Hoewel besturingssystemen de suggesties van het RA-bericht volgen, ligt de uiteindelijke beslissing bij de host zelf.

## 1. Stateless Address Autoconfiguration (SLAAC)

SLAAC is een **stateless dienst** die hosts in staat stelt hun eigen unieke IPv6 GUA te creëren zonder de diensten van een DHCPv6-server. Dit betekent dat er geen server is die adresinformatie bijhoudt over welke IPv6-adressen in gebruik zijn. Hosts verkrijgen de 64-bits IPv6-subnetinformatie van het router-RA.

De resterende 64-bits **Interface ID** wordt op een van de volgende manieren gegenereerd:

*   **Willekeurig gegenereerd:** Het 64-bits Interface ID wordt willekeurig gegenereerd door het client-besturingssysteem (standaard voor Windows 10). Dit heeft de voorkeur vanwege privacyoverwegingen.
*   **EUI-64:** De host maakt een Interface ID aan met behulp van zijn 48-bits MAC-adres en voegt de hexadecimale waarde 'fffe' in het midden van het adres in. Deze methode wordt minder vaak gebruikt vanwege privacyoverwegingen, aangezien het MAC-adres deel uitmaakt van het GUA.

### Duplicate Address Detection (DAD)

Om de uniciteit van een gegenereerd IPv6 GUA te garanderen, voert een SLAAC-host **Duplicate Address Detection (DAD)** uit. De host stuurt een ICMPv6 **Neighbor Solicitation (NS) bericht** met een speciaal geconstrueerd "solicited-node multicast address". Als er geen andere apparaten reageren met een **Neighbor Advertisement (NA) bericht**, wordt het adres als uniek beschouwd. Hoewel de kans op een duplicaatadres extreem klein is (een 64-bits Interface ID biedt 18 quintiljoen mogelijkheden), raadt de IETF DAD aan, en de meeste besturingssystemen voeren het uit op alle IPv6 unicast-adressen.

### Cisco IOS Configuratie voor SLAAC (SLAAC-alleen)

De SLAAC-alleen methode is de standaardconfiguratie op een Cisco IOS-router wanneer `ipv6 unicast-routing` is ingeschakeld en een GUA op een interface is geconfigureerd. De A-vlag wordt automatisch op 1 gezet, terwijl de O- en M-vlaggen op 0 blijven.

Om IPv6-routing in te schakelen en een IPv6 GUA te configureren op een interface:

{% highlight bash %}
R1(config)# ipv6 unicast-routing
R1(config)# interface GigabitEthernet0/0/1
R1(config-if)# ipv6 address 2001:db8:acad:1::1/64
R1(config-if)# no shutdown
{% endhighlight %}

Deze configuratie zorgt ervoor dat de router periodiek RA-berichten stuurt met de A-vlag op 1, wat hosts instrueert om SLAAC te gebruiken. De router zal ook een **Link-Local Adres (LLA)** aanmaken, wat essentieel is voor IPv6-communicatie binnen het lokale segment.

## 2. DHCPv6: Stateless vs. Stateful

DHCPv6 wordt gebruikt wanneer een host volledige of aanvullende configuratie-informatie van een server nodig heeft. De communicatie tussen client en server verloopt via een reeks stappen: RS, RA, SOLICIT, ADVERTISE, REPLY. Hierbij gebruikt de client UDP-bestemmingspoort 547 en de server UDP-bestemmingspoort 546.

### Stateless DHCPv6 (A=1, O=1, M=0)

Bij Stateless DHCPv6 verkrijgt de host zijn IPv6 GUA via SLAAC (gebruikmakend van de prefix in het RA-bericht), maar neemt vervolgens contact op met een DHCPv6-server voor **aanvullende informatie** zoals DNS-servers en domeinnamen. De DHCPv6-server in dit scenario is **stateless**; het houdt geen lijst bij van toegewezen IPv6-adresbindingen.

#### Cisco IOS Server Configuratie voor Stateless DHCPv6

1.  **Schakel IPv6-routing in:**
    {% highlight bash %}
    R1(config)# ipv6 unicast-routing
    {% endhighlight %}
2.  **Definieer een DHCPv6-pool:**
    {% highlight bash %}
    R1(config)# ipv6 dhcp pool STATELESS_POOL
    {% endhighlight %}
3.  **Configureer pool-opties (bijv. DNS-server, domeinnaam):**
    {% highlight bash %}
    R1(config-dhcpv6)# dns-server 2001:db8::53
    R1(config-dhcpv6)# domain-name example.com
    {% endhighlight %}
4.  **Bind de interface aan de pool en stel de O-vlag in:** De `ipv6 nd other-config-flag` opdracht zet de O-vlag op 1. De A-vlag is standaard 1, wat SLAAC toestaat.
    {% highlight bash %}
    R1(config)# interface GigabitEthernet0/0/1
    R1(config-if)# ipv6 address 2001:db8:acad:1::1/64
    R1(config-if)# ipv6 dhcp server STATELESS_POOL
    R1(config-if)# ipv6 nd other-config-flag
    {% endhighlight %}

#### Cisco IOS Client Configuratie voor Stateless DHCPv6

Een router kan ook fungeren als een DHCPv6-client. Om een Cisco IOS-router als stateless DHCPv6-client te configureren:

1.  **Schakel IPv6-routing in:**
    {% highlight bash %}
    R2(config)# ipv6 unicast-routing
    {% endhighlight %}
2.  **Creëer een LLA (indien nog niet aanwezig):** Cisco IOS gebruikt EUI-64 om de Interface ID voor het LLA te creëren.
    {% highlight bash %}
    R2(config)# interface GigabitEthernet0/0/1
    R2(config-if)# ipv6 enable
    {% endhighlight %}
3.  **Configureer om SLAAC te gebruiken:**
    {% highlight bash %}
    R2(config-if)# ipv6 address autoconfig
    {% endhighlight %}

### Stateful DHCPv6 (A=0, O=0, M=1)

Bij Stateful DHCPv6 neemt de host contact op met een DHCPv6-server voor **alle configuratie-informatie**, inclusief het IPv6 GUA zelf. De DHCPv6-server is hier **stateful** en onderhoudt een lijst met toegewezen IPv6-adresbindingen.

#### Cisco IOS Server Configuratie voor Stateful DHCPv6

1.  **Schakel IPv6-routing in:**
    {% highlight bash %}
    R1(config)# ipv6 unicast-routing
    {% endhighlight %}
2.  **Definieer een DHCPv6-pool:**
    {% highlight bash %}
    R1(config)# ipv6 dhcp pool STATEFUL_POOL
    {% endhighlight %}
3.  **Configureer pool-opties (adresprefix, DNS-server, domeinnaam):**
    {% highlight bash %}
    R1(config-dhcpv6)# address prefix 2001:db8:acad:2::/64
    R1(config-dhcpv6)# dns-server 2001:db8::54
    R1(config-dhcpv6)# domain-name secure.example.com
    {% endhighlight %}
4.  **Bind de interface aan de pool en stel de M- en A-vlaggen in:** De `ipv6 nd managed-config-flag` opdracht zet de M-vlag op 1. De `ipv6 nd prefix default no-autoconfig` opdracht zet de A-vlag op 0, wat hosts vertelt geen SLAAC te gebruiken.
    {% highlight bash %}
    R1(config)# interface GigabitEthernet0/0/1
    R1(config-if)# ipv6 dhcp server STATEFUL_POOL
    R1(config-if)# ipv6 nd managed-config-flag
    R1(config-if)# ipv6 nd prefix default no-autoconfig
    {% endhighlight %}

#### Cisco IOS Client Configuratie voor Stateful DHCPv6

Om een Cisco IOS-router als stateful DHCPv6-client te configureren:

1.  **Schakel IPv6-routing in:**
    {% highlight bash %}
    R2(config)# ipv6 unicast-routing
    {% endhighlight %}
2.  **Creëer een LLA (indien nog niet aanwezig):**
    {% highlight bash %}
    R2(config)# interface GigabitEthernet0/0/1
    R2(config-if)# ipv6 enable
    {% endhighlight %}
3.  **Configureer om DHCPv6 te gebruiken:**
    {% highlight bash %}
    R2(config-if)# ipv6 address dhcp
    {% endhighlight %}

## 3. DHCPv6 Relay Agent

Een DHCPv6 Relay Agent is noodzakelijk wanneer de DHCPv6-server zich op een ander netwerksegment bevindt dan de DHCPv6-clients. Zonder een relay agent zouden de DHCPv6-verzoeken van clients de server niet bereiken, aangezien DHCPv6-berichten standaard niet router-overschrijdend worden doorgestuurd.

#### Cisco IOS Configuratie voor DHCPv6 Relay Agent

De opdracht wordt geconfigureerd op de interface die naar de DHCPv6-clients wijst en specificeert het IPv6-adres van de DHCPv6-server. De egress-interface is alleen vereist wanneer het next-hop-adres een Link-Local Adres (LLA) is.

{% highlight bash %}
R1(config)# interface GigabitEthernet0/0/2
R1(config-if)# ipv6 dhcp relay destination 2001:db8:relay::1
{% endhighlight %}

## Verificatiecommando's (Cisco IOS)

Voor het controleren van de DHCPv6-werking op Cisco IOS-routers zijn diverse commando's beschikbaar:

*   **`show ipv6 dhcp pool`**: Verifieert de naam en parameters van de DHCPv6-pool en het aantal actieve clients.
*   **`show ipv6 dhcp binding`**: Toont de link-local adressen van clients en de toegewezen GUA's. Deze informatie wordt alleen bijgehouden door een stateful DHCPv6-server.
*   **`show ipv6 dhcp interface [interface-type interface-number]`**: Toont DHCPv6-informatie voor een specifieke interface, zoals ontvangen opties.
*   **`show ipv6 interface brief`**: Verifieert de toegewezen IPv6 GUA's op routerinterfaces.
*   **`ipconfig /all`** (op Windows hosts): Verifieert de ontvangen IPv6-adresseringsinformatie op de client.

## Cisco IOS vs. Cisco XE en Andere Vendors

De verstrekte bronnen zijn specifiek gericht op de "Switching, Routing and Wireless Essentials v7.0 (SRWE) - Instructor Materials" van Cisco, en refereren consistent aan **"Cisco IOS-routers"**. Er worden **geen expliciete verschillen** tussen Cisco IOS en Cisco XE benoemd, noch worden configuratievoorbeelden voor Cisco XE gegeven in deze materialen. De hierboven beschreven commando's en configuratiestappen zijn derhalve van toepassing op Cisco IOS.

Verder bevatten de beschikbare bronnen **geen informatie** over hoe andere netwerkleveranciers, zoals HPE of Juniper, SLAAC en DHCPv6 implementeren. De focus ligt volledig op de Cisco-specifieke benadering van deze protocollen.

## Conclusie

De dynamische toewijzing van IPv6-adressen via SLAAC en DHCPv6 is een fundamenteel aspect van modern netwerkbeheer. Het correct configureren van deze methoden op Cisco IOS-routers stelt beheerders in staat om flexibele, schaalbare en eenvoudig te beheren IPv6-netwerken te creëren. Het begrijpen van de nuances tussen SLAAC, stateless DHCPv6 en stateful DHCPv6, inclusief de rol van de A-, O- en M-vlaggen in RA-berichten, is cruciaal voor het implementeren van de juiste adresseringsstrategie voor elke netwerkomgeving.

Door de juiste combinatie van deze technieken te kiezen, kunnen organisaties profiteren van de efficiëntie en robuustheid die IPv6 biedt, terwijl de administratieve last wordt geminimaliseerd.