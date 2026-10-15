# Linux networking parancsok

## 1. Miről szól ez a fejezet?

Ebben a fejezetben azt nézzük meg, hogyan tudjuk Linux alatt megvizsgálni a hálózat tényleges állapotát.

Az eddigi elméleti fogalmak itt válnak kézzelfoghatóvá:

```text
interface
IP address
subnet
MAC address
default gateway
routing table
ARP / neighbor table
port
socket
TCP connection
DNS
HTTP
network path
```

A fő kérdések:

```text
Milyen hálózati interfészeim vannak?

Milyen IP-címem van?

Melyik subnetben vagyok?

Mi a default gateway?

Milyen route-okat ismer a gép?

Milyen MAC-címeket ismer?

Milyen portokon figyelnek szolgáltatások?

Milyen TCP kapcsolatok aktívak?

Elérhető-e egy másik gép?

Merre halad hozzá a forgalom?

Működik-e a DNS?

Működik-e a HTTP/HTTPS kapcsolat?
```

A legfontosabb parancsok:

```bash
ip link
ip addr
ip route
ip neigh

ss

ping
traceroute

dig
nslookup

curl

hostname
```

Később:

```bash
tcpdump
```

---

# 2. Linux network interface

A **network interface** az a hálózati interfész, amelyen keresztül a Linux hálózati adatokat tud küldeni és fogadni.

Lehet fizikai:

```text
Ethernet adapter
Wi-Fi adapter
```

de lehet virtuális is:

```text
loopback
Docker interface
bridge
VPN interface
WSL virtual interface
```

Példák interface-nevekre:

```text
lo
eth0
enp0s3
wlan0
docker0
br-...
```

A pontos név rendszerenként eltérhet.

Mental model:

```text
Linux
↓
network interface
↓
hálózat
```

---

# 3. Loopback interface

A speciális:

```text
lo
```

interface a **loopback interface**.

Tipikus IPv4 címe:

```text
127.0.0.1
```

Ez mindig a saját gépre mutat.

Mental model:

```text
127.0.0.1
→ this computer
```

Például:

```text
127.0.0.1:3000
```

azt jelenti, hogy egy szolgáltatás a saját gép loopback címén, a 3000-es porton érhető el.

Ha egy alkalmazás csak:

```text
127.0.0.1
```

címen figyel, akkor általában nem érhető el közvetlenül más gépekről.

Ez később Nginx, Docker és szerverüzemeltetés során nagyon fontos lesz.

---

# 4. `ip link`

A:

```bash
ip link
```

parancs a hálózati interfészek link-szintű információit mutatja.

Példa:

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536
    link/loopback 00:00:00:00:00:00

2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    link/ether 00:15:5d:12:34:56
```

---

# 5. Interface neve

Például:

```text
2: eth0
```

Az interface neve:

```text
eth0
```

Egy másik Linux rendszeren például lehet:

```text
enp0s3
```

vagy:

```text
ens33
```

---

# 6. Interface állapot

Példa:

```text
<UP,LOWER_UP>
```

## UP

```text
UP
```

Az interface adminisztratívan engedélyezve van.

## LOWER_UP

```text
LOWER_UP
```

Az alsóbb hálózati kapcsolat is működőképes.

Egyszerű mental model:

```text
UP
+
LOWER_UP
↓
interface használható
```

Ha:

```text
DOWN
```

állapotot látunk, akkor az interface nincs aktív állapotban.

---

# 7. MTU

Példa:

```text
mtu 1500
```

MTU:

```text
Maximum Transmission Unit
```

Ez meghatározza, hogy az adott hálózati linken mekkora IP packet szállítható tipikusan fragmentáció nélkül.

Ethernet hálózatokon gyakori:

```text
1500 bytes
```

Mental model:

```text
MTU
→ egy linken továbbítható packet maximális tipikus mérete
```

---

# 8. MAC address

Példa:

```text
link/ether 00:15:5d:12:34:56
```

Ez az interface MAC-címe.

```text
eth0
↓
MAC
↓
00:15:5d:12:34:56
```

A MAC-cím a helyi hálózati kommunikációnál fontos.

Kapcsolódó témák:

```text
Ethernet
ARP
local network
next hop
```

---

# 9. `ip addr`

A:

```bash
ip addr
```

parancs megmutatja az interface-ekhez rendelt IP-címeket.

Rövid forma:

```bash
ip a
```

Példa:

```text
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    link/ether 00:15:5d:12:34:56
    inet 172.24.100.15/20 brd 172.24.111.255 scope global eth0
    inet6 fe80::215:5dff:fe12:3456/64 scope link
```

---

# 10. IPv4 address

A:

```text
inet
```

IPv4 címet jelöl.

Példa:

```text
inet 172.24.100.15/20
```

Ebben:

```text
172.24.100.15
→ host IP address
```

és:

```text
/20
→ subnet prefix
```

Tehát ugyanaz a CIDR jelölés jelenik meg, amit subnettingnél tanultunk.

---

# 11. Prefix és subnet mask

Például:

```text
/20
```

ugyanazt a hálózati maszkot fejezi ki, mint:

```text
255.255.240.0
```

Linux networking parancsokban gyakran CIDR jelöléssel találkozunk.

Példák:

```text
/24
/20
/16
/8
```

---

# 12. Broadcast address

Példa:

```text
brd 172.24.111.255
```

A:

```text
brd
```

jelenti:

```text
broadcast
```

Ez az adott IPv4 subnet broadcast címe.

---

# 13. `scope global`

Példa:

```text
scope global
```

Fontos:

```text
global
≠
public internet IP
```

A `scope global` Linux szempontból az address hatókörét jelenti.

Egy private IPv4 cím is megjelenhet:

```text
scope global
```

értékkel.

---

# 14. IPv6 address

Példa:

```text
inet6 fe80::215:5dff:fe12:3456/64 scope link
```

Az:

```text
inet6
```

IPv6 címet jelöl.

A:

```text
fe80::
```

tartomány tipikusan:

```text
IPv6 link-local
```

address.

A:

```text
scope link
```

azt jelenti, hogy az address az adott linken használható.

---

# 15. `ip link` és `ip addr` különbsége

## `ip link`

Főleg:

```text
interface
MAC
MTU
interface state
```

## `ip addr`

Főleg:

```text
IPv4 address
IPv6 address
CIDR prefix
broadcast address
```

Mental model:

```text
ip link
→ milyen hálózati interfészeim vannak?
```

```text
ip addr
→ milyen IP-címek vannak rajtuk?
```

---

# 16. Első troubleshooting lépések

Ha nincs hálózat:

```text
1. Van interface?
2. UP állapotban van?
3. Van IP-címe?
4. Van megfelelő subnet?
```

Parancsok:

```bash
ip link
ip addr
```

Például:

```text
eth0 DOWN
```

már problémára utalhat.

Ha nincs:

```text
inet ...
```

sor, akkor lehet, hogy nincs IPv4 cím konfigurálva.

---

# 17. `ip route`

A következő alapvető parancs:

```bash
ip route
```

Röviden:

```bash
ip r
```

Ez megmutatja a Linux routing table-jét.

Példa:

```text
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.10
```

---

# 18. Közvetlenül elérhető subnet

Példa:

```text
192.168.1.0/24 dev eth0
```

Jelentése:

```text
192.168.1.0/24
→ ez a hálózat
```

```text
dev eth0
→ ezen az interface-en közvetlenül elérhető
```

Mental model:

```text
192.168.1.0/24
↓
directly connected
↓
eth0
```

Nem kell hozzá router.

---

# 19. Default route

Példa:

```text
default via 192.168.1.1 dev eth0
```

Ez nagyjából ugyanazt jelenti, mint:

```text
0.0.0.0/0 via 192.168.1.1
```

Jelentése:

```text
ha nincs specifikusabb route
↓
küldd ide
↓
192.168.1.1
```

Ez tipikusan a:

```text
default gateway
```

---

# 20. `via`

Példa:

```text
via 192.168.1.1
```

A:

```text
via
```

megadja a következő routert:

```text
next hop
```

Tehát:

```text
via 192.168.1.1
→ küldd a következő routerhez
```

---

# 21. `dev`

Példa:

```text
dev eth0
```

Azt jelenti:

```text
melyik interface-en küldjük ki?
```

Tehát:

```text
destination
↓
route
↓
next hop
↓
interface
```

---

# 22. Routing table mental model

```text
destination IP
↓
routing table lookup
↓
best matching route
↓
next hop
↓
network interface
```

Például:

```text
destination = 8.8.8.8
```

Nincs hozzá specifikus route.

Ezért:

```text
default route
↓
192.168.1.1
↓
eth0
```

---

# 23. Specifikus route előnye

Ha van:

```text
10.0.0.0/8 via 192.168.1.5
```

és:

```text
default via 192.168.1.1
```

akkor:

```text
10.20.30.40
```

cél esetén a specifikusabb:

```text
10.0.0.0/8
```

route nyer.

Ez kapcsolódik a:

```text
longest prefix match
```

fogalmához.

---

# 24. `ip route get`

Nagyon hasznos parancs:

```bash
ip route get 8.8.8.8
```

Ezzel megkérdezhetjük a Linuxot:

> pontosan milyen route-ot használnál ehhez az IP-hez?

Példa:

```text
8.8.8.8 via 192.168.1.1 dev eth0 src 192.168.1.10
```

Ez azt mutatja:

```text
destination
→ 8.8.8.8

gateway
→ 192.168.1.1

interface
→ eth0

source IP
→ 192.168.1.10
```

Ez troubleshooting során nagyon hasznos.

---

# 25. `ip neigh`

Parancs:

```bash
ip neigh
```

Ez mutatja a Linux:

```text
neighbor table
```

tábláját.

IPv4 esetén ez nagyrészt ahhoz kapcsolódik, amit korábban:

```text
ARP cache
```

néven tanultunk.

Példa:

```text
192.168.1.1 dev eth0 lladdr aa:bb:cc:dd:ee:ff REACHABLE
```

---

# 26. Neighbor table értelmezése

```text
192.168.1.1
→ IP address
```

```text
dev eth0
→ interface
```

```text
lladdr aa:bb:cc:dd:ee:ff
→ MAC address
```

```text
REACHABLE
→ neighbor jelenleg elérhetőnek tekinthető
```

Mental model:

```text
local IP
↓
MAC address
```

---

# 27. ARP és routing együtt

Ha távoli géphez kommunikálunk:

```text
destination:
8.8.8.8
```

a gép nem:

```text
8.8.8.8 MAC-jét
```

keresi.

Hanem először routing:

```text
8.8.8.8
↓
default gateway
↓
192.168.1.1
```

majd neighbor/ARP:

```text
192.168.1.1
↓
MAC address
```

Tehát:

```text
Routing
→ kinek adjam tovább?
```

```text
ARP / neighbor
→ mi annak a helyi MAC-címe?
```

---

# 28. `ss`

A:

```bash
ss
```

jelentése:

```text
socket statistics
```

Segítségével megvizsgálhatjuk:

```text
listening socketek
aktív TCP kapcsolatok
UDP socketek
portok
processzek
```

---

# 29. Listening portok

Nagyon hasznos:

```bash
ss -lnt
```

Tipikus jelentések:

```text
-l
→ listening
```

```text
-n
→ ne oldja fel a címeket/neveket
```

```text
-t
→ TCP
```

Példa:

```text
LISTEN 0 511 0.0.0.0:80
LISTEN 0 511 127.0.0.1:3000
```

---

# 30. `0.0.0.0` listening address

Ha egy program:

```text
0.0.0.0:80
```

címen figyel, az általában azt jelenti:

```text
minden IPv4 interface-en figyel
```

Ezzel szemben:

```text
127.0.0.1:3000
```

azt jelenti:

```text
csak local loopback
```

Ez szerverüzemeltetésben rendkívül fontos különbség.

---

# 31. Példa Node alkalmazásra

Ha:

```text
127.0.0.1:3000
```

címen fut:

```text
Internet
↓
nem közvetlenül elérhető
```

de Nginx ugyanazon a szerveren elérheti:

```text
Nginx
↓
127.0.0.1:3000
↓
Node
```

Ez egy tipikus későbbi szerverarchitektúra lesz.

---

# 32. Processzek megjelenítése

Hasznos:

```bash
sudo ss -lntp
```

A:

```text
-p
```

megpróbálja megmutatni a sockethez tartozó processt.

Például:

```text
users:(("node",pid=1234,fd=20))
```

Így láthatjuk:

```text
3000 port
↓
node process
```

---

# 33. Aktív TCP kapcsolatok

Például:

```bash
ss -nt
```

Megmutathatja az aktív TCP kapcsolatokat.

Példa:

```text
ESTAB 0 0 192.168.1.10:52341 142.250.x.x:443
```

Értelmezése:

```text
local:
192.168.1.10:52341

remote:
142.250.x.x:443
```

Ez pontosan az a socketes modell, amit korábban tanultunk.

---

# 34. TCP connection state-ek

Gyakori állapotok:

```text
LISTEN
ESTAB
TIME-WAIT
SYN-SENT
SYN-RECV
CLOSE-WAIT
```

A legfontosabb most:

```text
LISTEN
→ szerver vár kapcsolatot
```

```text
ESTAB
→ established TCP connection
```

---

# 35. `ping`

Parancs:

```bash
ping 8.8.8.8
```

vagy:

```bash
ping example.com
```

A `ping` ICMP-t használ.

Segítségével ellenőrizhető például:

```text
elérhető-e a cél
mekkora a válaszidő
van-e packet loss
```

---

# 36. Ping példa

```text
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=12.3 ms
```

Ebben:

```text
icmp_seq=1
→ packet sorszáma
```

```text
ttl=117
→ visszaérkező IP packet TTL értéke
```

```text
time=12.3 ms
→ round-trip time
```

---

# 37. Ping nem bizonyít mindent

Fontos:

```text
ping működik
```

nem jelenti automatikusan:

```text
webserver is működik
```

Például:

```text
ping OK
HTTPS FAIL
```

lehetséges.

Ugyanígy:

```text
ping FAIL
```

nem feltétlenül jelenti, hogy a szerver nem működik.

Lehet, hogy:

```text
ICMP tiltva van
```

de:

```text
TCP 443
```

működik.

---

# 38. `traceroute`

A:

```bash
traceroute example.com
```

megpróbálja megmutatni, milyen hálózati hopokon keresztül halad a forgalom.

Példa:

```text
1  192.168.1.1
2  10.x.x.x
3  ISP-router
4  ...
5  destination
```

Mental model:

```text
PC
↓
router
↓
ISP router
↓
router
↓
router
↓
destination
```

---

# 39. Hogyan működik a traceroute?

A traceroute kihasználja az IP:

```text
TTL
```

mezőjét.

TTL:

```text
Time To Live
```

Minden router csökkenti:

```text
TTL = TTL - 1
```

Ha eléri a nullát, a router eldobja a packetet és tipikusan ICMP választ küld.

A traceroute fokozatosan próbál:

```text
TTL 1
TTL 2
TTL 3
...
```

értékeket.

Így lehet feltérképezni a hopokat.

---

# 40. `dig`

A:

```bash
dig
```

DNS lekérdezések vizsgálatára használható.

Példa:

```bash
dig example.com
```

Vagy:

```bash
dig example.com A
```

---

# 41. Rövid DNS válasz

```bash
dig +short example.com
```

Példa:

```text
93.184.216.34
```

Ez gyorsan megmutatja:

```text
domain
→ IP
```

---

# 42. Más DNS resolver használata

Például:

```bash
dig @8.8.8.8 example.com
```

A:

```text
@8.8.8.8
```

azt jelenti:

```text
ezt a DNS servert kérdezd
```

Ez troubleshooting során hasznos lehet.

---

# 43. DNS record típusok

Példák:

```bash
dig example.com A
```

```bash
dig example.com AAAA
```

```bash
dig example.com MX
```

```bash
dig example.com NS
```

```bash
dig example.com TXT
```

---

# 44. `nslookup`

Egyszerűbb DNS teszt:

```bash
nslookup example.com
```

Más resolver:

```bash
nslookup example.com 8.8.8.8
```

A `dig` általában részletesebb és Linuxos troubleshootinghoz kényelmesebb, de mindkettő hasznos.

---

# 45. `curl`

A:

```bash
curl
```

HTTP/HTTPS kérések küldésére használható.

Ez az egyik legfontosabb DevOps troubleshooting eszköz.

Példa:

```bash
curl https://example.com
```

A szerver response body-ját adhatja vissza.

---

# 46. HTTP headers megjelenítése

```bash
curl -I https://example.com
```

Például:

```text
HTTP/2 200
content-type: text/html
server: nginx
```

Így gyorsan ellenőrizhető:

```text
működik-e HTTP
milyen status code érkezik
milyen response headers vannak
```

---

# 47. Verbose curl

Nagyon hasznos:

```bash
curl -v https://example.com
```

A:

```text
-v
```

jelenti:

```text
verbose
```

Ezzel láthatunk többek között:

```text
DNS / connection information
target IP
target port
TLS handshake részletek
certificate információk
HTTP request headers
HTTP response headers
```

Ez gyakorlatilag összeköti az előző fejezet nagy részét egy valódi paranccsal.

---

# 48. HTTP POST curl-lel

Példa:

```bash
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Tamas"}'
```

Itt:

```text
-X POST
→ HTTP method
```

```text
-H
→ header
```

```text
-d
→ request data / body
```

Ez nagyon hasonlít arra, amit Postmanben grafikus felületen csinálunk.

---

# 49. Postman és curl kapcsolata

Postmanben például megadjuk:

```text
method
URL
headers
body
```

Curl-lel ugyanezt parancssorból tehetjük meg.

Mental model:

```text
Postman
→ GUI HTTP client
```

```text
curl
→ CLI HTTP client
```

---

# 50. `hostname`

Egyszerű:

```bash
hostname
```

megmutatja a gép hostname-jét.

Például:

```text
ubuntu-server
```

A hostname a gép logikai neve.

---

# 51. IP-cím megjelenítése hostname paranccsal

Bizonyos rendszereken:

```bash
hostname -I
```

megmutathatja a géphez tartozó IP-címeket.

Példa:

```text
192.168.1.10
```

Ez gyors ellenőrzéshez hasznos, bár részletes hálózati vizsgálathoz:

```bash
ip addr
```

jobb.

---

# 52. `tcpdump`

A:

```bash
tcpdump
```

egy parancssoros packet capture eszköz.

Segítségével tényleges hálózati packeteket lehet megfigyelni.

Például:

```bash
sudo tcpdump -i eth0
```

Jelentése:

```text
capture packets
↓
eth0 interface
```

---

# 53. TCP port szűrése tcpdump-pal

Példa:

```bash
sudo tcpdump -i eth0 tcp port 443
```

Ezzel HTTPS-hez tartozó TCP forgalmat figyelhetünk meg.

Például láthatjuk:

```text
source IP
destination IP
source port
destination port
TCP flags
```

---

# 54. DNS forgalom figyelése

Példa:

```bash
sudo tcpdump -i eth0 port 53
```

Ez segíthet megfigyelni a DNS kommunikációt.

---

# 55. tcpdump és Wireshark

Mental model:

```text
tcpdump
→ CLI packet analyzer / capture
```

```text
Wireshark
→ graphical packet analyzer
```

Mindkettővel ugyanazt a hálózati világot vizsgáljuk, csak más felületen.

---

# 56. Régebbi parancsok

Régebbi Linux rendszereken gyakran találkozhatunk:

```bash
ifconfig
route
arp
netstat
```

parancsokkal.

Modern Linux rendszereken ezek helyett általában:

```text
ifconfig
→ ip addr / ip link

route
→ ip route

arp
→ ip neigh

netstat
→ ss
```

használata ajánlott.

---

# 57. `lsof`

A:

```bash
lsof
```

eredetileg:

```text
list open files
```

de Linuxon a socketek is file descriptorokon keresztül kezelhetők.

Ezért például:

```bash
sudo lsof -i :3000
```

segíthet megkeresni:

```text
melyik process használja a 3000-es portot?
```

Ez később troubleshooting során hasznos lehet.

---

# 58. Egy egyszerű Linux networking troubleshooting sorrend

Tegyük fel:

> Nem érem el a weboldalt.

Nem érdemes rögtön találgatni.

Menjünk rétegenként.

---

## 1. Van network interface?

```bash
ip link
```

Kérdés:

```text
UP?
```

---

## 2. Van IP address?

```bash
ip addr
```

Kérdés:

```text
van inet cím?
```

---

## 3. Van route?

```bash
ip route
```

Kérdés:

```text
van default route?
```

---

## 4. Elérhető a gateway?

```bash
ping <gateway-ip>
```

Például:

```bash
ping 192.168.1.1
```

---

## 5. Elérhető külső IP?

```bash
ping 8.8.8.8
```

Ha ez működik, de:

```bash
ping example.com
```

nem, akkor DNS-probléma is felmerülhet.

---

# 59. DNS troubleshooting

```bash
dig example.com
```

vagy:

```bash
nslookup example.com
```

Kérdés:

```text
domain
→ IP
```

működik-e?

---

# 60. Routing troubleshooting

```bash
ip route
```

és:

```bash
ip route get 8.8.8.8
```

Kérdés:

```text
milyen útvonalat választana a kernel?
```

---

# 61. Network path troubleshooting

```bash
traceroute example.com
```

Kérdés:

```text
meddig jut el a forgalom?
```

---

# 62. Port / socket troubleshooting

Ha saját szervert vizsgálunk:

```bash
ss -lntp
```

Kérdés:

```text
figyel valami a megfelelő porton?
```

Például:

```text
:80
:443
:3000
:5432
```

---

# 63. HTTP troubleshooting

```bash
curl -v https://example.com
```

Kérdések:

```text
létrejön TCP kapcsolat?
```

```text
működik TLS?
```

```text
milyen HTTP status code jön?
```

---

# 64. Egy jó troubleshooting mental model

```text
Interface
↓
IP
↓
Subnet
↓
Route
↓
Gateway
↓
DNS
↓
Remote IP
↓
Port
↓
TCP
↓
TLS
↓
HTTP
↓
Application
```

Nem feltétlenül kell mindig minden lépést lefuttatni.

A lényeg:

> próbáljuk meghatározni, hogy a kommunikáció melyik ponton romlik el.

---

# 65. Példa: domain nem működik

Tegyük fel:

```bash
curl https://example.com
```

nem működik.

Először:

```bash
dig example.com
```

Ha nincs DNS response:

```text
DNS probléma
```

Ha van IP:

```text
DNS valószínűleg működik
↓
vizsgáljuk tovább
```

---

# 66. Példa: DNS működik, szerver nem

```bash
dig example.com
```

működik.

De:

```bash
curl https://example.com
```

nem.

Ekkor lehet például:

```text
routing
firewall
TCP port
TLS
server
```

probléma.

Ezért a DNS siker önmagában nem jelenti, hogy a szolgáltatás működik.

---

# 67. Példa: port nem figyel

Saját szerveren:

```bash
ss -lntp
```

Nem látjuk:

```text
:3000
```

portot.

Akkor valószínűleg:

```text
application nem fut
```

vagy:

```text
más porton fut
```

---

# 68. Példa: csak localhoston figyel

```text
LISTEN ... 127.0.0.1:3000
```

Ez azt jelenti:

```text
local access
→ igen
```

```text
remote direct access
→ nem
```

Ha ezt látjuk:

```text
0.0.0.0:3000
```

akkor az alkalmazás minden IPv4 interface-en figyel.

Ez azonban még nem garantálja a külső elérhetőséget, mert:

```text
firewall
routing
NAT
cloud firewall
```

is beleszólhat.

---

# 69. Példa: teljes webes diagnosztika

```text
1.
ip link
```

Van működő interface?

```text
2.
ip addr
```

Van IP?

```text
3.
ip route
```

Van default gateway?

```text
4.
ping gateway
```

Elérhető a helyi router?

```text
5.
ping external-IP
```

Van külső hálózati kapcsolat?

```text
6.
dig domain
```

Működik DNS?

```text
7.
curl -v https://domain
```

Működik TCP/TLS/HTTP?

---

# 70. Fontos kapcsolat az előző fejezetekkel

## `ip addr`

Kapcsolódik:

```text
IP addressing
subnetting
IPv4
IPv6
```

## `ip route`

Kapcsolódik:

```text
routing
gateway
next hop
default route
```

## `ip neigh`

Kapcsolódik:

```text
ARP
MAC
local network
```

## `ss`

Kapcsolódik:

```text
TCP
UDP
ports
sockets
```

## `dig`

Kapcsolódik:

```text
DNS
```

## `curl`

Kapcsolódik:

```text
HTTP
HTTPS
TLS
TCP
```

## `traceroute`

Kapcsolódik:

```text
routing
hop
TTL
```

## `tcpdump`

Kapcsolódik gyakorlatilag az összes hálózati témához.

---

# 71. Legfontosabb parancsok összefoglalva

```bash
ip link
```

```text
network interfaces
```

---

```bash
ip addr
```

```text
IP addresses
```

---

```bash
ip route
```

```text
routing table
default gateway
```

---

```bash
ip neigh
```

```text
IP → MAC neighbor information
```

---

```bash
ss -lntp
```

```text
listening TCP ports + processes
```

---

```bash
ss -nt
```

```text
active TCP connections
```

---

```bash
ping
```

```text
basic IP reachability
latency
packet loss
```

---

```bash
traceroute
```

```text
network hops
```

---

```bash
dig
```

```text
DNS lookup
```

---

```bash
curl
```

```text
HTTP / HTTPS testing
```

---

```bash
tcpdump
```

```text
packet capture
```

---

# 72. Rövid szakmai angol

> The network interface has an IP address.

> The interface is up.

> The server has a default route.

> The default gateway is reachable.

> Linux uses the routing table to select the next hop.

> The neighbor table maps local IP addresses to MAC addresses.

> The application is listening on port 3000.

> The TCP connection is established.

> DNS resolves the hostname to an IP address.

> Curl sends an HTTP request.

> Traceroute shows the network path.

> Tcpdump captures network packets.

---

# 73. Amit ebből biztosan érteni kell

Nem kell minden kapcsolót megjegyezni.

A fontos mental model:

```text
ip link
→ interface
```

```text
ip addr
→ IP
```

```text
ip route
→ merre?
```

```text
ip neigh
→ local IP ↔ MAC
```

```text
ss
→ socket / port / TCP
```

```text
ping
→ alap elérhetőség
```

```text
traceroute
→ milyen hopokon halad?
```

```text
dig
→ DNS
```

```text
curl
→ HTTP / HTTPS
```

```text
tcpdump
→ valódi packetek
```

A legfontosabb összkép:

```text
interface
↓
IP
↓
routing
↓
neighbor
↓
connection
↓
DNS
↓
HTTP
↓
packet inspection
```

---

# 74. Későbbi Linux/networking labor

A laborban érdemes lesz valódi WSL/Linux rendszeren végrehajtani:

```bash
ip link
ip addr
ip route
ip neigh
```

majd az eredményből azonosítani:

```text
interface
MAC
IPv4
IPv6
subnet
broadcast
default gateway
```

Ezután:

```bash
ss -lntp
ss -nt
```

és felismerni:

```text
listening socket
local IP
local port
remote IP
remote port
TCP state
```

Majd:

```bash
ping
traceroute
dig
curl -v
```

parancsokkal egy teljes hálózati kapcsolatot végigvizsgálni.

Végül:

```bash
tcpdump
```

és később:

```text
Wireshark
```

segítségével tényleges packetekben is megkeresni:

```text
Ethernet
IP
TCP
UDP
DNS
TLS
HTTP
```

információkat.

A cél nem a parancsok bemagolása.

A cél:

> ha egy szolgáltatás nem működik, tudd lépésenként megvizsgálni, hogy a hálózati kommunikáció melyik pontján van a probléma.