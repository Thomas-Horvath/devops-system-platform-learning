# Networking Lab – WSL

## 1. A labor célja

Ebben a laborban az eddigi networking elméletet saját WSL környezetben vizsgáljuk meg.

A cél nem a parancsok bemagolása.

A cél:

> Lásd a saját gépeden, hogyan kapcsolódik össze az interface, IP, subnet, gateway, routing, DNS, TCP, port, socket, HTTP és firewall.

A labor során többek között ezt fogjuk megvizsgálni:

```text
WSL
↓
network interface
↓
IP address
↓
subnet
↓
routing table
↓
default gateway
↓
neighbor table
↓
DNS
↓
remote host
↓
TCP connection
↓
ports / sockets
↓
HTTP / HTTPS
↓
basic troubleshooting
```

---

# 2. Környezet

A labor:

```text
Windows 11
↓
WSL2
↓
Ubuntu
```

környezetben készül.

Fontos:

WSL2 virtuális hálózatot használhat, ezért az itt látott IP-címek és gateway-ek nem feltétlenül ugyanazok, mint a Windows vagy az otthoni router címei.

Ez nem hiba.

A cél pont az, hogy megértsük, mit lát **maga a Linux kernel a WSL-en belül**.

---

# 3. Szabályok

A labor során először próbáld saját magad értelmezni a kimenetet.

Ne csak futtasd a parancsokat.

Minden feladatnál válaszolj röviden:

```text
Mit látok?
Mit jelent?
Miért fontos?
```

Ha elakadsz, kérj segítséget.

---

# 4. Előkészítés

Nyiss egy WSL Ubuntu terminált.

Ellenőrizd:

```bash
uname -a
```

majd:

```bash
hostname
```

és:

```bash
pwd
```

Feladat:

1. Írd le a WSL hostname-jét.
2. Írd le, milyen kernel fut.
3. Gondold végig: ez a hostname ugyanaz-e, mint a Windows géped neve?

---

# 5. Network interface-ek

Futtasd:

```bash
ip link
```

Feladat:

1. Hány interface-et látsz?
2. Mi a nevük?
3. Melyik a loopback?
4. Melyik tűnik a tényleges hálózati interface-nek?
5. Melyik interface van `UP` állapotban?
6. Látsz `LOWER_UP` állapotot?
7. Mekkora az MTU?
8. Mi a MAC address?

Keresd például:

```text
lo
eth0
UP
LOWER_UP
mtu
link/ether
```

---

# 6. Loopback interface

Vizsgáld meg külön:

```bash
ip addr show lo
```

Feladat:

1. Mi az IPv4 loopback address?
2. Mi az IPv6 loopback address?
3. Mekkora prefix tartozik hozzájuk?
4. Mire használjuk a loopback interface-et?

Próbáld:

```bash
ping -c 4 127.0.0.1
```

majd:

```bash
ping -c 4 localhost
```

Kérdés:

Miért működhet mindkettő internetkapcsolat nélkül is?

---

# 7. Saját IPv4 address

Futtasd:

```bash
ip addr
```

vagy:

```bash
ip a
```

Keresd meg a fő hálózati interface-edet.

Feladat:

Írd ki:

```text
Interface:
IPv4:
CIDR prefix:
Broadcast:
MAC:
```

Példa formátum:

```text
Interface: eth0
IPv4: ...
CIDR: /...
Broadcast: ...
MAC: ...
```

Ne másold a példát; a saját értékeidet használd.

---

# 8. Saját subnet meghatározása

A saját:

```text
IP + CIDR
```

alapján számold ki:

```text
network address
broadcast address
usable host range
```

Először próbáld manuálisan.

Utána ellenőrizhetjük eszközzel.

Feladat:

```text
IP:
CIDR:
Network:
Broadcast:
Usable range:
```

Kérdés:

A WSL IP-d private vagy public address?

---

# 9. IPv6 vizsgálata

Az `ip addr` kimenetben keress:

```text
inet6
```

Feladat:

1. Van IPv6 címed?
2. Látsz `fe80::` kezdetű címet?
3. Mit jelent ez?
4. Mi a `scope link`?

Ne kelljen hexadecimális subnetet számolni.

A cél csak az IPv6 address típus felismerése.

---

# 10. Routing table

Futtasd:

```bash
ip route
```

Feladat:

Keresd meg:

```text
default route
local subnet route
gateway
interface
source address
```

Írd le:

```text
Default gateway:
Outgoing interface:
Directly connected network:
Source IP:
```

---

# 11. Route értelmezése

Válassz ki egy külső IP-t:

```text
8.8.8.8
```

Futtasd:

```bash
ip route get 8.8.8.8
```

Feladat:

Azonosítsd:

```text
destination
gateway
interface
source IP
```

Majd magyarázd el egy mondatban:

> Ha a WSL packetet küld a 8.8.8.8 címre, merre küldi először?

---

# 12. Same subnet vagy different subnet?

Hasonlítsd össze:

```text
saját WSL IP
default gateway IP
8.8.8.8
```

Feladat:

Döntsd el:

```text
Gateway → same subnet / different subnet?
8.8.8.8 → same subnet / different subnet?
```

Indokold a CIDR alapján.

---

# 13. Gateway elérhetőség

Pingeld meg a gateway-t:

```bash
ping -c 4 <gateway-ip>
```

Feladat:

1. Érkezik válasz?
2. Mennyi az RTT?
3. Van packet loss?
4. Mit bizonyít ez?
5. Mit NEM bizonyít?

---

# 14. Neighbor table

Futtasd:

```bash
ip neigh
```

Ha kevés adat van, előtte pingeld meg a gateway-t.

Majd újra:

```bash
ip neigh
```

Feladat:

Keresd meg a gateway-t.

Írd ki:

```text
IP:
MAC:
Interface:
State:
```

Kérdés:

Melyik korábbi networking fogalomhoz kapcsolódik ez?

---

# 15. ARP / neighbor logika

Saját szavaiddal válaszolj:

Ha a WSL:

```text
8.8.8.8
```

felé küld packetet, melyik MAC addressre van közvetlenül szüksége?

A:

```text
8.8.8.8
```

MAC addressére?

Vagy:

```text
default gateway
```

MAC addressére?

Indokold.

---

# 16. Külső IP elérése

Futtasd:

```bash
ping -c 4 8.8.8.8
```

Feladat:

Ha működik, mit tudunk már biztosan vagy nagy valószínűséggel?

Gondold végig:

```text
interface
IP
route
gateway
internet IP connectivity
DNS
```

Melyiket bizonyítja?

Melyiket nem?

---

# 17. DNS teszt

Próbáld:

```bash
dig example.com
```

Ha nincs `dig`:

```bash
sudo apt update
sudo apt install dnsutils
```

Ezután ismét:

```bash
dig example.com
```

Feladat:

Keresd meg:

```text
QUESTION SECTION
ANSWER SECTION
A record
TTL
DNS server
```

---

# 18. Rövid DNS válasz

Futtasd:

```bash
dig +short example.com
```

Feladat:

Mit ad vissza?

Mi a különbség a:

```bash
dig example.com
```

és:

```bash
dig +short example.com
```

között?

---

# 19. Más DNS resolver használata

Futtasd:

```bash
dig @8.8.8.8 example.com
```

majd:

```bash
dig @1.1.1.1 example.com
```

Feladat:

1. Mindkettő válaszol?
2. Ugyanazt az IP-t kapod?
3. Ha nem, feltétlenül hiba?
4. Mit jelent a `@8.8.8.8`?

---

# 20. DNS record típusok

Próbáld:

```bash
dig example.com A
```

```bash
dig example.com AAAA
```

```bash
dig example.com NS
```

Feladat:

Magyarázd el röviden:

```text
A
AAAA
NS
```

---

# 21. DNS vs IP elérés

Teszt:

```bash
ping -c 4 8.8.8.8
```

majd:

```bash
ping -c 4 example.com
```

Feladat:

Gondold végig ezt a hibát:

```text
8.8.8.8 ping works
example.com does not resolve
```

Melyik rendszer hibájára gyanakodnál?

---

# 22. Traceroute

Ha nincs telepítve:

```bash
sudo apt install traceroute
```

Futtasd:

```bash
traceroute 8.8.8.8
```

majd:

```bash
traceroute example.com
```

Feladat:

1. Hány hop látható?
2. Mi az első hop?
3. Látsz `* * *` sort?
4. Ez biztosan hibát jelent?
5. Hogyan kapcsolódik a traceroute a TTL-hez?

---

# 23. Aktív socketek

Futtasd:

```bash
ss
```

Majd:

```bash
ss -nt
```

Feladat:

Keress legalább egy TCP kapcsolatot.

Azonosítsd:

```text
State:
Local address:
Local port:
Remote address:
Remote port:
```

---

# 24. Listening socketek

Futtasd:

```bash
ss -lnt
```

majd:

```bash
sudo ss -lntp
```

Feladat:

Keress legalább egy listening socketet.

Írd ki:

```text
Address:
Port:
Process:
```

Ha nincs érdekes listening TCP socket, később készítünk sajátot.

---

# 25. `127.0.0.1` vs `0.0.0.0`

A `ss` kimenetben keress:

```text
127.0.0.1:...
```

és esetleg:

```text
0.0.0.0:...
```

Feladat:

Magyarázd el:

```text
127.0.0.1
→ ?

0.0.0.0
→ ?
```

Miért fontos ez egy szervernél?

---

# 26. Saját egyszerű listening service

Ha van Python telepítve:

```bash
python3 --version
```

Indíts egy egyszerű HTTP servert:

```bash
python3 -m http.server 8000
```

Ne zárd be ezt a terminált.

Nyiss egy másik WSL terminált.

---

# 27. Ellenőrizd a listening socketet

Másik terminálban:

```bash
ss -lntp
```

Keresd:

```text
:8000
```

Feladat:

1. Milyen addressen figyel?
2. Milyen process tartozik hozzá?
3. TCP vagy UDP?
4. LISTEN állapotban van?

---

# 28. Local HTTP teszt

Próbáld:

```bash
curl http://127.0.0.1:8000
```

majd:

```bash
curl http://localhost:8000
```

Feladat:

Mindkettő működik?

Miért?

---

# 29. HTTP headers vizsgálata

Futtasd:

```bash
curl -I http://127.0.0.1:8000
```

Feladat:

Keresd:

```text
HTTP status
Server
Content-Type
Content-Length
```

---

# 30. Verbose curl

Futtasd:

```bash
curl -v http://127.0.0.1:8000
```

Feladat:

Keresd meg:

```text
connection target
destination port
HTTP request
HTTP response
headers
```

Próbáld meg különválasztani:

```text
TCP connection information
```

és:

```text
HTTP information
```

---

# 31. Connection refused létrehozása

Állítsd le a Python servert:

```text
Ctrl + C
```

Majd:

```bash
curl -v http://127.0.0.1:8000
```

Feladat:

Milyen hibát kapsz?

Miért?

Mit tudunk ebből a:

```text
host
port
service
```

állapotáról?

---

# 32. Indítsd újra a servert

```bash
python3 -m http.server 8000
```

Majd ellenőrizd:

```bash
ss -lntp
```

és:

```bash
curl http://127.0.0.1:8000
```

Cél:

Lásd közvetlenül a kapcsolatot:

```text
process
↓
listening socket
↓
port
↓
curl
```

---

# 33. Aktív kapcsolat megfigyelése

Nyiss egy új terminált.

Futtasd:

```bash
watch -n 0.5 'ss -nt'
```

Másik terminálból küldj több requestet:

```bash
curl http://127.0.0.1:8000
```

Figyeld meg, hogy megjelenik-e rövid időre TCP kapcsolat.

Feladat:

Keresd:

```text
local port
remote port
connection state
```

---

# 34. Ephemeral port felismerése

A curl kliens nem szintén a `8000`-es portot használja kliensoldalon.

Valami ilyesmi történik:

```text
client:ephemeral-port
↓
server:8000
```

Feladat:

Próbáld megfigyelni a kliens ephemeral portját.

Magyarázd el:

Miért lehet ugyanazt a server portot sok kliens egyszerre használni?

---

# 35. Külső HTTPS kérés

Futtasd:

```bash
curl -v https://example.com
```

Feladat:

Próbáld azonosítani a kimenetben:

```text
DNS resolution
destination IP
destination port
TCP connection
TLS
certificate
HTTP request
HTTP response
status code
```

---

# 36. HTTP vs HTTPS

Futtasd:

```bash
curl -v http://example.com
```

majd:

```bash
curl -v https://example.com
```

Feladat:

Hasonlítsd össze.

Keresd:

```text
port 80
port 443
TLS handshake
certificate
HTTP response
```

---

# 37. Response headers

Futtasd:

```bash
curl -I https://example.com
```

Feladat:

Azonosítsd:

```text
status code
content-type
content-length
server-related headers
cache-related headers
```

Nem biztos, hogy minden felsorolt header jelen lesz.

---

# 38. HTTP redirect vizsgálata

Próbálj olyan oldalt, ami HTTP-ről HTTPS-re redirectel.

Például egy ismert domainnel:

```bash
curl -I http://github.com
```

Feladat:

Keresd:

```text
3xx status
Location header
```

Mit jelent a `Location`?

---

# 39. Redirect követése

Próbáld:

```bash
curl -IL http://github.com
```

A:

```text
-L
```

segítségével a curl követi a redirecteket.

Feladat:

Hány HTTP response történik?

Milyen status code-ok jelennek meg?

---

# 40. DNS → HTTPS teljes flow

Most válassz egy domaint:

```text
example.com
```

és próbáld végigkövetni manuálisan.

1.

```bash
dig +short example.com
```

2.

```bash
ip route get <kapott-ip>
```

3.

```bash
ip neigh
```

4.

```bash
curl -v https://example.com
```

Feladat:

Készíts saját folyamatábrát:

```text
domain
↓
DNS result
↓
route
↓
gateway
↓
TCP
↓
TLS
↓
HTTP
```

A saját kimeneted adataival.

---

# 41. Saját gép teljes network snapshot

Gyűjtsd össze egy helyre:

```bash
ip link
```

```bash
ip addr
```

```bash
ip route
```

```bash
ip neigh
```

```bash
ss -lntp
```

Feladat:

Ezek alapján írd le a WSL hálózati állapotát saját szavaiddal.

Maximum 10 mondat.

---

# 42. Troubleshooting Scenario 1 – DNS

Tegyük fel:

```text
ping 8.8.8.8
→ works
```

de:

```text
dig example.com
→ fails
```

Feladat:

1. Melyik réteg működik?
2. Hol keresnéd a hibát?
3. Melyik parancsot futtatnád következőnek?
4. Mit tesztelnél más DNS resolverrel?

---

# 43. Troubleshooting Scenario 2 – No route

Tegyük fel:

```text
ip addr
→ van IP
```

de:

```text
ip route
```

nem tartalmaz default route-ot.

Feladat:

Mi működhet még?

```text
same subnet communication?
internet communication?
```

Miért?

---

# 44. Troubleshooting Scenario 3 – Connection refused

Tegyük fel:

```bash
curl http://127.0.0.1:8000
```

eredménye:

```text
Connection refused
```

Feladat:

Mit néznél meg?

Használd:

```bash
ss -lntp
```

Mit keresel?

---

# 45. Troubleshooting Scenario 4 – Timeout

Tegyük fel:

```text
DNS resolves correctly
```

de:

```text
curl
→ Connection timed out
```

Feladat:

Sorolj fel legalább négy lehetséges területet:

```text
?
?
?
?
```

---

# 46. Troubleshooting Scenario 5 – Loopback

Tegyük fel:

```text
127.0.0.1:3000 LISTEN
```

Lokálisan működik.

Másik gépről nem.

Feladat:

1. Mi a probléma valószínű neve?
2. Mit jelent a `127.0.0.1`?
3. Milyen bind address kellhet távoli eléréshez?
4. Milyen más biztonsági réteget kell még ellenőrizni?

---

# 47. Troubleshooting Scenario 6 – HTTP 404

```text
curl
→ HTTP 404
```

Feladat:

Mely rétegekről tudjuk, hogy már működnek?

Gondold végig:

```text
DNS?
routing?
TCP?
TLS?
HTTP?
```

Hol keresnéd tovább?

---

# 48. Troubleshooting Scenario 7 – HTTP 500

```text
HTTP 500
```

Feladat:

Elsőként vizsgálnád-e:

```text
ip addr?
```

Miért?

Hol keresnéd inkább?

---

# 49. Troubleshooting Scenario 8 – HTTP 502

Architektúra:

```text
Client
↓
Nginx
↓
Node :3000
```

Kliens:

```text
502 Bad Gateway
```

Feladat:

Mely komponensről tudjuk, hogy elérhető?

Mit tesztelnél szerveroldalon?

Gondolj:

```bash
ss -lntp
```

és:

```bash
curl http://127.0.0.1:3000
```

---

# 50. Troubleshooting Scenario 9 – TLS

```text
certificate verify failed
```

Feladat:

Mely részek jutottak már valószínűleg sikeresen végig?

Mit vizsgálnál?

```text
domain?
expiry?
CA?
certificate chain?
system clock?
```

---

# 51. Firewall megfigyelés

Nézd meg:

```bash
sudo ufw status
```

WSL-ben előfordulhat, hogy:

```text
inactive
```

vagy a firewall használata eltér a normál Linux szerverétől.

Ez rendben van.

A cél most:

```text
parancs ismerete
+
output értelmezése
```

---

# 52. Fontos WSL sajátosság

WSL2 nem ugyanaz a hálózati környezet, mint egy önálló Ubuntu szerver.

A forgalom útjában lehet:

```text
WSL Linux
↓
virtual network
↓
Windows host
↓
Windows network stack
↓
physical network
```

Ezért a későbbi DigitalOcean szerveren bizonyos részletek eltérhetnek.

Viszont az alapfogalmak ugyanazok:

```text
IP
route
DNS
TCP
port
socket
HTTP
```

---

# 53. WSL és Windows összehasonlítás

WSL-ben:

```bash
ip addr
```

Windowsban:

```powershell
ipconfig
```

Feladat:

Hasonlítsd össze:

```text
WSL IPv4
Windows IPv4
```

Ugyanaz?

Ha nem, miért lehet?

Ne próbáld még részletesen megoldani; csak jegyezd fel a különbséget.

---

# 54. Extra feladat – localhost Windows és WSL között

Ha van idő:

Indíts WSL-ben:

```bash
python3 -m http.server 8000
```

Windows böngészőben próbáld:

```text
http://localhost:8000
```

Feladat:

Működik?

Ha igen:

Mit mond ez a WSL és Windows közötti localhost forwardingról?

Ha nem:

Ne kezdj konfigurálni, csak jegyezd fel a tapasztalatot.

---

# 55. Extra feladat – hostname feloldás

Futtasd:

```bash
getent hosts localhost
```

majd:

```bash
getent hosts example.com
```

Feladat:

Mit csinál a:

```text
getent hosts
```

parancs?

Miben különbözik gondolkodásban a `dig`-től?

---

# 56. Extra feladat – `/etc/hosts`

Nézd meg:

```bash
cat /etc/hosts
```

Feladat:

Keresd meg:

```text
127.0.0.1 localhost
```

Kérdés:

Mi történhet névfeloldáskor DNS lekérdezés nélkül?

---

# 57. Extra feladat – DNS konfiguráció

Nézd meg:

```bash
cat /etc/resolv.conf
```

Feladat:

Keresd:

```text
nameserver
```

Milyen DNS resolver van konfigurálva?

WSL esetén ez lehet virtuális / Windows által biztosított cím.

---

# 58. Extra feladat – TCP kapcsolat létrehozása

Egy terminálban:

```bash
python3 -m http.server 8000
```

Másikban:

```bash
curl http://127.0.0.1:8000
```

Közben:

```bash
ss -nt
```

Próbáld meg elkapni:

```text
ESTAB
TIME-WAIT
```

állapotokat.

---

# 59. Extra feladat – process és port

Futtasd:

```bash
sudo ss -lntp
```

Keresd:

```text
python3
```

Feladat:

Kapcsold össze:

```text
process
↓
socket
↓
IP
↓
port
```

---

# 60. Extra feladat – port foglaltság

Miközben fut:

```bash
python3 -m http.server 8000
```

egy másik terminálban próbáld:

```bash
python3 -m http.server 8000
```

Feladat:

Milyen hibát kapsz?

Miért?

Mit tanít ez a:

```text
IP + port
```

és a listening socket kapcsolatáról?

---

# 61. Extra feladat – másik port

Indíts:

```bash
python3 -m http.server 8001
```

miközben a másik server 8000-en fut.

Feladat:

Működhet egyszerre?

Miért?

---

# 62. Extra feladat – két listening socket

Ellenőrizd:

```bash
ss -lntp
```

Keresd:

```text
:8000
:8001
```

Feladat:

Melyik process melyik porthoz tartozik?

---

# 63. Mini troubleshooting challenge

Indíts:

```bash
python3 -m http.server 8000
```

Ezután egy másik terminálban ellenőrizd:

```bash
curl http://127.0.0.1:8000
```

Majd állítsd le a servert.

Ezután ugyanaz:

```bash
curl http://127.0.0.1:8000
```

Feladat:

Írd le a két állapot közötti különbséget:

```text
Server running:
Server stopped:
```

és magyarázd el networking fogalmakkal.

---

# 64. Mini troubleshooting challenge – rossz port

Server:

```text
127.0.0.1:8000
```

Próbáld:

```bash
curl http://127.0.0.1:9999
```

Feladat:

Mi történik?

Miért?

---

# 65. Mini troubleshooting challenge – rossz hostname

Próbálj egy biztosan nem létező domainhez csatlakozni.

Például:

```bash
curl https://this-domain-should-not-exist.invalid
```

Feladat:

Milyen hibaüzenetet kapsz?

Melyik réteg bukott el?

---

# 66. Mini troubleshooting challenge – HTTP vs application

Működő Python servernél:

```bash
curl -I http://127.0.0.1:8000/nincs-ilyen-file
```

Feladat:

Milyen HTTP status code érkezik?

Mit bizonyít ez a hálózatról?

---

# 67. Lab összefoglaló feladat

A labor végén próbáld saját szavaiddal elmagyarázni ezt:

```text
Amikor WSL-ből megnyitok egy HTTPS weboldalt...
```

Használd ezeket a fogalmakat:

```text
interface
IP
subnet
default gateway
DNS
route
TCP
ephemeral port
443
TLS
HTTP
response
```

Nem kell tankönyvi definíció.

A cél, hogy a folyamatot értsd.

---

# 68. Kötelező eredmények

A labor végén tudnod kell a saját WSL-edből megmondani:

```text
milyen interface-ek vannak
```

```text
mi a saját IP
```

```text
milyen subnetben vagy
```

```text
mi a default gateway
```

```text
mi a default route
```

```text
mi a gateway MAC-címe
```

```text
milyen DNS resolver működik
```

```text
milyen socketek figyelnek
```

```text
milyen aktív TCP kapcsolatok vannak
```

```text
hogyan tesztelsz HTTP/HTTPS kapcsolatot
```

```text
hogyan különbözteted meg a DNS, TCP és HTTP hibákat
```

---

# 69. Parancslista – segítség, nem megoldás

```bash
uname -a
hostname

ip link
ip addr
ip route
ip route get <IP>
ip neigh

ping
traceroute

dig
dig +short
nslookup
getent hosts

ss
ss -nt
ss -lnt
sudo ss -lntp

curl
curl -v
curl -I
curl -IL

sudo ufw status

cat /etc/hosts
cat /etc/resolv.conf

python3 -m http.server 8000
```

---

# 70. A labor fő mental modelje

```text
Linux process
↓
socket
↓
port
↓
TCP
↓
IP
↓
route
↓
interface
↓
gateway
↓
network
```

Webes kérésnél:

```text
domain
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
application
```

Troubleshootingnál:

```text
Mi az utolsó működő pont?
↓
Mi a következő komponens?
↓
Ott keresd a hibát.
```

---

# 71. A labor után

A következő külön gyakorlati fejezetek:

```text
12-wireshark-lab.md
```

Ebben valódi packeteket fogunk megfigyelni:

```text
ARP
DNS
TCP handshake
TLS
HTTP
```

Majd:

```text
13-packet-tracer-lab.md
```

Ebben virtuális hálózatot építünk:

```text
PC
switch
router
subnet
gateway
routing
```

A három labor együtt:

```text
WSL
→ valódi Linux hálózat
```

```text
Wireshark
→ valódi packetek
```

```text
Packet Tracer
→ hálózati topológia és routing
```

Ezután jöhet a Docker, ahol ugyanezek a fogalmak újra megjelennek container hálózatokkal.