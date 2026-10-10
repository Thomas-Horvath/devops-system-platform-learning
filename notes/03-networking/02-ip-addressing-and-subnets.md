# IP Addressing and Subnets

## 1. Mi az az IP-cím?

Az **IP address** egy logikai hálózati cím.

Segítségével a hálózat azonosítani tud egy hálózati célt.

Például:

```text
192.168.10.77
```

Ez egy IPv4 cím.

Egyszerű mental model:

```text
IP address
→ hol található a host a hálózaton?
```

---

# 2. IPv4 cím felépítése

Egy IPv4 cím:

```text
32 bit
```

hosszú.

Négy darab 8 bites részre van bontva:

```text
192 . 168 . 10 . 77
```

Egy ilyen 8 bites részt:

```text
octet
```

néven hívunk.

Tehát:

```text
4 octet × 8 bit
=
32 bit
```

---

# 3. Miért 0–255 egy octet?

Egy octet 8 bites.

A bináris helyiértékek:

```text
128  64  32  16   8   4   2   1
```

Ha minden bit 0:

```text
00000000
=
0
```

Ha minden bit 1:

```text
11111111
```

akkor:

```text
128 + 64 + 32 + 16 + 8 + 4 + 2 + 1
=
255
```

Ezért egy octet értéke:

```text
0–255
```

lehet.

---

# 4. Bináris számrendszer alapjai

A számítógépek kétállapotú bitekkel dolgoznak:

```text
0
1
```

Egy 8 bites octet helyiértékei:

```text
128  64  32  16   8   4   2   1
```

Ezt a sort subnetelésnél nagyon érdemes ismerni.

---

# 5. Binárisból tízes számrendszerbe

Példa:

```text
11000000
```

Helyiértékek:

```text
128 64 32 16 8 4 2 1
 1   1  0  0 0 0 0 0
```

Ahol `1` van, összeadjuk a helyiértékeket:

```text
128 + 64 = 192
```

Tehát:

```text
11000000₂ = 192₁₀
```

---

## Másik példa

```text
01001101
```

```text
128 64 32 16 8 4 2 1
 0   1  0  0 1 1 0 1
```

Ez:

```text
64 + 8 + 4 + 1
=
77
```

Tehát:

```text
01001101₂ = 77₁₀
```

---

# 6. Tízesből binárisba

Legyen:

```text
77
```

A helyiértékek:

```text
128 64 32 16 8 4 2 1
```

128 nem fér bele 77-be:

```text
0
```

64 igen:

```text
77 - 64 = 13
```

32 nem.

16 nem.

8 igen:

```text
13 - 8 = 5
```

4 igen:

```text
5 - 4 = 1
```

2 nem.

1 igen.

Ezért:

```text
77
=
01001101
```

---

# 7. Miért nem elég önmagában az IP-cím?

Legyen:

```text
192.168.10.77
```

Ebből önmagában még nem tudjuk:

```text
melyik rész jelöli a hálózatot?
melyik rész jelöli a hostot?
```

Ehhez kell:

```text
subnet mask
```

vagy annak rövidebb jelölése:

```text
CIDR prefix
```

Például:

```text
192.168.10.77/26
```

---

# 8. Mi az a subnet mask?

A **subnet mask** megmondja, hogy az IP-cím bitjei közül melyek tartoznak:

```text
network részhez
```

és melyek:

```text
host részhez
```

A maskban:

```text
1 = network bit pozíció
0 = host bit pozíció
```

Fontos:

> A mask 1-es bitje nem azt jelenti, hogy az IP-cím adott bitjének is 1-nek kell lennie.

A mask csak azt mondja:

> ezt a pozíciót network bitként kezeld.

---

# 9. Mi az a CIDR?

A **CIDR prefix** azt mutatja meg, hány network bit van.

Például:

```text
/24
```

jelentése:

```text
24 network bit
8 host bit
```

Mivel IPv4:

```text
32 bit
```

ezért:

```text
host bits
=
32 - CIDR
```

---

## Példák

```text
/24
→ 24 network
→ 8 host
```

```text
/26
→ 26 network
→ 6 host
```

```text
/27
→ 27 network
→ 5 host
```

```text
/28
→ 28 network
→ 4 host
```

---

# 10. /24 subnet mask

`/24` azt jelenti:

```text
11111111.11111111.11111111.00000000
```

Mivel:

```text
11111111 = 255
```

ezért:

```text
/24
=
255.255.255.0
```

---

# 11. A házas hasonlat – /24

Képzeljük el a `/24` hálózatot egy nagy házként.

Az utolsó octet:

```text
0–255
```

összesen 256 cím.

Mintha lenne egy:

```text
256 címes nagy épületünk
```

A `/24` esetén az utolsó 8 bit mind host bit:

```text
HHHHHHHH
```

Tehát egyetlen nagy tartományunk van:

```text
0–255
```

Például:

```text
192.168.10.77/24
```

esetén a `77` egyszerűen ennek az egy nagy hálózatnak az egyik host címe.

---

# 12. Network address és broadcast

Egy hagyományos IPv4 subnetben két speciális cím van.

## Network address

A subnet első címe.

Például:

```text
192.168.10.0/24
```

Itt:

```text
192.168.10.0
```

a network address.

A host bitek mind:

```text
0
```

értékűek.

---

## Broadcast address

A subnet utolsó címe.

Például:

```text
192.168.10.255
```

A host bitek mind:

```text
1
```

értékűek.

---

## Használható hostok

`/24` esetén:

```text
192.168.10.0
→ network

192.168.10.1–254
→ usable hosts

192.168.10.255
→ broadcast
```

Összesen:

```text
256 total address
254 usable host address
```

---

# 13. Mi történik /26 esetén?

Most képzeljük el, hogy a nagy 256 címes házat kisebb épületekre bontjuk.

`/24`:

```text
HHHHHHHH
```

`/26`:

```text
NNHHHHHH
```

Két host bitet:

```text
network bitté tettünk
```

Ezért marad:

```text
6 host bit
```

---

# 14. /26 subnet mask kiszámítása

Az első három octet:

```text
255.255.255
```

már 24 network bit.

A `/26` miatt az utolsó octetben még:

```text
2 network bit
```

kell.

Ezért:

```text
11000000
```

A bináris skála:

```text
128 64 32 16 8 4 2 1
 1   1  0  0 0 0 0 0
```

```text
128 + 64 = 192
```

Ezért:

```text
/26
=
255.255.255.192
```

---

# 15. Fontos különbség: mask és blokkméret

`/26` esetén:

```text
Subnet mask utolsó octetje:
192
```

de:

```text
blokkméret:
64
```

Ez a két szám nem ugyanaz.

A `192` innen jön:

```text
11000000
=
128 + 64
=
192
```

A `64` pedig innen:

```text
6 host bit
↓
2^6 (2 a 6dikon)
↓
64 cím
```

Mental model:

```text
192
→ subnet mask

64
→ subnet mérete
```

---

# 16. A házas hasonlat – /26

A `/24` nagy épülete:

```text
0–255
```

`/26` esetén ezt 4 kisebb épületre bontjuk.

Mindegyikben:

```text
64 cím
```

lehet.

A négy subnet:

```text
1. épület:
0–63

2. épület:
64–127

3. épület:
128–191

4. épület:
192–255
```

Ezt azért kapjuk, mert két network bit:

```text
2^2 = 4
```

különböző kombinációt ad.

---

# 17. Network bitek /26 esetén

A két network bit lehetséges értékei:

```text
00
01
10
11
```

Ezekhez tartoznak:

```text
00 → 0–63

01 → 64–127

10 → 128–191

11 → 192–255
```

Mindegyik subnetben:

```text
6 host bit
```

van.

Ez:

```text
2^6 = 64 cím
```

---

# 18. /26 példa – 192.168.10.77

Legyen:

```text
IP:
192.168.10.77

CIDR:
/26
```

Tudjuk:

```text
/26
→ 64-es blokkok
```

Ezért:

```text
0–63
64–127
128–191
192–255
```

A `77` ebbe esik:

```text
64–127
```

Tehát:

```text
Network:
192.168.10.64

Broadcast:
192.168.10.127
```

Használható hostok:

```text
192.168.10.65–126
```

---

# 19. A host-rész értéke

A teljes utolsó octet:

```text
77
```

A subnet kezdete:

```text
64
```

Ezért:

```text
77 - 64
=
13
```

A host rész értéke tehát:

```text
13
```

Ez nem azt jelenti, hogy a teljes IP:

```text
192.168.10.13
```

Hanem:

> a host ezen a subneten belül 13-as offsettel rendelkezik.

Mental model:

```text
subnet start + host part
=
full IP
```

Itt:

```text
64 + 13 = 77
```

---

# 20. A házas példa ugyanerre

Képzeljük el:

```text
/24
=
egy nagy 256 címes épület
```

`/26` esetén:

```text
4 kisebb épület
```

Mindegyik:

```text
64 címes
```

A `77` cím:

```text
64–127
```

épületben van.

A ház „kezdőcíme”:

```text
64
```

A host:

```text
13
```

Mert:

```text
64 + 13 = 77
```

Fontos:

Mindegyik subnetben lehet ugyanaz a host-rész.

Például host-rész:

```text
13
```

lehet:

```text
0   + 13 = 13
64  + 13 = 77
128 + 13 = 141
192 + 13 = 205
```

Tehát ugyanaz a host-rész négy külön subnetben négy külön teljes IP-címet jelenthet.

---

# 21. /27

`/27` esetén:

```text
27 network bit
5 host bit
```

Utolsó octet:

```text
NNNHHHHH
```

---

## Mask

```text
11100000
```

```text
128 + 64 + 32
=
224
```

Ezért:

```text
/27
=
255.255.255.224
```

---

## Blokkméret

```text
5 host bit
↓
2^5
↓
32 cím
```

Ezért:

```text
0–31
32–63
64–95
96–127
128–159
160–191
192–223
224–255
```

---

# 22. /27 példa

Legyen:

```text
10.10.5.142/27
```

A blokkok:

```text
0–31
32–63
64–95
96–127
128–159
160–191
192–223
224–255
```

A `142`:

```text
128–159
```

blokkban van.

Ezért:

```text
Network:
10.10.5.128

Broadcast:
10.10.5.159
```

Host-rész:

```text
142 - 128 = 14
```

Használható hostok:

```text
10.10.5.129–158
```

---

# 23. /28

`/28`:

```text
28 network bit
4 host bit
```

Mask:

```text
11110000
```

```text
128 + 64 + 32 + 16
=
240
```

Ezért:

```text
/28
=
255.255.255.240
```

Blokkméret:

```text
2^4 = 16
```

Subnetek:

```text
0–15
16–31
32–47
48–63
64–79
80–95
96–111
112–127
128–143
144–159
160–175
176–191
192–207
208–223
224–239
240–255
```

---

# 24. Gyors CIDR táblázat

```text
CIDR   Mask                  Host bit   Cím/subnet

/24    255.255.255.0            8         256
/25    255.255.255.128          7         128
/26    255.255.255.192          6          64
/27    255.255.255.224          5          32
/28    255.255.255.240          4          16
/29    255.255.255.248          3           8
/30    255.255.255.252          2           4
```

---

# 25. Gyors mask táblázat – utolsó octet

```text
10000000 = 128
11000000 = 192
11100000 = 224
11110000 = 240
11111000 = 248
11111100 = 252
11111110 = 254
11111111 = 255
```

Ezért:

```text
/25 → 128
/26 → 192
/27 → 224
/28 → 240
/29 → 248
/30 → 252
```

---

# 26. Ha CIDR-t kapunk

Példa:

```text
192.168.50.77/26
```

Lépések:

```text
1. /26
2. 32 - 26 = 6 host bit
3. 2^6 = 64-es blokkméret
4. blokkok:
   0–63
   64–127
   128–191
   192–255
5. 77 → 64–127 blokk
```

Ezért:

```text
Network:
192.168.50.64

Broadcast:
192.168.50.127

Usable:
192.168.50.65–126
```

---

# 27. Ha teljes subnet maskot kapunk

Példa:

```text
IP:
192.168.50.77

Mask:
255.255.255.192
```

Az első három:

```text
255
```

teljes network octet.

Ez:

```text
24 network bit
```

Az utolsó octet:

```text
192
```

Binárisan:

```text
128 64 32 16 8 4 2 1
 1   1  0  0 0 0 0 0
```

Ez:

```text
11000000
```

Két további network bit.

Ezért:

```text
24 + 2 = 26
```

Tehát:

```text
255.255.255.192
=
/26
```

Innen már használható a megszokott blokkmódszer:

```text
/26
→ 64-es blokkok
```

---

# 28. Subnet maskok fontos szabálya

A subnet mask bináris alakjában az `1` biteknek folyamatosan kell követniük egymást.

Például:

```text
11100000
```

szabályos.

Ez:

```text
224
```

Viszont:

```text
11010000
```

nem hagyományos CIDR subnet mask, mert az `1` bitek megszakadnak.

Tehát például:

```text
255.255.255.224
```

érvényes `/27`.

De:

```text
255.255.255.208
```

nem szabályos CIDR subnet mask.

---

# 29. Same subnet ellenőrzése

Két host local módon tud egymással kommunikálni, ha ugyanahhoz a subnethez tartoznak.

Például:

```text
192.168.1.10/24
192.168.1.50/24
```

Mindkettő hálózata:

```text
192.168.1.0/24
```

Tehát:

```text
same subnet
```

---

Másik példa:

```text
192.168.1.10/24
192.168.2.50/24
```

Networkek:

```text
192.168.1.0/24
192.168.2.0/24
```

Tehát:

```text
different subnet
```

Ilyenkor:

```text
gateway/router
```

szükséges.

---

# 30. IP AND subnet mask

A gép bitenkénti **AND** művelettel is meghatározhatja a network address-t.

Példa:

```text
IP:
77
=
01001101

Mask:
192
=
11000000
```

AND:

```text
01001101
11000000
────────
01000000
```

Ez:

```text
64
```

Tehát:

```text
192.168.10.77
AND
255.255.255.192
=
192.168.10.64
```

Mental model:

```text
IP AND subnet mask
=
network address
```

A mindennapi kézi számoláshoz azonban általában egyszerűbb a blokkméret alapján gondolkodni.

---

# 31. Private IPv4 tartományok

A private IPv4 hálózatok:

```text
10.0.0.0/8
```

```text
172.16.0.0/12
```

```text
192.168.0.0/16
```

Ezek belső hálózatokban használhatók.

Például:

```text
10.10.0.20
172.20.5.4
192.168.1.100
```

---

# 32. Public IPv4 cím

A **public IP** interneten routolható cím.

Egyszerű kép:

```text
Private host
192.168.1.20
↓
Router / NAT
↓
Public IP
↓
Internet
```

A NAT működését a:

```text
03-routing-gateway-nat.md
```

jegyzetben bontjuk ki.

---

# 33. IPv4 problémája: kevés cím

IPv4 összesen:

```text
2^32
```

különböző címet képes ábrázolni.

Ez körülbelül:

```text
4,3 milliárd cím
```

Ez a modern internet számára kevés.

Ez volt az IPv6 egyik fő létrejöttének oka.

---

# 34. IPv6

Az **IPv6** az IP protokoll újabb címzési rendszere.

Egy IPv6 cím:

```text
128 bit
```

hosszú.

Ez jóval nagyobb címtartományt ad, mint az IPv4.

---

# 35. IPv6 cím formátuma

Az IPv6 nem decimális számokat használ úgy, mint az IPv4.

Hexadecimális formában írjuk.

Példa:

```text
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

Nyolc darab csoportból áll.

Minden csoport:

```text
16 bit
```

azaz 4 hexadecimális karakter.

```text
8 × 16 bit
=
128 bit
```

---

# 36. Hexadecimális számrendszer

A hexadecimal rendszer 16 különböző számjegyet használ:

```text
0 1 2 3 4 5 6 7 8 9 A B C D E F
```

Az értékek:

```text
A = 10
B = 11
C = 12
D = 13
E = 14
F = 15
```

Egy hexadecimális karakter:

```text
4 bit
```

Ezért IPv6 címeket sokkal rövidebben lehet hexadecimálisan ábrázolni, mint binárisan.

---

# 37. IPv6 rövidítés

IPv6 címek gyakran sok nullát tartalmaznak.

Példa:

```text
2001:0db8:0000:0000:0000:0000:0000:0001
```

A csoportok elején lévő nullák elhagyhatók:

```text
2001:db8:0:0:0:0:0:1
```

Egymást követő nullás csoportok egyszer egy címen belül rövidíthetők:

```text
::
```

Ezért:

```text
2001:db8::1
```

ugyanazt a címet jelentheti.

---

# 38. IPv6 loopback

IPv4 loopback:

```text
127.0.0.1
```

IPv6 megfelelője:

```text
::1
```

Ez a saját hostra mutat.

---

# 39. IPv6 link-local cím

IPv6-ban fontos a **link-local** címtartomány.

Tipikusan:

```text
fe80::/10
```

Ezek a címek a helyi hálózati linken használhatók.

Nem az interneten történő globális routingra szolgálnak.

Linuxon például gyakran láthatunk:

```text
fe80::...
```

címeket az interface-eken.

---

# 40. IPv6 Global Unicast

A globálisan routolható IPv6 címek jelentős része a:

```text
2000::/3
```

tartományba esik.

Ez nagyjából az IPv4 public címekhez hasonló szerepet tölt be.

---

# 41. IPv6 és subnetek

IPv6-ban is használunk CIDR prefixeket.

Például:

```text
2001:db8:1234:5678::/64
```

IPv6 hálózatoknál a:

```text
/64
```

nagyon gyakori subnet méret.

Ez:

```text
64 network bit
+
64 interface/host bit
```

jellegű felosztást jelent.

IPv6 subnetelésnél nem ugyanaz a gyakorlati gondolkodás szükséges, mint az IPv4 címhiány miatt kialakult aprólékos `/26`, `/27`, `/28` felosztásoknál.

---

# 42. IPv6-ban nincs klasszikus broadcast

IPv4-ban használunk broadcast címet.

Például:

```text
192.168.1.255
```

IPv6-ban nincs ugyanilyen klasszikus broadcast mechanizmus.

Helyette több esetben:

```text
multicast
```

kommunikációt használ.

Ez fontos különbség IPv4 és IPv6 között.

---

# 43. IPv4 vs IPv6

```text
IPv4
→ 32 bit
→ decimális
→ pl. 192.168.1.10
```

```text
IPv6
→ 128 bit
→ hexadecimális
→ pl. 2001:db8::1
```

IPv4:

```text
broadcast létezik
```

IPv6:

```text
klasszikus broadcast nincs
```

IPv4:

```text
címhiány jelentős probléma
```

IPv6:

```text
rendkívül nagy címtartomány
```

---

# 44. Kell-e IPv6 subnetelést most mélyen tudni?

Egyelőre nem.

A jelenlegi tanulási cél:

```text
IPv4 subnetelés stabil megértése
```

és IPv6-nál:

```text
címformátum
128 bit
hexadecimal
:: rövidítés
::1
link-local
/64
IPv4-től való fő különbségek
```

megértése.

A későbbi Linux/cloud/networking laborok során IPv6-tal újra találkozunk majd.

---

# 45. Gyakorlati subnet számolási módszer

## Ha CIDR-t kapsz

Példa:

```text
192.168.1.77/26
```

### 1.

Host bitek:

```text
32 - 26 = 6
```

### 2.

Blokkméret:

```text
2^6 = 64
```

### 3.

Tartományok:

```text
0–63
64–127
128–191
192–255
```

### 4.

A `77`:

```text
64–127
```

blokkba esik.

### 5.

Ezért:

```text
Network:
192.168.1.64

Broadcast:
192.168.1.127

Usable:
192.168.1.65–126
```

---

# 46. Ha subnet maskot kapsz

Példa:

```text
192.168.1.77

255.255.255.192
```

### 1.

Az utolsó mask octet:

```text
192
```

### 2.

Binárisan:

```text
11000000
```

### 3.

Két network bit van az utolsó octetben.

```text
24 + 2 = 26
```

Tehát:

```text
/26
```

### 4.

Innen:

```text
6 host bit
→ 64-es blokkok
```

Majd ugyanaz a számolás, mint CIDR esetén.

---

# 47. Rövid subnet mental model

```text
IP
→ konkrét cím
```

```text
Subnet mask / CIDR
→ megmondja, hol a network/host határ
```

```text
Host bits
→ megadják a subnet méretét
```

```text
Block
→ megmutatja, melyik subnetben van az IP
```

```text
Block eleje
→ network address
```

```text
Block vége
→ broadcast
```

```text
Köztes címek
→ usable host addresses
```

---

# 48. A házas mental model összefoglalása

```text
/24
→ 1 nagy épület
→ 256 cím
```

```text
/25
→ 2 épület
→ 128 cím / épület
```

```text
/26
→ 4 épület
→ 64 cím / épület
```

```text
/27
→ 8 épület
→ 32 cím / épület
```

```text
/28
→ 16 épület
→ 16 cím / épület
```

Minden „épületben”:

```text
első cím
→ network
```

```text
utolsó cím
→ broadcast
```

```text
köztes címek
→ hostok
```

A teljes IP pedig elképzelhető így:

```text
subnet kezdete
+
host-rész
=
teljes cím
```

---

# 49. Rövid szakmai angol

> An IPv4 address contains 32 bits.

> An IPv6 address contains 128 bits.

> A subnet mask separates the network part from the host part.

> CIDR shows how many bits belong to the network.

> A /26 subnet contains 64 IPv4 addresses.

> The first address is the network address.

> The last address is the broadcast address.

> Hosts between them can normally be assigned to devices.

> IPv6 uses hexadecimal notation.

> IPv6 does not use broadcast in the same way as IPv4.

---

# 50. Amit ebből biztosan érteni kell

IPv4 esetén:

```text
IP address
+
subnet mask
↓
network membership
```

Meg kell tudni állapítani:

```text
melyik subnet?
mekkora a subnet?
mi a network address?
mi a broadcast?
mi a usable host range?
same subnet vagy different subnet?
```

Nem az a cél, hogy minden esetben fejben binárisan számolj.

A bináris rendszer azt segít megérteni:

> miért működik a subnet mask és a CIDR úgy, ahogy.

A gyakorlati számolásnál gyakran elég:

```text
CIDR
↓
host bits
↓
block size
↓
IP melyik blokkban van?
↓
network / broadcast / hosts
```

IPv6 esetén jelenleg az alapcél:

```text
128 bit
hexadecimal notation
:: compression
::1 loopback
link-local
/64
nincs klasszikus broadcast
```

megértése.
