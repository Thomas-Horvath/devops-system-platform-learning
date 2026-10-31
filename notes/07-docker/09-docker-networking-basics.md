# Docker Networking Basics

## 1. Mi ennek a fejezetnek a célja?

Ebben a fejezetben azt nézzük meg, hogyan kommunikálnak egymással a Docker containerek, hogyan érik el a hostot és az internetet, illetve hogyan tesszük elérhetővé a containerben futó szolgáltatásokat kívülről.

A Docker networking alapfogalmai szorosan kapcsolódnak a korábbi networking témákhoz:

```text
IP
port
TCP / UDP
DNS
routing
NAT
gateway
socket
firewall
```

A különbség az, hogy ezek most Docker által létrehozott virtuális hálózatokon jelennek meg.

---

# 2. Miért kell Docker networking?

Tegyük fel, hogy fut két container:

```text
Container A
→ Next.js

Container B
→ PostgreSQL
```

A Next.js alkalmazásnak kommunikálnia kell a PostgreSQL adatbázissal:

```text
Next.js
↓
TCP
↓
PostgreSQL :5432
```

Másik irányban a böngészőnek is el kell érnie a webalkalmazást:

```text
Browser
↓
Host
↓
Docker
↓
Next.js container :3000
```

Tehát Dockerben is ugyanazokat a kérdéseket tesszük fel:

```text
Mi a destination IP?
Mi a destination port?
Milyen protokoll?
Van route?
Van DNS?
Van NAT?
Publikálva van a port?
```

---

# 3. A container hálózati szempontból külön környezet

Egy containernek lehet:

```text
saját network namespace
saját network interface
saját IP address
saját routing table
saját localhost
saját listening socketek
```

Például:

```text
Docker Host
│
├── Container A
│     IP: 172.18.0.2
│
└── Container B
      IP: 172.18.0.3
```

Ha ugyanazon Docker networkön vannak:

```text
172.18.0.2
↓
TCP
↓
172.18.0.3:5432
```

kommunikáció létrejöhet közöttük.

---

# 4. Network namespace

Korábban megtanultuk:

```text
namespace
→ mit láthat egy process?
```

Networking esetén:

```text
network namespace
→ milyen network környezetet lát a process?
```

Egy container process például saját maga szempontjából ezt láthatja:

```text
eth0
lo
IP address
routes
gateway
ports
sockets
```

Ez elkülönül más containerek hálózati nézetétől.

---

# 5. Saját `localhost`

Nagyon fontos Docker networking szabály:

> A `localhost` mindig annak a hálózati namespace-nek a saját loopback interface-e, amelyben a process fut.

Ha két külön containerünk van:

```text
Container A
→ application

Container B
→ database
```

és Container A-ban ezt használjuk:

```text
localhost:5432
```

akkor nem Container B-t keressük.

Hanem:

```text
Container A saját localhostját
```

Mental model:

```text
Container A

localhost
↓
Container A
```

Nem:

```text
localhost
↓
host
↓
Container B
```

---

# 6. Tipikus `localhost` hiba

Hoston egy alkalmazás konfigurációja lehet:

```text
DATABASE_URL=postgres://localhost:5432/db
```

Ez működhet, ha PostgreSQL is közvetlenül a hoston fut.

De ha:

```text
Application
→ Container A

PostgreSQL
→ Container B
```

akkor Container A-ból:

```text
localhost
```

Container A saját loopback címe.

Ezért külön containerek között inkább:

```text
db:5432
```

vagy:

```text
database:5432
```

jellegű service/container nevet használunk.

---

# 7. Docker network

Docker külön network objektumokat kezel.

Listázás:

```bash
docker network ls
```

Tipikusan:

```text
NETWORK ID     NAME      DRIVER
...            bridge    bridge
...            host      host
...            none      null
```

Ebben a fejezetben a legfontosabb:

```text
bridge
```

network driver.

---

# 8. Bridge network

A bridge network egy virtuális hálózat a Docker hoston.

Egyszerűsítve:

```text
             Docker Host

        Docker bridge network
        172.18.0.0/16
              │
       ┌──────┴──────┐
       │             │
 Container A     Container B
172.18.0.2       172.18.0.3
```

Nagyon jó mental model:

> A Docker bridge network olyan, mint egy virtuális LAN a containerek számára.

Normál LAN:

```text
LAN
├── PC A
├── PC B
└── PC C
```

Docker:

```text
Docker bridge
├── Container A
├── Container B
└── Container C
```

---

# 9. Virtuális switch mental model

Egyszerűsítve:

```text
Container A
     │
     │ virtual Ethernet
     ↓
┌─────────────────────┐
│ Docker bridge       │
│ virtuális switch    │
└─────────────────────┘
     ↑
     │ virtual Ethernet
     │
Container B
```

Natív Linuxon a háttérben többek között:

```text
network namespace
veth pair
Linux bridge
routing
NAT
```

mechanizmusok jelenhetnek meg.

Most a logikai modell a fontos.

---

# 10. Container network interface

Containeren belül jellemzően lehet például:

```text
eth0
```

network interface.

Mental model:

```text
Container network namespace

eth0
│
├── IP address
├── subnet
├── routes
└── gateway
```

Ez ugyanaz a networking logika, amit Linuxnál már tanultunk.

---

# 11. Container → container kommunikáció

Saját Docker network létrehozása:

```bash
docker network create app-network
```

Majd például:

```text
app-network
├── web
└── db
```

Ha mindkét container ehhez a networkhöz csatlakozik:

```text
web
↓
db:5432
↓
PostgreSQL
```

kommunikáció létrejöhet.

---

# 12. User-defined bridge network

Saját bridge network létrehozása:

```bash
docker network create app-network
```

Container indítása ezen:

```bash
docker run --network app-network ...
```

A user-defined bridge network előnyei:

```text
jobb izoláció
jobb névfeloldás
logikus alkalmazáscsoportosítás
egyszerűbb service-to-service kommunikáció
```

Docker Compose később automatikusan is létrehozhat saját networköt.

---

# 13. Docker DNS

User-defined Docker networkön a containerek nevei alapján is kommunikálhatnak.

Például:

```text
web
database
```

A web containerből:

```text
database
```

név feloldható a database container IP-jére.

Mental model:

```text
web container
↓
DNS query: database
↓
Docker DNS
↓
172.18.0.3
↓
database container
```

Ezért configban gyakran jobb:

```text
DB_HOST=database
```

mint:

```text
DB_HOST=172.18.0.3
```

---

# 14. Miért ne használjunk fix container IP-t?

Container újralétrehozásakor az IP változhat.

Például:

```text
ma:

database
→ 172.18.0.3
```

később:

```text
database
→ 172.18.0.6
```

A logikai név viszont maradhat:

```text
database
```

Ezért:

```text
DNS name
→ stabil logikai referencia
```

míg:

```text
container IP
→ runtime részlet
```

---

# 15. Container port

Tegyük fel:

```text
Container IP:
172.18.0.2

Application port:
3000
```

A szolgáltatás a Docker networkön belül például:

```text
172.18.0.2:3000
```

címen lehet elérhető.

Ez azonban még nem jelenti azt, hogy a hostról automatikusan elérhető:

```text
localhost:3000
```

címen.

Ehhez kell a port publishing.

---

# 16. Port publishing

Példa:

```bash
docker run -p 8080:3000 myapp
```

A szintaxis:

```text
HOST PORT : CONTAINER PORT
```

Tehát:

```text
8080 : 3000
```

jelentése:

```text
host port 8080
↓
container port 3000
```

---

# 17. Port publishing mental model

```text
Browser
   │
   │ localhost:8080
   ↓
Docker Host
   │
   │ published port
   ↓
Container
   │
   │ :3000
   ↓
Next.js
```

A container alkalmazása továbbra is:

```text
3000
```

porton fut.

Docker kívül:

```text
8080
```

portról teszi elérhetővé.

---

# 18. Host port és container port különbözhet

Például:

```bash
docker run -p 9000:3000 myapp
```

A containerben:

```text
application
→ :3000
```

A hoston:

```text
localhost:9000
```

használható.

Az alkalmazásnak nem kell tudnia, milyen host portra publikáltuk.

---

# 19. Több azonos container ugyanazzal a belső porttal

Tegyük fel:

```text
Container A
→ :3000

Container B
→ :3000
```

Ez nem probléma.

Mindegyik saját network namespace-ben van.

Hoston viszont külön portok kellenek:

```bash
docker run -p 3001:3000 myapp
docker run -p 3002:3000 myapp
```

Ekkor:

```text
localhost:3001
↓
Container A :3000
```

és:

```text
localhost:3002
↓
Container B :3000
```

---

# 20. `EXPOSE` vs `-p`

Dockerfile:

```dockerfile
EXPOSE 3000
```

jelentése:

```text
image metadata
+
dokumentáció arra, hogy az alkalmazás várhatóan ezt a portot használja
```

Ez nem jelent automatikus host publishingot.

Tényleges publishing:

```bash
docker run -p 8080:3000 myapp
```

Tehát:

```text
EXPOSE
≠
-p
```

---

# 21. Nem kell minden portot publikálni

Például:

```text
Web container
→ hostról el kell érni
→ publish
```

Database:

```text
DB container
→ csak web container használja
→ nem feltétlenül publish
```

Ez security szempontból is fontos.

---

# 22. Példa belső adatbázis kapcsolatra

```text
Internet / Browser
        │
        ↓
host :8080
        │
        ↓
Web Container
     :3000
        │
        │ internal Docker network
        ↓
DB Container
     :5432
```

A hostnak csak:

```text
8080
```

portot tesszük ki.

A PostgreSQL:

```text
5432
```

portját nem feltétlenül publikáljuk a hostra.

---

# 23. Miért biztonságosabb ez?

Ha PostgreSQL csak belső Docker networkön érhető el:

```text
Web
↓
db:5432
```

akkor nincs szükség:

```text
host:5432
```

publikálására.

Ez csökkenti a fölöslegesen elérhető service-ek számát.

Mental model:

```text
csak azt publikáljuk,
amit ténylegesen kívülről is el kell érni
```

---

# 24. Container → internet

A container jellemzően elérheti a külső internetet.

Flow:

```text
Container
↓
Docker network
↓
Docker host/backend
↓
NAT
↓
external network
↓
Internet
```

Például:

```bash
curl https://example.com
```

működhet containerből.

---

# 25. NAT

A container például ezt az IP-t használhatja:

```text
172.18.0.2
```

Ez privát cím.

Az internet felé Docker/host NAT-ot használhat:

```text
172.18.0.2
↓
Docker host
↓
NAT
↓
host külső címe
↓
Internet
```

---

# 26. Otthoni router analógia

Otthon:

```text
Laptop
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

Az alapelv ugyanaz.

---

# 27. Port publishing és NAT

Bejövő kapcsolatnál:

```text
host:8080
↓
Docker networking
↓
container:3000
```

Kimenő kapcsolatnál:

```text
container private IP
↓
Docker host/backend
↓
NAT
↓
internet
```

Így Docker mindkét irányban kezeli a hálózati átmenetet.

---

# 28. Network isolation

Külön Docker networkök külön logikai hálózati tartományokat adhatnak.

Például:

```text
Network A
├── web
└── db

Network B
└── admin
```

Ha `admin` nincs a Network A hálózathoz csatlakoztatva:

```text
admin
↛
db
```

közvetlen kapcsolat nem feltétlenül lehetséges.

Mental model:

```text
külön Docker network
≈
külön virtuális LAN
```

---

# 29. Egy container több networkön is lehet

Példa:

```text
Frontend
   │
   │ frontend-network
   ↓
   API
   │
   │ backend-network
   ↓
Database
```

Az API:

```text
frontend-network
+
backend-network
```

mindkettőhöz tartozhat.

A frontend viszont nincs közvetlenül a backend networkön.

Ez hálózati szegmentációt ad.

---

# 30. Több networkös architektúra

```text
frontend-network

Frontend
   │
   ↓
API
   │
   │
   │ backend-network
   ↓
Database
```

Ennek előnye:

```text
Frontend
→ API elérhető

API
→ Database elérhető

Frontend
→ Database nem feltétlenül érhető el
```

Ez nagyon hasonló valódi hálózati szegmentációhoz.

---

# 31. `docker network inspect`

Network vizsgálata:

```bash
docker network inspect app-network
```

Megnézhetünk többek között:

```text
subnet
gateway
connected containers
container IP addresses
network driver
```

információkat.

Ez troubleshootingnál nagyon hasznos.

---

# 32. `docker inspect` és networking

Container részletes vizsgálata:

```bash
docker inspect mycontainer
```

A `NetworkSettings` részben például:

```text
IP address
network
gateway
ports
```

adatok jelenhetnek meg.

---

# 33. Networking troubleshooting – alapmódszer

Tegyük fel:

> A web container nem éri el a database containert.

Nem találomra próbálkozunk.

Menjünk sorrendben:

```text
1. Fut a web container?
↓
2. Fut a DB container?
↓
3. Ugyanazon networkön vannak?
↓
4. A DNS név működik?
↓
5. Helyes destination portot használunk?
↓
6. A DB process valóban listenel?
↓
7. Jó az application config?
```

A fő kérdés:

> Mi az utolsó pont, amelyről biztosan tudjuk, hogy működik?

---

# 34. Tipikus `localhost` troubleshooting

Ha application config:

```text
DB_HOST=localhost
```

de a DB külön container:

```text
Application container
DB container
```

akkor ez gyanús.

Helyette például:

```text
DB_HOST=db
```

ahol:

```text
db
```

a Docker networkön feloldható név.

---

# 35. Port troubleshooting

Például:

```text
localhost:8080
```

nem működik.

Vizsgálat:

```text
Container fut?
↓
docker ps
↓
Port publishing létezik?
↓
8080:3000?
↓
Application fut?
↓
Application :3000-en listenel?
↓
Milyen interface-re bindolt?
```

---

# 36. `127.0.0.1` vs `0.0.0.0`

Ez nagyon fontos containerben futó webalkalmazásnál.

Ha az application csak:

```text
127.0.0.1:3000
```

címen listenel:

```text
csak a container saját loopback interface-én érhető el
```

Ha:

```text
0.0.0.0:3000
```

címen listenel:

```text
minden container interface-en fogad kapcsolatot
```

Docker port publishinghez gyakran ez szükséges.

---

# 37. Loopback mental model

```text
127.0.0.1
→ saját network namespace loopback
```

Containerben:

```text
127.0.0.1
→ maga a container
```

Hoston:

```text
127.0.0.1
→ maga a host
```

Ezért a két `localhost` nem ugyanaz.

---

# 38. `0.0.0.0`

Listening socket esetén:

```text
0.0.0.0
```

jelentése nagyjából:

```text
minden IPv4 interface-en listenelj
```

Például:

```text
0.0.0.0:3000
```

így a service a container `eth0` interface-én keresztül is elérhető lehet.

---

# 39. Socket mental model

Korábban:

```text
socket
≈
IP + port + protocol
```

Például containerben:

```text
0.0.0.0:3000/TCP
```

egy listening socket.

Docker publishing:

```text
host :8080
↓
container :3000
```

A hálózati kapcsolat végül a containerben futó listening sockethez jut.

---

# 40. Teljes request flow

Tegyük fel:

```bash
docker run -p 8080:3000 myapp
```

és az app:

```text
0.0.0.0:3000
```

címen listenel.

Flow:

```text
Browser
↓
localhost:8080
↓
Host network
↓
Docker port publishing
↓
Container network
↓
Container :3000
↓
Application socket
```

---

# 41. Container → database flow

Tegyük fel:

```text
Web container
DB container
```

ugyanazon networkön vannak.

App config:

```text
DB_HOST=db
DB_PORT=5432
```

Flow:

```text
Application
↓
DNS query: db
↓
Docker DNS
↓
DB container IP
↓
TCP :5432
↓
PostgreSQL
```

---

# 42. Host → container és container → container különbség

Host → container:

```text
host port publishing szükséges lehet
```

Példa:

```text
localhost:8080
↓
container:3000
```

Container → container ugyanazon Docker networkön:

```text
host port publishing nem feltétlenül szükséges
```

Példa:

```text
web
↓
db:5432
```

Ez nagyon fontos különbség.

---

# 43. Port publishing nem kell belső kommunikációhoz

Ha:

```text
web
```

és:

```text
db
```

ugyanazon Docker networkön vannak, akkor:

```text
web → db:5432
```

kapcsolathoz nincs szükség:

```text
-p 5432:5432
```

publishingra.

A `-p` akkor kell, ha a host vagy külső rendszer felől akarjuk elérni.

---

# 44. Docker bridge vs user-defined bridge

Docker alapból rendelkezik:

```text
bridge
```

networkkel.

Saját létrehozása:

```bash
docker network create app-network
```

A user-defined bridge általában jobb alkalmazás-szintű használatra.

Előnyök:

```text
automatikus névfeloldás
jobb izoláció
logikus network design
könnyebb troubleshooting
```

---

# 45. `host` network röviden

Dockerben létezik:

```text
host
```

network driver is.

Ennél a container hálózati izolációja eltér a bridge modelltől, és közelebb kerülhet a host network stackjéhez.

Ezt most nem használjuk alapértelmezettként.

Részletesebben a deep dive fejezetben.

---

# 46. `none` network röviden

Létezik:

```text
none
```

network mód is.

Ez olyan containerhez használható, amelynek nincs szüksége normál külső hálózati kapcsolatra.

Mental model:

```text
container
↓
network isolation
↓
nincs normál external Docker networking
```

Ezt is később részletezzük.

---

# 47. Windows + Docker Desktop sajátosság

A jelenlegi környezetünk:

```text
Windows 11
↓
Docker Desktop
↓
WSL2 / Linux backend
↓
Docker Engine
↓
Docker networks
↓
Linux containers
```

Ezért a fizikai Windows host és a Linux container közötti háttér-implementáció összetettebb lehet, mint natív Linux host esetén.

A Docker azonban egységes logikai modellt ad:

```text
Docker network
container IP
port publishing
DNS
routing
NAT
```

Egyelőre ezt a modellt tanuljuk.

---

# 48. Docker Desktop és container IP

Windows + Docker Desktop esetén nem érdemes abból kiindulni, hogy a container belső IP-jét közvetlenül Windows hálózati címként fogjuk használni.

A stabil interface inkább:

```text
port publishing
```

Például:

```text
localhost:8080
↓
Docker Desktop networking
↓
container:3000
```

A belső container IP inkább Docker network runtime részlet.

---

# 49. Teljes alaparchitektúra

```text
                       INTERNET
                          ↑
                          │
                         NAT
                          │
                    Docker Host
                          │
              ┌───────────┴───────────┐
              │                       │
       published :8080          Docker bridge
              │                       │
              ↓               ┌───────┴───────┐
        Web Container          │               │
           :3000             Web             DB
                              │               │
                         172.18.0.2      172.18.0.3
                                              │
                                            :5432
```

---

# 50. Application flow

Kívülről:

```text
Browser
↓
host:8080
↓
web:3000
```

Belső DB kapcsolat:

```text
web
↓
db:5432
```

Itt:

```text
db
→ Docker DNS name
```

nem:

```text
localhost
```

---

# 51. Networking security alapelv

Csak azt a portot publikáljuk, amelyet ténylegesen kívülről is el kell érni.

Például:

```text
Web :3000
→ publish
```

Database:

```text
PostgreSQL :5432
→ internal only
```

Ez a:

```text
least exposure
```

gondolkodás.

---

# 52. Docker network mint security boundary

A Docker network nem teljes security megoldás önmagában, de hálózati izolációt adhat.

Például:

```text
public-network
├── reverse-proxy
└── app
```

és:

```text
backend-network
├── app
└── db
```

A database nincs a public networkön.

Ez csökkentheti a fölösleges hozzáférést.

---

# 53. Tipikus háromrétegű Docker architektúra

```text
Browser
↓
Reverse Proxy
↓
Application
↓
Database
```

Docker networkekkel:

```text
frontend-network
├── reverse-proxy
└── application

backend-network
├── application
└── database
```

Az application két networkön lehet.

---

# 54. Miért jó ez?

```text
reverse proxy
→ nem éri el közvetlenül feltétlenül a DB-t

database
→ nincs public networkön

application
→ közvetítő réteg
```

Ez már valódi rendszertervezési gondolkodás.

---

# 55. Docker networking troubleshooting kérdések

Ha kapcsolat nem működik:

```text
1. Container fut?
2. Service process fut?
3. Service milyen IP/interface-en listenel?
4. Helyes port?
5. Ugyanazon network?
6. DNS név feloldódik?
7. Routing működik?
8. Port publish megfelelő?
9. Host firewall?
10. Application config jó?
```

Nem mindig kell mindegyiket végignézni, de ez jó sorrend.

---

# 56. Timeout vs connection refused Dockerben

Korábbi networkingből:

```text
connection refused
```

gyakran azt jelenti:

```text
elértük a hostot/IP-t
↓
de nincs megfelelő listening service
```

Dockerben például:

```text
container elérhető
↓
de app nem listenel :3000
```

Timeout:

```text
útvonal / filtering / network / firewall / routing probléma is lehet
```

A pontos okot mindig rétegenként vizsgáljuk.

---

# 57. Container DNS troubleshooting

Ha:

```text
web → db
```

nem működik:

```text
Ugyanazon user-defined networkön vannak?
↓
A név tényleg `db`?
↓
Container létezik?
↓
DNS feloldás működik?
```

A fix IP használata inkább troubleshooting segédeszköz lehet, nem normál application config.

---

# 58. Port mapping troubleshooting

Ha:

```text
localhost:8080
```

nem működik:

```text
docker ps
↓
PORTS oszlop
```

például:

```text
0.0.0.0:8080->3000/tcp
```

jelzi a mappinget.

Utána:

```text
app tényleg :3000-en listenel?
```

---

# 59. Docker networking fontos parancsok

Network lista:

```bash
docker network ls
```

Network létrehozás:

```bash
docker network create app-network
```

Network inspect:

```bash
docker network inspect app-network
```

Container network megadása:

```bash
docker run --network app-network ...
```

Container inspect:

```bash
docker inspect mycontainer
```

Port publishing:

```bash
docker run -p 8080:3000 myapp
```

---

# 60. Container csatlakoztatása networkhöz

Futó container is csatlakoztatható networkhöz:

```bash
docker network connect app-network mycontainer
```

Leválasztás:

```bash
docker network disconnect app-network mycontainer
```

Ez mutatja, hogy:

```text
container
```

és:

```text
network
```

külön Docker object.

---

# 61. Container több networkön

Példa:

```bash
docker network connect backend-network api
```

Ha az `api` már a frontend networkön van:

```text
API
├── frontend-network
└── backend-network
```

két külön network interface / network kapcsolatot kaphat.

---

# 62. Network lifecycle

```text
docker network create
↓
network létezik
↓
containerek csatlakoznak
↓
kommunikáció
↓
containerek leválnak
↓
docker network rm
```

Network törlés:

```bash
docker network rm app-network
```

Használatban lévő networköt Docker nem feltétlenül enged egyszerűen törölni.

---

# 63. Docker network és subnet

Docker network saját subnetet használhat.

Például:

```text
172.18.0.0/16
```

Container:

```text
172.18.0.2
172.18.0.3
```

Gateway például:

```text
172.18.0.1
```

A pontos címeket Docker választhatja.

Nem kell ezeket kézzel megjegyezni.

---

# 64. Ugyanazon subnet mental model

Ha:

```text
Container A
172.18.0.2
```

és:

```text
Container B
172.18.0.3
```

ugyanazon Docker subnetben vannak:

```text
172.18.0.0/16
```

akkor ugyanazon virtuális LAN-on belül kommunikálhatnak.

Ez ugyanaz a networking logika, mint amit korábban subnettingnél tanultunk.

---

# 65. Gateway

A Docker bridge networknek lehet gateway címe.

Például:

```text
172.18.0.1
```

A container routing táblájában lehet default route:

```text
default
↓
Docker gateway
```

Ezen keresztül érhet el más hálózatokat.

---

# 66. Container routing mental model

```text
Destination ugyanazon subnet?
↓
közvetlen Docker network kommunikáció

Destination más hálózat?
↓
gateway
↓
Docker host/backend routing
```

Ugyanaz az alapelv, mint normál IP networkingnél.

---

# 67. Docker networking és korábbi tudás

A korábbi networking tananyag:

```text
IP addressing
subnet
gateway
routing
DNS
TCP
ports
sockets
NAT
firewall
```

Dockerben mind visszatér.

A Docker nem lecseréli ezeket.

Hanem:

> automatizálja és virtualizált környezetben alkalmazza őket.

---

# 68. Docker networking mental model

```text
Container
↓
network namespace
↓
virtual interface
↓
Docker network
↓
routing / DNS / NAT
↓
másik container / host / internet
```

---

# 69. Host → container

```text
HOST
↓
published port
↓
Docker networking
↓
CONTAINER
```

Példa:

```text
localhost:8080
↓
container:3000
```

---

# 70. Container → container

```text
WEB
↓
Docker DNS
↓
DB IP
↓
TCP :5432
↓
DATABASE
```

Példa:

```text
db:5432
```

---

# 71. Container → internet

```text
Container private IP
↓
Docker gateway
↓
Docker host/backend
↓
NAT
↓
Internet
```

---

# 72. Internet → container

Csak akkor automatikusan elérhető, ha megfelelő publishing / proxy / routing van.

Tipikus:

```text
Internet
↓
Host public IP
↓
published port / reverse proxy
↓
container
```

Productionben sokszor:

```text
Internet
↓
Nginx reverse proxy
↓
application container
```

---

# 73. Kapcsolat a saját szerveres tanulással

Később a saját szerveren:

```text
Internet
↓
DNS
↓
HTTPS
↓
Nginx
↓
Docker network
↓
Application container
↓
Database
```

teljes request flow-t fogunk elemezni.

Ez összeköti:

```text
Networking
Linux
Nginx
Docker
Next.js
Database
Security
```

témákat.

---

# 74. Gyakori félreértések

## `localhost` a hostot jelenti

Nem containerből.

```text
container localhost
→ container saját loopback
```

---

## Container IP mindig fix

Nem.

```text
container recreate
→ IP változhat
```

Használjunk logikai DNS nevet.

---

## `EXPOSE 3000` publikálja a portot

Nem.

```text
EXPOSE
→ metadata
```

Publishing:

```bash
-p 8080:3000
```

---

## Minden DB portot ki kell publikálni

Nem.

Belső Docker networkön publication nélkül is kommunikálhatnak.

---

## Két container nem használhatja ugyanazt a portot

De igen.

Saját network namespace miatt:

```text
Container A :3000
Container B :3000
```

teljesen rendben van.

Csak ugyanazt a host portot nem lehet egyszerre ugyanúgy bindolni.

---

## Docker networking teljesen új hálózati rendszer

Nem.

Ugyanazokra az alapokra épül:

```text
IP
routing
DNS
TCP/UDP
NAT
ports
sockets
```

---

# 75. Rövid cheatsheet

Network lista:

```bash
docker network ls
```

Saját network:

```bash
docker network create app-network
```

Container networkön:

```bash
docker run --network app-network myimage
```

Network inspect:

```bash
docker network inspect app-network
```

Container inspect:

```bash
docker inspect mycontainer
```

Port publishing:

```bash
docker run -p 8080:3000 myimage
```

Network connect:

```bash
docker network connect app-network mycontainer
```

Network disconnect:

```bash
docker network disconnect app-network mycontainer
```

Network törlés:

```bash
docker network rm app-network
```

---

# 76. Port mapping cheatsheet

```text
-p HOST:CONTAINER
```

Példa:

```bash
docker run -p 8080:3000 app
```

Jelentése:

```text
host :8080
↓
container :3000
```

---

# 77. DNS cheatsheet

Ne:

```text
DB_HOST=172.18.0.3
```

Inkább:

```text
DB_HOST=db
```

Ha a `db` service/container ugyanazon megfelelő Docker networkön található.

---

# 78. `localhost` cheatsheet

Hoston:

```text
localhost
→ host
```

Container A-ban:

```text
localhost
→ Container A
```

Container B-ben:

```text
localhost
→ Container B
```

Mindegyik saját network namespace.

---

# 79. Listening cheatsheet

```text
127.0.0.1:3000
→ csak local loopback
```

```text
0.0.0.0:3000
→ minden IPv4 interface
```

Containeres webservice-nél gyakran:

```text
0.0.0.0
```

szükséges.

---

# 80. Security cheatsheet

```text
Csak szükséges portot publisholj.
```

```text
Database lehetőleg maradjon internal networkön.
```

```text
Használj külön networköket logikai izolációra.
```

```text
Ne használj fix container IP-t application configban.
```

---

# 81. Troubleshooting cheatsheet

Ha container nem ér el containert:

```text
futnak?
↓
azonos network?
↓
DNS?
↓
helyes port?
↓
service listenel?
↓
application config?
```

Ha host nem ér el containert:

```text
container fut?
↓
port published?
↓
host port jó?
↓
container port jó?
↓
app listenel?
↓
0.0.0.0 vagy 127.0.0.1?
```

---

# 82. Teljes Docker networking mental model

```text
                         INTERNET
                            │
                            │
                           NAT
                            │
                            ↓
                       Docker Host
                            │
            ┌───────────────┴───────────────┐
            │                               │
      Published Port                  Docker Network
            │                               │
            ↓                       ┌───────┴───────┐
       Web Container                │               │
          :3000                   Web              DB
                                    │               │
                                DNS name: db      :5432
```

Kívülről:

```text
Browser
↓
host:8080
↓
web:3000
```

Belső kommunikáció:

```text
web
↓
db:5432
```

Kifelé:

```text
container
↓
gateway
↓
NAT
↓
internet
```

---

# 83. A legfontosabb alapelvek

```text
Docker network
→ virtuális hálózat containereknek
```

```text
Container
→ saját network namespace
```

```text
localhost
→ mindig az adott namespace saját loopbackje
```

```text
Bridge network
→ virtuális LAN jellegű kapcsolat
```

```text
Docker DNS
→ container/service név → IP
```

```text
-p
→ host port → container port
```

```text
EXPOSE
→ metadata, nem publishing
```

```text
Container → Internet
→ jellemzően NAT-on keresztül
```

```text
Internal service
→ nem kell feltétlenül publikálni
```

```text
User-defined network
→ jobb izoláció és service discovery
```

---

# 84. Kapcsolat a következő fejezettel

A következő rész:

```text
10-docker-networking-deep-dive.md
```

Ott részletesebben megnézzük:

```text
veth pair
Linux bridge
container network namespace
gateway
routing
NAT
port publishing mélyebben
DNS működés
default bridge vs user-defined bridge
host network
none network
network isolation
több network
Docker Desktop sajátosságok
troubleshooting
```

Így a Docker networking teljesen össze fog kapcsolódni a korábbi Linux networking tananyaggal.

---

# 85. Későbbi networking labor

A laborban majd konkrétan kipróbáljuk:

```text
1. Docker network létrehozása
2. Két container csatlakoztatása
3. IP címek vizsgálata
4. Container → container kommunikáció
5. DNS név alapján kommunikáció
6. localhost hibás használata
7. port publishing
8. több azonos belső port
9. network isolation
10. container több networkön
11. internet elérés
12. listening interface teszt
13. hibás port mapping
14. szándékosan elrontott DNS/network setup
15. network inspect és troubleshooting
```

A cél nem csak az lesz, hogy a parancsok működjenek, hanem hogy minden tesztnél meg tudjuk mondani:

```text
source
↓
destination
↓
IP / DNS
↓
port
↓
route
↓
eredmény
```