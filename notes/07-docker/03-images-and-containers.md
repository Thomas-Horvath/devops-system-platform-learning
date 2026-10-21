# Docker – Images és Containers

## 1. Miért kell különválasztani az image és container fogalmát?

A Docker egyik legfontosabb alapja, hogy pontosan külön tudjuk választani:

```text
Docker Image
```

és:

```text
Docker Container
```

fogalmát.

A kettő szorosan összetartozik, de nem ugyanaz.

A legegyszerűbb kapcsolat:

```text
Image
↓
docker run
↓
Container
```

Az image egy:

```text
sablon
```

amelyből containert hozunk létre.

A container pedig:

```text
az image egy konkrét példánya
```

amelyhez saját:

```text
állapot
filesystem módosítások
network konfiguráció
processzek
metadata
```

tartozhat.

---

# 2. Mi az a Docker image?

A Docker image egy:

```text
read-only rétegekből felépülő csomag
```

amely tartalmazza mindazt, ami egy alkalmazás containerben történő futtatásához szükséges lehet.

Például:

```text
alkalmazás kódja
runtime
library-k
függőségek
filesystem tartalom
alapértelmezett konfiguráció
indítási metadata
```

Egy Node.js alkalmazás image-e például logikailag így nézhet ki:

```text
Node Application Image
│
├── Linux userspace fájlok
├── Node.js runtime
├── npm dependencies
├── application source
└── image configuration
```

Fontos:

```text
az image nem fut
```

Az image csak egy tárolt artifact.

---

# 3. Az image nem operációs rendszer

Gyakran látunk ilyen image neveket:

```text
ubuntu
debian
alpine
```

Ez könnyen azt az érzést keltheti, hogy az image egy teljes operációs rendszer.

Valójában egy Linux container image:

```text
nem tartalmaz saját futó kernelt
```

Például:

```text
ubuntu image
=
Ubuntu userspace filesystem
+
konfiguráció
```

A kernel a host oldaláról érkezik.

Mental model:

```text
Ubuntu image
├── /bin
├── /etc
├── /usr
├── /lib
└── egyéb userspace fájlok

           +

Host Linux Kernel
```

Ezért:

```text
Ubuntu container
≠
Ubuntu virtuális gép
```

---

# 4. Image és container kapcsolata

Az image-ből container hozható létre.

```text
IMAGE
↓
CREATE
↓
CONTAINER
```

Például:

```bash
docker run nginx
```

esetén:

```text
nginx image
↓
container létrehozása
↓
nginx container
```

Ha ugyanazt a parancsot többször futtatjuk:

```bash
docker run nginx
docker run nginx
docker run nginx
```

nem ugyanaz a container indul el háromszor.

Hanem:

```text
1 db nginx image
↓
├── Container A
├── Container B
└── Container C
```

három külön container jön létre.

---

# 5. Egy image-ből sok container

Az image újra felhasználható.

Például:

```text
nginx:latest
```

image alapján létrehozhatunk:

```text
web-1
web-2
web-3
web-4
```

containereket.

Mindegyik ugyanazokra az image layer-ekre épülhet.

De mindegyik containernek lehet saját:

```text
Container ID
name
writable layer
network namespace
IP cím
hostname
process state
environment
mount
```

beállítása.

---

# 6. Image mint sablon

Hasznos mental model:

```text
Image
=
Blueprint / Template
```

és:

```text
Container
=
Instance
```

Programozási analógiával:

```text
Class
↓
Object
```

Dockerben:

```text
Image
↓
Container
```

Ez nem teljesen pontos technikai megfeleltetés, de a kapcsolat megértéséhez hasznos.

---

# 7. Image layer-ek

A Docker image rétegekből:

```text
layers
```

épül fel.

Például:

```text
┌─────────────────────────────┐
│ Application source          │
├─────────────────────────────┤
│ npm dependencies            │
├─────────────────────────────┤
│ Node.js runtime             │
├─────────────────────────────┤
│ Base filesystem             │
└─────────────────────────────┘
```

Minden ilyen rész lehet egy vagy több image layer.

A teljes image ezeknek a rétegeknek az együttes nézete.

---

# 8. Miért réteges az image?

A réteges felépítés több előnyt ad.

Például:

```text
újrafelhasználhatóság
cache-elés
hatékony tárolás
gyorsabb image build
gyorsabb image transfer
```

Tegyük fel, hogy két alkalmazás ugyanazt a base image-et használja:

```text
node:22
```

A két image logikailag:

```text
              Node base layers
              /              \
             /                \
     Application A        Application B
```

A közös layer-eket nem szükséges minden image-hez teljesen külön tárolni.

---

# 9. Image layer-ek read-only tulajdonsága

Az image layer-ek:

```text
immutable
```

jellegűek.

Docker használat közben:

```text
read-only
```

rétegként kezeljük őket.

Ha egy container módosít egy fájlt, az eredeti image layer:

```text
nem változik meg
```

Ez nagyon fontos.

---

# 10. Miért fontos az immutability?

Ha az image futás közben folyamatosan módosulna, elveszne az egyik legfontosabb Docker előny:

```text
reproducibility
```

Ugyanaz az image:

```text
Developer machine
Test server
Production server
```

környezetben ugyanazt az alapállapotot biztosítja.

Ezért Dockeres gondolkodásban:

```text
nem javítjuk kézzel az image-et
```

hanem:

```text
forrás módosítás
↓
új build
↓
új image
```

---

# 11. Image ID

Minden Docker image tartalom alapján azonosítható.

A Docker image-hez tartozik egy:

```text
Image ID
```

Például:

```text
sha256:3f8a...
```

A Docker CLI gyakran rövidített formában jeleníti meg.

Például:

```text
IMAGE ID
3f8a91bc34fd
```

---

# 12. Content-addressable storage

A Docker image-ek tárolásánál fontos elv:

```text
content-addressable storage
```

Ez azt jelenti, hogy a tartalomhoz cryptographic digest kapcsolódik.

Mental model:

```text
tartalom
↓
SHA-256
↓
digest
```

Ha a tartalom megváltozik:

```text
más digest
```

keletkezik.

Ez segít:

```text
integritás ellenőrzésében
azonos tartalom felismerésében
rétegek újrafelhasználásában
```

---

# 13. Image digest

Egy image registryben digesttel is azonosítható.

Például:

```text
nginx@sha256:...
```

A digest:

```text
tartalomhoz kötött azonosító
```

Ez fontos különbség a taggel szemben.

---

# 14. Image tag

Gyakori image referencia:

```text
nginx:latest
```

Itt:

```text
nginx
```

a repository neve,

és:

```text
latest
```

a tag.

Általános forma:

```text
repository:tag
```

Példák:

```text
nginx:1.27
node:22
node:22-alpine
postgres:17
ubuntu:24.04
```

---

# 15. A tag nem verzióbiztos azonosító

Nagyon fontos:

```text
tag
≠
immutable content identifier
```

A tag egy név vagy mutató.

Például:

```text
myapp:latest
```

ma mutathat egy bizonyos image-re,

holnap pedig ugyanaz a tag egy új image-re.

Mental model:

```text
latest
↓
Image A
```

később:

```text
latest
↓
Image B
```

A tag tehát:

```text
mozgatható referencia
```

---

# 16. Tag vs digest

Tag:

```text
nginx:latest
```

kényelmes emberi referencia.

Digest:

```text
nginx@sha256:abcd...
```

konkrét image contentet azonosít.

Röviden:

```text
TAG
→ név / pointer
```

```text
DIGEST
→ tartalomhoz kötött azonosító
```

Reprodukálhatóság szempontjából a digest pontosabb.

---

# 17. Mi történik, ha nincs tag?

Ha ezt írjuk:

```bash
docker pull nginx
```

akkor a Docker általában úgy értelmezi:

```text
nginx:latest
```

Tehát:

```text
docker pull nginx
```

nagyjából:

```text
docker pull nginx:latest
```

jelentésű.

Ez azonban nem jelenti azt, hogy:

```text
latest = legfrissebb szoftververzió
```

A `latest` csak egy tag neve.

---

# 18. Image repository

Az image-ek registryben repositorykban vannak szervezve.

Például:

```text
nginx
node
postgres
redis
ubuntu
```

Egy repository alatt több tag lehet.

Például:

```text
node
├── 20
├── 20-alpine
├── 22
├── 22-alpine
└── latest
```

---

# 19. Teljes image referencia

Egy image referencia lehet például:

```text
docker.io/library/nginx:latest
```

Részei:

```text
docker.io
→ registry
```

```text
library/nginx
→ repository
```

```text
latest
→ tag
```

Egyszerűsített alakban csak ezt írjuk:

```text
nginx
```

és Docker kitöltheti az alapértelmezéseket.

---

# 20. Official Images

Docker Hubon bizonyos népszerű image-ek:

```text
Docker Official Images
```

formában érhetők el.

Például:

```text
nginx
node
postgres
redis
ubuntu
```

Ezek használatakor is fontos figyelni:

```text
tag
platform
frissítések
security
```

szempontokra.

---

# 21. Image manifest

Egy modern container image nem csak filesystem layer-ekből áll.

Tartozhat hozzá:

```text
manifest
```

is.

A manifest többek között leírja:

```text
milyen config tartozik az image-hez
milyen layer-ek alkotják
milyen digestek tartoznak hozzájuk
```

Mental model:

```text
Image Manifest
├── config reference
├── layer 1 digest
├── layer 2 digest
├── layer 3 digest
└── ...
```

---

# 22. Image configuration

Az image-hez tartozik konfigurációs metadata is.

Például:

```text
default command
entrypoint
environment variables
working directory
user
exposed ports metadata
```

Ez nem feltétlenül ugyanaz, mint a filesystem layer tartalma.

Mental model:

```text
IMAGE
├── filesystem layers
└── image config / metadata
```

---

# 23. Multi-platform image

Egy image név mögött több platformhoz tartozó image is lehet.

Például:

```text
linux/amd64
linux/arm64
```

A registry használhat:

```text
image index
```

vagy:

```text
manifest list
```

struktúrát.

Mental model:

```text
node:22
↓
Image Index
├── linux/amd64 image
├── linux/arm64 image
└── más platform
```

---

# 24. Miért fontos a platform?

Nem minden image ugyanarra a CPU architektúrára készült.

Például:

```text
amd64
arm64
```

különböző instruction setet használ.

Ezért:

```text
image
+
platform
```

kapcsolat fontos.

A Docker gyakran automatikusan a hostnak megfelelő platform image-et választja.

---

# 25. `docker image ls`

Helyi image-ek listázása:

```bash
docker image ls
```

vagy rövidebb régi/alias forma:

```bash
docker images
```

Tipikus output:

```text
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    abc123...      ...            ...
node         22        def456...      ...            ...
```

Fontos oszlopok:

```text
REPOSITORY
TAG
IMAGE ID
CREATED
SIZE
```

---

# 26. `docker pull`

Image letöltése registryből:

```bash
docker pull nginx
```

Konkrét tag:

```bash
docker pull nginx:1.27
```

Mental model:

```text
Registry
↓
manifest / metadata
↓
layers
↓
local image store
```

---

# 27. Mi történik `docker pull` közben?

Nagyon leegyszerűsítve:

```text
1. image referencia feloldása
↓
2. registry kapcsolat
↓
3. manifest lekérése
↓
4. szükséges layer-ek meghatározása
↓
5. hiányzó layer-ek letöltése
↓
6. digest ellenőrzés
↓
7. local image store
```

Ha egy layer már helyben megvan:

```text
nem szükséges újra letölteni
```

---

# 28. Mi az a Docker container?

A container egy image-ből létrehozott:

```text
konkrét runtime instance
```

A container tartalmazhat saját:

```text
container metadata
writable filesystem layer
network configuration
mount configuration
resource configuration
runtime state
process state
```

adatokat.

Mental model:

```text
Image
+
Container-specific configuration
+
Writable layer
+
Runtime state
=
Container
```

---

# 29. Container létrehozása

A:

```bash
docker create nginx
```

parancs létrehoz egy containert az image alapján.

A container:

```text
létezik
```

de még nem feltétlenül fut.

Állapot:

```text
CREATED
```

Ez jól mutatja:

```text
container
≠
running process
```

A container objektum létezhet úgy is, hogy jelenleg nem fut.

---

# 30. Container indítása

Meglévő container indítása:

```bash
docker start <container>
```

Ekkor:

```text
Container configuration
↓
runtime
↓
main process indul
```

Állapot:

```text
RUNNING
```

---

# 31. `docker run`

A:

```bash
docker run nginx
```

magas szinten:

```text
image biztosítása
+
container létrehozása
+
container indítása
```

Mental model:

```text
docker run
≈
pull if needed
+
create
+
start
```

---

# 32. Container writable layer

Amikor container jön létre, saját írható filesystem layer tartozhat hozzá.

```text
┌─────────────────────────────┐
│ Container Writable Layer    │
├─────────────────────────────┤
│ Image Layer 4               │
├─────────────────────────────┤
│ Image Layer 3               │
├─────────────────────────────┤
│ Image Layer 2               │
├─────────────────────────────┤
│ Image Layer 1               │
└─────────────────────────────┘
```

Az image layer-ek:

```text
read-only
```

A container felső layer-e:

```text
writable
```

---

# 33. Unified filesystem view

A container process nem külön layer-eket lát.

Hanem egyetlen:

```text
filesystem view
```

jelenik meg számára.

Például:

```text
/
├── bin
├── etc
├── usr
├── var
└── app
```

A container process számára lényegtelen, hogy az adott fájl:

```text
base image layerből
felső image layerből
vagy writable layerből
```

származik.

---

# 34. Copy-on-write

A Docker rétegezett filesystem egyik fontos elve:

```text
copy-on-write
```

Röviden:

> Amíg egy alsó read-only layerben lévő fájlt nem kell módosítani, addig használható közvetlenül onnan.

Ha módosítás történik:

```text
read-only lower layer file
↓
copy / upper writable layer
↓
modified version
```

A container számára ezután a felső verzió látszik.

---

# 35. Copy-on-write példa

Tegyük fel, hogy az image-ben van:

```text
/etc/app.conf
```

Az image layerben:

```text
/etc/app.conf
version 1
```

A container módosítja.

Logikailag:

```text
Writable Layer
└── /etc/app.conf → version 2

Image Layer
└── /etc/app.conf → version 1
```

A container ezt látja:

```text
/etc/app.conf
→ version 2
```

Az image továbbra is változatlan.

---

# 36. Új fájl létrehozása

Ha a container létrehoz egy új fájlt:

```bash
touch /tmp/test.txt
```

az:

```text
container writable layer
```

részévé válik.

Nem kerül vissza automatikusan az image-be.

---

# 37. Fájl törlése

A réteges filesystem miatt a törlés sem feltétlenül azt jelenti, hogy egy alsó image layerből fizikailag eltűnik a fájl.

A felső rétegben a filesystem jelezheti:

```text
ezt a fájlt a container nézetében tekintsük töröltnek
```

Az eredeti lower layer:

```text
változatlan
```

marad.

Ez a layered filesystem természetes következménye.

---

# 38. Container törlése

Container törlése:

```bash
docker rm <container>
```

A container saját writable layer-e is eltűnik.

Mental model:

```text
Container
├── runtime metadata
├── writable layer
└── runtime state
↓
docker rm
↓
eltűnik
```

Az image layer-ek:

```text
megmaradnak
```

amíg magát az image-et nem töröljük.

---

# 39. Miért veszélyes fontos adatot writable layerben tárolni?

Tegyük fel, hogy adatbázist futtatunk.

```text
PostgreSQL Container
↓
/var/lib/postgresql/data
```

Ha az adat kizárólag a container writable layerében van:

```text
container törlése
↓
adat törlődhet
```

Ezért persistent adathoz:

```text
volume
```

vagy:

```text
bind mount
```

szükséges.

---

# 40. Container filesystem vs persistent storage

Container writable layer:

```text
container lifetime
```

élettartamhoz kötött.

Volume:

```text
külön storage object
```

lehet.

Mental model:

```text
Container
├── Writable Layer
│   └── ideiglenes módosítások
│
└── Volume Mount
    └── tartós adat
```

---

# 41. Két container ugyanabból az image-ből

Tegyük fel:

```text
Image: myapp:1.0
```

Két container:

```text
Container A
Container B
```

Felépítés:

```text
             Shared Image Layers
                /           \
               /             \
     Writable Layer A   Writable Layer B
             ↓                 ↓
       Container A         Container B
```

Container A módosítása:

```text
nem jelenik meg automatikusan
```

Container B-ben.

---

# 42. Container izoláció

A külön writable layer csak az egyik izolációs elem.

A containernek lehet saját:

```text
PID namespace
network namespace
mount namespace
hostname
cgroup
filesystem view
```

Ezért két ugyanabból az image-ből induló container:

```text
közös image-et használhat
```

de runtime szinten:

```text
külön instance
```

---

# 43. Container ID

Minden container egyedi azonosítót kap.

Például:

```text
5e71a672c445...
```

A CLI gyakran rövidített ID-t mutat:

```text
5e71a672c445
```

A container ID:

```text
egy adott container instance-hoz tartozik
```

Nem az image-et azonosítja.

---

# 44. Container name

Containerhez nevet is rendelhetünk.

Például:

```bash
docker run --name web nginx
```

Ekkor:

```text
Container Name
=
web
```

A név segítségével:

```bash
docker stop web
```

vagy:

```bash
docker inspect web
```

parancsokat is használhatunk.

---

# 45. Ha nem adunk nevet

Ha nem adunk explicit nevet:

```bash
docker run nginx
```

a Docker automatikusan generálhat egy nevet.

Például valamilyen:

```text
adjective_name
```

jellegű formában.

A pontos generált név nem fontos.

A lényeg:

```text
minden containernek van azonosítható neve/ID-ja
```

---

# 46. Container metadata

Egy containerhez sok konfiguráció tartozik.

Például:

```text
ID
name
image
command
entrypoint
environment variables
network settings
mounts
resource limits
restart policy
user
hostname
state
```

Ezeket részletesen:

```bash
docker inspect <container>
```

paranccsal lehet vizsgálni.

---

# 47. Image configuration és container configuration

Fontos különbség.

Az image tartalmazhat alapértelmezett konfigurációt.

Például:

```text
default command
default environment
working directory
user
entrypoint
```

Container létrehozásakor ez:

```text
örökölhető
```

de sok esetben:

```text
felülírható
```

---

# 48. Példa alapértelmezett command felülírására

Tegyük fel az image alapértelmezett commandja:

```text
node server.js
```

De container indításakor adhatunk más commandot.

Mental model:

```text
Image default
↓
node server.js
```

de:

```text
docker run image other-command
```

esetén:

```text
Container runtime command
↓
other-command
```

A részletes `ENTRYPOINT` és `CMD` működést Dockerfile fejezetben tanuljuk.

---

# 49. Image environment vs runtime environment

Image-ben lehet alapértelmezett environment variable metadata.

Például:

```text
NODE_ENV=production
```

Container futtatásakor:

```bash
docker run -e NODE_ENV=development ...
```

felülírható.

Ezért:

```text
image default config
≠
container végleges runtime config
```

---

# 50. Image nem tartalmazhat futás közbeni state-et

Az image célja:

```text
reprodukálható alapállapot
```

Ezért futás közbeni dolgokat:

```text
active sessions
current process memory
runtime connections
live socket state
```

nem image-ként kezelünk.

Ezek:

```text
container runtime state
```

részei.

---

# 51. Image state vs container state

Image:

```text
statikus artifact
```

Container:

```text
runtime instance
```

Röviden:

```text
IMAGE
├── filesystem layers
└── metadata
```

```text
CONTAINER
├── image
├── writable layer
├── runtime config
├── process state
├── network state
└── mounts
```

---

# 52. Container állapotok

Egy container több állapotban lehet.

Gyakori állapotok:

```text
created
running
paused
restarting
exited
dead
```

A mindennapi használatban legfontosabbak:

```text
CREATED
RUNNING
EXITED
```

---

# 53. CREATED

Container létrejött:

```bash
docker create nginx
```

Állapot:

```text
CREATED
```

Van container object.

De:

```text
main process nem fut
```

---

# 54. RUNNING

Container elindult:

```bash
docker start <container>
```

Állapot:

```text
RUNNING
```

A container main process fut.

---

# 55. EXITED

Ha a main process leáll:

```text
main process exits
↓
container state
↓
EXITED
```

A container objektum ettől még:

```text
megmarad
```

Ez nagyon fontos.

```text
stopped container
≠
deleted container
```

---

# 56. Stopped container újraindítása

Egy exited container:

```bash
docker start <container>
```

paranccsal újraindítható.

Itt nem új container jön létre.

Ugyanaz a container:

```text
same Container ID
same writable layer
same metadata
```

indul újra.

---

# 57. `docker run` új containert készít

Ez gyakori félreértés.

Ha futtatjuk:

```bash
docker run nginx
```

majd még egyszer:

```bash
docker run nginx
```

akkor:

```text
nem ugyanaz a container indul újra
```

hanem:

```text
Container A
Container B
```

jön létre.

Ha meglévőt akarunk újraindítani:

```bash
docker start
```

használható.

---

# 58. Image és container élettartama

Image:

```text
független a belőle létrehozott containerektől
```

Container:

```text
egy konkrét instance
```

Törölhetünk containert úgy, hogy az image megmarad.

```text
Image
├── Container A
├── Container B
└── Container C
```

Ha:

```text
Container B törlés
```

akkor:

```text
Image
├── Container A
└── Container C
```

továbbra is létezhet.

---

# 59. Image törlése

Image törlés:

```bash
docker image rm <image>
```

vagy alias:

```bash
docker rmi <image>
```

A Docker nem mindig engedi image törlését, ha azt még olyan container referálja, amelyhez szükség van rá.

Ez védi a container-image kapcsolat integritását.

---

# 60. Dangling image

Build során előfordulhatnak olyan image-ek vagy layer-referenciák, amelyekhez már nem tartozik normál tag.

Ezeket gyakran:

```text
dangling
```

image-nek nevezzük.

Például listázáskor:

```text
<none>:<none>
```

jellegű bejegyzés jelenhet meg.

A cleanup témát később külön tanuljuk.

---

# 61. Image size

A:

```bash
docker image ls
```

által mutatott image size értelmezésénél fontos, hogy a layer-ek megosztottak lehetnek.

Ezért:

```text
két image méretének összeadása
```

nem feltétlenül jelenti azt, hogy pontosan annyi extra fizikai tárhelyet használnak.

A shared layer-ek:

```text
egyszer tárolhatók
```

és több image is hivatkozhat rájuk.

---

# 62. Container size

Container esetén két külön dolgot érdemes elkülöníteni:

```text
image layer-ek mérete
```

és:

```text
container writable layer mérete
```

A container futása közben például:

```text
log
cache
temporary file
generated data
```

kerülhet a writable layerbe.

---

# 63. Miért nem jó mindent a writable layerbe írni?

A writable layer:

```text
container-specific
ephemeral
```

és storage szempontból nem minden workloadhoz ideális.

Nagy mennyiségű:

```text
database data
persistent application state
uploads
backup
```

adatot általában nem ide akarunk tenni.

Erre később:

```text
volumes
bind mounts
```

szolgálnak.

---

# 64. Image pull és container create külön folyamat

Fontos fejben elkülöníteni:

```text
PULL
→ image beszerzése
```

```text
CREATE
→ container object létrehozása
```

```text
START
→ container process indítása
```

```text
RUN
→ magas szinten create + start
```

és szükség esetén image pull.

---

# 65. Image nincs helyben

Ha futtatjuk:

```bash
docker run nginx
```

és nginx image nincs local:

```text
docker run
↓
image lookup
↓
local image nincs
↓
registry pull
↓
container create
↓
container start
```

---

# 66. Image már helyben van

Ha az image local:

```text
docker run nginx
↓
local image megtalálva
↓
container create
↓
container start
```

Registry kapcsolat ebben az esetben nem feltétlenül szükséges.

---

# 67. Image cache

A Docker képes már helyben lévő layer-eket újra használni.

Például:

```text
Image A
└── Layer X
```

és:

```text
Image B
└── ugyanaz a Layer X
```

esetén a layer:

```text
nem szükséges kétszer tárolni
```

Ez Docker buildnél különösen fontos lesz.

---

# 68. Image build

Saját image-et tipikusan:

```text
Dockerfile
```

alapján buildelünk.

Például:

```bash
docker build -t myapp:1.0 .
```

Mental model:

```text
Dockerfile
+
build context
↓
docker build
↓
Image
```

A részletes build működés külön fejezet lesz.

---

# 69. Image build és container run külön fogalom

Nagyon fontos:

```text
docker build
≠
docker run
```

`build`:

```text
image-et hoz létre
```

`run`:

```text
container instance-t hoz létre és indít
```

Mental model:

```text
Source Code
↓
BUILD
↓
Image
↓
RUN
↓
Container
```

---

# 70. Image push

Saját image registrybe feltölthető.

Mental model:

```text
Local Image
↓
tag
↓
docker push
↓
Registry
```

Például:

```text
myregistry.example/myapp:1.0
```

A registry ezután más Docker host számára is elérhetővé teheti.

---

# 71. Image pull másik hoston

Másik gépen:

```text
Registry
↓
docker pull
↓
Local Image
↓
docker run
↓
Container
```

Ez adja a Docker egyik legfontosabb workflow-ját:

```text
build once
↓
distribute image
↓
run many places
```

---

# 72. Mutable container, immutable image

Egyszerűen:

```text
Image
→ immutable
```

```text
Container
→ runtime state változhat
```

A containerben:

```text
fájl létrejöhet
process indulhat
environment eltérhet
network konfiguráció eltérhet
```

de az alap image:

```text
nem változik
```

---

# 73. Miért ne módosítsunk kézzel production containert?

Technikailag lehetséges belépni egy containerbe és módosítani fájlokat.

De ha:

```text
container törlődik
```

a módosítás elveszhet.

Ráadásul:

```text
másik instance
```

nem fogja automatikusan megkapni ugyanazt a változtatást.

Ezért production gondolkodásban:

```text
ne kézzel patch-elj containert
```

hanem:

```text
source / Dockerfile módosítás
↓
new image
↓
new container deployment
```

---

# 74. Pets vs cattle mental model

Infrastruktúrában gyakran használják ezt az analógiát.

Régi szerver:

```text
PET
```

egyedi:

```text
kézzel javítgatjuk
hosszú ideig él
különleges
```

Containeres gondolkodás:

```text
CATTLE
```

ha rossz:

```text
eldobjuk
↓
újat indítunk image-ből
```

A hasonlat nem tökéletes, de jól mutatja az immutable infrastruktúra gondolkodást.

---

# 75. Container restart vs replacement

Két külön fogalom:

```text
restart
```

ugyanazt a containert indítja újra.

```text
replacement
```

új containert hoz létre ugyanabból vagy új image-ből.

Deployment rendszerekben gyakran:

```text
régi container
↓
új image
↓
új container
↓
régi container törlés
```

minta jelenik meg.

---

# 76. Container main process

A container működésének központja:

```text
main process
```

Tipikusan:

```text
PID 1
```

a container PID namespace-ében.

Például:

```text
Container
└── PID 1: node server.js
```

vagy:

```text
Container
├── PID 1: nginx master
├── worker
├── worker
└── worker
```

---

# 77. Main process exit

Ha a main process kilép:

```text
PID 1 exits
↓
container stops
```

Ezért olyan image esetén, amely csak egy rövid commandot futtat:

```bash
docker run alpine echo hello
```

a container:

```text
elindul
↓
echo lefut
↓
process kilép
↓
container EXITED
```

Ez teljesen normális.

---

# 78. Container nem feltétlenül hosszú életű

Nem minden container:

```text
webserver
database
```

jellegű hosszú futású service.

Lehet például:

```text
batch job
migration
build task
one-off command
```

is.

Példa:

```bash
docker run alpine echo hello
```

Itt a container csak addig él, amíg az:

```text
echo
```

process fut.

---

# 79. Image ENTRYPOINT és CMD

Egy image meghatározhat alapértelmezett indítási konfigurációt.

Két fontos Dockerfile fogalom:

```text
ENTRYPOINT
CMD
```

Ezekből áll össze, milyen process induljon a containerben.

Például:

```text
nginx image
↓
default process
↓
nginx
```

A részletes működés Dockerfile fejezetben lesz.

---

# 80. Container command felülírhatósága

Container indításakor bizonyos image defaultok felülírhatók.

Ezért:

```text
same image
```

különböző container runtime konfigurációval indítható.

Például:

```text
Image
↓
Container A → command A
Container B → command B
```

Ez is mutatja:

```text
image config
≠
container végleges runtime config
```

---

# 81. Image és secrets

Fontos security-elv:

```text
secretet ne bake-elj image-be
```

Például:

```text
password
API key
private token
private key
```

nem jó image layerbe építeni.

Miért?

Mert az image:

```text
tárolható
pusholható
cache-elhető
layer history része lehet
```

A futásidejű secretet inkább:

```text
runtime secret management
environment / secret mechanism
mounted secret
```

jellegű megoldással adjuk át.

---

# 82. Image és konfiguráció

Nem minden konfigurációt kell image-be égetni.

Jó különbségtétel:

```text
Application artifact
→ image
```

```text
Environment-specific configuration
→ runtime
```

Például ugyanaz az image:

```text
myapp:1.0
```

futhat:

```text
development
staging
production
```

környezetben különböző runtime configgal.

---

# 83. Reusable image

Ideális esetben:

```text
same image
```

használható több környezetben.

Nem:

```text
myapp-dev
myapp-test
myapp-prod
```

három külön kézzel módosított artifact.

Inkább:

```text
myapp:1.0
+
runtime configuration
```

különböző környezetenként.

---

# 84. `docker container ls`

Futó containerek:

```bash
docker container ls
```

rövidebben:

```bash
docker ps
```

Tipikus output:

```text
CONTAINER ID
IMAGE
COMMAND
CREATED
STATUS
PORTS
NAMES
```

Itt jól látható:

```text
Container
→ melyik image-ből készült
```

---

# 85. Leállított containerek listázása

Alap:

```bash
docker ps
```

csak futó containereket mutat.

Minden container:

```bash
docker ps -a
```

vagy:

```bash
docker container ls -a
```

Itt megjelennek:

```text
running
exited
created
```

állapotú containerek is.

---

# 86. `docker inspect`

Az image és container részletesen vizsgálható.

Container:

```bash
docker inspect <container>
```

Image:

```bash
docker inspect <image>
```

A két objektum inspect outputja eltér.

Container esetén például:

```text
State
Config
NetworkSettings
Mounts
HostConfig
```

részek lehetnek fontosak.

---

# 87. `docker image inspect`

Image-specifikus vizsgálat:

```bash
docker image inspect nginx
```

Megmutathat például:

```text
image ID
created
architecture
OS
config
layers
entrypoint
command
environment
```

információkat.

---

# 88. `docker container inspect`

Container:

```bash
docker container inspect web
```

Megmutathat többek között:

```text
Container ID
State
Image reference
Process configuration
Network configuration
Mounts
Environment
Resource settings
```

adatokat.

---

# 89. Image history

Image layer-ek történetét megvizsgálhatjuk:

```bash
docker image history <image>
```

Például:

```bash
docker image history nginx
```

Ez segít látni:

```text
milyen build lépésekből
```

épült fel az image.

Dockerfile tanulásnál különösen hasznos lesz.

---

# 90. Container writable layer nem új image

Ha egy futó containerben módosítunk valamit:

```text
container state változik
```

de:

```text
nem jön létre automatikusan új image
```

Ez fontos.

```text
container modification
≠
image build
```

---

# 91. `docker commit`

Docker képes létező container változásaiból image-et készíteni:

```bash
docker commit
```

De modern DevOps workflow-ban ez általában:

```text
nem az elsődleges image build módszer
```

Reprodukálható image-hez:

```text
Dockerfile
+
docker build
```

a kívánatos megközelítés.

---

# 92. Miért jobb a Dockerfile, mint a kézi `commit`?

Kézi container módosítás:

```text
nem feltétlenül dokumentált
nem könnyen reprodukálható
emberfüggő
```

Dockerfile:

```text
version controlled
olvasható
ismételhető
automatizálható
CI/CD-be illeszthető
```

Ezért:

```text
Infrastructure / image definition as code
```

irányba haladunk.

---

# 93. Image layer sharing több container között

Azonos image-ből futó containerek:

```text
nem kapnak külön teljes image másolatot
```

Mind használhatják ugyanazokat a read-only layer-eket.

```text
            Image Layers
        /        |        \
       /         |         \
Container A  Container B  Container C
   + RW A       + RW B       + RW C
```

Ez nagyon hatékony.

---

# 94. Image layer sharing több image között

Nem csak containerek oszthatnak közös layer-eket.

Külön image-ek is.

Például:

```text
node:22
```

alapból épül:

```text
app-a:1
app-b:1
app-c:1
```

Mind használhat közös base layer-eket.

---

# 95. Image és container teljes mental model

```text
REGISTRY
   ↓ pull
LOCAL IMAGE
│
├── Read-only Layer 1
├── Read-only Layer 2
├── Read-only Layer 3
└── Image Config
        ↓
     create
        ↓
CONTAINER
│
├── ugyanazok az image layer-ek
├── saját writable layer
├── saját metadata
├── saját network config
├── saját mount config
├── saját runtime state
└── saját processzek
```

---

# 96. Egy image-ből három container

```text
                    IMAGE
             ┌────────────────┐
             │ Read-only      │
             │ shared layers  │
             └────────────────┘
               /      |      \
              /       |       \
             ↓        ↓        ↓

      Container A  Container B  Container C
      ┌──────────┐ ┌──────────┐ ┌──────────┐
      │ RW A     │ │ RW B     │ │ RW C     │
      ├──────────┤ ├──────────┤ ├──────────┤
      │ PID 1    │ │ PID 1    │ │ PID 1    │
      │ Network A│ │ Network B│ │ Network C│
      └──────────┘ └──────────┘ └──────────┘
```

---

# 97. Image lifecycle

Egyszerű image lifecycle:

```text
Dockerfile / source
↓
BUILD
↓
Local Image
↓
TAG
↓
PUSH
↓
Registry
↓
PULL
↓
Másik Docker Host
↓
RUN
```

---

# 98. Container lifecycle

Egyszerű container lifecycle:

```text
Image
↓
CREATE
↓
CREATED
↓
START
↓
RUNNING
↓
STOP
↓
EXITED
↓
START
↓
RUNNING
```

Vagy:

```text
EXITED
↓
RM
↓
container megszűnik
```

---

# 99. Image lifecycle és container lifecycle együtt

```text
Dockerfile
↓
BUILD
↓
IMAGE
│
│ docker create
↓
CONTAINER
│
│ docker start
↓
RUNNING
│
│ docker stop
↓
EXITED
│
├── docker start → RUNNING
│
└── docker rm → DELETED
```

Az image ettől függetlenül:

```text
megmaradhat
```

---

# 100. Gyakori félreértések

## „Az image fut.”

Nem pontos.

Helyesen:

```text
Container fut az image alapján.
```

---

## „A container egy image másolat.”

Nem teljesen.

Pontosabban:

```text
Container
=
shared read-only image layers
+
saját writable layer
+
runtime configuration
+
process state
```

---

## „Ha leállítom a containert, elveszik.”

Nem.

```text
docker stop
```

után a container:

```text
EXITED
```

állapotban megmarad.

---

## „Ha újra `docker run`-t adok ki, ugyanaz indul.”

Nem.

```text
docker run
```

új container instance-t készít.

Meglévő containerhez:

```text
docker start
```

kell.

---

## „Ha containerben módosítok fájlt, az image is módosul.”

Nem.

A változás:

```text
container writable layer
```

részébe kerül.

---

## „Ha törlöm a containert, az image is törlődik.”

Nem.

```text
container delete
≠
image delete
```

---

## „A tag garantálja a konkrét tartalmat.”

Nem.

Tag:

```text
mozgatható referencia
```

Digest:

```text
konkrét content identity
```

---

# 101. Alap parancsok összefoglalva

Image listázása:

```bash
docker image ls
```

Image letöltása:

```bash
docker pull nginx
```

Image részletei:

```bash
docker image inspect nginx
```

Image history:

```bash
docker image history nginx
```

Container létrehozása:

```bash
docker create nginx
```

Container indítása:

```bash
docker start <container>
```

Új container létrehozása és indítása:

```bash
docker run nginx
```

Futó containerek:

```bash
docker ps
```

Minden container:

```bash
docker ps -a
```

Container részletei:

```bash
docker inspect <container>
```

Container leállítása:

```bash
docker stop <container>
```

Container törlése:

```bash
docker rm <container>
```

Image törlése:

```bash
docker image rm <image>
```

---

# 102. Image és container gyors összehasonlítás

| Tulajdonság | Image | Container |
|---|---|---|
| Mi ez? | Sablon / artifact | Runtime instance |
| Fut? | Nem | Futhat |
| Filesystem | Read-only layer-ek | Image layer-ek + writable layer |
| Processzek | Nincsenek | Lehetnek |
| Runtime state | Nincs | Van |
| Network | Nincs futó network state | Saját network konfiguráció lehet |
| ID | Image ID / digest | Container ID |
| Tag | Lehet | Nem image tag alapján azonosítjuk |
| Törlés | Image artifact törlődik | Instance és writable layer törlődik |
| Újrafelhasználás | Sok container alapja lehet | Egy konkrét instance |

---

# 103. Legfontosabb definíciók

## Docker Image

```text
Read-only, rétegekből és metadata-ból álló container artifact,
amelyből container instance hozható létre.
```

## Docker Container

```text
Egy image alapján létrehozott konkrét runtime instance,
saját konfigurációval, writable layerrel és futás közbeni állapottal.
```

## Image Layer

```text
Az image immutable filesystem rétege.
```

## Writable Layer

```text
A containerhez tartozó írható filesystem réteg.
```

## Tag

```text
Egy image-re mutató emberbarát, módosítható referencia.
```

## Digest

```text
Tartalomhoz kötött cryptographic identifier.
```

## Repository

```text
Kapcsolódó image verziók/tagek gyűjteménye egy registryben.
```

---

# 104. Rövid mental model

```text
Dockerfile
↓
BUILD
↓
IMAGE
├── read-only layers
├── config
└── metadata
↓
RUN
↓
CONTAINER
├── image layers
├── writable layer
├── runtime config
├── network
├── mounts
└── process
```

---

# 105. Registryvel együtt

```text
Developer
↓
docker build
↓
Local Image
↓
docker push
↓
Registry
↓
docker pull
↓
Docker Host
↓
docker run
↓
Container
```

---

# 106. Storage mental model

```text
IMAGE
├── Layer A
├── Layer B
└── Layer C
        ↓
      shared

Container A
├── A
├── B
├── C
└── Writable A

Container B
├── A
├── B
├── C
└── Writable B
```

---

# 107. Lifecycle mental model

```text
IMAGE
↓
docker create
↓
CREATED CONTAINER
↓
docker start
↓
RUNNING
↓
main process exits
↓
EXITED
↓
docker start
↓
RUNNING
```

vagy:

```text
EXITED
↓
docker rm
↓
DELETED
```

---

# 108. DevOps mental model

Dockeres üzemeltetésben általában nem ezt akarjuk:

```text
production container
↓
SSH / exec
↓
kézi módosítás
↓
„most már működik”
```

hanem ezt:

```text
source
↓
Dockerfile
↓
build
↓
versioned image
↓
registry
↓
deployment
↓
new container
```

Ez:

```text
reproducible
versionable
automatable
auditable
```

működést tesz lehetővé.

---

# 109. Kapcsolat az előző fejezetekkel

Az első fejezetben:

```text
Container
=
izolált process environment
```

fogalmat tanultuk.

A második fejezetben:

```text
Docker CLI
↓
Docker Engine
↓
runtime
↓
Linux kernel
```

architektúrát láttuk.

Ebben a fejezetben azt tettük hozzá:

```text
miből jön létre a container?
↓
Image
```

és:

```text
hogyan különül el az image és a futó instance?
```

A teljes kép:

```text
Dockerfile
↓
Image
↓
Docker Engine
↓
Container
↓
Main Process
↓
Linux Kernel
```

---

# 110. Végső összefoglaló

A Docker image:

```text
read-only
immutable
rétegekből áll
metadata tartozik hozzá
registryben tárolható
taggel és digesttel hivatkozható
```

A Docker container:

```text
egy image konkrét instance-a
saját writable layerrel rendelkezik
saját runtime konfigurációja lehet
saját processzei lehetnek
saját network állapota lehet
leállítható és újraindítható
törölhető az image törlése nélkül
```

A legfontosabb kapcsolat:

```text
Image
=
reprodukálható sablon

Container
=
ennek futásidejű példánya
```

És a legfontosabb Docker workflow:

```text
Source Code
↓
Dockerfile
↓
BUILD
↓
Image
↓
Registry
↓
RUN
↓
Container
```

A legfontosabb üzemeltetési gondolat pedig:

> Az image legyen reprodukálható és verziózott artifact; a container pedig legyen eldobható runtime instance, amelyet szükség esetén ugyanabból az image-ből újra létre tudunk hozni.