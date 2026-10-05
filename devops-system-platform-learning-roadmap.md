# DevOps / System / Platform Engineer tanulási terv

## Cél

A cél nem az, hogy különálló technológiákat bemagoljunk, hanem hogy fokozatosan felépítsük azt a tudást, amellyel egy teljes alkalmazási rendszert:

- megértesz,
- felépítesz,
- üzemeltetsz,
- hibakeresel,
- biztonságosabbá teszel,
- monitorozol,
- automatizálsz,
- később skálázol.

A fő irány:

```text
System alapok
↓
Linux
↓
Networking
↓
Server / Web
↓
Containers
↓
Databases
↓
Security / Monitoring / Backup
↓
CI/CD
↓
Cloud
↓
Infrastructure as Code
↓
Kubernetes
↓
Reliability / SRE
↓
Platform Engineering
```

A full-stack Node.js / Next.js / React / SQL háttérre építünk, de a fókusz egyre inkább:

```text
DevOps
→ Cloud
→ Platform Engineering
```

---

# 0. Hol tartunk most?

## Computer Systems / Operating System Basics

Állapot:

```text
KÉSZ / ÁTISMÉTLÉS ALATT
```

Átvettük:

- hardware alapok,
- CPU,
- core / thread,
- clock,
- registers,
- cache,
- RAM,
- storage,
- firmware,
- driver,
- boot,
- kernel,
- user space / kernel space,
- program vs process,
- process,
- thread,
- scheduler,
- context switch,
- virtual memory,
- stack / heap,
- swap,
- system call.

### Mostani feladat

Nem kell tovább rohanni.

Olvasd át többször a jegyzetet, és próbáld saját szavaiddal elmondani:

```text
Mi történik attól a pillanattól,
hogy elindítok egy programot?
```

---

# 1. Git + GitHub + GitHub Actions

Állapot:

```text
ELMÉLET KÉSZ
LABOR KÖVETKEZIK
```

Átvettük:

## Git

- repository,
- working directory,
- staging area,
- commit,
- hash,
- HEAD,
- branch,
- merge,
- merge conflict,
- remote,
- origin,
- upstream,
- fetch,
- pull,
- push,
- reset,
- revert,
- stash,
- reflog,
- tags,
- alap Git config.

## GitHub

- remote repository,
- authentication,
- PAT,
- SSH,
- GitHub CLI,
- Issue,
- Pull Request,
- Code Review,
- Checks,
- branch protection,
- rulesets.

## GitHub Actions

- event,
- workflow,
- job,
- runner,
- step,
- action,
- `uses`,
- `run`,
- CI alapok,
- test / build automatizálás.

---

## Következő feladat: komplex Git/GitHub labor

A labor során:

```text
WSL Ubuntu
↓
Node.js project
↓
Git init
↓
commits
↓
branches
↓
GitHub repo
↓
Issue
↓
feature branch
↓
Pull Request
↓
merge conflict
↓
reset / revert / stash
↓
GitHub Actions
↓
hibás CI
↓
javítás
↓
zöld check
↓
merge
↓
tag / release
```

### Mikor tekintjük késznek?

Ha már nem csak a parancsokat ismered, hanem érted:

- mi local és mi remote,
- mi történik push/fetch/pull közben,
- mire való branch és PR,
- hogyan kerül változtatás mainbe,
- hogyan fut le automatikusan egy CI workflow.

A labor után készül:

```text
külön megoldókulcs
```

is.

---

# 2. Linux alapok

Ez lesz a következő nagy tananyag.

## Miért most?

A DevOps világ nagy része Linux szervereken fut.

Docker, cloud VM-ek, CI runner-ek, Nginx, Kubernetes node-ok és sok adatbázis szerver Linux-alapú.

A Computer Systems elmélet után most már meg tudjuk nézni:

```text
hogyan jelennek meg ezek a fogalmak
egy valódi Linux rendszerben?
```

---

## Fő témák

### Linux rendszer felépítése

- kernel,
- user space,
- shell,
- filesystem hierarchy,
- `/`,
- `/home`,
- `/etc`,
- `/var`,
- `/usr`,
- `/tmp`,
- `/proc`,
- `/dev`.

### Fájl- és könyvtárkezelés

- `pwd`
- `ls`
- `cd`
- `mkdir`
- `touch`
- `cp`
- `mv`
- `rm`
- `find`
- `grep`
- `cat`
- `less`
- `head`
- `tail`

### Jogosultságok

- user,
- group,
- owner,
- read,
- write,
- execute,
- `chmod`,
- `chown`,
- `sudo`.

### Processek

- `ps`,
- `top`,
- `htop`,
- PID,
- process state,
- foreground/background,
- `kill`,
- signals.

### Memória és rendszerterhelés

- `free`,
- RAM,
- swap,
- load average,
- CPU usage.

### Service-ek

- daemon,
- `systemd`,
- `systemctl`,
- service start/stop/restart/status,
- boot-time startup.

### Logok

- `/var/log`,
- `journalctl`,
- service logs,
- troubleshooting.

### Package management

- `apt`,
- repository,
- package,
- update,
- upgrade,
- install,
- remove.

### Shell

- Bash alapok,
- environment variables,
- PATH,
- pipes,
- redirection,
- alap shell scripting.

---

## Linux labor

Egy Ubuntu WSL / VM környezetben:

- fájlstruktúra felfedezése,
- user és permission gyakorlat,
- process indítás/leállítás,
- service kezelés,
- log keresés,
- egyszerű Bash script,
- egy Node.js alkalmazás Linux alatti futtatása.

### Késznek tekintjük, ha

önállóan tudsz:

```text
navigálni Linuxban
↓
fájlokat kezelni
↓
jogosultságot ellenőrizni
↓
processzt megkeresni
↓
service-t kezelni
↓
logot olvasni
↓
alap hibát diagnosztizálni
```

---

# 3. Networking alapok

## Miért Linux után?

Mert a hálózati fogalmakat már Linuxon is azonnal tudjuk vizsgálni.

Nem csak elmélet lesz:

```text
IP
↓
port
↓
socket
↓
process
```

kapcsolatát fogjuk látni.

---

## Fő témák

- LAN / WAN,
- IP address,
- IPv4,
- subnet,
- subnet mask / CIDR,
- gateway,
- MAC address,
- ARP,
- DNS,
- DHCP,
- TCP,
- UDP,
- port,
- socket,
- localhost,
- loopback,
- routing,
- NAT,
- firewall,
- HTTP / HTTPS alapok.

### Fontos Linux eszközök

- `ip`
- `ip addr`
- `ip route`
- `ping`
- `ss`
- `curl`
- `wget`
- `dig`
- `nslookup`
- `traceroute`
- `nc`
- `tcpdump`
- később Wireshark.

---

## Networking labor

Például:

```text
Node app :3000
↓
Linux socket
↓
localhost
↓
curl
```

Majd:

```text
másik process
↓
másik port
↓
kapcsolat
```

Később:

```text
Docker network
```

és:

```text
VM / cloud networking
```

is.

---

# 4. Server Administration

A Linux + networking után kezdünk valódi szerverben gondolkodni.

## Fő témák

- Ubuntu Server,
- SSH,
- users,
- groups,
- permissions,
- services,
- ports,
- processes,
- logs,
- firewall,
- package management,
- environment variables,
- basic hardening.

## Labor

DigitalOcean Ubuntu szerver:

```text
local machine
↓
SSH
↓
Ubuntu server
↓
Node.js application
```

Cél:

önállóan fel tudj tenni és működésre bírni egy egyszerű appot.

---

# 5. Nginx + HTTP + HTTPS + TLS

Itt kezd összeállni az első valódi webszerveres architektúra.

## Fő témák

- HTTP request / response,
- domain,
- DNS,
- port 80,
- port 443,
- reverse proxy,
- Nginx,
- TLS,
- certificate,
- Let's Encrypt,
- redirect HTTP → HTTPS.

Mentális modell:

```text
Browser
↓
DNS
↓
Server IP
↓
HTTPS :443
↓
Nginx
↓
Node app :3000
```

---

## Labor

Saját domain/subdomain:

```text
https://app.example.com
↓
Nginx
↓
Node.js app
```

---

# 6. Docker és Docker Compose

## Miért csak most?

Mert előbb érteni kell:

```text
process
Linux
network
port
filesystem
service
```

fogalmakat.

Különben a Docker csak varázslatnak tűnne.

---

## Fő témák

- image,
- container,
- Dockerfile,
- layer,
- volume,
- network,
- port mapping,
- environment variables,
- registry,
- Docker Compose.

## Labor

```text
frontend
+
backend
+
PostgreSQL
```

külön konténerekben.

Majd:

```text
Docker Compose
```

segítségével egyben indítjuk.

---

# 7. Databases – Operations szemlélet

A SQL-t nem nulláról tanuljuk újra.

A fókusz:

```text
adatbázis üzemeltetése
```

---

## Fő témák

- PostgreSQL / MySQL,
- users,
- roles,
- permissions,
- connection string,
- connection pool,
- migrations,
- indexes,
- basic performance,
- transactions,
- storage,
- logs,
- backup,
- restore,
- replication alapötlet.

## Labor

```text
Node app
↓
PostgreSQL
```

majd:

```text
backup
↓
adat törlés
↓
restore
```

---

# 8. Első nagy referencia projekt

Itt összekötjük az addigi tudást.

Architektúra:

```text
Internet
↓
DNS
↓
HTTPS
↓
Nginx / Load Balancer
↓
App 1
App 2
↓
PostgreSQL
↓
Backup
```

Lehetőleg Dockerrel.

Cél:

ne csak működjön, hanem tudd elmagyarázni:

```text
mi történik egy requesttel
elejétől a végéig?
```

---

# 9. Security alapok

Ekkorra már lesz mit védenünk.

## Fő témák

- authentication,
- authorization,
- least privilege,
- Linux permissions,
- SSH security,
- firewall,
- secrets,
- environment variables,
- TLS,
- patching,
- dependency security,
- container security alapok,
- basic attack surface.

## Labor

Az addigi referencia projekt hardeningje.

---

# 10. Monitoring és Logging

Eddig megépítettük a rendszert.

Most megtanuljuk:

```text
honnan tudjuk, hogy működik?
```

---

## Fő témák

- metrics,
- logs,
- traces alapötlet,
- CPU,
- memory,
- disk,
- network,
- application metrics,
- alerts,
- health checks.

Később például:

- Prometheus,
- Grafana,
- Loki / log stack.

---

# 11. Backup és Disaster Recovery

## Fő témák

- backup,
- restore,
- RPO,
- RTO,
- retention,
- off-site backup,
- database backup,
- configuration backup,
- disaster recovery.

A legfontosabb szabály:

```text
A backup csak akkor valódi backup,
ha a restore-t is teszteltük.
```

---

# 12. CI/CD mélyebben

GitHub Actions alapokat már korábban láttunk.

Itt térünk vissza rá komolyabban.

## Fő témák

- pipeline,
- build,
- test,
- artifact,
- Docker image build,
- container registry,
- environment,
- staging,
- production,
- secrets,
- deployment,
- rollback.

Példa:

```text
git push
↓
tests
↓
build
↓
Docker image
↓
registry
↓
deployment
↓
production
```

---

# 13. Cloud

Elsőként DigitalOcean, mert már használod és egyszerűbb tanulókörnyezet.

Utána a fogalmakat átfordítjuk Azure/AWS szemléletre.

## Fő témák

- VM,
- virtual network,
- firewall,
- public/private IP,
- block storage,
- object storage,
- load balancer,
- managed database,
- DNS,
- IAM alapok,
- high availability.

---

# 14. Terraform / Infrastructure as Code

Csak akkor jön, amikor már kézzel is tudod:

```text
mit akarunk létrehozni.
```

## Fő témák

- IaC,
- provider,
- resource,
- state,
- variables,
- outputs,
- plan,
- apply,
- destroy,
- modules alapok.

## Labor

Cloud infrastruktúra létrehozása kódból.

---

# 15. Kubernetes

Nem kezdő technológia.

Azért kerül későre, mert Kubernetes egyszerre épít:

- Linuxra,
- networkingre,
- Docker/container tudásra,
- storage-ra,
- securityre,
- monitoringra,
- deploymentre.

## Fő témák

- cluster,
- node,
- pod,
- deployment,
- service,
- ingress,
- config map,
- secret,
- volume,
- namespace,
- health check,
- scaling.

---

# 16. Reliability / SRE alapok

Itt már nem csak az a kérdés:

```text
működik?
```

hanem:

```text
mennyire megbízható?
```

## Fő témák

- availability,
- reliability,
- redundancy,
- failure domains,
- SLA,
- SLI,
- SLO,
- incident,
- postmortem,
- capacity,
- scaling,
- graceful degradation.

---

# 17. Platform Engineering

Ez már későbbi cél.

Itt a DevOps tudásból belső platformot építünk más fejlesztők számára.

## Fő témák

- developer platform,
- self-service,
- golden paths,
- reusable infrastructure,
- templates,
- automation,
- internal tooling,
- Kubernetes platform,
- observability,
- security guardrails.

Mentális modell:

```text
Developer
↓
belső platform
↓
CI/CD
↓
infrastructure
↓
cloud / Kubernetes
```

---

# Tanulási sorrend röviden

```text
0. Computer Systems / OS Basics
   KÉSZ

1. Git + GitHub + GitHub Actions
   ELMÉLET KÉSZ
   → komplex labor következik

2. Linux
   ↓
3. Networking
   ↓
4. Server Administration
   ↓
5. Nginx + HTTP/HTTPS
   ↓
6. Docker + Docker Compose
   ↓
7. Database Operations
   ↓
8. Nagy referencia projekt
   ↓
9. Security
   ↓
10. Monitoring / Logging
   ↓
11. Backup / Disaster Recovery
   ↓
12. CI/CD mélyebben
   ↓
13. Cloud
   ↓
14. Terraform
   ↓
15. Kubernetes
   ↓
16. Reliability / SRE
   ↓
17. Platform Engineering
```

---

# Hogyan tanulunk egy témát?

Minden nagyobb témánál ugyanazt a módszert használjuk.

## 1. Elmélet

Először:

```text
Mi ez?
Mire való?
Hogyan működik?
Miért fontos?
```

---

## 2. Markdown jegyzet

A témából készül egy strukturált `.md` jegyzet.

Nem kell minden mondatot bemagolni.

A cél:

```text
értsd a mentális modellt
```

---

## 3. Mini labor

Azonnal kipróbáljuk.

Például:

```text
process elmélet
↓
ps / top labor
```

vagy:

```text
network port
↓
ss / curl labor
```

---

## 4. Hibakeresés

Szándékosan elrontunk valamit.

Például:

```text
rossz port
rossz permission
leállított service
hibás config
```

Majd megkeressük:

```text
mi a tünet?
↓
hol keresem?
↓
milyen log?
↓
mi az ok?
↓
hogyan javítom?
```

---

## 5. Visszakérdezés

A témát saját szavaiddal próbálod elmondani.

Nem definíciót kell felmondani.

Például:

```text
Mi történik,
amikor beírom a böngészőbe a domain nevét?
```

---

## 6. Nagyobb labor / projekt

Több témát összekapcsolunk.

Példa:

```text
Linux
+
Networking
+
Nginx
+
Node
+
PostgreSQL
+
Docker
```

---

# Mit csináljunk most?

Jelenleg:

```text
Computer Systems jegyzet
↓
átolvasás / ülepedés

Git + GitHub jegyzetek
↓
átolvasás

HOLNAP
↓
Git + GitHub + Actions komplex labor
↓
megoldókulcs
```

Ezután:

```text
LINUX
```

következik.

Nem kell közben előre Dockerrel, Terraformmal vagy Kubernetesszel foglalkozni.

Mindig csak:

```text
aktuális réteg
+
az előző réteg ismétlése
```

legyen a fókusz.
