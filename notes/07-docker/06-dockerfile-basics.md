# Dockerfile Basics

## 1. Mi az a Dockerfile?

A Dockerfile egy egyszerű szövegfájl, amely leírja, hogyan kell egy Docker image-et felépíteni.

Mental model:

```text id="bkx9qm"
Dockerfile
↓
docker build
↓
Docker image
↓
docker run
↓
Container
```

A Dockerfile tehát nem maga az image, és nem is a container.

Inkább egy:

```text id="4cd35g"
build recipe
```

vagyis egy recept, amelyből Docker image-et készítünk.

---

# 2. Dockerfile mint recept

Jó mental model:

```text id="32vrch"
Dockerfile
→ recept

Image
→ elkészült eredmény

Container
→ az image futó példánya
```

Például:

```text id="aewxui"
Dockerfile
↓
node + application + dependencies
↓
Image
↓
Container
```

---

# 3. Egyszerű Node.js Dockerfile

Példa:

```dockerfile id="1gde4t"
FROM node:22

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

CMD ["npm", "start"]
```

Ez az egyik legegyszerűbb tipikus Node.js Dockerfile.

---

# 4. A Dockerfile fő feladata

A Dockerfile meghatározza többek között:

```text id="n37kfg"
milyen base image-ből indulunk

milyen fájlokat másolunk be

milyen build parancsokat futtatunk

milyen working directory legyen

milyen default runtime command induljon

milyen environment defaultok legyenek

milyen metadata tartozzon az image-hez
```

---

# 5. A legfontosabb Dockerfile utasítások

Ebben a fejezetben főleg ezekkel foglalkozunk:

```text id="zvmo6t"
FROM
WORKDIR
COPY
RUN
CMD
ENV
EXPOSE
ENTRYPOINT
```

A legfontosabb alapok:

```text id="c6j82v"
FROM
WORKDIR
COPY
RUN
CMD
```

---

# 6. `FROM`

A `FROM` megadja, milyen base image-ből indul az új image.

Példa:

```dockerfile id="82lm0x"
FROM node:22
```

Mental model:

```text id="4ad5yh"
node:22
↓
base image
↓
saját módosítások
↓
saját image
```

---

# 7. Mi az a base image?

A base image egy már létező image, amelyre ráépítjük a saját image-ünket.

Például:

```text id="xtf8be"
node:22
```

már tartalmazhatja többek között:

```text id="nx6mhi"
Linux userspace komponensek
Node.js runtime
npm
alap rendszerkönyvtárak
```

Ezért nem kell mindent nulláról felépíteni.

---

# 8. Base image nélkül?

Docker image technikailag teljesen minimális alapról is építhető.

Például:

```dockerfile id="x0m5h2"
FROM scratch
```

A:

```text id="qs8ax3"
scratch
```

egy speciális üres kiindulási pont.

Ez inkább:

```text id="1zhw3o"
nagyon minimalista
statikusan fordított
speciális alkalmazások
```

esetén használatos.

Alap Docker tanulásnál jellemzőbb:

```dockerfile id="fjsi7o"
FROM node:22
```

vagy:

```dockerfile id="eqxbhf"
FROM nginx
```

vagy:

```dockerfile id="n4r855"
FROM ubuntu
```

---

# 9. Fontos: az image-ben nincs saját kernel

Ha ezt írjuk:

```dockerfile id="njrh5m"
FROM ubuntu:24.04
```

az nem azt jelenti, hogy egy teljes Ubuntu virtuális gépet kapunk.

Az image főleg:

```text id="7c6wsh"
Ubuntu userspace
filesystem
libraries
commands
metadata
```

részeket tartalmaz.

A kernel továbbra is a host / backend Linux kernelből jön.

Mental model:

```text id="oqnnyk"
Ubuntu image
→ userspace

Linux host
→ kernel
```

---

# 10. `WORKDIR`

Példa:

```dockerfile id="i9p2dm"
WORKDIR /app
```

A `WORKDIR` beállítja az image-en belüli aktuális working directory-t.

Mental model:

```text id="ig4gwm"
WORKDIR /app
↓
későbbi utasítások innen dolgoznak
```

---

# 11. `WORKDIR` és `cd`

Gondolkodás szintjén hasonló:

```bash id="itlh5g"
cd /app
```

működéshez.

De nem ugyanaz a parancs.

A Dockerfile-ban:

```dockerfile id="ya9a54"
WORKDIR /app
```

metadata/build context szinten állítja be a munkakönyvtárat.

---

# 12. `WORKDIR` automatikusan létrehozhat könyvtárat

Ha:

```dockerfile id="0vs34o"
WORKDIR /app
```

és az `/app` még nem létezik, Docker létrehozhatja.

Ezért általában nem kell előtte:

```dockerfile id="923w9l"
RUN mkdir /app
```

---

# 13. Több `WORKDIR`

Lehetséges:

```dockerfile id="le4uy1"
WORKDIR /app
WORKDIR src
```

Ekkor a második relatív path.

A végeredmény:

```text id="n774b4"
/app/src
```

Általában egyszerűbb Dockerfile-ban egyértelmű abszolút pathokat használni.

---

# 14. `COPY`

A `COPY` fájlokat másol:

```text id="xuxyee"
build contextből
↓
image filesystembe
```

Példa:

```dockerfile id="c8o9av"
COPY package.json .
```

---

# 15. `COPY` általános formája

```dockerfile id="03wzmj"
COPY <source> <destination>
```

Például:

```dockerfile id="s6pr9f"
COPY package.json /app/package.json
```

---

# 16. Relatív destination

Ha előtte:

```dockerfile id="x3bccm"
WORKDIR /app
```

akkor:

```dockerfile id="s0rexc"
COPY package.json .
```

jelentése:

```text id="2mrcrj"
source:
package.json

destination:
/app/package.json
```

---

# 17. Mit jelent a `.`?

A `.` jelentése mindig kontextusfüggő.

Például:

```dockerfile id="le5ye2"
COPY . .
```

Bal oldalon:

```text id="tk9jm9"
.
→ build context root
```

Jobb oldalon:

```text id="0hri6r"
.
→ aktuális WORKDIR
```

Tehát ha:

```dockerfile id="jyrpjn"
WORKDIR /app
COPY . .
```

akkor:

```text id="x5cptg"
build context
↓
/app
```

---

# 18. Mi az a build context?

Amikor ezt futtatjuk:

```bash id="a2za3i"
docker build .
```

a végén lévő:

```text id="pj5gmj"
.
```

a build context.

Ez azt mondja Dockernek:

> ebből a könyvtárból dolgozhatsz a build során.

---

# 19. Build context példa

Projekt:

```text id="n4duhw"
myapp/
├── Dockerfile
├── package.json
├── package-lock.json
├── src/
└── README.md
```

Ha:

```bash id="eu32mx"
cd myapp
docker build .
```

akkor:

```text id="hp4h93"
myapp/
```

lesz a build context.

---

# 20. Miért fontos a build context?

A Dockerfile:

```dockerfile id="7du2u5"
COPY src ./src
```

csak olyan fájlt tud használni, amely a build context része.

Nem lehet tetszőlegesen:

```text id="2jtx7h"
akárhonnan a host filesystemből
```

másolni.

Ez biztonsági és reprodukálhatósági szempontból is fontos.

---

# 21. Build context nem egyenlő Dockerfile könyvtár

Gyakran ugyanaz a könyvtár, de nem kötelező.

Lehetséges például:

```bash id="eqpsq4"
docker build -f docker/Dockerfile .
```

Itt:

```text id="02sx66"
Dockerfile:
docker/Dockerfile
```

de:

```text id="j7vgyt"
build context:
.
```

---

# 22. `COPY package*.json ./`

Példa:

```dockerfile id="2nvnh7"
COPY package*.json ./
```

A wildcard:

```text id="xueclx"
package*.json
```

illeszkedhet például:

```text id="7o9gl3"
package.json
package-lock.json
```

fájlokra.

---

# 23. `RUN`

A `RUN` build időben futtat parancsot.

Példa:

```dockerfile id="e2nd0v"
RUN npm install
```

Flow:

```text id="yio348"
docker build
↓
RUN npm install
↓
npm fut
↓
dependencies települnek
↓
eredmény bekerül az image-be
```

---

# 24. `RUN` build-time utasítás

Ez nagyon fontos:

```text id="cf930k"
RUN
→ BUILD TIME
```

Nem container induláskor történik.

Például:

```dockerfile id="0h1w6m"
RUN npm install
```

egyszer lefut az image build során.

Utána a kész image már tartalmazza a telepített dependency-ket.

---

# 25. `RUN` és image layer

A Docker build során sok Dockerfile utasítás új image layert vagy build-state változást hozhat létre.

Például:

```dockerfile id="fr48x7"
RUN npm install
```

eredményeként az image filesystemében megjelenhet:

```text id="lv5c7u"
node_modules/
```

Ez bekerül a build eredményébe.

A layer működést a következő fejezetben részletesebben nézzük.

---

# 26. `RUN` példák

Package telepítés:

```dockerfile id="ra6xvp"
RUN npm install
```

Linux package telepítés:

```dockerfile id="dv2upz"
RUN apt-get update
```

Build:

```dockerfile id="3jip2v"
RUN npm run build
```

Fájlművelet:

```dockerfile id="3skut0"
RUN mkdir /app/data
```

---

# 27. Shell form `RUN`

Példa:

```dockerfile id="bsn0zp"
RUN npm install
```

Ez shell form.

Linux container esetén tipikusan shellen keresztül fut.

Mental model:

```text id="knv8ow"
RUN command
↓
shell
↓
command execution
```

---

# 28. Exec form `RUN`

Létezik JSON / exec forma is:

```dockerfile id="rq2x9s"
RUN ["npm", "install"]
```

A gyakorlatban `RUN` esetén nagyon gyakori a shell form:

```dockerfile id="cijkom"
RUN npm install
```

A `CMD` és `ENTRYPOINT` esetén viszont különösen fontos lesz az exec form.

---

# 29. `CMD`

A `CMD` meghatározza a container alapértelmezett induló parancsát.

Példa:

```dockerfile id="m37h3c"
CMD ["npm", "start"]
```

Fontos:

```text id="7vi3ol"
CMD
→ nem build time
→ runtime default
```

---

# 30. `CMD` build közben

Amikor:

```bash id="y30n3d"
docker build .
```

a:

```dockerfile id="z77r07"
CMD ["npm", "start"]
```

nem fut le.

Docker csak eltárolja az image configban.

Mental model:

```text id="ltxi4q"
docker build
↓
CMD metadata
↓
image
```

---

# 31. `CMD` runtime

Majd:

```bash id="hcochy"
docker run myapp
```

esetén:

```text id="l2wxhx"
image config
↓
CMD
↓
npm start
```

lefut.

---

# 32. `RUN` vs `CMD`

Ez az egyik legfontosabb Dockerfile különbség.

```text id="66aw07"
RUN
→ build time
```

```text id="4xwir0"
CMD
→ runtime default
```

Példa:

```dockerfile id="m59f90"
RUN npm install
CMD ["npm", "start"]
```

Jelentése:

```text id="rw8e63"
BUILD:
npm install

RUN:
npm start
```

---

# 33. Mi lesz a main process?

Ha:

```dockerfile id="knwx47"
CMD ["node", "server.js"]
```

akkor container induláskor:

```text id="nm24cr"
node server.js
```

lesz a fő alkalmazás process.

Mental model:

```text id="hby7at"
docker run
↓
CMD
↓
node server.js
↓
PID 1
↓
container RUNNING
```

---

# 34. Kapcsolat a lifecycle fejezettel

Ha a:

```text id="82an4a"
node server.js
```

process leáll:

```text id="g1rpyw"
PID 1 exit
↓
container EXITED
```

Tehát Dockerfile és container lifecycle közvetlenül összekapcsolódik.

---

# 35. `CMD` exec form

Ajánlott forma:

```dockerfile id="1shah6"
CMD ["node", "server.js"]
```

Ez:

```text id="0eajqw"
exec form
```

---

# 36. `CMD` shell form

Létezik:

```dockerfile id="j6mqal"
CMD node server.js
```

is.

Ez:

```text id="k28fnl"
shell form
```

---

# 37. Miért előnyös az exec form?

Exec form esetén a process közvetlenebbül indul.

Ez container lifecycle és signal handling szempontból fontos.

Például:

```dockerfile id="x7g1cc"
CMD ["node", "server.js"]
```

általában jobb containeres mental model, mint:

```dockerfile id="vwzx0t"
CMD node server.js
```

A signalkezelést később részletesebben is érintjük.

---

# 38. `CMD` felülírható

Ha az image-ben:

```dockerfile id="il3rqi"
CMD ["npm", "start"]
```

van, akkor:

```bash id="yvobtl"
docker run myimage npm test
```

a runtime command felülírhatja az alapértelmezett CMD-t.

Mental model:

```text id="6u3pkc"
Image CMD
→ default

docker run command
→ override
```

---

# 39. `ENV`

Environment variable default megadása:

```dockerfile id="xnrus6"
ENV NODE_ENV=production
```

Ez bekerül az image configjába.

Container indulásakor elérhető lehet:

```text id="18pmws"
NODE_ENV=production
```

---

# 40. `ENV` nem secret storage

Nagyon fontos:

```text id="a0kj72"
ENV
≠
biztonságos secret tárolás
```

Nem érdemes image-be beégetni:

```text id="0ephh5"
password
API key
token
private key
```

értékeket.

Az image:

```text id="ms3njc"
registrybe kerülhet
más gépre kerülhet
inspectálható
layer history része lehet
```

---

# 41. Runtime environment override

Ha image-ben:

```dockerfile id="fhazne"
ENV NODE_ENV=production
```

van, runtime felülírható például:

```bash id="8qru6n"
docker run -e NODE_ENV=development myapp
```

Tehát:

```text id="le0vsx"
Dockerfile ENV
→ default
```

```text id="s6gm1g"
runtime -e
→ override
```

---

# 42. `EXPOSE`

Példa:

```dockerfile id="jgmlh9"
EXPOSE 3000
```

Ez azt dokumentálja, hogy az alkalmazás várhatóan melyik portot használja.

Nagyon fontos:

> Az `EXPOSE` önmagában nem publikálja a portot a hostra.

---

# 43. `EXPOSE` vs `-p`

Dockerfile:

```dockerfile id="i6i9m8"
EXPOSE 3000
```

csak image metadata / dokumentációs jellegű információ.

A hostra publikáláshoz:

```bash id="ja99wc"
docker run -p 8080:3000 myapp
```

kell.

Mental model:

```text id="0y90a6"
EXPOSE
→ container alkalmazás port információ
```

```text id="ii1sr4"
-p
→ host ↔ container port mapping
```

---

# 44. Port példa

Container:

```text id="z7i3by"
Node app
↓
listen 3000
```

Dockerfile:

```dockerfile id="dcwe4x"
EXPOSE 3000
```

Runtime:

```bash id="48gtxk"
docker run -p 8080:3000 myapp
```

Flow:

```text id="q4719c"
Host :8080
↓
Docker port mapping
↓
Container :3000
↓
Node app
```

---

# 45. `ENTRYPOINT`

Az `ENTRYPOINT` meghatározhatja a container alapvető executable-jét.

Példa:

```dockerfile id="cpw03n"
ENTRYPOINT ["node"]
```

és:

```dockerfile id="hnj87j"
CMD ["server.js"]
```

együtt:

```text id="zeh27w"
node server.js
```

---

# 46. `ENTRYPOINT` és `CMD`

Egyszerű mental model:

```text id="1wphuk"
ENTRYPOINT
→ mit futtatunk?
```

```text id="3u8if7"
CMD
→ alapértelmezett argumentum / parancs
```

Ez leegyszerűsítés, de kezdésnek jól használható.

---

# 47. `ENTRYPOINT` példa

Dockerfile:

```dockerfile id="47u4k1"
ENTRYPOINT ["echo"]
CMD ["Hello"]
```

Container:

```bash id="9ux60s"
docker run myimage
```

eredmény:

```text id="e14hln"
echo Hello
```

Ha:

```bash id="dvoklt"
docker run myimage Docker
```

akkor:

```text id="x0guau"
echo Docker
```

lehet az eredmény.

---

# 48. `CMD` vs `ENTRYPOINT`

Alapszinten:

```text id="fewlws"
CMD
→ könnyen felülírható default command
```

```text id="ges5dk"
ENTRYPOINT
→ container fő executable-jének rögzítésére használható
```

A részletes viselkedést később gyakorlatban is megnézzük.

---

# 49. Egyszerű Node Dockerfile részletesen

```dockerfile id="x6trm2"
FROM node:22

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

---

# 50. Első lépés

```dockerfile id="gftt66"
FROM node:22
```

Jelentése:

```text id="0tw31f"
base image
↓
Node.js runtime környezet
```

---

# 51. Második lépés

```dockerfile id="2tgx0m"
WORKDIR /app
```

Jelentése:

```text id="uw5byf"
working directory
=
/app
```

---

# 52. Harmadik lépés

```dockerfile id="puk3b3"
COPY package*.json ./
```

Jelentése:

```text id="5u4kh7"
package.json
package-lock.json
↓
/app
```

---

# 53. Negyedik lépés

```dockerfile id="ri5tp7"
RUN npm install
```

Build közben:

```text id="bj60t2"
npm install
↓
node_modules
↓
image filesystem
```

---

# 54. Ötödik lépés

```dockerfile id="a6aod7"
COPY . .
```

A projekt többi része bekerül:

```text id="jrfyzy"
src/
public/
config/
stb.
↓
/app
```

---

# 55. Hatodik lépés

```dockerfile id="80upki"
EXPOSE 3000
```

Jelzi:

```text id="3eqngj"
application várható portja
=
3000
```

De port mapping még nincs.

---

# 56. Hetedik lépés

```dockerfile id="28u1yr"
CMD ["npm", "start"]
```

Container indulásakor:

```text id="2jiur2"
npm start
↓
Node process
↓
PID 1
↓
container RUNNING
```

---

# 57. Teljes build flow

```text id="ityf3e"
docker build
↓
FROM node:22
↓
WORKDIR /app
↓
COPY package files
↓
RUN npm install
↓
COPY source
↓
EXPOSE metadata
↓
CMD metadata
↓
IMAGE elkészül
```

---

# 58. Runtime flow

Később:

```bash id="ebgk4l"
docker run myapp
```

Flow:

```text id="0tpxrv"
IMAGE
↓
container create
↓
container start
↓
CMD
↓
npm start
↓
Node application
↓
RUNNING
```

---

# 59. `docker build`

Image build indítása:

```bash id="u5scs4"
docker build .
```

A:

```text id="edjpy8"
.
```

a build context.

---

# 60. Image név és tag build közben

Gyakoribb:

```bash id="r49bur"
docker build -t myapp:1.0 .
```

A:

```text id="frsuoe"
-t
```

tag / name beállítására szolgál.

Eredmény:

```text id="yx9iyh"
myapp:1.0
```

local image.

---

# 61. Build után

Ellenőrzés:

```bash id="50g2s3"
docker image ls
```

Megjelenhet:

```text id="bnq6sh"
REPOSITORY   TAG   IMAGE ID
myapp        1.0   ...
```

---

# 62. Dockerfile fájlnév

Alapértelmezett fájlnév:

```text id="s68its"
Dockerfile
```

Nincs:

```text id="p0oa0b"
.txt
```

kiterjesztés.

---

# 63. Más Dockerfile használata

Lehet például:

```text id="8m40p6"
Dockerfile.dev
Dockerfile.prod
```

és:

```bash id="dnb33r"
docker build -f Dockerfile.dev .
```

A:

```text id="4ii5l8"
-f
```

megadja a Dockerfile elérési útját.

---

# 64. `.dockerignore`

A build contextből bizonyos fájlokat ki lehet zárni:

```text id="70sb5c"
.dockerignore
```

Például:

```text id="ptyfjr"
node_modules
.git
.env
*.log
```

---

# 65. Miért kell `.dockerignore`?

Ha:

```dockerfile id="4li1t8"
COPY . .
```

akkor nem szeretnénk automatikusan bemásolni:

```text id="5b80wm"
.git/
node_modules/
secret fájlok
logok
temp fájlok
```

mindent.

---

# 66. `.dockerignore` előnye

Kisebb build context:

```text id="38mntz"
kevesebb adat
↓
gyorsabb build
```

Kevesebb fölösleges fájl:

```text id="xklc9r"
kisebb image
```

Kevesebb véletlenül bemásolt érzékeny adat:

```text id="rfrlqz"
jobb security
```

---

# 67. Fontos secret példa

Ha van:

```text id="1yfzyu"
.env
```

benne:

```text id="73dkzv"
DATABASE_PASSWORD=...
API_KEY=...
```

akkor nem szeretnénk:

```dockerfile id="0s1a1q"
COPY . .
```

sorral véletlenül image-be tenni.

Ezért például:

```text id="r1rt4l"
.dockerignore
```

tartalma:

```text id="znz1ix"
.env
```

---

# 68. Build-time vs runtime

Nagyon fontos gondolkodási modell:

```text id="8ofo8e"
BUILD TIME
│
├── FROM
├── WORKDIR
├── COPY
├── RUN
├── ENV
├── metadata
└── IMAGE
```

Majd:

```text id="lkh21l"
RUNTIME
│
├── docker run
├── container
├── CMD / ENTRYPOINT
└── application process
```

---

# 69. Dockerfile nem deployment file

A Dockerfile megmondja:

```text id="ljlelk"
hogyan épüljön az image
```

De nem feltétlenül mondja meg:

```text id="fvtcbi"
hány container fusson
milyen networkön
milyen database legyen mellette
milyen volume-ok legyenek
milyen restart policy legyen
```

Ezekhez később:

```text id="my6pyw"
docker run options
Docker Compose
Kubernetes
```

jellegű eszközök jönnek.

---

# 70. Dockerfile vs Docker Compose

Előzetes mental model:

```text id="o1oahv"
Dockerfile
→ hogyan épül egy image?
```

```text id="dmnr4c"
Docker Compose
→ hogyan fusson több container együtt?
```

Például:

```text id="day88p"
Dockerfile
→ Node app image
```

```text id="q0340n"
Compose
→ Node app + PostgreSQL + network + volume
```

---

# 71. Gyakori félreértések

## `RUN` container induláskor fut

Nem.

```text id="f9zmau"
RUN
→ build time
```

---

## `CMD` build közben fut

Nem.

```text id="628u4s"
CMD
→ runtime default
```

---

## `FROM ubuntu` teljes VM-et jelent

Nem.

```text id="yqpxmr"
Ubuntu userspace
≠
Ubuntu VM
```

---

## `EXPOSE` publikálja a host portot

Nem.

```text id="yr9vtb"
EXPOSE
→ metadata
```

Host mapping:

```bash id="f3wt48"
-p 8080:3000
```

---

## `COPY . .` mindenhonnan tud másolni

Nem.

A source:

```text id="ikrrkn"
build context
```

része kell legyen.

---

## `.dockerignore` csak performance miatt kell

Nem.

Segít:

```text id="9p4qg4"
performance
image size
security
build cache
```

területeken is.

---

# 72. Egyszerű Dockerfile mental model

```text id="novw6h"
FROM
↓
miből indulunk?

WORKDIR
↓
hol dolgozunk?

COPY
↓
mit teszünk bele?

RUN
↓
mit hajtunk végre build közben?

ENV
↓
milyen default environment legyen?

EXPOSE
↓
milyen portot használ az app?

CMD / ENTRYPOINT
↓
mi indul container runtime alatt?
```

---

# 73. Node.js példa teljes folyamat

Projekt:

```text id="dgg340"
myapp/
├── Dockerfile
├── .dockerignore
├── package.json
├── package-lock.json
└── src/
```

Dockerfile:

```dockerfile id="vbaf8l"
FROM node:22

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

Build:

```bash id="tzf98c"
docker build -t myapp:1.0 .
```

Run:

```bash id="zy7b2n"
docker run -p 8080:3000 myapp:1.0
```

Flow:

```text id="o8pjoj"
source code
↓
Dockerfile
↓
docker build
↓
myapp:1.0 image
↓
docker run
↓
container
↓
npm start
↓
Node app
```

---

# 74. Kapcsolat az előző fejezettel

Az előző fejezetben:

```text id="jf3myq"
IMAGE
↓
tag
↓
registry
```

Most megtanultuk:

```text id="gbdbbb"
SOURCE CODE
↓
Dockerfile
↓
docker build
↓
IMAGE
```

A kettő együtt:

```text id="1iwl4i"
SOURCE CODE
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

---

# 75. Kapcsolat a container lifecycle-lel

```text id="9zzp27"
Dockerfile
↓
CMD
↓
Image metadata
↓
docker run
↓
main process
↓
PID 1
↓
RUNNING
```

Ha a main process kilép:

```text id="ylvmk8"
PID 1 exit
↓
container EXITED
```

---

# 76. Legfontosabb parancsok

Build:

```bash id="z7pr5p"
docker build .
```

Build névvel:

```bash id="47anfc"
docker build -t myapp:1.0 .
```

Más Dockerfile:

```bash id="hf8rpo"
docker build -f Dockerfile.dev -t myapp:dev .
```

Image lista:

```bash id="98ppgm"
docker image ls
```

Container indítás:

```bash id="m20l57"
docker run myapp:1.0
```

Port mappinggel:

```bash id="pqk2ce"
docker run -p 8080:3000 myapp:1.0
```

---

# 77. Rövid cheatsheet

```text id="x5ocsl"
FROM
→ base image
```

```text id="0q8oy0"
WORKDIR
→ working directory
```

```text id="qagx56"
COPY
→ build context → image filesystem
```

```text id="8n00fh"
RUN
→ build-time command
```

```text id="m6z1kl"
ENV
→ default environment variable
```

```text id="91tkxt"
EXPOSE
→ port metadata
```

```text id="08f89d"
CMD
→ default runtime command
```

```text id="teivlr"
ENTRYPOINT
→ fő executable meghatározása
```

```text id="091nll"
.dockerignore
→ build context szűrése
```

---

# 78. A legfontosabb különbség

```text id="z2ijol"
RUN
→ image build
```

```text id="siwd03"
CMD / ENTRYPOINT
→ container runtime
```

Ezt mindig tartsuk fejben.

---

# 79. Végső mental model

```text id="ltiu8p"
                    SOURCE CODE
                         │
                         ↓
                     Dockerfile
                         │
                         │ docker build
                         ↓
                       IMAGE
                         │
                         │ docker run
                         ↓
                     CONTAINER
                         │
                         ↓
                CMD / ENTRYPOINT
                         │
                         ↓
                  MAIN PROCESS
                         │
                         ↓
                       PID 1
```

A Dockerfile tehát:

> egy deklaratív build leírás, amelyből reprodukálható Docker image-et készítünk.

---

# 80. Mi jön ezután?

A következő fejezet:

```text id="3whx1q"
07-dockerfile-build-and-layers.md
```

Ott azt nézzük meg részletesen:

```text id="xmh68a"
image layer-ek
build cache
Dockerfile instruction order
cache invalidation
COPY sorrend
RUN layer-ek
miért fontos a package.json előbb?
miért nem jó mindig COPY . . az elején?
image size optimalizálás alapjai
```

Ez fogja megmagyarázni, miért írják a tipikus Node Dockerfile-t így:

```dockerfile id="n5uhp3"
COPY package*.json ./
RUN npm install
COPY . .
```

és miért nem így:

```dockerfile id="18yhmw"
COPY . .
RUN npm install
```