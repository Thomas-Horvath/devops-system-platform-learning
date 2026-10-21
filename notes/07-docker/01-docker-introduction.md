# Docker – bevezetés és alapfogalmak

## 1. Mi az a Docker?

A Docker egy platform, amelynek segítségével alkalmazásokat tudunk:

- csomagolni,
- futtatni,
- izolálni,
- szállítani,
- telepíteni,
- kezelni.

A Docker alapegysége a **container**.

A containerization lényege:

```text
alkalmazás
+
függőségek
+
runtime
+
konfiguráció
↓
egységesen futtatható környezet
```

A fő cél:

> ugyanaz az alkalmazás lehetőleg ugyanúgy fusson fejlesztői gépen, tesztkörnyezetben, szerveren és cloudban.

Ez csökkenti az olyan klasszikus problémákat, mint:

```text
„Nálam működik.”
„Nálam más Node verzió van.”
„Hiányzik egy library.”
„Más a szerver konfigurációja.”
```

A Docker tehát nem egyszerűen egy futtatóprogram.

Inkább egy olyan platform, amelynek segítségével definiálhatjuk és reprodukálhatjuk egy alkalmazás futási környezetét.

---

# 2. Mi az a container?

A container egy:

```text
izolált futtatási környezet
```

amelyben egy vagy több process fut.

Nagyon fontos:

```text
container ≠ virtuális gép
```

és:

```text
container ≠ külön teljes operációs rendszer
```

Linux containereknél a containerben futó processzek valójában:

```text
a host Linux kernelén futó processzek
```

csak különböző Linux izolációs mechanizmusok választják el őket a host többi processétől és más containerektől.

Egyszerűsítve:

```text
HOST LINUX

├── normál host process
├── normál host process
│
├── Container A
│   ├── nginx main process
│   ├── nginx worker
│   └── nginx worker
│
└── Container B
    └── node server.js
```

A kernel szempontjából ezek mind valódi processzek.

A container tehát nem maga egyetlen process.

Pontosabban:

```text
Container
=
izolált környezet processzek számára
```

---

# 3. A host és a container kapcsolata

A container nem kap saját Linux kernelt.

Ehelyett:

```text
Container A ─┐
             │
Container B ─┼──→ Host Linux Kernel
             │
Container C ─┘
```

Mindegyik container ugyanazt a host kernelt használja.

Ez az egyik legfontosabb különbség a container és a virtuális gép között.

---

# 4. Container vs virtuális gép

## Virtuális gép

Egy virtuális gép leegyszerűsített felépítése:

```text
Physical Hardware
↓
Host OS
↓
Hypervisor
↓
Virtual Machine
├── Guest OS
├── Guest Kernel
├── Libraries
└── Application
```

A virtuális gép saját operációs rendszerrel és saját kernellel rendelkezik.

Például:

```text
Host
├── VM 1
│   ├── Ubuntu
│   ├── Linux Kernel
│   └── Application
│
└── VM 2
    ├── Debian
    ├── Linux Kernel
    └── Application
```

---

## Container

Containereknél:

```text
Physical Hardware
↓
Host OS
↓
Host Linux Kernel
↓
Docker / Container Runtime
├── Container A
├── Container B
└── Container C
```

A containerek:

```text
nem kapnak külön kernelt
```

hanem:

```text
megosztják a host kernelét
```

---

# 5. Miért könnyebb egy container, mint egy VM?

Egy VM-nek általában szüksége van:

```text
Application
Libraries
Guest OS
Guest Kernel
```

rétegekre.

A container inkább:

```text
Application
Libraries
Userspace
```

rétegeket tartalmaz, miközben a kernel:

```text
shared host kernel
```

Ezért a containerek általában:

- gyorsabban indulnak;
- kevesebb memóriát használnak;
- kisebbek;
- könnyebben létrehozhatók;
- könnyebben törölhetők;
- egyszerűbben skálázhatók.

Egy VM indulásakor egy teljes operációs rendszer bootolhat.

Egy container indulásakor lényegében:

```text
izoláció létrehozása
↓
process indítása
```

történik.

---

# 6. Kernel és userspace

Egy Linux rendszer nagyon leegyszerűsítve két nagy részre bontható:

```text
Linux
├── Kernel
└── Userspace
```

## Kernel

A kernel kezeli többek között:

- CPU használatot;
- memóriát;
- processzeket;
- filesystem műveleteket;
- hálózatot;
- hardvereszközöket.

Mental model:

```text
Application
↓
system call
↓
Linux kernel
↓
CPU / RAM / disk / network
```

---

## Userspace

A userspace-ben találhatók például:

```text
bash
ls
cat
nginx
node
python
library-k
konfigurációs fájlok
alkalmazások
```

---

# 7. Mit jelent akkor az, hogy Ubuntu vagy Alpine container?

Ez elsőre megtévesztő lehet.

Ha futtatunk egy:

```text
Ubuntu container
```

kifejezést, az nem azt jelenti, hogy a container saját Ubuntu kernelt futtat.

Inkább:

```text
Ubuntu userspace
+
Host Linux Kernel
```

Hasonlóan:

```text
Alpine container
=
Alpine userspace
+
Host Linux Kernel
```

Például:

```text
Ubuntu host
↓
Linux kernel
↓
Alpine container userspace
```

Ez teljesen lehetséges.

---

# 8. Hogyan vannak a processzek izolálva?

A Docker több Linux kernel funkciót használ.

A legfontosabbak közül:

```text
Namespaces
Cgroups
Layered filesystem
```

Ezek különböző problémákat oldanak meg.

---

# 9. Namespace

A namespace fő kérdése:

> Mit láthat egy process a rendszerből?

A Linux többféle namespace-t biztosít.

Fontosabb példák:

```text
PID namespace
→ processzek izolációja
```

```text
NET namespace
→ hálózat izolációja
```

```text
MNT namespace
→ mountok / filesystem nézet izolációja
```

```text
UTS namespace
→ hostname izoláció
```

```text
IPC namespace
→ interprocess communication izoláció
```

A namespace-ek miatt a container processzei másképp láthatják a rendszert, mint a host.

---

# 10. PID namespace

A PID namespace a processzek nézetét izolálja.

Tegyük fel, hogy a hoston:

```text
PID 4200 → nginx master
PID 4210 → nginx worker
PID 4211 → nginx worker
```

Ugyanezek a processzek a container saját PID namespace-ében látszódhatnak így:

```text
PID 1  → nginx master
PID 10 → nginx worker
PID 11 → nginx worker
```

Tehát:

```text
ugyanaz a valódi process
↓
más PID namespace
↓
más PID szám
```

---

# 11. A container main process-e

Egy containerben van egy kiemelt process.

Ez tipikusan:

```text
PID 1
```

Ez a container:

```text
main process
```

fő process-e.

Node alkalmazásnál például:

```text
Container
└── PID 1: node server.js
```

Nginxnél:

```text
Container
├── PID 1: nginx master
├── PID 8: nginx worker
├── PID 9: nginx worker
└── PID 10: nginx worker
```

---

# 12. A main process létrehozhat további processzeket

A container fő process-e normál Linux process.

Ezért létrehozhat child processzeket.

Például:

```text
PID 1
├── child process
├── child process
└── child process
```

Nginx esetén:

```text
nginx master
├── nginx worker
├── nginx worker
└── nginx worker
```

Fontos:

> Ezek a child processzek is a host kernelén futnak.

A container csak izolálja őket.

---

# 13. A container életciklusa és a main process

A container életciklusa szorosan kapcsolódik a main processhez.

```text
main process fut
↓
container RUNNING
```

Ha a main process kilép:

```text
main process EXIT
↓
container STOPPED
```

Ez nagyon fontos Docker-alapelv.

A container nem úgy működik, mint egy VM, amelyben egy teljes operációs rendszer fut tovább.

A container központja a fő process.

---

# 14. Cgroups

A namespace azt szabályozza:

```text
mit láthat?
```

A cgroup ezzel szemben azt szabályozza:

```text
mennyit használhat?
```

A cgroups segítségével erőforrásokat lehet szabályozni.

Például:

```text
Container A
├── maximum memória
└── CPU limit
```

```text
Container B
├── más memória limit
└── más CPU limit
```

Így egy container nem feltétlenül használhatja korlátlanul a host összes erőforrását.

---

# 15. Namespace vs cgroup

Fontos különbség:

```text
Namespace
→ isolation
→ mit láthat?
```

```text
Cgroup
→ resource control
→ mennyit használhat?
```

Együtt:

```text
Namespaces
+
Cgroups
↓
izolált és szabályozható process environment
```

---

# 16. Mi az a Docker image?

A Docker image egy:

```text
sablon / csomag
```

amelyből containereket indíthatunk.

Az image tartalmazhatja például:

```text
alkalmazás kódja
runtime
library-k
függőségek
konfiguráció
filesystem tartalom
indítási metadata
```

Például egy saját Node alkalmazás image-e:

```text
My Node Image
├── base filesystem
├── Node runtime
├── npm dependencies
├── application code
└── configuration
```

---

# 17. Image vs container

Ez az egyik legfontosabb Docker fogalompár.

## Image

```text
IMAGE
→ sablon
→ tárolt
→ nem fut
```

## Container

```text
CONTAINER
→ image futó példánya
```

Mental model:

```text
Image
↓
docker run
↓
Container
```

---

# 18. Egyszerű analógia

Az image lehet:

```text
recept
```

A container:

```text
a receptből elkészített étel
```

IT-sabb analógiával:

```text
Image
→ class
```

```text
Container
→ object instance
```

Ezek nem tökéletes analógiák, de jól mutatják:

```text
egy image
↓
több container
```

kapcsolatát.

---

# 19. Egy image-ből több container

Például van egy:

```text
nginx image
```

Ebből indíthatunk:

```text
nginx image
├── nginx-container-1
├── nginx-container-2
├── nginx-container-3
└── nginx-container-4
```

Mindegyik ugyanabból a sablonból indul.

De különálló container instance-ok.

---

# 20. Image layer-ek

A Docker image nem feltétlenül egyetlen nagy filesystem-csomag.

Rétegekből:

```text
layers
```

épül fel.

Például:

```text
Layer 4
→ alkalmazás kódja

Layer 3
→ npm dependencies

Layer 2
→ Node runtime

Layer 1
→ base filesystem
```

Ezek együtt alkotják az image-et:

```text
┌──────────────────────┐
│ Application code     │
├──────────────────────┤
│ Dependencies         │
├──────────────────────┤
│ Runtime              │
├──────────────────────┤
│ Base filesystem      │
└──────────────────────┘
          ↓
        IMAGE
```

---

# 21. Miért hasznosak a layer-ek?

A rétegek újra felhasználhatók.

Tegyük fel, hogy:

```text
app-a
```

és:

```text
app-b
```

ugyanarra épül:

```text
node:22
```

image-re.

A közös rétegeket nem feltétlenül kell kétszer tárolni:

```text
             Node base layers
              /           \
             /             \
        App A layer     App B layer
             ↓             ↓
          Image A        Image B
```

Ennek előnye lehet:

- kisebb tárhelyigény;
- gyorsabb image pull;
- gyorsabb image build;
- build cache használata.

---

# 22. Az image layer-ek read-only-k

Az image rétegeit alapvetően:

```text
read-only
```

rétegekként kezeljük.

Egy futó container ezért nem módosítja közvetlenül az image-et.

Ez nagyon fontos.

---

# 23. Container writable layer

Amikor az image-ből containert indítunk, a container kap egy saját:

```text
writable layer
```

réteget.

Felépítés:

```text
┌──────────────────────────────┐
│ Container Writable Layer     │ ← írható
├──────────────────────────────┤
│ Image Layer 4                │ ← read-only
├──────────────────────────────┤
│ Image Layer 3                │ ← read-only
├──────────────────────────────┤
│ Image Layer 2                │ ← read-only
├──────────────────────────────┤
│ Image Layer 1                │ ← read-only
└──────────────────────────────┘
```

---

# 24. Mi kerül a writable layerbe?

Például ha a containerben:

```bash
echo hello > /tmp/test.txt
```

parancsot futtatunk, akkor az új fájl a container saját writable layerébe kerül.

Az image:

```text
nem változik
```

Tehát:

```text
IMAGE
read-only
```

és:

```text
CONTAINER
image
+
saját módosítások
```

---

# 25. Két container ugyanabból az image-ből

Két container megoszthatja ugyanazokat a read-only image layer-eket.

De mindkettőnek saját writable layer-e van.

```text
             IMAGE
       shared read-only
            layers
          /        \
         /          \
Writable A        Writable B
    ↓                 ↓
Container A       Container B
```

Ezért:

```text
Container A
```

módosítása nem jelenik meg automatikusan:

```text
Container B
```

filesystemében.

---

# 26. Példa

Container A-ban létrehozunk:

```text
/test.txt
```

Ez:

```text
Container A writable layer
```

részébe kerül.

Container B:

```text
nem fogja látni
```

mert saját writable layerrel rendelkezik.

---

# 27. Mi történik a container törlésekor?

A writable layer a containerhez tartozik.

Ha töröljük a containert:

```text
container rm
↓
container eltűnik
↓
writable layer eltűnik
```

Az abban található adatok tehát elveszhetnek.

---

# 28. Ephemeral data

A container saját írható rétegében lévő adat:

```text
ephemeral
```

jellegű.

Vagyis életciklusa a containerhez kötődik.

Ezért nem jó ötlet fontos adatot kizárólag itt tárolni.

Például:

```text
database data
uploaded images
user files
fontos alkalmazásadatok
```

esetén külön persistent storage-ra lesz szükség.

---

# 29. Persistent storage

Dockerben erre használjuk többek között:

```text
Docker Volume
```

vagy:

```text
Bind Mount
```

megoldásokat.

Mental model:

```text
Container
↓
törölhető
```

de:

```text
Volume
↓
megmaradhat
```

A storage témát külön fejezetben fogjuk részletesen megtanulni.

---

# 30. Layered filesystem mental model

A container számára a rétegek egyetlen normál filesystemnek látszanak.

Például a container csak ezt látja:

```text
/
├── bin
├── etc
├── usr
├── var
└── app
```

Nem azt látja, hogy:

```text
ez a fájl Layer 1-ből jött
ez Layer 3-ból
ez pedig a writable layerből
```

Ezt a Docker filesystem-réteg kezelése oldja meg.

---

# 31. Mi történik fájl módosításakor?

Tegyük fel, hogy az image-ben van:

```text
/etc/app.conf
```

A container módosítja.

Az image eredeti read-only változata nem változik meg.

A módosított változat a container írható rétegében jelenik meg.

Logikailag:

```text
Container Writable Layer
└── /etc/app.conf ← módosított
```

alatta pedig továbbra is létezik:

```text
Image Layer
└── /etc/app.conf ← eredeti
```

A container számára a felső, módosított változat látszik.

---

# 32. Immutable image gondolkodás

Dockeres környezetben az image-re érdemes úgy gondolni, mint:

```text
immutable artifact
```

Vagyis általában nem egy meglévő image-et szerkesztünk.

Ha változtatni akarunk az alkalmazáson:

```text
forrás / Dockerfile módosítás
↓
új build
↓
új image
↓
új container
```

Ez később CI/CD-ben különösen fontos lesz.

---

# 33. Docker fő komponensek

Alapszinten ezeket kell megkülönböztetni:

```text
Docker CLI
Docker Engine
Docker daemon
Image
Container
Registry
```

---

# 34. Docker CLI

Amikor terminálban kiadjuk:

```bash
docker run nginx
```

a:

```text
docker
```

parancs a Docker CLI.

CLI:

```text
Command Line Interface
```

A CLI nem maga futtatja közvetlenül a containert.

Feladata leegyszerűsítve:

```text
felhasználó
↓
docker CLI
↓
kérés a Docker Engine felé
```

---

# 35. Docker daemon

A Docker háttérben futó fontos komponense:

```text
dockerd
```

A daemon többek között kezeli:

```text
images
containers
networks
volumes
```

Mental model:

```text
Docker CLI
↓
Docker daemon
↓
Docker objects
```

---

# 36. Docker Engine

Egyszerűsítve:

```text
Docker CLI
↓
Docker Engine
↓
container runtime
↓
Linux kernel
```

A pontosabb belső architektúrát a következő fejezetben vizsgáljuk majd.

Ott előkerül:

```text
dockerd
containerd
runc
Docker API
```

---

# 37. Docker Registry

A Docker Registry:

```text
Docker image-ek tárolására szolgáló rendszer
```

Mental model:

```text
Registry
→ image repository
```

Ilyen például:

```text
Docker Hub
```

---

# 38. Docker Hub

A Docker Hub egy széles körben használt public container registry.

Találhatunk rajta például:

```text
nginx
node
postgres
redis
ubuntu
alpine
```

image-eket.

---

# 39. Pull és push

Image letöltése:

```text
Registry
↓
pull
↓
Local Docker Host
```

Image feltöltése:

```text
Local Docker Host
↓
push
↓
Registry
```

Parancsok:

```bash
docker pull nginx
```

és később:

```bash
docker push ...
```

---

# 40. Mi történik `docker run nginx` esetén?

Most rakjuk össze az eddigieket.

Parancs:

```bash
docker run nginx
```

Egyszerűsített folyamat:

```text
docker run nginx
↓
Docker CLI
↓
Docker Engine
↓
van local nginx image?
├── IGEN
│    ↓
│  image használata
│
└── NEM
     ↓
   Registry
     ↓
   image pull
     ↓
   local image store
↓
container létrehozása
↓
writable layer létrehozása
↓
namespace-ek létrehozása
↓
cgroup beállítások
↓
network konfiguráció
↓
main process indítása
↓
CONTAINER RUNNING
```

---

# 41. `docker pull`

A:

```bash
docker pull nginx
```

csak letölti az image-et.

```text
Registry
↓
nginx image
↓
local image store
```

Ettől még nem indul container.

---

# 42. `docker create`

A:

```bash
docker create nginx
```

létrehoz egy container objectet az image alapján.

De:

```text
container created
≠
container running
```

---

# 43. `docker start`

Meglévő container indítása:

```bash
docker start <container>
```

Tehát:

```text
existing container
↓
start
↓
main process indul
↓
RUNNING
```

---

# 44. `docker run`

A:

```bash
docker run nginx
```

leegyszerűsítve ezt kombinálja:

```text
docker create
+
docker start
```

Mental model:

```text
RUN
=
CREATE + START
```

Ehhez szükség esetén még image pull is társulhat.

---

# 45. Docker és Linux kapcsolata

A Docker rengeteg olyan fogalomra épül, amit Linuxból már ismerünk.

```text
Linux process
↓
container process
```

```text
PID
↓
PID namespace
```

```text
filesystem
↓
container filesystem
```

```text
mount
↓
mount namespace
```

```text
CPU / RAM
↓
cgroups
```

Ezért a Docker valójában nagyon erősen Linux-alapú technológia.

---

# 46. Docker és networking kapcsolata

A container kaphat:

```text
network namespace
interface
IP address
subnet
routing table
DNS configuration
```

Például később:

```text
Container
│
├── eth0
├── 172.18.0.2/16
└── default route
      ↓
Docker bridge
      ↓
Host
      ↓
Internet
```

Itt újra előkerülnek:

```text
interface
IP
CIDR
subnet
gateway
route
DNS
port
NAT
```

fogalmak.

A Docker networking tehát nem egy teljesen új világ.

Inkább:

```text
Linux networking
+
network namespaces
+
virtual interfaces
+
bridges
+
routing
+
NAT
```

---

# 47. Docker fő előnyei

A Docker egyik legfontosabb előnye:

```text
reproducibility
```

Vagyis a futási környezet reprodukálható.

Mental model:

```text
Developer Laptop
      │
      │ same image
      ↓
Test Server
      │
      │ same image
      ↓
Production Server
```

Ez jelentősen csökkenti a környezetek közötti eltéréseket.

---

# 48. Portability

Egy megfelelően elkészített image:

```text
local machine
↓
server
↓
cloud
```

környezetek között mozgatható.

Ez azonban nem azt jelenti, hogy minden image minden CPU-architektúrán és minden operációs rendszeren automatikusan működik.

A hordozhatóságot mindig az adott image platformtámogatása mellett kell érteni.

---

# 49. Isolation

A containerek izolációja segít abban, hogy külön alkalmazások:

```text
külön process environment
külön filesystem
külön network namespace
külön resource control
```

környezetben fussanak.

Például:

```text
Container A
→ Node 20

Container B
→ Node 22
```

anélkül, hogy ugyanazon hoston közvetlenül összeakadnának.

---

# 50. Scaling

Egy image-ből könnyen indíthatunk több container instance-t.

```text
          Image
       /    |    \
      /     |     \
Container Container Container
   A         B        C
```

Ez később alapja lesz:

```text
replicas
load balancing
orchestration
```

témáknak.

---

# 51. Docker nem egyenlő Kubernetes-szel

Fontos különbség.

Dockerrel elsősorban:

```text
image-et készítünk
containert futtatunk
networköt kezelünk
storage-ot kezelünk
```

Kubernetes inkább:

```text
sok container
sok service
több node
scaling
scheduling
self-healing
orchestration
```

problémákat kezel.

Először a Docker alapjait kell stabilan érteni.

---

# 52. Legfontosabb különbségek

## Image vs Container

```text
Image
→ sablon

Container
→ futó példány
```

---

## Container vs VM

```text
VM
→ saját kernel

Container
→ host kernel használata
```

---

## Namespace vs Cgroup

```text
Namespace
→ mit láthat?

Cgroup
→ mennyit használhat?
```

---

## Image layer vs writable layer

```text
Image layer
→ read-only
```

```text
Container writable layer
→ írható
→ containerhez tartozik
```

---

## Ephemeral vs persistent storage

```text
Writable layer
→ ephemeral
```

```text
Volume / bind mount
→ persistent storage lehet
```

---

# 53. Teljes Docker mental model

```text
Developer
↓
Application
↓
Dockerfile
↓
docker build
↓
IMAGE
├── read-only layer
├── read-only layer
├── read-only layer
└── metadata
↓
docker run
↓
CONTAINER
├── writable layer
├── PID namespace
├── network namespace
├── mount namespace
├── cgroup
├── main process PID 1
└── child processzek
↓
HOST LINUX KERNEL
```

---

# 54. Registryvel együtt

```text
Developer
↓
docker build
↓
Image
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

# 55. Docker alapfogalmak röviden

### Docker

```text
container platform
```

### Container

```text
izolált futtatási környezet processzek számára
```

### Image

```text
read-only sablon containerek létrehozásához
```

### Main Process

```text
a container fő process-e
tipikusan PID 1
```

### Namespace

```text
izolálja a rendszer különböző nézeteit
```

### Cgroup

```text
szabályozza a processzek erőforrás-használatát
```

### Image Layer

```text
image read-only filesystem rétege
```

### Writable Layer

```text
a container saját írható filesystem rétege
```

### Registry

```text
Docker image-ek tárolója
```

### Volume

```text
container élettartamától elválasztható persistent storage
```

---

# 56. Amit ne keverj össze

```text
Image
≠
Container
```

```text
Container
≠
Virtual Machine
```

```text
Container
≠
külön kernel
```

```text
Container
≠
egyetlen process
```

```text
Namespace
≠
Cgroup
```

```text
Writable Layer
≠
Persistent Storage
```

```text
Registry
≠
Docker Host
```

---

# 57. Gyors ismétlés

Ha Docker alapokról beszélünk, ezt a folyamatot érdemes fejben látni:

```text
Image
↓
sablon

docker run
↓

Container
↓
izolált process environment

Main process
↓
PID 1

Linux namespaces
↓
izoláció

Cgroups
↓
erőforrás-szabályozás

Writable layer
↓
container ideiglenes módosításai

Host Linux Kernel
↓
valójában itt futnak a processzek
```

---

# 58. Végső összefoglalás

A Docker működésének alapja nem egy teljesen új operációs rendszer-technológia.

A Docker erősen Linux-alapú.

A container lényegében:

```text
processzek
+
namespace izoláció
+
cgroup resource control
+
izolált filesystem nézet
+
network namespace
```

A processzek:

```text
a host kernelén futnak
```

Az image:

```text
read-only sablon
```

A container:

```text
image
+
saját writable layer
+
futó processzek
+
izoláció
```

A legfontosabb mental model:

```text
Dockerfile
↓
Image
↓
docker run
↓
Container
↓
Main Process (PID 1)
↓
Host Linux Kernel
```

És az egyik legfontosabb Docker-elv:

> Ne úgy gondolj a containerre, mint egy könnyű virtuális gépre. Gondolj rá inkább úgy, mint egy izolált környezetre, amelyben a host kernelén futó processzek dolgoznak.