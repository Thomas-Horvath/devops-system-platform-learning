# Docker Storage

## 1. Mi ennek a fejezetnek a célja?

Ebben a fejezetben azt nézzük meg, hogyan kezeli Docker a containerben keletkező adatokat.

A fő kérdés:

> Hol él egy adat, és mi történik vele, ha a container leáll vagy törlődik?

A legfontosabb storage típusok:

```text
1. Container writable layer
2. Bind mount
3. Volume
4. tmpfs
```

Ezek eltérő életciklussal és felhasználási céllal rendelkeznek.

---

# 2. Image filesystem és container writable layer

A Docker image read-only jellegű layer-ekből áll.

Például:

```text
IMAGE

Layer 4
Layer 3
Layer 2
Layer 1
```

Amikor ebből containert indítunk:

```bash
docker run myapp
```

Docker a read-only image layer-ek fölé egy külön írható réteget ad.

```text
Container writable layer
────────────────────────
Image Layer 4
Image Layer 3
Image Layer 2
Image Layer 1
```

Ez:

```text
container writable layer
```

---

# 3. Mi kerül a writable layerbe?

Ha futó containerben:

- új fájl jön létre,
- fájl módosul,
- fájl törlődik,

akkor ezek a változások a container writable layeréhez kapcsolódnak.

Például:

```bash
echo "hello" > /tmp/test.txt
```

A:

```text
/tmp/test.txt
```

a container írható rétegében jelenik meg.

---

# 4. Writable layer és container lifecycle

Ha a containert csak leállítjuk:

```bash
docker stop mycontainer
```

akkor:

```text
RUNNING
↓
EXITED
```

de maga a container továbbra is létezik.

Ezért a writable layer is megmarad.

Ha ugyanazt a containert újraindítjuk:

```bash
docker start mycontainer
```

a korábban létrehozott adatok továbbra is ott lehetnek.

Tehát:

```text
STOP
≠
DATA DELETE
```

---

# 5. Mi történik container törléskor?

Ha:

```bash
docker rm mycontainer
```

akkor maga a container objektum törlődik.

Ezzel együtt a hozzá tartozó writable layer is megszűnik.

```text
Container
├── metadata
├── configuration
├── runtime state
└── writable layer
        ↓
      DELETE
```

Ezért:

```text
container writable layer
→ containerhez kötött
```

---

# 6. Miért probléma ez persistent adatoknál?

Tegyük fel, hogy PostgreSQL fut egy containerben.

Az adatfájlok:

```text
/var/lib/postgresql/data
```

alatt vannak.

Ha ezek kizárólag a container writable layerében vannak:

```text
PostgreSQL container
↓
writable layer
↓
database files
```

majd:

```bash
docker rm postgres
```

akkor az adat is elveszhet.

Ezért fontos a:

```text
persistent storage
```

---

# 7. Container lifecycle és data lifecycle szétválasztása

Ez az egyik legfontosabb Docker storage elv.

Container lifecycle:

```text
create
↓
start
↓
stop
↓
remove
```

Persistent data lifecycle:

```text
create
↓
használat
↓
container cserélődik
↓
adat megmarad
```

Tehát:

```text
container lifecycle
≠
data lifecycle
```

---

# 8. A három fő persistent storage megoldás

Alapszinten:

```text
Container writable layer
Bind mount
Volume
```

Kiegészítve:

```text
tmpfs
```

A különbségek:

```text
Writable layer
→ containerhez kötött

Bind mount
→ host konkrét filesystem path

Volume
→ Docker-managed persistent storage

tmpfs
→ memóriaalapú, nem persistent storage
```

---

# 9. Bind mount

A bind mount segítségével egy konkrét host könyvtárat vagy fájlt kapcsolunk be a container filesystemébe.

Például:

```text
HOST

/home/thomas/projects/wordy
            │
            │ bind mount
            ↓

CONTAINER

/app
```

A container `/app` alatt ugyanazokat a fájlokat látja.

---

# 10. Bind mount developmentnél

Tipikus Next.js development setup:

```text
HOST
│
└── project source
       │
       │ bind mount
       ↓
CONTAINER
│
└── /app
       ↓
   npm run dev
```

Ha VS Code-ban módosítunk:

```text
app/page.tsx
```

a container rögtön látja a változást.

Ezért a bind mount különösen jó:

```text
development
source code
configuration
```

esetén.

---

# 11. Bind mount rövid szintaxissal

Példa Linux / WSL környezetben:

```bash
docker run -v "$PWD:/app" myimage
```

Felbontva:

```text
$PWD
→ host jelenlegi könyvtár

/app
→ container path
```

---

# 12. Bind mount `--mount` szintaxissal

Olvashatóbb forma:

```bash
docker run \
  --mount type=bind,source="$PWD",target=/app \
  myimage
```

Felbontva:

```text
type=bind
→ bind mount

source
→ host path

target
→ container path
```

---

# 13. Bind mount adat lifecycle

Az adat fizikailag a host filesystemen található.

```text
Host filesystem
↓
bind mount
↓
Container
```

Ha a containert töröljük:

```bash
docker rm mycontainer
```

a host fájlok megmaradnak.

Tehát:

```text
container delete
↓
bind mount data
→ megmarad
```

---

# 14. Bind mount előnye

A host közvetlenül hozzáfér az adathoz.

Ez ideális például:

```text
source code
config files
static development files
```

esetén.

---

# 15. Bind mount hátránya

A konfiguráció kötődhet konkrét host pathhoz.

Például:

```text
/home/thomas/projects/app
```

másik gépen lehet, hogy nem létezik.

Ezért:

```text
bind mount
→ host-specific lehet
```

---

# 16. Volume

A Docker volume egy Docker által kezelt persistent storage object.

Létrehozás:

```bash
docker volume create postgres-data
```

Eredmény:

```text
postgres-data
```

egy külön Docker volume objektum.

---

# 17. Named volume

Ha saját nevet adunk:

```bash
docker volume create postgres-data
```

akkor:

```text
postgres-data
```

egy named volume.

Ez sokkal könnyebben kezelhető, mint egy automatikusan generált azonosító.

---

# 18. Volume használata

Példa PostgreSQL-lel:

```bash
docker run \
  --mount type=volume,source=postgres-data,target=/var/lib/postgresql/data \
  postgres
```

Mental model:

```text
Docker Volume
postgres-data
      │
      ↓
Container
/var/lib/postgresql/data
```

---

# 19. Volume rövid `-v` formában

Példa:

```bash
docker run \
  -v postgres-data:/var/lib/postgresql/data \
  postgres
```

Felbontva:

```text
postgres-data
→ volume name

/var/lib/postgresql/data
→ container path
```

---

# 20. Volume külön Docker object

Nagyon fontos:

```text
Container
≠
Volume
```

Például:

```text
Container A
↓
postgres-data
```

Container A törölhető:

```text
Container A
→ deleted
```

de:

```text
postgres-data
→ továbbra is létezhet
```

---

# 21. Új container ugyanazzal a volume-mal

Például:

```text
Postgres Container A
↓
postgres-data
```

Container A törlődik.

Majd:

```text
Postgres Container B
↓
postgres-data
```

Container B ugyanazt az adatot látja.

Ez a persistent storage egyik legfontosabb előnye.

---

# 22. PostgreSQL példa

Volume:

```text
postgres-data
```

Tartalom:

```text
database files
tables
indexes
WAL / internal database state
```

Container:

```text
postgres:17
↓
/var/lib/postgresql/data
```

Mount:

```text
postgres-data
↓
/var/lib/postgresql/data
```

A database adat így nincs a container életciklusához kötve.

---

# 23. Miért jó volume adatbázishoz?

A Docker kezeli a storage fizikai helyét.

Nekünk nem kell közvetlenül megadni például:

```text
/home/thomas/postgres-data
```

vagy:

```text
C:\database\postgres
```

pathot.

Csak:

```text
postgres-data
```

logikai nevet használunk.

---

# 24. Volume és host filesystem

A volume adatai továbbra is valamilyen fizikai storage-on találhatók.

A különbség az, hogy:

```text
bind mount
→ mi választjuk ki a host pathot
```

```text
volume
→ Docker kezeli a storage helyét
```

---

# 25. Docker Desktop és volume

Docker Desktop alatt a volume nem feltétlenül jelenik meg egyszerű Windows könyvtárként.

A storage a Docker Desktop Linux backend környezetében lehet.

Ezért Windows + Docker Desktop alatt a jó mental model:

```text
named volume
→ Docker-managed storage object
```

Nem:

```text
volume
→ biztosan egy kézzel kezelt C:\... könyvtár
```

---

# 26. Volume listázása

```bash
docker volume ls
```

Példa:

```text
DRIVER    VOLUME NAME
local     postgres-data
local     app-uploads
```

---

# 27. Volume vizsgálata

```bash
docker volume inspect postgres-data
```

A Docker részletes metadata-t ad a volume-ról.

Például:

```text
name
driver
mountpoint
labels
scope
```

---

# 28. Volume törlése

```bash
docker volume rm postgres-data
```

Ez már az adat lifecycle-ját érinti.

Fontos:

```text
docker rm container
→ container törlés
```

```text
docker volume rm volume
→ persistent storage törlés
```

A kettőt mindig külön kezeljük.

---

# 29. Volume törlés = adatvesztés lehet

Ha:

```bash
docker volume rm postgres-data
```

akkor a volume-ban lévő adat is törlődhet.

Ezért volume törlés előtt mindig kérdezzük meg:

```text
Biztosan nincs rá szükség?
Van backup?
Használja még valami?
```

---

# 30. Használatban lévő volume

Docker általában nem engedi egyszerűen törölni a volume-ot, ha egy container még használja.

Ez bizonyos védelmet ad véletlen törlés ellen.

De:

```text
force
prune
cleanup
```

műveleteknél továbbra is fontos tudni, mit csinálunk.

---

# 31. `docker volume prune`

Nem használt volume-ok takarítása:

```bash
docker volume prune
```

Ez veszélyes lehet, ha nem értjük pontosan, mi számít már nem használt volume-nak.

Ezért:

```text
prune
→ nem rutinparancs
→ előbb ellenőrzünk
```

---

# 32. Anonymous volume

Lehetséges név nélkül is volume-ot használni.

Például:

```bash
docker run -v /data myimage
```

Docker automatikusan generál egy volume azonosítót.

Ez:

```text
anonymous volume
```

---

# 33. Named vs anonymous volume

```text
Named volume
→ postgres-data
→ ember számára érthető
```

```text
Anonymous volume
→ automatikusan generált ID
→ nehezebben kezelhető
```

Tanulásnál és tipikus production setupban gyakran jobb:

```text
named volume
```

---

# 34. Bind mount vs volume

## Bind mount

```text
Host konkrét path
↓
Container
```

Előny:

```text
hostról közvetlenül hozzáférünk
```

Tipikus:

```text
source code
configuration
```

---

## Volume

```text
Docker-managed storage
↓
Container
```

Előny:

```text
nem kötődik közvetlen host pathhoz
Docker kezeli
```

Tipikus:

```text
database
persistent application data
uploads
```

---

# 35. Egyszerű döntési szabály

Ha az adatot hostról rendszeresen szerkeszteni akarjuk:

```text
bind mount
```

Ha az adatot főleg az alkalmazás kezeli:

```text
volume
```

Ez nem abszolút szabály, de nagyon jó kezdő mental model.

---

# 36. Read-only bind mount

Mountot read-only módban is lehet használni.

Példa:

```bash
docker run \
  --mount type=bind,source="$PWD/config",target=/app/config,readonly \
  myapp
```

Ekkor:

```text
container
→ olvashat
→ nem írhat
```

---

# 37. `:ro` rövid szintaxis

```bash
docker run \
  -v "$PWD/config:/app/config:ro" \
  myapp
```

A:

```text
ro
```

jelentése:

```text
read-only
```

A:

```text
rw
```

jelentése:

```text
read-write
```

és jellemzően ez az alapértelmezett.

---

# 38. Mikor jó a read-only mount?

Például:

```text
configuration
certificate
reference file
static config
```

esetén.

Mental model:

```text
containernek olvasnia kell
↓
módosítania nem
↓
read-only mount
```

---

# 39. Nginx config példa

Host:

```text
./nginx.conf
```

Container:

```text
/etc/nginx/nginx.conf
```

Mount:

```text
./nginx.conf
↓
read-only
↓
/etc/nginx/nginx.conf
```

Így nginx olvashatja a configot, de nem módosíthatja a host fájlt.

---

# 40. Mount elfedheti az image tartalmát

Ez egy nagyon fontos tulajdonság.

Tegyük fel, hogy az image-ben:

```text
/app
├── fileA
└── fileB
```

van.

Majd `/app` fölé bind mountolunk:

```text
HOST DIRECTORY
├── fileX
└── fileY
```

A container `/app` alatt ezután:

```text
fileX
fileY
```

fájlokat látja.

Az image eredeti tartalma nincs feltétlenül törölve.

A mount:

```text
eltakarja
```

az alatta lévő filesystem view-t.

---

# 41. Mount overlay mental model

```text
Bind mount
────────────
Image /app
```

A felső mount view látható.

Az alatta lévő image tartalom nem.

---

# 42. `node_modules` probléma developmentnél

Tegyük fel, hogy image build során:

```dockerfile
RUN npm install
```

így létrejön:

```text
/app/node_modules
```

Majd runtime:

```text
host project
↓ bind mount
/app
```

A bind mount elfedheti:

```text
/app/node_modules
```

image tartalmát is.

Ez problémát okozhat.

---

# 43. Development megoldás `node_modules`-ra

Gyakori logika:

```text
/app
→ bind mount source
```

de:

```text
/app/node_modules
→ külön volume
```

Ez lehetővé teszi, hogy:

```text
source
→ hostról jöjjön

dependencies
→ container/volume környezetben maradjanak
```

Ezt Docker Compose-nál részletesen meg fogjuk nézni.

---

# 44. Writable layer vs mount

Ha nincs mount:

```text
container writes
↓
writable layer
```

Ha ugyanarra a pathra mount van:

```text
container writes
↓
mount target
↓
bind mount / volume
```

Tehát a mount határozza meg, hova kerül az adat.

---

# 45. Egy container több storage típust is használhat

Például:

```text
Container
│
├── /app
│    ↑
│    bind mount
│
├── /app/uploads
│    ↑
│    volume
│
└── /tmp
     ↓
     writable layer vagy tmpfs
```

Ez teljesen normális.

---

# 46. tmpfs

A `tmpfs` egy memóriaalapú mount.

Mental model:

```text
RAM
↓
tmpfs
↓
container
```

Az adat nem normál persistent disk storage-on él.

---

# 47. tmpfs tulajdonságai

```text
memory-based
temporary
non-persistent
```

Container/runtime megszűnése után az adat nem marad persistent módon meg.

---

# 48. tmpfs használata

Példa:

```bash
docker run \
  --mount type=tmpfs,target=/app/tmp \
  myimage
```

Mental model:

```text
RAM
↓
/app/tmp
```

---

# 49. Mikor jó tmpfs?

Például:

```text
temporary files
short-lived runtime data
sensitive temporary data
```

esetén.

Nem jó:

```text
database persistent data
important uploads
business state
```

esetén.

---

# 50. Storage összehasonlítás

| Storage típus | Hol él? | Container törlés után? | Tipikus használat |
|---|---|---:|---|
| Writable layer | container storage | elveszik | temporary runtime data |
| Bind mount | host filesystem | megmarad | source, config |
| Volume | Docker-managed storage | megmaradhat | DB, uploads, persistent state |
| tmpfs | RAM | elveszik | temporary/sensitive runtime data |

---

# 51. Persistence szintek

Fontos:

```text
Writable layer
→ container lifecycle
```

```text
Volume
→ container lifecycle-től függetlenebb
```

```text
Host storage
→ host lifecycle-hoz kötött
```

```text
External backup/storage
→ hosttól is független lehet
```

---

# 52. Volume nem egyenlő backup

Nagyon fontos:

```text
volume
≠
backup
```

A volume túlélheti:

```text
container delete
```

de ha maga a host megsérül vagy eltűnik:

```text
host failure
↓
local volume is elveszhet
```

---

# 53. Példa DigitalOcean droplettel

Tegyük fel:

```text
DigitalOcean Droplet
↓
Docker Engine
↓
postgres-data volume
```

Ha csak a PostgreSQL containert töröljük:

```text
volume megmaradhat
```

De ha maga a droplet végleg elveszik:

```text
local volume is elveszhet
```

Ezért:

```text
persistence
≠
disaster recovery
```

---

# 54. Backup szükségessége

Persistent data esetén kellhet:

```text
backup
```

Példa:

```text
Docker Volume
↓
Backup process
↓
External storage
```

Majd restore:

```text
External backup
↓
Restore
↓
New volume
↓
New container
```

---

# 55. Image backup nem data backup

Ha PostgreSQL image:

```text
postgres:17
```

megvan, abból nem következik, hogy az adatbázis adatai is megvannak.

```text
IMAGE
→ PostgreSQL software
```

```text
VOLUME
→ PostgreSQL data
```

Tehát:

```text
image backup
≠
database backup
```

---

# 56. Egy rendszer több részből áll

Tipikus alkalmazási rendszer:

```text
Application artifact
+
configuration
+
persistent data
+
secrets
```

Például:

```text
Next.js image
+
environment config
+
PostgreSQL volume
+
credentials
```

Mindegyiknek külön lifecycle-ja lehet.

---

# 57. Database backup különleges eset

Adatbázisnál nem mindig jó egyszerűen raw fájlokat másolni működés közben.

Mert közben:

```text
transactions
writes
buffers
database files
```

változhatnak.

Ezért adatbázis esetén gyakran:

```text
database-aware backup
```

kell.

PostgreSQL-nél például később:

```text
pg_dump
```

eszközzel is találkozunk.

---

# 58. Filesystem backup vs database backup

```text
filesystem copy
≠
automatikusan konzisztens database backup
```

Ezért:

```text
database data
→ application-aware backup stratégia
```

szükséges lehet.

---

# 59. Bind mount backup

Bind mount esetén az adat a host filesystemben található.

Például:

```text
/srv/myapp/uploads
```

Ezért normál host backup rendszerrel is menthető.

```text
Host filesystem
↓
backup
```

---

# 60. Volume backup

Volume esetén az adat Docker-managed storage-ban van.

Mental model:

```text
Docker Volume
↓
backup process
↓
backup artifact
```

A konkrét backup módszert később külön backup/DR fejezetben nézzük.

---

# 61. Development storage példa

Next.js development:

```text
HOST
│
└── source code
       │
       │ bind mount
       ↓
Next.js Container
       │
       └── npm run dev
```

Lehetséges külön:

```text
node_modules
↓
volume
```

---

# 62. Production Next.js példa

Productionben a source code jellemzően már image-ben van.

```text
SOURCE
↓
docker build
↓
production image
↓
container
```

Ezért nincs szükség development bind mountra.

---

# 63. Production upload példa

Ha az alkalmazás fájlokat tölt fel:

```text
Next.js Container
↓
/app/uploads
```

ha ezek fontosak:

```text
/app/uploads
↓
volume
```

lehet megfelelő.

De cloud környezetben gyakran inkább:

```text
object storage
```

használatos.

---

# 64. Database production példa

```text
PostgreSQL Container
↓
/var/lib/postgresql/data
↓
postgres-data volume
```

Container cserélhető:

```text
Postgres A
↓
delete
```

majd:

```text
Postgres B
↓
same volume
```

az adat megmarad.

---

# 65. Stateless application gondolkodás

Ideális esetben az alkalmazás container:

```text
disposable
replaceable
stateless
```

Az állapot:

```text
database
volume
object storage
external service
```

helyen él.

Ez megkönnyíti:

```text
deployment
scaling
replacement
recovery
```

folyamatokat.

---

# 66. Mi az a stateless container?

Stateless alkalmazásnál a container önmagában nem tart fontos, pótolhatatlan állapotot.

Például:

```text
API Container A
↓ delete

API Container B
↓ start
```

A rendszer tovább működik, mert a fontos adat:

```text
database
external storage
```

helyen van.

---

# 67. Miért fontos ez DevOpsban?

Mert containereket gyakran:

```text
cserélünk
újraépítünk
újraindítunk
scale-elünk
áthelyezünk
```

Ezért az alkalmazás és az adat lifecycle-ját szét kell választani.

---

# 68. Storage döntési fa

Első kérdés:

```text
Kell az adat container törlés után?
```

Ha nem:

```text
writable layer
vagy
tmpfs
```

Ha igen:

```text
bind mount
vagy
volume
```

---

# 69. Következő kérdés

Ha persistent adat kell:

```text
Akarom hostról közvetlenül kezelni/szerkeszteni?
```

Ha igen:

```text
bind mount
```

Ha nem:

```text
volume
```

---

# 70. Harmadik kérdés

Ha az adat fontos:

```text
Kell host meghibásodása után is?
```

Ha igen:

```text
backup
external storage
managed storage
```

is szükséges.

---

# 71. Teljes döntési mental model

```text
                Adat keletkezik
                      │
                      ↓
          Kell container delete után?
                 /             \
               NEM             IGEN
                │                │
        writable/tmpfs      Hostról kell?
                           /          \
                         IGEN          NEM
                          │             │
                    bind mount       volume
                          \             /
                           \           /
                            ↓         ↓
                       Fontos adat?
                            │
                           IGEN
                            ↓
                     backup / external
                         storage
```

---

# 72. Storage és security

Mount használatnál fontos:

```text
Milyen host fájlokat lát a container?
Írhatja is őket?
Szüksége van valóban rájuk?
```

Ezért ahol lehet:

```text
read-only
```

mount előnyös lehet.

---

# 73. Veszélyes bind mount gondolat

Ha túl széles host pathot adunk a containernek:

```text
/
```

vagy érzékeny rendszerkönyvtárat:

```text
/etc
```

akkor a container túl sok host adatot érhet el.

Ez security szempontból veszélyes lehet.

Ezért:

```text
least privilege
```

elv storage-nál is fontos.

---

# 74. Storage és least privilege

Csak azt mountoljuk:

```text
amire az alkalmazásnak valóban szüksége van
```

És ha elég olvasni:

```text
readonly
```

módot használunk.

---

# 75. Gyakori félreértések

## `docker stop` törli a writable layer adatát

Nem.

```text
stop
→ container megmarad
→ writable layer megmarad
```

---

## `docker rm` után writable layer megmarad

Nem.

```text
container delete
→ writable layer delete
```

---

## Volume automatikusan backup

Nem.

```text
volume
→ persistent container lifecycle szempontból
```

de:

```text
backup
→ külön folyamat
```

---

## Volume mindig túléli a host elvesztését

Nem.

Local volume hosthoz kötődhet.

---

## Bind mount adata containerben van

Nem.

```text
adat
→ host filesystem
```

A container csak mounton keresztül látja.

---

## `EXPOSE` és volume hasonló

Nem.

```text
EXPOSE
→ networking metadata
```

```text
volume/mount
→ storage
```

---

# 76. Fontos storage parancsok

Volume létrehozás:

```bash
docker volume create mydata
```

Volume lista:

```bash
docker volume ls
```

Volume inspect:

```bash
docker volume inspect mydata
```

Volume törlés:

```bash
docker volume rm mydata
```

Nem használt volume takarítás:

```bash
docker volume prune
```

---

# 77. Bind mount parancs

```bash
docker run \
  --mount type=bind,source="$PWD",target=/app \
  myimage
```

---

# 78. Named volume parancs

```bash
docker run \
  --mount type=volume,source=mydata,target=/data \
  myimage
```

---

# 79. tmpfs parancs

```bash
docker run \
  --mount type=tmpfs,target=/tmp \
  myimage
```

---

# 80. Read-only mount

```bash
docker run \
  --mount type=bind,source="$PWD/config",target=/config,readonly \
  myimage
```

---

# 81. Storage cheatsheet

```text
Writable layer
→ container saját írható rétege
→ container törléssel eltűnik
```

```text
Bind mount
→ host path
→ host kezeli
→ developmentre kiváló
```

```text
Volume
→ Docker-managed
→ persistent data
→ DB-hez gyakori
```

```text
tmpfs
→ RAM
→ temporary
→ nem persistent
```

---

# 82. Development mental model

```text
Host source
↓ bind mount
Container /app
↓
dev server
```

Ha kell:

```text
node_modules
↓
volume
```

---

# 83. Production mental model

```text
Application code
↓
image
↓
container
```

Persistent state:

```text
volume
database
object storage
external service
```

---

# 84. Backup mental model

```text
Container
↓
Volume
↓
Backup
↓
External storage
```

A container törölhető.

A volume persistent lehet.

A backup pedig a host elvesztése ellen is védhet.

---

# 85. Teljes storage mental model

```text
                        CONTAINER
                            │
        ┌───────────────────┼────────────────────┐
        │                   │                    │
        ↓                   ↓                    ↓
 Writable layer        Bind mount             Volume
        │                   │                    │
 containerhez          host filesystem       Docker-managed
   kötött                  path                storage
        │                   │                    │
      temp              source/config        persistent data
```

Kiegészítés:

```text
tmpfs
↓
RAM
↓
temporary data
```

---

# 86. Kapcsolat az image layer-ekkel

Image:

```text
read-only layers
```

Container:

```text
writable layer
```

Mount:

```text
külső storage
```

Ezért:

```text
IMAGE
↓
CONTAINER
↓
MOUNTS
```

három külön storage szintet kell megkülönböztetni.

---

# 87. Kapcsolat a lifecycle-lel

```text
IMAGE
↓
docker run
↓
CONTAINER
↓
writable layer
```

Ha:

```text
docker stop
```

writable layer megmarad.

Ha:

```text
docker rm
```

writable layer eltűnik.

Ha mountolt volume van:

```text
volume
→ megmaradhat
```

---

# 88. Kapcsolat a Docker Compose-zal

Később Compose-ban például:

```text
Application container
↓
bind mount
```

és:

```text
Database container
↓
named volume
```

egyszerűen deklarálható lesz.

Például:

```text
services
├── app
└── database

volumes
└── postgres-data
```

A részletes syntax a Compose fejezetben jön.

---

# 89. Laborban kipróbálandó storage feladatok

A Docker labor során mindegyik storage típust külön ki fogjuk próbálni.

## 1. Writable layer

```text
container indítás
↓
fájl létrehozás
↓
stop/start
↓
fájl még megvan
↓
container delete
↓
fájl eltűnik
```

---

## 2. Bind mount

```text
host directory
↓
bind mount
↓
container
```

Teszt:

```text
hoston fájl módosítás
↓
container látja
```

és fordítva.

---

## 3. Read-only bind mount

```text
host config
↓
readonly mount
↓
container
```

Teszt:

```text
olvasás
→ működik

írás
→ tiltott
```

---

## 4. Named volume

```text
volume create
↓
Container A
↓
adat létrehozás
↓
Container A delete
↓
Container B
↓
same volume
↓
adat még megvan
```

---

## 5. Volume inspect

Megnézzük:

```bash
docker volume inspect
```

kimenetét.

---

## 6. Anonymous volume

Készítünk anonymous volume-ot is, hogy lássuk a különbséget.

---

## 7. tmpfs

```text
tmpfs mount
↓
adat létrehozás
↓
container megszűnik
↓
adat eltűnik
```

---

## 8. Mount overlay

Olyan könyvtárra mountolunk, amelyben az image-ben már van adat.

Megfigyeljük:

```text
mount
↓
image eredeti tartalma eltakarva
```

---

## 9. Development bind mount

Next.js project:

```text
host source
↓
bind mount
↓
container
↓
npm run dev
```

Megfigyeljük:

```text
source edit
↓
container azonnal látja
```

---

## 10. Database volume

PostgreSQL container:

```text
Container A
↓
volume
↓
adat
```

Container csere után ellenőrizzük:

```text
adat megmaradt-e?
```

---

# 90. Végső összefoglaló

A storage típus kiválasztását az adat lifecycle-ja határozza meg.

A legfontosabb kérdés:

> Ha ezt a containert most törlöm, ennek az adatnak is el szabad tűnnie?

Ha igen:

```text
writable layer
vagy
tmpfs
```

Ha nem:

```text
bind mount
vagy
volume
```

Ha az adat host meghibásodása után is kell:

```text
backup
external storage
managed persistent storage
```

is szükséges.

A legfontosabb Docker storage mental model:

```text
CONTAINER
│
├── Writable layer
│    → temporary/container-local
│
├── Bind mount
│    → host-controlled
│
├── Volume
│    → Docker-managed persistent
│
└── tmpfs
     → memory-only temporary
```

És a legfontosabb DevOps elv:

```text
Application container
→ könnyen cserélhető

Persistent data
→ container lifecycle-től független

Backup
→ host lifecycle-től is független
```