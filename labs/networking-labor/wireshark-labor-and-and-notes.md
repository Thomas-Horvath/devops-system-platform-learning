# Wireshark Lab – Packetek megfigyelése WSL forgalommal

## 1. A labor célja

Ebben a laborban két dolgot tanulunk egyszerre:

1. a Wireshark alapvető használatát;
2. az eddig tanult networking fogalmak felismerését valódi hálózati forgalomban.

A cél, hogy az eddigi elmélet:

```text
MAC
IP
ARP
DNS
TCP
UDP
port
socket
TLS
HTTP
packet
frame
segment
```

ne csak fogalom legyen.

A Wiresharkban ténylegesen látni fogjuk őket.

---

# 2. Mit csinál a Wireshark?

A Wireshark egy:

```text
packet analyzer
```

vagy:

```text
protocol analyzer
```

Program.

Képes a hálózati interfészen áthaladó forgalmat:

```text
capture
↓
decode
↓
display
↓
filter
↓
analyze
```

formában megmutatni.

Egyszerűen:

```text
hálózati forgalom
↓
Wireshark
↓
ember számára olvasható protokollinformációk
```

---

# 3. Wireshark vs tcpdump

Mindkettő hálózati forgalmat vizsgál.

## tcpdump

```text
CLI
→ terminál
```

## Wireshark

```text
GUI
→ grafikus felület
```

Mental model:

```text
tcpdump
→ parancssoros packet capture

Wireshark
→ grafikus packet capture + analysis
```

Később mindkettőt használhatjuk.

---

# 4. A labor környezete

A jelenlegi környezetünk:

```text
Windows 11
│
├── Wireshark
│
└── WSL2 Ubuntu
      ↓
    hálózati forgalom
```

A Wireshark Windows alatt fut.

A tesztforgalmat főleg WSL-ben generáljuk:

```bash
ping
dig
curl
```

parancsokkal.

---

# 5. Fontos WSL sajátosság

WSL2 virtuális hálózatot használ.

Ezért a Wireshark interface-listájában több hálózati interface is megjelenhet.

Például lehet:

```text
Ethernet
Wi-Fi
Loopback
vEthernet
WSL-related virtual interface
```

A pontos név Windows- és WSL-verziótól függhet.

Ezért nem tanuljuk meg azt, hogy:

> „mindig az XY nevű interface-t válaszd”.

Hanem megtanuljuk felismerni, melyiken történik ténylegesen a forgalom.

---

# 6. A Wireshark főképernyője

Indítsd el a Wiresharkot.

A kezdőlapon hálózati interface-eket fogsz látni.

Például:

```text
Wi-Fi
Ethernet
Loopback
virtual interfaces...
```

Az interface neve mellett gyakran kis grafikon mutatja a forgalmat.

Ahol aktív forgalom van:

```text
~~~~~~
```

ott valószínűleg packetek haladnak.

---

# 7. Első fontos fogalom: Capture

**Capture**:

```text
a hálózati packetek rögzítése
```

A Wireshark capture közben folyamatosan gyűjti a packeteket.

Mental model:

```text
network interface
↓
packet
packet
packet
packet
↓
Wireshark capture
```

---

# 8. Capture indítása

Válassz egy aktív interface-t.

Dupla kattintással elindítható a capture.

Ezután megjelennek a packetek.

A képernyő gyorsan elkezdhet megtelni.

Ez normális.

Egy átlagos gépen rengeteg háttérforgalom történik:

```text
DNS
HTTPS
Windows services
browser traffic
cloud sync
applications
```

---

# 9. Capture leállítása

A felső eszköztáron található piros stop gombbal állítsd le.

Mental model:

```text
Start Capture
↓
packetek gyűjtése
↓
Stop Capture
```

Elemzéshez sokszor jobb először:

```text
capture
↓
stop
↓
analyze
```

---

# 10. A Wireshark három fő panelje

A Wireshark fő nézete három fontos részre oszlik.

## Packet List

Felül:

```text
packetek listája
```

Általában oszlopok:

```text
No.
Time
Source
Destination
Protocol
Length
Info
```

---

## Packet Details

Középen:

```text
az adott packet protokollrétegei
```

Például:

```text
Frame
Ethernet II
Internet Protocol Version 4
Transmission Control Protocol
Transport Layer Security
```

---

## Packet Bytes

Alul:

```text
a packet nyers byte-jai
```

Hexadecimális formában.

Ez már közelebb van ahhoz, ami ténylegesen a hálózaton utazik.

---

# 11. A három panel kapcsolata

```text
Packet List
→ melyik packetet nézem?
```

```text
Packet Details
→ mit tartalmaz?
```

```text
Packet Bytes
→ milyen byte-okból áll?
```

Ha a felső listában kiválasztasz egy packetet, az alsó két rész annak tartalmát mutatja.

---

# 12. Első packet megnyitása

Kattints egy packetre.

A középső panelen valami ilyesmit láthatsz:

```text
Frame
Ethernet II
Internet Protocol Version 4
Transmission Control Protocol
Transport Layer Security
```

Ez pontosan az a rétegezés, amit korábban tanultunk.

---

# 13. Rétegek kibontása

A sorok melletti nyíllal bontsd ki például:

```text
Ethernet II
```

majd:

```text
Internet Protocol Version 4
```

majd:

```text
Transmission Control Protocol
```

Most már konkrét mezőket láthatsz.

---

# 14. Ethernet réteg

Az:

```text
Ethernet II
```

alatt keresd:

```text
Source
Destination
Type
```

Például:

```text
Source: aa:bb:cc:dd:ee:ff
Destination: 11:22:33:44:55:66
Type: IPv4
```

Itt MAC-címeket látunk.

Mental model:

```text
Ethernet
↓
MAC addresses
↓
local link
```

---

# 15. IP réteg

Nyisd ki:

```text
Internet Protocol Version 4
```

Keresd:

```text
Source Address
Destination Address
Time to Live
Protocol
```

Például:

```text
Source: 172.x.x.x
Destination: 93.x.x.x
TTL: 64
Protocol: TCP
```

Mental model:

```text
IP
↓
source host
destination host
routing
```

---

# 16. TCP réteg

Nyisd ki:

```text
Transmission Control Protocol
```

Keresd:

```text
Source Port
Destination Port
Sequence Number
Acknowledgment Number
Flags
Window
```

Példa:

```text
Source Port: 52341
Destination Port: 443
```

Itt válik láthatóvá:

```text
client ephemeral port
↓
server port
```

---

# 17. Encapsulation valódi packetben

Most már ténylegesen láthatod:

```text
Frame
└── Ethernet
    └── IP
        └── TCP
            └── TLS / application data
```

Ez az:

```text
encapsulation
```

amit elméletben már tanultunk.

---

# 18. Display Filter

Ez lesz a Wireshark egyik legfontosabb funkciója.

A felső:

```text
Display Filter
```

mezőbe szűrőfeltételeket írhatunk.

Például:

```text
tcp
```

Ezután csak TCP packetek jelennek meg.

Fontos:

A packetek nem törlődnek.

Csak:

```text
elrejtjük azt, ami nem illeszkedik a filterre
```

---

# 19. Display filter vs capture filter

Ezt nagyon fontos nem összekeverni.

## Display filter

Capture után:

```text
mit mutasson a Wireshark?
```

Példa:

```text
tcp
```

---

## Capture filter

Capture előtt:

```text
mit rögzítsen egyáltalán?
```

Ez más szintaxisú rendszer.

Kezdetben főleg:

```text
Display Filter
```

használatát tanuljuk.

---

# 20. Első display filterek

TCP:

```text
tcp
```

UDP:

```text
udp
```

DNS:

```text
dns
```

ARP:

```text
arp
```

ICMP:

```text
icmp
```

TLS:

```text
tls
```

HTTP:

```text
http
```

---

# 21. Filter törlése

Töröld ki a Display Filter mezőt, majd Enter.

Ekkor újra minden capture-ölt packet megjelenik.

---

# 22. IP alapján szűrés

Példa:

```text
ip.addr == 8.8.8.8
```

Ez megmutatja azokat a packeteket, ahol:

```text
source IP = 8.8.8.8
```

VAGY:

```text
destination IP = 8.8.8.8
```

---

# 23. Source IP filter

```text
ip.src == 192.168.1.10
```

Jelentése:

```text
csak azok,
amik erről az IP-ről jönnek
```

---

# 24. Destination IP filter

```text
ip.dst == 8.8.8.8
```

Jelentése:

```text
csak a 8.8.8.8 felé menő packetek
```

---

# 25. Port alapján szűrés

Például:

```text
tcp.port == 443
```

Ez megmutatja:

```text
source port = 443
VAGY
destination port = 443
```

TCP 8000:

```text
tcp.port == 8000
```

---

# 26. Destination port

```text
tcp.dstport == 443
```

Csak azok a TCP packetek:

```text
amelyek célportja 443
```

---

# 27. Source port

```text
tcp.srcport == 443
```

Csak azok:

```text
amelyek forrásportja 443
```

---

# 28. Filterek kombinálása

AND:

```text
tcp && tcp.port == 443
```

OR:

```text
dns || arp
```

NOT:

```text
!tcp
```

Példa:

```text
ip.addr == 8.8.8.8 && icmp
```

Jelentése:

```text
8.8.8.8-hoz kapcsolódó ICMP traffic
```

---

# 29. LAB 1 – ICMP / ping

Most kezdjük el a tényleges packet-generálást.

Indíts új capture-t.

WSL-ben:

```bash
ping -c 4 8.8.8.8
```

Állítsd le a capture-t.

Wireshark filter:

```text
icmp
```

---

# 30. Ping packetek felismerése

Keress:

```text
Echo request
```

és:

```text
Echo reply
```

packeteket.

A folyamat:

```text
WSL
↓
ICMP Echo Request
↓
8.8.8.8

8.8.8.8
↓
ICMP Echo Reply
↓
WSL
```

---

# 31. Ping packet elemzése

Nyiss meg egy Echo Request packetet.

Nézd meg:

```text
Source IP
Destination IP
TTL
ICMP Type
Sequence Number
```

Ezután nyisd meg a hozzá tartozó Echo Reply packetet.

Hasonlítsd össze:

```text
source
destination
TTL
sequence
```

---

# 32. LAB 1 kérdések

Válaszold meg:

1. Mi volt az Echo Request source IP-je?
2. Mi volt a destination IP?
3. Mi változott az Echo Reply-ban?
4. Milyen protokollt használtunk?
5. Volt TCP port?
6. Miért nincs TCP port?

Fontos felismerés:

```text
ping
→ ICMP
→ nincs TCP/UDP port
```

---

# 33. LAB 2 – DNS query

Indíts új capture-t.

WSL:

```bash
dig example.com
```

Állítsd le a capture-t.

Filter:

```text
dns
```

---

# 34. DNS packetek

Keress egy:

```text
Standard query
```

és egy:

```text
Standard query response
```

packetet.

A folyamat:

```text
WSL
↓
DNS query
↓
DNS resolver

DNS resolver
↓
DNS response
↓
WSL
```

---

# 35. DNS query elemzése

Nyisd ki:

```text
Domain Name System
```

Keresd:

```text
Queries
Name
Type
Class
```

Például:

```text
Name: example.com
Type: A
```

---

# 36. DNS response elemzése

Nyisd ki a választ.

Keresd:

```text
Answers
```

azon belül:

```text
Name
Type
TTL
Address
```

Most ténylegesen látod:

```text
domain
↓
DNS
↓
IP address
```

folyamatot.

---

# 37. DNS transport

Nézd meg, mi található a DNS alatt.

Gyakran:

```text
User Datagram Protocol
```

Nyisd ki.

Keresd:

```text
Source Port
Destination Port
```

Tipikusan láthatsz:

```text
client ephemeral port
→ DNS server:53
```

---

# 38. LAB 2 kérdések

1. Mi volt a DNS resolver IP-je?
2. UDP vagy TCP?
3. Mi volt a destination port?
4. Mi volt a kliens source portja?
5. Milyen recordot kérdeztünk?
6. Mi volt a válasz IP?
7. Milyen DNS TTL-t látsz?

---

# 39. Fontos felismerés

Most már valódi packetben látod:

```text
DNS
↓
UDP
↓
IP
↓
Ethernet / link layer
```

Tehát ugyanaz a rétegezés:

```text
application protocol
↓
transport
↓
network
↓
link
```

---

# 40. LAB 3 – TCP handshake

Ez az egyik legfontosabb gyakorlat.

Indíts egy új capture-t.

WSL-ben:

```bash
curl https://example.com
```

Állítsd le a capture-t.

Először derítsd ki:

```bash
dig +short example.com
```

milyen IP-re ment a kérés.

---

# 41. TCP filter

Wiresharkban:

```text
tcp.port == 443
```

Ha túl sok packet van, kombináld a szerver IP-jével:

```text
ip.addr == <SERVER-IP> && tcp.port == 443
```

---

# 42. SYN keresése

A packet listában keress TCP packetet:

```text
[SYN]
```

Ez a TCP kapcsolat első lépése.

Mental model:

```text
Client
↓
SYN
↓
Server
```

---

# 43. SYN-ACK

Ezután keresd:

```text
[SYN, ACK]
```

Mental model:

```text
Server
↓
SYN-ACK
↓
Client
```

---

# 44. ACK

Majd:

```text
[ACK]
```

Ezzel teljes:

```text
SYN
↓
SYN-ACK
↓
ACK
```

A:

```text
TCP three-way handshake
```

---

# 45. TCP flags megnyitása

Nyisd ki:

```text
Transmission Control Protocol
```

majd:

```text
Flags
```

A SYN packetben keresd:

```text
SYN = 1
ACK = 0
```

A SYN-ACK packetben:

```text
SYN = 1
ACK = 1
```

A harmadik packetben:

```text
ACK = 1
```

---

# 46. LAB 3 kérdések

1. Mi a kliens source IP-je?
2. Mi a kliens source portja?
3. Mi a szerver destination IP-je?
4. Mi a szerver portja?
5. Mi volt a kliens ephemeral port?
6. Milyen sorrendben érkeztek a TCP flag-ek?

---

# 47. Sequence és ACK number

A TCP részben keresd:

```text
Sequence Number
Acknowledgment Number
```

Nem kell még a teljes TCP számozást matematikailag elemezni.

A lényeg:

```text
sequence number
→ hol tart az adatfolyam?
```

```text
acknowledgment number
→ mit igazolt vissza a másik fél?
```

---

# 48. LAB 4 – TLS

Maradj ugyanabban a HTTPS capture-ben.

Filter:

```text
tls
```

Keress olyan packeteket, mint:

```text
Client Hello
Server Hello
Application Data
```

A pontos lista TLS-verziótól és capture környezettől függhet.

---

# 49. Client Hello

Nyisd meg:

```text
Transport Layer Security
```

és ha látható:

```text
Client Hello
```

Ebben a kliens megkezdi a TLS kapcsolat létrehozását.

Megjelenhetnek például:

```text
TLS version
cipher suites
extensions
```

Nem kell még ezeket részletesen megtanulni.

---

# 50. Server Hello

Keress:

```text
Server Hello
```

Ez a szerver TLS handshake válaszának része.

Mental model:

```text
TCP established
↓
TLS handshake
↓
encrypted connection
```

---

# 51. TLS Application Data

Később:

```text
Application Data
```

packeteket láthatsz.

Ebben már titkosított alkalmazási adat utazik.

Ezért normál HTTPS capture-ben nem feltétlenül látod:

```text
GET /
```

vagy:

```text
password=...
```

szöveget.

Helyette:

```text
TLS Application Data
```

jelenik meg.

---

# 52. LAB 4 kérdések

1. TCP handshake előbb történt vagy TLS?
2. Látsz Client Hello-t?
3. Látsz Server Hello-t?
4. Látható egyszerű szövegként a HTTP request?
5. Miért nem?

---

# 53. A teljes HTTPS sorrend

A capture-ből próbáld meg felismerni:

```text
DNS query
↓
DNS response
↓
TCP SYN
↓
TCP SYN-ACK
↓
TCP ACK
↓
TLS handshake
↓
TLS Application Data
```

Ez az eddig tanult elmélet valódi forgalomban.

---

# 54. LAB 5 – saját HTTP server

Most készítünk egyszerű titkosítatlan HTTP forgalmat, hogy magát a HTTP requestet is láthassuk.

WSL terminál:

```bash
python3 -m http.server 8000
```

Másik WSL terminál:

```bash
curl http://127.0.0.1:8000
```

---

# 55. Fontos: loopback capture

Mivel:

```text
127.0.0.1
```

loopback communication, olyan interface-en kell capture-ölni, amely képes ezt a helyi forgalmat megfigyelni.

Windows + WSL környezet miatt ennek megjelenése eltérhet.

Ha a Windows Wiresharkban nem jelenik meg ez a WSL-en belüli loopback forgalom, ez nem networking-hiba.

Ebben az esetben a labornál használhatunk:

```text
WSL tcpdump
```

vagy később más capture megoldást.

A fő cél most a Wireshark működésének megértése.

---

# 56. Alternatív HTTP capture

Ha a WSL loopback HTTP traffic nem látszik a Windows Wiresharkban, használj külső HTTP oldalt:

```bash
curl http://example.com
```

ha az adott oldal elfogad HTTP kapcsolatot.

Filter:

```text
http
```

---

# 57. HTTP packet

Titkosítatlan HTTP-nél Wireshark képes lehet megmutatni:

```text
GET /
Host: example.com
User-Agent: curl/...
```

Ez nagy különbség a HTTPS-hez képest.

---

# 58. HTTP request kibontása

Nyisd meg:

```text
Hypertext Transfer Protocol
```

Keresd:

```text
Request Method
Request URI
Host
User-Agent
```

---

# 59. HTTP response

Keresd a választ.

Például:

```text
HTTP/1.0 200 OK
```

vagy:

```text
HTTP/1.1 200 OK
```

Keresd:

```text
Status Code
Content-Type
Content-Length
Server
```

---

# 60. LAB 5 kérdések

1. Milyen HTTP method volt?
2. Mi volt a path?
3. Mi volt a Host header?
4. Mi volt a status code?
5. Milyen response header-eket látsz?
6. Miért látható HTTP-nél az adat, HTTPS-nél pedig nem?

---

# 61. LAB 6 – TCP connection refused

Most megfigyeljük, mit jelent packet szinten a:

```text
Connection refused
```

hiba.

Ne fusson semmi egy választott porton.

Például:

```text
9999
```

WSL:

```bash
curl http://127.0.0.1:9999
```

A loopback-capture korlátozás itt is érvényes.

---

# 62. Mit keresünk?

TCP esetén tipikusan valami ehhez hasonló történhet:

```text
SYN
↓
RST
```

vagy:

```text
RST, ACK
```

A célgép ezzel aktívan jelzi:

> ezen a porton nem fogadok kapcsolatot.

---

# 63. TCP Reset filter

Filter:

```text
tcp.flags.reset == 1
```

Ez megmutatja az RST flag-et tartalmazó packeteket.

---

# 64. Connection refused mental model

```text
Client
↓
SYN
↓
Server
↓
RST
↓
Client

Connection refused
```

Ezért mondtuk:

```text
refused
→ kaptunk választ
```

---

# 65. LAB 6 kérdések

1. Küldött a kliens SYN-t?
2. Érkezett SYN-ACK?
3. Érkezett RST?
4. Mit bizonyít az RST?
5. Miért más ez, mint a timeout?

---

# 66. LAB 7 – Timeout gondolkodás

Timeoutot mesterségesen előidézni már környezetfüggőbb, ezért itt nem konfigurálunk veszélyes vagy felesleges firewall szabályokat.

A packet-szintű mental model:

```text
SYN
↓
nincs válasz

SYN retransmission
↓
nincs válasz

SYN retransmission
↓
...
↓
timeout
```

---

# 67. TCP retransmission

Wireshark képes felismerni több TCP elemzési eseményt.

Hasznos filter:

```text
tcp.analysis.retransmission
```

vagy SYN újraküldéseknél a packet listában is megjelenhet elemzési információ.

---

# 68. Timeout vs refused packet-szinten

## Refused

```text
SYN
↓
RST
```

Gyors válasz.

---

## Timeout

```text
SYN
↓
semmi

SYN
↓
semmi

SYN
↓
semmi
```

Majd feladás.

Ez nagyon fontos troubleshooting különbség.

---

# 69. LAB 8 – Follow TCP Stream

Ez a Wireshark egyik legjobb funkciója.

Keress egy titkosítatlan HTTP TCP packetet.

Jobb kattintás:

```text
Follow
→ TCP Stream
```

---

# 70. Mit csinál a Follow TCP Stream?

A TCP kapcsolat packetjeit egyetlen kommunikációs folyamattá rakja össze.

Mental model:

```text
packet 1
packet 2
packet 3
packet 4
↓
Follow TCP Stream
↓
egy összefüggő beszélgetés
```

HTTP esetén láthatod:

```text
request
+
response
```

egy helyen.

---

# 71. Miért fontos?

Korábban tanultuk:

```text
TCP
→ byte stream
```

Az alkalmazási adat több TCP segmentben is utazhat.

A:

```text
Follow TCP Stream
```

segít ezt újra összerakni.

---

# 72. Stream filter

A Wireshark egy TCP connectionhöz rendelhet például:

```text
tcp.stream == 3
```

filtert.

Ezzel csak annak az egy TCP kapcsolatnak a packetjeit látod.

Ez nagyon hasznos zajos capture-nél.

---

# 73. LAB 8 feladat

Válassz egy TCP kapcsolatot.

Használd:

```text
Follow TCP Stream
```

Ezután figyeld meg, hogy a display filter valami ilyesmire változhat:

```text
tcp.stream eq X
```

Feladat:

1. Hány packet tartozik a streamhez?
2. Melyik két endpoint kommunikál?
3. Mik a portok?
4. Látható-e az alkalmazási adat?

---

# 74. LAB 9 – Endpointok

Menü:

```text
Statistics
→ Endpoints
```

Itt a capture-ben szereplő endpointokat tudod megvizsgálni.

Lehetnek külön lapok:

```text
Ethernet
IPv4
IPv6
TCP
UDP
```

---

# 75. Mire jó az Endpoints nézet?

Megmutathatja például:

```text
mely IP-k kommunikáltak?
mennyi packetet küldtek?
mennyi byte-ot küldtek?
```

Ez nagyobb capture elemzésénél hasznos.

---

# 76. LAB 9 feladat

Nyisd meg:

```text
Statistics
→ Endpoints
```

Nézd meg az IPv4 nézetet.

Keresd:

```text
saját / WSL-related IP
DNS resolver
külső szerver
```

Ha felismerhetők.

---

# 77. LAB 10 – Conversations

Menü:

```text
Statistics
→ Conversations
```

Ez már nem egyetlen endpointot, hanem két végpont közötti kommunikációt mutat.

Mental model:

```text
Endpoint A
↔
Endpoint B
```

---

# 78. TCP conversation

Egy TCP conversation például:

```text
172.x.x.x:52341
↔
93.x.x.x:443
```

Ez gyakorlatilag nagyon közel áll a socket kapcsolat mental modelünkhöz.

---

# 79. LAB 10 feladat

Menj:

```text
Statistics
→ Conversations
→ TCP
```

Keress egy HTTPS kapcsolatot.

Azonosítsd:

```text
client IP
client port
server IP
server port
bytes
packets
```

---

# 80. LAB 11 – Packet színek

A Wireshark különböző protokollokat és eseményeket különböző színekkel jeleníthet meg.

A színek célja:

```text
gyorsabb vizuális felismerés
```

Nem kell a színeket bemagolni.

Mindig a:

```text
Protocol
Info
Packet Details
```

információból indulj ki.

---

# 81. Expert Information

A Wireshark képes elemzési jelzéseket adni.

Menü:

```text
Analyze
→ Expert Information
```

Itt lehetnek például:

```text
Notes
Chats
Warnings
Errors
```

és TCP-vel kapcsolatos jelzések.

Fontos:

> A Wireshark „warning” nem feltétlenül jelent valódi alkalmazáshibát.

Ez csak elemzési nyom.

---

# 82. Packet keresése

Használhatod:

```text
Ctrl + F
```

funkciót packetek vagy tartalom keresésére.

De a legtöbb networking troubleshootingnál a:

```text
Display Filter
```

fontosabb.

---

# 83. Capture mentése

A capture elmenthető:

```text
File
→ Save As
```

Tipikus formátum:

```text
.pcapng
```

A `.pcapng` fájl később újra megnyitható és elemezhető.

---

# 84. Mi az a PCAP?

Egyszerű mental model:

```text
PCAP / PCAPNG
→ rögzített hálózati packetek fájlja
```

Ez troubleshootingnál nagyon fontos.

Például egy másik mérnök elküldheti:

```text
problem-capture.pcapng
```

és anélkül elemezheted, hogy ott lennél a hálózaton.

---

# 85. Capture újranyitása

Wireshark:

```text
File
→ Open
```

Válaszd ki:

```text
.pcap
```

vagy:

```text
.pcapng
```

fájlt.

A packeteket ugyanúgy filterezheted és vizsgálhatod.

---

# 86. Hasznos alap filter lista

## ARP

```text
arp
```

## DNS

```text
dns
```

## ICMP

```text
icmp
```

## TCP

```text
tcp
```

## UDP

```text
udp
```

## TLS

```text
tls
```

## HTTP

```text
http
```

---

# 87. IP filterek

Bármely irány:

```text
ip.addr == 8.8.8.8
```

Source:

```text
ip.src == 8.8.8.8
```

Destination:

```text
ip.dst == 8.8.8.8
```

---

# 88. TCP port filterek

Bármely irány:

```text
tcp.port == 443
```

Source:

```text
tcp.srcport == 443
```

Destination:

```text
tcp.dstport == 443
```

---

# 89. UDP port filter

Például DNS:

```text
udp.port == 53
```

---

# 90. TCP SYN filter

SYN packetek:

```text
tcp.flags.syn == 1
```

Csak kezdeti SYN, ACK nélkül:

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

---

# 91. TCP RST filter

```text
tcp.flags.reset == 1
```

---

# 92. TCP retransmission filter

```text
tcp.analysis.retransmission
```

---

# 93. TCP stream filter

```text
tcp.stream == X
```

Például:

```text
tcp.stream == 4
```

---

# 94. Kombinált filter példa

HTTPS egy adott hosttal:

```text
ip.addr == 93.184.216.34 && tcp.port == 443
```

DNS csak UDP-n:

```text
dns && udp
```

TCP reset egy adott hosttal:

```text
ip.addr == 203.0.113.10 && tcp.flags.reset == 1
```

---

# 95. ARP gyakorlat

WSL + Windows virtual networking miatt az ARP, amit Windows Wiresharkban látsz, nem feltétlenül pontosan a WSL belső neighbor table megfelelője.

Ettől függetlenül érdemes filterezni:

```text
arp
```

Keress:

```text
Who has ...?
Tell ...
```

---

# 96. ARP Request

Tipikus:

```text
Who has 192.168.1.1?
Tell 192.168.1.10
```

Mental model:

> Kié ez az IP? Mondja meg nekem a MAC-címét.

---

# 97. ARP Reply

Például:

```text
192.168.1.1 is at aa:bb:cc:dd:ee:ff
```

Mental model:

```text
IP
↓
MAC
```

---

# 98. Broadcast MAC

ARP requestnél gyakran:

```text
ff:ff:ff:ff:ff:ff
```

destination MAC jelenik meg.

Ez:

```text
Ethernet broadcast
```

Vagyis:

> mindenki ezen a helyi linken figyeljen.

---

# 99. ARP kérdések

1. Milyen IP-ről kérdezett?
2. Melyik IP MAC-címét kereste?
3. Mi volt az ARP Request destination MAC-je?
4. Mit válaszolt az ARP Reply?
5. Miért helyi hálózati protokoll az ARP?

---

# 100. Packet Length

A Packet Listben:

```text
Length
```

oszlop mutatja a frame rögzített hosszát.

Nézz meg különböző packeteket:

```text
TCP ACK
DNS query
TLS Application Data
```

Nem minden packet azonos méretű.

---

# 101. Packet Bytes panel

Az alsó panelben hexadecimális byte-ok jelennek meg.

Például:

```text
00 15 5d ...
45 00 ...
...
```

Nem kell hex dumpot fejből olvasni.

A lényeg:

> A középső Packet Details panel ezeket a nyers byte-okat értelmezi protokollmezőkké.

---

# 102. Packet Details és Packet Bytes kapcsolata

Kattints például:

```text
Destination Port
```

mezőre a TCP headerben.

Figyeld meg, hogy a Packet Bytes panelben kiemelődik a hozzá tartozó byte-rész.

Ez nagyon fontos felismerés:

```text
TCP destination port
```

nem valamilyen elméleti adat.

Tényleges byte-ok vannak a packetben, amelyeket Wireshark értelmez.

---

# 103. LAB – header byte megfigyelés

Válassz egy TCP packetet.

Kattints:

```text
Source Port
Destination Port
Sequence Number
Flags
```

mezőkre.

Figyeld az alsó panelben a kijelölést.

Cél:

```text
field
↔
raw bytes
```

kapcsolat megértése.

---

# 104. Teljes packet mental model

Egy HTTPS packet leegyszerűsítve:

```text
Ethernet
│
├── Source MAC
├── Destination MAC
│
└── IPv4
    │
    ├── Source IP
    ├── Destination IP
    ├── TTL
    │
    └── TCP
        │
        ├── Source Port
        ├── Destination Port
        ├── Sequence
        ├── ACK
        ├── Flags
        │
        └── TLS
            │
            └── Encrypted Application Data
```

---

# 105. DevOps troubleshooting példa – DNS

Felhasználó:

> Nem működik a weboldal.

Capture:

```text
DNS query
↓
DNS query
↓
DNS query
```

de:

```text
nincs DNS response
```

Mit gyanítunk?

```text
DNS resolver elérés
network filtering
DNS server
```

---

# 106. DevOps troubleshooting példa – TCP timeout

Capture:

```text
SYN
↓
nincs válasz

SYN retransmission
↓
nincs válasz
```

Gyanú:

```text
firewall DROP
network path
routing
cloud firewall
server unavailable
```

---

# 107. DevOps troubleshooting példa – Connection refused

Capture:

```text
SYN
↓
RST
```

Gyanú:

```text
nincs listening service
rossz port
active reject
```

---

# 108. DevOps troubleshooting példa – TLS

Capture:

```text
TCP handshake OK
↓
TLS handshake
↓
TLS alert / failure
```

Következtetés:

```text
IP connectivity OK
TCP OK
↓
TLS környékén van a hiba
```

---

# 109. DevOps troubleshooting példa – HTTP 500

Capture-ben / alkalmazási tesztben:

```text
HTTP response
↓
500
```

Ez azt jelenti:

```text
network
TCP
HTTP
```

már működik.

Tovább:

```text
application logs
database
dependencies
environment
```

---

# 110. Wireshark workflow troubleshootinghoz

Egy jó általános módszer:

```text
1. Start capture
2. Reproduce problem
3. Stop capture
4. Apply display filter
5. Find relevant flow
6. Inspect packets
7. Follow stream if useful
8. Determine last successful step
```

---

# 111. "Reproduce the problem"

Nagyon fontos DevOps fogalom.

Először:

```text
capture start
```

azután idézd elő a hibát:

```bash
curl ...
```

utána:

```text
capture stop
```

Így könnyebb tudni:

```text
mely packetek tartoznak a problémához
```

---

# 112. Ne capture-ölj órákig feleslegesen

Kezdőként jobb:

```text
start
↓
egy konkrét teszt
↓
stop
```

mint:

```text
30 perc teljes gépforgalom
```

mert az utóbbiban rengeteg zaj lesz.

---

# 113. Biztonság

Packet capture érzékeny információkat tartalmazhat.

Például:

```text
IP addresses
hostnames
DNS queries
unencrypted HTTP
tokens/cookies bizonyos körülmények között
network topology
```

Ezért PCAP fájlt ne ossz meg automatikusan nyilvánosan.

---

# 114. HTTPS sem rejt el mindent

HTTPS titkosítja az alkalmazási tartalom jelentős részét, de packet capture-ben továbbra is látható lehet például:

```text
source IP
destination IP
ports
packet size
timing
TCP behavior
TLS metadata egy része
```

Tehát:

```text
encrypted
≠
láthatatlan
```

---

# 115. Első Wireshark labor összefoglaló

A labor végére tudnod kell:

```text
capture indítása/leállítása
```

```text
interface kiválasztása
```

```text
packet kiválasztása
```

```text
Packet List olvasása
```

```text
Packet Details kibontása
```

```text
Packet Bytes felismerése
```

```text
Display Filter használata
```

```text
IP/port/protokoll szerinti szűrés
```

```text
TCP stream követése
```

```text
PCAP mentése
```

---

# 116. Protokollok, amelyeket fel kell ismerned

```text
ARP
```

```text
ICMP
```

```text
DNS
```

```text
UDP
```

```text
TCP
```

```text
TLS
```

```text
HTTP
```

---

# 117. TCP események, amelyeket fel kell ismerned

```text
SYN
```

```text
SYN-ACK
```

```text
ACK
```

```text
RST
```

```text
FIN
```

```text
Retransmission
```

---

# 118. Fő filterek, amelyeket érdemes megjegyezni

```text
arp
dns
icmp
tcp
udp
tls
http
```

```text
ip.addr == X
```

```text
tcp.port == X
```

```text
tcp.flags.syn == 1
```

```text
tcp.flags.reset == 1
```

```text
tcp.stream == X
```

---

# 119. Gyors Wireshark fogalomtár

## Capture

```text
packetek rögzítése
```

## Packet List

```text
rögzített packetek listája
```

## Packet Details

```text
packet protokollstruktúrája
```

## Packet Bytes

```text
nyers packet byte-ok
```

## Display Filter

```text
már rögzített packetek megjelenítésének szűrése
```

## Capture Filter

```text
capture közben mit rögzítsünk
```

## TCP Stream

```text
egy TCP kapcsolat packetjei
```

## PCAP / PCAPNG

```text
elmentett packet capture
```

## Endpoint

```text
kommunikáció egyik végpontja
```

## Conversation

```text
két endpoint közötti kommunikáció
```

---

# 120. Wireshark + networking mental model

A Wiresharkban:

```text
Ethernet
↓
MAC
```

```text
IP
↓
source/destination host
```

```text
TCP/UDP
↓
ports
```

```text
DNS
↓
name resolution
```

```text
TLS
↓
secure communication
```

```text
HTTP
↓
application request/response
```

---

# 121. Végső komplex feladat

Készíts egy új capture-t.

WSL-ben:

```bash
dig example.com
```

majd:

```bash
curl -v https://example.com
```

Állítsd le a capture-t.

Próbáld meg egyedül megtalálni:

```text
1. DNS query
2. DNS response
3. destination IP
4. TCP SYN
5. TCP SYN-ACK
6. TCP ACK
7. client ephemeral port
8. server 443 port
9. TLS handshake
10. TLS Application Data
```

Ezután válassz egy packetet, és bontsd ki:

```text
Frame
↓
Link layer
↓
IP
↓
TCP
↓
TLS
```

---

# 122. A komplex feladat végső magyarázata

A saját capture-ed alapján mondd el:

> Mi történik attól a pillanattól, hogy a `curl https://example.com` parancsot kiadom addig, amíg megérkezik a válasz?

Használd ezeket a fogalmakat:

```text
DNS
IP
route
TCP
ephemeral port
443
SYN
SYN-ACK
ACK
TLS
HTTP
response
```

Ha ezt a saját capture-ed alapján el tudod magyarázni, akkor az eddigi networking elmélet nagy része már nem csak elmélet.

---

# 123. Következő lépés

A Wireshark labor után:

```text
13-packet-tracer-lab.md
```

következik.

Ott már nem egy meglévő hálózat packetjeit figyeljük, hanem saját virtuális hálózatot építünk:

```text
PC
↓
Switch
↓
Router
↓
másik subnet
↓
Server
```

és gyakoroljuk:

```text
IP addressing
subnetting
gateway
ARP
switching
routing
ping
packet flow
```

A három gyakorlati nézőpont:

```text
WSL networking lab
→ mit lát a Linux host?
```

```text
Wireshark lab
→ mi utazik ténylegesen a hálózaton?
```

```text
Packet Tracer lab
→ hogyan épül fel maga a hálózat?
```