# Firewalls

## 1. Mi az a firewall?

A **firewall**, magyarul tűzfal, hálózati forgalmat szabályoz.

Feladata:

```text
network traffic
↓
firewall rules
↓
ALLOW / DENY / DROP / REJECT
```

A firewall szabályok alapján eldönti, hogy egy packet vagy kapcsolat átmehet-e.

---

# 2. Miért kell firewall?

Tegyük fel, egy Linux szerveren ezek a szolgáltatások futnak:

```text
SSH        :22
HTTP       :80
HTTPS      :443
Node       :3000
PostgreSQL :5432
```

Attól, hogy egy szolgáltatás fut és figyel egy porton, még nem biztos, hogy szeretnénk az internet felől elérhetővé tenni.

Például:

```text
22   SSH        → ALLOW
80   HTTP       → ALLOW
443  HTTPS      → ALLOW

3000 Node       → BLOCK
5432 PostgreSQL → BLOCK
```

Mental model:

```text
Internet
   │
   ├── :22  ─────→ SSH
   ├── :80  ─────→ HTTP
   ├── :443 ─────→ HTTPS
   ├── :3000 ──X
   └── :5432 ──X
```

---

# 3. Listening port ≠ publicly reachable

Ez nagyon fontos.

Ha:

```bash
ss -lntp
```

ezt mutatja:

```text
0.0.0.0:3000 LISTEN
```

akkor van egy process, ami a 3000-es TCP porton figyel.

Ez azonban még nem jelenti automatikusan azt, hogy internetről elérhető.

Lehet például:

```text
Node app
↓
0.0.0.0:3000
↓
Linux firewall
↓
DROP
```

Tehát:

```text
LISTEN
≠
publicly reachable
```

---

# 4. Bind address + port + firewall

A szolgáltatás elérhetőségét több dolog együtt határozza meg.

Például:

```text
127.0.0.1:3000
```

jelentése:

```text
csak a saját gépen érhető el
```

A `127.0.0.1` loopback address.

Mental model:

```text
127.0.0.1
→ saját gép
```

Ha a Node app:

```text
127.0.0.1:3000
```

címen figyel:

```text
Internet ──X──→ Node
```

de ugyanazon a gépen:

```text
Nginx
↓
127.0.0.1:3000
↓
Node
```

működhet.

---

# 5. `0.0.0.0`

Ha egy szolgáltatás:

```text
0.0.0.0:3000
```

címen figyel, az általában azt jelenti:

```text
figyelj minden IPv4 interface-en
```

Ez potenciálisan lehetővé teszi a távoli elérést.

Viszont a firewall még mindig blokkolhatja:

```text
0.0.0.0:3000
↓
firewall
↓
DENY
```

Tehát:

```text
bind address
≠
firewall
```

Két külön dolog.

---

# 6. Egy szolgáltatás elérhetősége

Egy szolgáltatás eléréséhez több feltételnek is teljesülnie kell:

```text
application fut?
↓
megfelelő IP-re bindolt?
↓
megfelelő porton figyel?
↓
routing működik?
↓
firewall engedi?
↓
NAT / port forwarding rendben?
↓
cloud firewall engedi?
```

Ha ezek közül bármelyik hibás:

```text
service unavailable
```

---

# 7. Inbound traffic

**Inbound traffic**:

```text
network
↓
saját gép
```

Valaki a hálózatról a mi gépünk felé küld forgalmat.

Példa:

```text
Internet
↓
server:22
```

SSH kapcsolat.

---

# 8. Outbound traffic

**Outbound traffic**:

```text
saját gép
↓
network
```

A saját gépünk kezdeményez kifelé.

Példa:

```text
Linux server
↓
api.github.com:443
```

---

# 9. Inbound vs outbound

```text
inbound
→ bejövő forgalom
```

```text
outbound
→ kimenő forgalom
```

Egy tipikus szervernél:

```text
incoming
→ szigorúbban korlátozott

outgoing
→ gyakran alapból engedélyezett
```

---

# 10. ALLOW / ACCEPT

```text
ALLOW
```

vagy:

```text
ACCEPT
```

jelentése:

```text
engedjük át
```

---

# 11. DENY

Általános értelemben:

```text
DENY
```

jelentése:

```text
ne engedjük át
```

A pontos működés implementációtól függően lehet például reject vagy drop jellegű.

---

# 12. REJECT

A:

```text
REJECT
```

jelentése:

```text
blokkoljuk
+
visszajelzést küldünk
```

Példa:

```text
Client
↓
connection attempt
↓
Firewall
↓
REJECT
↓
hiba vissza a kliensnek
```

---

# 13. DROP

A:

```text
DROP
```

jelentése:

```text
dobjuk el csendben
```

Példa:

```text
Client
↓
connection attempt
↓
Firewall
X
nincs válasz
```

Ezért egy tiltott kapcsolat sokszor:

```text
timeout
```

formájában jelenik meg.

---

# 14. Firewall rule

Egy firewall rule több feltételt tartalmazhat.

Például:

```text
source IP
destination IP
protocol
source port
destination port
interface
connection state
action
```

---

# 15. Példa firewall rule-ra

```text
source IP:
203.0.113.10

protocol:
TCP

destination port:
22

action:
ALLOW
```

Jelentése:

> erről az IP-ről engedd be a TCP SSH forgalmat.

---

# 16. Port megnyitása mindenkinek

Példa:

```text
source:
anywhere

protocol:
TCP

destination port:
443

action:
ALLOW
```

Jelentése:

```text
HTTPS bárhonnan jöhet
```

---

# 17. Stateful firewall

A modern firewallok jellemzően **statefulak**.

Ez azt jelenti, hogy követik a hálózati kapcsolatok állapotát.

Tegyük fel:

```text
Server:52000
↓
GitHub:443
```

A szerver kezdeményezett kapcsolatot.

A válasz:

```text
GitHub:443
↓
Server:52000
```

technikailag bejövő traffic.

A firewall viszont tudja:

```text
ez egy korábban létrehozott kapcsolat válasza
```

Ezért:

```text
ESTABLISHED
```

kapcsolathoz tartozó packeteket engedheti.

---

# 18. Connection state

Gyakori state-ek:

```text
NEW
ESTABLISHED
RELATED
INVALID
```

Egyszerű mental model:

```text
NEW
→ új kapcsolat
```

```text
ESTABLISHED
→ már létező kapcsolat
```

```text
RELATED
→ meglévő kapcsolathoz kapcsolódó forgalom
```

---

# 19. Tipikus szerveres firewall policy

Egy internetes webszervernél például:

```text
INBOUND

22/tcp
→ SSH
→ ALLOW

80/tcp
→ HTTP
→ ALLOW

443/tcp
→ HTTPS
→ ALLOW

everything else
→ DENY
```

Outbound:

```text
általában engedett
```

Ez csak általános mental model, nem univerzális biztonsági szabály.

---

# 20. Least exposure

Fontos biztonsági alapelv:

> csak azt tegyük elérhetővé, amit valóban el kell érni.

Például:

```text
Internet
↓
PostgreSQL :5432
```

általában nem szükséges.

Jobb:

```text
Internet
↓
Nginx
↓
backend
↓
PostgreSQL
```

A database csak belső hálózaton legyen elérhető.

---

# 21. Linux firewall rétegek

Egyszerűsített modell:

```text
UFW
↓
nftables / iptables
↓
Linux kernel
↓
packet filtering
```

---

# 22. UFW

UFW:

```text
Uncomplicated Firewall
```

Ez egy egyszerűbb kezelőfelület Linux firewall szabályok kezelésére.

Tipikus Ubuntu szervereken gyakran használják.

---

# 23. iptables

Az:

```text
iptables
```

egy régebbi, hosszú ideig nagyon elterjedt Linux firewall-kezelési rendszer.

Segítségével részletes packet-filter szabályokat lehet létrehozni.

Például logikailag:

```text
TCP
+
destination port 22
→ ACCEPT
```

---

# 24. nftables

Az:

```text
nftables
```

a modernebb Linux packet filtering framework.

Sok modern Linux rendszeren ez a jelenlegi háttérrendszer.

Mental model:

```text
UFW
→ egyszerű kezelés

iptables
→ klasszikus / régebbi megoldás

nftables
→ modernebb framework
```

---

# 25. Kernel szerepe

A tényleges packet filtering a Linux kernel hálózati rendszerében történik.

A UFW nem maga „fogja meg” fizikailag a packetet.

A UFW inkább konfigurálja a háttérben használt firewall szabályokat.

---

# 26. UFW státusz

```bash
sudo ufw status
```

Példa:

```text
Status: inactive
```

vagy:

```text
Status: active
```

---

# 27. Default policy

Gyakori konfiguráció:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Jelentése:

```text
incoming
→ alapból tiltott
```

```text
outgoing
→ alapból engedett
```

---

# 28. Default deny gondolkodás

Mental model:

```text
mindent tilts
↓
csak a szükséges dolgokat engedd
```

Például:

```text
default deny incoming

+
22/tcp allow
80/tcp allow
443/tcp allow
```

---

# 29. SSH engedélyezése

```bash
sudo ufw allow 22/tcp
```

vagy:

```bash
sudo ufw allow OpenSSH
```

---

# 30. Fontos SSH figyelmeztetés

Távoli szerveren, mielőtt a firewallt bekapcsoljuk:

```text
először engedjük az SSH-t
```

különben akár kizárhatjuk saját magunkat.

Tehát:

```text
SSH rule
↓
firewall enable
```

---

# 31. HTTP engedélyezése

```bash
sudo ufw allow 80/tcp
```

---

# 32. HTTPS engedélyezése

```bash
sudo ufw allow 443/tcp
```

---

# 33. UFW bekapcsolása

```bash
sudo ufw enable
```

---

# 34. Példa UFW státuszra

```bash
sudo ufw status
```

Példa:

```text
Status: active

To        Action      From
22/tcp    ALLOW       Anywhere
80/tcp    ALLOW       Anywhere
443/tcp   ALLOW       Anywhere
```

---

# 35. Mi történik egy engedélyezett packet esetén?

Tegyük fel:

```text
Internet
↓
server:443
```

Van szabály:

```text
443/tcp ALLOW
```

Folyamat:

```text
packet
↓
firewall
↓
ALLOW
↓
TCP stack
↓
listening socket
↓
Nginx
```

---

# 36. Mi történik tiltott port esetén?

Tegyük fel:

```text
Internet
↓
server:5432
```

Nincs engedélyező szabály.

A default policy:

```text
deny incoming
```

Ezért:

```text
packet
↓
firewall
↓
DENY / DROP
↓
application nem kapja meg
```

---

# 37. Firewall engedi, de nincs process

Tegyük fel:

```text
443/tcp
→ ALLOW
```

de nincs program, ami 443-on figyel.

Ekkor:

```text
packet
↓
firewall
↓
ALLOW
↓
kernel
↓
nincs listening socket
```

Tehát:

```text
firewall OK
```

de:

```text
service NOT running
```

---

# 38. Source IP alapján engedélyezés

Példa:

```bash
sudo ufw allow from 203.0.113.10 to any port 22 proto tcp
```

Jelentése:

```text
203.0.113.10
↓
TCP 22
↓
ALLOW
```

Más IP-ről:

```text
SSH
→ nincs engedélyezve
```

---

# 39. Subnet engedélyezése

Példa:

```bash
sudo ufw allow from 192.168.1.0/24
```

Ez visszakapcsolódik a subnettinghez.

```text
192.168.1.0/24
```

jelentése:

```text
teljes 192.168.1.x subnet
```

---

# 40. Szabályok listázása sorszámmal

```bash
sudo ufw status numbered
```

Példa:

```text
[1] 22/tcp  ALLOW  Anywhere
[2] 80/tcp  ALLOW  Anywhere
[3] 443/tcp ALLOW  Anywhere
```

---

# 41. Szabály törlése

```bash
sudo ufw delete 2
```

Törli a második szabályt.

---

# 42. Rule order

Firewall rendszereknél számíthat a szabályok sorrendje.

Általános mental model:

```text
packet
↓
firewall rules
↓
matching rule
↓
action
```

Egyes szabályrendszerekben az első megfelelő rule dönt.

Ezért fontos:

```text
rule order
```

---

# 43. Miért fontos a rule order?

Például:

```text
1. deny all TCP 22
2. allow TCP 22 from 203.0.113.10
```

bizonyos szabályrendszerekben az első tiltó rule miatt a második már nem érvényesülne.

Ezért a konkrét firewall rendszer szabályértékelési logikáját mindig ismerni kell.

---

# 44. Firewall logging

A firewall képes lehet eseményeket naplózni.

Például:

```text
blocked connection
source IP
destination port
protocol
timestamp
```

Ez troubleshootingnál és security vizsgálatnál nagyon hasznos.

---

# 45. UFW logging

Például:

```bash
sudo ufw logging on
```

A naplók helye rendszer- és konfigurációfüggő lehet.

A lényeg:

```text
firewall logs
→ mit blokkolt?
→ honnan jött?
→ melyik port?
```

---

# 46. Firewall vs bind address

Nagyon fontos különbség.

## Bind address

Megmondja:

```text
melyik helyi IP/interface-en figyel az application?
```

Példa:

```text
127.0.0.1:3000
```

---

## Firewall

Megmondja:

```text
engedjük-e a hálózati forgalmat?
```

Példa:

```text
3000/tcp DENY
```

---

# 47. Példa

Node:

```text
127.0.0.1:3000
```

Firewall:

```text
3000/tcp ALLOW
```

A Node ettől még nem lesz internetről elérhető.

Miért?

Mert:

```text
bind address
→ loopback only
```

---

# 48. Másik példa

Node:

```text
0.0.0.0:3000
```

Firewall:

```text
3000/tcp DENY
```

A Node minden IPv4 interface-en figyel.

De:

```text
firewall
→ blokkolja
```

Ezért kívülről nem elérhető.

---

# 49. Firewall vs NAT

A két fogalom különböző.

## NAT

```text
address / port translation
```

Például:

```text
192.168.1.10:52341
↓
84.x.x.x:41052
```

---

## Firewall

```text
traffic filtering
```

Vagyis:

```text
engedjük?
tiltjuk?
```

Mental model:

```text
NAT
→ átír
```

```text
Firewall
→ szűr
```

---

# 50. NAT és firewall együtt

Egy router egyszerre végezhet:

```text
routing
NAT
firewall
DHCP
```

feladatokat.

Ezért a funkciókat fejben külön kell választani.

---

# 51. Firewall vs routing

Routing:

```text
merre menjen a packet?
```

Firewall:

```text
mehet-e a packet?
```

Nagyon röviden:

```text
routing
→ direction
```

```text
firewall
→ permission
```

---

# 52. Firewall vs port

Port:

```text
melyik szolgáltatás?
```

Firewall:

```text
engedjük-e elérni?
```

Például:

```text
443
→ HTTPS service
```

és:

```text
443/tcp ALLOW
→ firewall engedi
```

---

# 53. Firewall vs socket

Socket:

```text
kommunikációs végpont
```

Firewall:

```text
a hozzá vezető traffic szűrése
```

---

# 54. Firewall és reverse proxy

Tipikus szerver:

```text
Internet
↓
443
↓
firewall
↓
Nginx
↓
127.0.0.1:3000
↓
Node
```

Itt:

```text
443
→ public
```

```text
3000
→ local only
```

A Node portját nem kell publikusan megnyitni.

---

# 55. Miért jó ez?

Mert az internet közvetlenül csak:

```text
Nginx
```

szolgáltatást éri el.

A backend:

```text
Node
```

el van rejtve mögötte.

Ez csökkenti a public attack surface-t.

---

# 56. PostgreSQL példa

Rosszabb kialakítás:

```text
Internet
↓
PostgreSQL :5432
```

Jobb kialakítás:

```text
Internet
↓
Nginx
↓
Node backend
↓
PostgreSQL
```

A database:

```text
internal only
```

---

# 57. Docker és firewall

Docker használatakor további hálózati réteg jelenik meg.

Példa:

```bash
docker run -p 3000:3000 app
```

Mental model:

```text
host:3000
↓
Docker port mapping
↓
container:3000
```

---

# 58. Docker elérhetőség

Egy container elérhetősége függhet:

```text
application bind address
container port
Docker network
port mapping
host firewall
routing
```

---

# 59. Docker belső hálózat

Később például:

```text
Internet
↓
Nginx container
↓
Docker internal network
↓
Node container
↓
Docker internal network
↓
PostgreSQL container
```

A Node és PostgreSQL portját nem feltétlenül kell hoston publikálni.

---

# 60. Cloud firewall

Cloud környezetben lehet külön infrastruktúra-szintű firewall.

Például:

```text
Internet
↓
DigitalOcean Cloud Firewall
↓
Droplet
↓
Linux UFW
↓
Application
```

---

# 61. Két firewall egyszerre

Lehetséges:

```text
Cloud firewall
→ ALLOW 443
```

de:

```text
UFW
→ DENY 443
```

Eredmény:

```text
HTTPS nem működik
```

Vagy fordítva:

```text
Cloud firewall
→ DENY 443
```

```text
UFW
→ ALLOW 443
```

Eredmény:

```text
HTTPS továbbra sem működik
```

Minden releváns rétegnek engednie kell.

---

# 62. Teljes public request útja

Példa:

```text
Internet
↓
cloud firewall
↓
public IP
↓
routing
↓
Linux firewall
↓
network interface
↓
TCP stack
↓
listening socket
↓
Nginx
↓
backend
```

Ha a kérés nem ér célba:

```text
valahol ezen az útvonalon megállt
```

---

# 63. Firewall troubleshooting

Fontos kérdések:

```text
fut az application?
```

```text
milyen addressre bindolt?
```

```text
milyen porton figyel?
```

```text
Linux firewall engedi?
```

```text
cloud firewall engedi?
```

```text
routing rendben van?
```

```text
Docker port mapping rendben van?
```

---

# 64. Hasznos parancsok

Listening socket:

```bash
ss -lntp
```

UFW:

```bash
sudo ufw status
```

Routing:

```bash
ip route
```

IP address:

```bash
ip addr
```

HTTP/HTTPS teszt:

```bash
curl -v https://example.com
```

---

# 65. Rövid szakmai angol

> A firewall filters network traffic.

> Incoming traffic is denied by default.

> Outgoing traffic is allowed.

> The server is listening on port 443.

> The firewall allows HTTPS traffic.

> The firewall blocks port 5432.

> The application is bound to localhost.

> The service is not publicly reachable.

> The firewall is stateful.

> Established connections are allowed.

> The cloud firewall blocks the connection.

> The packet is dropped by the firewall.

---

# 66. Amit biztosan érteni kell

```text
firewall
→ hálózati forgalom szűrése
```

```text
inbound
→ bejövő traffic
```

```text
outbound
→ kimenő traffic
```

```text
ALLOW
→ enged
```

```text
DROP
→ csendben eldob
```

```text
REJECT
→ tilt + visszajelez
```

```text
stateful
→ kapcsolat állapotát követi
```

```text
bind address
→ hol figyel az application
```

```text
port
→ melyik szolgáltatás
```

```text
firewall
→ elérhető-e azon az útvonalon
```

---

# 67. Gyors fogalomtár

Ez a rész gyors ismétlésre szolgál.

## Firewall

```text
forgalomszűrő
→ enged vagy tilt
```

---

## Firewall rule

```text
feltétel + action
```

Például:

```text
TCP + port 443 → ALLOW
```

---

## Inbound

```text
hálózat → saját gép
```

---

## Outbound

```text
saját gép → hálózat
```

---

## ALLOW / ACCEPT

```text
engedd át
```

---

## DENY

```text
tiltsd
```

---

## DROP

```text
dobd el
ne válaszolj
```

---

## REJECT

```text
tiltsd
és jelezd vissza
```

---

## Stateful firewall

```text
követi a kapcsolatok állapotát
```

---

## NEW

```text
új kapcsolat
```

---

## ESTABLISHED

```text
már létrejött kapcsolat
```

---

## Bind address

```text
melyik helyi IP-n figyel az alkalmazás?
```

---

## `127.0.0.1`

```text
loopback
→ csak saját gép
```

---

## `0.0.0.0`

```text
minden IPv4 interface-en figyel
```

---

## Loopback

```text
a gép saját magával kommunikál
```

---

## Private network

```text
belső IP hálózat
pl. 192.168.x.x
```

Nem ugyanaz, mint a loopback.

---

## Port

```text
melyik szolgáltatás?
```

---

## Listening port

```text
egy process kapcsolatot vár az adott porton
```

---

## Socket

```text
OS által kezelt kommunikációs végpont
```

---

## UFW

```text
Uncomplicated Firewall
→ egyszerű Linux firewall kezelés
```

---

## iptables

```text
klasszikus Linux firewall rule rendszer
```

---

## nftables

```text
modernebb Linux packet filtering framework
```

---

## Default policy

```text
mi történjen akkor,
ha nincs külön megfelelő szabály?
```

Példa:

```text
deny incoming
```

---

## Default deny

```text
alapból tilts
csak szükséges dolgokat engedj
```

---

## Source IP

```text
honnan jön a packet?
```

---

## Destination IP

```text
melyik gép a cél?
```

---

## Source port

```text
küldő oldali port
```

---

## Destination port

```text
cél szolgáltatás portja
```

---

## Rule order

```text
a firewall szabályok sorrendje
```

Bizonyos rendszereknél az első megfelelő rule dönt.

---

## Firewall logging

```text
tűzfal események naplózása
```

Hasznos:

```text
blocked source IP
blocked port
protocol
timestamp
```

---

## NAT

```text
IP / port átírás
```

Nem ugyanaz, mint a firewall.

---

## Routing

```text
merre menjen a packet?
```

Nem ugyanaz, mint a firewall.

---

## Gateway

```text
következő router,
amin keresztül más hálózatot érünk el
```

---

## Reverse proxy

```text
kérést fogad
és továbbít backend szolgáltatásnak
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

## Cloud firewall

```text
cloud infrastruktúra hálózati szűrője
```

Példa:

```text
DigitalOcean Cloud Firewall
```

---

## Docker port mapping

```text
host port
↓
container port
```

Például:

```text
host:3000
↓
container:3000
```

---

# 68. Fogalmak, amiket könnyű összekeverni

## Loopback vs private network

```text
loopback
→ csak saját gép

private network
→ több belső gép hálózata
```

---

## Bind address vs firewall

```text
bind address
→ hol figyel az application?

firewall
→ engedjük-e oda a trafficet?
```

---

## Port vs firewall

```text
port
→ melyik szolgáltatás?

firewall
→ elérhető-e?
```

---

## Routing vs firewall

```text
routing
→ merre?

firewall
→ szabad-e?
```

---

## NAT vs firewall

```text
NAT
→ átír

firewall
→ szűr
```

---

## Listening vs reachable

```text
LISTEN
→ process vár kapcsolatot

REACHABLE
→ a hálózatról ténylegesen el is érhető
```

A kettő nem ugyanaz.

---

# 69. Egyetlen nagy mental model

```text
Internet
↓
cloud firewall
↓
routing
↓
Linux firewall
↓
network interface
↓
IP
↓
TCP
↓
port
↓
socket
↓
Nginx
↓
127.0.0.1:3000
↓
Node
↓
internal network
↓
database
```

Ha valami nem működik, mindig ezt kérdezzük:

> Meddig jutott el a kérés?

Ez lesz a következő fejezet, a hálózati troubleshooting alapja.