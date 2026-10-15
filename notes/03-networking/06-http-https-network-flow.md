# HTTP, HTTPS és a teljes hálózati kérés útja

## 1. Miről szól ez a fejezet?

Ebben a fejezetben azt követjük végig, hogy mi történik egy egyszerű webes kérésnél a kliens és a szerver között.

Például:

```text
https://api.example.com/users
```

A cél az, hogy az eddig külön megtanult fogalmak egyetlen folyamatként álljanak össze:

```text
URL
↓
DNS
↓
IP
↓
routing
↓
TCP
↓
TLS
↓
HTTP
↓
server
↓
response
```

Ez a fejezet összeköti az eddigi networking témákat.

---

# 2. A HTTP jelentése

HTTP:

```text
HyperText Transfer Protocol
```

A HTTP egy application layer protokoll.

Feladata:

```text
client
↓ request
server
↓ response
client
```

A kliens kér valamit.

A szerver válaszol.

---

# 3. HTTP request

Példa:

```http
GET /users HTTP/1.1
Host: example.com
Accept: application/json
```

Fő részei:

```text
method
path
protocol version
headers
body (ha van)
```

---

# 4. HTTP methodok

## GET

Adat lekérése.

```http
GET /users
```

---

## POST

Adat küldése / új erőforrás létrehozása.

```http
POST /users
```

---

## PUT

Teljes erőforrás frissítése.

```http
PUT /users/123
```

---

## PATCH

Részleges módosítás.

```http
PATCH /users/123
```

---

## DELETE

Törlés.

```http
DELETE /users/123
```

---

# 5. HTTP headers

A header kiegészítő információt tartalmaz az üzenetről.

Például:

```http
Host: api.example.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer ...
```

Mental model:

```text
header
→ információ az üzenetről
```

---

# 6. HTTP body

A body maga a tényleges adat.

Például:

```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "name": "Tamas",
  "age": 40
}
```

Itt:

```text
headers
+
body
```

A body:

```json
{
  "name": "Tamas",
  "age": 40
}
```

---

# 7. HTTP response

Példa:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "Tamas"
}
```

Fő részei:

```text
status line
headers
body
```

---

# 8. HTTP status code-ok

## 1xx

Információs válaszok.

---

## 2xx

Siker.

Példák:

```text
200 OK
201 Created
204 No Content
```

---

## 3xx

Átirányítás.

Példák:

```text
301 Moved Permanently
302 Found
```

---

## 4xx

Kliensoldali probléma.

Példák:

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
```

---

## 5xx

Szerveroldali probléma.

Példák:

```text
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
```

DevOps szempontból a:

```text
502
503
```

különösen fontos.

---

# 9. URL felépítése

Példa:

```text
https://api.example.com:443/users/123?active=true
```

Felbontva:

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
→ query parameter
```

Általános forma:

```text
scheme://host:port/path?query
```

---

# 10. HTTP és TCP kapcsolata

A HTTP nem szállítja önmagában az adatot a hálózaton.

Klasszikus HTTP/1.1 és HTTP/2 esetén:

```text
HTTP
↓
TCP
↓
IP
↓
Ethernet / Wi-Fi
```

A HTTP:

```text
mit kér a kliens?
```

A TCP:

```text
hogyan szállítsuk megbízhatóan?
```

---

# 11. HTTPS

HTTPS:

```text
HTTP over TLS
```

Egyszerűen:

```text
HTTP
↓
TLS
↓
TCP
↓
IP
```

A HTTPS célja, hogy a HTTP kommunikáció biztonságos legyen.

---

# 12. TLS

TLS:

```text
Transport Layer Security
```

Fő céljai:

```text
titkosítás
hitelesítés
integritás
```

---

# 13. Titkosítás

A cél:

```text
aki lehallgatja a hálózatot,
ne tudja egyszerűen elolvasni az adatot
```

Nem ezt látja:

```text
username=tamas
password=secret
```

hanem titkosított adatot.

---

# 14. Hitelesítés

A kliensnek tudnia kell:

```text
tényleg azzal a szerverrel beszélek,
akivel szeretnék?
```

Például:

```text
https://example.com
```

esetén ellenőrizni kell, hogy a szerver valóban jogosult-e az:

```text
example.com
```

domainhez tartozó certificate használatára.

---

# 15. Integritás

A TLS segít biztosítani, hogy az adatot útközben ne lehessen észrevétlenül módosítani.

Mental model:

```text
encryption
→ ne lehessen elolvasni

authentication
→ tudjam, kivel beszélek

integrity
→ ne lehessen észrevétlenül módosítani
```

---

# 16. TLS certificate

A szerver bemutat egy:

```text
TLS certificate
```

tanúsítványt.

Ez tartalmazhat például:

```text
domain information
issuer
validity period
public key
```

---

# 17. CA

CA:

```text
Certificate Authority
```

Magyarul:

```text
hitelesítésszolgáltató
```

A böngésző és az operációs rendszer megbízható CA-k listáját tartja nyilván.

A certificate ellenőrzésénél többek között azt nézzük:

```text
megbízható CA írta alá?
érvényes még?
egyezik a domain?
```

---

# 18. Public key és private key

A szerver rendelkezik:

```text
public key
private key
```

kulcspárral.

## Public key

```text
megosztható
```

## Private key

```text
titkos
csak a szerver birtokolja
```

A kettő matematikailag összetartozik.

---

# 19. Symmetric session key

A tényleges adatforgalmat nem feltétlenül public/private key párral titkosítjuk minden egyes packetnél.

A TLS kapcsolat létrehozásakor létrejöhet:

```text
symmetric session key
```

Mental model:

```text
public/private key
↓
hitelesítés + kulcsegyeztetés
↓
session key
↓
gyors titkosított adatforgalom
```

---

# 20. TLS handshake

Nagyon leegyszerűsítve:

```text
Client
↓
Client Hello

Server
↓
Server Hello
Certificate

Client
↓
certificate ellenőrzése

Client + Server
↓
kulcsegyeztetés

↓
encrypted TLS connection
```

---

# 21. Mi történik egy teljes HTTPS kérésnél?

Vegyük ezt:

```text
https://api.example.com/users
```

---

# 22. 1. URL értelmezése

A böngésző látja:

```text
https
```

Ezért tudja:

```text
HTTPS kapcsolat kell
```

Hostname:

```text
api.example.com
```

Path:

```text
/users
```

Alapértelmezett HTTPS port:

```text
443
```

Tehát logikailag:

```text
api.example.com:443
```

---

# 23. 2. DNS lookup

A gépnek IP-címre van szüksége.

```text
api.example.com
↓
DNS
↓
203.0.113.20
```

Most már:

```text
destination IP = 203.0.113.20
destination port = 443
```

---

# 24. 3. Same subnet ellenőrzés

Tegyük fel:

```text
client:
192.168.1.10/24
```

Cél:

```text
203.0.113.20
```

A kliens megállapítja:

```text
different subnet
```

Ezért:

```text
send to default gateway
```

---

# 25. 4. Default gateway

Példa:

```text
default gateway:
192.168.1.1
```

A kliens a packetet a gateway felé küldi.

Fontos:

```text
destination IP
→ távoli szerver
```

```text
destination MAC
→ helyi next hop / router
```

---

# 26. 5. ARP

A kliensnek szüksége lehet a gateway MAC-címére.

```text
192.168.1.1
↓
ARP
↓
router MAC
```

Ezután létrejöhet a helyi frame.

---

# 27. 6. Socket létrehozása

A böngésző TCP kapcsolatot akar.

Az OS ad egy ideiglenes kliensportot.

Példa:

```text
client:
192.168.1.10:52341
```

A szerver:

```text
203.0.113.20:443
```

Kapcsolat:

```text
192.168.1.10:52341
↔
203.0.113.20:443
```

---

# 28. 7. TCP 3-way handshake

```text
Client                     Server

SYN  -------------------->

     <-------------------- SYN + ACK

ACK  -------------------->
```

Ezután:

```text
TCP connection established
```

---

# 29. 8. Routing

A packet útja:

```text
PC
↓
home router
↓
ISP router
↓
más routerek
↓
destination network
```

Minden router:

```text
destination IP
↓
routing table lookup
↓
next hop
```

---

# 30. 9. NAT / PAT

Az otthoni router átírhatja:

```text
192.168.1.10:52341
```

például erre:

```text
84.x.x.x:41052
```

Kifelé a public IP jelenik meg.

---

# 31. 10. TLS handshake

A TCP után:

```text
Client Hello
↓
Server Hello
↓
Certificate
↓
certificate check
↓
key agreement
↓
encrypted channel
```

---

# 32. 11. HTTP request

Most küldhető például:

```http
GET /users HTTP/1.1
Host: api.example.com
Accept: application/json
```

HTTPS esetén a HTTP adat TLS-en belül titkosítva utazik.

---

# 33. Encapsulation

A teljes adatstruktúra leegyszerűsítve:

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

# 34. Egymásba csomagolva

```text
[ Ethernet header ]
    [ IP header ]
        [ TCP header ]
            [ TLS record ]
                [ HTTP data ]
```

Ezt nevezzük:

```text
encapsulation
```

---

# 35. Egy konkrét POST példa

Application data:

```http
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{"name":"Tamas"}
```

HTTPS miatt a tényleges HTTP tartalom titkosítva utazik.

---

# 36. Ethernet header

Példa:

```text
Source MAC:
AA:AA:AA:AA:AA:AA

Destination MAC:
BB:BB:BB:BB:BB:BB

Type:
IPv4
```

Feladata:

```text
helyi hálózati továbbítás
```

---

# 37. IP header

Példa:

```text
Source IP:
192.168.1.10

Destination IP:
203.0.113.20

TTL:
64

Protocol:
TCP
```

Feladata:

```text
end-to-end IP címzés
routing
```

---

# 38. TCP header

Példa:

```text
Source port:
52341

Destination port:
443

Sequence number:
1234567

Acknowledgment number:
7654321

Flags:
ACK, PSH
```

Feladata:

```text
kapcsolatkezelés
portok
sorrend
visszaigazolás
```

---

# 39. TLS record

Példa:

```text
Type:
Application Data

Length:
...

Encrypted payload:
...
```

A payloadban van a titkosított HTTP tartalom.

---

# 40. Az egész packet vizuálisan

```text
┌─────────────────────────────────────┐
│ Ethernet                            │
│                                     │
│ source MAC                          │
│ destination MAC                     │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ IP                              │ │
│ │                                 │ │
│ │ source IP                       │ │
│ │ destination IP                  │ │
│ │ TTL                             │ │
│ │                                 │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ TCP                         │ │ │
│ │ │                             │ │ │
│ │ │ source port                 │ │ │
│ │ │ destination port            │ │ │
│ │ │ sequence number             │ │ │
│ │ │ ACK                         │ │ │
│ │ │                             │ │ │
│ │ │ ┌─────────────────────────┐ │ │ │
│ │ │ │ TLS                     │ │ │ │
│ │ │ │                         │ │ │ │
│ │ │ │ encrypted HTTP data     │ │ │ │
│ │ │ └─────────────────────────┘ │ │ │
│ │ └─────────────────────────────┘ │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

---

# 41. Többféle cím egyszerre

Ez nagyon fontos.

## MAC

```text
következő helyi állomás
```

## IP

```text
végső gép
```

## Port

```text
végső szolgáltatás / alkalmazás
```

Mental model:

```text
MAC
→ next hop

IP
→ destination host

Port
→ destination service
```

---

# 42. Mi változik routerenként?

Az Ethernet / link-layer információ hoponként változhat.

Például:

```text
PC → router
MAC A → MAC B
```

Következő linken:

```text
router → ISP
MAC C → MAC D
```

Viszont:

```text
destination IP
```

továbbra is a végső szerver IP-je marad.

Kivétel lehet például NAT.

---

# 43. Mit néz a router?

A router tipikusan nem bontja ki az adatot egészen a HTTP requestig.

A routing fő inputja:

```text
destination IP
```

A router:

```text
destination IP
↓
routing table
↓
next hop
```

---

# 44. Mit néz a szerver?

A célgépen:

```text
destination IP
↓
TCP destination port
↓
socket
↓
webserver process
```

Például:

```text
destination port = 443
```

így a HTTPS szolgáltatás kapja a kapcsolatot.

---

# 45. Decapsulation

A szerveren:

```text
bits
↓
frame
↓
IP packet
↓
TCP segment
↓
TLS decrypt
↓
HTTP request
↓
application
```

Ez:

```text
decapsulation
```

---

# 46. Webserver feldolgozás

A webszerver megkapja például:

```http
GET /users
Host: api.example.com
```

Ezután a kérés továbbmehet:

```text
webserver
↓
backend application
↓
database
```

---

# 47. Backend és adatbázis

Például:

```sql
SELECT * FROM users;
```

A backend feldolgozza az eredményt.

Majd létrehozza a HTTP response-t.

---

# 48. HTTP response

Példa:

```http
HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "id": 1,
    "name": "Tamas"
  }
]
```

---

# 49. A response visszaútja

```text
HTTP response
↓
TLS encryption
↓
TCP
↓
IP
↓
Ethernet
↓
network
↓
client
```

---

# 50. NAT visszafelé

Ha kifelé:

```text
192.168.1.10:52341
```

átfordult:

```text
84.x.x.x:41052
```

akkor a válasz:

```text
84.x.x.x:41052
```

címre érkezik.

A router NAT táblája alapján:

```text
84.x.x.x:41052
↓
192.168.1.10:52341
```

---

# 51. A kliensoldali kicsomagolás

```text
bits
↓
Ethernet
↓
IP
↓
TCP
↓
TLS decrypt
↓
HTTP response
↓
browser
```

---

# 52. Teljes request flow egyben

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
client socket
↓
TCP handshake
↓
routing
↓
NAT/PAT
↓
TLS handshake
↓
HTTP request
↓
encapsulation
↓
network
↓
destination server
↓
decapsulation
↓
webserver
↓
application
↓
database
↓
HTTP response
↓
TLS
↓
TCP/IP
↓
network
↓
NAT reverse translation
↓
browser
```

---

# 53. Hogyan kapcsolódnak az eddigi fejezetek?

## DNS

```text
domain
→ IP
```

## Subnet

```text
local vagy remote?
```

## ARP

```text
local IP
→ MAC
```

## Routing

```text
destination IP
→ next hop
```

## NAT

```text
private IP
→ public IP
```

## TCP

```text
megbízható transport
```

## Port

```text
melyik szolgáltatás?
```

## TLS

```text
biztonságos kommunikáció
```

## HTTP

```text
mit kérünk és mit válaszolunk?
```

---

# 54. HTTP vs HTTPS

## HTTP

```text
HTTP
↓
TCP
```

Titkosítás alapból nincs.

---

## HTTPS

```text
HTTP
↓
TLS
↓
TCP
```

TLS biztosítja:

```text
encryption
authentication
integrity
```

---

# 55. HTTPS port

Alapértelmezett port:

```text
443
```

HTTP alapértelmezett port:

```text
80
```

---

# 56. Modern HTTP/3

Fontos kivétel:

```text
HTTP/3
↓
QUIC
↓
UDP
```

Tehát nem minden modern HTTP kapcsolat használ TCP-t.

A klasszikus modell:

```text
HTTP/1.1
HTTP/2
↓
TCP
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

# 57. Wiresharkban mit fogunk látni?

Egy HTTPS kapcsolatnál például:

```text
Ethernet II
Internet Protocol Version 4
Transmission Control Protocol
Transport Layer Security
```

TCP alatt:

```text
Source Port
Destination Port
Sequence Number
Acknowledgment Number
Flags
Window
```

IP alatt:

```text
Source IP
Destination IP
TTL
Protocol
```

Ethernet alatt:

```text
Source MAC
Destination MAC
```

---

# 58. HTTPS és Wireshark

Titkosított HTTPS kapcsolatnál normál esetben nem látjuk egyszerűen:

```text
POST /login
password=...
```

hanem:

```text
TLS Application Data
```

jelenik meg.

Viszont továbbra is látható lehet:

```text
IP címek
portok
TCP flags
packet méret
TLS handshake adatok
```

---

# 59. Miért hasznos a Wireshark?

A Wireshark segít összekötni:

```text
elmélet
↓
valódi packet
```

Láthatóvá válik:

```text
frame
packet
segment
MAC
IP
port
TCP
TLS
```

kapcsolata.

---

# 60. Egy teljes csomag hozzávetőleges overheadje

Egy egyszerű Ethernet + IPv4 + TCP esetben tipikusan lehet például:

```text
Ethernet header
≈ 14 byte

IPv4 header
≈ 20 byte

TCP header
≈ 20 byte

TLS header
≈ néhány byte

application data
= változó
```

Fontos:

> a pontos méretek függhetnek opcióktól és protokollverzióktól.

---

# 61. Miért kell ennyi plusz adat?

Mert minden réteg más kérdésre válaszol.

```text
MAC
→ kinek adjam át helyben?
```

```text
IP
→ melyik géphez menjen?
```

```text
Port
→ melyik szolgáltatásnak?
```

```text
TCP
→ megérkezett-e és jó sorrendben?
```

```text
TLS
→ biztonságos-e?
```

```text
HTTP
→ mit kér az alkalmazás?
```

---

# 62. Nagyon rövid mental model

```text
DNS
→ Mi az IP?
```

```text
Routing
→ Merre menjek?
```

```text
ARP
→ Mi a következő helyi MAC?
```

```text
TCP
→ Hogyan szállítsuk megbízhatóan?
```

```text
Port
→ Melyik programnak?
```

```text
TLS
→ Hogyan legyen biztonságos?
```

```text
HTTP
→ Mit kérünk?
```

---

# 63. Rövid szakmai angol

> HTTP is an application-layer protocol.

> A client sends an HTTP request.

> A server returns an HTTP response.

> HTTPS uses TLS to secure HTTP traffic.

> TLS provides encryption, authentication and integrity.

> DNS resolves a domain name to an IP address.

> Routing determines the next hop.

> TCP provides reliable transport.

> A port identifies a service.

> The destination MAC address identifies the next local hop.

> The destination IP address identifies the remote host.

> HTTP data is encapsulated inside lower-layer protocols.

---

# 64. Amit ebből biztosan érteni kell

## HTTP

```text
request
↓
response
```

## HTTPS

```text
HTTP
↓
TLS
↓
transport
```

## Full flow

```text
URL
↓
DNS
↓
IP
↓
routing
↓
TCP
↓
TLS
↓
HTTP
↓
server
↓
response
```

## Encapsulation

```text
HTTP
↓
TLS
↓
TCP
↓
IP
↓
Ethernet
↓
bits
```

## Decapsulation

```text
bits
↓
Ethernet
↓
IP
↓
TCP
↓
TLS
↓
HTTP
```

## Címzések

```text
MAC
→ next local hop

IP
→ destination host

Port
→ destination service
```

## Biztonság

```text
TLS
→ encryption
→ authentication
→ integrity
```

---

# 65. Későbbi laborfeladatok

A későbbi networking laborban érdemes lesz:

```text
curl használata
```

```text
HTTP request / response vizsgálata
```

```text
status code-ok elemzése
```

```text
headers vizsgálata
```

```text
TLS certificate ellenőrzése
```

```text
DNS lookup
```

```text
routing ellenőrzése
```

```text
Wireshark packet capture
```

és abban megkeresni:

```text
Ethernet
IP
TCP
TLS
HTTP
```

rétegeket.

A cél:

```text
egy konkrét webkérés teljes útját
valódi hálózati forgalomban is felismerni
```