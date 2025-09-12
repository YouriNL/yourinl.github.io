---
layout: post
title: "VLAN's Ontrafeld: Logische Netwerksegmentatie voor Moderne Infrastructuur"
date: 2023-10-27 10:00:00 +0100
categories: [Netwerken, VLAN, Trunking, Netwerkbeheer]
tags: [VLAN, Trunking, DTP, Netwerksegmentatie, 802.1Q, Netwerkbeveiliging]
---

**Virtuele Local Area Networks (VLAN's)** vormen de ruggengraat van moderne, efficiënte en veilige netwerkinfrastructuren. Ze stellen organisaties in staat om hun netwerken logisch te segmenteren, onafhankelijk van fysieke locatie, wat cruciaal is voor schaalbaarheid en beheer. Deze blogpost duikt diep in de concepten van VLAN's, hun typen, configuratie en gerelateerde technologieën zoals trunking en Dynamic Trunking Protocol (DTP), gebaseerd op uitgebreide materialen over netwerkfundamentals [1-6].

## Wat zijn VLAN's?

Een VLAN is een **logische verbinding** die apparaten met vergelijkbare behoeften groepeert, ongeacht waar ze fysiek op de switch zijn aangesloten [4, 7-9]. Denk aan VLAN's als virtuele muren die verschillende afdelingen of typen verkeer van elkaar scheiden binnen één fysiek netwerk [8, 10].

De voordelen van VLAN's zijn significant:
*   **Kleinere Broadcast-domeinen**: Door het netwerk op te delen, vermindert het de omvang van broadcastverkeer. Unicasts, multicasts en broadcasts blijven geïsoleerd binnen hun individuele VLAN, wat de netwerkprestaties verbetert en onnodig verkeer vermindert [4, 7-9, 11].
*   **Verbeterde Beveiliging**: Alleen gebruikers binnen hetzelfde VLAN kunnen direct met elkaar communiceren. Dit beperkt de toegang tot gevoelige informatie en verhoogt de netwerkbeveiliging [4, 8, 9, 11].
*   **Verbeterde IT-efficiëntie en Eenvoudiger Beheer**: Apparaten met vergelijkbare behoeften (bijv. studenten versus docenten, of IP-telefoons versus computers) kunnen logisch worden gegroepeerd. Dit vereenvoudigt het beheer en de toewijzing van resources [4, 8, 9, 11].
*   **Lagere Kosten**: Eén enkele switch kan meerdere groepen of VLAN's ondersteunen, waardoor de behoefte aan aparte fysieke switches voor elke afdeling afneemt [4, 8, 9, 11].
*   **Unieke IP-adressering**: Elk VLAN heeft zijn eigen unieke IP-adresbereik [4, 7-9].

Het is belangrijk te onthouden dat Layer 2-VLAN's wel broadcast-domeinen creëren, maar **geen verkeer kunnen routeren** tussen deze groepen. Hiervoor is een Layer 3-apparaat (zoals een router of een Layer 3-switch) nodig [8, 10, 12, 13].

## Typen VLAN's

Verschillende typen VLAN's dienen specifieke doeleinden binnen een netwerkinfrastructuur [14-17]:

*   **Standaard-VLAN (VLAN 1)**: Dit is het standaard-VLAN voor het hele netwerk. Het fungeert ook als het standaard Native VLAN en Management VLAN. VLAN 1 kan niet worden verwijderd of hernoemd [14-17]. Hoewel functioneel, raden best practices aan om deze standaardfuncties toe te wijzen aan andere, specifieke VLAN's [3, 14-18].
*   **Data-VLAN**: Dit VLAN is bestemd voor door gebruikers gegenereerd verkeer, zoals e-mail en webverkeer [15-17, 19]. Standaard is VLAN 1 vaak het Data-VLAN, aangezien alle poorten hieraan zijn toegewezen [15-17, 19].
*   **Native VLAN**: Dit type VLAN wordt uitsluitend gebruikt voor trunk-verbindingen [15-17, 19]. Op een 802.1Q trunk-link worden alle frames getagd, behalve die van het Native VLAN [13, 19-22]. Het is cruciaal dat **beide uiteinden van een trunk-link met hetzelfde Native VLAN zijn geconfigureerd** om communicatieproblemen en foutmeldingen te voorkomen [3, 13, 20-23].
*   **Management-VLAN**: Dit VLAN wordt gebruikt voor beheerdoeleinden, zoals toegang tot de switch via SSH of Telnet (VTY-verkeer) [15-17, 19]. Het moet gescheiden worden gehouden van eindgebruikersverkeer [15-17, 19].
*   **Voice-VLAN**: Een apart VLAN is essentieel voor spraakverkeer (Voice over IP - VoIP) vanwege de strikte eisen die dit type verkeer stelt [15-17, 24]:
    *   Gegarandeerde bandbreedte [15-17, 24].
    *   Hoge QoS (Quality of Service) prioriteit [15-17, 24].
    *   Het vermijden van congestie [15-17, 24].
    *   Een vertraging van minder dan 150 ms van bron tot bestemming [15-17, 24].
    Spraak- en dataverkeer moeten gescheiden worden om 'TCP starvation' te voorkomen, waarbij UDP-spraakverkeer prioriteit krijgt en TCP-dataverkeer wordt onderdrukt [16, 25, 26]. Een VoIP-telefoon tagt doorgaans zijn eigen verkeer en kan CoS (Cost of Service) instellen [13, 22, 27, 28]. De telefoon zal falen om op te starten als het een IP-adres voor het datanetwerk krijgt in plaats van voor spraak [28, 29].

## VLAN Bereiken op Netwerkswitches

Moderne netwerkswitches ondersteunen vaak duizenden VLAN's, verdeeld over verschillende bereiken [30-33]:

*   **Normal Range VLAN's (1 – 1005)**:
    *   Vaak gebruikt in kleine tot middelgrote bedrijven [30-33].
    *   VLAN 1, en nummers 1002 – 1005 (gereserveerd voor legacy-VLAN's), worden automatisch aangemaakt en kunnen niet worden verwijderd [30-33].
    *   Configuratiegegevens worden doorgaans opgeslagen in een speciaal bestand in het flashgeheugen van de switch (bijv. `vlan.dat` op Cisco-apparaten) [30-34].
    *   VLAN Trunking Protocol (VTP) kan deze synchroniseren tussen switches [30-33].
*   **Extended Range VLAN's (1006 - 4095)**:
    *   Meestal gebruikt door serviceproviders voor grotere netwerken [30-33].
    *   Deze VLAN's worden direct opgeslagen in de actieve configuratie (running-config) [30-33].
    *   Ondersteunen vaak minder VLAN-functies en vereisen specifieke VTP-configuraties (of de switch in VTP transparante modus te zetten) [30-33].

## VLAN Trunks

Een **trunk** is een point-to-point-verbinding tussen twee netwerkapparaten (meestal switches) die het mogelijk maakt om **verkeer voor meer dan één VLAN** over één fysieke link te transporteren [13, 21, 22, 35]. Dit is essentieel; de "legacy"-methode verbruikte voor elk VLAN aparte access-poorten tussen switches, wat onpraktisch was [18, 21, 36].

### 802.1Q Trunking en Tagging

**Tagging is cruciaal** voor de werking van trunking [13, 21, 22, 25, 37]. De IEEE 802.1Q-standaard definieert een 4-bytes header die wordt toegevoegd aan Ethernet-frames die over een trunk worden verzonden [13, 21, 22, 37]. Deze tag bevat belangrijke informatie [13, 21, 22, 37]:
*   **Tag Protocol ID (TPID)**: Een 2-byte veld met de hexadecimale waarde 0x8100, wat aangeeft dat het frame een 802.1Q-tag bevat [13, 21, 22, 37].
*   **User Priority**: Een 3-bits waarde die Layer 2 Quality of Service (QoS) prioriteit ondersteunt [13, 21, 22, 37].
*   **Canonical Format Identifier (CFI)**: Een 1-bits waarde die token ring frames op Ethernet kan ondersteunen [13, 21, 22, 37].
*   **VLAN ID (VID)**: Een 12-bits VLAN-identifier, die tot 4096 VLAN's kan ondersteunen [13, 21, 22, 37].

Wanneer een tag wordt toegevoegd, moet de Frame Check Sequence (FCS) opnieuw worden berekend [13, 21, 37]. Bij verzending naar eindapparaten moet deze tag worden verwijderd en de FCS teruggebracht naar zijn oorspronkelijke waarde [13, 21, 37]. De 802.1Q-standaard heeft de voorkeur boven oudere, zoals ISL (een propriëtair protocol van één leverancier), mede vanwege de ondersteuning voor QoS [3, 25]. Frames die behoren tot het **Native VLAN** worden *niet* getagd op een 802.1Q trunk-link [13, 19-22].

## Dynamic Trunking Protocol (DTP)

Het **Dynamic Trunking Protocol (DTP)** is een **propriëtair protocol** van een specifieke netwerkleverancier (Cisco) dat trunk-onderhandelingen beheert [3, 38-42]. Hoewel het configureren kan vereenvoudigen, raden best practices aan om trunk- of access-interfaces **statisch in te stellen** en DTP uit te schakelen om onderhandelingsproblemen te voorkomen [3, 38, 41-44]. Dit voorkomt onbedoelde trunk-links of communicatieproblemen wanneer de configuraties aan beide zijden van een link niet overeenkomen [43, 45].

DTP biedt verschillende onderhandelingsmodi [41, 42, 44, 46]:
*   `access`: Permanente access-modus. Probeert de naburige link om te zetten naar een access-link [41, 42, 44, 46].
*   `dynamic auto`: Wordt een trunk als de naburige interface is ingesteld op `trunk` of `desirable` modus [41, 42, 44, 46].
*   `dynamic desirable`: Zoekt actief naar een trunk door te onderhandelen met andere `auto` of `desirable` interfaces [41, 42, 44, 46].
*   `trunk`: Permanente trunking-modus. Probeert de naburige link om te zetten naar een trunk-link [41, 42, 44, 46].

De standaardinstelling voor DTP kan variëren per switchmodel en softwareversie. Het kan worden uitgeschakeld met specifieke commando's (bijv. `nonegotiate` op Cisco-apparatuur) [38, 41, 42, 44].

## VLAN Configuratie & Beheer

Het aanmaken en toewijzen van VLAN's is een kernvaardigheid in netwerkbeheer. Hoewel de precieze commando's per leverancier verschillen, is het onderliggende proces vergelijkbaar [32-34, 47-52].

### Belangrijke Opmerking over Configuratievoorbeelden
De onderstaande commando's zijn voorbeelden gebaseerd op de Cisco IOS Command Line Interface (CLI). De syntaxis kan aanzienlijk verschillen bij switches van andere leveranciers. Raadpleeg altijd de documentatie van uw specifieke netwerkapparatuur.

### VLAN's Aanmaken
VLAN-details worden doorgaans opgeslagen in een speciaal configuratiebestand op de switch [32-34, 49].
Switch# configure terminal Switch(config)# vlan <vlan-id> Switch(config-vlan)# name <vlan-naam> Switch(config-vlan)# end
Bijvoorbeeld, voor VLAN 20 genaamd 'student' [32, 49, 53]:
S1# configure terminal S1(config)# vlan 20 S1(config-vlan)# name student S1(config-vlan)# end

### Poorten Toewijzen aan VLAN's
Zodra een VLAN is aangemaakt, wijs je de fysieke interfaces (poorten) toe aan het juiste VLAN [32, 33, 47, 49]. Een poort die aan een eindapparaat (zoals een computer) is gekoppeld, wordt een 'access-poort' genoemd en kan slechts aan één Data-VLAN worden toegewezen [32, 33, 49, 54]. Echter, het kan ook aan één Voice-VLAN worden toegewezen wanneer een IP-telefoon met ingebouwde switch op dezelfde poort wordt aangesloten [32, 33, 49, 54].
Switch# configure terminal Switch(config)# interface <interface-id> Switch(config-if)# switchport mode access  # Zet de poort in access-modus Switch(config-if)# switchport access vlan <vlan-id> # Wijs de poort toe aan een Data-VLAN Switch(config-if)# end

### Trunk-links Configureren
Trunk-poorten zijn essentieel om VLAN's tussen switches te transporteren [48, 50-52].
Switch# configure terminal Switch(config)# interface <interface-id> Switch(config-if)# switchport mode trunk # Zet de poort in permanente trunking modus Switch(config-if)# switchport trunk native vlan <vlan-id> # Stelt het Native VLAN in Switch(config-if)# switchport trunk allowed vlan <vlan-lijst> # Specificeert toegestane VLAN's Switch(config-if)# end
Bijvoorbeeld, voor een trunk-poort op interface fa0/1 met Native VLAN 99 en toegestane VLAN's 10, 20, 30, 99 [50, 51, 55]:
S1(config)# interface fa0/1 S1(config-if)# switchport mode trunk S1(config-if)# switchport trunk native vlan 99 S1(config-if)# switchport trunk allowed vlan 10,20,30,99 S1(config-if)# end

### VLAN-informatie Verifiëren
Gebruik verificatiecommando's om VLAN-informatie en de status van poorten te controleren [32, 33, 49-52, 56-58]:
show vlan [brief | id <vlan-id> | name <vlan-naam> | summary] show interfaces <interface-id> switchport show interfaces trunk
`show vlan brief` toont een overzicht van VLAN-namen, -statussen en bijbehorende poorten [32, 33, 49, 56, 59]. `show interfaces <interface-id> switchport` kan details tonen over de toegewezen data- en voice-VLAN's en trunk-status [22, 32, 33, 50, 52, 57-60]. `show interfaces trunk` geeft een overzicht van alle trunk-poorten en hun configuratie [50-52, 59].

### VLAN's Verwijderen
VLAN's kunnen individueel worden verwijderd. Het is een best practice om alle poorten die lid waren van de te verwijderen VLAN, eerst opnieuw toe te wijzen aan een ander VLAN [32, 33, 49, 61].
no vlan <vlan-id>
Om alle VLAN's te verwijderen, kunt u het configuratiebestand wissen (bijv. `delete flash:vlan.dat` of `delete vlan.dat`) en vervolgens de switch opnieuw opstarten [32, 33, 49, 61].

## Belangrijke Best Practices

*   **VLAN 1 Vermijden**: Hoewel het standaard werkt, is het een goede gewoonte om VLAN 1 niet te gebruiken voor gebruikersdata of managementverkeer. Maak dedicated VLAN's aan voor deze doeleinden [3, 14-18].
*   **Native VLAN Mismatches**: Zorg ervoor dat het Native VLAN aan beide zijden van een trunk-link hetzelfde is. Dit voorkomt communicatieproblemen [3, 13, 20-23].
*   **Voice VLAN Implementatie**: Scheid spraak- en dataverkeer. Een VoIP-telefoon wordt meestal aangesloten op een access-interface waaraan zowel een data- als een voice-VLAN is toegewezen [13, 22, 25-28, 54].
*   **Statische Trunk- en Access-configuratie**: Stel interfaces **statisch in** als access of trunk en schakel DTP uit (indien aanwezig op uw apparatuur) om onderhandelingsproblemen te voorkomen [3, 38, 41-44].

## Conclusie

VLAN's, trunking en de bijbehorende protocollen zijn fundamentele concepten voor iedereen die betrokken is bij netwerkontwerp en -beheer. Door netwerken logisch te segmenteren, kunnen organisaties profiteren van verbeterde beveiliging, betere prestaties en vereenvoudigd beheer. Het correct toepassen van deze concepten, inclusief de bewustwording van vendor-specifieke implementaties en best practices, is essentieel voor het bouwen van robuuste en efficiënte netwerkinfrastructuren [62-64]. Het uitvoeren van praktische oefeningen met netwerksimulatietools zoals Packet Tracer helpt enorm bij het beheersen van deze concepten [59-61, 65-73].