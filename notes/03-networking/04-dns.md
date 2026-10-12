# DNS – Domain Name System

## 1. Mi az a DNS?

A **DNS** jelentése:

```text
Domain Name System
```

A DNS feladata, hogy az ember számára könnyen megjegyezhető neveket IP-címekhez kapcsolja.

Például:

```text
example.com
↓
93.184.216.34
```

A hálózati kommunikációhoz végül IP-címre van szükség.

Ezért a DNS egyik legfontosabb feladata:

```text
domain name
↓
IP address
```

Egyszerű mental model:

```text
DNS
→ név → IP
```

---

# 2. DNS és routing különbsége

A DNS és a routing nem ugyanaz.

## DNS

A kérdés:

```text
Mi az IP-címe ennek a névnek?
```

Például:

```text
api.example.com
↓
203.0.113.20
```

---

## Routing

A kérdés:

```text
Merre kell elküldeni a packetet ehhez az IP-címhez?
```

Tehát:

```text
DNS
→ name → IP

Routing
→ destination IP → path
```

A tipikus sorrend:

```text
domain name
↓
DNS lookup
↓
IP address
↓
routing
↓
destination
```

---

# 3. Honnan tudja a gép, melyik DNS szervert kérdezze?

A gépen van beállítva egy vagy több DNS-szerver.

Ezt megkaphatjuk például DHCP-n keresztül.

Otthoni hálózatban gyakori:

```text
PC
↓
DHCP
↓
IP address
gateway
DNS server
```

A DNS-szerver lehet például:

```text
192.168.1.1
```

ami maga az otthoni router.

De lehet külső DNS resolver is, például:

```text
1.1.1.1
8.8.8.8
```

A gép tehát tudja:

```text
ha DNS feloldás kell
→ ezt a szervert kérdezd
```

---

# 4. DNS resolver

A **DNS resolver** olyan DNS-szolgáltatás, amely a kliens számára megpróbálja megtalálni a szükséges választ.

A kliens például ezt kérdezi:

```text
Mi az example.com IP-címe?
```

A resolver:

```text
DNS resolver
```

megpróbálja megkeresni a választ.

Egyszerű folyamat:

```text
Client
↓
DNS resolver
↓
DNS system
↓
answer
↓
Client
```

---

# 5. DNS cache

Mielőtt a resolver hosszabb keresésbe kezdene, megnézi, van-e már eltárolt válasz.

Ez a:

```text
DNS cache
```

Például:

```text
example.com
→ 93.184.216.34
```

Ha ez még érvényes a cache-ben:

```text
cache hit
```

és a válasz azonnal visszaadható.

Ha nincs:

```text
cache miss
```

akkor új DNS feloldás szükséges.

---

# 6. DNS TTL

A DNS rekordokhoz tartozhat egy:

```text
TTL
```

érték.

A TTL jelentése:

```text
Time To Live
```

DNS esetén azt mondja meg:

> mennyi ideig tartható cache-ben a rekord.

Például:

```text
example.com
TTL = 300
```

Ez:

```text
300 másodperc
=
5 perc
```

Ezután a rekordot újra le kell kérdezni.

---

# 7. Fontos: DNS TTL és IP TTL nem ugyanaz

Ugyanaz a rövidítés:

```text
TTL
=
Time To Live
```

de két külön kontextusban használjuk.

## IP TTL

A packet élettartamát korlátozza router hopok alapján.

```text
64
↓ router
63
↓ router
62
```

Ha eléri:

```text
0
```

a packet eldobásra kerül.

---

## DNS TTL

A DNS rekord cache-élettartama.

```text
example.com → IP
TTL = 300 sec
```

Mental model:

```text
IP TTL
→ packet élettartam hopokban

DNS TTL
→ DNS cache élettartam időben
```

---

# 8. A DNS hierarchikus rendszer

A DNS nem egyetlen hatalmas szerver.

Hierarchikus, elosztott rendszer.

Például:

```text
www.example.com
```

DNS szempontból jobbról balra érdemes értelmezni:

```text
.
└── com
    └── example
        └── www
```

A szintek:

```text
.        → root
com      → TLD
example  → domain
www      → host / subdomain
```

---

# 9. Root

A DNS legfelső szintje:

```text
.
```

Ez a:

```text
DNS root
```

A teljesen pontos domainnév akár így is írható:

```text
www.example.com.
```

A végén lévő pont:

```text
.
```

a DNS rootot jelöli.

A gyakorlatban ezt általában elhagyjuk:

```text
www.example.com
```

---

# 10. TLD

A **TLD** jelentése:

```text
Top-Level Domain
```

Magyarul:

```text
legfelső szintű domain
```

Példák:

```text
.com
.hu
.org
.net
```

Például:

```text
example.com
```

esetén:

```text
.com
```

a TLD.

---

# 11. Domain hierarchia

Példa:

```text
api.dev.example.com
```

Felépítése:

```text
.
└── com
    └── example
        └── dev
            └── api
```

Egy másik példa:

```text
www.valami.hu
```

Felépítése:

```text
.
└── hu
    └── valami
        └── www
```

Mental model:

```text
root
↓
TLD
↓
domain
↓
subdomain / host
```

---

# 12. FQDN

A **FQDN** jelentése:

```text
Fully Qualified Domain Name
```

Magyarul:

```text
teljesen minősített domainnév
```

Példa:

```text
www.example.com
```

Ez pontosan meghatározza a host helyét a DNS hierarchiában.

Felbontva:

```text
www
→ host / subdomain

example
→ domain

com
→ TLD
```

Teljes DNS formában:

```text
www.example.com.
```

---

# 13. Authoritative DNS server

Az:

```text
authoritative DNS server
```

nem rövidítés.

Az **authoritative** jelentése ebben a kontextusban:

```text
hivatalos / mérvadó
```

Ez az a DNS-szerver, amely az adott domain hivatalos DNS-rekordjait kezeli.

Például:

```text
example.com
```

esetén az authoritative DNS server tudhatja például:

```text
www.example.com → 203.0.113.20
```

Ez tekinthető a domain DNS-adatainak hivatalos forrásának.

---

# 14. DNS feloldás teljes folyamata

Tegyük fel, hogy ezt szeretnénk elérni:

```text
www.example.com
```

A kliens megkérdezi a resolverét:

```text
Mi az IP-címe a www.example.com címnek?
```

A resolver először:

```text
cache lookup
```

Ha nincs találat:

```text
resolver
↓
root DNS
↓
TLD DNS
↓
authoritative DNS
↓
answer
```

---

# 15. Root DNS szerepe

A root DNS szerver nem feltétlenül tudja:

```text
www.example.com → IP
```

Viszont tudhatja:

```text
.com domainek ügyében
→ ezeket a TLD szervereket kérdezd
```

Tehát:

```text
Resolver
↓
Root DNS

"Hol találom a .com-ot?"

↓
.com TLD server
```

---

# 16. TLD DNS szerepe

A TLD szerver például a:

```text
.com
```

alá tartozó domainekhez tud irányt mutatni.

A resolver megkérdezi:

```text
Ki felel az example.com domainért?
```

A válasz lehet:

```text
example.com
→ authoritative nameserver
```

---

# 17. Authoritative DNS szerepe

A resolver végül megkérdezi az authoritative DNS servert:

```text
Mi a www.example.com A rekordja?
```

A válasz lehet:

```text
www.example.com
→ 203.0.113.20
```

Ezután a resolver visszaadja az IP-címet a kliensnek.

---

# 18. A teljes DNS lookup vizuálisan

```text
Client
↓
DNS Resolver
↓
cache?
├── YES
│   ↓
│ answer
│
└── NO
    ↓
   Root DNS
    ↓
   TLD DNS
    ↓
   Authoritative DNS
    ↓
   IP address
    ↓
   Resolver cache
    ↓
   Client
```

---

# 19. Recursive query

A kliens általában olyan DNS-választ szeretne, ahol a resolver intézi a keresést.

A kliens lényegében ezt mondja:

```text
"Keresd meg nekem a végső választ."
```

Ez:

```text
recursive query
```

Példa:

```text
Client
↓
DNS Resolver

"Mi az example.com IP-je?"
```

A resolver dolgozik tovább.

---

# 20. Iterative query

Az iterative lekérdezésnél a DNS-szerver nem feltétlenül ad végső választ.

Lehet, hogy csak ezt mondja:

```text
"Én nem tudom,
de ezt a szervert kérdezd."
```

Például:

```text
Resolver
↓
Root DNS
```

Root válasz:

```text
.com szerverek itt találhatók
```

Majd:

```text
Resolver
↓
.com TLD
```

TLD válasz:

```text
example.com authoritative DNS itt található
```

Majd:

```text
Resolver
↓
Authoritative DNS
```

Végül megkapjuk a tényleges rekordot.

---

# 21. Recursive és iterative együtt

A kliens szemszögéből:

```text
recursive query
```

A resolver háttérmunkája pedig lehet:

```text
iterative lookup
```

Egyszerűen:

```text
Client
↓
recursive request
↓
Resolver
↓
iterative DNS lookups
↓
Root
↓
TLD
↓
Authoritative
↓
Resolver
↓
Client
```

---

# 22. DNS rekordok

A DNS nem csak IP-címeket tárol.

Többféle:

```text
DNS record
```

létezik.

A legfontosabbak:

```text
A
AAAA
CNAME
MX
NS
TXT
```

---

# 23. A record

Az:

```text
A
```

rekord IPv4 címhez kapcsolja a nevet.

Példa:

```text
example.com
A
203.0.113.20
```

Mental model:

```text
A
→ IPv4
```

---

# 24. AAAA record

Az:

```text
AAAA
```

rekord IPv6 címhez kapcsolja a nevet.

Példa:

```text
example.com
AAAA
2001:db8::20
```

Mental model:

```text
AAAA
→ IPv6
```

---

# 25. CNAME record

A **CNAME** jelentése:

```text
Canonical Name
```

A CNAME egy alias.

Egy név egy másik névre mutathat.

Példa:

```text
www.example.com
CNAME
example.com
```

Tehát:

```text
www.example.com
↓
example.com
↓
A / AAAA record
↓
IP
```

Mental model:

```text
CNAME
→ name → another name
```

---

# 26. MX record

Az **MX** jelentése:

```text
Mail Exchange
```

Megmondja, melyik mail server kezeli a domain leveleit.

Példa:

```text
example.com
MX
mail.example.com
```

Mental model:

```text
MX
→ mail server
```

---

# 27. NS record

Az **NS** jelentése:

```text
Name Server
```

Az NS rekord megmutatja, melyik DNS-szerver authoritative az adott domainhez.

Példa:

```text
example.com
NS
ns1.example-dns.com
```

Mental model:

```text
NS
→ authoritative nameserver
```

---

# 28. TXT record

A TXT rekord szöveges információt tartalmazhat.

Például:

```text
example.com
TXT
"some verification value"
```

Gyakori felhasználások:

```text
domain verification
email security
SPF
DKIM related configuration
miscellaneous metadata
```

A részletes email DNS-beállítások későbbi témák lehetnek.

---

# 29. DNS rekordok röviden

```text
A
→ IPv4

AAAA
→ IPv6

CNAME
→ alias / another name

MX
→ mail server

NS
→ nameserver

TXT
→ text / metadata
```

---

# 30. DNS és a teljes hálózati folyamat

Ha a felhasználó beírja:

```text
https://example.com
```

egyszerűsítve:

```text
Browser
↓
DNS lookup
↓
example.com → IP
↓
destination IP known
↓
same subnet?
↓
default gateway
↓
routing
↓
internet
↓
destination server
```

Tehát:

```text
DNS
→ előkészíti a hálózati kapcsolatot az IP megkeresésével
```

---

# 31. Gyakorlati eszközök

DNS vizsgálatára nagyon gyakori eszközök:

```text
nslookup
dig
```

Windows alatt az:

```text
nslookup
```

általában alapból elérhető.

Linuxon / WSL-ben a:

```text
dig
```

nagyon hasznos.

---

# 32. nslookup

Egyszerű DNS lookup:

```bash
nslookup example.com
```

Példakimenet:

```text
Server:         192.168.1.1
Address:        192.168.1.1#53

Non-authoritative answer:
Name:   example.com
Address: 93.184.216.34
```

---

# 33. nslookup kimenetének értelmezése

```text
Server: 192.168.1.1
```

Ez az a DNS resolver, amelyet a kliens megkérdezett.

```text
Address: 192.168.1.1#53
```

A:

```text
#53
```

azt jelzi, hogy a DNS szolgáltatás a:

```text
53-as port
```

használatával érhető el.

A DNS jellemzően UDP és bizonyos esetekben TCP 53-at használ.

A portokat részletesen később:

```text
05-tcp-udp-ports-sockets.md
```

fejezetben vesszük át.

---

A:

```text
Non-authoritative answer
```

azt jelenti, hogy nem közvetlenül az adott domain authoritative DNS szerverétől kaptuk a választ.

A resolver például cache-ből is visszaadhatta.

---

# 34. Másik DNS szerver lekérdezése nslookup segítségével

Megadhatjuk azt is, melyik resolvert akarjuk használni.

Példa:

```bash
nslookup example.com 8.8.8.8
```

vagy:

```bash
nslookup example.com 1.1.1.1
```

Ezzel össze lehet hasonlítani különböző resolverek válaszait.

---

# 35. dig alap használata

Linux / WSL alatt:

```bash
dig example.com
```

Példakimenet:

```text
; <<>> DiG <<>> example.com
;; ->>HEADER<<- opcode: QUERY, status: NOERROR

;; QUESTION SECTION:
;example.com.        IN      A

;; ANSWER SECTION:
example.com.  300    IN      A       93.184.216.34

;; Query time: 22 msec
;; SERVER: 192.168.1.1#53
```

---

# 36. dig – QUESTION SECTION

```text
;; QUESTION SECTION:
;example.com. IN A
```

Ez azt jelenti:

```text
example.com
```

domainhez kérjük az:

```text
A
```

rekordot.

Tehát:

```text
Milyen IPv4 cím tartozik az example.com névhez?
```

---

# 37. dig – ANSWER SECTION

Példa:

```text
example.com. 300 IN A 93.184.216.34
```

Felbontva:

```text
example.com.
→ domain
```

```text
300
→ TTL
```

```text
IN
→ Internet class
```

```text
A
→ IPv4 rekord
```

```text
93.184.216.34
→ válasz
```

Mental model:

```text
NAME          TTL   CLASS   TYPE   VALUE

example.com   300   IN      A      93.184.216.34
```

---

# 38. dig – SERVER

A kimenet végén:

```text
SERVER: 192.168.1.1#53
```

Ez azt mutatja, melyik DNS resolvert kérdezte meg a `dig`.

Itt:

```text
192.168.1.1
```

a DNS server.

```text
53
```

a DNS port.

---

# 39. A rekord lekérdezése

IPv4:

```bash
dig example.com A
```

Például:

```text
example.com. 300 IN A 93.184.216.34
```

---

# 40. AAAA rekord lekérdezése

IPv6:

```bash
dig example.com AAAA
```

Például:

```text
example.com. 300 IN AAAA 2001:db8::20
```

---

# 41. NS rekord lekérdezése

```bash
dig example.com NS
```

Például:

```text
example.com. 86400 IN NS ns1.example-dns.com.
example.com. 86400 IN NS ns2.example-dns.com.
```

Ez azt mutatja:

```text
mely nameserverek authoritative-ok
```

az adott domainhez.

---

# 42. MX rekord lekérdezése

```bash
dig example.com MX
```

Például:

```text
example.com. 3600 IN MX 10 mail.example.com.
```

A:

```text
10
```

az MX prioritása.

Általában az alacsonyabb érték magasabb prioritást jelent.

---

# 43. TXT rekord lekérdezése

```bash
dig example.com TXT
```

Példa:

```text
example.com. 3600 IN TXT "example verification value"
```

Valós domaineknél itt több TXT rekord is lehet.

---

# 44. CNAME lekérdezése

```bash
dig www.example.com CNAME
```

Például:

```text
www.example.com. 300 IN CNAME example.com.
```

Ez azt jelenti:

```text
www.example.com
→ alias
→ example.com
```

---

# 45. Rövid dig kimenet

Ha csak a tényleges válasz érdekel:

```bash
dig example.com +short
```

Példakimenet:

```text
93.184.216.34
```

Ez scriptingnél különösen hasznos lehet.

---

# 46. DNS resolver megadása dig használatakor

Például Google DNS:

```bash
dig @8.8.8.8 example.com
```

Cloudflare:

```bash
dig @1.1.1.1 example.com
```

Az:

```text
@
```

után adjuk meg a DNS-servert.

Mental model:

```text
dig @DNS_SERVER DOMAIN
```

---

# 47. dig +trace

Az egyik legérdekesebb DNS parancs:

```bash
dig +trace example.com
```

Ez segít végignézni a DNS delegációs láncot.

Nagyon leegyszerűsített példakimenet:

```text
.               NS   a.root-servers.net.
.               NS   b.root-servers.net.

com.             NS   a.gtld-servers.net.
com.             NS   b.gtld-servers.net.

example.com.     NS   ns1.example-dns.com.

example.com.     A    93.184.216.34
```

---

# 48. dig +trace értelmezése

Első rész:

```text
.
→ root
```

Innen megtudjuk:

```text
ki kezeli a .com TLD-t?
```

Következő rész:

```text
com.
```

A TLD szerver megmutatja:

```text
ki authoritative az example.com domainhez?
```

Majd:

```text
example.com
```

authoritative DNS szervere visszaadja a tényleges rekordot.

Ez gyakorlatilag megmutatja ezt:

```text
Root
↓
TLD
↓
Authoritative DNS
↓
A record
```

---

# 49. Példa gyakorló labor

Később WSL alatt futtatható:

```bash
nslookup example.com
```

Majd:

```bash
dig example.com
```

Majd:

```bash
dig example.com A
```

```bash
dig example.com AAAA
```

```bash
dig example.com NS
```

```bash
dig example.com MX
```

```bash
dig example.com TXT
```

Végül:

```bash
dig +trace example.com
```

---

# 50. Mit érdemes figyelni a labor során?

Keressük meg:

```text
melyik DNS resolvert használja a gép?
```

```text
milyen A rekord érkezik?
```

```text
van-e AAAA rekord?
```

```text
mennyi a TTL?
```

```text
mely NS szerverek authoritative-ok?
```

```text
milyen útvonalat mutat a dig +trace?
```

Majd hasonlítsuk össze:

```bash
dig @1.1.1.1 example.com
```

és:

```bash
dig @8.8.8.8 example.com
```

válaszát.

---

# 51. DNS hibakeresés alapgondolata

Ha egy domain nem működik, az egyik első kérdés:

```text
Feloldódik-e egyáltalán IP-címre?
```

Például:

```bash
nslookup mydomain.com
```

vagy:

```bash
dig mydomain.com
```

Ha nincs megfelelő válasz, a probléma lehet DNS oldalon.

Ha van IP:

```text
DNS valószínűleg működik
↓
következő lépés lehet routing / port / service / HTTP vizsgálata
```

Ez később nagyon fontos troubleshooting gondolkodás lesz.

---

# 52. DNS teljes mental model

```text
User enters domain
↓
Client checks / asks resolver
↓
Resolver checks cache
↓
if needed:
Root
↓
TLD
↓
Authoritative DNS
↓
DNS record
↓
Resolver caches answer
↓
Client receives IP
↓
Routing begins
```

---

# 53. Legfontosabb rövidítések

```text
DNS
→ Domain Name System
```

```text
TLD
→ Top-Level Domain
```

```text
FQDN
→ Fully Qualified Domain Name
```

```text
NS
→ Name Server
```

```text
MX
→ Mail Exchange
```

```text
CNAME
→ Canonical Name
```

```text
TTL
→ Time To Live
```

---

# 54. Rövid szakmai angol

> DNS resolves domain names to IP addresses.

> A DNS resolver looks up DNS records for clients.

> DNS responses can be cached.

> TTL defines how long a DNS record may stay in cache.

> Root servers point resolvers toward TLD servers.

> TLD servers help locate authoritative name servers.

> An authoritative DNS server stores the official DNS records for a domain.

> An A record contains an IPv4 address.

> An AAAA record contains an IPv6 address.

> A CNAME record creates an alias.

> MX records identify mail servers.

> NS records identify authoritative name servers.

---

# 55. Amit ebből biztosan érteni kell

A fő folyamat:

```text
domain name
↓
DNS resolver
↓
cache?
↓
Root
↓
TLD
↓
Authoritative DNS
↓
IP address
```

A szerepek:

```text
Root
→ hol vannak a TLD szerverek?
```

```text
TLD
→ ki authoritative az adott domainhez?
```

```text
Authoritative DNS
→ mi a tényleges DNS rekord?
```

A legfontosabb rekordok:

```text
A
→ IPv4

AAAA
→ IPv6

CNAME
→ alias

MX
→ mail server

NS
→ nameserver

TXT
→ text metadata
```

És a három fontos hálózati fogalom továbbra is külön:

```text
DNS
→ name → IP
```

```text
Routing
→ IP → path
```

```text
NAT
→ address translation
```

Gyakorlati ellenőrzéshez:

```text
nslookup
dig
dig +trace
```

A `dig +trace` különösen hasznos, mert vizuálisan is megmutatja a DNS hierarchia működését:

```text
Root
↓
TLD
↓
Authoritative DNS
↓
final record
```