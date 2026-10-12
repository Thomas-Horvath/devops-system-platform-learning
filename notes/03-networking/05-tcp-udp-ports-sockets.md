# TCP, UDP, Ports, Sockets és az OSI/TCP-IP modell

## 1. Miről szól ez a fejezet?

Ebben a fejezetben azt nézzük meg, hogyan jut el egy alkalmazás adata egy másik alkalmazáshoz a hálózaton keresztül.

A fő kérdések:

```text
Melyik géphez megy az adat?
Melyik programnak szól?
Hogyan épül fel a kapcsolat?
Hogyan tudjuk, hogy az adat megérkezett?
Mi történik, ha elveszik?
Mi az a socket?
Mi a különbség TCP és UDP között?
Hogyan illeszkedik mindez az OSI modellbe?
```

A legfontosabb fogalmak:

```text
OSI model
TCP/IP model
encapsulation
port
socket
TCP
UDP
segment
datagram
connection
3-way handshake
sequence number
ACK
retransmission
```

---

# 2. Először a nagy kép

Tegyük fel, hogy a böngésző elküld egy kérést:

```text
POST /api/user
{"name":"Tamas"}
```

A böngésző szempontjából ez application data.

De ahhoz, hogy eljusson a távoli szerverhez, további hálózati információkra van szükség.

Nagyon leegyszerűsítve:

```text
Application data
↓
TCP / UDP
↓
IP
↓
Ethernet / Wi-Fi
↓
fizikai továbbítás
```

A fogadó oldalon ugyanez visszafelé:

```text
fizikai jel
↓
Ethernet / Wi-Fi
↓
IP
↓
TCP / UDP
↓
application
```

---

# 3. Mi az OSI modell?

Az **OSI model** egy gondolkodási modell.

Nem azt jelenti, hogy a számítógépben fizikailag hét külön doboz van.

Arra szolgál, hogy külön tudjuk választani a hálózati kommunikáció különböző feladatait.

A hét réteg:

```text
7 Application
6 Presentation
5 Session
4 Transport
3 Network
2 Data Link
1 Physical
```

Magyarul:

```text
7 Alkalmazási
6 Megjelenítési
5 Viszonykezelési
4 Szállítási
3 Hálózati
2 Adatkapcsolati
1 Fizikai
```

---

# 4. Az OSI modell rétegei egyszerűen

## 7. Application layer

Kérdés:

```text
Mit akar az alkalmazás csinálni?
```

Példák:

```text
HTTP
DNS
SMTP
SSH
```

Itt dolgozik például:

```text
browser
web server
DNS client
SSH client
```

---

## 6. Presentation layer

Feladata lehet:

```text
adatformátum
kódolás
titkosítás
```

Példák:

```text
TLS
JSON
UTF-8
```

A modern hálózati stackekben ez nem mindig különül el élesen az application layertől.

---

## 5. Session layer

A kommunikációs munkamenetek kezelésének gondolata.

Például:

```text
session
kapcsolati állapot
munkamenet
```

A modern TCP/IP rendszerekben ez sem feltétlenül külön, önálló technikai rétegként jelenik meg.

---

## 4. Transport layer

Itt jön:

```text
TCP
UDP
ports
```

Fő kérdés:

```text
Melyik alkalmazási folyamatnak szól az adat,
és hogyan szállítsuk?
```

---

## 3. Network layer

Itt találjuk:

```text
IP
routing
routers
TTL
```

Fő kérdés:

```text
Melyik géphez / hálózathoz kell eljutni?
```

---

## 2. Data Link layer

A helyi hálózati továbbítás.

Példák:

```text
Ethernet
MAC address
switch
frame
```

Fő kérdés:

```text
A következő helyi hálózati lépés kicsoda?
```

---

## 1. Physical layer

A tényleges fizikai továbbítás.

Példák:

```text
kábel
Wi-Fi rádiójel
elektromos jel
fényjel
```

---

# 5. OSI – hogyan kell elképzelni a folyamatot?

A küldő oldalon:

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

A fogadó oldalon:

```text
Physical
↓
Data Link
↓
Network
↓
Transport
↓
Application
```

Mental model:

```text
küldő
→ becsomagol

fogadó
→ kicsomagol
```

---

# 6. Encapsulation

Az **encapsulation** azt jelenti, hogy az adat köré rétegenként további információ kerül.

Például:

```text
{"name":"Tamas"}
```

Ez maga az application data.

Erre kerülhet TCP header:

```text
[TCP header]
{"name":"Tamas"}
```

Erre IP header:

```text
[IP header]
[TCP header]
{"name":"Tamas"}
```

Majd Ethernet header:

```text
[Ethernet header]
[IP header]
[TCP header]
{"name":"Tamas"}
```

Nagyon leegyszerűsítve:

```text
data
↓
segment
↓
packet
↓
frame
↓
bits
```

---

# 7. Decapsulation

A fogadó oldalon ennek fordítottja történik.

```text
bits
↓
frame
↓
packet
↓
segment
↓
application data
```

A rendszer rétegenként feldolgozza a hozzá tartozó információkat.

---

# 8. A TCP/IP modell

A gyakorlatban sokszor egyszerűbb a TCP/IP modell.

Ez kevesebb réteget használ:

```text
Application
Transport
Internet
Network Access
```

Nagyjából:

```text
OSI 7-6-5
→ Application

OSI 4
→ Transport

OSI 3
→ Internet

OSI 2-1
→ Network Access
```

A valós modern operációs rendszerek hálózati működésének elképzeléséhez ez sokszor intuitívabb.

---

# 9. Network stack

Az operációs rendszer hálózati protokollokat kezelő részeit gyakran:

```text
network stack
```

néven emlegetjük.

Például:

```text
Browser
↓
OS network stack
├── TCP
├── IP
└── network interface
     ↓
Ethernet / Wi-Fi
```

A browser nem maga készíti kézzel az Ethernet frame-eket.

Az operációs rendszer hálózati szolgáltatásait használja.

---

# 10. Mi az a port?

Az IP-cím megmondja:

```text
melyik gép / hálózati interfész?
```

A port megmondja:

```text
azon a gépen melyik hálózati szolgáltatás?
```

Példa:

```text
203.0.113.20:443
```

Itt:

```text
203.0.113.20
→ IP address

443
→ port
```

---

# 11. Portok tartománya

A portszám:

```text
0–65535
```

között lehet.

Néhány gyakori port:

```text
22   → SSH
53   → DNS
80   → HTTP
443  → HTTPS
3306 → MySQL
5432 → PostgreSQL
```

---

# 12. IP + port

Ugyanazon a szerveren több szolgáltatás futhat.

Például:

```text
203.0.113.20:22
→ SSH

203.0.113.20:443
→ HTTPS

203.0.113.20:5432
→ PostgreSQL
```

Mental model:

```text
IP
→ melyik épület?

Port
→ melyik ajtó?
```

---

# 13. Mi az a socket?

A socket egy operációs rendszer által kezelt hálózati kommunikációs végpont.

Egyszerű mental model:

```text
program = iroda

socket = kommunikációs szoba

port = a szoba ajtószáma
```

A program a socketen keresztül tud hálózati adatot küldeni és fogadni.

---

# 14. Socketes hasonlat

Képzelj el egy irodát:

```text
Browser
```

Az iroda több kommunikációs szobát is használhat.

```text
Browser
├── Socket 1
├── Socket 2
└── Socket 3
```

Mindegyik sockethez tartozhat egy helyi port.

Például:

```text
Socket 1
192.168.1.10:52341

Socket 2
192.168.1.10:52342

Socket 3
192.168.1.10:52343
```

---

# 15. Egy socket és a port

Egy sockethez egy adott helyi kommunikációs végponton egy konkrét port tartozik.

Például:

```text
192.168.1.10:52341
```

Itt:

```text
192.168.1.10
→ helyi IP

52341
→ helyi port
```

A port csak egy szám.

A socket már egy tényleges operációs rendszer által kezelt kommunikációs objektum/végpont.

---

# 16. Egy program több socketet használhat

Például egy böngésző:

```text
Browser

Socket 1
192.168.1.10:52341
→ Server A:443

Socket 2
192.168.1.10:52342
→ Server B:443

Socket 3
192.168.1.10:52343
→ Server C:443
```

Tehát:

```text
egy app
→ sok socket
```

---

# 17. Szerver socket

Egy webszerver például figyelhet:

```text
TCP port 443
```

A program létrehoz egy listening socketet.

Mental model:

```text
Webserver
↓
listening socket
↓
port 443
```

Például Node.js-ben:

```javascript
server.listen(3000)
```

mögött az operációs rendszer socketkezelése dolgozik.

---

# 18. Egy szerverport sok kapcsolatot kezelhet

Fontos:

```text
Client A:52341 ─┐
Client B:53110 ─┼──→ Server:443
Client C:60222 ─┘
```

Mindhárom kliens ugyanahhoz a szerverporthoz csatlakozik:

```text
443
```

de külön TCP kapcsolatról van szó.

---

# 19. Ephemeral port

A kliens által használt ideiglenes portot gyakran:

```text
ephemeral port
```

néven emlegetjük.

Példa:

```text
Client:
192.168.1.10:52341

Server:
203.0.113.20:443
```

A `52341` egy ideiglenesen kiosztott kliensport lehet.

---

# 20. Egy TCP kapcsolat azonosítása

Egy TCP kapcsolatot a végpontok kombinációja különböztet meg.

Leegyszerűsítve:

```text
source IP
source port
destination IP
destination port
protocol
```

Példa:

```text
TCP

192.168.1.10:52341
↔
203.0.113.20:443
```

---

# 21. Mi az a TCP?

A **TCP** jelentése:

```text
Transmission Control Protocol
```

Kapcsolatorientált transport protokoll.

Fő célja:

```text
megbízható
sorrendhelyes
kapcsolatalapú
adatátvitel
```

---

# 22. Mit jelent, hogy connection-oriented?

TCP-ben a két végpont előbb kapcsolatot épít fel.

Nem rögtön az application data küldésével kezdünk.

Először:

```text
connection establishment
```

történik.

---

# 23. TCP 3-way handshake

A kapcsolatfelépítés tipikus folyamata:

```text
Client                     Server

SYN  -------------------->

     <-------------------- SYN + ACK

ACK  -------------------->
```

Ez a:

```text
3-way handshake
```

---

# 24. A handshake egyszerűen

Mental model:

```text
Client:
"Szeretnék kapcsolatot."
```

```text
Server:
"Rendben, hallak.
Te is hallasz engem?"
```

```text
Client:
"Igen."
```

Ezután:

```text
connection established
```

---

# 25. SYN

A:

```text
SYN
```

jelentése:

```text
Synchronize
```

A TCP kapcsolat kezdetekor a végpontok szinkronizálják a kommunikációhoz szükséges állapotot, többek között a sequence numberöket.

---

# 26. ACK

Az:

```text
ACK
```

jelentése:

```text
Acknowledgment
```

Egyszerűen:

```text
"megkaptam"
```

jellegű visszaigazolás.

---

# 27. TCP sequence number

A TCP adatfolyam részeit sorrendben kell kezelni.

Ehhez használ:

```text
sequence number
```

információkat.

Nagyon leegyszerűsítve:

```text
Data 1
Data 2
Data 3
```

Ha a hálózaton ilyen sorrendben érkezik:

```text
Data 2
Data 1
Data 3
```

a TCP képes helyes sorrendbe rendezni az adatfolyamot.

---

# 28. Miért kell sorrend?

Például:

```text
HELLO
```

nem ugyanaz, mint:

```text
LHELO
```

Az alkalmazás számára fontos lehet, hogy a byte-ok eredeti sorrendben érkezzenek.

A TCP ezt kezeli.

---

# 29. TCP acknowledgment

A fogadó fél visszajelezheti, hogy meddig kapta meg az adatot.

Ez az:

```text
ACK
```

mechanizmus.

Például nagyon leegyszerűsítve:

```text
Sender:
küldök adatot

Receiver:
ACK
→ megkaptam
```

---

# 30. Retransmission

Ha egy szükséges TCP adatdarab elveszik, a TCP képes újraküldeni.

Ez:

```text
retransmission
```

Például:

```text
1 → megérkezett
2 → elveszett
3 → megérkezett
```

A TCP felismerheti, hogy hiányzik adat, és a szükséges rész újraküldhető.

---

# 31. TCP reliability

A TCP fő megbízhatósági mechanizmusai közé tartozik például:

```text
sequence numbers
ACK
retransmission
ordering
flow control
congestion control
```

Ezek részletes algoritmusait egyelőre nem kell tudni.

A lényeg:

```text
TCP megpróbál megbízható byte streamet biztosítani az alkalmazásnak.
```

---

# 32. TCP byte stream

A TCP-t érdemes adatfolyamként elképzelni.

Az alkalmazás küld byte-okat:

```text
A B C D E F G ...
```

A TCP feladata, hogy a másik oldalon ezt megbízható, megfelelő sorrendű adatfolyamként adja át.

Fontos:

> A TCP nem őrzi meg feltétlenül az alkalmazás eredeti „üzenethatárait”.

Ez később programozásnál fontos.

---

# 33. Flow control

A **flow control** célja, hogy a küldő ne öntse el gyorsabban adatokkal a fogadót, mint ahogy az képes feldolgozni őket.

Mental model:

```text
receiver:
"ennyi adatot tudok most fogadni"
```

A TCP ehhez is tartalmaz mechanizmusokat.

---

# 34. Congestion control

A TCP a hálózat túlterheltségéhez is próbál alkalmazkodni.

Ez:

```text
congestion control
```

Nem ugyanaz, mint a flow control.

```text
flow control
→ receiver védelme

congestion control
→ network túlterhelésének kezelése
```

Ezt később elég mélyebben megtanulni.

---

# 35. TCP kapcsolat lezárása

A TCP kapcsolatot kontrolláltan le is lehet zárni.

Itt találkozhatunk:

```text
FIN
ACK
```

flag-ekkel.

Nagyon leegyszerűsítve:

```text
"Én már nem küldök több adatot."
```

A részletes lezárási folyamatot egyelőre nem szükséges bemagolni.

---

# 36. Mi az a TCP segment?

A transport layer TCP egységére gyakran azt mondjuk:

```text
TCP segment
```

Nagyon leegyszerűsítve:

```text
TCP header
+
application data
```

Példa:

```text
[TCP header]
[HTTP/TLS data]
```

---

# 37. TCP headerben található információk

Példák:

```text
source port
destination port
sequence number
acknowledgment number
flags
window information
checksum
```

Ezek segítségével kezeli a TCP a kapcsolatot.

---

# 38. Mi az UDP?

Az **UDP** jelentése:

```text
User Datagram Protocol
```

Egy egyszerűbb transport protokoll.

Fő jellemzője:

```text
connectionless
```

vagyis nincs TCP-szerű kapcsolatfelépítés.

---

# 39. UDP-nél nincs 3-way handshake

Nem:

```text
SYN
SYN-ACK
ACK
```

hanem az alkalmazás küldhet UDP datagramot közvetlenül.

Például:

```text
Client
↓
UDP datagram
↓
Server
```

---

# 40. UDP datagram

UDP esetén a transport réteg adat-egységét gyakran:

```text
datagram
```

néven emlegetjük.

Leegyszerűsítve:

```text
UDP header
+
application data
```

---

# 41. Mit NEM garantál az UDP?

Az UDP önmagában nem garantálja:

```text
hogy az adat megérkezik
```

nem garantálja:

```text
hogy csak egyszer érkezik meg
```

és nem garantálja:

```text
hogy sorrendben érkezik meg
```

---

# 42. UDP elveszett packet

Például:

```text
Datagram 1 → megérkezik
Datagram 2 → elveszik
Datagram 3 → megérkezik
```

Az UDP önmagában nem mondja:

```text
"Hiányzik a 2-es, küldd újra."
```

Ha erre szükség van, azt az alkalmazási protokollnak kell kezelnie.

---

# 43. Miért használunk mégis UDP-t?

Mert sok esetben nincs szükség TCP minden szolgáltatására.

UDP:

```text
egyszerűbb
kevesebb protokollállapot
nincs TCP handshake
üzenetorientált
```

Ez hasznos lehet olyan rendszereknél, ahol az alkalmazás maga akarja kezelni a szükséges megbízhatóságot vagy ahol a késleltetés fontos.

---

# 44. TCP vs UDP

```text
TCP
→ connection-oriented
→ reliable delivery mechanisms
→ ordering
→ retransmission
→ byte stream
```

```text
UDP
→ connectionless
→ no built-in delivery guarantee
→ no built-in ordering
→ no built-in retransmission
→ datagram-oriented
```

---

# 45. TCP és UDP sebesség

Nem teljesen pontos azt mondani:

```text
TCP = lassú
UDP = gyors
```

A jobb megfogalmazás:

```text
UDP
→ kevesebb beépített mechanizmus
→ kisebb transport overhead
```

TCP:

```text
→ több állapotot és megbízhatósági mechanizmust kezel
```

A tényleges teljesítmény mindig a konkrét alkalmazástól és hálózattól függ.

---

# 46. Mikor TCP?

Tipikus példák:

```text
HTTP/1.1
HTTP/2
HTTPS klasszikusan
SSH
SMTP
IMAP
PostgreSQL
MySQL
```

Itt általában fontos, hogy az adat:

```text
megérkezzen
helyes sorrendben érkezzen
ne hiányozzon belőle rész
```

---

# 47. Mikor UDP?

Tipikus területek lehetnek:

```text
DNS lekérdezések
real-time voice/video
online gaming
streaming bizonyos részei
network discovery
```

Fontos:

> Sok modern alkalmazás saját protokollt épít UDP fölé.

---

# 48. QUIC és HTTP/3

Modern példa:

```text
HTTP/3
↓
QUIC
↓
UDP
```

A QUIC UDP-t használ alatta, de saját maga valósít meg több olyan mechanizmust, amelyet hagyományosan TCP-től várnánk.

Tehát:

```text
UDP használata
≠
az alkalmazás biztosan megbízhatatlan
```

Az UDP csak azt jelenti, hogy maga az UDP nem biztosítja ezeket a mechanizmusokat.

---

# 49. HTTPS és TCP

Klasszikus HTTPS esetben:

```text
HTTP
↓
TLS
↓
TCP
↓
IP
↓
Ethernet / Wi-Fi
```

A szerver általában:

```text
TCP 443
```

porton figyel.

Modern HTTP/3 esetében a transport alapja eltérhet:

```text
HTTP/3
↓
QUIC
↓
UDP
```

---

# 50. DNS és UDP/TCP

DNS lekérdezések gyakran használhatnak:

```text
UDP 53
```

portot.

Bizonyos helyzetekben DNS TCP-t is használhat.

Ezért nem érdemes úgy megtanulni:

```text
DNS = mindig UDP
```

hanem inkább:

```text
DNS
→ gyakran UDP
→ szükség esetén TCP is lehet
```

---

# 51. Port és protocol együtt

Ugyanaz a portszám TCP és UDP esetén külön namespace-ben értelmezhető.

Például:

```text
TCP 53
UDP 53
```

nem ugyanaz a transport endpoint.

Ezért fontos:

```text
IP + port + protocol
```

együtt.

---

# 52. Socket egyszerű technikai mental model

Egy socketet közelítőleg elképzelhetünk így:

```text
protocol
+
local IP
+
local port
```

Kapcsolt TCP socketnél ehhez társul a remote endpoint is:

```text
remote IP
remote port
```

Példa:

```text
TCP

Local:
192.168.1.10:52341

Remote:
203.0.113.20:443
```

---

# 53. Listening socket és connected socket

Szerveroldalon fontos különbség.

A server először:

```text
listen
```

állapotban várhat.

Például:

```text
0.0.0.0:443
LISTEN
```

Amikor kliens csatlakozik, abból külön kapcsolat jön létre.

Például:

```text
192.168.1.10:52341
↔
203.0.113.20:443
```

A listening socket továbbra is képes új klienseket fogadni.

---

# 54. A socket-hasonlat összefoglalása

```text
Program
→ iroda
```

```text
Socket
→ kommunikációs szoba
```

```text
Port
→ a szoba ajtószáma
```

```text
TCP
→ a kommunikáció/szállítás szabályrendszere
```

Egy iroda:

```text
Browser
```

használhat sok szobát:

```text
Socket 1
Socket 2
Socket 3
```

és ezeknek lehet külön portjuk.

---

# 55. Hogyan kapcsolódik mindez a NAT-hoz?

Például:

```text
192.168.1.10:52341
→
203.0.113.20:443
```

A NAT/PAT router kifelé módosíthatja:

```text
84.x.x.x:41052
→
203.0.113.20:443
```

Ezért fontosak a portok a NAT kapcsolatkövetésénél is.

---

# 56. Hogyan kapcsolódik mindez a routinghoz?

A router elsősorban:

```text
destination IP
```

alapján routingol.

A port viszont a végpontnál segít eldönteni:

```text
melyik szolgáltatásnak / socketnek kell megkapnia?
```

Nagyon leegyszerűsítve:

```text
IP
→ elvisz a gépig

port
→ elvisz a megfelelő programig
```

---

# 57. Egy teljes webes példa

A browser szeretne HTTPS kapcsolatot:

```text
https://example.com
```

DNS után megvan:

```text
203.0.113.20
```

A browser létrehoz egy kapcsolatot:

```text
Local:
192.168.1.10:52341

Remote:
203.0.113.20:443

Protocol:
TCP
```

Ezután:

```text
TCP handshake
↓
TLS
↓
HTTP request
```

A hálózati stack:

```text
HTTP/TLS data
↓
TCP segment
↓
IP packet
↓
Ethernet frame
↓
physical network
```

---

# 58. Mi változik routerenként?

A packet végső cél IP-je továbbra is a cél szerverre mutat.

Például:

```text
Destination IP:
203.0.113.20
```

A helyi frame destination MAC-je viszont:

```text
default gateway MAC
```

A következő hálózati linken új frame készül.

Tehát:

```text
MAC
→ hoponként változhat
```

Az IP végpontok jellemzően end-to-end értelmezésűek, bár NAT módosíthatja őket.

---

# 59. Frame, packet, segment

Nagyon gyakori fogalmak:

```text
TCP segment
```

Transport layer.

```text
IP packet
```

Network layer.

```text
Ethernet frame
```

Data Link layer.

Egyszerűsítve:

```text
Ethernet frame
└── IP packet
    └── TCP segment
        └── application data
```

---

# 60. Wiresharkban mit fogunk látni?

Például egy HTTPS kapcsolatnál:

```text
Ethernet II
Internet Protocol Version 4
Transmission Control Protocol
Transport Layer Security
```

Ezeket ki lehet nyitni és megnézni külön-külön.

Például TCP-nél:

```text
Source Port
Destination Port
Sequence Number
Acknowledgment Number
Flags
Window
```

IP-nél:

```text
Source IP
Destination IP
TTL
Protocol
```

Ethernetnél:

```text
Source MAC
Destination MAC
```

Ez lesz az egyik legjobb módja annak, hogy az OSI/TCP-IP rétegek ne csak elméleti fogalmak legyenek.

---

# 61. Fontos TCP flag-ek

Első körben ezekkel érdemes találkozni:

```text
SYN
ACK
FIN
RST
```

Egyszerűen:

```text
SYN
→ kapcsolat indítása
```

```text
ACK
→ visszaigazolás
```

```text
FIN
→ normál kapcsolatlezárás
```

```text
RST
→ kapcsolat azonnali resetelése / megszakítása
```

---

# 62. Rövid TCP mental model

```text
kapcsolat létrehozása
↓
3-way handshake
↓
adatküldés
↓
sequence + ACK
↓
ha kell retransmission
↓
kapcsolat lezárása
```

---

# 63. Rövid UDP mental model

```text
adat elkészítése
↓
UDP datagram
↓
elküldés
```

Nincs beépített:

```text
3-way handshake
ACK
ordering
retransmission
```

---

# 64. TCP vs UDP táblázat

| Tulajdonság | TCP | UDP |
|---|---|---|
| Kapcsolatorientált | Igen | Nem |
| Handshake | Igen | Nincs TCP-szerű handshake |
| Sorrendhelyesség | Igen | Nem garantált |
| Újraküldés | Van | Nincs beépítve |
| ACK | Van | Nincs beépítve |
| Adatmodell | Byte stream | Datagram |
| Transport overhead | Nagyobb | Kisebb |
| Tipikus használat | Web, SSH, DB | DNS, realtime, gaming |

---

# 65. Rövid szakmai angol

> An IP address identifies a network endpoint.

> A port identifies a service on a host.

> A socket is a communication endpoint managed by the operating system.

> A process can use multiple sockets.

> TCP is connection-oriented.

> TCP provides reliable and ordered delivery.

> UDP is connectionless.

> UDP does not guarantee delivery or ordering.

> TCP uses sequence numbers and acknowledgments.

> Lost TCP data can be retransmitted.

> The TCP three-way handshake establishes a connection.

> A TCP segment is encapsulated inside an IP packet.

> An IP packet can be encapsulated inside an Ethernet frame.

---

# 66. Amit ebből biztosan érteni kell

## Rétegek

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

## Encapsulation

```text
Application data
↓
TCP/UDP
↓
IP
↓
Ethernet
↓
bits
```

## Címzés

```text
IP
→ melyik gép?

Port
→ melyik szolgáltatás?
```

## Socket

```text
program által használt
OS-managed
kommunikációs végpont
```

Mental model:

```text
program = iroda
socket = kommunikációs szoba
port = ajtószám
```

## TCP

```text
connection-oriented
reliable
ordered
ACK
retransmission
```

## UDP

```text
connectionless
no built-in delivery guarantee
no built-in ordering
no built-in retransmission
```

## TCP kapcsolat

```text
client IP:port
↔
server IP:port
```

Példa:

```text
192.168.1.10:52341
↔
203.0.113.20:443
```

## A teljes gondolat

```text
Application
↓
socket
↓
TCP / UDP
↓
IP
↓
Ethernet / Wi-Fi
↓
network
↓
remote host
↓
remote socket
↓
remote application
```

---

# 67. Későbbi laborban gyakorolandó

A gyakorlati laborban érdemes lesz megnézni:

```text
ss
netstat
lsof
```

vagy hasonló eszközökkel:

```text
mely portok figyelnek?
milyen socketek vannak?
milyen TCP kapcsolatok aktívak?
```

Majd Wiresharkkal:

```text
TCP 3-way handshake
SYN
SYN-ACK
ACK
```

és:

```text
source/destination IP
source/destination port
sequence number
ACK
TTL
MAC address
```

megfigyelését.

Ez segít összekötni:

```text
OSI
TCP/IP
socket
port
TCP
UDP
IP
Ethernet
```

fogalmait a tényleges hálózati forgalommal.