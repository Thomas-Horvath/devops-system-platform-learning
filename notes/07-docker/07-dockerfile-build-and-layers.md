# Dockerfile Build, Layers és Cache

## 1. Mi ennek a fejezetnek a célja?

Ebben a fejezetben azt nézzük meg, hogyan működik a Docker image build a háttérben.

A fő témák:

```text
Dockerfile build steps
image layers
build cache
cache invalidation
instruction order
COPY sorrend
npm install / npm ci cache
build context
.dockerignore
image size
--no-cache
--pull
multi-stage build előkészítése
```

A cél nem az, hogy minden belső implementációs részletet bemagoljunk, hanem hogy értsük:

> miért számít a Dockerfile sorrendje, és miért tud egy jól megírt Dockerfile sokkal gyorsabban buildelni.

---

# 2. Docker build alapfolyamat

Például:

```dockerfile
FROM node:22

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm ci

COPY . .

RUN npm run build

CMD ["npm", "start"]
```

Build:

```bash
docker build -t myapp:1.0 .
```

Magas szintű flow:

```text
Dockerfile
↓
build step 1
↓
build step 2
↓
build step 3
↓
...
↓
IMAGE
```

---

# 3. Mi az a build step?

A Dockerfile minden utasítása egy build lépésként értelmezhető.

Például:

```text
FROM node:22
↓
WORKDIR /app
↓
COPY package files
↓
RUN npm ci
↓
COPY source
↓
RUN npm run build
```

A modern Docker build rendszer, jellemzően BuildKit, ezeket a lépéseket és azok eredményeit képes cache-elni.

---

# 4. Mi az az image layer?

Az image filesystem változásai rétegekben épülnek egymásra.

Egyszerűsített mental model:

```text
Layer 4
→ source code / build output

Layer 3
→ dependencies

Layer 2
→ package files

Layer 1
→ base image
```

A felsőbb rétegek az alsóbb rétegekre épülnek.

---

# 5. A layer-ek read-only jellegűek

A kész image rétegeit úgy kezeljük, mint:

```text
read-only image layers
```

Például:

```text
IMAGE
├── Layer 4
├── Layer 3
├── Layer 2
└── Layer 1
```

A container futásakor ezek fölé kerül egy külön írható layer.

---

# 6. Image layer és container writable layer

Ha:

```bash
docker run myapp:1.0
```

akkor:

```text
Container writable layer
────────────────────────
Image Layer 4
Image Layer 3
Image Layer 2
Image Layer 1
```

Az image layer-ek közösek és read-only jellegűek.

A container saját változásai a writable layerbe kerülnek.

---

# 7. Több container ugyanabból az image-ből

Ha ugyanabból az image-ből három containert indítunk:

```text
                 Container A writable layer
                /
IMAGE LAYERS ── Container B writable layer
                \
                 Container C writable layer
```

Nem kell három teljes image másolat.

A base image layer-ek megoszthatók.

Ez storage szempontból nagyon hatékony.

---

# 8. Mi az a build cache?

A Docker megpróbálja újrahasználni korábbi build lépések eredményét.

Mental model:

```text
Dockerfile step
↓
ugyanaz a bemenet?
↓
cache találat
↓
nem kell újra végrehajtani
```

Ez teszi gyorsabbá az ismételt buildet.

---

# 9. Első build

Dockerfile:

```dockerfile
FROM node:22

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm ci

COPY . .

RUN npm run build
```

Első build:

```text
FROM node:22
→ fut / letöltés

WORKDIR /app
→ build

COPY package files
→ build

RUN npm ci
→ build

COPY source
→ build

RUN npm run build
→ build
```

Mivel még nincs megfelelő cache, a lépések ténylegesen végrehajtódnak.

---

# 10. Második build változtatás nélkül

Ha semmi nem változott:

```text
FROM
→ CACHED

WORKDIR
→ CACHED

COPY package files
→ CACHED

RUN npm ci
→ CACHED

COPY source
→ CACHED

RUN npm run build
→ CACHED
```

A build sokkal gyorsabb lehet.

---

# 11. Mi az a cache invalidation?

A cache invalidation jelentése:

```text
a korábbi cache eredmény már nem használható
```

mert valamelyik bemenet megváltozott.

Például:

```dockerfile
COPY package.json package-lock.json ./
```

Ha a `package.json` változik:

```text
COPY package files
↓
cache invalid
```

Utána:

```text
RUN npm ci
```

lépést is újra kell értékelni / futtatni.

---

# 12. A build lépések egymásra épülnek

Mental model:

```text
Step 1
↓
Step 2
↓
Step 3
↓
Step 4
```

Ha:

```text
Step 2 változik
```

akkor:

```text
Step 1
→ cache használható

Step 2
→ rebuild

Step 3
→ rebuild / újraértékelés

Step 4
→ rebuild / újraértékelés
```

Ezért számít nagyon a Dockerfile sorrendje.

---

# 13. A legfontosabb Dockerfile sorrendi szabály

Általános elv:

```text
ritkán változó dolgok
↓
Dockerfile eleje

gyakran változó dolgok
↓
Dockerfile vége
```

Ez cache szempontból hatékonyabb.

---

# 14. Node.js projektben mi változik gyakran?

Gyakran változik:

```text
src/
app/
components/
page.tsx
CSS
application code
```

Ritkábban változik:

```text
package.json
package-lock.json
```

Ezért érdemes külön kezelni őket.

---

# 15. Rosszabb Dockerfile sorrend

Példa:

```dockerfile
FROM node:22

WORKDIR /app

COPY . .

RUN npm install
```

Ha csak egy source fájlt módosítunk:

```text
app/page.tsx
```

akkor:

```text
COPY . .
→ megváltozik
↓
RUN npm install
→ cache is invalidálódhat
↓
dependency install újra lefut
```

Pedig a dependency lista nem változott.

---

# 16. Jobb Dockerfile sorrend

```dockerfile
FROM node:22

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm ci

COPY . .
```

Ha csak:

```text
app/page.tsx
```

változik:

```text
COPY package files
→ cache

RUN npm ci
→ cache

COPY source
→ rebuild
```

Pont ezt akarjuk.

---

# 17. Miért külön másoljuk a package fájlokat?

Mert a dependency install eredménye csak akkor változik, ha:

```text
package.json
vagy
package-lock.json
```

változik.

Ezért:

```dockerfile
COPY package.json package-lock.json ./

RUN npm ci
```

egy stabil cache-határt hoz létre.

---

# 18. Source változás példa

Első állapot:

```text
package.json
package-lock.json
app/page.tsx
```

Build után minden cache-elhető.

Módosítjuk:

```text
app/page.tsx
```

Új build:

```text
FROM
→ cache

WORKDIR
→ cache

COPY package files
→ cache

RUN npm ci
→ cache

COPY . .
→ rebuild

RUN npm run build
→ rebuild
```

Ez ideális.

---

# 19. Dependency változás példa

Most módosítjuk:

```text
package.json
```

Új build:

```text
FROM
→ cache

WORKDIR
→ cache

COPY package files
→ rebuild

RUN npm ci
→ rebuild

COPY . .
→ rebuild

RUN npm run build
→ rebuild
```

Ez helyes, mert dependency változás történt.

---

# 20. `npm install` vs `npm ci`

Fejlesztésnél gyakori:

```bash
npm install
```

Reprodukálható buildnél gyakori:

```bash
npm ci
```

A `npm ci`:

```text
package-lock.json alapján dolgozik
clean installt végez
CI / build környezetre alkalmas
```

Ezért Docker production buildben gyakran:

```dockerfile
COPY package.json package-lock.json ./
RUN npm ci
```

látható.

---

# 21. Miért jó a lock file?

A:

```text
package-lock.json
```

rögzíti a dependency-k konkrét verzióit.

Mental model:

```text
package.json
→ mit szeretnénk

package-lock.json
→ pontosan milyen verziók kerüljenek telepítésre
```

Ez segíti a reprodukálható buildet.

---

# 22. Build cache nem internet-frissesség ellenőrzés

Fontos:

```text
Docker build cache
≠
külső szolgáltatások automatikus frissességellenőrzése
```

Példa:

```dockerfile
RUN apt-get update
```

Ha Docker szerint a build step cache-elhető:

```text
RUN apt-get update
→ CACHED
```

Docker nem feltétlenül fogja újra lefuttatni csak azért, mert a külső package repository közben változott.

---

# 23. `--no-cache`

Ha biztosan nem akarjuk használni a korábbi build cache-t:

```bash
docker build --no-cache -t myapp:1.0 .
```

Mental model:

```text
--no-cache
→ ne használj korábbi build step cache eredményeket
```

Ez hasznos lehet troubleshootingnál.

Normál buildnél nem szükséges.

---

# 24. `--pull`

Ha szeretnénk ellenőrizni a `FROM` base image frissebb változatát:

```bash
docker build --pull .
```

Mental model:

```text
--pull
→ ellenőrizd a base image friss változatát
```

---

# 25. `--no-cache` és `--pull` nem ugyanaz

```text
--no-cache
→ build cache kikapcsolása
```

```text
--pull
→ base image frissességének ellenőrzése
```

Akár együtt is használhatók:

```bash
docker build --pull --no-cache .
```

De ez nem tipikus mindennapi build parancs.

---

# 26. Build context

Amikor:

```bash
docker build .
```

a végén lévő:

```text
.
```

a build context.

Példa:

```text
myapp/
├── Dockerfile
├── package.json
├── package-lock.json
├── app/
├── public/
└── ...
```

Ha innen buildelünk:

```bash
docker build .
```

akkor:

```text
myapp/
```

a build context.

---

# 27. Build context és `COPY`

Példa:

```dockerfile
COPY . .
```

Bal oldali `.`:

```text
build context root
```

Jobb oldali `.`:

```text
aktuális WORKDIR
```

Ha:

```dockerfile
WORKDIR /app
COPY . .
```

akkor:

```text
build context
↓
/app
```

---

# 28. Miért baj a túl nagy build context?

Ha a projektben van:

```text
node_modules/
.git/
logs/
tmp/
.env
.next/
```

akkor ezek fölöslegesen bekerülhetnek a contextbe.

Ez:

```text
lassíthatja a buildet
ronthatja a cache hatékonyságát
növelheti az image-et
security kockázatot okozhat
```

---

# 29. `.dockerignore`

A:

```text
.dockerignore
```

segít kizárni fájlokat a build contextből.

Példa:

```text
node_modules
.git
.env
*.log
.next
```

A pontos lista projekttől függ.

---

# 30. `.dockerignore` előnyei

```text
kisebb build context
↓
gyorsabb build
```

```text
kevesebb fölösleges fájl
↓
tisztább image
```

```text
kevesebb érzékeny adat
↓
jobb security
```

```text
kevesebb gyakran változó fájl
↓
jobb cache
```

---

# 31. Image size és layer-ek

A layer-ek az image méretére is hatással vannak.

Példa:

```dockerfile
RUN curl -O huge-file.zip
RUN rm huge-file.zip
```

Első gondolat:

```text
letöltjük
↓
kitöröljük
↓
nincs többé
```

Layer szinten viszont:

```text
Layer A
→ huge-file.zip létrejön

Layer B
→ huge-file.zip töröltként látszik
```

Az alsó layer tartalma ettől még része lehet az image historynak.

---

# 32. Miért nem mindig csökkenti a törlés az image méretét?

Mert az image rétegei immutable jellegűek.

Ha egy korábbi layerbe bekerült egy fájl:

```text
Layer 1
→ file exists
```

egy későbbi layer:

```text
Layer 2
→ file deleted
```

nem írja át Layer 1-et.

Csak azt eredményezi, hogy a final filesystem view-ban már ne látszódjon.

---

# 33. Ideiglenes fájlok kezelése ugyanabban a `RUN` stepben

Jobb minta lehet:

```dockerfile
RUN curl -O huge-file.zip \
    && do-something huge-file.zip \
    && rm huge-file.zip
```

Mental model:

```text
egy RUN step
↓
temp file létrejön
↓
felhasználjuk
↓
töröljük
↓
final build resultbe nem szükséges bekerülnie
```

---

# 34. Package manager példa

Gyakori Linuxos minta:

```dockerfile
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
```

Cél:

```text
package metadata használata
↓
package telepítés
↓
fölösleges cache / metadata eltávolítása
```

ugyanazon build stepen belül.

---

# 35. Miért nem kell mindenáron minden `RUN`-t összevonni?

Fontos:

```text
kevesebb layer
≠
automatikusan jobb Dockerfile
```

A cél inkább:

```text
logikus build steps
jó cache
kis final image
olvasható Dockerfile
```

Nem kell mesterségesen mindent egyetlen óriási `RUN` parancsba tenni.

---

# 36. Layer vs build step – technikai pontosítás

Tanulási mental modelként gyakran mondjuk:

```text
Dockerfile sor
→ layer
```

De ez túl egyszerű.

Pontosabban:

> minden Dockerfile instruction egy build step, és bizonyos utasítások filesystem változásokat hoznak létre, amelyeket a builder és az image rétegstruktúra kezel.

Különösen fontos filesystem változások:

```text
COPY
ADD
RUN
```

A:

```text
CMD
ENTRYPOINT
ENV
WORKDIR
EXPOSE
```

jelentős része config / metadata jellegű.

---

# 37. Modern Docker és BuildKit

A modern Docker build rendszer tipikusan:

```text
BuildKit
```

technológiát használ.

A BuildKit többek között:

```text
cache-el
build dependency graphot kezel
párhuzamosíthat bizonyos munkát
hatékonyabban kezeli a build contextet
```

A részletes BuildKit internals nem szükséges most.

A fontos mental model:

```text
modern docker build
→ okos cache + build graph
```

---

# 38. Build output

Build közben ilyesmit láthatunk:

```text
[1/6] FROM node:22
[2/6] WORKDIR /app
[3/6] COPY package.json package-lock.json ./
[4/6] RUN npm ci
[5/6] COPY . .
[6/6] RUN npm run build
```

Ha cache találat van:

```text
CACHED
```

megjelenhet.

---

# 39. Mit jelent a `CACHED`?

Például:

```text
[4/6] RUN npm ci
CACHED
```

jelentése:

```text
Docker talált megfelelő korábbi build eredményt
↓
nem kellett újra végrehajtani az npm ci parancsot
```

---

# 40. Cache gyakorlati megfigyelése

Érdemes három buildet összehasonlítani.

## Első build

```text
minden fut
```

## Source módosítás után

```text
dependency install cache
source copy rebuild
application build rebuild
```

## package.json módosítás után

```text
package copy rebuild
dependency install rebuild
source rebuild
application build rebuild
```

Ez lesz majd a labor egyik fontos része.

---

# 41. Production Next.js példa

Egyszerű build Dockerfile:

```dockerfile
FROM node:22

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm ci

COPY . .

RUN npm run build

CMD ["npm", "start"]
```

Cache-logika:

```text
FROM
↓
stabil

package files
↓
ritkábban változnak

npm ci
↓
dependency cache

source
↓
gyakran változik

npm run build
↓
source változáskor újra fut
```

---

# 42. Development és production build cache

Developmentnél gyakran:

```text
source hoston
↓
bind mount
↓
container dev server
```

Nem kell minden source változáskor image-et buildelni.

Productionnél:

```text
source
↓
docker build
↓
npm run build
↓
production image
```

Itt a build cache nagyon fontos.

---

# 43. Development mental model

```text
HOST SOURCE
↓ bind mount
CONTAINER
↓
npm run dev
```

Ha fájlt módosítunk:

```text
host file változik
↓
container azonnal látja
↓
hot reload
```

Image rebuild nem szükséges minden source módosításnál.

---

# 44. Production mental model

```text
SOURCE
↓
docker build
↓
dependencies
↓
npm run build
↓
IMAGE
↓
docker run
```

Itt a source belekerül az image build folyamatába.

---

# 45. Miért különbözik a development és production Dockerfile?

Development cél:

```text
gyors fejlesztés
hot reload
debugging
bind mount
dev dependencies
```

Production cél:

```text
reprodukálható artifact
kisebb image
kevesebb dependency
kevesebb attack surface
gyors startup
stabil runtime
```

---

# 46. Multi-stage build előkészítés

A következő fontos Dockerfile technika:

```text
multi-stage build
```

Mental model:

```text
BUILD STAGE
↓
Node
dev dependencies
compiler/build tools
source
npm run build
↓
build artifact

FINAL STAGE
↓
csak runtime
+
csak szükséges build artifact
```

---

# 47. Miért jó a multi-stage build?

A builder stage lehet nagy:

```text
source
dev dependencies
compiler
build tooling
```

De ezek nem feltétlenül kellenek production runtime alatt.

Ezért:

```text
builder
→ elkészíti az alkalmazást

runtime
→ csak futtatja
```

---

# 48. Multi-stage alap példa

```dockerfile
FROM node:22 AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build


FROM node:22-alpine

WORKDIR /app

COPY --from=builder /app ./

CMD ["npm", "start"]
```

Ez csak az alapelvet mutatja.

Egy optimalizált Next.js production image ennél általában pontosabb.

---

# 49. `COPY --from`

Multi-stage buildben:

```dockerfile
COPY --from=builder /app ./
```

jelentése:

```text
builder stage
↓
/app
↓
final stage
```

Így a final image-be kiválaszthatjuk, mi kerüljön át.

---

# 50. Miért lehet kisebb a final image?

Mert például:

```text
builder stage
├── compiler
├── dev dependencies
├── source
├── temp build files
└── build output
```

de final stage:

```text
runtime
├── runtime dependency
└── build output
```

A fölösleges build tooling nem kerül át.

---

# 51. Build artifact

A build artifact az alkalmazás build során elkészített futtatható / deployolható eredménye.

Next.js esetén például:

```text
.next/
```

és a pontos production struktúra a Next.js build konfigurációjától függ.

Mental model:

```text
source
↓
build
↓
artifact
↓
runtime
```

---

# 52. Image layer és registry kapcsolat

A layer-ek nemcsak local buildnél számítanak.

Registry push/pull során is:

```text
layer-ek
↓
digest alapján
↓
tárolhatók / újrafelhasználhatók
```

Ezért amikor pusholunk:

```bash
docker push myapp:1.0
```

a már registryben lévő blobokat nem feltétlenül kell újra feltölteni.

---

# 53. Image layer és content-addressing

Layer tartalma hash alapján azonosítható.

Mental model:

```text
layer content
↓
SHA-256 digest
↓
content identity
```

Ez segít:

```text
cache
deduplication
integrity
registry transfer
```

területeken.

---

# 54. Dockerfile optimalizálás alapelvei

Egy jó Dockerfile általában:

```text
1. megfelelő base image-et használ
2. kicsi build contextből dolgozik
3. használ .dockerignore-t
4. ritkán változó dolgokat előre tesz
5. dependency installt jól cache-eli
6. nem tartalmaz secretet
7. nem hagy fölösleges build artifactokat
8. productionnél csak szükséges runtime dolgokat tart meg
```

---

# 55. Rossz minta

```dockerfile
FROM node:22

WORKDIR /app

COPY . .

RUN npm install

RUN npm run build

CMD ["npm", "start"]
```

Mi a probléma?

```text
COPY . .
↓
bármely source változás
↓
npm install cache invalidálódhat
```

---

# 56. Jobb minta

```dockerfile
FROM node:22

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm ci

COPY . .

RUN npm run build

CMD ["npm", "start"]
```

Előny:

```text
source változás
↓
dependency install cache maradhat
```

---

# 57. Még jobb production minta

Később:

```text
multi-stage build
```

segítségével:

```text
builder stage
↓
runtime stage
```

szétválasztható.

Ez kisebb és tisztább production image-et eredményezhet.

---

# 58. Tipikus cache hibakeresés

Ha azt látjuk:

```text
miért fut újra mindig npm ci?
```

kérdések:

```text
Megváltozott package.json?
Megváltozott package-lock.json?
Megváltozott egy előző build step?
Túl korán van COPY . .?
Változik fölösleges fájl a build contextben?
Hiányzik .dockerignore?
--no-cache paramétert használunk?
```

---

# 59. Build cache troubleshooting mental model

```text
Melyik az első step, ami már nem CACHED?
↓
Mi változott annak a bemenetében?
↓
Miért invalidálja az utána következő stepeket?
```

Ugyanaz a gondolkodás, amit networkingnél is használunk:

> mi az utolsó pont, amely biztosan működik, és mi a következő?

---

# 60. Image méret troubleshooting

Ha túl nagy az image:

```text
Milyen base image-et használunk?
↓
Milyen fájlokat COPY-zunk?
↓
Van .dockerignore?
↓
Bekerül node_modules?
↓
Bekerül build tooling?
↓
Marad temp fájl?
↓
Használható multi-stage build?
```

---

# 61. Build context troubleshooting

Ha lassú a build context:

```text
Mi van a projektmappában?
↓
.git?
node_modules?
.next?
logs?
large files?
```

Majd:

```text
.dockerignore
```

ellenőrzése.

---

# 62. Gyakori félreértések

## Minden Dockerfile sor pontosan egy filesystem layer

Nem feltétlenül.

Pontosabb:

```text
Dockerfile instruction
→ build step
```

és bizonyos step-ek filesystem változást hoznak létre.

---

## Cache azt jelenti, hogy Docker megnézi az internetet

Nem.

```text
cache
→ build input / korábbi state alapján
```

nem:

```text
külső világ frissességellenőrzése
```

---

## `--no-cache` frissíti a base image-et

Nem feltétlenül.

Base image frissességhez:

```bash
docker build --pull .
```

---

## Ha egy fájlt később törlünk, biztosan kisebb lesz az image

Nem feltétlenül.

Korábbi layerben a tartalom megmaradhat.

---

## Kevesebb layer mindig jobb

Nem.

A fontosabb:

```text
jó cache
olvashatóság
kis final image
jó build structure
```

---

## `COPY . .` mindig jó az elején

Nem.

Gyakran fölöslegesen invalidálja a dependency cache-t.

---

# 63. Rövid cache cheatsheet

```text
CACHE HIT
→ korábbi build eredmény használható
```

```text
CACHE MISS
→ step újra fut
```

```text
CACHE INVALIDATION
→ valami megváltozott a step bemenetében
```

```text
CACHED
→ Docker nem futtatta újra a stepet
```

---

# 64. Dockerfile sorrend cheatsheet

```text
ritkán változó
↓
előre
```

```text
gyakran változó
↓
hátra
```

Node:

```dockerfile
COPY package.json package-lock.json ./
RUN npm ci

COPY . .
```

---

# 65. Build parancsok cheatsheet

Normál build:

```bash
docker build -t myapp:1.0 .
```

Cache nélkül:

```bash
docker build --no-cache -t myapp:1.0 .
```

Base image frissítéssel:

```bash
docker build --pull -t myapp:1.0 .
```

Mindkettő:

```bash
docker build --pull --no-cache -t myapp:1.0 .
```

---

# 66. Build context cheatsheet

```bash
docker build .
```

```text
.
→ build context
```

Dockerfile:

```dockerfile
COPY . .
```

```text
első .
→ context

második .
→ WORKDIR
```

---

# 67. Layer mental model

```text
IMAGE
│
├── Layer 4
├── Layer 3
├── Layer 2
└── Layer 1
```

Container:

```text
Writable Layer
──────────────
Image Layers
```

---

# 68. Cache mental model

```text
Dockerfile
↓
STEP A
↓
cache result
↓
STEP B
↓
cache result
↓
STEP C
↓
cache result
↓
IMAGE
```

Ha:

```text
STEP B input változik
```

akkor:

```text
A → cache
B → rebuild
C → rebuild / újraértékelés
```

---

# 69. Next.js production mental model

```text
package files
↓
npm ci
↓
source code
↓
npm run build
↓
production artifact
↓
production image
```

---

# 70. Next.js development mental model

```text
Development image
↓
Node runtime / tooling
↓
container
↑
bind mount
↑
host source code
```

Majd:

```text
npm run dev
```

---

# 71. Development vs production

## Development

```text
source változik folyamatosan
↓
bind mount
↓
hot reload
```

## Production

```text
source kész
↓
docker build
↓
build artifact
↓
immutable image
```

---

# 72. Immutable artifact gondolkodás

Productionnél ideális:

```text
build once
↓
same image
↓
test
↓
staging
↓
production
```

Nem:

```text
production gépen kézzel módosítjuk a containert
```

Ez DevOpsban nagyon fontos elv.

---

# 73. Kapcsolat a registry fejezettel

```text
SOURCE
↓
Dockerfile
↓
docker build
↓
IMAGE
↓
docker tag
↓
docker push
↓
REGISTRY
```

A layer/cache mechanizmus a buildet gyorsítja.

A layer/digest mechanizmus a registry push/pull hatékonyságát is segíti.

---

# 74. Kapcsolat a container lifecycle-lel

```text
Dockerfile build
↓
IMAGE
↓
docker run
↓
Container writable layer
↓
CMD / ENTRYPOINT
↓
PID 1
↓
RUNNING
```

Így a Dockerfile build, image layer és container lifecycle egyetlen folyamatként kapcsolódik össze.

---

# 75. Teljes mental model

```text
                     SOURCE CODE
                          │
                          ↓
                     Dockerfile
                          │
                          ↓
                   docker build
                          │
             ┌────────────┴────────────┐
             │                         │
          BUILD CACHE              BUILD STEPS
             │                         │
             └────────────┬────────────┘
                          ↓
                     IMAGE LAYERS
                          │
                          ↓
                         IMAGE
                          │
                          ├── docker push → Registry
                          │
                          └── docker run
                                  ↓
                        Container writable layer
                                  ↓
                              Main process
```

---

# 76. A legfontosabb gondolat

A Dockerfile sorrendje nem csak stíluskérdés.

A sorrend meghatározza:

```text
mennyire jól használható a cache
milyen gyors a build
mikor fut újra dependency install
mennyi fölösleges adat kerül az image-be
mennyire reprodukálható a build
```

Ezért:

> A Dockerfile-t úgy tervezzük, hogy a ritkán változó és drága build lépések cache-e minél tovább újrahasználható legyen.

---

# 77. A következő gyakorlati labor ötlete

Ezt később érdemes konkrétan végigpróbálni.

## 1. Első build

```bash
docker build -t cache-demo:1.0 .
```

Megfigyeljük:

```text
mely step fut le?
```

## 2. Ugyanaz újra

```bash
docker build -t cache-demo:1.0 .
```

Megfigyeljük:

```text
CACHED
```

## 3. Source módosítás

```text
app/page.tsx
```

Új build.

Megnézzük:

```text
npm ci cache-ben marad-e?
```

## 4. package.json módosítás

Új build.

Megnézzük:

```text
npm ci újra lefut-e?
```

## 5. `--no-cache`

```bash
docker build --no-cache -t cache-demo:1.0 .
```

Megfigyeljük a különbséget.

---

# 78. Fejezet végső összefoglaló

```text
Dockerfile
→ build instructions
```

```text
Build step
→ egy Dockerfile instruction feldolgozása
```

```text
Image layer
→ filesystem változások rétege
```

```text
Build cache
→ korábbi build eredmények újrafelhasználása
```

```text
Cache invalidation
→ valamely build input megváltozott
```

```text
.dockerignore
→ build context szűrése
```

```text
--no-cache
→ build cache mellőzése
```

```text
--pull
→ base image frissebb változatának ellenőrzése
```

```text
Multi-stage build
→ build környezet és runtime környezet szétválasztása
```

És a legfontosabb Dockerfile minta Node / Next.js esetén:

```dockerfile
FROM node:22

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .

RUN npm run build

CMD ["npm", "start"]
```

Azért jó ez a sorrend, mert:

```text
dependency fájlok
→ ritkábban változnak
↓
npm ci jól cache-elhető

source
→ gyakran változik
↓
csak a későbbi build step-ek futnak újra
```