# Docker Images és Registry

## 1. Mi ennek a fejezetnek a célja?

Ebben a fejezetben azt nézzük meg, hogyan:

- nevezzük el a Docker image-eket,
- különböztetjük meg a verzióikat,
- tároljuk őket registryben,
- töltjük le őket,
- töltjük fel őket,
- használunk tag-eket és digesteket,
- kezeljük a local image store-t,
- működik a multi-platform image kezelés.

A teljes folyamat:

```text
Dockerfile
↓
docker build
↓
LOCAL IMAGE
↓
docker tag
↓
docker push
↓
REGISTRY
↓
docker pull
↓
másik Docker host
↓
docker run
```

---

# 2. Mi az a Docker Registry?

A Docker Registry egy olyan szolgáltatás, ahol Docker / OCI image-eket lehet:

```text
tárolni
verziózni
megosztani
letölteni
feltölteni
```

A legismertebb registry:

```text
Docker Hub
```

De sok más registry is létezik:

```text
Docker Hub
GitHub Container Registry
GitLab Container Registry
Amazon ECR
Google Artifact Registry
Azure Container Registry
saját privát registry
```

Mental model:

```text
IMAGE
↓
REGISTRY
↓
másik Docker host
```

---

# 3. Docker Hub weboldal vs registry cím

A Docker Hub böngészőből elérhető webes felülete:

```text
hub.docker.com
```

A Docker Hub registry namespace alapértelmezett hostneve viszont:

```text
docker.io
```

A tényleges registry API forgalomban például előfordulhat:

```text
registry-1.docker.io
```

Tehát:

```text
hub.docker.com
→ webes felület
```

```text
docker.io
→ image reference-eknél használt registry host
```

Például:

```text
docker.io/library/nginx:latest
```

---

# 4. Registry és repository

A kettő nem ugyanaz.

## Registry

A teljes image tároló szolgáltatás.

Például:

```text
docker.io
```

## Repository

Egy adott image-család vagy alkalmazás image-einek logikai gyűjteménye.

Például:

```text
nginx
postgres
node
redis
```

Mental model:

```text
REGISTRY
│
├── nginx repository
├── node repository
├── postgres repository
└── redis repository
```

---

# 5. Egy repository több image-verziót tartalmazhat

Például:

```text
node
├── 22
├── 22-alpine
├── 20
├── 20-alpine
└── latest
```

Ezeket a neveket:

```text
tag
```

segítségével különböztetjük meg.

---

# 6. Mi az a tag?

A tag egy ember számára könnyen olvasható referencia egy image-verzióhoz.

Példa:

```text
nginx:1.28
```

Felbontva:

```text
nginx
→ repository

1.28
→ tag
```

Másik példa:

```text
node:22-alpine
```

```text
node
→ repository

22-alpine
→ tag
```

---

# 7. Mi történik, ha nem adunk meg taget?

Ha ezt írjuk:

```bash
docker pull nginx
```

Docker alapértelmezésben:

```text
nginx:latest
```

reference-et használ.

Tehát:

```text
nginx
≈
nginx:latest
```

Fontos:

> A `latest` csak egy tag neve.

Nem azt jelenti, hogy Docker automatikusan megkeresi az objektíven legújabb verziót.

---

# 8. A `latest` tag nem különleges verzió

A repository tulajdonosa döntheti el, mire mutat.

Például:

```text
myapp:latest
↓
Image A
```

Később:

```text
myapp:latest
↓
Image B
```

Tehát a tag:

```text
mutable reference
```

vagyis változtatható referencia.

---

# 9. Tag mint mozgatható címke

Például:

```text
Image A
████████████

Image B
████████████

latest ─────→ Image B
```

Később:

```text
Image C
████████████

latest ─────→ Image C
```

Az image-tartalom nem változik.

Csak a referencia kerül át egy másik image-re.

---

# 10. Egy image-hez több tag is tartozhat

Például:

```text
myapp:1.0 ──────┐
                ↓
              IMAGE
                ↑
myapp:latest ───┘
```

Ebben az esetben:

```text
nem két image másolat van
```

hanem:

```text
két tag/reference
↓
ugyanarra az image-tartalomra
```

mutat.

---

# 11. Mi az a digest?

Az image tartalmához cryptographic hash tartozhat.

Például:

```text
sha256:a8f3c8...
```

Ezt digestnek nevezzük.

Image reference digesttel:

```text
nginx@sha256:abcdef123456...
```

Mental model:

```text
TAG
→ emberbarát
→ változhat
```

```text
DIGEST
→ tartalomhoz kötött
→ stabil azonosító
```

---

# 12. Tag és digest különbsége

Példa:

```text
nginx:1.28
↓
sha256:AAA
```

Később elképzelhető:

```text
nginx:1.28
↓
sha256:BBB
```

A tag változhat.

Viszont:

```text
nginx@sha256:AAA
```

mindig ugyanarra a tartalomra hivatkozik.

---

# 13. Miért fontos a digest DevOpsban?

Production deploymentnél szeretnénk, hogy ugyanazt a buildet lehessen reprodukálni.

Ha ezt használjuk:

```text
mycompany/api:latest
```

akkor a tag később más image-re mutathat.

Ezzel szemben:

```text
mycompany/api@sha256:...
```

egy konkrét image-tartalomhoz kötött.

Ez fontos:

```text
reproducibility
```

vagyis reprodukálhatóság miatt.

---

# 14. Verziózott tag-ek

Gyakori megoldás:

```text
myapp:1.0
myapp:1.1
myapp:2.0
```

vagy:

```text
myapp:2.4.7
```

Ez ember számára sokkal olvashatóbb, mint egy teljes digest.

Productionben ezért gyakran:

```text
pontos verzió tag
```

vagy:

```text
digest
```

használatos.

---

# 15. Teljes Docker image név

Példa:

```text
docker.io/library/nginx:1.28
```

Felbontva:

```text
docker.io
→ registry
```

```text
library
→ namespace
```

```text
nginx
→ repository
```

```text
1.28
→ tag
```

---

# 16. Image reference általános formája

Egyszerűsítve:

```text
[registry/]namespace/repository[:tag]
```

Például:

```text
docker.io/thomas/wordy:1.0
```

vagy rövidebben:

```text
thomas/wordy:1.0
```

ha Docker Hubot használunk.

---

# 17. Mi az a `library`?

A Docker Hub official image-ek speciális namespace-e:

```text
library
```

Például:

```text
docker.io/library/nginx:latest
```

Ezért amikor ezt írjuk:

```bash
docker pull nginx
```

Docker automatikusan kiegészíti a reference-et.

Mental model:

```text
nginx
↓
docker.io/library/nginx:latest
```

---

# 18. Saját namespace

Ha például a Docker Hub felhasználónév:

```text
thomas
```

és van egy alkalmazás:

```text
wordy
```

akkor a repository lehet:

```text
thomas/wordy
```

Taggel:

```text
thomas/wordy:1.0
```

Teljes formában:

```text
docker.io/thomas/wordy:1.0
```

---

# 19. Local image és registry image

Fontos különbség:

```text
local image
≠
registryben lévő image
```

Ha ezt futtatjuk:

```bash
docker build -t wordy:1.0 .
```

akkor az image:

```text
a local Docker image store-ban
```

jön létre.

Még nincs automatikusan Docker Hubon.

---

# 20. Local image store

A Docker Engine helyileg tárol image-eket.

Ezeket listázhatjuk:

```bash
docker image ls
```

vagy:

```bash
docker images
```

Példa:

```text
REPOSITORY      TAG       IMAGE ID        SIZE
nginx           latest    123abc...       190MB
node            22        456def...       ...
wordy           1.0       789ghi...       ...
```

Ez:

```text
LOCAL IMAGE STORE
```

tartalma.

Nem a registry teljes tartalmát látjuk.

---

# 21. `docker pull`

Image letöltése registryből:

```bash
docker pull nginx:1.28
```

Magas szintű folyamat:

```text
Docker CLI
↓
Docker Engine
↓
registry
↓
manifest / image metadata
↓
szükséges layer-ek
↓
local image store
```

---

# 22. Mi történik pull során?

Egyszerűsítve:

```text
1. Docker feloldja az image reference-et
↓
2. kapcsolatba lép a registryvel
↓
3. lekéri a szükséges manifestet
↓
4. ellenőrzi, milyen layer-ek kellenek
↓
5. csak a hiányzó layer-eket tölti le
↓
6. image bekerül a local image store-ba
```

---

# 23. Layer újrafelhasználás pull közben

Például:

```text
Image A
├── Layer 1
├── Layer 2
└── Layer 3
```

```text
Image B
├── Layer 1
├── Layer 2
└── Layer 4
```

Ha:

```text
Layer 1
Layer 2
```

már megtalálható lokálisan, akkor Dockernek nem feltétlenül kell újra letölteni őket.

Ez:

```text
storage
+
network
```

szempontjából is hatékony.

---

# 24. Mi az a manifest?

A Docker image nem egyszerűen egyetlen nagy fájl.

Az image-hez metadata is tartozik.

Egyszerűsített modell:

```text
IMAGE REFERENCE
↓
MANIFEST
├── config
├── layer A
├── layer B
└── layer C
```

A manifest többek között meghatározza:

```text
milyen image config tartozik hozzá
milyen layer-ek tartoznak hozzá
milyen digestekkel
```

---

# 25. Image configuration

Az image config tartalmazhat például:

```text
default command
entrypoint
environment defaultok
working directory
user
architecture
filesystem metadata
```

Az image tehát:

```text
filesystem content
+
metadata
```

kombináció.

---

# 26. Multi-platform image

Ugyanaz az image név több platformhoz tartozó image-verzióra is mutathat.

Például:

```text
nginx:latest
↓
IMAGE INDEX
├── linux/amd64
├── linux/arm64
└── linux/arm/v7
```

Ez különösen fontos például:

```text
Intel / AMD PC
```

és:

```text
Apple Silicon Mac
```

esetén.

---

# 27. CPU architektúra

Gyakori platformok:

```text
linux/amd64
linux/arm64
```

Például:

```text
tipikus Intel/AMD PC
→ amd64
```

```text
Apple M-series Mac
→ arm64
```

Docker automatikusan megpróbálja kiválasztani a megfelelő image-et.

---

# 28. Image index / manifest list

A multi-platform image felső szintjén lehet:

```text
image index
```

amely platformonként külön manifestre mutat.

Mental model:

```text
nginx:latest
↓
IMAGE INDEX
│
├── linux/amd64
│      ↓
│    Manifest A
│
└── linux/arm64
       ↓
     Manifest B
```

---

# 29. Platform explicit megadása

Szükség esetén:

```bash
docker pull --platform linux/amd64 nginx
```

vagy:

```bash
docker run --platform linux/amd64 nginx
```

Normál esetben azonban Docker automatikusan választ.

---

# 30. `docker tag`

Új image reference készítése:

```bash
docker tag wordy:1.0 thomas/wordy:1.0
```

Ez nem azt jelenti, hogy Docker lemásolja az egész image-et.

Mental model:

```text
wordy:1.0 ──────────────┐
                        ↓
                      IMAGE
                        ↑
thomas/wordy:1.0 ───────┘
```

---

# 31. Miért kell tag push előtt?

Registrybe pusholásnál az image reference megadja:

```text
melyik registry
melyik namespace
melyik repository
melyik tag
```

Például:

```text
thomas/wordy:1.0
```

Docker Hub esetén:

```text
docker.io/thomas/wordy:1.0
```

---

# 32. `docker login`

Registry autentikáció:

```bash
docker login
```

A folyamat:

```text
Docker CLI
↓
registry authentication
↓
credential / token
↓
hozzáférési jogosultság
```

Pushhoz tipikusan szükség van hitelesítésre.

---

# 33. `docker push`

Local image feltöltése registrybe:

```bash
docker push thomas/wordy:1.0
```

Magas szintű flow:

```text
LOCAL IMAGE
↓
Docker ellenőrzi layer-eket
↓
registry
↓
hiányzó layer-ek feltöltése
↓
config / manifest feltöltése
↓
tag/reference létrejön vagy frissül
```

---

# 34. Pull és push kapcsolata

```text
PULL

REGISTRY
↓
LOCAL IMAGE STORE
```

```text
PUSH

LOCAL IMAGE STORE
↓
REGISTRY
```

---

# 35. A registryben image-ek vannak, nem containerek

Nagyon fontos:

```text
REGISTRY
→ image-eket tárol
```

Nem:

```text
running container-eket
```

A flow:

```text
REGISTRY
↓ pull
IMAGE
↓ run
CONTAINER
```

---

# 36. Production flow

Példa:

```text
Developer machine
↓
docker build
↓
company/api:2.3
↓
docker push
↓
Registry
↓
Production server
↓
docker pull
↓
docker run
↓
Container
```

---

# 37. CI/CD kapcsolat

Később a folyamat automatizálható:

```text
git push
↓
GitHub Actions
↓
docker build
↓
docker tag
↓
docker push
↓
deployment
```

Ez tipikus DevOps workflow.

---

# 38. Image reference DevOps szempontból

Productionben kerülendő lehet a túl általános:

```text
myapp:latest
```

mert nem feltétlenül egyértelmű, milyen build fut.

Jobb lehet:

```text
myapp:2.4.7
```

vagy:

```text
myapp@sha256:...
```

Mental model:

```text
latest
→ kényelmes
→ kevésbé determinisztikus
```

```text
version tag
→ ember számára olvasható
→ jobban kontrollálható
```

```text
digest
→ konkrét content identity
→ maximális reprodukálhatóság
```

---

# 39. Tag nem image másolat

Ha:

```text
myapp:1.0
```

és:

```text
myapp:latest
```

ugyanarra az image ID/contentre mutat, akkor:

```text
nem duplikáljuk az image filesystemet
```

hanem több reference van ugyanahhoz az image-hez.

Ezért a tag inkább:

```text
név / pointer / referencia
```

mint külön image.

---

# 40. Image ID és digest

A kettő kapcsolódó, de nem minden esetben ugyanabban a kontextusban használjuk őket.

A local Docker image store mutathat:

```text
IMAGE ID
```

például:

```text
abc123...
```

Registry oldalon gyakran:

```text
DIGEST
```

például:

```text
sha256:...
```

A lényeg:

```text
content-addressed identity
```

vagyis a tartalom alapján történő azonosítás.

---

# 41. Content-addressable storage

Docker és OCI image-ek esetén a tartalom cryptographic hash alapján azonosítható.

Mental model:

```text
CONTENT
↓
SHA-256
↓
DIGEST
```

Ha a tartalom megváltozik:

```text
digest is megváltozik
```

Ez segít:

```text
integrity
cache
deduplication
reproducibility
```

területeken.

---

# 42. Repository vs tag vs digest

```text
Repository
→ image-család
```

```text
Tag
→ emberbarát referencia
```

```text
Digest
→ konkrét content identity
```

Példa:

```text
nginx:1.28
```

```text
repository = nginx
tag = 1.28
```

Digesttel:

```text
nginx@sha256:...
```

---

# 43. Registry namespace mental model

Példa:

```text
docker.io/thomas/wordy:1.0
```

Felbontva:

```text
docker.io
→ registry
```

```text
thomas
→ namespace / account
```

```text
wordy
→ repository
```

```text
1.0
→ tag
```

---

# 44. Saját registry használata

Nem kötelező Docker Hubot használni.

Példa:

```text
registry.example.com/company/api:2.1
```

Felbontva:

```text
registry.example.com
→ registry
```

```text
company
→ namespace
```

```text
api
→ repository
```

```text
2.1
→ tag
```

---

# 45. Registry lehet public vagy private

## Public repository

Mások is lehúzhatják az image-et.

## Private repository

Csak jogosultsággal rendelkező felhasználók / rendszerek érhetik el.

Tipikus production használat:

```text
private registry
↓
company images
↓
production servers
```

---

# 46. Image-ek törlése lokálisan

Például:

```bash
docker image rm nginx:latest
```

Rövid forma:

```bash
docker rmi nginx:latest
```

Ez local image eltávolítás.

Nem azt jelenti, hogy:

```text
a registryből is törlődik
```

---

# 47. Local delete és registry delete

```text
docker image rm
→ LOCAL
```

```text
registry UI/API delete
→ REMOTE
```

A kettő külön művelet.

---

# 48. Mi van, ha egy image-et container használ?

Ha egy image-ből meglévő container van, Docker bizonyos törléseket megtagadhat.

Mental model:

```text
IMAGE
↓
CONTAINER
```

Ha dependency van, először meg kell érteni:

```text
mely container használja?
```

Nem érdemes automatikusan force delete-et használni.

---

# 49. Dangling image röviden

Buildelések után maradhatnak olyan image layer-ek vagy image reference nélküli objektumok, amelyek már nem kellenek.

Ezekre gyakran használjuk a:

```text
dangling
```

kifejezést.

Megtekintés:

```bash
docker image ls -f dangling=true
```

---

# 50. Image prune

Nem használt dangling image-ek takarítása:

```bash
docker image prune
```

Óvatosan használjuk.

Először mindig nézzük meg:

```text
mit akarunk törölni?
```

---

# 51. `docker system prune`

Létezik szélesebb takarító parancs:

```bash
docker system prune
```

Ez több Docker objektumot is eltávolíthat.

Ezért:

```text
nem rutinból futtatjuk
```

hanem tudatosan.

Részletesebben a troubleshooting / maintenance fejezetben foglalkozunk vele.

---

# 52. Registry authentication

Registry használatkor a hitelesítés:

```text
authentication
```

és a jogosultság:

```text
authorization
```

külön fogalom.

Például:

```text
Ki vagy?
→ authentication
```

```text
Pusholhatsz ebbe a repositoryba?
→ authorization
```

---

# 53. Image security alapelv

Registryből letöltött image esetén is fontos kérdés:

```text
Ki készítette?
Mennyire megbízható?
Milyen verzió?
Mikor frissült?
Milyen base image-et használ?
Van ismert vulnerability?
```

Ezért fontos különbség:

```text
official image
verified publisher
random public image
```

között.

A security részt később részletesen külön fejezetben vesszük.

---

# 54. Ne tegyünk secretet image-be

Image pusholható registrybe.

Ezért image-be nem érdemes beégetni:

```text
password
API key
private key
access token
database password
```

Ha secret egy image layerbe bekerül, később sokkal nehezebb biztonságosan eltávolítani.

Mental model:

```text
IMAGE
→ portable artifact
→ lehet registryben
→ ne tartalmazzon secretet
```

---

# 55. Local config vs image artifact

Az image tartalmazhat:

```text
application
runtime
libraries
default configuration
filesystem
metadata
```

De environment-specifikus dolgokat gyakran runtime adjuk át.

Például:

```text
database URL
production secret
API token
```

nem feltétlenül kerül image-be.

---

# 56. Pull során honnan tudja Docker, mit töltsön le?

A Docker image reference alapján.

Például:

```bash
docker pull docker.io/library/nginx:1.28
```

A Docker feloldja:

```text
registry
repository
tag
```

majd lekéri a hozzá tartozó metadata struktúrát és layer-eket.

---

# 57. Pull során platformválasztás

Például:

```text
Host:
linux/amd64
```

Docker:

```text
image index
↓
linux/amd64 manifest
↓
megfelelő layer-ek
```

Apple Silicon esetén:

```text
Host:
linux/arm64
```

Docker:

```text
image index
↓
linux/arm64 manifest
```

---

# 58. Azonos image név, eltérő platform

Ezért:

```bash
docker pull nginx
```

ugyanúgy működhet:

```text
Windows + WSL2 amd64
```

és:

```text
Mac ARM64
```

gépen.

A tényleges letöltött platform-specific image azonban eltérhet.

---

# 59. Image és container kapcsolata

A teljes modell:

```text
REGISTRY
↓
docker pull
↓
LOCAL IMAGE
↓
docker run
↓
CONTAINER
```

Ha ugyanabból az image-ből több container indul:

```text
IMAGE
├── Container A
├── Container B
└── Container C
```

A read-only image layer-ek megoszthatók, miközben minden container saját writable layert kap.

---

# 60. Image distribution teljes mental model

```text
SOURCE CODE
↓
Dockerfile
↓
docker build
↓
LOCAL IMAGE
↓
docker tag
↓
IMAGE REFERENCE
↓
docker login
↓
docker push
↓
REGISTRY
↓
docker pull
↓
OTHER HOST
↓
LOCAL IMAGE
↓
docker run
↓
CONTAINER
```

---

# 61. Tipikus DevOps workflow

```text
Developer
↓
commit
↓
Git repository
↓
CI pipeline
↓
docker build
↓
myapp:2.4.7
↓
docker push
↓
Container Registry
↓
Deployment
↓
docker pull
↓
docker run
```

Később:

```text
Docker Compose
Kubernetes
CI/CD
```

fogja ezt magasabb szinten kezelni.

---

# 62. Legfontosabb parancsok

Local image lista:

```bash
docker image ls
```

Image letöltése:

```bash
docker pull nginx:1.28
```

Új tag/reference:

```bash
docker tag wordy:1.0 thomas/wordy:1.0
```

Registry login:

```bash
docker login
```

Push:

```bash
docker push thomas/wordy:1.0
```

Local image törlése:

```bash
docker image rm nginx:1.28
```

Dangling image-ek:

```bash
docker image ls -f dangling=true
```

Takarítás:

```bash
docker image prune
```

---

# 63. Gyakori félreértések

## A registry futtatja a containert

Nem.

```text
REGISTRY
→ image storage/distribution
```

```text
Docker Engine
→ container runtime
```

---

## A repository egy registry

Nem.

```text
Registry
↓
sok repository
```

---

## A tag maga az image

Nem.

```text
tag
→ reference
```

---

## A `latest` mindig a legfrissebb release

Nem.

```text
latest
→ csak egy tag neve
```

---

## Két tag mindig két külön image

Nem.

```text
myapp:1.0
myapp:latest
```

mutathat ugyanarra az image-re.

---

## `docker push` minden alkalommal mindent újra feltölt

Nem feltétlenül.

Docker content/layer alapú működés miatt a már meglévő blobokat nem feltétlenül kell újra feltölteni.

---

## `docker pull` mindig mindent újra letölt

Nem.

A lokálisan már meglévő layer-ek újrafelhasználhatók.

---

## `docker image rm` törli Docker Hubról is

Nem.

Csak a local image store-ból töröl.

---

# 64. Rövid fogalmi összefoglaló

```text
Registry
→ image tároló szolgáltatás
```

```text
Repository
→ egy image-család
```

```text
Tag
→ emberbarát, változtatható referencia
```

```text
Digest
→ content-based identity
```

```text
Pull
→ registry → local
```

```text
Push
→ local → registry
```

```text
Image index
→ több platform image-einek listája
```

```text
Manifest
→ egy konkrét image struktúrájának leírása
```

---

# 65. Rövid cheatsheet

```bash
docker image ls
```

```text
Milyen image-ek vannak lokálisan?
```

```bash
docker pull nginx
```

```text
Image letöltése registryből.
```

```bash
docker tag old:new account/app:new
```

```text
Új reference létrehozása ugyanahhoz az image-hez.
```

```bash
docker login
```

```text
Registry autentikáció.
```

```bash
docker push account/app:1.0
```

```text
Image feltöltése registrybe.
```

```bash
docker image rm image:tag
```

```text
Local image eltávolítása.
```

---

# 66. Fejezet végső mental model

```text
                    REGISTRY
                       │
            ┌──────────┴──────────┐
            │                     │
          PULL                  PUSH
            │                     │
            ↓                     ↑
                 LOCAL IMAGE
                     │
                     │ docker run
                     ↓
                  CONTAINER
```

Az image reference:

```text
registry / namespace / repository : tag
```

Például:

```text
docker.io/thomas/wordy:1.0
```

És a legfontosabb összefüggés:

```text
TAG
→ név/reference
```

```text
DIGEST
→ tartalomazonosság
```

```text
REGISTRY
→ image terjesztés
```

```text
DOCKER ENGINE
→ container futtatás
```

---

# 67. A későbbi laborban ezt fogjuk gyakorolni

A témához érdemes majd végigcsinálni egy teljes image lifecycle labort:

```text
1. Public image letöltése Docker Hubról
↓
2. Local image-ek vizsgálata
↓
3. Container indítása belőle
↓
4. Saját image buildelése
↓
5. Több tag létrehozása
↓
6. Docker Hub repository létrehozása
↓
7. docker login
↓
8. Saját image pusholása
↓
9. Local image törlése
↓
10. Saját image visszahúzása Docker Hubról
↓
11. Container indítása a registryből visszatöltött image-ből
```

Ezzel egyben végig fogjuk látni a teljes:

```text
BUILD
↓
TAG
↓
PUSH
↓
PULL
↓
RUN
```

folyamatot.