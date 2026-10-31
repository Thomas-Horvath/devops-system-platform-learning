# Docker Networking Deep Dive

## 1. Mi ennek a fejezetnek a célja?

Az előző fejezetben megtanultuk a Docker networking használati modelljét:

```text
Container
→ saját IP

Docker network
→ virtuális hálózat

Docker DNS
→ container/service név feloldása

-p 8080:3000
→ host port → container port

Container → Internet
→ Docker networking + NAT
```

Ebben a fejezetben egy szinttel mélyebbre megyünk.

Azt nézzük meg:

```text
network namespace
veth pair
Linux bridge
container interface
gateway
routing
ARP
DNS
NAT
port publishing
network isolation
több network
default bridge
user-defined bridge
host network
none network
Docker Desktop / WSL2 sajátosságok
troubleshooting
```

A cél:

> Ne csak azt tudjuk, milyen Docker parancsot kell kiadni, hanem értsük, mi történik a hálózatban a háttérben.

---

# 2. A Docker networking Linux networkingre épül

A Docker nem talált fel egy teljesen új hálózati rendszert.

A Linux már rendelkezik olyan mechanizmusokkal, amelyekből container networking építhető.

Ilyenek például:

```text
network namespaces
virtual Ethernet interfaces
Linux bridges
routing
NAT
firewall rules
DNS
```

Ezért a Docker networking valójában nagyon erősen kapcsolódik ahhoz, amit korábban Linux networkingből tanultunk.

Mental model:

```text
Docker networking
=
Linux networking automatizálva
+
container lifecycle-höz kötve
```

---

# 3. Network namespace

A container egyik legfontosabb izolációs mechanizmusa a:

```text
network namespace
```

A namespace lényege:

```text
milyen hálózati objektumokat lát egy process?
```

Egy container saját network namespace-ben láthat például:

```text
interfaces
IP addresses
routing table
ARP / neighbor table
localhost
sockets
ports
```

---

# 4. Host network namespace vs container namespace

Egyszerűsítve:

```text
HOST NETWORK NAMESPACE

eth0
docker bridge
routing
host sockets
```

és külön:

```text
CONTAINER NETWORK NAMESPACE

eth0
lo
container IP
container routes
container sockets
```

A container process számára úgy tűnhet, mintha saját kis hálózati gépe lenne.

De valójában továbbra is ugyanazon Linux kernel kezeli ezeket.

---

# 5. A container saját `localhost`-ja

Minden network namespace saját loopback interface-t kaphat.

Ezért:

```text
Host localhost
≠
Container A localhost
≠
Container B localhost
```

Például:

```text
Container A
127.0.0.1
```

csak Container A saját namespace-ét jelenti.

Ha Container B-ben PostgreSQL fut:

```text
Container B
127.0.0.1:5432
```

Container A ezt nem éri el:

```text
localhost:5432
```

címen.

---

# 6. A container virtuális hálózati interfésze

A containerben gyakran látható egy:

```text
eth0
```

interface.

Például:

```text
eth0
IP: 172.18.0.2
```

Ez nem feltétlenül fizikai hálózati kártya.

Egy virtuális interface.

A host és a container között gyakran:

```text
veth pair
```

kapcsolat jön létre.

---

# 7. Mi az a veth pair?

A:

```text
veth
```

jelentése:

```text
virtual Ethernet
```

A veth pair két összekapcsolt virtuális Ethernet interface.

Úgy képzelhetjük el, mint egy virtuális Ethernet kábelt.

```text
veth-A
────────────
veth-B
```

Ami az egyik végére érkezik, megjelenik a másikon.

---

# 8. Container és veth pair

Egyszerűsítve:

```text
CONTAINER
   │
   │ eth0
   │
 [veth egyik vége]
        │
        │
 [veth másik vége]
        │
        ↓
HOST
```

A containerben az egyik vég általában:

```text
eth0
```

néven jelenik meg.

A másik vég a host networking világában létezik.

---

# 9. Miért kell veth?

Mert a container külön network namespace-ben van.

Valamilyen virtuális kapcsolat kell:

```text
container namespace
↕
host namespace
```

között.

A veth pair ezt a kapcsolatot biztosíthatja.

Mental model:

```text
Container eth0
↓
veth pair
↓
Host networking
```

---

# 10. Linux bridge

A host oldali veth interface-eket egy:

```text
Linux bridge
```

kötheti össze.

A Linux bridge működése hasonlít egy Ethernet switchhez.

Mental model:

```text
Container A
    │
    ↓
  veth
    │
    ↓
┌───────────────┐
│ Linux bridge  │
└───────────────┘
    ↑
    │
  veth
    ↑
Container B
```

---

# 11. Bridge mint virtuális switch

Normál fizikai hálózat:

```text
PC A
 │
 │
Switch
 │
 │
PC B
```

Docker/Linux környezet:

```text
Container A
 │
 │
Linux bridge
 │
 │
Container B
```

Ezért jó mental model:

> A Linux bridge egy virtuális Ethernet switch.

---

# 12. Teljes egyszerű bridge architektúra

```text
                HOST

         Linux bridge
       ┌───────────────┐
       │               │
      veth            veth
       │               │
       │               │
       ↓               ↓
 Container A       Container B

 eth0              eth0
 172.18.0.2        172.18.0.3
```

Mindkét container ugyanazon virtuális Layer 2 hálózathoz kapcsolódhat.

---

# 13. Container subnet

A Docker network jellemzően saját IP subnetet használ.

Például:

```text
172.18.0.0/16
```

A containerek ebből kaphatnak IP-t:

```text
Container A
172.18.0.2

Container B
172.18.0.3
```

A gateway lehet például:

```text
172.18.0.1
```

A konkrét címeket Docker választja.

---

# 14. Ugyanazon subnet kommunikáció

Korábbi networkingből:

```text
same subnet
→ közvetlen Layer 2 kommunikáció
```

Dockerben ugyanez.

Például:

```text
Container A
172.18.0.2/16

Container B
172.18.0.3/16
```

Mindkettő ugyanazon subnetben:

```text
172.18.0.0/16
```

Ezért A közvetlenül elérheti B-t a bridge networkön keresztül.

---

# 15. ARP szerepe

IPv4 esetén ugyanazon Ethernet hálózaton a küldőnek tudnia kell:

```text
destination IP
↓
destination MAC
```

kapcsolatot.

Ezért használható:

```text
ARP
```

A korábbi networking modell itt is érvényes.

Példa:

```text
Container A:

Kié a 172.18.0.3?
```

A válasz:

```text
Container B virtuális MAC címe
```

Ezután az Ethernet frame továbbítható a bridge-en keresztül.

---

# 16. Container → container packet flow

Tegyük fel:

```text
Container A
172.18.0.2

Container B
172.18.0.3
```

A kapcsolat:

```text
A → B:5432
```

Magas szinten:

```text
Application A
↓
TCP socket
↓
Container A eth0
↓
veth
↓
Linux bridge
↓
másik veth
↓
Container B eth0
↓
TCP :5432
↓
Application B
```

---

# 17. Docker bridge Layer 2 szerepe

A bridge elsősorban Ethernet frame-ek továbbításában vesz részt.

Mental model:

```text
IP packet
↓
Ethernet frame
↓
bridge
↓
megfelelő veth interface
```

Ez nagyon hasonló egy fizikai switch működéséhez.

---

# 18. Gateway

Ha a destination nem ugyanazon Docker subneten van, a container routing táblája gatewayt használhat.

Például:

```text
Container
172.18.0.2

Default gateway
172.18.0.1
```

Routing mental model:

```text
destination ugyanazon subnet?
↓
igen → direct

nem
↓
default gateway
```

Pont ugyanaz az elv, mint normál hálózatnál.

---

# 19. Container routing table

Egyszerűsített routing:

```text
172.18.0.0/16
→ eth0

default
→ 172.18.0.1
```

Ez azt jelenti:

```text
172.18.x.x
→ közvetlen Docker network

minden más
→ gateway
```

---

# 20. Container → internet routing

Tegyük fel:

```text
Container
172.18.0.2
```

megpróbálja elérni:

```text
8.8.8.8
```

Ez nem része:

```text
172.18.0.0/16
```

subnetnek.

Ezért:

```text
Container
↓
default gateway
↓
Docker host/backend
↓
routing + NAT
↓
külső network
↓
Internet
```

---

# 21. Miért kell NAT?

A container IP:

```text
172.18.0.2
```

privát cím.

Az interneten ezt általában nem route-olják közvetlenül.

Ezért a host/backend megváltoztathatja a source címet.

Mental model:

```text
BEFORE NAT

source:
172.18.0.2

destination:
93.x.x.x
```

majd:

```text
AFTER NAT

source:
host/backend külső IP

destination:
93.x.x.x
```

---

# 22. Source NAT / masquerading

Linux Docker környezetben a kifelé menő container trafficnél használható:

```text
SNAT / masquerading
```

logika.

Mental model:

```text
Container private IP
↓
NAT
↓
Host/backend IP
↓
Internet
```

A válasz visszaérkezésekor a rendszer tudja, melyik belső container kapcsolatához tartozik.

---

# 23. Otthoni router analógia

Otthon:

```text
PC
192.168.1.10
↓
Router
↓
NAT
↓
Public IP
↓
Internet
```

Dockerben:

```text
Container
172.18.0.2
↓
Docker host/backend
↓
NAT
↓
Host network
↓
Internet
```

Az alapelv nagyon hasonló.

---

# 24. Port publishing mélyebben

Például:

```bash
docker run -p 8080:3000 myapp
```

A kívülről érkező kapcsolat:

```text
Host :8080
```

portra érkezik.

Docker networking ezt a megfelelő container:

```text
Container :3000
```

portjára továbbítja.

Mental model:

```text
Client
↓
Host IP:8080
↓
Docker port publishing
↓
Container IP:3000
↓
Application
```

---

# 25. Destination NAT mental model

Port publishinget gondolkodásban tekinthetjük NAT/forwarding jellegű működésnek.

Bejövő:

```text
destination:
HOST_IP:8080
```

Docker továbbítja:

```text
destination:
CONTAINER_IP:3000
```

Ez destination address/port átírással és forwardinggal valósítható meg a platformtól függően.

Mental model:

```text
HOST:8080
↓
DNAT / forwarding jellegű működés
↓
CONTAINER:3000
```

---

# 26. Fontos: a Docker implementáció változhat

Nem érdemes azt memorizálni, hogy:

```text
docker -p
=
mindig pontosan egy konkrét iptables szabály
```

Mert Docker verziótól, host rendszertől és backendtől függhet:

```text
iptables
nftables
userland proxy
Docker Desktop networking
egyéb backend mechanizmus
```

A stabil mental model:

```text
host port
↓
Docker networking
↓
container port
```

---

# 27. Port publishing és listening socket

Ha:

```bash
docker run -p 8080:3000 myapp
```

de a containerben semmi nem listenel:

```text
:3000
```

porton, akkor a mapping önmagában nem segít.

Szükséges:

```text
Application
↓
listening socket
↓
container :3000
```

---

# 28. `127.0.0.1` probléma

Ha a container app csak:

```text
127.0.0.1:3000
```

címen listenel, akkor csak saját loopback interface-én fogad kapcsolatot.

Docker networking viszont jellemzően a container hálózati interface felől érkezik.

Ezért gyakran:

```text
0.0.0.0:3000
```

szükséges.

Mental model:

```text
127.0.0.1
→ csak loopback

0.0.0.0
→ minden IPv4 interface
```

---

# 29. Port publishing teljes útvonala

```text
Browser
↓
localhost:8080
↓
Windows/Linux host
↓
Docker networking
↓
Container virtual interface
↓
Container IP
↓
TCP :3000
↓
Application socket
```

Ha bármelyik láncszem hibás:

```text
kapcsolat nem működik
```

---

# 30. User-defined bridge network

Létrehozás:

```bash
docker network create app-network
```

Majd:

```text
app-network
├── web
└── db
```

Ez saját virtuális networköt ad.

---

# 31. Mi történik network létrehozáskor?

Magas szinten Docker létrehozhat:

```text
network object
subnet
gateway
bridge/network infrastructure
DNS/service discovery környezet
```

Majd containerek csatlakoztatásakor:

```text
network interface
IP address
route
DNS kapcsolat
```

konfigurálódhat.

---

# 32. Default bridge vs user-defined bridge

Docker rendelkezik egy alapértelmezett:

```text
bridge
```

networkkel.

De alkalmazásokhoz általában jobb:

```text
user-defined bridge
```

használata.

Példa:

```bash
docker network create app-network
```

---

# 33. Miért jobb a user-defined bridge?

Fő előnyök:

```text
jobb container name resolution
jobb izoláció
dinamikus network connect/disconnect
átláthatóbb architektúra
jobb Compose-kompatibilis mental model
```

Ezért tipikus alkalmazási setup:

```text
saját application network
```

nem egyszerűen a default bridge használata.

---

# 34. Docker DNS

User-defined networkön Docker beépített DNS/service discovery funkciót biztosít.

Például:

```text
web
db
```

containerek.

Webből:

```text
db
```

név feloldható:

```text
172.18.0.3
```

címre.

---

# 35. DNS flow

```text
Application
↓
connect("db", 5432)
↓
DNS lookup
↓
Docker embedded DNS
↓
db container IP
↓
TCP connection
```

Ezért nem kell fix IP-t konfigurálni.

---

# 36. Miért jobb DNS név, mint IP?

Container recreate:

```text
előtte:
db → 172.18.0.3
```

utána:

```text
db → 172.18.0.7
```

A név:

```text
db
```

változatlan maradhat.

Ez a service discovery egyik alapelve.

---

# 37. Service discovery

A service discovery azt jelenti:

> Hogyan találja meg az egyik szolgáltatás a másikat?

Docker networknél:

```text
service/container name
↓
DNS
↓
IP
```

Például:

```text
DATABASE_HOST=db
DATABASE_PORT=5432
```

---

# 38. DNS nem helyettesíti a portot

A:

```text
db
```

csak azt oldja meg:

```text
melyik IP?
```

A szolgáltatás portját továbbra is tudni kell.

Például:

```text
db:5432
```

Tehát:

```text
DNS
→ IP

port
→ application endpoint
```

---

# 39. Network isolation

Külön Docker networkök egymástól elkülönített kommunikációs tartományokat adhatnak.

Példa:

```text
frontend-network
├── frontend
└── api
```

```text
backend-network
├── api
└── database
```

A frontend:

```text
frontend → api
```

kapcsolatot használhat.

Az API:

```text
api → database
```

kapcsolatot használhat.

De:

```text
frontend → database
```

nem feltétlenül lehetséges közvetlenül.

---

# 40. Több network mint szegmentáció

Normál vállalati hálózatban:

```text
frontend subnet
backend subnet
database subnet
```

Dockerben:

```text
frontend-network
backend-network
```

használható logikai szegmentációra.

Mental model:

```text
külön network
≈
külön virtuális Layer 2 domain / logikai LAN
```

---

# 41. Container több networkön

Egy container egyszerre több Docker networkhöz is csatlakozhat.

Például:

```text
           frontend-network
                  │
                  ↓
Frontend ─────── API
                  │
                  │ backend-network
                  ↓
              Database
```

Az API:

```text
network A
+
network B
```

tag.

---

# 42. Több network esetén több interface

Egy container több networkhöz csatlakoztatva több virtuális interface-t/IP konfigurációt is kaphat.

Mental model:

```text
Container API

eth0
→ frontend network

eth1
→ backend network
```

A konkrét interface elnevezés és implementáció változhat, ezért ezt ne mereven memorizáljuk.

A lényeg:

```text
egy container
→ több network kapcsolat
```

---

# 43. Network connect

Futó container csatlakoztatása:

```bash
docker network connect backend-network api
```

Ezután:

```text
api
├── original network
└── backend-network
```

---

# 44. Network disconnect

```bash
docker network disconnect backend-network api
```

Ez leválasztja a containert az adott networkről.

Ez mutatja:

```text
container lifecycle
és
network membership
```

külön kezelhető.

---

# 45. `host` network driver

Docker bizonyos környezetekben támogat:

```text
host
```

network módot.

Itt a container nem ugyanúgy kap külön bridge network namespace kommunikációt, mint a megszokott bridge esetben.

Mental model:

```text
Container
↓
host network stack közvetlenebb használata
```

---

# 46. Host network következménye

Bridge esetén:

```text
Container A :3000
Container B :3000
```

nem feltétlenül ütközik.

Host networkingnél azonban a host port namespace-éhez való erősebb kötődés miatt portütközés lehet.

Ezért:

```text
host network
```

más működési modell.

---

# 47. Mikor használunk host networköt?

Speciális esetekben például:

```text
performance
network tooling
special networking applications
```

miatt.

Normál webalkalmazásoknál általában:

```text
bridge / user-defined bridge
```

a természetesebb választás.

---

# 48. Docker Desktop és host network

Windows/macOS Docker Desktop esetén a:

```text
host
```

fogalma különösen óvatos értelmezést igényel, mert Linux containerek nem közvetlenül a fizikai Windows/macOS kernel network stackjén futnak.

Ezért Docker Desktop alatt:

```text
physical host
Docker Desktop Linux backend
container network
```

külön rétegek lehetnek.

A natív Linux mental model nem minden részletében másolható át.

---

# 49. `none` network driver

Container indítható hálózati kapcsolat nélkül is:

```bash
docker run --network none ...
```

Mental model:

```text
Container
↓
network namespace
↓
csak minimális networking
```

Általában:

```text
loopback
```

maradhat, de normál külső hálózati kapcsolat nincs.

---

# 50. Mire jó a `none`?

Például:

```text
offline processing
security isolation
network nélküli batch job
tesztelés
```

esetén.

Ez megmutatja, hogy a network hozzáférés sem kötelező része egy containernek.

---

# 51. Default bridge

Ha containert indítunk network megadása nélkül:

```bash
docker run nginx
```

Docker tipikusan az alapértelmezett:

```text
bridge
```

networkhöz csatlakoztatja.

Ez használható, de több alkalmazásból álló rendszernél gyakran jobb saját network.

---

# 52. Miért ne építsünk default bridge-re túl sokat?

Mert:

```text
kevésbé explicit architektúra
kevésbé jó service discovery modell
kevesebb logikai izoláció
```

lehet.

Jobb:

```bash
docker network create app-network
```

és explicit:

```bash
docker run --network app-network ...
```

---

# 53. Docker Compose és networking előzetesen

Compose később sok mindent automatikusan elvégez.

Például:

```text
services:
  web
  db
```

esetén tipikusan létrejön egy Compose network.

Mental model:

```text
Compose project
↓
default application network
├── web
└── db
```

A service nevek DNS névként használhatók.

Példa:

```text
db:5432
```

---

# 54. Docker Compose service discovery

Példa:

```text
web service
db service
```

A web app config:

```text
DB_HOST=db
```

A Compose networkön:

```text
db
↓
Docker DNS
↓
db container IP
```

Ezért Compose sokkal kényelmesebbé teszi több container kezelését.

---

# 55. Port publishing vs container-to-container traffic

Ez az egyik legfontosabb különbség.

Container → container:

```text
web
↓
db:5432
```

nem igényel host publishingot.

Host → container:

```text
browser
↓
localhost:8080
↓
web:3000
```

publishingot igényelhet.

Tehát:

```text
internal communication
≠
host exposure
```

---

# 56. Miért veszélyes minden portot publikálni?

Tegyük fel:

```text
web :3000
db :5432
redis :6379
```

Ha mindet publikáljuk:

```text
host:3000
host:5432
host:6379
```

akkor több service válik host felől elérhetővé.

Pedig lehet, hogy csak:

```text
web
```

kell kívülről.

Jobb:

```text
Host
↓
Web

Web
↓
DB
↓
Redis
```

belső Docker networkön.

---

# 57. Least exposure

Networking security elv:

> Csak azt a szolgáltatást tedd kívülről elérhetővé, amelyet ténylegesen el kell érni kívülről.

Példa:

```text
Public:
Reverse proxy :443
```

Internal:

```text
App :3000
Postgres :5432
Redis :6379
```

Ez sokkal jobb architecture.

---

# 58. Reverse proxy Docker networkben

Később lehet:

```text
Internet
↓
Nginx
↓
Docker network
↓
Next.js
```

Nginx például:

```text
app:3000
```

néven éri el az alkalmazást.

Nem kell a Next.js portját közvetlenül internetre publikálni.

---

# 59. Komplexebb production architektúra

```text
Internet
   │
   ↓
Host :443
   │
   ↓
Nginx Container
   │
   │ frontend-network
   ↓
Next.js Container
   │
   │ backend-network
   ↓
PostgreSQL Container
```

Network membership:

```text
Nginx
→ frontend-network

Next.js
→ frontend-network
→ backend-network

PostgreSQL
→ backend-network
```

Ez szegmentált architektúra.

---

# 60. Docker firewall kapcsolat

Docker networking a host firewall rendszerével is kapcsolatban állhat.

Natív Linuxon például:

```text
iptables
vagy
nftables
```

szabályok is részt vehetnek:

```text
NAT
forwarding
filtering
port publishing
```

megvalósításában.

---

# 61. Ne módosítsunk Docker szabályokat vakon

Mivel Docker maga is kezelhet firewall/NAT szabályokat:

```text
manual firewall modification
```

könnyen összetörheti a Docker networkinget.

Ezért troubleshootingnál előbb:

```text
docker inspect
docker network inspect
docker ps
```

szintjén vizsgáljuk a konfigurációt.

A host firewall mélyebb vizsgálata csak később jön.

---

# 62. Docker Desktop + Windows architektúra

A saját környezetünk:

```text
Windows 11
↓
Docker Desktop
↓
WSL2 / Linux backend
↓
Linux kernel
↓
Docker Engine
↓
Docker networks
↓
Linux containers
```

Ez nagyon fontos.

A Linux containerek hálózata:

```text
nem közvetlenül egy klasszikus natív Windows Docker bridge
```

hanem a Docker Desktop Linux backendjén keresztül működik.

---

# 63. Docker Desktop networking rétegek

Egyszerűsített mental model:

```text
Windows application / browser
↓
Windows networking
↓
Docker Desktop networking integration
↓
Linux backend
↓
Docker bridge/network
↓
Container
```

Ezért például:

```text
localhost:8080
```

Windowsból működhet, miközben a container:

```text
Linux backend
```

belső networkjében él.

---

# 64. Miért ne használjunk közvetlen container IP-t Docker Desktopon?

Natív Linuxon bizonyos helyzetekben a host közvetlenebbül elérheti a container bridge IP-jét.

Docker Desktopnál:

```text
Windows host
≠
Docker Linux backend network namespace
```

Ezért:

```text
172.x.x.x container IP
```

nem biztos, hogy megfelelő vagy stabil Windows host interface.

Használjuk inkább:

```text
localhost + published port
```

modellt.

---

# 65. Stabil Docker interface

Hostról:

```text
localhost:8080
```

Container-to-container:

```text
service-name:port
```

Ez a két legfontosabb stabil interface.

Mental model:

```text
HOST → CONTAINER
published port
```

```text
CONTAINER → CONTAINER
DNS name + container port
```

---

# 66. Docker network inspect

```bash
docker network inspect app-network
```

Nagyon fontos troubleshooting eszköz.

Megnézhetjük:

```text
network driver
subnet
gateway
connected containers
container IP addresses
network options
```

---

# 67. Container inspect

```bash
docker inspect web
```

A networking részben láthatunk például:

```text
Networks
IPAddress
Gateway
Aliases
Ports
```

információkat.

---

# 68. `docker port`

Port mapping gyors ellenőrzésére használható:

```bash
docker port web
```

Példa eredmény:

```text
3000/tcp -> 0.0.0.0:8080
```

Mental model:

```text
container :3000
→ host :8080
```

---

# 69. `docker ps` networking szempontból

```bash
docker ps
```

`PORTS` oszlop:

```text
0.0.0.0:8080->3000/tcp
```

Ez mutatja:

```text
host :8080
↓
container :3000/TCP
```

---

# 70. Container belső network vizsgálat

Ha az image tartalmazza a szükséges eszközöket:

```bash
ip addr
ip route
ss -lnt
```

segíthetnek.

Például:

```bash
docker exec web ip addr
```

De minimal image-ekben ezek nem feltétlenül vannak telepítve.

---

# 71. `ip addr`

Containerben:

```bash
ip addr
```

megmutathatja:

```text
lo
eth0
IP address
```

Ezzel láthatjuk a network namespace saját interface-eit.

---

# 72. `ip route`

```bash
ip route
```

például:

```text
default via 172.18.0.1
172.18.0.0/16 dev eth0
```

Ez pontosan ugyanaz a routing logika, amit networkingnél tanultunk.

---

# 73. `ss`

Listening socketek:

```bash
ss -lnt
```

Például:

```text
LISTEN
0.0.0.0:3000
```

Ezzel meg tudjuk állapítani:

```text
az alkalmazás ténylegesen listenel?
```

---

# 74. Ping

Container-to-container teszthez:

```bash
ping db
```

használható, ha az image tartalmaz ping eszközt.

De fontos:

```text
ping siker
≠
application működik
```

Ping csak ICMP szintű elérhetőséget vizsgál.

---

# 75. Application port teszt

Jobb lehet például:

```bash
curl http://web:3000
```

vagy:

```bash
nc -vz db 5432
```

ha az eszközök elérhetők.

Ez már application/TCP port szintű teszt.

---

# 76. DNS teszt

Ha rendelkezésre áll:

```bash
getent hosts db
```

megmutathatja:

```text
db név
↓
IP
```

feloldást.

Vagy más DNS eszköz:

```text
nslookup
dig
```

ha telepítve van.

---

# 77. Minimal image probléma

Sok production image direkt kicsi.

Ezért lehet, hogy nincs benne:

```text
ping
curl
ip
ss
netstat
dig
bash
```

Ez nem feltétlenül hiba.

A container lehet teljesen működőképes ezek nélkül.

---

# 78. Debug container

Hálózati troubleshootingnál külön debug container indítható ugyanarra a networkre.

Mental model:

```text
Application network
├── web
├── db
└── debug container
```

A debug container tartalmazhat:

```text
curl
dig
ping
nc
ip
```

eszközöket.

Ez jobb lehet, mint production image-be telepíteni minden debug toolt.

---

# 79. Troubleshooting rétegek

Docker networking hiba esetén:

```text
1. Container lifecycle
2. Docker network membership
3. DNS
4. IP/routing
5. TCP/UDP
6. Listening socket
7. Port publishing
8. Application protocol
9. Host firewall
10. External network
```

Mindig lentről/fundamentumtól haladunk.

---

# 80. Példa: web nem éri el DB-t

Első kérdés:

```text
DB fut?
```

```bash
docker ps
```

Utána:

```text
Ugyanazon network?
```

```bash
docker network inspect app-network
```

Utána:

```text
DNS feloldódik?
```

Például debug containerből:

```text
getent hosts db
```

Utána:

```text
5432 port elérhető?
```

Majd:

```text
PostgreSQL tényleg listenel?
```

Végül:

```text
helyes username/password/database?
```

Ne kezdjünk rögtön application kódot módosítani.

---

# 81. Példa: böngésző nem éri el web containert

Vizsgálat:

```text
Container RUNNING?
↓
docker ps
```

Majd:

```text
Port mapping?
↓
docker port web
```

Majd:

```text
App listenel?
↓
0.0.0.0:3000?
```

Majd:

```text
Containerből működik?
↓
curl localhost:3000
```

Majd:

```text
Hostról localhost:8080?
```

Ez rétegenként szűkíti a hibát.

---

# 82. Connection refused

Például:

```text
connection refused
```

gyakran azt jelzi:

```text
a cél IP elérhető
↓
de azon a porton nincs listening socket
```

Docker esetben lehetséges:

```text
rossz container port
app nem fut
app másik portra listenel
app csak más interface-re bindol
```

---

# 83. Timeout

Timeout esetén lehet:

```text
routing probléma
network isolation
firewall/filter
rossz destination
nem elérhető host
```

A timeout általában kevésbé közvetlen visszajelzés, mint a connection refused.

---

# 84. DNS failure

Például:

```text
could not resolve host db
```

Első gyanú:

```text
container nincs megfelelő networkön
```

vagy:

```text
rossz service/container név
```

Ekkor nem a TCP porttal kezdünk.

Mert:

```text
DNS még nem oldódott fel
```

---

# 85. Network troubleshooting flow

```text
Container fut?
↓
Networkhöz csatlakozik?
↓
DNS működik?
↓
IP elérhető?
↓
Route jó?
↓
Port nyitva/listening?
↓
Application protocol működik?
```

Ez az egyik legfontosabb mental model.

---

# 86. Docker network security

A network izoláció hasznos, de nem teljes security modell.

További rétegek lehetnek:

```text
host firewall
application auth
TLS
secrets
container permissions
network policies
reverse proxy
```

Docker network önmagában nem helyettesíti ezeket.

---

# 87. Internal network gondolat

Dockerben lehet olyan hálózati kialakítás, ahol bizonyos networkök csak belső kommunikációra szolgálnak.

Mental model:

```text
public-facing network
↓
application

internal backend network
↓
database
```

Ezzel csökkenthető a közvetlen külső elérés.

---

# 88. Port publishing bind address

Nem feltétlenül kell minden host interface-re publikálni.

Például:

```bash
docker run -p 127.0.0.1:8080:3000 myapp
```

Mental model:

```text
csak host loopback :8080
↓
container :3000
```

Ez eltér:

```bash
docker run -p 8080:3000 myapp
```

tipikus viselkedésétől, amely platformtól/configtól függően szélesebb host interface-en lehet elérhető.

---

# 89. Miért fontos a bind address?

Production szerveren:

```text
0.0.0.0:5432
```

PostgreSQL publishing például szükségtelenül kinyithatja a szolgáltatást.

Míg:

```text
127.0.0.1:5432
```

csak host-local hozzáférést engedhet.

Még jobb lehet, ha egyáltalán nincs publishing, és csak belső Docker networkön használjuk.

---

# 90. TCP és UDP publishing

Alapértelmezésben sok példa TCP-t használ.

De UDP is publikálható.

Példa:

```bash
docker run -p 5353:5353/udp myimage
```

Ez:

```text
host UDP :5353
↓
container UDP :5353
```

Tehát a protocol is része a socket/networking modellnek.

---

# 91. TCP és UDP ugyanazt a portszámot használhatja

Mert:

```text
TCP :53
```

és:

```text
UDP :53
```

külön transport protocol namespace.

Ezért:

```text
IP + port + protocol
```

együtt fontos.

---

# 92. Docker DNS és UDP/TCP

DNS hagyományosan főként:

```text
UDP :53
```

de bizonyos esetekben:

```text
TCP :53
```

is használható.

Docker embedded DNS ezt a service discovery mechanizmus részeként kezeli a container network környezetben.

---

# 93. DNS nem routing

Fontos különbség:

```text
DNS
→ nevet IP-re old
```

```text
Routing
→ megmondja, merre menjen az IP packet
```

Például:

```text
db
↓ DNS
172.18.0.3
↓ routing
Docker network
↓
DB container
```

Mindkettő kell.

---

# 94. Bridge nem router

A bridge elsősorban Layer 2 Ethernet forwarding szerepet tölt be.

Gateway/routing/NAT más hálózati funkció.

Mental model:

```text
Bridge
→ Layer 2

Routing
→ Layer 3

Port
→ Layer 4
```

Ez a korábbi OSI/TCP-IP gondolkodáshoz kapcsolható.

---

# 95. Docker networking rétegenként

```text
Application
→ HTTP / PostgreSQL protocol

Transport
→ TCP / UDP + ports

Network
→ IP + routing

Link
→ Ethernet / bridge / veth
```

Docker mindegyik réteget érintheti, de más-más módon.

---

# 96. Teljes container-to-container flow

Példa:

```text
web
→ db:5432
```

Flow:

```text
1. web process megkapja: db
↓
2. Docker DNS feloldja
↓
3. destination IP = 172.18.0.3
↓
4. routing table szerint same subnet
↓
5. ARP/neighbor feloldás
↓
6. Ethernet frame → eth0
↓
7. veth
↓
8. Linux bridge
↓
9. db veth
↓
10. db eth0
↓
11. TCP :5432
↓
12. PostgreSQL socket
```

Ez már a teljes networking gondolkodás.

---

# 97. Teljes container-to-internet flow

```text
Container
172.18.0.2
↓
destination: example.com
↓
DNS
↓
public IP
↓
routing table
↓
default gateway
↓
Docker backend
↓
NAT
↓
host/external network
↓
internet
```

A válasz:

```text
Internet
↓
host/backend
↓
connection tracking / NAT reverse mapping
↓
container
```

---

# 98. Teljes host-to-container flow

```text
Browser
↓
localhost:8080
↓
host socket/network
↓
Docker port publishing
↓
container IP:3000
↓
container eth0
↓
TCP listening socket
↓
Next.js
```

Ez három nagyon különböző flow:

```text
container → container
container → internet
host → container
```

Mindegyiket külön kell tudni fejben követni.

---

# 99. Docker networking cheatsheet

```text
Network namespace
→ saját network view
```

```text
veth pair
→ virtuális Ethernet kapcsolat
```

```text
Linux bridge
→ virtuális Layer 2 switch
```

```text
Gateway
→ más hálózatok felé vezet
```

```text
DNS
→ név → IP
```

```text
NAT
→ cím/port átírás
```

```text
-p
→ host port publishing
```

```text
User-defined bridge
→ saját alkalmazás network
```

```text
host network
→ host network stackhez szorosabb kapcsolat
```

```text
none network
→ nincs normál external networking
```

---

# 100. Legfontosabb parancsok

Network lista:

```bash
docker network ls
```

Network létrehozás:

```bash
docker network create app-network
```

Inspect:

```bash
docker network inspect app-network
```

Container indítás networkön:

```bash
docker run --network app-network ...
```

Csatlakoztatás:

```bash
docker network connect app-network container
```

Leválasztás:

```bash
docker network disconnect app-network container
```

Port publishing:

```bash
docker run -p 8080:3000 myapp
```

Csak localhost publishing:

```bash
docker run -p 127.0.0.1:8080:3000 myapp
```

UDP:

```bash
docker run -p 5353:5353/udp myapp
```

Port mapping:

```bash
docker port container
```

Container inspect:

```bash
docker inspect container
```

---

# 101. Troubleshooting parancsok

Docker oldal:

```bash
docker ps
docker inspect container
docker network ls
docker network inspect network
docker port container
docker logs container
```

Containerben, ha elérhetők:

```bash
ip addr
ip route
ss -lnt
getent hosts db
ping db
curl http://web:3000
nc -vz db 5432
```

---

# 102. A legfontosabb troubleshooting kérdések

```text
Fut a container?
↓
Mely networkön van?
↓
Milyen IP-t kapott?
↓
Működik a DNS?
↓
Van route?
↓
Mely portot használjuk?
↓
Listenel rajta valami?
↓
Melyik interface-en listenel?
↓
Kell port publishing?
↓
Helyes a host port?
↓
Firewall/filter akadályozza?
```

---

# 103. Amit nem szabad összekeverni

```text
Container name
≠
IP
```

```text
DNS
≠
routing
```

```text
Bridge
≠
router
```

```text
Container port
≠
host port
```

```text
EXPOSE
≠
publish
```

```text
localhost
≠
másik container
```

```text
volume
≠
network
```

```text
Docker network isolation
≠
teljes security
```

---

# 104. Kapcsolat a korábbi networking tananyaggal

A korábbi hálózatos flow:

```text
DNS
↓
IP
↓
subnet
↓
routing
↓
gateway
↓
TCP
↓
port
↓
socket
```

Dockerben:

```text
Docker DNS
↓
container IP
↓
Docker subnet
↓
container routing
↓
Docker gateway
↓
TCP
↓
container port
↓
application socket
```

Tehát ugyanazokat az alapokat használjuk.

Docker csak:

```text
létrehozza
automatizálja
izolálja
összekapcsolja
```

ezeket a hálózati elemeket.

---

# 105. DevOps szempontból miért fontos ezt érteni?

Mert tipikus hibák:

```text
app nem éri el DB-t
reverse proxy nem éri el appot
port nem elérhető
container internet nélkül marad
rossz DNS név
rossz network membership
localhost hibás használata
rossz interface bind
```

mind networking problémák lehetnek.

Ha csak Docker parancsokat tudunk:

```text
nehéz diagnosztizálni
```

Ha értjük:

```text
IP
route
DNS
port
socket
bridge
NAT
```

akkor rétegenként meg tudjuk találni a hibát.

---

# 106. Teljes mental model

```text
                     INTERNET
                         │
                         │
                        NAT
                         │
                         ↓
                 Docker Host/Backend
                         │
              ┌──────────┴──────────┐
              │                     │
        Port Publishing        Linux Bridge
        host :8080                  │
              │               ┌─────┴─────┐
              │               │           │
              ↓              veth        veth
        Web Container         │           │
       172.18.0.2:3000        ↓           ↓
                           Web          Database
                                         │
                                      :5432
```

Egy másik nézet:

```text
Browser
↓
Host :8080
↓
Docker publishing
↓
Web :3000
↓
Docker DNS
↓
db
↓
Docker bridge
↓
Database :5432
```

---

# 107. A három legfontosabb flow

## Host → Container

```text
Host
↓
published port
↓
Container
```

Példa:

```text
localhost:8080
↓
web:3000
```

---

## Container → Container

```text
Web
↓
DNS
↓
DB IP
↓
TCP
↓
DB port
```

Példa:

```text
db:5432
```

---

## Container → Internet

```text
Container
↓
default gateway
↓
Docker backend
↓
NAT
↓
Internet
```

---

# 108. Későbbi Docker networking labor

A laborban végig fogjuk próbálni:

```text
1. user-defined bridge létrehozása
2. subnet vizsgálata
3. container IP vizsgálata
4. gateway vizsgálata
5. routing table vizsgálata
6. két container közötti kapcsolat
7. DNS névfeloldás
8. fix IP helyett service név
9. port publishing
10. host port vs container port
11. 127.0.0.1 vs 0.0.0.0
12. container internet access
13. network isolation
14. két külön Docker network
15. container két networkön
16. network connect/disconnect
17. host network
18. none network
19. debug container használata
20. szándékosan elrontott networking hibák
```

A hibák között lesz például:

```text
rossz DNS név
rossz port
rossz network
localhost használata
nem publikált port
rossz host port
127.0.0.1-re bindolt alkalmazás
leállított service
```

Minden esetben a cél:

> Ne találgassunk, hanem networking rétegenként bizonyítsuk, hol szakad meg a kapcsolat.

---

# 109. Fejezet végső összefoglaló

A Docker networking belső logikája:

```text
Container process
↓
network namespace
↓
virtual interface
↓
veth pair
↓
Linux bridge
↓
routing / gateway
↓
NAT / external network
```

Container-to-container esetén:

```text
service name
↓
Docker DNS
↓
container IP
↓
bridge
↓
destination container
```

Host-to-container esetén:

```text
host port
↓
Docker port publishing
↓
container port
↓
application socket
```

És a legfontosabb gondolat:

> Docker networking nem egy külön „mágikus Docker hálózat”, hanem Linux networking elemekre épülő automatizált és izolált hálózati környezet.

Ezért a Docker networking hibakeresésnél ugyanazokat a kérdéseket tesszük fel, mint bármely más hálózatnál:

```text
Mi a source?
Mi a destination?
Mi az IP?
Mi a subnet?
Mi a route?
Mi a gateway?
Működik a DNS?
Melyik port?
Melyik protokoll?
Van listening socket?
Hol szakad meg a kapcsolat?
```