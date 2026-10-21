# Docker – architektúra

## 1. Miért fontos a Docker architektúrája?

Amikor kiadunk egy ilyen parancsot:

```bash
docker run nginx
```

kívülről úgy tűnhet, mintha a `docker` program egyszerűen elindítaná az nginx containert.

Valójában több komponens működik együtt:

```text
Docker CLI
Docker API
Docker daemon (dockerd)
containerd
containerd shim
OCI runtime
Linux kernel
container process
```

A Docker architektúra megértése azért fontos, mert később ebből fogjuk megérteni:

```text
hogyan indul el egy container
hogyan kommunikál a CLI a Docker Engine-nel
hol tárolódnak az image-ek
ki kezeli a containerek életciklusát
mi indítja el ténylegesen a processzt
hol jönnek létre a namespace-ek és cgroupok
hogyan lehet Docker hibákat rétegenként keresni
```

Ez különösen fontos lesz:

```text
Docker troubleshooting
networking
security
logging
CI/CD
Kubernetes
container runtime
```

témáknál.

---

# 2. A Docker magas szintű architektúrája

A Docker Engine kliens–szerver architektúrát használ.

A legegyszerűbb modell:

```text
Felhasználó
↓
Docker CLI
↓
Docker API
↓
Docker daemon
↓
container runtime réteg
↓
Linux kernel
↓
container process
```

Részletesebben:

```text
Felhasználó
↓
Docker CLI
↓
Docker API
↓
dockerd
↓
containerd
↓
containerd shim
↓
OCI runtime
↓
Linux kernel
↓
container process
```

Fontos, hogy ez elsősorban **logikai architektúra**.

A konkrét implementáció részletei Docker-verziótól és platformtól függhetnek.

---

# 3. Docker Client / Docker CLI

Amikor terminálban kiadjuk:

```bash
docker ps
```

vagy:

```bash
docker run nginx
```

akkor a:

```text
docker
```

programot használjuk.

Ez a:

```text
Docker CLI
```

vagy:

```text
Docker Client
```

A CLI jelentése:

```text
Command Line Interface
```

A Docker CLI feladata:

```text
felhasználói parancs
↓
Docker API kérés
↓
Docker daemon
```

A Docker CLI önmagában nem:

```text
hoz létre namespace-eket
állít be cgroupokat
indít közvetlenül container processzt
```

Hanem kérést küld a Docker Engine háttérkomponensei felé.

---

# 4. Docker kliens–szerver modell

A Docker kliens és a Docker daemon külön komponens.

Mental model:

```text
Docker CLI
    │
    │ request
    ↓
Docker daemon
    │
    │ response
    ↑
Docker CLI
```

Ez hasonló más kliens–szerver rendszerekhez.

Például:

```text
browser
↓
web server
```

vagy:

```text
psql client
↓
PostgreSQL server
```

Docker esetén:

```text
docker CLI
↓
dockerd
```

---

# 5. Docker API

A Docker CLI a Docker daemon funkcióit API-n keresztül éri el.

Egyszerűsítve:

```text
Docker CLI
↓
Docker API
↓
dockerd
```

A Docker API HTTP-alapú API.

Ez azért fontos, mert nem csak a Docker CLI tudja használni.

Például más programok is kommunikálhatnak vele:

```text
Docker Desktop
CI/CD rendszerek
management toolok
scriptek
SDK-k
egyedi alkalmazások
```

Tehát a CLI csak egy kliens a Docker API előtt.

---

# 6. Docker socket Linuxon

Natív Linux Docker Engine esetén a Docker kliens és daemon közötti kommunikáció tipikus endpointja:

```text
/var/run/docker.sock
```

Ez egy:

```text
Unix domain socket
```

Egyszerűsítve:

```text
docker CLI
↓
/var/run/docker.sock
↓
dockerd
```

A Unix socket segítségével két ugyanazon rendszeren futó process kommunikálhat egymással.

---

# 7. Mi az a Unix domain socket?

A Unix domain socket egy helyi IPC-mechanizmus.

IPC:

```text
Inter-Process Communication
```

Mental model:

```text
Process A
↕
Unix socket
↕
Process B
```

Docker esetén például:

```text
docker CLI
↕
docker.sock
↕
dockerd
```

Ez nem ugyanaz, mint amikor két gép TCP/IP hálózaton keresztül kommunikál.

---

# 8. Miért fontos a `docker.sock`?

A Docker socket nagyon erős jogosultságot biztosíthat.

Aki hozzáfér a Docker daemon API-jához, az például képes lehet:

```text
container indítására
container törlésére
volume mountolására
host könyvtár mountolására
network létrehozására
privileged container indítására
```

Ezért:

```text
/var/run/docker.sock
```

biztonsági szempontból különösen érzékeny.

Nem szabad úgy kezelni, mint egy közönséges socketet.

---

# 9. Docker daemon – `dockerd`

A Docker Engine egyik legfontosabb háttérkomponense:

```text
dockerd
```

Ez a Docker daemon.

A daemon hosszú ideig futó management process.

Feladata többek között:

```text
Docker API kérések fogadása
container objektumok kezelése
image-ek kezelése
networkök kezelése
volume-ok kezelése
registryvel való kommunikáció
runtime komponensek koordinálása
```

Egyszerűsített mental model:

```text
Docker CLI
↓
dockerd
↓
Docker resources
```

---

# 10. Docker daemon és container process

Nagyon fontos különbség:

```text
dockerd
≠
container process
```

A `dockerd`:

```text
Docker management daemon
```

A containerben futó például:

```text
nginx
node
postgres
redis
```

pedig az alkalmazás processze.

Például:

```text
HOST

├── dockerd
├── containerd
│
├── Container A
│   └── nginx
│
└── Container B
    └── node
```

---

# 11. Docker Engine

A:

```text
Docker Engine
```

nem egyszerűen a `dockerd` másik neve.

A Docker Engine egy teljes container platform.

A kliens–szerver architektúrához tartozik többek között:

```text
Docker CLI
Docker API
Docker daemon
runtime komponensek
```

A mindennapi beszédben gyakran lazán mondjuk:

```text
Docker Engine
Docker daemon
```

de technikailag nem pontosan ugyanazt jelentik.

A:

```text
dockerd
```

egy konkrét daemon process.

A:

```text
Docker Engine
```

pedig a teljesebb Docker runtime és management környezet.

---

# 12. Docker objectek

A Docker többféle objektumot kezel.

A legfontosabbak:

```text
Images
Containers
Networks
Volumes
```

Például:

```bash
docker image ls
```

kilistázza az image-eket.

```bash
docker container ls
```

kilistázza a containereket.

```bash
docker network ls
```

kilistázza a networköket.

```bash
docker volume ls
```

kilistázza a volume-okat.

---

# 13. Mi az a container runtime?

A Docker daemon magasabb szintű management feladatokat végez.

De a container tényleges futtatásához szükség van runtime komponensekre.

A runtime réteg felel azért, hogy a container konfigurációból valódi Linux process legyen.

Egyszerűsítve:

```text
Docker management
↓
container runtime management
↓
low-level runtime
↓
Linux kernel
↓
container process
```

---

# 14. `containerd`

A Docker Engine fontos runtime komponense:

```text
containerd
```

A `containerd` egy iparági szabványként széles körben használt container runtime.

Feladatai közé tartozhat:

```text
container lifecycle kezelése
runtime taskok kezelése
image-ek kezelése
image layer-ek kezelése
snapshotok kezelése
storage backend használata
```

Container lifecycle alatt például:

```text
create
start
stop
delete
```

műveleteket értünk.

---

# 15. `dockerd` és `containerd` kapcsolata

Logikai modell:

```text
dockerd
↓
containerd
```

A `dockerd` magasabb szintű Docker managementet végez.

A `containerd` alacsonyabb szinten a container lifecycle és runtime működéshez kapcsolódó feladatokat kezeli.

Fontos:

> Nem érdemes ezt egyszerű process parent-child kapcsolatként elképzelni.

Ez elsősorban:

```text
szolgáltatás
↓
szolgáltatás
```

jellegű együttműködés.

---

# 16. A `containerd` és az image-ek

A `containerd` nem csak futó containerekkel foglalkozhat.

Modern Docker Engine környezetben image storage feladatokat is kezelhet.

A containerd image store például:

```text
image manifest
image config
layer content
snapshot
```

adatokat kezelhet.

Modern Docker Engine verziókban a containerd image store egyre fontosabb szerepet kap.

Ezért a `containerd`-t nem érdemes egyszerűen így definiálni:

```text
container indító szolgáltatás
```

Pontosabban:

```text
container és image runtime infrastruktúra
```

része.

---

# 17. Mi az a snapshot?

A container filesystem rétegek kezelésénél szükség van arra, hogy image layer-ekből futtatható filesystem-nézet készüljön.

A `containerd` erre:

```text
snapshotter
```

mechanizmusokat használhat.

Mental model:

```text
Image layers
↓
Snapshotter
↓
Container filesystem view
```

A snapshotter kezeli, hogyan jelenjenek meg és hogyan tárolódjanak a filesystem layer-ek.

A storage működésével később külön foglalkozunk.

---

# 18. `containerd shim`

A `containerd` és a konkrét container task között jellemzően van egy köztes komponens:

```text
containerd shim
```

Egyszerűsítve:

```text
containerd
↓
containerd shim
↓
OCI runtime
↓
container process
```

A shim feladata többek között, hogy köztes réteget biztosítson a `containerd` és a container runtime/task között.

---

# 19. Miért létezik shim?

A shim egyik fontos szerepe, hogy a container process ne legyen közvetlenül a `containerd` életciklusához kötve.

Mental model:

```text
containerd
↓
shim
↓
container process
```

Ez lehetővé teszi, hogy a container task tovább fusson akkor is, ha a magasabb szintű management komponenssel valamilyen probléma történik.

A pontos implementáció runtime-tól és containerd verziótól függhet, de kezdő szinten ezt a szerepet érdemes megjegyezni.

---

# 20. OCI

A container világ egyik fontos szabványszervezete:

```text
OCI
```

jelentése:

```text
Open Container Initiative
```

Az OCI nyílt szabványokat definiál container technológiákhoz.

Két fontos terület:

```text
OCI Image Specification
OCI Runtime Specification
```

---

# 21. OCI Image Specification

Az OCI Image Specification meghatározza, hogyan épül fel egy szabványos container image.

Például:

```text
image manifest
configuration
layers
content digests
```

struktúrákhoz kapcsolódó szabályokat definiál.

Mental model:

```text
OCI Image Specification
↓
hogyan nézzen ki egy kompatibilis container image?
```

---

# 22. OCI Runtime Specification

Az OCI Runtime Specification meghatározza, hogyan kell egy container runtime-nak container környezetet létrehoznia és futtatnia.

Mental model:

```text
OCI Runtime Specification
↓
hogyan legyen egy container process létrehozva?
```

Például a runtime konfiguráció tartalmazhat információkat:

```text
process
root filesystem
mountok
namespace-ek
cgroup beállítások
environment
```

területekről.

---

# 23. `runc`

A Docker alapértelmezett low-level OCI runtime-ja jellemzően:

```text
runc
```

A `runc` feladata nagyon leegyszerűsítve:

```text
OCI runtime configuration
↓
Linux kernel funkciók használata
↓
container process létrehozása
```

A `runc` például részt vesz:

```text
namespace-ek létrehozásában
mountok beállításában
root filesystem előkészítésében
cgroup konfiguráció alkalmazásában
container process indításában
```

---

# 24. `runc` és a Linux kernel

A `runc` nem saját izolációs technológiát implementál.

A Linux kernel funkcióit használja.

Például:

```text
runc
↓
Linux kernel
├── namespaces
├── cgroups
├── mounts
├── process management
└── networking primitives
```

Tehát végül továbbra is:

```text
Linux kernel
```

biztosítja az izoláció nagy részét.

---

# 25. A `runc` nem marad feltétlenül futva

Ez nagyon fontos.

Nem ezt kell elképzelni:

```text
runc fut
↓
container fut
```

Hanem inkább:

```text
runc elindul
↓
container környezet létrejön
↓
container main process elindul
↓
runc befejezheti a saját feladatát
↓
container process tovább fut
```

A container main process:

```text
normál Linux process
```

amelyet a Linux kernel futtat.

---

# 26. Mi marad futva egy container mellett?

Tipikus architektúrában lehet például:

```text
dockerd
containerd
containerd shim
container main process
```

miközben a:

```text
runc
```

nem feltétlenül marad folyamatosan futva.

Egyszerűsített állapot:

```text
dockerd          → fut
containerd       → fut
shim             → fut
runc             → már befejezhette
nginx            → fut
```

---

# 27. Nem csak `runc` létezhet

A Docker és containerd más runtime-okat is képes lehet használni.

Ezért pontosabb mental model:

```text
containerd
↓
runtime shim
↓
OCI-kompatibilis runtime
```

ahol az OCI runtime gyakran:

```text
runc
```

de elméletileg lehet más kompatibilis runtime is.

---

# 28. Container process

A folyamat végén mindig egy valódi operációs rendszer processhez jutunk.

Például:

```text
nginx
node
python
postgres
```

A containerben futó process:

```text
nem emulált Docker process
```

hanem:

```text
valódi Linux process
```

A host Linux kernel futtatja.

---

# 29. Container process és PID namespace

Tegyük fel a hoston:

```text
PID 5421 → nginx
```

A container saját PID namespace-ében ugyanez lehet:

```text
PID 1 → nginx
```

Tehát:

```text
ugyanaz a kernel által kezelt process
↓
külön PID namespace
↓
más PID nézet
```

---

# 30. Docker Registry

A Registry nem a local runtime lánc egyik köztes eleme.

Ez egy külön szolgáltatás.

Feladata:

```text
container image-ek tárolása
container image-ek terjesztése
```

Például:

```text
Docker Hub
```

egy public registry.

Más registryk is létezhetnek:

```text
private registry
cloud registry
enterprise registry
```

---

# 31. Registry és Docker Engine kapcsolata

Image letöltésnél:

```text
Docker Engine
↓
Registry
↓
image manifest + layers
↓
Local image storage
```

Image feltöltésnél:

```text
Local image
↓
Docker Engine
↓
Registry
```

Ezért:

```text
Registry
≠
Docker Host
```

és:

```text
Registry
≠
container runtime
```

---

# 32. `docker pull`

Ha ezt futtatjuk:

```bash
docker pull nginx
```

a Docker:

```text
registryből image metadata és layer tartalmakat kér
↓
helyben eltárolja őket
```

Nem indul container.

Mental model:

```text
Registry
↓
nginx image
↓
Local image store
```

---

# 33. `docker run nginx` – magas szintű folyamat

Most rakjuk össze az egészet.

Parancs:

```bash
docker run nginx
```

Első lépések:

```text
Felhasználó
↓
Docker CLI
↓
Docker API
↓
dockerd
```

A daemon feldolgozza a container indítási kérést.

---

# 34. Image ellenőrzése

A Docker megvizsgálja, rendelkezésre áll-e a szükséges image.

```text
nginx image local?
```

Ha igen:

```text
local image használható
```

Ha nincs:

```text
Registry
↓
pull
↓
local image storage
```

---

# 35. Container konfiguráció létrehozása

A Docker létrehozza a containerhez szükséges konfigurációt.

Ez tartalmazhat például:

```text
image
command
environment variables
mountok
network configuration
resource limits
hostname
user
```

adatokat.

Ez még nem maga a futó process.

---

# 36. Runtime réteg

A container létrehozási kérés továbbkerül a runtime infrastruktúrához.

Egyszerűsítve:

```text
dockerd
↓
containerd
↓
shim
↓
OCI runtime
```

A pontos belső működés verzió- és konfigurációfüggő lehet, de ez jó logikai mental model.

---

# 37. Linux runtime környezet létrehozása

Az OCI runtime a Linux kernel funkcióit használva létrehozza a container környezetét.

Például:

```text
PID namespace
network namespace
mount namespace
UTS namespace
cgroup
filesystem mounts
root filesystem
```

beállítások jöhetnek létre.

---

# 38. Main process indítása

Ezután elindul az image által meghatározott process.

Nginx esetén például:

```text
nginx
```

A containerben ez tipikusan:

```text
PID 1
```

lesz.

Mental model:

```text
Container environment
↓
PID 1
↓
nginx
```

---

# 39. `docker run` teljes logikai flow

```text
User
↓
Docker CLI
↓
Docker API
↓
dockerd
↓
image ellenőrzése
↓
Registry pull, ha szükséges
↓
container configuration
↓
containerd
↓
runtime shim
↓
OCI runtime
↓
Linux kernel
├── namespaces
├── cgroups
├── mounts
└── networking primitives
↓
container main process
↓
RUNNING
```

---

# 40. `docker run` nem pusztán processindítás

A:

```bash
docker run nginx
```

több dolgot kombinál.

Magas szinten:

```text
image biztosítása
+
container létrehozása
+
container indítása
```

Ezért tanulási szinten mondhatjuk:

```text
docker run
≈
docker create
+
docker start
```

de a `run` természetesen további konfigurációkat és szükség esetén image letöltést is kezel.

---

# 41. `docker create`

Parancs:

```bash
docker create nginx
```

Container objektumot hoz létre az image alapján.

Ekkor:

```text
container létrejött
```

de a main process még nem feltétlenül fut.

Állapot:

```text
CREATED
```

---

# 42. `docker start`

Parancs:

```bash
docker start <container>
```

egy már létező containert indít el.

Mental model:

```text
existing container
↓
runtime start
↓
main process indul
↓
RUNNING
```

---

# 43. `docker stop`

Parancs:

```bash
docker stop <container>
```

A Docker megpróbálja kontrolláltan leállítani a container main processét.

Magas szinten:

```text
CLI
↓
API
↓
dockerd
↓
runtime layer
↓
container task
↓
termination signal
```

Ha a process leáll:

```text
container state
↓
EXITED
```

---

# 44. `docker kill`

A:

```bash
docker kill <container>
```

más jellegű művelet.

Itt a Docker közvetlenül signal küldését kezdeményezi a container main process felé.

Alapértelmezésben ez tipikusan:

```text
SIGKILL
```

Ez nem graceful shutdown.

A signalkezelést később külön gyakoroljuk.

---

# 45. `docker rm`

Parancs:

```bash
docker rm <container>
```

eltávolítja a container objektumot.

Ezzel eltűnhet:

```text
container metadata
container writable layer
runtime state
```

De az image:

```text
nem törlődik automatikusan
```

Ezért továbbra is:

```text
image
≠
container
```

---

# 46. Docker metadata

Egy containerhez számos metadata tartozik.

Például:

```text
Container ID
name
image reference
command
environment variables
network settings
mounts
resource limits
restart policy
state
```

Ezeket például:

```bash
docker inspect <container>
```

paranccsal lehet részletesen megvizsgálni.

---

# 47. Container ID

Minden Docker container rendelkezik egy egyedi ID-val.

Például:

```text
6d8f4b7c3e21...
```

A Docker CLI sok esetben elfogadja:

```text
teljes ID
rövidített ID
container name
```

formátumokat.

---

# 48. Container name

Egy containernek ember által olvasható nevet is adhatunk.

Például:

```bash
docker run --name web nginx
```

Ebben az esetben:

```text
container name
=
web
```

Ez egyszerűbbé teszi a kezelést.

---

# 49. Docker API példa `docker ps` esetén

Amikor futtatjuk:

```bash
docker ps
```

nem a CLI saját maga keresi meg a processzeket.

A folyamat inkább:

```text
Docker CLI
↓
Docker API request
↓
dockerd
↓
container state lekérdezése
↓
API response
↓
Docker CLI
↓
eredmény megjelenítése
```

---

# 50. `docker logs` architekturális nézete

Ha futtatjuk:

```bash
docker logs web
```

a CLI nem közvetlenül „belenéz” a containerbe.

Magas szinten:

```text
Docker CLI
↓
Docker API
↓
dockerd
↓
logging subsystem
↓
log data
↓
Docker CLI
```

Ez később fontos lesz log troubleshootingnál.

---

# 51. Docker Engine natív Linuxon

Egy Linux szerveren egyszerűbb az architektúra.

Például:

```text
Ubuntu Server
│
├── Docker CLI
│
├── dockerd
│
├── containerd
│
├── shimek
│
└── container processzek
        ↓
   Linux kernel
```

A későbbi DigitalOcean Ubuntu szerverünk ehhez a modellhez lesz közel.

---

# 52. Docker daemon konfiguráció natív Linuxon

Natív Linux Docker Engine esetén a daemon fő konfigurációs fájlja tipikusan:

```text
/etc/docker/daemon.json
```

Itt lehet különböző Docker daemon beállításokat megadni.

Például később:

```text
logging
registry
networking
runtime
storage
```

jellegű beállításokkal találkozhatunk.

---

# 53. Docker Desktop nem ugyanaz, mint Docker Engine

A:

```text
Docker Desktop
```

egy teljes desktop alkalmazás és integrációs környezet.

A:

```text
Docker Engine
```

pedig container engine.

Docker Desktop biztosíthat többek között:

```text
Docker Engine környezetet
Docker CLI-t
Docker Compose-ot
GUI-t
WSL integrációt
desktop networkinget
file sharinget
Kubernetes integrációt
```

Ezért:

```text
Docker Desktop
≠
Docker Engine
```

---

# 54. Docker Desktop Windows alatt

Windows nem Linux kernelre épül.

A Linux containerek viszont Linux kernel funkciókat igényelnek.

Ezért Docker Desktop Windows alatt valamilyen Linux környezetet használ.

Modern Windows rendszeren tipikusan:

```text
WSL 2 backend
```

használható.

---

# 55. WSL 2

WSL:

```text
Windows Subsystem for Linux
```

WSL 2 egy valódi Linux kernelt használ egy könnyű virtualizált környezetben.

Mental model:

```text
Windows 11
↓
WSL 2 virtualization
↓
Linux kernel
```

A WSL 2-ben több Linux disztribúció is futhat.

Például:

```text
Ubuntu
docker-desktop
más Linux distro
```

---

# 56. Docker Desktop és `docker-desktop`

Docker Desktop WSL2 backend használatakor saját WSL disztribúciót használ:

```text
docker-desktop
```

Ebben fut a Docker Desktop Linux backendje.

Egyszerűsített modell:

```text
Windows 11
↓
WSL 2
↓
docker-desktop
↓
Docker backend
↓
Linux containers
```

Ez fontos:

> A Docker Engine nem egyszerűen a saját Ubuntu WSL disztribúciónkban fut.

---

# 57. A saját Ubuntu WSL disztribúciónk

A mi fejlesztői környezetünkben van:

```text
Ubuntu WSL
```

Ezt használjuk:

```text
bash
Linux parancsok
git
project files
docker CLI
```

munkára.

A Docker Desktop WSL integration segítségével az Ubuntu WSL-ből elérjük a Docker Desktop backendjét.

Mental model:

```text
Ubuntu WSL
↓
docker CLI
↓
Docker Desktop integration
↓
Docker Desktop backend
```

---

# 58. A saját környezetünk pontosabb modellje

```text
Windows 11
│
├── WSL2 Ubuntu
│      │
│      ├── bash
│      ├── projektfájlok
│      └── docker CLI
│             │
│             │ WSL integration
│             ↓
│
└── Docker Desktop
       │
       └── docker-desktop WSL distro
              │
              └── Docker backend
                     │
                     ├── dockerd
                     ├── containerd/runtime infrastruktúra
                     └── Linux containers
```

Mindeközben a WSL2 disztribúciók ugyanazon WSL2 Linux kernel infrastruktúrára épülnek.

---

# 59. Miért nem telepítünk külön Docker Engine-t Ubuntu WSL-be?

Docker Desktop dokumentációja kifejezetten azt javasolja, hogy Docker Desktop WSL2 használatakor ne fusson mellette külön Docker Engine vagy külön Docker CLI telepítés ugyanabban a WSL disztribúcióban, mert konfliktushoz vezethet.

A kívánt modell:

```text
Windows 11
│
├── Docker Desktop backend
│
└── Ubuntu WSL
      └── docker CLI
             ↓
        ugyanaz a Docker Desktop környezet
```

Nem ezt akarjuk:

```text
Docker Desktop Engine
+
külön Ubuntu WSL Docker Engine
```

---

# 60. Hogyan ellenőrizhető a kliens és szerver?

A:

```bash
docker version
```

parancs külön információt mutat:

```text
Client
Server
```

részekről.

Ez jól szemlélteti a Docker kliens–szerver architektúrát.

Mental model:

```text
Client
→ Docker CLI
```

```text
Server
→ Docker Engine backend
```

---

# 61. `docker info`

A:

```bash
docker info
```

részletes Docker Engine információkat mutat.

Itt többek között láthatunk adatokat:

```text
Containers
Images
Server Version
Storage Driver
Logging Driver
Cgroup Driver
Cgroup Version
Runtimes
Kernel Version
Operating System
Docker Root Dir
```

területekről.

Ez később nagyon fontos troubleshooting parancs lesz.

---

# 62. Docker context

A Docker CLI képes különböző Docker endpointokhoz kapcsolódni.

Erre szolgál:

```text
Docker context
```

Például:

```bash
docker context ls
```

megmutatja az elérhető context-eket.

Egy context többek között azt határozhatja meg:

```text
melyik Docker endpointot használja a CLI
```

Ez azért fontos, mert:

```text
docker CLI
```

nem feltétlenül csak a helyi Docker Engine-t tudja kezelni.

Elméletileg remote Docker hosthoz is kapcsolódhat.

---

# 63. Docker CLI nem feltétlenül ugyanazon a gépen van

A Docker kliens és daemon architektúrája lehet:

```text
Machine A
Docker CLI
      ↓
network
      ↓
Machine B
Docker daemon
```

is.

Tehát:

```text
Docker Client
≠
Docker Host
```

szükségszerűen.

A mi jelenlegi környezetünkben azonban a kapcsolat lokális.

---

# 64. Docker Host

A:

```text
Docker Host
```

az a rendszer, amelyen a Docker Engine runtime környezete és a containerek futnak.

Natív Linux esetén:

```text
Ubuntu Server
↓
Docker Engine
↓
Containers
```

Docker Desktop esetén a host fogalom kevésbé közvetlen, mert a desktop virtualization/backend réteg is részt vesz a működésben.

---

# 65. Docker networking architekturális helye

A Docker daemon Docker network objektumokat kezel.

A tényleges Linux networkinghez pedig kernel mechanizmusok tartozhatnak.

Például:

```text
Docker network object
↓
network namespace
↓
virtual interface
↓
bridge
↓
routing
↓
NAT
```

A részletes networkinget külön fejezetben tanuljuk.

---

# 66. Docker volume architekturális helye

A Docker volume szintén Docker-managed object.

Mental model:

```text
Docker daemon
↓
Volume object
↓
storage
↓
mount
↓
Container filesystem
```

A volume maga nem a container writable layerének része.

Ezért tud persistent storage-ként működni.

---

# 67. Docker image storage

A Docker helyben image-eket tárol.

Modern Docker Engine környezetben a containerd image store használata egyre fontosabb.

Friss Docker Engine telepítéseknél a containerd image store lehet az alapértelmezett storage backend, amely snapshottereket használ a hagyományos graph driver modell helyett.

Mental model:

```text
Image manifest
+
Image config
+
Layers
↓
local image store
```

---

# 68. Storage driver vs snapshotter

Régebbi és más konfigurációkban találkozhatunk:

```text
storage driver
```

fogalommal.

Például:

```text
overlay2
```

Modern containerd image store esetén:

```text
snapshotter
```

kezeli az image és container filesystem rétegekhez tartozó snapshotokat.

Ezt később a storage fejezetben bontjuk ki részletesen.

---

# 69. Docker architecture és security

Az architektúra megértése security szempontból is fontos.

Például:

```text
docker.sock hozzáférés
```

nagyon erős jogosultságot adhat.

Hasonlóan:

```text
--privileged
```

jelentősen csökkentheti a container izolációját.

Ezért később fontos lesz:

```text
least privilege
Linux capabilities
seccomp
non-root container
read-only filesystem
resource limits
```

használata.

---

# 70. Container isolation nem VM isolation

A containerek közös kernelt használnak.

Ezért:

```text
container isolation
≠
külön VM kernel izoláció
```

Egy VM esetén:

```text
Guest Kernel
```

külön van a host kerneltől.

Containernél:

```text
shared Linux kernel
```

van.

Ez security szempontból fontos különbség.

---

# 71. Docker architecture és troubleshooting

A Docker architektúra segítségével rétegenként tudunk hibát keresni.

Például:

```text
docker parancs működik?
↓
daemon elérhető?
↓
image rendelkezésre áll?
↓
container létrejött?
↓
runtime elindította?
↓
main process fut?
↓
network működik?
↓
application működik?
```

Ez ugyanaz a gondolkodás, amit networkingnél már használtunk:

> Mi az utolsó pont, amiről biztosan tudom, hogy működik?

---

# 72. Példa: `Cannot connect to the Docker daemon`

Ha ezt kapjuk:

```text
Cannot connect to the Docker daemon
```

akkor a probléma valószínűleg a:

```text
Docker CLI
↓
Docker endpoint
↓
Docker daemon
```

kapcsolat környékén van.

Nem az nginx alkalmazás konfigurációját érdemes elsőként vizsgálni.

---

# 73. Példa: Docker működik, app nem

Ha:

```bash
docker ps
```

működik,

és a container:

```text
RUNNING
```

de a weboldal nem érhető el,

akkor már tudjuk:

```text
CLI
✓

Docker daemon
✓

runtime
valószínűleg ✓

container process
valószínűleg ✓
```

Ezután érdemes vizsgálni:

```text
port publishing
container network
application bind
application logs
firewall
HTTP
```

rétegeket.

---

# 74. Docker architecture és Kubernetes

Kubernetesnél különösen fontos lesz:

```text
container runtime
```

fogalma.

Kubernetes nem egyszerűen:

```text
Docker parancsokat futtat
```

hanem szabványos runtime infrastruktúrán keresztül kezel container workloadokat.

Ezért fontos már most különválasztani:

```text
Docker
```

mint teljes container platformot

és:

```text
container runtime
```

mint alacsonyabb szintű végrehajtási infrastruktúrát.

---

# 75. A Docker architektúra rétegei

Jó mental model:

```text
USER INTERFACE
Docker CLI

        ↓

API LAYER
Docker API

        ↓

DOCKER MANAGEMENT
dockerd

        ↓

CONTAINER RUNTIME MANAGEMENT
containerd

        ↓

RUNTIME ADAPTER / SUPERVISION
containerd shim

        ↓

LOW-LEVEL OCI RUNTIME
runc vagy más runtime

        ↓

OPERATING SYSTEM
Linux kernel

        ↓

HARDWARE
CPU / RAM / Disk / Network
```

---

# 76. Hol helyezkedik el maga a container?

A container nem egy külön doboz a Docker daemon memóriájában.

A végén:

```text
Linux process
```

jön létre.

Mental model:

```text
Docker CLI
↓
dockerd
↓
containerd
↓
shim
↓
OCI runtime
↓
Linux kernel
↓
Container Process
```

A process:

```text
Linux kernel által futtatott process
```

csak izolált namespace-ekben és cgroupokban.

---

# 77. Teljes architektúra – natív Linux

```text
                         Registry
                            ↑
                            │ pull / push
                            ↓

User
 │
 ↓
Docker CLI
 │
 │ Docker API
 ↓
dockerd
 │
 ├── Images
 ├── Containers
 ├── Networks
 ├── Volumes
 │
 ↓
containerd
 │
 ↓
containerd shim
 │
 ↓
OCI Runtime
 │
 ↓
Linux Kernel
 │
 ├── Process management
 ├── Namespaces
 ├── Cgroups
 ├── Mounts
 ├── Networking
 │
 ↓
Container Main Process
```

---

# 78. Teljes architektúra – Windows 11 + WSL2

A mi környezetünk:

```text
Windows 11
│
├── WSL2 Ubuntu
│      │
│      ├── projekt
│      ├── bash
│      └── docker CLI
│             │
│             │ Docker Desktop WSL integration
│             ↓
│
└── Docker Desktop
       │
       └── docker-desktop WSL környezet
              │
              ├── Docker backend
              ├── Docker Engine komponensek
              ├── container runtime infrastruktúra
              │
              ↓
          WSL2 Linux kernel
              │
              ↓
          Linux containers
```

A Docker Desktop WSL2 backend saját `docker-desktop` disztribúcióban fut, míg a felhasználói Ubuntu disztribúció integráción keresztül éri el a Docker CLI funkciókat.

---

# 79. WSL2 kernel megjegyzés

WSL2-ben több Linux disztribúció ugyanazt a WSL2 Linux kernelt használhatja.

Tehát nem úgy kell elképzelni:

```text
Ubuntu → saját kernel
docker-desktop → másik saját kernel
```

hanem inkább:

```text
WSL2 virtualization
↓
közös Linux kernel infrastruktúra
├── Ubuntu distro
└── docker-desktop distro
```

Ez a klasszikus külön VM-es gondolkodástól eltérő modell.

---

# 80. Docker Desktop networking

Docker Desktop esetén a networking nem teljesen ugyanúgy működik, mint egy natív Linux Docker hoston.

Windows WSL2 backendnél Docker Desktop saját backend komponensei is részt vesznek például:

```text
Docker API kezelésben
port forwardingban
proxy funkciókban
desktop networkingben
```

Ezért Docker Desktop networkingnél nem mindig szabad egy az egyben natív Linux hálózati topológiát feltételezni.

---

# 81. Fő komponensek röviden

## Docker CLI

```text
Felhasználói Docker kliens.
Parancsokat küld a Docker API felé.
```

## Docker API

```text
Kommunikációs interfész a kliens és Docker daemon között.
```

## dockerd

```text
Magas szintű Docker management daemon.
```

## containerd

```text
Container lifecycle-, runtime- és image-infrastruktúra fontos komponense.
```

## containerd shim

```text
Köztes runtime/task supervision réteg.
```

## OCI runtime

```text
Low-level container runtime.
Tipikus implementáció: runc.
```

## Linux kernel

```text
Ténylegesen futtatja a processzeket,
és biztosítja a namespaces, cgroups, mountok stb. funkcióit.
```

## Registry

```text
Container image-ek tárolására és terjesztésére szolgáló rendszer.
```

---

# 82. Amit ne keverj össze

```text
Docker CLI
≠
Docker daemon
```

```text
Docker daemon
≠
container process
```

```text
Docker Engine
≠
Docker Desktop
```

```text
containerd
≠
container
```

```text
runc
≠
hosszú ideig futó container daemon
```

```text
Docker Registry
≠
Docker Host
```

```text
Docker Host
≠
Container
```

```text
Ubuntu WSL
≠
Docker Desktop docker-desktop backend
```

---

# 83. `docker run nginx` gyors ismétlés

```text
docker run nginx
↓
Docker CLI
↓
Docker API
↓
dockerd
↓
image rendelkezésre áll?
├── igen
└── nem → registry pull
↓
container konfiguráció
↓
containerd
↓
runtime shim
↓
OCI runtime
↓
Linux kernel
├── namespaces
├── cgroups
├── mounts
└── networking
↓
nginx main process
↓
container RUNNING
```

---

# 84. Troubleshooting mental model

Ha Docker-problémát keresünk:

```text
1. Docker CLI működik?
↓
2. Jó Docker endpointot/contextet használ?
↓
3. Docker daemon elérhető?
↓
4. Image rendelkezésre áll?
↓
5. Container létrejött?
↓
6. Runtime el tudta indítani?
↓
7. Main process fut?
↓
8. Mountok rendben vannak?
↓
9. Network rendben van?
↓
10. Application működik?
```

Ez fontos DevOps-gondolkodási minta.

---

# 85. Végső mental model

A Docker architektúrát ne egyetlen programként képzeld el.

Inkább rétegek együttműködéseként:

```text
Felhasználó
↓
Docker CLI
↓
Docker API
↓
dockerd
↓
containerd
↓
containerd shim
↓
OCI runtime
↓
Linux kernel
↓
Container process
```

A Registry ettől külön áll:

```text
Docker Engine
↕
Registry
```

és image-ek tárolására, terjesztésére szolgál.

A legfontosabb felismerés:

> A Docker magasabb szintű komponensei leírják, kezelik és koordinálják a containereket, de a végén valódi Linux processzek futnak, amelyek izolációját és erőforrás-kezelését a Linux kernel biztosítja.

---

# 86. Kapcsolat az előző fejezettel

Az első fejezetben ezt tanultuk:

```text
Container
=
izolált futtatási környezet
+
processzek
```

Most már azt is látjuk, hogyan jutunk el idáig:

```text
Felhasználó
↓
Docker CLI
↓
Docker Engine
↓
runtime infrastruktúra
↓
Linux kernel
↓
izolált container process
```

A két fejezet együtt:

```text
Docker introduction
→ Mi az image?
→ Mi a container?
→ Miért nem VM?
→ Mi az izoláció?

Docker architecture
→ Ki kezeli az image-et?
→ Ki fogadja a parancsot?
→ Ki menedzseli a runtime-ot?
→ Ki indítja el a processzt?
→ Hol végzi el a tényleges munkát a Linux kernel?
```