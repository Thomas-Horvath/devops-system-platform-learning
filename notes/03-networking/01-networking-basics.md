# Networking Basics

## 1. Mi az a network?

A **network = hálózat** olyan eszközök összessége, amelyek képesek egymással adatot cserélni.

Egy egyszerű otthoni hálózatban lehet például:

```text
Laptop
Telefon
TV
Nyomtató
NAS
Router
```

A legalapvetőbb kommunikáció:

```text
sender
↓
network
↓
receiver
```

A hálózat célja tehát:

> Eszközök összekapcsolása és az egymás közötti kommunikáció lehetővé tétele.

### English

> A network connects devices so they can communicate.

---

# 2. Host

A **host** egy hálózathoz csatlakozó eszköz vagy rendszer.

Host lehet például:

```text
laptop
server
phone
printer
virtual machine
container
NAS
```

Egy hostnak tipikusan van:

```text
network interface
+
IP address
```

Egyszerű mental model:

```text
host
→ hálózaton kommunikáló eszköz
```

---

# 3. Client és Server

A **client** és a **server** nem feltétlenül külön gépet jelent.

Ezek szerepek.

## Client

A client valamilyen szolgáltatást vagy erőforrást kér.

Például egy böngésző:

```text
Chrome
```

lehet client.

## Server

A server fogadja a kérést és választ küld.

```text
Client
↓ request
Server
↓ response
Client
```

Webes példa:

```text
Browser
↓
HTTP request
↓
Web server
↓
HTTP response
```

Akár ugyanazon a gépen is futhat mindkettő:

```text
Browser
→ client

Node.js application
→ server
```

### English

> The client sends a request.

> The server sends a response.

---

# 4. LAN

**LAN = Local Area Network**

Egy kisebb, helyi hálózat.

Például:

```text
otthoni hálózat
irodai hálózat
egyetemi hálózat
```

Egyszerű példa:

```text
Laptop ─┐
PC     ─┼── Switch / Router
Printer─┘
```

Egy otthoni LAN-ban lehet például:

```text
Router     192.168.1.1
Laptop     192.168.1.10
PC         192.168.1.20
Printer    192.168.1.30
NAS        192.168.1.40
```

A LAN nem egyszerűen azt jelenti, hogy az eszközök fizikailag egy szobában vannak.

Inkább egy helyi hálózati környezet.

---

# 5. WAN

**WAN = Wide Area Network**

Nagyobb földrajzi területet vagy több különálló hálózatot összekötő hálózat.

A legismertebb példa:

```text
Internet
```

Egyszerű kép:

```text
Home LAN
↓
Router
↓
ISP
↓
Internet
↓
Remote network
```

Az internet lényegében nagyon sok különböző hálózat összekapcsolása.

---

# 6. ISP

**ISP = Internet Service Provider**

Magyarul:

```text
internetszolgáltató
```

Az otthoni hálózat általában az ISP hálózatán keresztül kapcsolódik az internethez.

```text
Laptop
↓
Home router
↓
ISP
↓
Internet
```

---

# 7. Network Interface

A **network interface** az a hálózati csatlakozási pont, amin keresztül egy host kommunikál.

Lehet például:

```text
Ethernet interface
Wi-Fi interface
virtual interface
Docker interface
loopback interface
```

Linuxon például:

```bash
ip addr
```

paranccsal lehet majd ezeket megvizsgálni.

Interface-nevek lehetnek például:

```text
eth0
ens3
wlan0
lo
```

Mental model:

```text
Application
↓
Operating System
↓
Network Interface
↓
Network
```

---

# 8. MAC address

A **MAC address** egy hálózati interface helyi hálózati azonosítója.

Példa:

```text
00:1A:2B:3C:4D:5E
```

Nagyon leegyszerűsítve:

```text
IP address
→ logikai hálózati cím

MAC address
→ helyi hálózati interface azonosító
```

Példa:

```text
PC A

IP:
192.168.1.10

MAC:
AA:AA:AA:AA:AA:AA
```

Másik gép:

```text
PC B

IP:
192.168.1.20

MAC:
BB:BB:BB:BB:BB:BB
```

A MAC-cím főleg a helyi hálózaton fontos.

---

# 9. ARP

**ARP = Address Resolution Protocol**

Az ARP egyik alapfeladata:

```text
IP address
↓
ARP
↓
MAC address
```

Tegyük fel:

```text
PC A:
192.168.1.10
```

adatot akar küldeni:

```text
PC B:
192.168.1.20
```

PC A ismeri PC B IP-címét, de a helyi Ethernet-kommunikációhoz szüksége van a MAC-címére is.

Ezért lényegében megkérdezi:

```text
Who has 192.168.1.20?
```

PC B válaszol:

```text
192.168.1.20
→ BB:BB:BB:BB:BB:BB
```

Ezután PC A már tudja a cél MAC-címét.

---

# 10. ARP cache

A hostnak nem kell minden egyes adatküldés előtt újra megkérdeznie ugyanazt.

Az IP–MAC kapcsolatokat egy ideig eltárolhatja.

Például:

```text
192.168.1.1  → CC:CC:CC:CC:CC:CC
192.168.1.20 → BB:BB:BB:BB:BB:BB
```

Ez az:

```text
ARP cache
```

Linuxon később például:

```bash
ip neigh
```

paranccsal fogjuk vizsgálni.

---

# 11. IP address

Az **IP address** egy logikai hálózati cím.

IPv4 példa:

```text
192.168.1.10
```

Egyszerű mental model:

```text
IP
→ melyik hálózati cél?
```

Egy IPv4 cím összesen:

```text
32 bit
```

és négy darab 8 bites részre, úgynevezett **octetre** van bontva:

```text
192 . 168 . 1 . 10
```

Egy octet értéke:

```text
0–255
```

lehet.

Az IP-címek, subnet maskok, CIDR-ek és subnetek részletes működését külön jegyzetben bontjuk ki:

```text
02-ip-addressing-and-subnets.md
```

---

# 12. Private és Public IP

## Private IP

Belső hálózatokban használt címek.

Fontos private IPv4 tartományok:

```text
10.0.0.0/8

172.16.0.0/12

192.168.0.0/16
```

Példák:

```text
10.10.0.5
172.20.1.10
192.168.1.100
```

Ezek nem közvetlenül a nyilvános interneten használt címek.

---

## Public IP

Az interneten routolható cím.

Egyszerű kép:

```text
Laptop
private IP
↓
Router
↓
public IP
↓
Internet
```

A private és public címek közötti kapcsolatnál később fontos lesz:

```text
NAT
```

---

# 13. Loopback és localhost

A saját gépre mutató speciális IPv4 cím:

```text
127.0.0.1
```

A hozzá gyakran használt név:

```text
localhost
```

Például:

```text
localhost:3000
```

jelentése:

```text
localhost
→ saját gép

3000
→ port
```

Fejlesztés közben ezzel nagyon gyakran találkozunk.

---

# 14. Port

Az IP-cím megmondja:

> melyik hostot akarjuk elérni?

De ugyanazon a hoston több szolgáltatás is futhat.

Például:

```text
SSH server
web server
database
Node.js application
```

A **port** segít meghatározni:

> melyik szolgáltatást akarjuk elérni az adott hoston?

Mental model:

```text
IP
→ which host?

Port
→ which service?
```

Példa:

```text
192.168.1.20:3000
```

jelentése:

```text
192.168.1.20
→ host

3000
→ port / service
```

Gyakori portok:

```text
22    SSH
80    HTTP
443   HTTPS
3306  MySQL
5432  PostgreSQL
```

A portok, TCP/UDP és socketek részletes működését később külön bontjuk ki.

---

# 15. Protocol

A **protocol = kommunikációs szabályrendszer**.

Két rendszer akkor tud egymással értelmesen kommunikálni, ha ugyanazokat a szabályokat követik.

Hálózati protokoll például:

```text
Ethernet
ARP
IP
TCP
UDP
HTTP
HTTPS
DNS
SSH
DHCP
```

Ezek nem ugyanazt a feladatot végzik.

Például nagyon leegyszerűsítve:

```text
HTTP
↓
TCP
↓
IP
↓
Ethernet
```

A rétegek pontos működését később részletesebben kibontjuk.

---

# 16. Switch

A **switch** főleg egy helyi hálózaton kapcsolja össze az eszközöket.

Példa:

```text
PC A ──┐
       │
PC B ──┼── Switch
       │
NAS  ──┘
```

A switch elsősorban MAC-címekkel dolgozik.

Egyszerűsítve megtanulhatja például:

```text
MAC A → switch port 1
MAC B → switch port 2
MAC C → switch port 3
```

Ha PC A PC B-nek küld adatot:

```text
PC A
↓
Switch
↓
PC B
```

Mental model:

```text
Switch
→ local network
→ MAC addresses
```

---

# 17. Router

A **router** különböző hálózatokat köt össze.

Például:

```text
Network A
↓
Router
↓
Network B
```

Otthoni példában:

```text
Home LAN
↓
Router
↓
Internet
```

A router IP-címek és routing információ alapján dönti el, hogy egy packetet merre kell továbbítani.

Egyszerű mental model:

```text
Switch
→ hálózaton belül

Router
→ hálózatok között
```

A routing működését részletesen később vizsgáljuk:

```text
03-routing-gateway-nat.md
```

---

# 18. Gateway

A **gateway** az a hálózati pont, amelyen keresztül egy host egy másik hálózat felé tud kommunikálni.

Például:

```text
Laptop IP:
192.168.1.20

Default gateway:
192.168.1.1
```

Ha a cél nem a local hálózaton van:

```text
destination not local
↓
default gateway
↓
router
↓
remote network
```

---

# 19. Same subnet vs different subnet

Ez az egyik legfontosabb hálózati alapelv.

Legyen:

```text
PC A
192.168.1.10/24
```

és:

```text
PC B
192.168.1.20/24
```

Mindketten ugyanabba a hálózatba tartoznak:

```text
192.168.1.0/24
```

Ezért:

```text
same subnet
→ local communication
```

Egyszerűsítve:

```text
PC A
↓
Switch / LAN
↓
PC B
```

---

Másik eset:

```text
PC A
192.168.1.10/24
```

cél:

```text
Server
192.168.2.20/24
```

A két gép külön subnetben van.

Ezért:

```text
different subnet
↓
default gateway
↓
router
↓
remote network
```

Mental model:

```text
same subnet
→ local communication

different subnet
→ gateway/router
```

---

# 20. ARP local és remote kommunikációnál

Ez fontos különbség.

## Local destination

Ha a cél ugyanazon a subneten van:

```text
PC A
↓
ARP
↓
destination host MAC
↓
PC B
```

PC A a cél host MAC-címét keresi.

---

## Remote destination

Legyen a cél például:

```text
8.8.8.8
```

PC A megállapítja:

```text
8.8.8.8
→ nem local
```

Ilyenkor nem próbálja megszerezni a `8.8.8.8` MAC-címét.

Ehelyett:

```text
default gateway
192.168.1.1
```

MAC-címére van szüksége.

```text
Destination IP:
8.8.8.8

↓ not local

ARP:
Who has 192.168.1.1?

↓
router MAC
```

Nagyon fontos mental model:

```text
same subnet
→ destination host MAC

different subnet
→ default gateway MAC
```

---

# 21. DNS

**DNS = Domain Name System**

A DNS segítségével neveket használhatunk IP-címek helyett.

Példa:

```text
example.com
↓
DNS
↓
IP address
```

Mental model:

```text
DNS
→ name to IP resolution
```

A részletes DNS működést külön jegyzetben nézzük:

```text
04-dns.md
```

---

# 22. DHCP

**DHCP = Dynamic Host Configuration Protocol**

A DHCP automatikusan hálózati konfigurációt adhat egy hostnak.

Például:

```text
IP address
subnet mask
default gateway
DNS server
lease time
```

Otthoni példa:

```text
Laptop csatlakozik a Wi-Fihez
↓
DHCP
↓
kap hálózati konfigurációt
```

Például:

```text
IP:
192.168.1.50

Subnet:
/24

Gateway:
192.168.1.1

DNS:
192.168.1.1
```

Így ezeket általában nem kell kézzel beállítanunk.

---

# 23. DHCP DORA

A klasszikus DHCP folyamat rövidítése:

```text
DORA
```

Jelentése:

```text
D = Discover
O = Offer
R = Request
A = Acknowledge
```

Egyszerűsítve:

```text
Client:
Van DHCP server?

↓ Discover

DHCP server:
Ezt az IP-t tudom felajánlani.

↓ Offer

Client:
Ezt szeretném használni.

↓ Request

Server:
Rendben.

↓ Acknowledge
```

Nem kell most a DHCP minden technikai részletét megtanulni.

Az alapgondolat:

> DHCP automatikusan hálózati konfigurációt oszt.

---

# 24. DHCP és DNS különbsége

Ezt fontos különválasztani.

```text
DHCP
→ hálózati konfigurációt ad
```

```text
DNS
→ nevet old fel IP-címre
```

Egyszerű példa:

```text
DHCP:
"Te használd a 192.168.1.50 címet."

DNS:
"example.com IP-címe 203.0.113.10."
```

---

# 25. NAS

**NAS = Network Attached Storage**

Hálózatra csatlakoztatott adattároló.

Úgy képzelhető el, mint egy külön kis szerver háttértárakkal, amelyet több host is elérhet a hálózaton keresztül.

```text
Laptop ─┐
PC     ─┼── Switch / Router ── NAS
Phone  ─┘
```

A NAS-on lehet például:

```text
dokumentum
fotó
videó
backup
megosztott fájl
```

Egyszerű különbség:

```text
külső HDD
→ általában egy géphez kapcsolódik

NAS
→ hálózathoz kapcsolódik
→ több host is használhatja
```

A NAS saját IP-címmel is rendelkezhet:

```text
192.168.1.50
```

DevOps/System környezetben például backup célpontként is használható.

---

# 26. Network topology

A **network topology** azt mutatja meg, hogy a hálózat eszközei hogyan kapcsolódnak egymáshoz.

Például:

```text
             Internet
                │
             Router
                │
             Switch
          ┌─────┼─────┐
         PC    NAS   Server
```

Ez egy egyszerű hálózati topológia.

---

# 27. Star topology

A **star topology** esetén az eszközök egy központi eszközhöz kapcsolódnak.

Például:

```text
        PC
         |
Printer--Switch--NAS
         |
       Server
```

Modern LAN-okban ez nagyon gyakori.

A központi eszköz tipikusan switch.

---

# 28. Bus topology

A **bus topology** esetén az eszközök egy közös kommunikációs vonalat használnak.

Egyszerűsített kép:

```text
PC ─── PC ─── PC ─── Server
```

Régebbi hálózatokban volt gyakoribb.

Modern Ethernet LAN-okban már nem ez a jellemző topológia.

---

# 29. Ring topology

A **ring topology** esetén az eszközök körben kapcsolódnak.

```text
PC A ─── PC B
 |         |
PC D ─── PC C
```

Az adat a gyűrű mentén haladhat.

---

# 30. Mesh topology

A **mesh topology** esetén az eszközök több másik eszközhöz is kapcsolódhatnak.

Egyszerűsítve:

```text
A────B
|\  /|
| \/ |
| /\ |
|/  \|
C────D
```

Előnye lehet a redundancia.

Ha egy útvonal kiesik, rendelkezésre állhat másik.

Hátránya:

```text
több kapcsolat
→ nagyobb komplexitás
```

---

# 31. Tree topology

A **tree topology** hierarchikus hálózati felépítés.

Példa:

```text
            Router
              |
         Core Switch
          /       \
     Switch A    Switch B
      /   \        /   \
    PC1   PC2    PC3   PC4
```

Nagyobb hálózatokban gyakori a hierarchikus gondolkodás.

---

# 32. Mi fontos a topológiákból DevOps szempontból?

Nem elsősorban az a cél, hogy minden topológiát kívülről bemagolj.

Fontosabb, hogy egy hálózati diagramon fel tudd ismerni:

```text
hol van a router?
hol van a switch?
melyik host hová kapcsolódik?
milyen hálózatok vannak?
van-e alternatív útvonal?
hol lehet single point of failure?
```

---

# 33. Physical és Logical Network

## Physical network

A tényleges fizikai infrastruktúra.

Például:

```text
network cable
network card
switch
router
Wi-Fi hardware
```

---

## Logical network

A hálózat logikai szerkezete.

Például:

```text
IP addresses
subnets
routes
virtual networks
```

Két virtual machine lehet ugyanazon a fizikai szerveren, miközben logikailag külön hálózatokhoz tartoznak.

Ez Docker, virtualizáció és cloud környezetben különösen fontos.

---

# 34. Bandwidth

A **bandwidth = sávszélesség** azt jelzi, mennyi adat továbbítható egy adott idő alatt.

Például:

```text
100 Mbps
1 Gbps
10 Gbps
```

Mental model:

```text
bandwidth
→ mennyi adat / másodperc
```

---

# 35. Latency

A **latency = késleltetés** azt mutatja meg, mennyi idő alatt jut el az adat egyik pontból a másikba.

Például:

```text
10 ms
50 ms
200 ms
```

Mental model:

```text
latency
→ mennyi idő alatt?
```

Fontos:

```text
high bandwidth
≠
low latency
```

Lehet nagy sávszélességű kapcsolatunk magas késleltetéssel is.

---

# 36. Packet loss

A **packet loss** azt jelenti, hogy bizonyos hálózati csomagok elvesznek az átvitel során.

Példa:

```text
100 packet sent
98 packet received
```

Ez:

```text
2% packet loss
```

Következménye lehet:

```text
lassulás
lag
kapcsolati problémák
hang/videó akadozása
```

---

# 37. OSI / réteges hálózati modell – csak alapötlet

A hálózati kommunikáció több különböző feladatra bontható.

Ezeket rétegekben szokás elképzelni.

Egyelőre csak ezt az egyszerű képet használjuk:

```text
Application
↓
Transport
↓
Network
↓
Data Link
↓
Physical
```

Nagyon egyszerű példákkal:

```text
HTTP
→ Application
```

```text
TCP
→ Transport
```

```text
IP
→ Network
```

```text
Ethernet / MAC
→ Data Link
```

```text
cable / radio signal
→ Physical
```

A teljes OSI modellt és a rétegek részletes működését **később külön újra kibontjuk**.

Egyelőre csak az a fontos:

> A hálózati kommunikáció nem egyetlen művelet, hanem több egymásra épülő réteg együttműködése.

---

# 38. Frame, Packet és Segment – csak előzetes kép

Ugyanaz az alkalmazási adat különböző hálózati rétegeken más formában jelenik meg.

Nagyon leegyszerűsítve:

```text
Application data
↓
TCP segment
↓
IP packet
↓
Ethernet frame
```

Egyelőre:

```text
segment
→ TCP

packet
→ IP

frame
→ Ethernet
```

A részletes működést később a TCP/IP és teljes network flow résznél ismét át fogjuk venni.

---

# 39. Local kommunikáció egyszerűsített folyamata

Legyen:

```text
PC A
192.168.1.10
```

és:

```text
PC B
192.168.1.20
```

Ugyanabban a subnetben vannak.

Egyszerűsítve:

```text
PC A adatot akar küldeni
↓
destination IP ismert
↓
same subnet
↓
ARP
↓
destination MAC
↓
Ethernet / local network
↓
Switch
↓
PC B
```

---

# 40. Remote kommunikáció egyszerűsített folyamata

Legyen:

```text
PC A
192.168.1.10
```

A cél:

```text
8.8.8.8
```

A cél nem local.

```text
PC A adatot akar küldeni
↓
destination IP = 8.8.8.8
↓
different subnet
↓
default gateway kell
↓
ARP a gateway MAC-címéhez
↓
frame a routernek
↓
router
↓
másik hálózat
↓
destination
```

Nagyon fontos:

```text
IP destination:
8.8.8.8

MAC destination a local LAN-on:
default gateway MAC
```

Az IP a végső hálózati célt jelöli.

A MAC a local network aktuális továbbítási lépéséhez kell.

---

# 41. Egyszerű webes rendszerkép

Ha megnyitunk egy weboldalt:

```text
Browser
↓
domain name
↓
DNS
↓
IP address
↓
network communication
↓
remote server
```

Később ezt részletesen kibontjuk:

```text
Browser
↓
DNS
↓
IP
↓
TCP
↓
TLS
↓
HTTPS
↓
Nginx
↓
Node.js
↓
Database
```

Ez lesz az egyik legfontosabb DevOps rendszerképünk.

---

# 42. Networking mental model

Nagyon leegyszerűsítve:

```text
Application
↓
Port
↓
TCP / UDP
↓
IP
↓
Network Interface
↓
Ethernet / Wi-Fi
↓
Switch
↓
Router
↓
ISP
↓
Internet
↓
Remote Network
↓
Remote Host
```

Egyelőre nem kell minden elem működését részletesen tudni.

Az alapozó fejezet célja:

> Tudd, hogy a fontos hálózati fogalmak nagyjából hol helyezkednek el a teljes rendszerben.

---

# 43. Legfontosabb alapfogalmak röviden

```text
Network
→ kommunikáló eszközök hálózata
```

```text
Host
→ hálózaton kommunikáló eszköz
```

```text
Client
→ kérést indít
```

```text
Server
→ szolgáltatást nyújt / választ ad
```

```text
LAN
→ helyi hálózat
```

```text
WAN
→ nagyobb hálózatokat összekötő hálózat
```

```text
ISP
→ internetszolgáltató
```

```text
Network Interface
→ host hálózati csatlakozási pontja
```

```text
MAC
→ helyi interface azonosító
```

```text
ARP
→ IP → MAC feloldás local networkön
```

```text
IP
→ logikai hálózati cím
```

```text
Port
→ szolgáltatás azonosítása a hoston
```

```text
Protocol
→ kommunikációs szabályrendszer
```

```text
Switch
→ local network / MAC
```

```text
Router
→ hálózatok közötti továbbítás / IP
```

```text
Gateway
→ kijárat más hálózatok felé
```

```text
DNS
→ név → IP
```

```text
DHCP
→ automatikus hálózati konfiguráció
```

```text
NAS
→ hálózatra kapcsolt adattároló
```

```text
Topology
→ hálózati eszközök kapcsolódási szerkezete
```

```text
Bandwidth
→ mennyi adat vihető át
```

```text
Latency
→ hálózati késleltetés
```

```text
Packet loss
→ elveszett hálózati csomagok
```

---

# 44. Rövid szakmai angol

> A network connects devices so they can communicate.

> A host is a device connected to a network.

> A client sends a request.

> A server provides a service.

> A LAN is a local network.

> A router connects different networks.

> A switch connects devices inside a local network.

> An IP address is a logical network address.

> A MAC address identifies an interface on the local network.

> ARP maps an IP address to a MAC address.

> DHCP provides network configuration automatically.

> DNS resolves domain names to IP addresses.

> A port identifies a service on a host.

> Bandwidth describes how much data can be transferred.

> Latency describes network delay.

---

# 45. Mit fogunk később részletesen kibontani?

Ez a jegyzet csak az alapozó térkép.

## 02 – IP Addressing and Subnets

```text
IPv4
binary numbers
subnet mask
CIDR
network bits
host bits
network address
broadcast
usable host range
subnet calculation
```

## 03 – Routing, Gateway, NAT

```text
routing
routing table
default route
default gateway
next hop
NAT
```

## 04 – DNS

```text
DNS resolution
resolver
recursive DNS
authoritative DNS
DNS records
cache
```

## 05 – TCP, UDP, Ports, Sockets

```text
TCP
UDP
ports
sockets
connections
TCP handshake
```

## 06 – HTTP / HTTPS Network Flow

```text
DNS
↓
TCP
↓
TLS
↓
HTTP
↓
server
```

Itt fogjuk újra, részletesebben összerakni a réteges működést, frame/packet/segment fogalmakat és az encapsulationt is.

## 07 – Linux Networking Commands

```text
ip addr
ip route
ip neigh
ss
ping
traceroute
dig
curl
```

## 08 – Firewalls

```text
inbound
outbound
ports
rules
firewall
```

## 09 – Network Troubleshooting

```text
interface?
IP?
route?
DNS?
port?
service?
firewall?
```

## 10 – Networking Cheatsheet

A teljes networking modul rövid összefoglalója.