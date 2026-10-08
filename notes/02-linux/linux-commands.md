# Linux parancsok – gyakorlati referencia
## DevOps / System Administration alapokhoz

Ez a jegyzet nem teljes Linux kézikönyv.

A célja, hogy gyors referencia legyen a leggyakrabban használt parancsokhoz és kapcsolókhoz.

---

# 1. Hol vagyok?

## `pwd`

Az aktuális könyvtár teljes útvonalát mutatja.

```bash
pwd
```

Példa:

```text
/home/tamas/projects
```

`pwd` = **print working directory**

---

# 2. Könyvtár tartalmának listázása

## `ls`

```bash
ls
```

Gyakori kapcsolók:

```text
-l  long listing
-a  all, rejtett fájlok is
-h  human-readable méretek
-R  recursive
```

Példák:

```bash
ls -l
ls -la
ls -lh
ls -lah
```

A leggyakoribb:

```bash
ls -la
```

## Egy könyvtár saját adatainak megtekintése

```bash
ls -ld directory/
```

Ez nagyon hasznos permission hibáknál.

Példa:

```bash
ls -ld /srv/devops-lab
```

---

# 3. Könyvtárváltás

## `cd`

```bash
cd directory
```

Home:

```bash
cd ~
```

Egy szinttel feljebb:

```bash
cd ..
```

Root:

```bash
cd /
```

Előző könyvtár:

```bash
cd -
```

---

# 4. Könyvtár létrehozása

## `mkdir`

```bash
mkdir test
```

Többszintű könyvtár:

```bash
mkdir -p project/src/app
```

`-p` = **parents**

A szükséges szülőkönyvtárakat is létrehozza.

---

# 5. Fájl létrehozása

## `touch`

```bash
touch file.txt
```

Ha a fájl nem létezik, létrehozza.

---

# 6. Másolás

## `cp`

Fájl:

```bash
cp source.txt copy.txt
```

Könyvtár:

```bash
cp -r source/ destination/
```

`-r` = **recursive**

Interaktív megerősítés:

```bash
cp -i source.txt destination.txt
```

`-i` = **interactive**

---

# 7. Mozgatás és átnevezés

## `mv`

Átnevezés:

```bash
mv old.txt new.txt
```

Mozgatás:

```bash
mv file.txt directory/
```

Megerősítést kérhet felülírás előtt:

```bash
mv -i old.txt destination/
```

---

# 8. Törlés

## `rm`

Fájl:

```bash
rm file.txt
```

Könyvtár:

```bash
rm -r directory/
```

Kapcsolók:

```text
-r  recursive
-i  interactive
-f  force
```

Óvatosan:

```bash
rm -rf
```

veszélyes lehet.

Törlés előtt jó rutin:

```bash
pwd
ls -la
```

---

# 9. Fájl tartalmának megtekintése

## `cat`

```bash
cat file.txt
```

Rövid fájlhoz jó.

---

## `less`

```bash
less file.txt
```

Hosszabb fájlhoz.

Kilépés:

```text
q
```

---

## `head`

Első 10 sor:

```bash
head file.txt
```

Első 5 sor:

```bash
head -n 5 file.txt
```

`-n` = **number of lines**

---

## `tail`

Utolsó 10 sor:

```bash
tail file.txt
```

Utolsó 20:

```bash
tail -n 20 file.txt
```

Folyamatos követés:

```bash
tail -f application.log
```

`-f` = **follow**

---

# 10. Szöveg kiírása

## `echo`

```bash
echo "Hello"
```

Environment variable:

```bash
echo $HOME
```

---

# 11. Output átirányítás

Felülírás:

```bash
echo "Hello" > file.txt
```

Hozzáfűzés:

```bash
echo "Next line" >> file.txt
```

```text
>   overwrite
>>  append
```

---

# 12. Standard streams

```text
0 = stdin
1 = stdout
2 = stderr
```

Normál output:

```bash
command > output.log
```

Hibakimenet:

```bash
command 2> error.log
```

Mindkettő ugyanoda:

```bash
command > output.log 2>&1
```

---

# 13. Pipe

## `|`

Az első parancs outputját a második parancs inputjára küldi.

```bash
ps aux | grep node
```

Mental model:

```text
command1
  ↓ stdout
  |
  ↓ stdin
command2
```

---

# 14. Szöveg keresése

## `grep`

```bash
grep "ERROR" application.log
```

Gyakori kapcsolók:

```text
-i  ignore case
-r  recursive
-n  line number
-v  inverse match
```

Példák:

```bash
grep -i "error" application.log
grep -n "ERROR" application.log
grep -r "database" .
```

---

# 15. Fájl keresése

## `find`

```bash
find . -name "config.json"
```

Minden `.log`:

```bash
find . -name "*.log"
```

Csak directory:

```bash
find . -type d
```

Csak file:

```bash
find . -type f
```

Gyakori:

```text
-name  név
-type  típus
```

---

# 16. Sorok / szavak számolása

## `wc`

Sorok száma:

```bash
wc -l file.txt
```

Szavak:

```bash
wc -w file.txt
```

Karakter/byte:

```bash
wc -c file.txt
```

---

# 17. Rendezés

## `sort`

```bash
sort file.txt
```

Fordított sorrend:

```bash
sort -r file.txt
```

Numerikus:

```bash
sort -n numbers.txt
```

---

# 18. Duplikációk eltávolítása

## `uniq`

```bash
sort file.txt | uniq
```

Darabszámmal:

```bash
sort file.txt | uniq -c
```

`-c` = **count**

---

# 19. Aktuális user

## `whoami`

```bash
whoami
```

Megmondja, melyik userként dolgozol.

**English:**

> `whoami` shows the current user.

---

# 20. User részletes adatainak megtekintése

## `id`

Aktuális user:

```bash
id
```

Másik user:

```bash
id devuser
```

Például:

```text
uid=1001(devuser)
gid=1001(devuser)
groups=1001(devuser),1004(devops-lab)
```

Jelentése:

```text
UID    User ID
GID    primary Group ID
groups további group tagságok
```

---

# 21. Felhasználók listázása

Linuxban a userek adatbázisa elérhető például:

```bash
cat /etc/passwd
```

Ez sok system usert is tartalmaz.

Egyszerű username lista:

```bash
cut -d: -f1 /etc/passwd
```

Hasznosabb keresés egy konkrét userre:

```bash
getent passwd devuser
```

Ez általában jobb, mint kézzel keresni a `/etc/passwd` fájlban.

---

# 22. Groupok listázása

Összes group:

```bash
cat /etc/group
```

Egyszerű group lista:

```bash
cut -d: -f1 /etc/group
```

Konkrét group:

```bash
getent group devops-lab
```

Példa:

```text
devops-lab:x:1004:devuser,opsuser,qauser
```

---

# 23. Mely groupokban van egy user?

Egyszerűen:

```bash
groups
```

Aktuális user.

Másik user:

```bash
groups devuser
```

Vagy:

```bash
id devuser
```

Az `id` általában részletesebb.

---

# 24. User létrehozása

Ubuntu alatt kényelmes:

## `adduser`

```bash
sudo adduser devuser
```

Ez interaktívan létrehozza:

- usert,
- home directoryt,
- primary groupot,
- jelszót.

Az `adduser` Ubuntu/Debian alatt barátságosabb frontend.

---

# 25. Group létrehozása

## `addgroup`

```bash
sudo addgroup devops-lab
```

Alternatíva:

```bash
sudo groupadd devops-lab
```

---

# 26. User hozzáadása grouphoz

Ubuntu/Debian:

```bash
sudo adduser devuser devops-lab
```

vagy általánosabb:

```bash
sudo usermod -aG devops-lab devuser
```

Fontos:

```text
-a  append
-G  supplementary groups
```

A `-a` nagyon fontos.

Enélkül más group membershipöket véletlenül felülírhatsz.

---

# 27. Userváltás

Laborban:

```bash
sudo -iu devuser
```

Itt:

```text
-i  login environment
-u  target user
```

Kilépés:

```bash
exit
```

Ellenőrzés váltás után:

```bash
whoami
id
echo $HOME
pwd
```

---

# 28. User törlése

Labor végén:

```bash
sudo deluser devuser
```

Home directoryval együtt:

```bash
sudo deluser --remove-home devuser
```

Csak tudatosan használd.

---

# 29. Group törlése

```bash
sudo delgroup devops-lab
```

vagy:

```bash
sudo groupdel devops-lab
```

---

# 30. File owner és group

## `ls -l`

Példa:

```text
-rw-r----- 1 tamas developers 120 file.txt
```

Itt:

```text
owner = tamas
group = developers
```

---

# 31. Owner módosítása

## `chown`

User:

```bash
sudo chown devuser file.txt
```

User + group:

```bash
sudo chown devuser:devops-lab file.txt
```

Recursive:

```bash
sudo chown -R devuser:devops-lab directory/
```

`-R` = **recursive**

Nagyon figyelj, melyik directoryra alkalmazod.

---

# 32. Group módosítása

## `chgrp`

```bash
sudo chgrp devops-lab file.txt
```

Recursive:

```bash
sudo chgrp -R devops-lab directory/
```

---

# 33. Permissionök megtekintése

```bash
ls -l file.txt
```

Példa:

```text
-rwxr-x---
```

Felbontva:

```text
-    file type

rwx  user
r-x  group
---  others
```

---

# 34. Permissionök jelentése

```text
r = read
w = write
x = execute
```

Kategóriák:

```text
u = user
g = group
o = others
a = all
```

---

# 35. chmod – szimbolikus forma

## `chmod`

Execute hozzáadása usernek:

```bash
chmod u+x script.sh
```

Write hozzáadása groupnak:

```bash
chmod g+w file.txt
```

Read elvétele otherstől:

```bash
chmod o-r file.txt
```

Mindenkinek read:

```bash
chmod a+r file.txt
```

---

# 36. chmod – numerikus forma

Értékek:

```text
r = 4
w = 2
x = 1
```

Gyakori kombinációk:

```text
7 = rwx
6 = rw-
5 = r-x
4 = r--
0 = ---
```

Példák:

```bash
chmod 755 script.sh
```

jelentése:

```text
user   rwx
group  r-x
others r-x
```

---

```bash
chmod 644 file.txt
```

jelentése:

```text
user   rw-
group  r--
others r--
```

---

```bash
chmod 750 directory/
```

jelentése:

```text
user   rwx
group  r-x
others ---
```

---

# 37. sudo

Parancs root jogosultsággal:

```bash
sudo command
```

Példa:

```bash
sudo apt update
```

Elv:

```text
normal user by default
sudo only when required
```

---

# 38. Processzek

## `ps`

```bash
ps
```

Aktuális shellhez kapcsolódó processzek.

Részletesebb:

```bash
ps aux
```

Gyakori keresés:

```bash
ps aux | grep node
```

`ps aux` tipikusan mutatja:

```text
USER
PID
CPU
MEM
COMMAND
```

---

# 39. Interaktív process monitor

## `top`

```bash
top
```

Kilépés:

```text
q
```

---

## `htop`

Ha telepítve van:

```bash
htop
```

Felhasználóbarátabb process monitor.

---

# 40. Background job

Background indítás:

```bash
node app.js &
```

Jobok:

```bash
jobs
```

Foregroundba:

```bash
fg
```

Backgroundba:

```bash
bg
```

---

# 41. Process leállítása

## `kill`

```bash
kill PID
```

Alapból jellemzően:

```text
SIGTERM
```

Kényszerített:

```bash
kill -9 PID
```

Ez:

```text
SIGKILL
```

Elv:

```text
SIGTERM first
SIGKILL only when needed
```

---

# 42. Environment variables

Összes:

```bash
env
```

Példák:

```bash
echo $HOME
echo $USER
echo $PATH
```

Saját:

```bash
export APP_ENV=development
```

```bash
export PORT=4000
```

Ellenőrzés:

```bash
echo $APP_ENV
echo $PORT
```

---

# 43. PATH

```bash
echo $PATH
```

Program helye:

```bash
which node
which git
which bash
```

A shell a `PATH` könyvtáraiban keresi a parancsokat.

---

# 44. Exit code

Az előző parancs exit code-ja:

```bash
echo $?
```

Általában:

```text
0     success
non-0 error
```

---

# 45. Package management – APT

Package index frissítés:

```bash
sudo apt update
```

Telepített csomagok frissítése:

```bash
sudo apt upgrade
```

Telepítés:

```bash
sudo apt install htop
```

Eltávolítás:

```bash
sudo apt remove htop
```

Keresés:

```bash
apt search htop
```

---

# 46. systemd ellenőrzése

```bash
ps -p 1 -o comm=
```

Ha:

```text
systemd
```

akkor systemd fut init/service managerként.

---

# 47. Service állapot

## `systemctl`

```bash
systemctl status nginx
```

Indítás:

```bash
sudo systemctl start nginx
```

Leállítás:

```bash
sudo systemctl stop nginx
```

Restart:

```bash
sudo systemctl restart nginx
```

Reload:

```bash
sudo systemctl reload nginx
```

Bootkor induljon:

```bash
sudo systemctl enable nginx
```

Bootkor ne induljon:

```bash
sudo systemctl disable nginx
```

Mindkettő egyszerre:

```bash
sudo systemctl enable --now nginx
```

Fontos:

```text
start  = now
enable = at boot
```

---

# 48. systemd frissítése unit file módosítás után

Ha service unit file-t hoztál létre vagy módosítottál:

```bash
sudo systemctl daemon-reload
```

Utána például:

```bash
sudo systemctl restart myservice
```

---

# 49. journalctl

System log:

```bash
journalctl
```

Service:

```bash
journalctl -u nginx
```

Utolsó 20 sor:

```bash
journalctl -u nginx -n 20
```

Folyamatos:

```bash
journalctl -u nginx -f
```

Kapcsolók:

```text
-u  unit
-n  number of lines
-f  follow
```

---

# 50. Disk space

## `df`

```bash
df -h
```

`-h` = **human-readable**

Megmutatja a filesystemek:

- méretét,
- használt helyét,
- szabad helyét,
- mount pointját.

---

# 51. Directory mérete

## `du`

```bash
du -sh directory/
```

Kapcsolók:

```text
-s  summary
-h  human-readable
```

---

# 52. Block devices

## `lsblk`

```bash
lsblk
```

Megmutathat:

```text
disk
partition
mount point
```

információkat.

---

# 53. Archive – tar

Archive készítés:

```bash
tar -cf backup.tar directory/
```

Gzip tömörítéssel:

```bash
tar -czf backup.tar.gz directory/
```

Kicsomagolás:

```bash
tar -xzf backup.tar.gz
```

Archive tartalmának listázása:

```bash
tar -tzf backup.tar.gz
```

Kapcsolók:

```text
c  create
x  extract
t  list
z  gzip
f  file
```

---

# 54. SSH

Kapcsolódás:

```bash
ssh user@server
```

Másik port:

```bash
ssh -p 2222 user@server
```

`-p` = port

---

# 55. SSH kulcs generálás

Modern alapválasztás:

```bash
ssh-keygen -t ed25519
```

Megadhatsz külön fájlnevet:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/devops-lab-key
```

Kapcsolók:

```text
-t  key type
-f  output file
```

Létrejön:

```text
devops-lab-key      → PRIVATE KEY
devops-lab-key.pub  → PUBLIC KEY
```

A private keyt soha ne oszd meg.

---

# 56. SSH fájlok

Gyakori könyvtár:

```text
~/.ssh/
```

Szerveren:

```text
~/.ssh/authorized_keys
```

Gyakori permissionök:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Private keynél tipikusan:

```bash
chmod 600 ~/.ssh/private-key
```

---

# 57. Alias

Aktuális shell sessionre:

```bash
alias ll='ls -la'
```

Megtekintés:

```bash
alias
```

Eltávolítás:

```bash
unalias ll
```

---

# 58. Bash konfiguráció

Gyakori:

```text
~/.bashrc
```

Újraolvasás:

```bash
source ~/.bashrc
```

vagy:

```bash
. ~/.bashrc
```

---

# 59. Parancs dokumentáció

## `--help`

```bash
mkdir --help
```

## `man`

```bash
man mkdir
```

Kilépés:

```text
q
```

Ez nagyon fontos:

**Nem kell minden flaget fejből tudni.**

---

# 60. Gyors user/group troubleshooting referencia

Ha nem világos, ki vagy és milyen jogaid vannak:

```bash
whoami
id
groups
```

Ha másik user érdekel:

```bash
id devuser
groups devuser
getent passwd devuser
```

Group ellenőrzés:

```bash
getent group devops-lab
```

Fájl owner/group:

```bash
ls -l file.txt
```

Directory owner/group:

```bash
ls -ld directory/
```

Ez a Linux labor elején az egyik legfontosabb blokk.

---

# 61. Gyors permission troubleshooting referencia

Ha:

```text
Permission denied
```

akkor először:

```bash
whoami
id
ls -l file
ls -ld directory
```

Majd kérdezd:

```text
Who owns it?
Which group owns it?
Am I the owner?
Am I in the group?
Which permission set applies to me?
```

Ne az legyen az első reakció:

```bash
sudo
```

vagy:

```bash
chmod 777
```

---

# 62. Gyors service troubleshooting referencia

Ha egy service nem működik:

```bash
systemctl status service
```

majd:

```bash
journalctl -u service -n 50
```

Utána vizsgáld:

```text
service user
command
path
permissions
environment
configuration
```

---

# 63. Gyors process troubleshooting referencia

```bash
ps aux | grep process-name
```

vagy:

```bash
pgrep process-name
```

PID + parancs:

```bash
pgrep -a node
```

Ha le kell állítani:

```bash
kill PID
```

Csak végső esetben:

```bash
kill -9 PID
```

---

# 64. Gyors disk troubleshooting referencia

```bash
df -h
```

Ha valamelyik filesystem tele van:

```bash
du -sh directory/
```

Majd keresd a nagy könyvtárakat.

---

# 65. Gyors log troubleshooting referencia

Fájl vége:

```bash
tail -n 50 application.log
```

Élő követés:

```bash
tail -f application.log
```

ERROR keresése:

```bash
grep -i "error" application.log
```

Systemd service:

```bash
journalctl -u service -n 50
```

---

# 66. A legfontosabb Linux rutin

Mielőtt módosítasz valamit:

```text
1. Hol vagyok?
2. Ki vagyok?
3. Mi az aktuális állapot?
4. Ki az owner?
5. Mi a group?
6. Mik a permissionök?
7. Fut-e a process/service?
8. Mit mondanak a logok?
```

Parancsok:

```bash
pwd
whoami
id
ls -la
ps aux
systemctl status <service>
journalctl -u <service>
```

Ezután:

```text
understand
↓
change
↓
verify
```

---

# 67. Alap angol kifejezések

| English | Magyar |
|---|---|
| current user | aktuális user |
| user account | felhasználói fiók |
| group membership | csoporttagság |
| file owner | fájl tulajdonosa |
| file permissions | fájljogosultságok |
| read permission | olvasási jog |
| write permission | írási jog |
| execute permission | végrehajtási jog |
| process | folyamat |
| process ID | folyamatazonosító |
| service | szolgáltatás |
| service status | szolgáltatás állapota |
| log file | naplófájl |
| exit code | kilépési kód |
| disk space | lemezterület |
| environment variable | környezeti változó |
| private key | privát kulcs |
| public key | nyilvános kulcs |
| permission denied | hozzáférés megtagadva |

---

# 68. B1 technikai mondatok

- I checked the current user.
- I checked the user's groups.
- The user belongs to the `devops-lab` group.
- This file is owned by `devuser`.
- The group can read the file.
- The user does not have write permission.
- I checked the file permissions.
- The process is running.
- I found the process ID.
- I restarted the service.
- The service failed to start.
- I checked the logs for errors.
- The disk has enough free space.
- The environment variable is not set.
- The private key must remain secret.
- I changed the file owner.
- I changed the group permissions.
- I fixed the permission problem.