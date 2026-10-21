# Networking Cheatsheet

## 1. Alapfogalmak

### Network

```text
network
→ egymással kommunikáló eszközök hálózata
```

### Host

```text
host
→ hálózatra csatlakozó eszköz
```

### Client

```text
client
→ kérést kezdeményez
```

### Server

```text
server
→ szolgáltatást biztosít / válaszol
```

### LAN

```text
LAN
→ Local Area Network
→ helyi hálózat
```

### WAN

```text
WAN
→ Wide Area Network
→ nagyobb kiterjedésű hálózat
```

---

# 2. Network interface

```text
network interface
→ amin keresztül a gép hálózaton kommunikál
```

Lehet:

```text
Ethernet
Wi-Fi
loopback
Docker interface
bridge
VPN
WSL virtual interface
```

Linux:

```bash
ip link
```

---

# 3. MAC address

```text
MAC address
→ helyi hálózati azonosító
```

Főleg:

```text
Ethernet / local network
```

szinten használjuk.

Mental model:

```text
MAC
→ következő helyi állomás
```

---

# 4. IP address

```text
IP address
→ melyik host?
```

IPv4:

```text
192.168.1.10
```

IPv6:

```text
2001:db8::1
```

Linux:

```bash
ip addr
```

---

# 5. IPv4

```text
IPv4
→ 32 bit
→ 4 octet
```

Példa:

```text
192.168.1.10
```

Egy octet:

```text
0–255
```

---

# 6. IPv6

```text
IPv6
→ 128 bit
→ hexadecimal
```

Példa:

```text
2001:db8::1
```

Loopback:

```text
::1
```

Link-local tartomány:

```text
fe80::/10
```

Gyakori subnet:

```text
/64
```

---

# 7. Loopback

IPv4:

```text
127.0.0.1
```

IPv6:

```text
::1
```

Jelentése:

```text
saját gép
```

Nem ugyanaz, mint a private network.

---

# 8. Private IPv4 tartományok

```text
10.0.0.0/8
```

```text
172.16.0.0/12
```

```text
192.168.0.0/16
```

Mental model:

```text
private IP
→ belső hálózat
```

---

# 9. Subnet

```text
subnet
→ egy IP-hálózat egy logikai része
```

Példa:

```text
192.168.1.10/24
```

Itt:

```text
192.168.1.10
→ host IP
```

```text
/24
→ network prefix
```

---

# 10. CIDR

```text
/24
/26
/27
```

Megmondja:

```text
hány bit tartozik a network részhez
```

Példa:

```text
/24
→ 255.255.255.0
```

```text
/26
→ 255.255.255.192
```

---

# 11. Subnet mask

```text
1 bit
→ network rész

0 bit
→ host rész
```

Példa:

```text
255.255.255.0
→ /24
```

---

# 12. Block size

```text
block size
= 2^(host bitek száma)
```

Példa:

```text
/26
→ 6 host bit
→ 64 address/block
```

Blokkok:

```text
0–63
64–127
128–191
192–255
```

---

# 13. Network address

```text
subnet első címe
```

Példa:

```text
192.168.1.64/26
```

Network address:

```text
192.168.1.64
```

---

# 14. Broadcast address

```text
subnet utolsó IPv4 címe
```

Példa:

```text
192.168.1.64/26
```

Broadcast:

```text
192.168.1.127
```

---

# 15. Usable host range

A network és broadcast közötti címek.

Példa:

```text
192.168.1.65
...
192.168.1.126
```

---

# 16. Same subnet

Ha két IP ugyanabba a subnetbe tartozik:

```text
host
↓
közvetlen helyi kommunikáció
↓
ARP
↓
destination MAC
```

---

# 17. Different subnet

Ha a cél más subnetben van:

```text
host
↓
default gateway
↓
router
```

A cél IP továbbra is a távoli host IP-je.

A helyi destination MAC viszont:

```text
gateway MAC
```

---

# 18. ARP

```text
ARP
→ IPv4 cím → MAC cím
```

Csak helyi linken.

Linux neighbor table:

```bash
ip neigh
```

Mental model:

```text
routing
→ kinek adjam tovább?

ARP
→ mi annak a MAC-címe?
```

---

# 19. Router

```text
router
→ hálózatok között továbbít packeteket
```

Fő kérdése:

```text
destination IP
→ merre tovább?
```

---

# 20. Default gateway

```text
default gateway
→ alapértelmezett router más hálózatok felé
```

Példa:

```text
192.168.1.1
```

---

# 21. Routing

```text
routing
→ merre menjen a packet?
```

A router routing table-t használ.

---

# 22. Routing table

Példa Linuxon:

```bash
ip route
```

Kimenet:

```text
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0
```

---

# 23. Default route

```text
0.0.0.0/0
```

jelentése:

```text
minden olyan destination,
amire nincs specifikusabb route
```

Példa:

```text
default via 192.168.1.1
```

---

# 24. Next hop

```text
next hop
→ következő router
```

A routernek nem kell ismernie a teljes útvonalat.

Csak:

```text
merre küldje tovább?
```

---

# 25. Longest prefix match

Routingnál:

```text
a legspecifikusabb matching route nyer
```

Példa:

```text
10.0.0.0/8
```

specifikusabb, mint:

```text
0.0.0.0/0
```

---

# 26. Hop

```text
hop
→ egy routeren való áthaladás
```

---

# 27. TTL

IP TTL:

```text
Time To Live
```

Routerenként csökken.

```text
TTL = TTL - 1
```

Cél:

```text
routing loop megakadályozása
```

---

# 28. NAT

```text
NAT
→ Network Address Translation
```

Mental model:

```text
IP-cím átírás
```

Példa:

```text
192.168.1.10
↓
public IP
```

---

# 29. PAT

```text
PAT
→ Port Address Translation
```

Példa:

```text
192.168.1.10:52341
↓
84.x.x.x:41052
```

Így több belső kapcsolat használhat egy public IP-t.

---

# 30. NAT vs routing vs firewall

```text
Routing
→ merre?
```

```text
NAT
→ mit írjunk át?
```

```text
Firewall
→ szabad-e?
```

---

# 31. DNS

```text
DNS
→ Domain Name System
```

Feladata:

```text
domain name
→ IP address
```

Példa:

```text
example.com
↓
DNS
↓
93.x.x.x
```

---

# 32. DNS resolver

```text
resolver
→ DNS lekérdezéseket intéző szerver/szolgáltatás
```

A kliens tipikusan resolverhez fordul.

---

# 33. DNS hierarchy

```text
root
↓
TLD
↓
authoritative DNS
```

Példa:

```text
.
↓
.com
↓
example.com authoritative server
```

---

# 34. DNS cache

```text
cache hit
→ válasz már megvan
```

```text
cache miss
→ DNS lekérdezés kell
```

---

# 35. DNS TTL

```text
DNS TTL
→ mennyi ideig cache-elhető a record
```

Nem ugyanaz, mint az IP TTL.

```text
IP TTL
→ hopok száma
```

```text
DNS TTL
→ idő
```

---

# 36. DNS recordok

```text
A
→ IPv4
```

```text
AAAA
→ IPv6
```

```text
CNAME
→ alias
```

```text
MX
→ mail server
```

```text
NS
→ name server
```

```text
TXT
→ text metadata
```

---

# 37. DNS parancsok

```bash
dig example.com
```

```bash
dig +short example.com
```

```bash
nslookup example.com
```

Más resolver:

```bash
dig @8.8.8.8 example.com
```

---

# 38. Port

```text
port
→ melyik szolgáltatás?
```

Tartomány:

```text
0–65535
```

---

# 39. Gyakori portok

```text
22
→ SSH
```

```text
53
→ DNS
```

```text
80
→ HTTP
```

```text
443
→ HTTPS
```

```text
3306
→ MySQL
```

```text
5432
→ PostgreSQL
```

---

# 40. Ephemeral port

```text
ephemeral port
→ ideiglenes kliensoldali port
```

Példa:

```text
client:52341
→ server:443
```

---

# 41. Socket

```text
socket
→ OS által kezelt kommunikációs végpont
```

Egyszerű analógia:

```text
program
→ iroda

socket
→ kommunikációs szoba

port
→ ajtószám
```

---

# 42. Egy TCP kapcsolat

Példa:

```text
192.168.1.10:52341
↔
203.0.113.20:443
```

Kapcsolat azonosításában szerepel:

```text
source IP
source port
destination IP
destination port
protocol
```

---

# 43. Listening socket

```text
LISTEN
→ program új kapcsolatot vár
```

Példa:

```text
0.0.0.0:443
```

---

# 44. Bind address

```text
bind address
→ melyik helyi IP-n figyel a program?
```

---

# 45. `127.0.0.1`

```text
loopback
→ csak saját gép
```

Példa:

```text
127.0.0.1:3000
```

---

# 46. `0.0.0.0`

```text
minden IPv4 interface
```

Példa:

```text
0.0.0.0:3000
```

Nem jelenti automatikusan, hogy internet felől is elérhető.

Firewall még blokkolhatja.

---

# 47. TCP

```text
TCP
→ Transmission Control Protocol
```

Tulajdonságok:

```text
connection-oriented
reliable
ordered
retransmission
byte stream
```

---

# 48. TCP 3-way handshake

```text
Client                Server

SYN ----------------->

    <---------------- SYN-ACK

ACK ----------------->
```

Ezután:

```text
TCP connection established
```

---

# 49. SYN

```text
SYN
→ Synchronize
```

Kapcsolat kezdeményezése.

---

# 50. ACK

```text
ACK
→ Acknowledgment
```

Visszaigazolás.

---

# 51. FIN

```text
FIN
→ normál TCP kapcsolatlezárás
```

---

# 52. RST

```text
RST
→ TCP connection reset
```

---

# 53. TCP sequence number

```text
sequence number
→ adatfolyam sorrendjének követése
```

---

# 54. TCP retransmission

Ha adat elveszik:

```text
TCP
→ újraküldheti
```

---

# 55. Flow control

```text
flow control
→ ne terheljük túl a fogadó oldalt
```

---

# 56. Congestion control

```text
congestion control
→ hálózati torlódás kezelése
```

---

# 57. UDP

```text
UDP
→ User Datagram Protocol
```

Tulajdonságok:

```text
connectionless
datagram oriented
no built-in delivery guarantee
no built-in ordering
no built-in retransmission
```

---

# 58. TCP vs UDP

```text
TCP
→ kapcsolat
→ megbízhatósági mechanizmusok
→ sorrend
→ retransmission
```

```text
UDP
→ nincs TCP-style kapcsolat
→ nincs beépített garancia
→ datagram
```

---

# 59. DNS és UDP/TCP

DNS gyakran:

```text
UDP 53
```

de használhat:

```text
TCP 53
```

is.

---

# 60. QUIC

```text
QUIC
→ UDP fölött működő modern transport
```

HTTP/3:

```text
HTTP/3
↓
QUIC
↓
UDP
```

---

# 61. HTTP

```text
HTTP
→ HyperText Transfer Protocol
```

Application layer protocol.

Mental model:

```text
client
↓ request
server
↓ response
```

---

# 62. HTTP request

Fő részei:

```text
method
path
headers
body
```

Példa:

```http
GET /users HTTP/1.1
Host: example.com
```

---

# 63. HTTP methodok

```text
GET
→ lekérés
```

```text
POST
→ adat küldése / létrehozás
```

```text
PUT
→ teljes módosítás
```

```text
PATCH
→ részleges módosítás
```

```text
DELETE
→ törlés
```

---

# 64. HTTP header

```text
header
→ információ az üzenetről
```

Példa:

```text
Content-Type: application/json
```

---

# 65. HTTP body

```text
body
→ tényleges tartalom
```

Példa:

```json
{"name":"Tamas"}
```

---

# 66. HTTP response

Példa:

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

---

# 67. Status code csoportok

```text
1xx
→ információ
```

```text
2xx
→ siker
```

```text
3xx
→ redirect
```

```text
4xx
→ client-side probléma
```

```text
5xx
→ server-side probléma
```

---

# 68. Gyakori status code-ok

```text
200
→ OK
```

```text
201
→ Created
```

```text
301 / 302
→ Redirect
```

```text
400
→ Bad Request
```

```text
401
→ authentication hiányzik / hibás
```

```text
403
→ Forbidden
```

```text
404
→ Not Found
```

```text
500
→ Internal Server Error
```

```text
502
→ Bad Gateway
```

```text
503
→ Service Unavailable
```

---

# 69. URL

Példa:

```text
https://api.example.com:443/users/123?active=true
```

Felépítése:

```text
https
→ scheme
```

```text
api.example.com
→ hostname
```

```text
443
→ port
```

```text
/users/123
→ path
```

```text
active=true
→ query
```

---

# 70. HTTPS

```text
HTTPS
=
HTTP over TLS
```

Klasszikus modell:

```text
HTTP
↓
TLS
↓
TCP
↓
IP
```

---

# 71. TLS

```text
TLS
→ Transport Layer Security
```

Fő célok:

```text
encryption
authentication
integrity
```

---

# 72. TLS céljai

```text
encryption
→ ne lehessen elolvasni
```

```text
authentication
→ tudjuk, kivel beszélünk
```

```text
integrity
→ ne lehessen észrevétlenül módosítani
```

---

# 73. Certificate

```text
TLS certificate
→ szerver identitásának ellenőrzése
```

Tartalmazhat:

```text
domain
issuer
validity
public key
```

---

# 74. CA

```text
CA
→ Certificate Authority
```

Megbízható hitelesítésszolgáltató.

---

# 75. Public key

```text
public key
→ megosztható
```

---

# 76. Private key

```text
private key
→ titkos
→ csak a tulajdonos birtokolja
```

---

# 77. Session key

A tényleges TLS adatforgalomnál gyors symmetric encryption használható.

Mental model:

```text
public/private crypto
↓
key agreement
↓
session key
↓
encrypted traffic
```

---

# 78. Teljes web request flow

```text
URL
↓
DNS
↓
destination IP
↓
same subnet?
↓
default gateway
↓
ARP
↓
TCP socket
↓
TCP handshake
↓
routing
↓
NAT
↓
TLS handshake
↓
HTTP request
↓
server
↓
application
↓
database
↓
HTTP response
↓
browser
```

---

# 79. Encapsulation

Küldéskor:

```text
HTTP data
↓
TLS
↓
TCP segment
↓
IP packet
↓
Ethernet frame
↓
bits
```

---

# 80. Decapsulation

Fogadáskor:

```text
bits
↓
Ethernet frame
↓
IP packet
↓
TCP segment
↓
TLS
↓
HTTP
```

---

# 81. Frame / packet / segment

```text
Ethernet
→ frame
```

```text
IP
→ packet
```

```text
TCP
→ segment
```

---

# 82. Többféle címzés egyszerre

```text
MAC
→ következő helyi állomás
```

```text
IP
→ végső host
```

```text
Port
→ végső szolgáltatás
```

---

# 83. Firewall

```text
firewall
→ hálózati forgalom szűrése
```

Fő kérdés:

```text
engedjük vagy tiltjuk?
```

---

# 84. Inbound

```text
network
↓
saját gép
```

---

# 85. Outbound

```text
saját gép
↓
network
```

---

# 86. ALLOW

```text
ALLOW / ACCEPT
→ engedd át
```

---

# 87. DROP

```text
DROP
→ dobd el
→ ne válaszolj
```

---

# 88. REJECT

```text
REJECT
→ tiltsd
→ jelezd vissza
```

---

# 89. Stateful firewall

```text
stateful
→ követi a kapcsolatok állapotát
```

Például:

```text
NEW
ESTABLISHED
RELATED
```

---

# 90. UFW

```text
UFW
→ Uncomplicated Firewall
```

Egyszerű Linux firewall management.

---

# 91. UFW alapparancsok

Státusz:

```bash
sudo ufw status
```

SSH:

```bash
sudo ufw allow 22/tcp
```

HTTP:

```bash
sudo ufw allow 80/tcp
```

HTTPS:

```bash
sudo ufw allow 443/tcp
```

Bekapcsolás:

```bash
sudo ufw enable
```

---

# 92. Default firewall policy

Gyakori modell:

```text
deny incoming
allow outgoing
```

---

# 93. Firewall vs bind

```text
bind address
→ hol figyel?
```

```text
firewall
→ odaengedjük-e a trafficet?
```

---

# 94. Firewall vs NAT

```text
NAT
→ átír
```

```text
firewall
→ szűr
```

---

# 95. Firewall vs routing

```text
routing
→ merre?
```

```text
firewall
→ szabad-e?
```

---

# 96. Reverse proxy

```text
reverse proxy
→ fogadja a kliens requestet
→ továbbítja a backendnek
```

Példa:

```text
Internet
↓
Nginx :443
↓
Node :3000
```

---

# 97. Upstream

```text
upstream
→ reverse proxy mögötti backend
```

---

# 98. Tipikus biztonságos szervermodell

```text
Internet
↓
:443
↓
Nginx
↓
127.0.0.1:3000
↓
Node
↓
internal network
↓
PostgreSQL
```

Public:

```text
443
```

Nem public:

```text
3000
5432
```

---

# 99. Linux networking parancsok

## Interface

```bash
ip link
```

---

## IP address

```bash
ip addr
```

Röviden:

```bash
ip a
```

---

## Routing

```bash
ip route
```

---

## Konkrét route

```bash
ip route get 8.8.8.8
```

---

## Neighbor / ARP

```bash
ip neigh
```

---

## Listening TCP sockets

```bash
ss -lntp
```

---

## Active TCP connections

```bash
ss -nt
```

---

## Basic reachability

```bash
ping 8.8.8.8
```

---

## Route path

```bash
traceroute example.com
```

---

## DNS

```bash
dig example.com
```

---

## HTTP/HTTPS

```bash
curl https://example.com
```

---

## Verbose HTTP/HTTPS

```bash
curl -v https://example.com
```

---

## Headers

```bash
curl -I https://example.com
```

---

## Packet capture

```bash
tcpdump
```

---

# 100. `ip link`

Megmutatja:

```text
interface
state
MAC
MTU
```

---

# 101. `ip addr`

Megmutatja:

```text
IPv4
IPv6
CIDR
broadcast
```

---

# 102. `ip route`

Megmutatja:

```text
routing table
default gateway
interfaces
```

---

# 103. `ip neigh`

Megmutatja:

```text
local IP
↔
MAC
```

---

# 104. `ss`

Megmutatja:

```text
ports
sockets
TCP connections
LISTEN state
```

---

# 105. `ping`

Megmutathatja:

```text
alap IP reachability
latency
packet loss
```

De:

```text
ping fail
≠
biztosan server down
```

---

# 106. `traceroute`

```text
network hops
```

TTL segítségével.

---

# 107. `dig`

```text
DNS troubleshooting
```

---

# 108. `curl`

```text
HTTP/HTTPS troubleshooting
```

Különösen:

```bash
curl -v
```

---

# 109. `tcpdump`

```text
CLI packet capture
```

---

# 110. Wireshark

```text
grafikus packet analyzer
```

Megfigyelhető például:

```text
Ethernet
IP
TCP
UDP
DNS
TLS
```

---

# 111. Network troubleshooting sorrend

```text
Interface
↓
IP
↓
Route
↓
Gateway
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
Backend / DB
```

---

# 112. Legfontosabb troubleshooting kérdés

> Mi az utolsó dolog, amiről biztosan tudom, hogy működik?

Utána:

> Mi a következő pont a láncban?

---

# 113. DNS hiba

Példa:

```text
Could not resolve host
```

Első gyanú:

```text
DNS
```

Teszt:

```bash
dig example.com
```

---

# 114. Network unreachable

```text
Network is unreachable
```

Első gyanú:

```text
interface
IP
route
```

---

# 115. Timeout

```text
Connection timed out
```

Mental model:

```text
nem érkezett válasz
```

Gyanú:

```text
firewall DROP
routing
network path
cloud firewall
remote host
```

---

# 116. Connection refused

```text
Connection refused
```

Mental model:

```text
cél elérhető
de a port nem fogad
```

Gyanú:

```text
service nem fut
rossz port
rossz bind
active reject
```

---

# 117. TLS hiba

Példa:

```text
certificate verify failed
```

Ami valószínűleg már működik:

```text
DNS
routing
TCP
```

Gyanú:

```text
certificate
TLS config
domain mismatch
expiry
CA
```

---

# 118. HTTP 404

```text
server elérhető
HTTP működik
resource nincs
```

Gyanú:

```text
URL
route
Nginx location
file
```

---

# 119. HTTP 500

```text
request eljutott az applicationig
```

Gyanú:

```text
application bug
database
environment
permissions
dependency
```

---

# 120. HTTP 502

```text
reverse proxy elérhető
backend/upstream problémás
```

Gyanú:

```text
backend nem fut
rossz port
rossz upstream
container network
internal DNS
```

Teszt:

```bash
curl http://127.0.0.1:3000
```

---

# 121. HTTP 503

```text
service jelenleg nem tud kiszolgálni
```

Gyanú:

```text
backend unavailable
maintenance
overload
no healthy backend
```

---

# 122. Timeout vs refused

```text
TIMEOUT
→ nincs válasz
```

```text
REFUSED
→ válasz van, kapcsolatot nem fogadják
```

---

# 123. Listening vs reachable

```text
LISTEN
→ process vár kapcsolatot
```

```text
REACHABLE
→ hálózatról ténylegesen elérhető
```

Nem ugyanaz.

---

# 124. Loopback vs private network

```text
loopback
→ saját gép
```

```text
private network
→ belső hálózat több hosttal
```

---

# 125. Bind address vs port

```text
bind address
→ hol figyel?
```

```text
port
→ melyik szolgáltatás?
```

---

# 126. DNS vs routing

```text
DNS
→ mi az IP?
```

```text
routing
→ merre menjünk az IP felé?
```

---

# 127. ARP vs routing

```text
routing
→ melyik next hop?
```

```text
ARP
→ mi a next hop MAC-címe?
```

---

# 128. TCP vs HTTP

```text
TCP
→ megbízható transport
```

```text
HTTP
→ application request/response szabályok
```

---

# 129. TLS vs HTTP

```text
HTTP
→ mit kérünk?
```

```text
TLS
→ hogyan legyen biztonságos?
```

---

# 130. Teljes mental model

```text
Domain
↓
DNS
↓
IP
↓
Subnet
↓
Route
↓
Gateway
↓
ARP / MAC
↓
TCP / UDP
↓
Port
↓
Socket
↓
TLS
↓
HTTP
↓
Application
↓
Database
```

---

# 131. Egy internetes request fejben

```text
example.com
↓
DNS
↓
93.x.x.x
↓
más subnet
↓
default gateway
↓
gateway MAC ARP-pal
↓
TCP client ephemeral port → server:443
↓
TCP handshake
↓
routing + NAT
↓
TLS handshake
↓
HTTP request
↓
server
↓
HTTP response
```

---

# 132. Legfontosabb fogalmak egy sorban

```text
IP
→ melyik gép?
```

```text
MAC
→ melyik következő helyi eszköz?
```

```text
Port
→ melyik szolgáltatás?
```

```text
Socket
→ kommunikációs végpont
```

```text
DNS
→ domain → IP
```

```text
Route
→ merre?
```

```text
Gateway
→ következő router más hálózatok felé
```

```text
ARP
→ local IP → MAC
```

```text
NAT
→ cím/port átírás
```

```text
Firewall
→ enged vagy tilt
```

```text
TCP
→ megbízható kapcsolat
```

```text
UDP
→ kapcsolat nélküli datagram transport
```

```text
TLS
→ titkosítás + hitelesítés + integritás
```

```text
HTTP
→ request / response
```

```text
Loopback
→ saját gép
```

```text
Bind address
→ hol figyel az application?
```

```text
Listening
→ process kapcsolatot vár
```

```text
Reverse proxy
→ requestet továbbít backendnek
```

---

# 133. Gyors DevOps troubleshooting lista

Ha nem működik egy szolgáltatás:

```text
1. Fut?
2. Figyel a porton?
3. Jó addressre bindolt?
4. Van IP?
5. Van route?
6. DNS működik?
7. Firewall engedi?
8. Cloud firewall engedi?
9. TCP létrejön?
10. TLS működik?
11. HTTP mit válaszol?
12. Proxy eléri a backendet?
13. Backend eléri a DB-t?
14. Mit mondanak a logok?
```

---

# 134. A három legfontosabb hibakeresési parancscsoport

## Hálózati alapok

```bash
ip link
ip addr
ip route
ip neigh
```

## Kapcsolatok

```bash
ss -lntp
ping
traceroute
```

## Application/network teszt

```bash
dig
curl -v
tcpdump
```

---

# 135. Végső emlékeztető

Ne ezt kérdezd:

> Mi romlott el?

Hanem:

> Meddig jutott el biztosan a kérés?

Majd:

> Mi a következő komponens?

A networking troubleshooting alapja:

```text
working
↓
working
↓
working
↓
FAIL
```

A hibát a:

```text
legutolsó működő pont
```

és az:

```text
első nem működő pont
```

között keresd.