# Network Troubleshooting

## 1. Mi a network troubleshooting célja?

A hálózati hibakeresés célja nem az, hogy találomra parancsokat futtassunk.

A legfontosabb kérdés:

> **Meddig jutott el biztosan a kommunikáció, és hol állt meg?**

Mental model:

```text
Interface
↓
IP address
↓
Subnet
↓
Route / Gateway
↓
DNS
↓
Remote host
↓
Port
↓
Firewall
↓
TCP
↓
TLS
↓
HTTP
↓
Application
↓
Backend / Database
```

A troubleshooting lényegében kizárásos nyomozás.

---

# 2. Ne találgassunk

Rossz megközelítés:

```text
"Biztos a DNS."

"Biztos a firewall."

"Biztos az Nginx."
```

Jobb megközelítés:

```text
Mi működik biztosan?
↓
Mi a következő réteg?
↓
Ott működik?
↓
Ha nem, ott keresünk tovább.
```

---

# 3. Alap troubleshooting sorrend

Egy jó kiindulási sorrend:

```text
1. Interface
2. IP address
3. Subnet
4. Route
5. Gateway
6. DNS
7. Remote host
8. Port / socket
9. Firewall
10. TCP
11. TLS
12. HTTP
13. Application
14. Backend / Database
15. Logs
```

Nem mindig kell minden lépést végigcsinálni.

A hibaüzenetek alapján gyakran több lépést is átugorhatunk.

---

# 4. 1. Van működő network interface?

Parancs:

```bash
ip link
```

Példa:

```text
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
```

Jó jel:

```text
UP
LOWER_UP
```

Ha:

```text
DOWN
```

akkor már nagyon korán megtalálhattuk a problémát.

---

# 5. Mit tegyünk, ha az interface DOWN?

Vizsgáljuk meg:

```bash
ip link
```

Kérdések:

```text
létezik az interface?
engedélyezve van?
van fizikai / virtuális link?
```

Ilyenkor még nincs értelme HTTP hibát keresni.

Mental model:

```text
interface DOWN
↓
előbb ezt javítsd
↓
utána IP / route / DNS
```

---

# 6. 2. Van IP address?

Parancs:

```bash
ip addr
```

Példa:

```text
inet 192.168.1.10/24
```

Ez azt jelenti:

```text
IP:
192.168.1.10

prefix:
/24
```

Ha nincs megfelelő:

```text
inet
```

sor, lehet:

```text
nincs IPv4 address
DHCP probléma
hibás konfiguráció
rossz interface
```

---

# 7. IP nélkül ne menjünk tovább HTTP felé

Ha nincs IP:

```text
no IP
↓
no normal IP communication
```

Ilyenkor nem elsőként ezt vizsgáljuk:

```text
TLS
HTTP
application
```

hanem az alap hálózati konfigurációt.

---

# 8. 3. Van route?

Parancs:

```bash
ip route
```

Példa:

```text
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0
```

Fontos:

```text
default via 192.168.1.1
```

Ez a default route.

---

# 9. Mit jelent, ha nincs default route?

Ha csak a helyi subnet ismert:

```text
192.168.1.0/24 dev eth0
```

de nincs:

```text
default via ...
```

akkor a gép a saját subnetjén kommunikálhat, de nem biztos, hogy tudja:

> merre küldje a távoli hálózatoknak szánt packeteket?

---

# 10. Egy konkrét route vizsgálata

Parancs:

```bash
ip route get 8.8.8.8
```

Példa:

```text
8.8.8.8 via 192.168.1.1 dev eth0 src 192.168.1.10
```

Ebből tudjuk:

```text
destination:
8.8.8.8

gateway:
192.168.1.1

interface:
eth0

source:
192.168.1.10
```

---

# 11. 4. Elérhető a gateway?

Tegyük fel:

```text
default gateway:
192.168.1.1
```

Teszt:

```bash
ping 192.168.1.1
```

Ha a gateway sem érhető el:

```text
host
↓
X
gateway
```

akkor helyi hálózati problémát keresünk.

Lehetséges területek:

```text
interface
IP
subnet
Wi-Fi / Ethernet
ARP / neighbor
local firewall
virtual network
```

---

# 12. 5. Elérhető egy külső IP?

Teszt:

```bash
ping 8.8.8.8
```

Ha működik:

```text
local interface
→ működik

IP
→ van

route
→ valószínűleg működik

gateway
→ valószínűleg működik

internet IP connectivity
→ van
```

Ez már sok mindent kizár.

---

# 13. Ping nem bizonyít mindent

Fontos:

```text
ping works
```

nem jelenti automatikusan:

```text
HTTPS works
```

A ping:

```text
ICMP
```

protokollt használ.

A weboldal például:

```text
TCP 443
```

porton működhet.

Ezért lehet:

```text
ping OK
HTTPS FAIL
```

---

# 14. Ping fail sem bizonyít mindent

Egy szerver blokkolhatja az ICMP forgalmat.

Ezért:

```text
ping FAIL
```

nem feltétlenül jelenti:

```text
server DOWN
```

Lehetséges:

```text
ICMP blocked
TCP 443 still works
```

---

# 15. 6. Működik a DNS?

Teszt:

```bash
dig example.com
```

vagy:

```bash
nslookup example.com
```

Egyszerű:

```bash
dig +short example.com
```

Példa:

```text
93.184.216.34
```

---

# 16. Gyakorlati helyzet: IP működik, domain nem

Ez működik:

```bash
ping 8.8.8.8
```

Ez nem:

```bash
dig example.com
```

Következtetés:

```text
IP connectivity
→ működik

DNS name resolution
→ problémás
```

Valószínűbb területek:

```text
DNS resolver
DNS server
DNS configuration
DNS network access
```

Nem elsőként routing hibát keresünk.

---

# 17. Tipikus DNS hibaüzenet

Például:

```text
Could not resolve host: example.com
```

Mental model:

```text
domain
↓
DNS
X
```

A kliens még nem jutott el oda, hogy normálisan csatlakozzon a webserverhez.

---

# 18. DNS szerver közvetlen tesztje

Például:

```bash
dig @8.8.8.8 example.com
```

Ha ez működik, de a normál:

```bash
dig example.com
```

nem, akkor a rendszer által konfigurált DNS resolver környékén lehet probléma.

---

# 19. 7. Elérhető a távoli hálózat?

Használhatjuk:

```bash
traceroute example.com
```

A traceroute segíthet megmutatni:

```text
PC
↓
gateway
↓
ISP
↓
más routerek
↓
destination
```

---

# 20. Traceroute és TTL

A traceroute a TTL mechanizmust használja.

```text
TTL 1
↓
első router

TTL 2
↓
második router

TTL 3
↓
harmadik router
```

Így közelítőleg láthatjuk a hálózati hopokat.

---

# 21. Traceroute kimenet nem mindig teljes

Lehetnek ilyen sorok:

```text
* * *
```

Ez nem feltétlenül jelenti, hogy ott megszakadt az internet.

Bizonyos routerek:

```text
nem válaszolnak traceroute-ra
szűrik az ICMP-t
```

miközben továbbítják a tényleges forgalmat.

---

# 22. 8. Elérhető a port?

Egy host elérhetősége és egy szolgáltatás elérhetősége nem ugyanaz.

Példa:

```text
server IP reachable
```

de:

```text
server:443 unreachable
```

Ilyenkor vizsgálni kell:

```text
port
firewall
service
TCP
bind address
```

---

# 23. Saját szerveren: figyel valami a porton?

Parancs:

```bash
ss -lntp
```

Példa:

```text
LISTEN 0 511 127.0.0.1:3000
```

Ebből tudjuk:

```text
service fut
port = 3000
bind address = loopback
```

---

# 24. Gyakorlati példa: localhost működik, távolról nem

Ez működik:

```bash
curl http://127.0.0.1:3000
```

De másik gépről:

```text
server-ip:3000
```

nem.

`ss`:

```text
127.0.0.1:3000 LISTEN
```

Következtetés:

```text
application működik
↓
de csak loopbacken figyel
```

Ez:

```text
bind address probléma
```

Nem elsőként firewall hibát keresünk.

---

# 25. Ha `0.0.0.0:3000` LISTEN

Például:

```text
0.0.0.0:3000 LISTEN
```

Ez azt jelenti:

```text
application minden IPv4 interface-en figyel
```

Ha mégsem érhető el kívülről:

```text
firewall?
cloud firewall?
routing?
NAT?
Docker port mapping?
```

irányba megyünk tovább.

---

# 26. 9. Firewall vizsgálata

UFW:

```bash
sudo ufw status
```

Példa:

```text
22/tcp   ALLOW
80/tcp   ALLOW
443/tcp  ALLOW
```

Ha a szolgáltatás:

```text
3000/tcp
```

és nincs engedélyezve:

```text
firewall lehet a blokkoló réteg
```

---

# 27. Cloud firewall

Cloud szervernél külön vizsgálni kell:

```text
Cloud Firewall
↓
Linux Firewall
```

Lehet:

```text
UFW allows 443
```

de:

```text
Cloud Firewall blocks 443
```

Ekkor továbbra sem lesz elérhető.

---

# 28. 10. TCP kapcsolat

HTTP előtt TCP kapcsolatnak kell létrejönnie klasszikus HTTP/1.1 és HTTP/2 esetén.

Ezért:

```text
TCP fail
↓
HTTP request még nem történt meg
```

Ha a TCP kapcsolat nem jön létre, nem érdemes rögtön HTTP status code-okat keresni.

---

# 29. `Connection timed out`

Példa:

```text
Connection timed out
```

Ez általában arra utal, hogy:

```text
nem érkezett megfelelő válasz időben
```

Lehetséges okok:

```text
firewall DROP
cloud firewall
network filtering
routing probléma
remote host nem válaszol
NAT probléma
```

Mental model:

```text
kérés
↓
...
↓
?
↓
nincs válasz
```

---

# 30. `Connection refused`

Példa:

```text
Connection refused
```

Ez fontos különbség.

Általában azt jelenti, hogy:

```text
a célhostot elértük
↓
de a célport nem fogad kapcsolatot
```

Lehetséges:

```text
service nem fut
rossz port
service más addressre bindolt
aktív firewall REJECT
```

---

# 31. Timeout vs refused

Gyors mental model:

```text
TIMEOUT
→ nincs válasz
→ valahol elakad / DROP
```

```text
REFUSED
→ válasz érkezett
→ port/service nem fogad
```

Nem mindig lehet pusztán ebből 100%-osan megmondani a konkrét okot, de nagyon jó nyom.

---

# 32. Gyakorlati helyzet: DNS működik, timeout van

Ez működik:

```bash
dig example.com
```

De:

```bash
curl -v https://example.com
```

eredménye:

```text
Connection timed out
```

Mit tudunk?

```text
DNS
→ működik
```

De a kapcsolat:

```text
TCP / network path
→ nem épül fel megfelelően
```

Vizsgálandó:

```text
routing
firewall
cloud firewall
remote availability
port 443
```

---

# 33. Gyakorlati helyzet: DNS működik, refused van

```bash
dig example.com
```

működik.

`curl`:

```text
Connection refused
```

Következtetés:

```text
DNS OK
host valószínűleg elérhető
célport nem fogad
```

Szerveroldalon:

```bash
ss -lntp
```

Keresnénk:

```text
:443
```

---

# 34. 11. TLS troubleshooting

Ha már TLS hibát kapunk, nagyon sok hálózati lépés sikeres volt.

Példa:

```text
certificate verify failed
```

Ez azt jelenti, hogy jellemzően:

```text
DNS
→ működött

routing
→ működött

TCP
→ létrejött

TLS handshake
→ elkezdődött
```

A hiba a TLS / certificate ellenőrzésnél van.

---

# 35. TLS hibák lehetséges okai

Például:

```text
certificate expired
domain mismatch
untrusted CA
missing intermediate certificate
wrong certificate
system clock problem
```

---

# 36. Curl verbose TLS vizsgálathoz

```bash
curl -v https://example.com
```

A verbose kimenetben láthatunk:

```text
resolved IP
connection
TLS handshake
certificate
HTTP request
HTTP response
```

Ez az egyik legfontosabb DevOps troubleshooting parancs.

---

# 37. 12. HTTP troubleshooting

Ha HTTP status code-ot kapunk:

```text
HTTP 404
HTTP 500
HTTP 502
HTTP 503
```

akkor a kommunikáció már eljutott egy HTTP szolgáltatásig.

Ez nagyon fontos.

---

# 38. HTTP status code = jó nyom

Ha például:

```text
HTTP 500
```

akkor jellemzően:

```text
DNS OK
routing OK
TCP OK
TLS OK, ha HTTPS
HTTP OK
↓
application/server-side problem
```

Tehát ne kezdjük újra az `ip addr` paranccsal, ha nincs rá külön ok.

---

# 39. HTTP 404

```text
404 Not Found
```

Általában:

```text
webserver elérhető
HTTP működik
kért resource / route nincs
```

Lehetséges:

```text
rossz URL
rossz application route
rossz Nginx location
hiányzó file
```

---

# 40. HTTP 500

```text
500 Internal Server Error
```

Jellemzően:

```text
request eljutott az applicationig
↓
application hibázott
```

Vizsgálandó:

```text
application logs
exceptions
database errors
environment variables
permissions
dependencies
```

---

# 41. HTTP 502 Bad Gateway

Ez különösen fontos reverse proxy használatakor.

Tipikus út:

```text
Client
↓
Nginx
↓
Backend
```

Ha:

```text
502 Bad Gateway
```

érkezik az Nginxtől, akkor:

```text
client elérte az Nginxet
↓
Nginx nem kapott megfelelő választ a backendtől
```

---

# 42. Mit tudunk 502 esetén?

Ha a kliens HTTPS-en kapott 502-t:

```text
DNS
→ működött

routing
→ működött

TCP
→ működött

TLS
→ működött

HTTP
→ eljutott az Nginxig
```

Ezért nem a publikus DNS-t kezdjük elsőként vizsgálni.

---

# 43. 502 gyakori okai

```text
backend nem fut
rossz backend port
rossz upstream IP
rossz hostname
Docker service nem elérhető
internal DNS probléma
connection refused backend felé
backend idő előtt bontja a kapcsolatot
```

---

# 44. 502 gyakorlati ellenőrzése

Először:

```bash
ss -lntp
```

Fut-e például:

```text
127.0.0.1:3000
```

Majd:

```bash
curl http://127.0.0.1:3000
```

Ha ez sem működik:

```text
Nginx mögötti backend probléma
```

Ha működik:

```text
Nginx proxy konfiguráció
```

felé vizsgálódunk tovább.

---

# 45. HTTP 503 Service Unavailable

```text
503 Service Unavailable
```

Jelentése:

```text
a HTTP szolgáltatás elérhető
↓
de jelenleg nem tudja kiszolgálni a kérést
```

Lehetséges okok:

```text
backend unavailable
maintenance
overload
no healthy upstream
service temporarily unavailable
```

---

# 46. 500 vs 502 vs 503

Gyors mental model:

```text
500
→ application belső hiba
```

```text
502
→ proxy / gateway nem kap jó választ upstreamtől
```

```text
503
→ service jelenleg nem tud kiszolgálni
```

---

# 47. 13. Application troubleshooting

Ha tudjuk, hogy a request eljutott az applicationig:

```text
networking nagy része működik
```

Innentől:

```text
application logs
process state
environment variables
permissions
dependencies
database
external API
```

irányba megyünk tovább.

---

# 48. Process fut?

Például:

```bash
ps aux
```

vagy service esetén:

```bash
systemctl status myapp
```

Node / PM2 esetén később például:

```bash
pm2 status
```

---

# 49. Porton figyel?

```bash
ss -lntp
```

Például:

```text
node
↓
127.0.0.1:3000
```

---

# 50. Application log

Nagyon fontos:

```text
network symptom
↓
application log
```

Például:

```text
database connection failed
```

Ilyenkor már nem a kliens DNS-beállítását javítjuk.

---

# 51. 14. Backend / database troubleshooting

Tipikus architektúra:

```text
Client
↓
Nginx
↓
Node
↓
PostgreSQL
```

Lehet, hogy:

```text
Client → Nginx
```

működik.

```text
Nginx → Node
```

is működik.

De:

```text
Node → PostgreSQL
```

nem.

Ekkor a felhasználó akár:

```text
500
```

hibát kaphat.

---

# 52. Minden szakaszt külön lehet tesztelni

```text
Client → Nginx
```

teszt:

```bash
curl https://domain.hu
```

---

```text
Nginx → Node
```

teszt ugyanazon a szerveren:

```bash
curl http://127.0.0.1:3000
```

---

```text
Node → Database
```

vizsgálat:

```text
DB hostname
DB port
credentials
network
firewall
logs
```

---

# 53. "Divide and conquer"

Ez az egyik legjobb troubleshooting elv.

Ne az egész rendszert egyszerre nézd.

```text
A → B → C → D
```

Teszteld:

```text
A → B
```

utána:

```text
B → C
```

utána:

```text
C → D
```

Így megtalálható, melyik kapcsolat hibás.

---

# 54. Kliensoldali troubleshooting sorrend

Ha a saját gépedről nem érsz el egy weboldalt:

```text
1. ip link
2. ip addr
3. ip route
4. gateway
5. external IP
6. DNS
7. remote host/path
8. TCP port
9. TLS
10. HTTP
```

---

# 55. Szerveroldali troubleshooting sorrend

Ha a saját szervered szolgáltatása nem érhető el:

```text
1. fut a process?
2. figyel a porton?
3. milyen addressre bindolt?
4. lokálisan elérhető?
5. Linux firewall?
6. cloud firewall?
7. routing?
8. reverse proxy?
9. backend?
10. application logs?
```

---

# 56. Példa: Node app nem érhető el

Elvárt:

```text
server:3000
```

Először:

```bash
ss -lntp
```

Ha nincs:

```text
:3000 LISTEN
```

akkor:

```text
app nem fut
vagy más porton fut
```

---

# 57. Node figyel, de csak loopbacken

```text
127.0.0.1:3000 LISTEN
```

Lokálisan:

```bash
curl http://127.0.0.1:3000
```

működik.

Távolról nem.

Következtetés:

```text
bind address
```

---

# 58. Node minden interface-en figyel, de kívülről nem megy

```text
0.0.0.0:3000 LISTEN
```

Lokálisan működik.

Kívülről nem.

Vizsgálandó:

```text
UFW
cloud firewall
NAT
routing
security rules
Docker mapping
```

---

# 59. Nginx működik, Node nem

Kívülről:

```text
502 Bad Gateway
```

Lokálisan:

```bash
curl http://127.0.0.1:3000
```

eredménye:

```text
Connection refused
```

Nagyon valószínű:

```text
Node nem fut
vagy rossz porton fut
```

---

# 60. Nginx működik, backend is működik lokálisan

Kívül:

```text
502
```

De:

```bash
curl http://127.0.0.1:3000
```

működik.

Ekkor vizsgálandó:

```text
Nginx upstream config
proxy_pass
hostname
port
protocol
container network
```

---

# 61. Gyakorlati példa: domain nem oldódik fel

Hiba:

```text
Could not resolve host
```

Lépések:

```bash
dig domain.hu
```

Ha fail:

```bash
dig @8.8.8.8 domain.hu
```

Ha más DNS szerverrel működik:

```text
local resolver/configuration probléma lehet
```

---

# 62. Gyakorlati példa: timeout

Hiba:

```text
Connection timed out
```

Lépések:

```text
1. DNS feloldódik?
2. route van?
3. cél IP megfelelő?
4. port engedett?
5. local firewall?
6. cloud firewall?
7. service elérhető?
```

Szerveroldalon:

```bash
ss -lntp
sudo ufw status
```

---

# 63. Gyakorlati példa: refused

Hiba:

```text
Connection refused
```

Első kérdés:

```text
figyel valami azon a porton?
```

Szerveren:

```bash
ss -lntp
```

Ha nincs:

```text
indítsd el a szolgáltatást
vagy ellenőrizd a portot
```

---

# 64. Gyakorlati példa: TLS error

Hiba:

```text
certificate verify failed
```

Vizsgálandó:

```text
domain name
certificate domain
expiry
certificate chain
trusted CA
system date/time
```

Nem elsőként:

```text
default route
```

hibát keresünk.

---

# 65. Gyakorlati példa: 404

```text
HTTP 404
```

Tudjuk:

```text
network path
→ működik

HTTP server
→ válaszol
```

Vizsgálandó:

```text
URL path
Nginx location
application route
file/resource
```

---

# 66. Gyakorlati példa: 500

```text
HTTP 500
```

Vizsgálandó:

```text
application logs
database
environment
exception
permissions
external service
```

---

# 67. Gyakorlati példa: 502

```text
HTTP 502
```

Vizsgálandó:

```text
reverse proxy
upstream
backend process
backend port
internal network
container DNS
```

---

# 68. Gyakorlati példa: 503

```text
HTTP 503
```

Vizsgálandó:

```text
service availability
healthy backend
load
maintenance
application state
```

---

# 69. Hasznos parancsok együtt

## Interface

```bash
ip link
```

---

## IP

```bash
ip addr
```

---

## Routing

```bash
ip route
```

---

## Egy konkrét route

```bash
ip route get 8.8.8.8
```

---

## Neighbor

```bash
ip neigh
```

---

## Reachability

```bash
ping 8.8.8.8
```

---

## Network path

```bash
traceroute example.com
```

---

## DNS

```bash
dig example.com
```

```bash
dig +short example.com
```

---

## Listening sockets

```bash
ss -lntp
```

---

## Active TCP connections

```bash
ss -nt
```

---

## Firewall

```bash
sudo ufw status
```

---

## HTTP / HTTPS

```bash
curl -v https://example.com
```

---

## HTTP headers

```bash
curl -I https://example.com
```

---

# 70. Hibajelzés → első gyanú

## `Could not resolve host`

```text
DNS
```

---

## `Network is unreachable`

```text
routing / interface / IP
```

---

## `No route to host`

```text
routing / network path / filtering
```

---

## `Connection timed out`

```text
firewall / DROP / routing / remote availability
```

---

## `Connection refused`

```text
port / service / bind / active reject
```

---

## `certificate verify failed`

```text
TLS / certificate
```

---

## `404`

```text
HTTP resource / application route
```

---

## `500`

```text
application
```

---

## `502`

```text
reverse proxy → backend
```

---

## `503`

```text
service unavailable / backend availability
```

---

# 71. Gyors döntési fa

```text
Nem működik a weboldal
        ↓
Van IP?
├─ NEM → interface / DHCP / config
└─ IGEN
     ↓
Van route?
├─ NEM → routing
└─ IGEN
     ↓
DNS működik?
├─ NEM → DNS
└─ IGEN
     ↓
TCP kapcsolat létrejön?
├─ NEM
│   ├─ timeout → firewall / path / DROP
│   └─ refused → service / port
└─ IGEN
     ↓
TLS működik?
├─ NEM → certificate / TLS
└─ IGEN
     ↓
HTTP response?
├─ 404 → route/resource
├─ 500 → application
├─ 502 → backend/upstream
└─ 503 → unavailable service
```

---

# 72. A legfontosabb troubleshooting elv

Mindig ezt kérdezd:

> **Mi az utolsó dolog, amiről biztosan tudom, hogy működik?**

Utána:

> **Mi a következő komponens a láncban?**

Például:

```text
DNS works
↓
next question:
TCP connection works?
```

Vagy:

```text
Nginx returns 502
↓
Nginx works
↓
next question:
backend works?
```

---

# 73. Ne menj vissza feleslegesen rétegeket

Ha már:

```text
HTTP 500
```

választ kapsz, akkor nem valószínű, hogy az elsődleges probléma:

```text
nincs IP address
```

Hiszen IP nélkül nem kaptál volna HTTP 500-at.

Ez a hibakeresés egyik legfontosabb logikai módszere:

```text
a későbbi réteg működése
↓
bizonyítja több korábbi réteg működését
```

---

# 74. Troubleshooting és logok

A hálózati eszközök mellett mindig fontosak a logok.

Például:

```text
Nginx logs
application logs
system logs
firewall logs
Docker logs
database logs
```

A hibaüzenet gyakran pontosan megmondja:

```text
connection refused
permission denied
database unreachable
timeout
certificate error
```

---

# 75. Packet capture

Ha a hagyományos eszközök már nem elegendők:

```bash
tcpdump
```

vagy:

```text
Wireshark
```

segítségével megnézhetjük:

```text
elindul-e a packet?
érkezik-e válasz?
van-e SYN?
van-e SYN-ACK?
DNS query kimegy?
DNS response visszajön?
TCP reset érkezik?
```

Ez lesz a későbbi Wireshark labor egyik fő célja.

---

# 76. Példa Wireshark gondolkodásra

Ha látunk:

```text
SYN
↓
SYN
↓
SYN
↓
```

de nincs:

```text
SYN-ACK
```

akkor a kliens próbál csatlakozni, de nem kap választ.

Ez jól illik például:

```text
timeout
```

helyzethez.

---

# 77. Ha RST érkezik

Például:

```text
SYN
↓
RST
```

az arra utalhat, hogy a cél aktívan visszautasítja a TCP kapcsolatot.

Ez kapcsolatban lehet:

```text
Connection refused
```

hibával.

---

# 78. Troubleshooting mental model DevOpsként

Ne technológiákban gondolkodjunk:

```text
Nginx?
Docker?
Linux?
DNS?
```

hanem kommunikációs útban:

```text
Client
↓
DNS
↓
Network
↓
Firewall
↓
Host
↓
Socket
↓
Proxy
↓
Application
↓
Database
```

Majd kérdezzük:

```text
Hol áll meg?
```

---

# 79. Rövid szakmai angol

> The network interface is up.

> The host has an IP address.

> The server has a default route.

> DNS resolution works.

> The connection timed out.

> The connection was refused.

> The server is listening on port 443.

> The application is bound to localhost.

> The firewall blocks the connection.

> The TLS certificate is invalid.

> The request reached the web server.

> The backend service is unavailable.

> Nginx cannot connect to the upstream server.

> Check the application logs.

---

# 80. Gyors fogalomtár

## Troubleshooting

```text
szisztematikus hibakeresés
```

---

## Reachability

```text
elérhető-e hálózatilag a cél?
```

---

## Name resolution

```text
név → IP feloldás
```

---

## Timeout

```text
nem érkezett válasz időben
```

---

## Connection refused

```text
cél elérhető,
de a kapcsolatot nem fogadják
```

---

## Listening

```text
egy process kapcsolatot vár egy porton
```

---

## Bind address

```text
melyik helyi IP-n figyel a process?
```

---

## Loopback

```text
127.0.0.1
→ saját gép
```

---

## Route

```text
merre menjen a packet?
```

---

## Default gateway

```text
alapértelmezett következő router
```

---

## Firewall

```text
engedjük-e a trafficet?
```

---

## TCP handshake

```text
SYN
SYN-ACK
ACK
```

---

## TLS

```text
titkosítás
hitelesítés
integritás
```

---

## HTTP status code

```text
az HTTP request eredményéről ad információt
```

---

## Upstream

```text
a reverse proxy mögötti backend szolgáltatás
```

---

## Reverse proxy

```text
fogadja a kliens kérését,
majd továbbítja a backendnek
```

---

## 502

```text
proxy elérhető,
backend/upstream kapcsolat problémás lehet
```

---

## 500

```text
application/server-side hiba
```

---

## 404

```text
kért resource nincs
```

---

# 81. Gyors hibakeresési emlékeztető

```text
ip link
→ van interface?
```

```text
ip addr
→ van IP?
```

```text
ip route
→ van útvonal?
```

```text
ping
→ van alap IP reachability?
```

```text
dig
→ működik DNS?
```

```text
ss
→ figyel a service?
```

```text
ufw
→ engedi a firewall?
```

```text
curl -v
→ TCP / TLS / HTTP hol tart?
```

```text
logs
→ mit mond az application?
```

```text
tcpdump / Wireshark
→ mi történik ténylegesen a packetekkel?
```

---

# 82. Végső mental model

Ha valami nem működik:

```text
Interface
↓
IP
↓
Route
↓
DNS
↓
TCP
↓
TLS
↓
HTTP
↓
Proxy
↓
Application
↓
Backend / Database
```

Mindig keresd meg:

```text
az utolsó biztosan működő pontot
```

majd vizsgáld:

```text
a következő lépést
```

Ez a network troubleshooting alapja.