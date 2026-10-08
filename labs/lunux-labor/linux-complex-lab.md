# Linux System Administration – komplex DevOps labor

## A labor célja

Ebben a laborban egy kis Node.js alkalmazást fogsz Linux rendszeren **üzemeltetési szemmel átvenni és beállítani**.

Nem az a cél, hogy Linux-parancsokat egymástól függetlenül bemagolj.

A cél:

```text
Linux filesystem
        ↓
users + groups
        ↓
permissions
        ↓
application files
        ↓
process
        ↓
systemd service
        ↓
environment variables
        ↓
logs
        ↓
troubleshooting
        ↓
backup
```

A labor során szándékosan fogsz hibákat is létrehozni és kijavítani.

---

# 0. Környezet

A labor ugyanabban a környezetben készül, mint a Git/GitHub labor.

```text
Windows 11
↓
WSL2
↓
Ubuntu
```

Jelenlegi verziók:

```text
Git:  2.34.1
Node: v20.19.6
npm:  11.12.1
```

Használható:

- WSL2 Ubuntu
- Bash
- VS Code
- Node.js
- npm
- sudo

---

# 1. Nagyon fontos biztonsági szabály

A rendszerben jelenleg van:

```text
root
+
a saját normál felhasználód
```

A saját felhasználód beállításait **nem módosítjuk**.

Nem változtatjuk:

- a saját UID-det,
- primary groupodat,
- home directorydat,
- sudo jogosultságodat,
- shell beállításaidat,
- `.ssh` tartalmadat.

A laborhoz külön felhasználókat hozunk létre.

A labor végén csak ezeket a labor-useröket törölhetjük.

## Különösen óvatosan

Ne használj vakon:

```bash
rm -rf
chmod -R
chown -R
userdel
groupdel
kill -9
```

Mindig ellenőrizd előtte:

```bash
pwd
whoami
id
ls -la
```

---

# 2. A történet

Egy képzeletbeli cégnél junior DevOps/System Engineer vagy.

A fejlesztők elkészítettek egy kis alkalmazást:

```text
Status API
```

A te feladatod:

1. megfelelő Linux userek és groupok kialakítása,
2. alkalmazás elhelyezése,
3. jogosultságok beállítása,
4. program kézi futtatása,
5. processzek vizsgálata,
6. service létrehozása,
7. environment variable-ök használata,
8. logok ellenőrzése,
9. hibák diagnosztizálása,
10. backup készítése.

---

# 3. Laborfelhasználók

A labor során az alábbi useröket használjuk:

```text
devuser
opsuser
qauser
```

Szerepek:

```text
devuser
→ developer

opsuser
→ operations / DevOps

qauser
→ tester / QA
```

Hozz létre egy közös groupot is:

```text
devops-lab
```

## Feladat

Hozd létre a három felhasználót és a groupot.

A megfelelő felhasználókat add a `devops-lab` grouphoz.

Ne adj automatikusan mindenkinek sudo jogosultságot.

## Ellenőrzés

Vizsgáld meg:

```bash
id devuser
id opsuser
id qauser
```

és:

```bash
getent group devops-lab
```

## Gondolkodási kérdés

Miért jobb egy közös groupnak jogosultságot adni, mint minden usert külön kezelni?

---

# 4. Felhasználóváltás

A laborban többször át fogsz váltani más userre.

Használhatsz például:

```bash
sudo -iu devuser
```

vagy:

```bash
sudo -iu opsuser
```

## Feladat

Válts át egymás után:

```text
devuser
opsuser
qauser
```

Minden váltás után ellenőrizd:

```bash
whoami
id
pwd
echo $HOME
```

## Figyeld meg

Minden usernek külön:

```text
UID
GID
HOME
groups
```

tartozik.

---

# 5. Alkalmazáskönyvtár

Az alkalmazásunk helye legyen:

```text
/srv/devops-lab/
```

Struktúra:

```text
/srv/devops-lab/
├── app/
├── config/
├── data/
└── backup/
```

## Feladat

Hozd létre a struktúrát.

Utána:

```bash
ls -la /srv/devops-lab
```

Vizsgáld meg:

- owner
- group
- permissions

---

# 6. Ownership beállítása

Az alkalmazásért az üzemeltetési csoport felel.

A könyvtár groupja legyen:

```text
devops-lab
```

## Feladat

Állítsd be úgy a tulajdonjogokat, hogy megfelelő laboruserek hozzáférjenek.

Használd:

```text
chown
chgrp
```

ahol indokolt.

## Ellenőrzés

```bash
ls -ld /srv/devops-lab
ls -la /srv/devops-lab
```

---

# 7. Permission labor

Most tudatosan gyakoroljuk:

```text
user
group
others
```

és:

```text
r
w
x
```

## Első feladat

A `config` könyvtárhoz:

```text
owner → read/write/execute
group → read/execute
others → no access
```

Állítsd be először **szimbolikus chmod** segítségével.

Például gondolkodj:

```text
u = ?
g = ?
o = ?
```

Ne csak másold a parancsot.

---

# 8. Numerikus chmod

Ugyanezt a permissiont írd le numerikus formában.

Használd:

```text
r = 4
w = 2
x = 1
```

Számítsd ki:

```text
user
group
others
```

értékeit.

## Ellenőrzés

```bash
ls -ld /srv/devops-lab/config
```

Magyarázd el saját szavaiddal a kapott:

```text
rwx...
```

permission stringet.

---

# 9. Permission denied szimuláció

Válts át:

```text
qauser
```

Próbálj létrehozni egy fájlt a `config` könyvtárban.

## Elvárt eredmény

```text
Permission denied
```

Ne használj rögtön sudo-t.

Vizsgáld meg:

```bash
whoami
id
ls -ld /srv/devops-lab/config
```

## Kérdés

Miért nincs írási joga a usernek?

---

# 10. Application starter code

Az alkalmazás fájlja legyen:

```text
/srv/devops-lab/app/server.js
```

Tartalma:

```javascript
const http = require("http");

const PORT = Number(process.env.PORT || 3000);
const APP_ENV = process.env.APP_ENV || "development";

const server = http.createServer((req, res) => {
  const now = new Date().toISOString();

  console.log(`[INFO] ${now} ${req.method} ${req.url}`);

  if (req.url === "/health") {
    res.writeHead(200, { "Content-Type": "application/json" });
    return res.end(
      JSON.stringify({
        status: "ok",
        environment: APP_ENV,
      })
    );
  }

  if (req.url === "/error") {
    console.error(`[ERROR] ${now} Test error requested`);

    res.writeHead(500, { "Content-Type": "application/json" });
    return res.end(
      JSON.stringify({
        error: "Test error",
      })
    );
  }

  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("DevOps Linux Lab\n");
});

server.listen(PORT, () => {
  console.log(
    `[INFO] Application started on port ${PORT} in ${APP_ENV} mode`
  );
});

process.on("SIGTERM", () => {
  console.log("[INFO] SIGTERM received. Shutting down.");

  server.close(() => {
    console.log("[INFO] Server stopped.");
    process.exit(0);
  });
});
```

Nem a JavaScript megértése a feladat.

A kód azért ilyen, hogy legyen:

```text
process
environment variable
stdout
stderr
SIGTERM
service
log
```

amit vizsgálni tudunk.

---

# 11. Application ownership

Az alkalmazás fájljainak megfelelő:

```text
owner
group
permissions
```

beállítása legyen a feladatod.

A developer módosíthassa az alkalmazáskódot.

A group tagjai legalább olvasni tudják.

Az `others` ne kapjon felesleges írási jogot.

## Ellenőrzés

```bash
ls -la /srv/devops-lab/app
```

---

# 12. Environment variables

Az alkalmazás két environment variable-t használ:

```text
PORT
APP_ENV
```

Válts a megfelelő laboruserre.

Állíts be:

```text
APP_ENV=development
PORT=4000
```

az aktuális shell sessionben.

## Ellenőrzés

```bash
echo $APP_ENV
echo $PORT
```

Nézd meg:

```bash
env
```

és keresd meg őket.

---

# 13. Application kézi indítása

Indítsd el:

```text
server.js
```

Node.js segítségével.

Ne systemd service-ként.

## Feladat

Figyeld meg:

```text
foreground process
```

viselkedését.

Másik terminálban keresd meg:

```bash
ps aux
```

és:

```bash
ps aux | grep node
```

## Azonosítsd

```text
USER
PID
CPU
MEM
COMMAND
```

---

# 14. Process ID

Jegyezd fel a Node process:

```text
PID
```

értékét.

Vizsgáld meg:

```bash
ps
```

és:

```bash
ps aux
```

közötti különbséget.

---

# 15. Process leállítása

Küldj a Node processnek szabályos leállítási signalt.

Ne használj rögtön:

```text
SIGKILL
```

## Figyeld meg

Az alkalmazásnak ki kell írnia:

```text
SIGTERM received
```

majd le kell állnia.

## Kérdés

Miért jobb először SIGTERM-et használni SIGKILL helyett?

---

# 16. Background process

Indítsd el az alkalmazást backgroundban.

Használd:

```text
&
```

Majd vizsgáld meg:

```bash
jobs
```

## Feladat

Hozd vissza foregroundba.

Majd állítsd le.

---

# 17. Standard output

Indítsd el az alkalmazást úgy, hogy a normál output egy fájlba kerüljön.

Például:

```text
application.log
```

Használd az output redirection fogalmát.

## Vizsgáld meg

```bash
cat
less
tail
```

segítségével.

---

# 18. stdout vs stderr

Az alkalmazás:

```text
/health
```

kérésnél normál logot ír.

Az:

```text
/error
```

kérésnél hibát ír stderr-re.

A konkrét HTTP-kéréseket később networkingnél részletesen tanuljuk; itt csak a logok miatt használjuk őket.

## Feladat

Irányítsd külön fájlba:

```text
stdout
stderr
```

kimenetet.

Használd a:

```text
1
2
```

file descriptorok ismeretét.

---

# 19. Exit code

Futtass egy sikeres parancsot.

Utána:

```bash
echo $?
```

Majd futtass egy szándékosan hibás parancsot.

Ismét:

```bash
echo $?
```

## Figyeld meg

```text
0
```

és:

```text
non-zero
```

exit code különbségét.

## Kapcsolat

Írd le:

```text
Linux exit code
↓
npm test
↓
GitHub Actions
```

kapcsolatát.

---

# 20. grep labor

Készíts egy több soros logfájlt, amely tartalmaz például:

```text
INFO
WARNING
ERROR
```

sorokat.

## Feladat

Keress benne:

```text
ERROR
```

sorokat.

Majd:

```text
error
ERROR
Error
```

mindegyikre érzéketlenül.

## Használd

```text
grep
grep -i
```

---

# 21. Pipe labor

Használj pipe-ot.

Például:

```text
process list
↓
pipe
↓
grep
```

A cél:

csak a Node processt megtalálni.

## Magyarázd el

Mi az első program:

```text
stdout
```

kimenete?

Mi lesz a következő program:

```text
stdin
```

bemenete?

---

# 22. find labor

A `/srv/devops-lab` alatt hozz létre néhány:

```text
.log
.txt
.conf
```

fájlt.

## Feladat

Keresd meg:

1. az összes `.log` fájlt,
2. egy konkrét nevű config fájlt,
3. az összes fájlt egy adott könyvtárfa alatt.

Használd:

```text
find
-name
*
```

---

# 23. sort / uniq / wc

Készíts egy fájlt ilyen tartalommal:

```text
INFO
ERROR
INFO
WARNING
ERROR
ERROR
INFO
```

## Feladat

Használd együtt:

```text
sort
uniq
wc
```

parancsokat.

Próbáld megállapítani:

- milyen típusú logbejegyzések vannak,
- hány sor található a fájlban.

---

# 24. PATH

Vizsgáld meg:

```bash
echo $PATH
```

Majd:

```bash
which node
which git
which bash
```

## Magyarázd el

Miért elég ezt írni:

```bash
node
```

ahelyett, hogy teljes útvonalat írnánk?

---

# 25. Bash alias

Az egyik laboruserhez hozz létre ideiglenesen alias-t:

```text
ll
```

ami:

```text
ls -la
```

parancsot futtat.

## Feladat

Vizsgáld meg:

```text
alias
```

és próbáld ki.

A permanent `.bashrc` módosítást csak akkor végezd el, ha pontosan érted, mit csinálsz.

---

# 26. Package management

Ellenőrizd az APT működését.

## Feladat

Használd:

```text
apt search
apt update
```

Majd keress egy egyszerű csomagot, például:

```text
htop
```

Ha nincs telepítve, telepítsd.

## Ellenőrzés

```bash
which htop
```

és:

```bash
htop
```

---

# 27. systemd ellenőrzése

Mielőtt saját service-t készítünk, ellenőrizd:

```bash
ps -p 1 -o comm=
```

Ideális esetben:

```text
systemd
```

## Fontos

Ha a WSL környezetben nem systemd fut PID 1-ként, itt állj meg.

Ne módosítsd önállóan a WSL konfigurációját.

A következő alkalommal együtt beállítjuk.

---

# 28. Saját systemd service

Ha systemd működik, hozz létre egy service-t:

```text
devops-lab-app.service
```

A unit file kerüljön a megfelelő systemd könyvtárba.

A service:

```text
Node.js
↓
/srv/devops-lab/app/server.js
```

alkalmazást futtassa.

## Fontos

Ne root userként futtasd az alkalmazást.

Használj megfelelő laborusert.

---

# 29. Service environment

A service kapja meg:

```text
PORT
APP_ENV
```

értékeket.

Például:

```text
PORT=4000
APP_ENV=production
```

A konfigurációt lehetőleg ne égesd bele magába az alkalmazáskódba.

## Gondolkodási kérdés

Miért jobb configurationt és application code-ot szétválasztani?

---

# 30. systemctl

A saját service-en gyakorold:

```text
status
start
stop
restart
```

Majd:

```text
enable
disable
```

## Magyarázd el

Mi a különbség:

```text
start
```

és:

```text
enable
```

között?

---

# 31. journalctl

Nézd meg a saját service logját.

Használd:

```text
journalctl
```

és a megfelelő:

```text
-u
```

kapcsolót.

## Feladat

Nézd meg:

- az összes service-logot,
- csak az utolsó 20 sort,
- folyamatos logkövetést.

---

# 32. Szándékosan hibás service

Most direkt törjük el.

Módosítsd a service konfigurációját úgy, hogy hibás:

```text
JavaScript file path
```

szerepeljen benne.

Majd indítsd újra.

## Elvárt eredmény

```text
service failed
```

## Tilos

Ne kezdj véletlenszerűen módosítani dolgokat.

Először:

```text
inspect state
```

---

# 33. Service troubleshooting

A hibás service-nél vizsgáld meg:

```bash
systemctl status <service>
```

majd:

```bash
journalctl -u <service>
```

## Feladat

Találd meg:

1. mi hibázott,
2. milyen command futott volna,
3. milyen fájlt keresett,
4. miért nem találta.

Ezután javítsd.

---

# 34. Permission error service-nél

Újabb szándékos hiba.

Változtasd meg úgy egy szükséges fájl vagy könyvtár permissionjét, hogy a service user ne férjen hozzá.

## Elvárt

```text
Permission denied
```

## Hibakeresés

Vizsgáld:

```text
service user
owner
group
permissions
```

adatokat.

Ne oldd meg automatikusan `chmod 777` használatával.

---

# 35. Miért nem használunk chmod 777-et mindenre?

Írd le saját szavaiddal.

Vizsgáld meg:

```text
least privilege
```

elvét.

A user csak annyi jogosultságot kapjon, amennyire valóban szüksége van.

---

# 36. Disk usage

Nézd meg:

```bash
df -h
```

## Feladat

Azonosítsd:

- filesystemeket,
- méretet,
- felhasznált helyet,
- szabad helyet,
- mount pointokat.

---

# 37. Directory size

Használd:

```text
du
```

a labor könyvtárra.

Nézd meg:

```text
/srv/devops-lab
```

teljes méretét.

Majd külön:

```text
app
data
backup
```

méreteit.

---

# 38. Block devices

Futtasd:

```bash
lsblk
```

## Feladat

Próbáld meg azonosítani:

```text
disk
partition
mount point
```

fogalmakat.

WSL alatt az eredmény eltérhet egy hagyományos fizikai Linux szervertől.

Ne módosíts mountokat ebben a laborban.

Ez megfigyelési feladat.

---

# 39. Backup

Készíts backupot:

```text
/srv/devops-lab/app
```

könyvtárról.

A célfájl legyen például:

```text
app-backup.tar.gz
```

Használj:

```text
tar
gzip
```

kombinációt.

---

# 40. Backup ellenőrzése

A backup elkészítése még nem jelenti azt, hogy működik.

## Feladat

Vizsgáld meg az archive tartalmát.

Majd hozz létre külön:

```text
restore-test
```

könyvtárat.

Csomagold ki oda.

## Ellenőrzés

Hasonlítsd össze:

```text
eredeti
backupból visszaállított
```

struktúrát.

---

# 41. SSH kulcspár

A saját meglévő SSH kulcsaidhoz nem nyúlunk.

Az egyik laboruser alatt hozz létre külön, csak laborhoz használt SSH key pairt.

A kulcs neve jelezze egyértelműen, hogy:

```text
LAB
```

## Vizsgáld meg

```text
private key
public key
```

közti különbséget.

## Fontos

Private key tartalmát:

```text
SOHA NE MÁSold be a chatbe
SOHA NE commitold Gitbe
```

---

# 42. `.ssh` permissions

Vizsgáld meg:

```text
~/.ssh
```

könyvtár permissionjeit.

Vizsgáld meg a private key permissionjét.

## Gondolkodási kérdés

Miért probléma, ha minden user olvashatja a private keyt?

---

# 43. SSH authorized_keys elméleti gyakorlat

Készíts a laboruser home directoryjában megfelelő:

```text
~/.ssh/
```

struktúrát.

Vizsgáld meg:

```text
authorized_keys
```

szerepét.

Nem szükséges tényleges távoli szerverhez kapcsolódni.

A valódi SSH server labort később a Networking/Server Administration témánál csináljuk.

---

# 44. Összetett troubleshooting scenario

Most tegyük fel:

```text
The application is not working.
```

Semmi más információt nem kapsz.

## Feladat

Készíts saját troubleshooting sorrendet.

Használd legalább:

```text
process
service
logs
permissions
disk space
configuration
environment variables
```

fogalmakat.

Alapelv:

```text
inspect
↓
understand
↓
change
↓
verify
```

---

# 45. Szándékosan összetett hiba

A labor utolsó részében egyszerre több hibát fogunk létrehozni.

Például:

```text
wrong environment variable
+
wrong file permission
+
stopped service
```

A konkrét hibákat a labor közben kapod meg.

Nem fogod előre tudni, melyik probléma van jelen.

A cél valódi troubleshooting.

---

# 46. Hasznos diagnosztikai parancsok

Ezeket a labor során használd:

```bash
whoami
id
pwd

ls
ls -la
ls -ld

ps
ps aux

jobs

systemctl status <service>
journalctl -u <service>

env
echo $PATH

which node

df -h
du -sh

find
grep

echo $?
```

Ne a listát memorizáld.

Azt tanuld meg:

```text
milyen kérdésre
melyik parancs
ad választ
```

---

# 47. Ellenőrző kérdések

A labor végén válaszolj saját szavaiddal.

## Users / Groups

1. Mi a user?
2. Mi a UID?
3. Mi a group?
4. Mi a GID?
5. Miért használunk groupokat?
6. Mi az owner?

## Permissions

7. Mit jelent `r`, `w`, `x` fájlnál?
8. Mit jelent `r`, `w`, `x` directorynál?
9. Mi a user/group/others?
10. Mit jelent `chmod 755`?
11. Mit jelent `chmod 644`?
12. Mire jó a `chown`?
13. Mire jó a `chgrp`?
14. Miért rossz alapmegoldás a `chmod 777`?
15. Mi a least privilege?

## Processes

16. Mi a process?
17. Mi a PID?
18. Mi a különbség foreground és background között?
19. Mi az a shell job?
20. Mire jó a `ps aux`?
21. Mi a különbség SIGTERM és SIGKILL között?

## Services

22. Mi a különbség process és service között?
23. Mi a systemd?
24. Mi a `systemctl`?
25. Mi a különbség `start` és `enable` között?
26. Mire jó a `restart`?
27. Mire jó a `reload`?

## Logs

28. Miért fontosak a logok?
29. Mire jó a `journalctl`?
30. Mire jó a `tail -f`?
31. Mit néznél meg először failed service esetén?

## Shell

32. Mi az stdin?
33. Mi az stdout?
34. Mi az stderr?
35. Mit csinál a `>`?
36. Mit csinál a `>>`?
37. Mi a pipe?
38. Mire jó a grep?
39. Mire jó a find?
40. Mi az exit code?

## Environment

41. Mi az environment variable?
42. Mire jó a PATH?
43. Mit csinál az `export`?
44. Miért használunk environment variable-t konfigurációhoz?

## Storage

45. Mire jó a `df`?
46. Mire jó a `du`?
47. Mi a mount point?
48. Mit mutat az `lsblk`?

## SSH

49. Mi az SSH?
50. Mi a private key?
51. Mi a public key?
52. Mire jó az `authorized_keys`?
53. Miért kell védeni a private keyt?

## Backup

54. Mire jó a `tar`?
55. Mit jelent a gzip?
56. Miért kell a backupot restore-ral is tesztelni?

---

# 48. B1 szakmai angol gyakorlat

A labor végén próbáld saját szavaiddal elmondani:

> I created three users for the lab.

> I added the users to a common group.

> I configured file permissions.

> The application runs as a Linux service.

> The service failed to start.

> I checked the service status and the logs.

> I found a permission problem.

> I changed the file owner and permissions.

> I restarted the service.

> The application is running again.

> I checked the available disk space.

> I created and tested a backup.

> The private SSH key must remain secret.

---

# 49. Mikor tekinthető késznek?

A Linux labor akkor kész, ha:

- [ ] létrehoztad a labor usereket,
- [ ] létrehoztad a közös groupot,
- [ ] váltottál userek között,
- [ ] használtál `whoami` és `id` parancsot,
- [ ] beállítottál owner/group értékeket,
- [ ] használtál szimbolikus chmodot,
- [ ] használtál numerikus chmodot,
- [ ] kaptál és kijavítottál `Permission denied` hibát,
- [ ] futtattál foreground processt,
- [ ] futtattál background processt,
- [ ] használtál PID-et,
- [ ] használtál SIGTERM-et,
- [ ] használtál pipe-ot,
- [ ] használtál grep-et,
- [ ] használtál findot,
- [ ] vizsgáltál stdout/stderr kimenetet,
- [ ] ellenőriztél exit code-ot,
- [ ] használtál environment variable-t,
- [ ] megvizsgáltad a PATH-ot,
- [ ] használtál package managert,
- [ ] létrehoztál systemd service-t,
- [ ] használtál systemctl-t,
- [ ] vizsgáltál journal logot,
- [ ] kijavítottál hibás service-t,
- [ ] kijavítottál permission hibát,
- [ ] használtál `df` és `du` parancsot,
- [ ] megvizsgáltad az `lsblk` eredményt,
- [ ] létrehoztál tar.gz backupot,
- [ ] tesztelted a restore-t,
- [ ] létrehoztál külön labor SSH key pairt,
- [ ] érted a private/public key különbségét,
- [ ] végigcsináltál egy összetett troubleshooting feladatot.

---

# 50. A labor végső rendszerképe

```text
Linux server
│
├── Users
│   ├── devuser
│   ├── opsuser
│   └── qauser
│
├── Group
│   └── devops-lab
│
├── /srv/devops-lab
│   ├── application
│   ├── config
│   ├── data
│   └── backup
│
├── Permissions
│   ├── owner
│   ├── group
│   └── others
│
├── Node.js process
│
├── systemd
│   └── devops-lab-app.service
│
├── Environment
│   ├── PORT
│   └── APP_ENV
│
├── Logs
│   └── journalctl
│
├── Storage
│   ├── df
│   ├── du
│   └── lsblk
│
├── SSH
│   ├── private key
│   └── public key
│
└── Backup
    └── tar.gz
```

A labor legfontosabb gondolkodási mintája:

```text
WHAT IS THE CURRENT STATE?
          ↓
WHAT SHOULD THE STATE BE?
          ↓
WHAT IS DIFFERENT?
          ↓
CHANGE IT
          ↓
VERIFY THE RESULT
```

Magyarul:

```text
Mi a jelenlegi állapot?
        ↓
Mi lenne a helyes állapot?
        ↓
Mi a különbség?
        ↓
Módosítsd
        ↓
Ellenőrizd az eredményt
```

Ez a Linux rendszerüzemeltetés és a későbbi DevOps troubleshooting egyik legfontosabb alapelve.