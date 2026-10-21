# Docker – Container Lifecycle és CLI

## 1. Docker elindítása Windows és Linux alatt

Mielőtt containereket tudnánk kezelni, magának a Docker környezetnek is futnia kell.

A pontos indítás attól függ, milyen operációs rendszeren és milyen Docker telepítést használunk.

---

# 2. Docker indítása Windows alatt

Windows 11-en, Docker Desktop használata esetén a Docker backendjét a Docker Desktop biztosítja.

A Docker Desktop parancssorból elindítható:

```powershell
docker desktop start
```

Állapot lekérdezése:

```powershell
docker desktop status
```

Leállítás:

```powershell
docker desktop stop
```

Újraindítás:

```powershell
docker desktop restart
```

A Docker Desktop CLI tehát lehetővé teszi, hogy a Desktop backendjét grafikus kattintás nélkül is kezeljük.

A saját környezetünkben:

```text
Windows 11
↓
Docker Desktop
↓
WSL2 / Linux backend
↓
Docker Engine
↓
Linux containers
```

A Docker Desktop elindítása után a Docker CLI használható:

```text
Windows Terminalból
és
WSL Ubuntu alól
```

is.

---

# 3. Fontos: Windows alatt a Docker Desktop nem csak GUI

A Docker Desktopot nem úgy kell elképzelni, mint egy egyszerű grafikus kezelőfelületet a Docker Engine fölött.

Windows alatt Linux containerek futtatásához Docker Desktop több háttérkomponenst is kezel.

Például:

```text
Docker Desktop
↓
WSL2 Linux backend
↓
Linux kernel
↓
Docker Engine
↓
Containers
```

Ezért a mi jelenlegi Windows környezetünkben nem egy külön:

```text
dockerd.exe
```

processzt indítunk kézzel Linux containerekhez.

A Docker Desktop kezeli a szükséges backend környezetet.

---

# 4. Docker indítása Linux alatt

Natív Linux Docker Engine telepítés esetén más a működés.

Például Ubuntu vagy Debian alatt a Docker daemon tipikusan systemd service-ként fut.

Állapot ellenőrzése:

```bash
sudo systemctl status docker
```

Indítás:

```bash
sudo systemctl start docker
```

Leállítás:

```bash
sudo systemctl stop docker
```

Újraindítás:

```bash
sudo systemctl restart docker
```

A Docker automatikus indításának engedélyezése:

```bash
sudo systemctl enable docker
```

Docker és containerd automatikus indulásának engedélyezése:

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

A Docker dokumentáció szerint Ubuntu és Debian rendszereken a Docker service jellemzően automatikusan elindul telepítés után, de manuálisan is indítható. 

---

# 5. A Docker daemon kézi indítása Linuxon

A Docker daemon neve:

```text
dockerd
```

Manuálisan is elindítható:

```bash
sudo dockerd
```

Ilyenkor a daemon:

```text
foregroundban fut
```

és a logokat közvetlenül a terminálra írja.

Leállítás:

```text
Ctrl+C
```

Ez tesztelésre vagy hibakeresésre használható, de normál szerveres működésnél általában:

```text
systemd
```

kezeli a Docker daemon lifecycle-ját.

Mental model:

```text
Natív Linux:

systemd
↓
dockerd
↓
Docker Engine
↓
containers
```

---

# 6. Windows vs Linux Docker startup

## Windows + Docker Desktop

```text
docker desktop start
↓
Docker Desktop backend
↓
Docker Engine
↓
containers
```

## Natív Linux Docker Engine

```text
sudo systemctl start docker
↓
dockerd
↓
Docker Engine
↓
containers
```

Vagy kézzel:

```text
sudo dockerd
```

---

# 7. A mi jelenlegi környezetünk

A saját gépünkön:

```text
Windows 11
│
├── Docker Desktop
│      ↓
│   Docker backend
│
└── WSL2 Ubuntu
       ↓
    docker CLI
```

Ezért a napi indulási workflow:

```text
Windows elindul
↓
docker desktop start
↓
Docker backend elindul
↓
docker ps
↓
Docker használható
```

WSL-ben ezután ugyanazt a Docker környezetet érjük el:

```bash
docker ps
```

Nem telepítünk külön második Docker Engine-t a WSL Ubuntu környezetbe.

---

# 8. Docker Engine ellenőrzése

A legegyszerűbb ellenőrzés:

```bash
docker version
```

Ha csak:

```text
Client
```

információ jelenik meg, de:

```text
Server
```

nem érhető el, akkor a Docker Engine valószínűleg nem fut vagy nem elérhető.

Ha mindkettő megjelenik:

```text
Client
Server
```

akkor a CLI sikeresen kommunikál a Docker Engine-nel.

---

# 9. Gyors ellenőrzés `docker ps` segítségével

```bash
docker ps
```

Ha működik, akkor:

```text
Docker CLI
↓
Docker Engine
↓
kommunikáció működik
```

Ha olyan hibát kapunk, hogy:

```text
Cannot connect to the Docker daemon
```

akkor elsőként nem a containert vizsgáljuk.

Hanem:

```text
Docker Engine fut?
```

---

# 10. Mi az a container lifecycle?

A container lifecycle azt jelenti:

> milyen állapotokon megy keresztül egy container a létrehozásától a törléséig.

Egyszerű lifecycle:

```text
IMAGE
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
REMOVE
↓
DELETED
```

De a lifecycle nem feltétlenül egyirányú.

Például:

```text
EXITED
↓
START
↓
RUNNING
```

---

# 11. Fontos container állapotok

A Docker több állapotot is használhat:

```text
created
running
paused
restarting
exited
dead
```

A három legfontosabb:

```text
CREATED
RUNNING
EXITED
```

---

# 12. CREATED

A container objektum már létrejött.

Van:

```text
Container ID
container name
configuration
metadata
writable layer
network configuration
mount configuration
```

De:

```text
main process még nem fut
```

Mental model:

```text
Container exists
↓
Process does not run
↓
CREATED
```

---

# 13. RUNNING

A container main process-e fut.

```text
Container
↓
PID 1 fut
↓
RUNNING
```

Például:

```text
Container
├── PID 1: nginx master
├── nginx worker
└── nginx worker
```

---

# 14. EXITED

Ha a main process leáll:

```text
PID 1 exits
↓
container process leáll
↓
EXITED
```

Fontos:

```text
EXITED
≠
DELETED
```

A container objektum továbbra is létezik.

Megmarad például:

```text
Container ID
name
writable layer
configuration
metadata
```

Ezért újraindítható.

---

# 15. Container object vs process

A container nem egyenlő a benne futó process-szel.

```text
Container
≠
running process
```

A container egy Docker által kezelt objektum.

A process csak a runtime állapot része.

Ezért létezhet container:

```text
CREATED
```

vagy:

```text
EXITED
```

állapotban is.

---

# 16. `docker create`

Container létrehozása:

```bash
docker create --name web nginx
```

Flow:

```text
nginx image
↓
container configuration
↓
container object
↓
CREATED
```

A container létezik.

De:

```text
main process nem fut
```

---

# 17. Mit hoz létre a `docker create`?

Többek között:

```text
Container ID
container name
configuration
writable filesystem layer
network configuration
mount configuration
metadata
```

jön létre.

De az application process:

```text
még nincs elindítva
```

---

# 18. `docker start`

Meglévő container indítása:

```bash
docker start web
```

Flow:

```text
CREATED
↓
START
↓
main process indul
↓
RUNNING
```

EXITED container esetén:

```text
EXITED
↓
START
↓
main process újraindul
↓
RUNNING
```

---

# 19. `docker start` nem készít új containert

```bash
docker start web
```

ugyanazt a containert indítja.

Megmarad:

```text
Container ID
name
writable layer
configuration
```

Tehát:

```text
docker start
→ existing container
```

---

# 20. `docker run`

A leggyakoribb container indító parancs:

```bash
docker run nginx
```

Magas szinten:

```text
image megvan?
↓
ha nincs → pull
↓
docker create
↓
docker start
```

Ezért:

```text
docker run
≈
create + start
```

és szükség esetén:

```text
pull
```

is történhet.

---

# 21. `docker run` mindig új containert készít

Ha ezt futtatjuk kétszer:

```bash
docker run nginx
docker run nginx
```

akkor két külön container keletkezik.

```text
nginx image
├── Container A
└── Container B
```

Meglévő containert:

```bash
docker start
```

paranccsal indítunk újra.

---

# 22. `run` vs `start`

```text
docker run
→ új container
```

```text
docker start
→ meglévő container
```

Ez az egyik legfontosabb Docker CLI különbség.

---

# 23. Container elnevezése

```bash
docker run --name web nginx
```

A container neve:

```text
web
```

Ezután:

```bash
docker stop web
docker start web
docker logs web
docker inspect web
```

használható.

---

# 24. `docker stop`

Kontrollált leállítás:

```bash
docker stop web
```

A Docker megpróbál graceful shutdown-t végrehajtani.

Mental model:

```text
RUNNING
↓
termination signal
↓
alkalmazás leáll
↓
main process exit
↓
EXITED
```

---

# 25. Graceful shutdown

Graceful shutdown során az alkalmazásnak van lehetősége:

```text
aktív kapcsolatokat lezárni
folyamatban lévő munkát befejezni
buffereket kiírni
fájlokat lezárni
erőforrásokat felszabadítani
```

Ez különösen fontos:

```text
webserver
database
queue
worker
```

workloadoknál.

---

# 26. `docker stop` és signal

Linux container esetén gyakran:

```text
SIGTERM
```

érkezik először.

Ha a process nem áll le időben:

```text
SIGKILL
```

is következhet.

Mental model:

```text
SIGTERM
↓
várakozás
↓
nem áll le?
↓
SIGKILL
```

A pontos stop signal konfigurálható.

---

# 27. Mi marad meg stop után?

Megmarad:

```text
Container ID
name
writable layer
configuration
metadata
```

A main process:

```text
nem fut
```

Állapot:

```text
EXITED
```

Ezért:

```text
STOP
≠
DELETE
```

---

# 28. `docker restart`

```bash
docker restart web
```

Logikailag:

```text
RUNNING
↓
STOP
↓
START
↓
RUNNING
```

Ugyanaz a container marad.

---

# 29. Restart után mi marad meg?

Megmarad:

```text
Container ID
name
writable layer
container configuration
mountok
network configuration
```

Ezért:

```text
restart
≠
new container
```

---

# 30. `docker kill`

```bash
docker kill web
```

Alapértelmezésben tipikusan:

```text
SIGKILL
```

küldés történik.

Mental model:

```text
docker stop
→ állj le szabályosan
```

```text
docker kill
→ állj le azonnal
```

---

# 31. Mikor használunk `kill`-t?

Például:

```text
process lefagyott
stop nem működik
crash scenariót tesztelünk
```

Normál esetben:

```text
docker stop
```

előnyösebb.

---

# 32. `docker rm`

Container törlése:

```bash
docker rm web
```

Ekkor eltűnik:

```text
container object
metadata
writable layer
runtime state
```

Az image:

```text
megmarad
```

---

# 33. Container törlés vs image törlés

```bash
docker rm web
```

jelentése:

```text
container törlés
```

Ezzel szemben:

```bash
docker image rm nginx
```

jelentése:

```text
image törlés
```

---

# 34. Futó container törlése

Normál workflow:

```bash
docker stop web
docker rm web
```

Létezik:

```bash
docker rm -f web
```

is.

Ez kényszerített törlés.

Nem ezt használjuk alapértelmezett működésként.

---

# 35. `--rm`

Egyszer használatos container:

```bash
docker run --rm alpine echo hello
```

Flow:

```text
container create
↓
start
↓
echo hello
↓
main process exit
↓
container automatikusan törlődik
```

Ez különösen jó:

```text
one-off command
script
teszt
utility
```

jellegű használathoz.

---

# 36. Rövid életű container

```bash
docker run --rm alpine echo "Hello Docker"
```

A main process:

```text
echo
```

Lifecycle:

```text
CREATE
↓
START
↓
echo fut
↓
echo kilép
↓
EXITED
↓
--rm
↓
DELETED
```

---

# 37. A container nem feltétlenül service

Container lehet hosszú életű:

```text
webserver
database
API
```

de lehet rövid életű:

```text
script
batch job
migration
backup
one-off command
```

A container addig RUNNING:

```text
amíg a main process fut
```

---

# 38. `docker ps`

Futó containerek:

```bash
docker ps
```

Teljesebb forma:

```bash
docker container ls
```

---

# 39. `docker ps` output

Tipikus oszlopok:

```text
CONTAINER ID
IMAGE
COMMAND
CREATED
STATUS
PORTS
NAMES
```

Például:

```text
CONTAINER ID   IMAGE   STATUS       NAMES
a12bc34de56f   nginx   Up 2 mins    web
```

---

# 40. `docker ps -a`

Minden container:

```bash
docker ps -a
```

Megjelenhet:

```text
CREATED
RUNNING
EXITED
```

állapot is.

Mental model:

```text
docker ps
→ mi fut?
```

```text
docker ps -a
→ milyen containerek léteznek?
```

---

# 41. Exit code

Container leállásakor exit code tartozhat a processhez.

```text
Exited (0)
```

általában:

```text
sikeres befejezés
```

Non-zero:

```text
valamilyen hiba vagy speciális process eredmény
```

Például:

```text
Exited (1)
```

gyakran hibás befejezést jelent.

---

# 42. Első troubleshooting flow

Ha az alkalmazás nem működik:

```bash
docker ps
```

Ha nincs ott:

```bash
docker ps -a
```

Ha:

```text
Exited (1)
```

akkor:

```bash
docker logs <container>
```

---

# 43. `docker logs`

```bash
docker logs web
```

A Docker logging rendszerén keresztül a container standard outputját és standard errorját vizsgálhatjuk.

Fontos:

```text
STDOUT
STDERR
```

---

# 44. stdout és stderr

```text
STDOUT
→ standard output
```

```text
STDERR
→ standard error
```

Gyakori containeres logging minta:

```text
Application
↓
stdout / stderr
↓
Docker logging
↓
docker logs
```

---

# 45. Élő log követés

```bash
docker logs -f web
```

A:

```text
-f
```

jelentése:

```text
follow
```

Hasonló:

```bash
tail -f
```

működéshez.

---

# 46. Utolsó log sorok

```bash
docker logs --tail 50 web
```

Csak az utolsó 50 sort mutatja.

---

# 47. Timestamp

```bash
docker logs -t web
```

Timestampet is mutathat.

Hasznos kérdés:

```text
mikor történt a hiba?
```

---

# 48. `docker exec`

Parancs futtatása egy már futó containerben:

```bash
docker exec web <command>
```

Például:

```bash
docker exec web ls /etc/nginx
```

Flow:

```text
meglévő container
↓
új process
↓
ls
↓
process kilép
```

---

# 49. `docker exec` nem hoz létre új containert

```text
docker run
→ új container
```

```text
docker exec
→ új process meglévő containerben
```

---

# 50. Interaktív shell

```bash
docker exec -it web sh
```

Ha van Bash:

```bash
docker exec -it web bash
```

Ekkor:

```text
Container
├── PID 1: application
└── új shell process
```

---

# 51. `-i`

```text
-i
→ interactive
→ STDIN nyitva marad
```

---

# 52. `-t`

```text
-t
→ pseudo-TTY
```

Terminálszerű környezetet biztosít.

Ezért együtt gyakori:

```text
-it
```

---

# 53. `exec` process modell

Eredetileg:

```text
Container
└── PID 1: nginx
```

Majd:

```bash
docker exec -it web sh
```

után:

```text
Container
├── PID 1: nginx
└── PID 25: sh
```

Ha:

```bash
exit
```

akkor a shell process leáll.

Az nginx tovább fut.

A container RUNNING marad.

---

# 54. Minimal image

Egy minimalista image-ben lehet, hogy nincs:

```text
bash
curl
ping
ps
vim
nano
```

Ez normális lehet.

Kevesebb package:

```text
kisebb image
+
kisebb attack surface
```

előnyt jelenthet.

---

# 55. `docker run -it`

```bash
docker run -it ubuntu bash
```

Itt:

```text
új container
↓
PID 1 = bash
↓
interaktív terminál
```

Ha:

```bash
exit
```

akkor Bash kilép.

Mivel Bash a main process:

```text
container EXITED
```

---

# 56. `run -it` vs `exec -it`

```bash
docker run -it ubuntu bash
```

jelentése:

```text
új container
+
bash process
```

```bash
docker exec -it web sh
```

jelentése:

```text
meglévő futó container
+
új shell process
```

Röviden:

```text
RUN
→ new container
```

```text
EXEC
→ new process
```

---

# 57. Attached mód

```bash
docker run nginx
```

esetén a CLI attached maradhat a container standard streamjeihez.

```text
CLI
↔
container process
```

---

# 58. Detached mód

```bash
docker run -d nginx
```

A:

```text
-d
```

jelentése:

```text
detached
```

Flow:

```text
container indul
↓
CLI leválik
↓
prompt visszatér
↓
container tovább fut
```

---

# 59. Detached nem daemon

```text
detached
≠
daemon
```

A:

```text
-d
```

csak azt jelenti:

```text
CLI nem marad attached
```

A container main process ugyanúgy normál Linux process.

---

# 60. Detached container logja

```bash
docker run -d --name web nginx
```

után:

```bash
docker logs web
```

vagy:

```bash
docker logs -f web
```

használható.

---

# 61. `docker attach`

```bash
docker attach web
```

a meglévő container main process standard streamjeihez kapcsolódik.

Ez más, mint:

```bash
docker exec
```

---

# 62. Attach vs exec

```text
docker attach
→ meglévő main process streamjei
```

```text
docker exec
→ új process a containerben
```

Debugoláshoz gyakran:

```bash
docker exec -it ...
```

kényelmesebb.

---

# 63. `docker inspect`

```bash
docker inspect web
```

részletes Docker metadata és konfiguráció megtekintésére szolgál.

Tipikusan JSON outputot ad.

---

# 64. Fontos inspect részek

```text
State
Config
HostConfig
NetworkSettings
Mounts
```

---

# 65. State

Tartalmazhat:

```text
Status
Running
Pid
ExitCode
StartedAt
FinishedAt
```

információkat.

---

# 66. Config

Tartalmazhat:

```text
Env
Cmd
Entrypoint
WorkingDir
User
Hostname
```

adatokat.

---

# 67. NetworkSettings

Itt találhatunk:

```text
IP address
gateway
network
ports
```

információkat.

---

# 68. Mounts

Megmutathatja:

```text
volume
bind mount
source
destination
```

kapcsolatokat.

---

# 69. Inspect format

Például:

```bash
docker inspect --format '{{.State.Status}}' web
```

Output:

```text
running
```

A teljes template syntaxet nem kell megjegyezni.

A fontos mental model:

```text
inspect
→ Docker object részletes metadata
```

---

# 70. `docker top`

```bash
docker top web
```

Megmutatja a containerhez tartozó processzeket.

Például:

```text
nginx master
nginx worker
nginx worker
```

Ez bizonyítja:

```text
container
≠
egy process
```

---

# 71. `docker top` vs `docker exec ps`

```bash
docker top web
```

Docker/host oldalról mutat processinformációt.

```bash
docker exec web ps aux
```

új `ps` processzt futtat a containerben.

A minimalista image-ben lehet, hogy nincs `ps`.

Ezért:

```bash
docker top
```

gyakran megbízhatóbb.

---

# 72. `docker stats`

```bash
docker stats
```

real-time resource monitoring.

Tipikusan:

```text
CPU %
MEM USAGE / LIMIT
MEM %
NET I/O
BLOCK I/O
PIDS
```

---

# 73. `docker stats` és cgroups

A resource használat kapcsolódik:

```text
cgroups
```

fogalomhoz.

Mental model:

```text
Container processes
↓
cgroup accounting / limits
↓
docker stats
```

---

# 74. Egy konkrét container

```bash
docker stats web
```

csak a `web` containert figyeli.

Kilépés:

```text
Ctrl+C
```

---

# 75. `docker pause`

```bash
docker pause web
```

felfüggeszti a container processzeinek végrehajtását.

Állapot:

```text
PAUSED
```

A processzek:

```text
nem exitelnek
```

---

# 76. `docker unpause`

```bash
docker unpause web
```

Flow:

```text
RUNNING
↓
PAUSE
↓
PAUSED
↓
UNPAUSE
↓
RUNNING
```

---

# 77. PAUSED vs EXITED

```text
PAUSED
→ process létezik
→ végrehajtás felfüggesztve
```

```text
EXITED
→ main process nem fut
```

---

# 78. Restart policy

Containerhez restart policy rendelhető.

Például:

```bash
docker run --restart unless-stopped ...
```

A restart policy azt szabályozza:

```text
mikor próbálja Docker automatikusan újraindítani a containert
```

---

# 79. Gyakori restart policy-k

```text
no
on-failure
always
unless-stopped
```

Alapértelmezés:

```text
no
```

---

# 80. `on-failure`

```text
on-failure
```

esetén a Docker újraindíthatja a containert, ha:

```text
non-zero exit code
```

miatt áll le.

---

# 81. `always`

```text
always
```

esetén Docker általában megpróbálja újraindítani a containert, ha leáll.

Hosszú életű service-eknél használható.

---

# 82. `unless-stopped`

```text
unless-stopped
```

mental model:

```text
tartsd futva
↓
kivéve, ha explicit én állítottam le
```

Saját szerveren futó service-eknél gyakori policy.

---

# 83. Teljes lifecycle mental model

```text
IMAGE
↓
docker create
↓
CREATED
↓
docker start
↓
RUNNING
│
├── docker pause
│      ↓
│    PAUSED
│      ↓
│    docker unpause
│      ↓
│    RUNNING
│
├── docker restart
│      ↓
│    RUNNING
│
├── docker stop
│      ↓
│    EXITED
│
└── docker kill
       ↓
     EXITED
```

EXITED-ből:

```text
EXITED
├── docker start
│      ↓
│    RUNNING
│
└── docker rm
       ↓
     DELETED
```

---

# 84. Futó container vizsgálata

```text
RUNNING
│
├── docker ps
│     → fut?
│
├── docker logs
│     → mit logol?
│
├── docker exec
│     → futtass processzt benne
│
├── docker inspect
│     → milyen configgal fut?
│
├── docker top
│     → milyen processzek vannak?
│
└── docker stats
      → mennyi erőforrást használ?
```

---

# 85. Troubleshooting workflow

Ha egy Docker alkalmazás nem működik:

```text
1. Docker Engine fut?
```

Windows:

```powershell
docker desktop status
```

Linux:

```bash
sudo systemctl status docker
```

Utána:

```text
2. Container fut?
```

```bash
docker ps
```

Ha nem:

```text
3. Létezik?
```

```bash
docker ps -a
```

Ha EXITED:

```text
4. Miért állt le?
```

```bash
docker logs <container>
```

Ha RUNNING:

```text
5. Milyen konfigurációval fut?
```

```bash
docker inspect <container>
```

További vizsgálat:

```text
6. Processzek?
→ docker top
```

```text
7. Belső állapot?
→ docker exec
```

```text
8. CPU / memória?
→ docker stats
```

---

# 86. CLI parancsok kérdésekhez kötve

## Docker platform fut?

Windows:

```powershell
docker desktop status
```

Linux:

```bash
sudo systemctl status docker
```

---

## Docker indítása

Windows:

```powershell
docker desktop start
```

Linux:

```bash
sudo systemctl start docker
```

---

## Új container kell?

```bash
docker run
```

---

## Csak létrehozni akarom?

```bash
docker create
```

---

## Meglévő containert indítok?

```bash
docker start
```

---

## Kulturált leállítás?

```bash
docker stop
```

---

## Azonnali leállítás?

```bash
docker kill
```

---

## Újraindítás?

```bash
docker restart
```

---

## Törlés?

```bash
docker rm
```

---

## Mi fut?

```bash
docker ps
```

---

## Mi létezik összesen?

```bash
docker ps -a
```

---

## Mit logol?

```bash
docker logs
```

---

## Parancsot futtatnék benne?

```bash
docker exec
```

---

## Milyen configgal fut?

```bash
docker inspect
```

---

## Milyen processzek futnak?

```bash
docker top
```

---

## Mennyi erőforrást használ?

```bash
docker stats
```

---

# 87. Gyakori félreértések

## `docker run` meglévő containert indít

Nem.

```text
docker run
→ új container
```

Meglévőhöz:

```text
docker start
```

---

## `docker stop` törli a containert

Nem.

```text
STOP
→ EXITED
```

A container megmarad.

---

## `docker restart` új containert készít

Nem.

```text
restart
→ ugyanaz a container
```

---

## `docker exec` belép a main processbe

Nem.

```text
exec
→ új process a containerben
```

---

## `docker attach` és `docker exec` ugyanaz

Nem.

```text
attach
→ meglévő process stream
```

```text
exec
→ új process
```

---

## `docker kill` és `docker stop` ugyanaz

Nem.

```text
stop
→ graceful shutdown
```

```text
kill
→ alapból azonnali leállítás
```

---

## EXITED container törölve van

Nem.

```text
EXITED
→ container létezik
→ process nem fut
```

---

# 88. Alap parancsok összefoglalása

## Docker platform

Windows:

```powershell
docker desktop start
docker desktop stop
docker desktop restart
docker desktop status
```

Linux:

```bash
sudo systemctl start docker
sudo systemctl stop docker
sudo systemctl restart docker
sudo systemctl status docker
```

Manuális daemon Linuxon:

```bash
sudo dockerd
```

---

## Container létrehozás

```bash
docker create
docker run
```

---

## Lifecycle

```bash
docker start
docker stop
docker restart
docker kill
docker pause
docker unpause
docker rm
```

---

## Listázás

```bash
docker ps
docker ps -a
```

---

## Vizsgálat

```bash
docker logs
docker exec
docker inspect
docker top
docker stats
```

---

# 89. Rövid lifecycle cheatsheet

```text
CREATE
→ container létrejön
```

```text
START
→ main process indul
```

```text
RUN
→ új container + start
```

```text
STOP
→ graceful leállítás
```

```text
KILL
→ azonnali leállítás
```

```text
RESTART
→ ugyanaz a container újraindul
```

```text
RM
→ container törlődik
```

```text
PS
→ futó containerek
```

```text
PS -A
→ minden container
```

```text
LOGS
→ alkalmazás output
```

```text
EXEC
→ új process meglévő containerben
```

```text
INSPECT
→ metadata / config
```

```text
TOP
→ processzek
```

```text
STATS
→ CPU / RAM / I/O
```

---

# 90. Windows és Linux startup mental model

## Windows 11 + Docker Desktop

```text
Windows
↓
docker desktop start
↓
Docker Desktop backend
↓
Docker Engine
↓
docker CLI
↓
containers
```

## Natív Linux

```text
Linux
↓
systemd
↓
dockerd
↓
Docker Engine
↓
docker CLI
↓
containers
```

---

# 91. Kapcsolat a korábbi fejezetekkel

Az első fejezetben:

```text
Container
=
izolált process environment
```

A másodikban:

```text
Docker CLI
↓
Docker Engine
↓
runtime
↓
Linux kernel
```

A harmadikban:

```text
Image
↓
Container
```

Ebben a fejezetben:

```text
Container
↓
CREATED
↓
RUNNING
↓
EXITED
↓
DELETED
```

és a hozzá tartozó CLI parancsokat tanultuk.

---

# 92. Teljes mental model

```text
Docker platform
↓
Docker Engine elérhető

          IMAGE
            │
            │ docker create
            ↓
         CREATED
            │
            │ docker start
            ↓
         RUNNING
            │
            ├── docker logs
            ├── docker exec
            ├── docker inspect
            ├── docker top
            ├── docker stats
            │
            ├── docker pause
            │       ↓
            │     PAUSED
            │       ↓
            │    unpause
            │       ↓
            │    RUNNING
            │
            ├── docker stop
            │
            └── docker kill
                    ↓
                 EXITED
                  /   \
                 /     \
            start       rm
              ↓          ↓
           RUNNING     DELETED
```

---

# 93. Végső összefoglaló

A container lifecycle alapja:

```text
CREATE
↓
START
↓
RUNNING
↓
STOP / EXIT
↓
EXITED
↓
START vagy REMOVE
```

A legfontosabb lifecycle parancsok:

```text
create
run
start
stop
restart
kill
rm
```

A legfontosabb vizsgáló parancsok:

```text
ps
logs
exec
inspect
top
stats
```

A legfontosabb különbségek:

```text
run
→ új container
```

```text
start
→ meglévő container
```

```text
stop
→ process leáll, container megmarad
```

```text
rm
→ container törlődik
```

```text
exec
→ új process fut meglévő containerben
```

```text
attach
→ meglévő main process streamjeihez kapcsolódik
```

Windows és Linux alatt pedig magának a Docker platformnak az indítása eltér:

```text
Windows + Docker Desktop
→ docker desktop start
```

```text
Linux + natív Docker Engine
→ sudo systemctl start docker
```

A legfontosabb troubleshooting gondolkodás:

```text
Docker Engine fut?
↓
Container létezik?
↓
Container fut?
↓
Main process fut?
↓
Mit mond a log?
↓
Milyen konfigurációval fut?
↓
Milyen processzek vannak?
↓
Mennyi erőforrást használ?
```

Mindig azt keressük:

> Mi az utolsó pont, amelyről biztosan tudjuk, hogy működik, és mi a következő elem a láncban?