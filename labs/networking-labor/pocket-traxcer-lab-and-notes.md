# Packet Tracer Lab – Kis hálózat építése, adatküldés és troubleshooting

## 1. A labor célja

Ebben a laborban egyszerre tanuljuk meg:

- a Cisco Packet Tracer alapvető használatát;
- egy kis hálózat felépítését;
- IP-címek és subnetek konfigurálását;
- switch és router szerepét;
- default gateway működését;
- ARP működését;
- routing működését;
- ping és ICMP használatát;
- packetek útjának megfigyelését Simulation módban;
- egyszerű hálózati hibák szándékos előidézését és javítását.

A cél nem Cisco parancsok bemagolása.

A fő cél:

> Lásd vizuálisan, hogyan jut el egy packet egyik hosttól a másikig.

---

# 2. Használt Packet Tracer verzió

A labor ehhez a verzióhoz készül:

```text
Cisco Packet Tracer
8.2.0.0162
```

A felület fő elemei:

```text
Logical Workspace
Realtime
Simulation
Device-Type Selection
Device-Specific Selection
Connections
Add Simple PDU
```

A pontos ikonok helye minimálisan eltérhet képernyőmérettől és elrendezéstől függően.

---

# 3. Mit építünk?

Két külön LAN-t építünk.

```text
LAN A                         LAN B

PC-A                         PC-C
  \                           /
   \                         /
    Switch-A             Switch-B
       |                     |
       |                     |
       └────── Router ───────┘
       |                     |
     PC-B                  Server
```

Logikailag:

```text
192.168.10.0/24
        ↓
      Router
        ↓
192.168.20.0/24
```

Ez már elég összetett ahhoz, hogy legyen:

```text
same-subnet communication
different-subnet communication
ARP
switching
default gateway
routing
ICMP
server access
troubleshooting
```

---

# 4. Addressing plan

## LAN A

Network:

```text
192.168.10.0/24
```

Router interface:

```text
192.168.10.1
```

PC-A:

```text
192.168.10.10
```

PC-B:

```text
192.168.10.20
```

---

## LAN B

Network:

```text
192.168.20.0/24
```

Router interface:

```text
192.168.20.1
```

PC-C:

```text
192.168.20.10
```

Server:

```text
192.168.20.100
```

---

# 5. Addressing table

| Device | Interface | IP address | Mask | Default gateway |
|---|---|---|---|---|
| Router | LAN-A interface | 192.168.10.1 | 255.255.255.0 | - |
| Router | LAN-B interface | 192.168.20.1 | 255.255.255.0 | - |
| PC-A | Ethernet | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC-B | Ethernet | 192.168.10.20 | 255.255.255.0 | 192.168.10.1 |
| PC-C | Ethernet | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |
| Server | Ethernet | 192.168.20.100 | 255.255.255.0 | 192.168.20.1 |

---

# 6. Packet Tracer fő munkaterülete

Indítsd el a Packet Tracert.

Alapesetben:

```text
Logical
```

workspace-ben dolgozz.

Ez az a nézet, ahol az eszközöket elhelyezzük és összekötjük.

A Physical nézet most nem szükséges.

---

# 7. Eszközválasztó

A képernyő alsó részén található a device selection terület.

Fő kategóriák például:

```text
Network Devices
End Devices
Components
Connections
```

---

# 8. Network Devices

Itt találhatók például:

```text
Routers
Switches
Wireless Devices
```

---

# 9. End Devices

Itt találhatók például:

```text
PC
Laptop
Server
```

---

# 10. Connections

A villám / kábel jellegű kategória alatt találhatók a hálózati kábelek.

Kezdőként használható:

```text
Automatically Choose Connection Type
```

is.

Ez automatikusan megfelelő kapcsolatot választ.

Később megtanuljuk külön a kábelválasztást is.

---

# 11. Első feladat – helyezd el az eszközöket

Helyezz el:

```text
1 router
2 switch
3 PC
1 server
```

Javasolt elrendezés:

```text
PC-A       PC-C
  |          |
Switch-A   Switch-B
   \        /
     Router
   /        \
PC-B       Server
```

Valójában a router legyen középen.

---

# 12. Eszközök átnevezése

A könnyebb átláthatóság érdekében nevezd el:

```text
Router1
Switch-A
Switch-B
PC-A
PC-B
PC-C
Server
```

A név módosítható az eszköz konfigurációjában vagy a workspace labeljén.

---

# 13. Milyen routert válassz?

A laborhoz olyan router kell, amelyen legalább két Ethernet/GigabitEthernet interface van.

Például megfelelő lehet egy Packet Tracerben elérhető ISR router.

A lényeg:

```text
Interface 1
→ LAN A

Interface 2
→ LAN B
```

Nem a router modell bemagolása a cél.

---

# 14. Switchek

Két Layer 2 switch kell.

Például egy általános Cisco switch modell.

A szerepük:

```text
Switch-A
→ LAN A hostjai

Switch-B
→ LAN B hostjai
```

---

# 15. Kábelezés

Kapcsold össze:

```text
PC-A
→ Switch-A

PC-B
→ Switch-A

Switch-A
→ Router
```

Másik oldal:

```text
PC-C
→ Switch-B

Server
→ Switch-B

Switch-B
→ Router
```

Használhatod az automatikus cable selectiont.

---

# 16. Link állapot

A kapcsolat mellett kis jelzések jelenhetnek meg.

Ha minden rendben:

```text
green
```

állapotot szeretnénk látni.

Router interface esetén előfordulhat, hogy kezdetben nincs aktív állapotban.

Ez normális.

---

# 17. PC konfigurálása

Kattints:

```text
PC-A
```

Majd:

```text
Desktop
→ IP Configuration
```

Itt beállítható:

```text
IP Address
Subnet Mask
Default Gateway
```

---

# 18. PC-A konfiguráció

```text
IP:
192.168.10.10

Subnet Mask:
255.255.255.0

Default Gateway:
192.168.10.1
```

---

# 19. PC-B konfiguráció

```text
IP:
192.168.10.20

Subnet Mask:
255.255.255.0

Default Gateway:
192.168.10.1
```

---

# 20. PC-C konfiguráció

```text
IP:
192.168.20.10

Subnet Mask:
255.255.255.0

Default Gateway:
192.168.20.1
```

---

# 21. Server konfiguráció

Server:

```text
Desktop
→ IP Configuration
```

Állítsd:

```text
IP:
192.168.20.100

Subnet Mask:
255.255.255.0

Default Gateway:
192.168.20.1
```

---

# 22. Ellenőrzés

Mielőtt továbbmennél, nézd át:

```text
PC-A → 192.168.10.10
PC-B → 192.168.10.20

PC-C → 192.168.20.10
Server → 192.168.20.100
```

Gateway-ek:

```text
LAN A → 192.168.10.1
LAN B → 192.168.20.1
```

---

# 23. Router konfigurálása

Kattints a routerre.

Menj:

```text
CLI
```

A Cisco CLI-ben több konfigurációs mód létezik.

Most csak annyit tanulunk, amennyi ehhez a hálózathoz kell.

---

# 24. Cisco CLI alaplogika

Kezdet:

```text
Router>
```

Ez user EXEC mode.

Írd:

```text
enable
```

Ezután:

```text
Router#
```

Privileged EXEC mode.

---

# 25. Configuration mode

Írd:

```text
configure terminal
```

vagy röviden:

```text
conf t
```

Ezután:

```text
Router(config)#
```

---

# 26. Első router interface

Először derítsd ki, melyik interface megy a Switch-A felé.

Például lehet:

```text
GigabitEthernet0/0
```

vagy:

```text
GigabitEthernet0/0/0
```

Ez routermodelltől függ.

Ne vakon másold a példát.

---

# 27. Interface konfiguráció

Példa:

```text
interface gigabitEthernet 0/0
```

Ezután:

```text
Router(config-if)#
```

---

# 28. LAN A router IP

Állítsd be:

```text
ip address 192.168.10.1 255.255.255.0
```

Majd:

```text
no shutdown
```

A:

```text
no shutdown
```

bekapcsolja az interface-t.

---

# 29. Második router interface

Lépj vissza:

```text
exit
```

Majd válaszd a másik interfészt.

Példa:

```text
interface gigabitEthernet 0/1
```

Állítsd:

```text
ip address 192.168.20.1 255.255.255.0
```

Majd:

```text
no shutdown
```

---

# 30. Router interface ellenőrzése

Lépj ki konfigurációs módból.

Hasznos parancs:

```text
show ip interface brief
```

Keresd:

```text
192.168.10.1
192.168.20.1
```

és állapotként lehetőleg:

```text
up
up
```

---

# 31. Mit jelent az `up/up`?

Nagyon leegyszerűsítve:

```text
Status up
→ fizikai/interface oldal működik

Protocol up
→ line protocol működik
```

Most a cél:

```text
up/up
```

---

# 32. Ha administratively down

Ha ezt látod:

```text
administratively down
```

akkor valószínűleg hiányzik:

```text
no shutdown
```

---

# 33. Router routing table

Futtasd:

```text
show ip route
```

Keress két directly connected networköt:

```text
192.168.10.0/24
192.168.20.0/24
```

Ez azért fontos, mert a router mindkét hálózathoz közvetlenül csatlakozik.

---

# 34. Kell static route?

Ebben a topológiában:

```text
NEM
```

Miért?

Mert mindkét hálózat:

```text
directly connected
```

a routerhez.

A router routing table-je automatikusan tud róluk.

---

# 35. Első teszt – same subnet

PC-A:

```text
Desktop
→ Command Prompt
```

Pingeld PC-B-t:

```text
ping 192.168.10.20
```

A két gép ugyanabban a subnetben van.

---

# 36. Mit várunk?

```text
PC-A
↓
Switch-A
↓
PC-B
```

A router nem szükséges.

Mental model:

```text
same subnet
→ direct communication
```

---

# 37. Első ping lehet sikertelen

Packet Tracerben az első ping néha elveszhet, miközben ARP történik.

Ez fontos.

A gépnek először meg kell tudnia:

```text
192.168.10.20
↓
milyen MAC?
```

Ezért első próbálkozáskor ARP történhet.

---

# 38. Ismételd meg

```text
ping 192.168.10.20
```

Második alkalommal már gyakran sikeres.

Miért?

```text
ARP cache
```

már tartalmazhatja a szükséges IP → MAC információt.

---

# 39. Második teszt – másik subnet

PC-A-ról:

```text
ping 192.168.20.10
```

Ez PC-C.

Most:

```text
source:
192.168.10.10

destination:
192.168.20.10
```

Más subnet.

---

# 40. Mi történik?

PC-A megvizsgálja:

```text
destination same subnet?
```

Válasz:

```text
NO
```

Ezért:

```text
default gateway
→ 192.168.10.1
```

---

# 41. A packet útja

```text
PC-A
↓
Switch-A
↓
Router
↓
Switch-B
↓
PC-C
```

Ez már routing.

---

# 42. Harmadik teszt – Server

PC-A:

```text
ping 192.168.20.100
```

Útvonal:

```text
PC-A
↓
Switch-A
↓
Router
↓
Switch-B
↓
Server
```

---

# 43. Realtime mode

Alapállapotban a Packet Tracer:

```text
Realtime
```

módban működik.

Ilyenkor a hálózat nagyjából folyamatosan, valós időhöz hasonlóan működik.

Pingnél csak az eredményt látjuk.

---

# 44. Simulation mode

A Packet Tracer egyik legfontosabb oktatási funkciója:

```text
Simulation
```

mode.

A jobb alsó területen válts:

```text
Realtime
→ Simulation
```

Simulation módban a packetek útját lépésenként tudjuk követni.

---

# 45. Simulation Panel

Simulation módban megjelenik egy panel.

Fontos részei lehetnek:

```text
Event List
Play Controls
Event List Filters
```

---

# 46. Event List

Itt látjuk:

```text
milyen esemény történt
melyik eszköznél
milyen protokoll
milyen sorrendben
```

Például:

```text
ARP
ICMP
```

---

# 47. Event List Filters

A hálózat sok protokollt generálhat.

Ezért használhatjuk:

```text
Event List Filters
```

A cél:

> csak azt mutassa, amit éppen tanulmányozunk.

---

# 48. Első Simulation filter

Kapcsold ki a fölösleges protokollokat.

Első körben hagyd:

```text
ARP
ICMP
```

Így a ping működése sokkal átláthatóbb lesz.

---

# 49. Simple PDU

Packet Tracerben található:

```text
Add Simple PDU
```

eszköz.

Az ikon általában borítékhoz hasonló.

Ezzel egyszerűen indíthatunk connectivity tesztet két eszköz között.

---

# 50. Simple PDU használata

Válaszd:

```text
Add Simple PDU
```

Ezután kattints:

1. source device;
2. destination device.

Például:

```text
PC-A
↓
PC-B
```

Ez egyszerű ICMP/ping jellegű kommunikációt indít.

---

# 51. Első Simulation – same subnet

Simulation mode.

Filter:

```text
ARP
ICMP
```

Simple PDU:

```text
PC-A
→ PC-B
```

Ne indítsd automatikusan gyorsan végig.

Használd a léptetést.

---

# 52. Capture / Forward

A Simulation panelben található:

```text
Capture / Forward
```

funkció.

Ezzel egy-egy eseméssel tovább tudjuk léptetni a szimulációt.

Ez az egyik legjobb tanulási eszköz.

---

# 53. Mit figyelj?

Az első lépésekben valószínűleg ARP történik.

Például:

```text
PC-A
↓
ARP Request
↓
Switch
↓
PC-B
```

---

# 54. ARP Request

A PC-A azt akarja megtudni:

```text
Who has 192.168.10.20?
```

Vagyis:

> melyik MAC-cím tartozik PC-B IP-jéhez?

---

# 55. Switch szerepe

Az ARP Request broadcast.

A switch továbbítja a broadcast frame-et a megfelelő portokra.

Fontos:

```text
switch
→ MAC alapján kapcsol
```

A switch nem routingol a két subnet között.

---

# 56. ARP Reply

PC-B válaszol:

```text
192.168.10.20
→ az én MAC-címem ...
```

Ez visszajut PC-A-hoz.

---

# 57. Ezután jöhet ICMP

Miután PC-A tudja PC-B MAC-címét:

```text
ICMP Echo Request
```

mehet közvetlenül PC-B felé.

Majd:

```text
ICMP Echo Reply
```

vissza.

---

# 58. PDU részleteinek megnyitása

Simulation Event Listben kattints egy eseményre vagy PDU-ra.

Megjelenhet részletes nézet.

Itt különböző füleken látható lehet például:

```text
OSI Model
Inbound PDU Details
Outbound PDU Details
```

A pontos elrendezés verziófüggő lehet.

---

# 59. OSI Model nézet

Ez nagyon hasznos.

Itt Packet Tracer megmutathatja:

```text
Layer 1
Layer 2
Layer 3
...
```

szinten, hogy az adott eszköz mit csinál a packettel.

Nem azt jelenti, hogy fizikailag hét külön géprész dolgozik.

Ez ugyanaz a logikai modell, amit már tanultunk.

---

# 60. Inbound PDU Details

Megmutathatja, milyen információval érkezett a packet.

Például:

```text
source MAC
destination MAC
source IP
destination IP
ICMP
```

---

# 61. Outbound PDU Details

Megmutathatja, milyen formában küldi tovább az eszköz.

Ez különösen routernél érdekes.

---

# 62. LAB A – same subnet packet

Vizsgáld:

```text
PC-A
→ PC-B
```

Simulation mode-ban.

Feladat:

Jegyezd fel:

```text
Source IP:
Destination IP:

Source MAC:
Destination MAC:
```

---

# 63. LAB A kérdés

Eljut-e a packet a routerhez?

Miért?

Helyes mental model:

```text
same subnet
→ router nem szükséges
```

---

# 64. LAB B – different subnet

Most Simple PDU:

```text
PC-A
→ PC-C
```

Filter:

```text
ARP
ICMP
```

Lépdelj végig.

---

# 65. PC-A döntése

PC-A látja:

```text
192.168.10.10/24
```

és destination:

```text
192.168.20.10
```

Ez más subnet.

Ezért a packetet a:

```text
default gateway
```

felé küldi.

---

# 66. Fontos MAC/IP különbség

PC-A oldalán:

```text
Destination IP:
192.168.20.10
```

de az Ethernet destination MAC:

```text
Router LAN-A MAC
```

Miért?

Mert:

```text
IP
→ végső destination

MAC
→ következő local hop
```

Ez az egyik legfontosabb networking felismerés.

---

# 67. Routerhez érkezés

A router megkapja a frame-et.

Az Ethernet frame a router MAC-jére szólt.

A router kibontja az IP packetet és megnézi:

```text
Destination IP:
192.168.20.10
```

---

# 68. Router routing decision

A router routing table-jében:

```text
192.168.20.0/24
→ directly connected
```

Ezért tudja:

```text
küldd ki LAN B felé
```

---

# 69. Új Ethernet frame

Nagyon fontos:

A router a következő linken új Layer 2 frame-et készít.

Tehát:

```text
LAN A frame
```

nem egyszerűen ugyanaz a frame megy végig az egész hálózaton.

---

# 70. IP és MAC változások

PC-A → Router:

```text
Source IP:
192.168.10.10

Destination IP:
192.168.20.10
```

MAC:

```text
PC-A MAC
→ Router LAN-A MAC
```

Router → PC-C:

IP továbbra is:

```text
192.168.10.10
→ 192.168.20.10
```

de MAC már:

```text
Router LAN-B MAC
→ PC-C MAC
```

---

# 71. LAB B fő feladata

Packet Tracerben figyeld meg:

```text
IP címek
```

és:

```text
MAC címek
```

változását minden hopnál.

Írd le:

> Mi marad ugyanaz, és mi változik?

---

# 72. Simulation Reset

Új teszt előtt érdemes:

```text
Reset Simulation
```

funkciót használni.

Ezzel tisztább lesz a következő teszt.

Ha régi PDU-k maradnak, töröld vagy indíts új scenario-t.

---

# 73. ARP cache hatása

Futtasd ugyanazt a kommunikációt kétszer.

Első alkalom:

```text
ARP
+
ICMP
```

Második alkalommal lehet, hogy:

```text
ICMP
```

hamarabb indul, mert ARP információ már rendelkezésre áll.

---

# 74. PC ARP table megtekintése

PC:

```text
Desktop
→ Command Prompt
```

Próbáld:

```text
arp -a
```

Ha az adott Packet Tracer PC támogatja, megjelenhet az ARP cache.

Nézd meg ping előtt és után.

---

# 75. Router ARP table

Router CLI:

```text
show arp
```

vagy eszköz/modell függvényében megfelelő ARP megjelenítő parancs.

Keresd:

```text
192.168.10.x
192.168.20.x
```

címeket.

---

# 76. Switch MAC table

Switch CLI:

```text
show mac address-table
```

Itt láthatod:

```text
MAC address
→ switch port
```

kapcsolatot.

Ez a switch egyik legfontosabb működési táblája.

---

# 77. Switch tanulása

Amikor frame érkezik:

```text
source MAC
↓
bejövő port
```

A switch megtanulja:

> ez a MAC ezen a porton található.

Mental model:

```text
MAC table
→ MAC → port
```

---

# 78. LAB C – MAC address table

Pingelj:

```text
PC-A → PC-B
```

Majd Switch-A:

```text
show mac address-table
```

Feladat:

Azonosítsd:

```text
PC-A MAC
PC-B MAC
Router MAC
```

ha megjelennek.

---

# 79. LAB D – router routing table

Router:

```text
show ip route
```

Feladat:

Keresd:

```text
192.168.10.0/24
192.168.20.0/24
```

Magyarázd el:

Miért ismeri őket static route nélkül?

---

# 80. LAB E – default gateway nélkül

Most kezdődik a troubleshooting.

PC-A jelenlegi gateway:

```text
192.168.10.1
```

Ideiglenesen töröld ki vagy állíts be rosszat.

Például:

```text
Default Gateway:
üres
```

---

# 81. Mit tesztelj?

PC-A → PC-B:

```text
ping 192.168.10.20
```

PC-A → PC-C:

```text
ping 192.168.20.10
```

---

# 82. Mit várunk?

PC-A → PC-B:

```text
működhet
```

Miért?

```text
same subnet
→ gateway nem kell
```

PC-A → PC-C:

```text
nem működik
```

Miért?

```text
different subnet
→ gateway kell
```

---

# 83. Hiba felismerése

Ez klasszikus helyzet:

```text
local LAN works
remote LAN fails
```

Első gyanúk:

```text
default gateway
routing
router
```

---

# 84. Simulation megfigyelés

Próbáld PC-A → PC-C Simple PDU-val.

Figyeld:

```text
meddig jut a packet?
```

Ez lesz a Packet Tracer troubleshooting legfontosabb kérdése is.

---

# 85. Javítás

Állítsd vissza:

```text
192.168.10.1
```

gateway-t.

Teszteld újra.

---

# 86. LAB F – rossz subnet mask

PC-A subnet maskját változtasd hibásra.

Például:

```text
255.255.0.0
```

a helyes:

```text
255.255.255.0
```

helyett.

---

# 87. Mit okozhat?

PC-A tévesen azt gondolhatja bizonyos címekről:

```text
same subnet
```

miközben valójában más subnetben vannak.

Ez megváltoztatja azt, hogy:

```text
destination MAC
```

kitől próbálja megszerezni.

---

# 88. Simulation feladat

Hibás maskkal próbáld:

```text
PC-A
→ PC-C
```

Figyeld:

```text
ARP viselkedés
```

Mit próbál PC-A megtudni?

Miért?

---

# 89. Javítsd vissza

```text
255.255.255.0
```

Majd teszteld újra.

---

# 90. LAB G – rossz IP-cím

PC-B IP-jét állítsd:

```text
192.168.20.20
```

miközben továbbra is fizikailag Switch-A-hoz kapcsolódik.

Most logikailag más subnetbe került.

---

# 91. Teszt

PC-A:

```text
ping 192.168.20.20
```

Figyeld meg, mi történik.

Gondold végig:

```text
fizikai kapcsolat
≠
logikai IP hálózat
```

Ez nagyon fontos.

---

# 92. Physical vs logical network

Két gép lehet fizikailag ugyanazon switchen:

```text
PC-A
↓
Switch
↑
PC-B
```

de hibás IP-konfiguráció miatt logikailag más hálózatba kerülhetnek.

---

# 93. LAB H – router interface shutdown

Routeren kapcsold le a LAN B interface-t.

CLI:

```text
enable
configure terminal
interface <LAN-B-interface>
shutdown
```

---

# 94. Ellenőrzés

```text
show ip interface brief
```

Keresd:

```text
administratively down
```

---

# 95. Teszt

PC-A → PC-B:

```text
?
```

PC-A → PC-C:

```text
?
```

PC-C → Server:

```text
?
```

Gondold végig előre, majd teszteld.

---

# 96. Mit várunk?

LAN A belső kommunikáció:

```text
működik
```

LAN B belső kommunikáció:

```text
működhet
```

Routeren keresztüli kapcsolat:

```text
nem működik
```

---

# 97. Javítás

Router:

```text
interface <LAN-B-interface>
no shutdown
```

Ellenőrzés:

```text
show ip interface brief
```

---

# 98. LAB I – rossz gateway PC-C-n

PC-C gateway:

helyes:

```text
192.168.20.1
```

állítsd például:

```text
192.168.20.254
```

---

# 99. Teszt

PC-C → Server:

```text
ping 192.168.20.100
```

PC-C → PC-A:

```text
ping 192.168.10.10
```

Mi működik?

Mi nem?

Miért?

---

# 100. Tipikus következtetés

```text
same subnet works
remote subnet fails
```

→ gateway hiba valószínű.

---

# 101. LAB J – kábelhiba

Töröld vagy bontsd le ideiglenesen például:

```text
PC-A
→ Switch-A
```

kapcsolatot.

Figyeld a link state-et.

---

# 102. Teszt

```text
PC-A
→ bármi
```

Mi történik?

Ez már:

```text
Layer 1
```

jellegű probléma.

---

# 103. Troubleshooting rétegenként

Most már láthatjuk:

```text
rossz kábel
→ Layer 1
```

```text
MAC / switching
→ Layer 2
```

```text
IP / mask / gateway / routing
→ Layer 3
```

Ez az OSI modell gyakorlati használata.

---

# 104. LAB K – server HTTP szolgáltatás

Kattints a Serverre.

Menj:

```text
Services
```

Keresd:

```text
HTTP
```

Kapcsold be, ha nincs bekapcsolva.

---

# 105. Web Browser teszt

PC-A:

```text
Desktop
→ Web Browser
```

Írd be:

```text
http://192.168.20.100
```

Ha minden jól működik, meg kell jelennie a szerver weboldalának.

---

# 106. Mit bizonyít a sikeres HTTP?

Ha PC-A böngészőből betölt:

```text
192.168.20.100
```

akkor több réteg már működik:

```text
PC IP config
default gateway
switching
routing
server IP
TCP
HTTP
```

---

# 107. LAB L – HTTP szolgáltatás kikapcsolása

Server:

```text
Services
→ HTTP
→ Off
```

Majd PC-A-ról próbáld újra.

---

# 108. Mit tanulunk ebből?

A network továbbra is működhet:

```text
ping server
→ OK
```

de:

```text
HTTP
→ FAIL
```

Ez nagyon fontos troubleshooting lecke:

```text
host reachable
≠
service reachable
```

---

# 109. Ping vs HTTP

```text
ping
→ ICMP / reachability
```

```text
HTTP
→ alkalmazási szolgáltatás
```

Egyik működése nem bizonyítja automatikusan a másikat.

---

# 110. LAB M – HTTP packet Simulation módban

Kapcsold vissza HTTP-t.

Simulation mode.

Event filtersnél próbáld megtartani:

```text
ARP
TCP
HTTP
```

Ha szükséges, ICMP kikapcsolható.

---

# 111. PC-A Web Browser

Nyisd:

```text
http://192.168.20.100
```

Simulation módban kövesd az eseményeket.

---

# 112. Mit próbálunk felismerni?

Valami ehhez hasonló sorrendet:

```text
ARP
↓
TCP
↓
HTTP
```

A pontos eseménysor a szimuláció és cache állapotától függ.

---

# 113. TCP kapcsolat felismerése

Keresd a TCP eseményeket.

Próbáld megtalálni:

```text
SYN
SYN-ACK
ACK
```

ha a Packet Tracer részletes nézete megjeleníti őket.

---

# 114. HTTP request

Keresd:

```text
HTTP
```

PDU-t.

Vizsgáld meg a részleteket.

Próbáld azonosítani:

```text
source IP
destination IP
source port
destination port
```

---

# 115. Destination port

HTTP server:

```text
80
```

A kliens oldalán várható:

```text
ephemeral port
```

Így visszajön a korábbi socket mental model.

---

# 116. HTTP flow

```text
PC-A
↓
TCP connection
↓
Router
↓
Server:80
↓
HTTP request
↓
HTTP response
```

---

# 117. LAB N – DNS hozzáadása

A Packet Tracer Server képes DNS szolgáltatást is szimulálni.

A Serveren:

```text
Services
→ DNS
```

Kapcsold be.

---

# 118. DNS record

Adj hozzá például:

```text
Name:
lab.local

Address:
192.168.20.100
```

---

# 119. Kliens DNS beállítása

PC-A IP Configuration:

DNS Server:

```text
192.168.20.100
```

Ugyanezt beállíthatod PC-B/PC-C számára is.

---

# 120. DNS teszt

PC-A Web Browser:

```text
http://lab.local
```

Ha működik:

```text
lab.local
↓
DNS
↓
192.168.20.100
↓
HTTP
```

---

# 121. Simulation DNS

Simulation mode.

Filter:

```text
DNS
UDP
ARP
TCP
HTTP
```

Majd:

```text
http://lab.local
```

---

# 122. Mit keresünk?

A teljes láncot:

```text
DNS query
↓
DNS response
↓
TCP
↓
HTTP
```

Ez nagyon közel áll a valódi web request flow-hoz.

---

# 123. DNS hiba létrehozása

PC-A-n állíts be rossz DNS Server címet.

Például:

```text
192.168.20.200
```

ahol nincs DNS server.

Próbáld:

```text
http://lab.local
```

---

# 124. Ezután próbáld IP-vel

```text
http://192.168.20.100
```

Ha IP-vel működik, névvel nem:

```text
network / HTTP
→ működik

DNS
→ hibás
```

Ez pontosan a korábbi troubleshooting logikánk.

---

# 125. LAB O – DNS service kikapcsolása

Server:

```text
Services
→ DNS
→ Off
```

Majd:

```text
http://lab.local
```

és:

```text
http://192.168.20.100
```

Hasonlítsd össze.

---

# 126. Komplex troubleshooting challenge

Most állítsunk elő egy ismeretlen hibát.

Válassz egyet:

```text
wrong IP
wrong subnet mask
wrong gateway
router interface shutdown
HTTP service off
DNS service off
wrong DNS server
broken cable
```

Ne jegyezd fel fejben túl erősen, majd később próbáld diagnosztizálni.

---

# 127. Troubleshooting sorrend

Kérdezd végig:

```text
1. Van link?
2. Jó az IP?
3. Jó a subnet mask?
4. Same subnet működik?
5. Jó a gateway?
6. Router interface up?
7. Routing table jó?
8. Ping működik?
9. DNS működik?
10. Application service működik?
```

---

# 128. Ping mint első szétválasztás

Például:

```text
PC-A → PC-B
```

működik.

De:

```text
PC-A → Server
```

nem.

Ebből:

```text
LAN A
→ valószínűleg jó
```

A hiba később lehet:

```text
gateway
router
LAN B
server
```

---

# 129. Divide and conquer

Teszt:

```text
PC-A
→ Router LAN-A
```

Majd:

```text
Router
→ PC-C
```

Majd:

```text
PC-A
→ PC-C
```

Keressük:

> melyik kapcsolat az első, ami nem működik?

---

# 130. Router CLI troubleshooting parancsok

Hasznos:

```text
show ip interface brief
```

```text
show ip route
```

```text
show arp
```

```text
show running-config
```

---

# 131. Switch troubleshooting parancsok

Hasznos:

```text
show mac address-table
```

és szükség esetén interface állapotot mutató parancsok.

---

# 132. PC troubleshooting

PC:

```text
Desktop
→ IP Configuration
```

ellenőrizd:

```text
IP
Mask
Gateway
DNS
```

Command Prompt:

```text
ipconfig
```

ha támogatott.

---

# 133. Packet Tracer Simple PDU troubleshootinghoz

Ha gyorsan connectivityt akarsz tesztelni:

```text
Add Simple PDU
```

Source:

```text
PC-A
```

Destination:

```text
Server
```

Simulation mode-ban pedig pontosan láthatod:

```text
hol áll meg?
```

---

# 134. Sikertelen PDU

Ha a Simple PDU sikertelen:

ne csak annyit jegyezz meg:

```text
Failed
```

Hanem:

```text
hol állt meg?
melyik eszköznél?
melyik protokollnál?
```

Ez a valódi troubleshooting gondolkodás.

---

# 135. Simulation workflow

Jó munkamódszer:

```text
1. Simulation mode
2. Reset Simulation
3. Filters beállítása
4. Simple PDU létrehozása
5. Capture / Forward
6. Event List figyelése
7. PDU Details megnyitása
8. Következtetés
```

---

# 136. Realtime vs Simulation

## Realtime

Használd, ha:

```text
csak gyorsan tesztelni akarod, működik-e
```

## Simulation

Használd, ha:

```text
meg akarod érteni, hogyan működik
```

---

# 137. Egyszerű troubleshooting táblázat

```text
Same LAN ping FAIL
→ IP / mask / cable / switch / host
```

```text
Same LAN OK
Remote LAN FAIL
→ gateway / router / route
```

```text
Ping server OK
HTTP FAIL
→ HTTP service / application
```

```text
IP web access OK
Hostname FAIL
→ DNS
```

```text
Everything FAIL from one PC
→ PC config / cable / interface
```

---

# 138. Hibaszituáció 1

PC-A:

```text
IP jó
mask jó
gateway üres
```

Eredmény:

```text
PC-A → PC-B
?
```

```text
PC-A → Server
?
```

Magyarázd el előre, majd teszteld.

---

# 139. Hibaszituáció 2

PC-A:

```text
mask = 255.255.0.0
```

Eredmény:

```text
PC-A → PC-C
```

Figyeld Simulation módban.

Mit gondol a kliens a destination subnetről?

---

# 140. Hibaszituáció 3

Router LAN-B:

```text
shutdown
```

Vizsgáld:

```text
show ip interface brief
```

Majd pingek.

---

# 141. Hibaszituáció 4

Server HTTP:

```text
Off
```

Ping:

```text
OK
```

Browser:

```text
FAIL
```

Melyik réteg hibás?

---

# 142. Hibaszituáció 5

DNS:

```text
Off
```

IP alapú browser:

```text
OK
```

domain alapú:

```text
FAIL
```

Mit bizonyít?

---

# 143. Hibaszituáció 6

PC-C rossz gateway:

```text
192.168.20.254
```

PC-C → Server:

```text
?
```

PC-C → PC-A:

```text
?
```

---

# 144. Hibaszituáció 7

PC-B rossz IP:

```text
192.168.20.20
```

de fizikailag továbbra is LAN A switchen van.

Miért érdekes ez?

```text
physical topology
≠
logical topology
```

---

# 145. Komplex végső topológia

A végén működjön:

```text
PC-A ─┐
      ├─ Switch-A ─ Router ─ Switch-B ─ PC-C
PC-B ─┘                          │
                              Server
```

Címek:

```text
LAN A:
192.168.10.0/24

LAN B:
192.168.20.0/24
```

Server:

```text
DNS
HTTP
```

---

# 146. Végső tesztek

PC-A:

```text
ping PC-B
```

```text
ping Router LAN-A
```

```text
ping PC-C
```

```text
ping Server
```

Browser:

```text
http://192.168.20.100
```

Majd:

```text
http://lab.local
```

---

# 147. Végső Simulation teszt

Filterek:

```text
ARP
ICMP
DNS
UDP
TCP
HTTP
```

Majd PC-A-ról:

```text
http://lab.local
```

Próbáld az egész folyamatot követni.

---

# 148. Mit kell felismerni?

Ideális esetben:

```text
DNS szükséges
↓
DNS query
↓
DNS response
↓
server IP ismert
↓
different subnet
↓
default gateway
↓
routing
↓
TCP kapcsolat
↓
HTTP request
↓
HTTP response
```

---

# 149. MAC és IP megfigyelése router előtt

PC-A → Router:

```text
Source IP:
192.168.10.10

Destination IP:
192.168.20.100
```

Ethernet:

```text
Source MAC:
PC-A

Destination MAC:
Router LAN-A
```

---

# 150. MAC és IP router után

Router → Server:

IP:

```text
Source IP:
192.168.10.10

Destination IP:
192.168.20.100
```

MAC:

```text
Source MAC:
Router LAN-B

Destination MAC:
Server
```

Ez az egyik legfontosabb feladat az egész laborban.

---

# 151. Mit kell megérteni ebből?

```text
IP
→ end-to-end host addressing
```

```text
MAC
→ local-hop addressing
```

Routerenként a Layer 2 frame változik.

A Layer 3 destination alapvetően a végső célra mutat.

---

# 152. Packet Tracer és a valóság

Packet Tracer:

```text
simulation
```

Nem valódi hálózati hardver.

Bizonyos protokollokat és viselkedéseket leegyszerűsít.

Viszont kiváló arra, hogy vizuálisan megértsük:

```text
switching
routing
ARP
ICMP
DNS
TCP
HTTP
```

alaplogikáját.

---

# 153. Packet Tracer vs Wireshark

## Packet Tracer

```text
virtuális hálózat
↓
mi történne a hálózati eszközök között?
```

## Wireshark

```text
valódi network traffic
↓
mi történt ténylegesen a packetekben?
```

---

# 154. Packet Tracer vs WSL

## WSL lab

```text
Linux host nézőpontja
```

Látjuk:

```text
ip addr
ip route
ss
dig
curl
```

---

## Packet Tracer

```text
hálózat nézőpontja
```

Látjuk:

```text
PC
Switch
Router
Server
packet path
```

---

# 155. A három labor együtt

```text
WSL
→ mit lát a host?
```

```text
Wireshark
→ mit tartalmaz a packet?
```

```text
Packet Tracer
→ hogyan mozog a packet a hálózatban?
```

Ez a három nézőpont együtt ad igazán jó networking mental modelt.

---

# 156. Packet Tracer alap fogalomtár

## Logical Workspace

```text
a hálózat logikai topológiájának munkaterülete
```

## Realtime

```text
a hálózat folyamatosan működik
```

## Simulation

```text
packetek lépésenként követhetők
```

## Simple PDU

```text
gyors grafikus connectivity teszt
```

## Event List

```text
szimuláció eseményeinek listája
```

## Event Filters

```text
mely protokollok eseményeit mutassa?
```

## Capture / Forward

```text
egy szimulációs lépéssel tovább
```

## Auto Capture / Play

```text
automatikusan futtatja az eseményeket
```

## Reset Simulation

```text
szimuláció visszaállítása / újrakezdése
```

---

# 157. Hálózati fogalmak a topológiában

## Switch

```text
LAN-on belüli Layer 2 továbbítás
```

Főként:

```text
MAC
```

alapján.

## Router

```text
külön hálózatok közötti Layer 3 továbbítás
```

Főként:

```text
destination IP
routing table
```

alapján.

## Default gateway

```text
router, amelyhez a host más subnetek forgalmát küldi
```

---

# 158. Gyors parancslista

## PC

```text
ping <IP>
arp -a
ipconfig
```

ha az adott Packet Tracer eszköz támogatja.

---

## Router

```text
enable
configure terminal
```

```text
show ip interface brief
show ip route
show arp
show running-config
```

Interface:

```text
interface <name>
ip address <IP> <MASK>
no shutdown
shutdown
```

---

## Switch

```text
show mac address-table
```

---

# 159. Legfontosabb troubleshooting mental model

```text
Physical link
↓
IP config
↓
Subnet
↓
Gateway
↓
Switching
↓
Routing
↓
Destination host
↓
Transport
↓
Application
```

Mindig kérdezd:

> Hol volt az utolsó sikeres pont?

---

# 160. Végső önálló challenge

Állítsd vissza teljesen működő állapotra a hálózatot.

Ezután okozz benne három hibát.

Például:

```text
1. wrong gateway
2. router interface down
3. DNS off
```

Ne egyszerre javítsd őket.

Menj végig:

```text
link?
↓
local ping?
↓
gateway?
↓
remote ping?
↓
DNS?
↓
HTTP?
```

és dokumentáld:

```text
Tünet:
Első gyanú:
Teszt:
Eredmény:
Valódi hiba:
Javítás:
```

---

# 161. Végső magyarázó feladat

A saját topológiád alapján magyarázd el:

> Mi történik, amikor PC-A megnyitja a `http://lab.local` oldalt?

Használd:

```text
DNS
ARP
MAC
IP
subnet
default gateway
switch
router
routing table
TCP
port 80
HTTP
server
response
```

---

# 162. A labor végére ezt kell tudnod

- eszközt elhelyezni Packet Tracerben;
- eszközöket összekötni;
- PC-nek IP-t, maskot, gateway-t adni;
- router interface-eket konfigurálni;
- `no shutdown` jelentését érteni;
- routing table-t megvizsgálni;
- same subnet és different subnet kommunikációt elkülöníteni;
- ARP működését Simulation módban felismerni;
- switch MAC-table szerepét érteni;
- router packet-forwarding szerepét érteni;
- Simple PDU-t használni;
- Simulation módot használni;
- Event Filtert használni;
- packeteket lépésenként követni;
- OSI/PDU részleteket megvizsgálni;
- pinget hibakeresésre használni;
- DNS és HTTP szolgáltatást konfigurálni;
- rossz gateway, mask, IP, down interface és application hiba között különbséget tenni;
- megkeresni, hogy egy packet hol akad el.

---

# 163. Kapcsolat a következő Docker tananyaggal

Később Dockerben újra találkozunk:

```text
host
container
bridge
internal network
gateway
DNS
port
routing
```

fogalmakkal.

Például:

```text
Browser
↓
Host :443
↓
Nginx container
↓
Docker network
↓
Node container
↓
Docker network
↓
Postgres container
```

A Packet Tracerben megtanult:

```text
ki melyik hálózatban van?
hogyan találja meg a következő hopot?
melyik szolgáltatás érhető el?
hol akad el a packet?
```

gondolkodás ott ugyanúgy hasznos lesz.