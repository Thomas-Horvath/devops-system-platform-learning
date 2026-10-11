# Routing, Gateway and NAT

## 1. Mi történik, ha a cél ugyanabban a subnetben van?

Tegyük fel:

```text
PC1:
192.168.1.10/24

PC2:
192.168.1.50/24
```

Mindkét gép hálózata:

```text
192.168.1.0/24
```

Ezért:

```text
same subnet
```

A PC1-nek nem kell routerhez fordulnia ahhoz, hogy a PC2-vel kommunikáljon.

Egyszerűen:

```text
PC1
192.168.1.10
   │
   │ local communication
   ↓
PC2
192.168.1.50
```

A PC1 például ARP segítségével megtudhatja a PC2 MAC-címét, majd közvetlenül el tudja neki küldeni a frame-et.

Mental model:

```text
same subnet
→ direct/local communication
```

---

# 2. Mi történik, ha a cél másik subnetben van?

Példa:

```text
PC1:
192.168.1.10/24

PC2:
192.168.2.50/24
```

PC1 hálózata:

```text
192.168.1.0/24
```

PC2 hálózata:

```text
192.168.2.0/24
```

Ezért:

```text
different subnet
```

A PC1 nem tudja közvetlenül elküldeni a packetet a PC2-nek.

Ilyenkor a packetet a:

```text
default gateway
```

felé küldi.

---

# 3. Default gateway

A **default gateway** általában egy router interfészének IP-címe a helyi hálózaton.

Példa:

```text
PC:
192.168.1.10/24

Default gateway:
192.168.1.1
```

A host logikája:

```text
Destination IP
↓
same subnet?
```

Ha igen:

```text
direct communication
```

Ha nem:

```text
send to default gateway
```

Egyszerűen:

```text
PC
192.168.1.10
   │
   ↓
Default Gateway
192.168.1.1
   │
   ↓
other network
```

Mental model:

> A default gateway az a következő eszköz, amelynek átadjuk a packetet, ha a cél nincs a saját subnetünkben.

---

# 4. Routing

A **routing** az a folyamat, amely során egy router eldönti:

```text
merre kell továbbküldeni a packetet?
```

A router ehhez a destination IP-címet vizsgálja.

Például:

```text
Destination:
192.168.2.50
```

A router megkeresi, melyik hálózat tartalmazza ezt az IP-címet.

---

# 5. Routing table

A routernek van egy:

```text
routing table
```

azaz útválasztási táblája.

Például:

```text
192.168.1.0/24 → interface 1
192.168.2.0/24 → interface 2
10.0.0.0/8     → Router B
0.0.0.0/0      → ISP
```

A routing table nem feltétlenül konkrét gépeket tartalmaz.

Általában hálózatokra vonatkozó útvonalak vannak benne.

Például:

```text
192.168.2.0/24
```

nem egy konkrét host, hanem egy teljes hálózat.

---

# 6. Routing table lookup

Ha a packet célja:

```text
192.168.2.50
```

a router megvizsgálja:

```text
melyik ismert hálózatba tartozik?
```

Megtalálja:

```text
192.168.2.0/24
```

és továbbítja a hozzá tartozó interfész vagy következő router felé.

Mental model:

```text
Destination IP
↓
routing table
↓
best matching route
↓
next hop / interface
```

---

# 7. Next hop

A **next hop** a következő router vagy hálózati eszköz, amelynek a packetet tovább kell adni.

Fontos:

> Egy routernek nem feltétlenül kell ismernie a teljes útvonalat a célhoz.

Elég, ha tudja:

```text
ehhez a hálózathoz
→ erre küldd tovább
```

Példa:

```text
Router A
↓
Router B
↓
Router C
↓
destination network
```

Router A-nak elég lehet annyit tudnia:

```text
destination network
→ Router B
```

Router B pedig saját routing table-je alapján dönt tovább.

---

# 8. A packet útja több routeren keresztül

Tegyük fel:

```text
PC
192.168.1.10
```

A cél:

```text
203.0.113.50
```

A PC látja:

```text
different subnet
```

Ezért:

```text
PC
↓
default gateway
```

A router:

```text
destination IP
↓
routing table lookup
↓
next hop
```

Majd:

```text
Router A
↓
Router B
↓
Router C
↓
destination network
```

Minden router új routing döntést végez.

---

# 9. Fontos: routing nem broadcast keresés

Ha egy router nem tudja pontosan, hol van a cél, nem kezdi el broadcasttal keresni az interneten.

Nem ez történik:

```text
"Ki tudja, merre van 203.0.113.50?"
```

Hanem:

```text
routing table lookup
```

Ha van megfelelő route:

```text
forward packet
```

Ha nincs megfelelő route, de van default route:

```text
use default route
```

Ha egyik sincs:

```text
drop packet
```

---

# 10. DNS és routing különbsége

A DNS és a routing két teljesen külön feladat.

## DNS

A DNS kérdése:

```text
Mi az IP-címe ennek a domain névnek?
```

Példa:

```text
example.com
↓
93.x.x.x
```

---

## Routing

A routing kérdése:

```text
Merre küldjem ezt az IP packetet?
```

Tehát:

```text
DNS
→ name → IP

Routing
→ destination IP → path / next hop
```

---

# 11. Domain név használatának folyamata

Ha egy program ezt akarja elérni:

```text
example.com
```

először szükség van az IP-címre.

```text
1. domain name

2. DNS lookup

3. DNS response

4. destination IP known

5. same subnet?

6. if remote → default gateway

7. routing
```

Fontos:

> A DNS-kérést is a saját gép indítja.

Ha a DNS-szerver másik hálózatban van, akkor maga a DNS-kérés is a default gatewayen keresztül jut el hozzá.

---

# 12. Default route

A **default route** olyan útvonal, amelyet akkor használunk, ha nincs pontosabb találat.

IPv4-ben:

```text
0.0.0.0/0
```

A `/0` azt jelenti, hogy nincs fix network bit, ezért bármely IPv4 címre illeszkedik.

Példa:

```text
192.168.1.0/24 → LAN
0.0.0.0/0      → ISP
```

Ez gyakorlatilag:

```text
local network?
→ LAN

anything else?
→ ISP
```

---

# 13. Legpontosabb route választása

Tegyük fel:

```text
192.168.1.0/24 → LAN
0.0.0.0/0      → ISP
```

Destination:

```text
192.168.1.50
```

A `/24` pontosabb találat, ezért:

```text
LAN
```

Destination:

```text
8.8.8.8
```

Nem illeszkedik a `/24` route-ra.

Ezért:

```text
0.0.0.0/0
→ ISP
```

A router mindig a legjobb, legspecifikusabb megfelelő route-ot választja.

---

# 14. Hogyan kerülnek route-ok a routing table-be?

A route-ok többféle forrásból kerülhetnek be.

Fontosabbak:

```text
directly connected routes
static routes
dynamic routes
```

---

# 15. Directly connected route

Ha a router egyik interfésze:

```text
192.168.1.1/24
```

akkor automatikusan tudja:

```text
192.168.1.0/24
→ directly connected
```

Ha egy másik interfésze:

```text
192.168.2.1/24
```

akkor:

```text
192.168.2.0/24
→ directly connected
```

A routernek ezeket nem kell más routerektől megtanulnia.

---

# 16. Static route

A **static route** kézzel konfigurált útvonal.

Példa:

```text
10.20.0.0/16
→ next hop: 192.168.1.254
```

Jelentése:

> Ha a 10.20.0.0/16 hálózat felé küldesz packetet, add tovább a 192.168.1.254 routernek.

Mental model:

```text
static route
→ administrator configures it
```

---

# 17. Dynamic route

Nagyobb hálózatokban nem praktikus minden route-ot kézzel konfigurálni.

Ezért routerek routing protocolok segítségével útvonal-információkat oszthatnak meg egymással.

Példák:

```text
OSPF
BGP
RIP
```

Ezek részletes működését ezen a szinten még nem szükséges tudni.

Mental model:

```text
dynamic routing
→ routers learn routes from routers
```

---

# 18. Fontos: a router nem attól tanul route-ot, hogy egyszer már járt ott packet

A routing table nem úgy működik, mint egy böngészési előzmény.

Nem:

```text
"Ezt a hálózatot egyszer már megtaláltam,
ezért megjegyzem."
```

A route forrása inkább:

```text
direct connection
static configuration
dynamic routing protocol
```

---

# 19. Routing table vs ARP cache

Ezt könnyű összekeverni.

## Routing table

Kérdés:

```text
Melyik hálózat merre érhető el?
```

Például:

```text
10.20.0.0/16
→ Router B
```

---

## ARP cache

Kérdés:

```text
Melyik helyi IP-hez milyen MAC-cím tartozik?
```

Például:

```text
192.168.1.1
→ aa:bb:cc:dd:ee:ff
```

Tehát:

```text
Routing table
→ network → direction

ARP cache
→ local IP → MAC
```

---

# 20. Hop

Egy **hop** egy routeren történő áthaladást jelent.

Példa:

```text
PC
↓
Router A
↓
Router B
↓
Router C
↓
Server
```

A packet több hopon keresztül jut el a célhoz.

Egyszerűen:

```text
Router A → one hop
Router B → next hop
Router C → next hop
```

---

# 21. TTL – Time To Live

IPv4 packetekben van egy:

```text
TTL
```

mező.

Teljes neve:

```text
Time To Live
```

A gyakorlatban minden router csökkenti az értékét.

Például:

```text
TTL = 64
```

Router A után:

```text
63
```

Router B után:

```text
62
```

Router C után:

```text
61
```

---

# 22. Miért kell TTL?

Előfordulhat hibás routing miatt routing loop.

Például:

```text
Router A
↓
Router B
↓
Router A
↓
Router B
↓
...
```

TTL nélkül a packet nagyon sokáig keringhetne.

Ha:

```text
TTL = 0
```

a packetet eldobják.

Mental model:

```text
TTL
→ limits how many routing hops a packet can survive
```

---

# 23. Mi történik, ha nincs route?

Tegyük fel:

```text
192.168.1.0/24 → directly connected
10.0.0.0/8     → Router B
```

Nincs:

```text
0.0.0.0/0
```

default route.

Destination:

```text
8.8.8.8
```

A router:

```text
192.168.1.0/24?
→ no

10.0.0.0/8?
→ no

default route?
→ none
```

Ezért:

```text
packet dropped
```

A router bizonyos esetekben ICMP hibát is visszaküldhet.

Például:

```text
Destination Unreachable
```

---

# 24. NAT

A **NAT** jelentése:

```text
Network Address Translation
```

A NAT IP-címeket fordít át.

Otthoni hálózatban tipikus feladata:

```text
private IP
↓
public IP
```

---

# 25. Private hálózat otthon

Például:

```text
Laptop:
192.168.1.10

Telefon:
192.168.1.20

TV:
192.168.1.30
```

Ezek private IPv4 címek.

A router LAN interfésze:

```text
192.168.1.1
```

A router külső oldala pedig például rendelkezhet egy ISP-től kapott public IP-címmel.

Példánkban:

```text
84.2.100.50
```

Ez csak szemléltető példa.

---

# 26. DHCP és az otthoni címzés

Otthoni hálózatokban gyakori, hogy maga a router DHCP-szerverként is működik.

Például:

```text
router DHCP server
↓
Laptop → 192.168.1.10
Telefon → 192.168.1.20
TV → 192.168.1.30
```

Ez azonban külön feladat a NAT-tól.

```text
DHCP
→ local IP configuration

NAT
→ address translation
```

---

# 27. Kifelé menő kapcsolat NAT-tal

Tegyük fel:

```text
PC:
192.168.1.10
```

A PC szeretne elérni egy internetes szervert.

Eredeti packet:

```text
Source:
192.168.1.10

Destination:
203.0.113.20
```

A router NAT-ot alkalmaz.

Kifelé például:

```text
Source:
84.2.100.50

Destination:
203.0.113.20
```

A távoli szerver a public IP-t látja forrásként.

---

# 28. Egy public IP több belső géphez

Otthon lehet:

```text
PC
Telefon
Laptop
TV
Tablet
```

de az internet felé sok esetben ugyanazt a public IPv4 címet használják.

Például:

```text
PC      192.168.1.10 ─┐
Phone   192.168.1.20 ─┤
Laptop  192.168.1.30 ─┤
                       ↓
                    Router
                    NAT/PAT
                       ↓
                  84.2.100.50
                       ↓
                    Internet
```

---

# 29. PAT

Ehhez a gyakorlatban gyakran portokat is használunk.

Ez a:

```text
PAT
```

azaz:

```text
Port Address Translation
```

Gyakran:

```text
NAT overload
```

néven is találkozhatunk vele.

---

# 30. NAT/PAT példa

Belső kapcsolat:

```text
192.168.1.10:52341
```

A router kifelé például átfordíthatja erre:

```text
84.2.100.50:41052
```

Másik gép:

```text
192.168.1.20:53000
```

kifelé:

```text
84.2.100.50:41053
```

Így ugyanaz a public IP több kapcsolatot is képes kiszolgálni.

---

# 31. Egyszerű NAT tábla

A router valamilyen kapcsolatállapotot tart nyilván.

Leegyszerűsítve:

```text
Inside                    Public

192.168.1.10:52341   ↔   84.2.100.50:41052

192.168.1.20:53000   ↔   84.2.100.50:41053
```

Ha visszaérkezik:

```text
84.2.100.50:41052
```

a router tudja:

```text
→ 192.168.1.10:52341
```

---

# 32. A NAT-tábla valójában több adatot követ

A port önmagában nem feltétlenül elég egy kapcsolat azonosítására.

A router kapcsolatkövetése több információt is figyelembe vehet.

Például:

```text
protocol
source IP
source port
destination IP
destination port
```

Egy TCP kapcsolat például elképzelhető így:

```text
TCP

192.168.1.10:52341
→
203.0.113.20:443
```

NAT után:

```text
84.2.100.50:41052
→
203.0.113.20:443
```

---

# 33. IP és port együtt

Egy kapcsolatnál gyakran együtt találkozunk:

```text
IP address
+
port
```

Például:

```text
192.168.1.10:52341
```

Itt:

```text
192.168.1.10
→ host / interface IP

52341
→ port
```

A portok működését részletesen a:

```text
05-tcp-udp-ports-sockets.md
```

jegyzetben vesszük át.

---

# 34. Mi történik a visszaérkező válasszal?

Tegyük fel kifelé:

```text
192.168.1.10:52341
↓
NAT
↓
84.2.100.50:41052
↓
server
```

A szerver válaszol:

```text
server
↓
84.2.100.50:41052
```

A router megnézi a NAT kapcsolatállapotot:

```text
84.2.100.50:41052
↔
192.168.1.10:52341
```

Majd továbbítja:

```text
192.168.1.10:52341
```

felé.

---

# 35. NAT mental model

```text
private host
↓
router
↓
NAT/PAT
↓
public IP
↓
internet
```

Visszafelé:

```text
internet response
↓
public IP + translated port
↓
NAT table lookup
↓
private IP + original port
```

---

# 36. NAT és routing nem ugyanaz

Ez nagyon fontos.

## Routing

Kérdés:

```text
Merre küldjem a packetet?
```

## NAT

Kérdés:

```text
Milyen IP-címet / portot kell átfordítanom?
```

Tehát:

```text
Routing
→ path selection

NAT
→ address translation
```

A kettő együtt is működhet ugyanabban a routerben.

---

# 37. Gateway és router különbsége

A **router** egy hálózati eszköz vagy funkció, amely hálózatok között továbbít packeteket.

A **gateway** inkább egy szerep.

A host szemszögéből:

```text
default gateway
→ where to send traffic for remote networks
```

Otthon gyakran ugyanaz a fizikai eszköz végzi:

```text
router
gateway
DHCP server
NAT
firewall
Wi-Fi access point
```

feladatokat.

Ezért könnyű azt hinni, hogy ezek mind ugyanazt jelentik.

Nem ugyanazok, csak gyakran egyetlen eszközbe vannak összeépítve.

---

# 38. Mi történik egy internetes kapcsolatnál?

Egyszerűsített folyamat:

```text
Application
↓
destination IP known
↓
same subnet?
↓
NO
↓
default gateway
↓
router
↓
NAT/PAT
↓
routing table lookup
↓
default route / next hop
↓
ISP
↓
more routers
↓
destination network
↓
server
```

A válasz hasonló úton visszajut.

---

# 39. Domain névvel ugyanez

Ha a cél:

```text
api.example.com
```

először:

```text
DNS
↓
domain → IP
```

Ezután:

```text
destination IP
↓
routing
↓
gateway
↓
NAT
↓
internet
```

Nagyon röviden:

```text
DNS
→ What is the IP?

Routing
→ Where should the packet go?

NAT
→ Which address should be translated?
```

---

# 40. Packetek és rétegek

Amikor konkrét adatot küldünk, nem csak maga az application data utazik.

A hálózati kommunikáció több réteg információját tartalmazza.

Nagyon leegyszerűsítve:

```text
HTTP data
↓
TCP information
↓
IP information
↓
Ethernet information
↓
physical transmission
```

Például egy POST kérésnél:

```text
HTTP:
POST /users
JSON data
```

Ehhez később kapcsolódhat:

```text
TCP:
source port
destination port
sequence information
```

majd:

```text
IP:
source IP
destination IP
TTL
```

majd helyi hálózaton:

```text
Ethernet:
source MAC
destination MAC
```

Ezt a témát részletesen később vesszük át:

```text
05-tcp-udp-ports-sockets.md
06-http-https-network-flow.md
```

---

# 41. Fontos: MAC és IP nem ugyanazt a célt jelöli

Távoli szerver esetén:

```text
Destination IP
→ final remote destination
```

Viszont az Ethernet frame:

```text
Destination MAC
→ next local hop
```

Például:

```text
Destination IP:
203.0.113.20

Destination MAC:
default gateway MAC
```

Mert a távoli szerver nincs a helyi LAN-on.

---

# 42. Router és frame

Amikor egy router megkap egy packetet, a link-layer frame csak az adott helyi hálózati szakaszhoz tartozik.

Következő linken új frame készülhet.

Például:

```text
PC
MAC A
↓
Router
MAC B
```

Majd:

```text
Router
MAC C
↓
Next router
MAC D
```

Tehát a MAC-címek az út során változhatnak.

Az IP packet destination címe viszont a végső célra mutat.

NAT esetén a source IP is módosulhat.

---

# 43. Wireshark

A hálózati forgalom ténylegesen megfigyelhető:

```text
Wireshark
```

segítségével.

Egy capture-ben például láthatunk:

```text
Ethernet
IPv4
TCP
TLS
```

vagy titkosítatlan HTTP esetén:

```text
Ethernet
IPv4
TCP
HTTP
```

Ez később segít vizuálisan megérteni:

```text
frame
packet
segment
ports
IP addresses
MAC addresses
TCP flags
TTL
TLS
HTTP
```

kapcsolatát.

---

# 44. Routing folyamat összefoglalva

A host:

```text
Destination IP
↓
same subnet?
```

Ha igen:

```text
local delivery
```

Ha nem:

```text
default gateway
```

A router:

```text
destination IP
↓
routing table lookup
↓
best matching route
```

Ha van route:

```text
next hop / interface
```

Ha nincs konkrét route, de van:

```text
0.0.0.0/0
```

akkor:

```text
default route
```

Ha semmi nincs:

```text
drop packet
```

---

# 45. Rövid mental model

```text
Same subnet
→ direct communication
```

```text
Different subnet
→ default gateway
```

```text
Router
→ checks routing table
```

```text
Routing table
→ network → direction
```

```text
Next hop
→ next router
```

```text
Default route
→ use when no better route exists
```

```text
NAT
→ translate IP addresses
```

```text
PAT
→ allows many internal connections to share one public IP
```

```text
TTL
→ prevents packets from looping forever
```

---

# 46. Rövid szakmai angol

> A host sends remote traffic to its default gateway.

> A router checks its routing table before forwarding a packet.

> The next hop is the next router on the path.

> A default route is used when no more specific route exists.

> Static routes are configured manually.

> Dynamic routes can be learned from other routers.

> TTL is reduced at every routing hop.

> NAT translates network addresses.

> PAT allows multiple private hosts to share one public IPv4 address.

> DNS resolves names to IP addresses.

> Routing determines where packets should go.

---

# 47. Amit ebből biztosan érteni kell

A legfontosabb folyamat:

```text
destination IP
↓
same subnet?
├── YES
│   ↓
│ local communication
│
└── NO
    ↓
default gateway
    ↓
routing table
    ↓
best route
    ↓
next hop
    ↓
destination network
```

Otthoni internetnél ehhez jön:

```text
private IP
↓
NAT/PAT
↓
public IP
↓
internet
```

A három könnyen keverhető fogalom:

```text
DNS
→ name → IP
```

```text
Routing
→ destination IP → path
```

```text
NAT
→ private/public address translation
```

A routing table útvonalai származhatnak:

```text
direct connection
static configuration
dynamic routing
```

És végül:

```text
no matching route
+
no default route
↓
packet dropped
```