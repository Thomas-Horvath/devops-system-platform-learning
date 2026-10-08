# Linux alapok II
## Processes, Services, Logs és rendszerüzemeltetési alapok

# 1. Processes

A process egy futó program példánya.

Minden process kap egy egyedi Process ID-t:

```text
PID = Process ID
```

Példa:

```text
node app.js
    ↓
Node.js process
PID: 4281
```

Processzek listázása:

```bash
ps
ps aux
```

Interaktív megfigyelés:

```bash
top
```

vagy:

```bash
htop
```

Egy processhez tartozhat:

- memória,
- CPU használat,
- user,
- nyitott fájl,
- network connection,
- environment variable,
- thread.

**English:**

> The Node.js process is running.

---

# 2. Foreground és background

Foreground:

```bash
node app.js
```

Background:

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

Fogalmak:

```text
process = operációs rendszer által kezelt futó program
job     = shell által kezelt feladat
```

---

# 3. Signals és kill

Process leállítása:

```bash
kill PID
```

Ez jellemzően:

```text
SIGTERM
```

signalt küld.

Erőszakos leállítás:

```bash
kill -9 PID
```

Ez:

```text
SIGKILL
```

Javasolt sorrend:

```text
SIGTERM
   ↓
graceful shutdown
   ↓
ha nem működik
   ↓
SIGKILL
```

**English:**

> I sent a termination signal to the process.

---

# 4. Service

Egy service egy rendszer által kezelt szolgáltatás.

Példák:

```text
nginx
ssh
postgresql
docker
```

Mental model:

```text
service
   ↓
service manager
   ↓
process
```

---

# 5. systemd és systemctl

Sok Linux distribution a systemd rendszert használja service managerként.

Állapot:

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

Újraindítás:

```bash
sudo systemctl restart nginx
```

Reload:

```bash
sudo systemctl reload nginx
```

Automatikus indulás:

```bash
sudo systemctl enable nginx
```

Kikapcsolása:

```bash
sudo systemctl disable nginx
```

Fontos:

```text
start  = induljon el most
enable = induljon automatikusan bootkor
```

Mindkettő:

```bash
sudo systemctl enable --now nginx
```

**English:**

> The service is running.

> The service starts automatically at boot.

---

# 6. Logs

A log eseményeket és hibákat tartalmazó napló.

Gyakori hely:

```text
/var/log
```

Megtekintés:

```bash
ls /var/log
```

Fájl követése:

```bash
tail -f application.log
```

**English:**

> Check the logs for errors.

---

# 7. journalctl

Systemd journal megtekintése:

```bash
journalctl
```

Service logjai:

```bash
journalctl -u nginx
```

Utolsó 50 sor:

```bash
journalctl -u nginx -n 50
```

Folyamatos követés:

```bash
journalctl -u nginx -f
```

Tipikus troubleshooting:

```text
service failed
    ↓
systemctl status
    ↓
journalctl
    ↓
error
```

**English:**

> The service failed to start, so I checked the logs.

---

# 8. Package management

Ubuntu/Debian rendszeren:

```text
APT
```

Csomaglista frissítése:

```bash
sudo apt update
```

Telepített csomagok frissítése:

```bash
sudo apt upgrade
```

Csomag telepítése:

```bash
sudo apt install nginx
```

Eltávolítás:

```bash
sudo apt remove nginx
```

Keresés:

```bash
apt search nginx
```

Mental model:

```text
package repository
      ↓
apt
      ↓
download
      ↓
install
```

---

# 9. Pipe

A pipe:

```text
|
```

egy parancs outputját egy másik parancs inputjára irányítja.

Példa:

```bash
ps aux | grep node
```

Mental model:

```text
ps aux
  ↓ stdout
  |
  ↓ stdin
grep node
```

**English:**

> The pipe sends the output of one command to another command.

---

# 10. grep

Szöveg keresése:

```bash
grep "error" application.log
```

Case insensitive:

```bash
grep -i "error" application.log
```

Recursive:

```bash
grep -r "database" .
```

Gyakori kombináció:

```bash
ps aux | grep node
```

---

# 11. find

Fájl keresése:

```bash
find . -name "config.json"
```

Logfájlok:

```bash
find /var/log -name "*.log"
```

A `*` wildcard.

```text
*.log
```

jelentése:

```text
bármilyen név + .log
```

---

# 12. További hasznos parancsok

Sorok száma:

```bash
wc -l file.txt
```

Rendezés:

```bash
sort file.txt
```

Duplikációk kiszűrése:

```bash
sort file.txt | uniq
```

Program helye:

```bash
which node
```

---

# 13. Standard streams

Három alap stream:

```text
stdin  = standard input
stdout = standard output
stderr = standard error
```

Azonosítók:

```text
0 = stdin
1 = stdout
2 = stderr
```

stdout fájlba:

```bash
command > output.txt
```

Hozzáfűzés:

```bash
command >> output.txt
```

stderr:

```bash
command 2> error.log
```

stdout és stderr együtt:

```bash
command > output.log 2>&1
```

---

# 14. Exit code

Az utolsó parancs exit code-ja:

```bash
echo $?
```

Általában:

```text
0     = success
!= 0  = error
```

Ez CI környezetben különösen fontos.

```text
npm test
    ↓
exit code
    ↓
0      → CI SUCCESS
non-0  → CI FAILED
```

---

# 15. Environment variables

Environment variable megtekintése:

```bash
env
```

Példák:

```bash
echo $HOME
echo $USER
echo $PATH
```

Saját változó:

```bash
export APP_ENV=development
```

Lekérdezés:

```bash
echo $APP_ENV
```

Példák alkalmazáskonfigurációra:

```text
NODE_ENV
DATABASE_URL
PORT
API_URL
```

**English:**

> Environment variables store configuration values.

---

# 16. PATH

A PATH megtekintése:

```bash
echo $PATH
```

Példa:

```text
/usr/local/bin:/usr/bin:/bin
```

Amikor ezt futtatjuk:

```bash
node
```

a shell a PATH könyvtáraiban keresi a programot.

Program helyének ellenőrzése:

```bash
which node
```

**English:**

> The shell searches for commands in the PATH.

---

# 17. Bash configuration

Gyakori Bash konfigurációs fájl:

```text
~/.bashrc
```

Itt lehet például:

- alias,
- environment variable,
- PATH módosítás.

Alias példa:

```bash
alias ll='ls -la'
```

---

# 18. Disk space

Filesystem szabad területe:

```bash
df -h
```

```text
-h = human-readable
```

Könyvtár mérete:

```bash
du -sh directory/
```

```text
-s = summary
-h = human-readable
```

Például:

```bash
du -sh node_modules/
```

Troubleshooting:

```text
disk almost full
      ↓
df -h
      ↓
find filesystem
      ↓
du
      ↓
find large directory
```

---

# 19. Block devices és mount

Block device-ek:

```bash
lsblk
```

Mental model:

```text
disk
 ↓
partition
 ↓
filesystem
 ↓
mount point
 ↓
Linux directory tree
```

Példa mount point:

```text
/mnt/data
```

---

# 20. SSH

SSH:

```text
Secure Shell
```

Távoli szerverhez kapcsolódás:

```bash
ssh user@server
```

Mental model:

```text
local computer
     ↓
encrypted connection
     ↓
remote Linux server
     ↓
shell
```

---

# 21. SSH keys

SSH key pair:

```text
private key
public key
```

Private key:

```text
marad a kliensen
titkos
```

Public key:

```text
mehet a szerverre
```

Gyakori szerveroldali hely:

```text
~/.ssh/authorized_keys
```

**English:**

> The private key must remain secret.

---

# 22. tar és gzip

Archive létrehozása:

```bash
tar -cf backup.tar project/
```

Gzip tömörítéssel:

```bash
tar -czf backup.tar.gz project/
```

Kicsomagolás:

```bash
tar -xzf backup.tar.gz
```

Kapcsolók:

```text
c = create
x = extract
z = gzip
f = file
```

---

# 23. Linux troubleshooting mental model

Ha egy alkalmazás nem működik:

```text
APPLICATION NOT WORKING
         ↓
Is the process running?
         ↓
Is the service running?
         ↓
What do the logs say?
         ↓
Are permissions correct?
         ↓
Is there enough disk space?
         ↓
Is the configuration correct?
         ↓
Later: Is the network working?
```

Hasznos parancsok:

```bash
ps aux
systemctl status <service>
journalctl -u <service>
ls -l
df -h
```

Alapelv:

```text
inspect state
    ↓
identify problem
    ↓
change state
    ↓
verify result
```

---

# 24. Fontos angol szókincs

| English | Magyar |
|---|---|
| process | folyamat |
| PID | folyamatazonosító |
| foreground | előtér |
| background | háttér |
| signal | jelzés |
| terminate | leállítani |
| service | szolgáltatás |
| service manager | szolgáltatáskezelő |
| log | napló |
| package | csomag |
| package manager | csomagkezelő |
| pipe | csővezeték / output-input összekötés |
| standard input | szabványos bemenet |
| standard output | szabványos kimenet |
| standard error | szabványos hibakimenet |
| exit code | kilépési kód |
| environment variable | környezeti változó |
| executable | futtatható program |
| disk space | lemezterület |
| mount point | csatolási pont |
| remote server | távoli szerver |
| private key | privát kulcs |
| public key | nyilvános kulcs |
| archive | archívum |

---

# 25. B1 technikai mondatok

- The Node.js process is running.
- The process has a unique PID.
- I stopped the process with SIGTERM.
- The service failed to start.
- I checked the service status.
- I checked the logs for errors.
- The service starts automatically at boot.
- I installed the package with APT.
- The pipe sends output to another command.
- I searched the log file with grep.
- Environment variables store configuration values.
- The shell searches for commands in the PATH.
- The disk is almost full.
- I checked the available disk space.
- I connected to the server using SSH.
- The private key must remain secret.
- The test returned a non-zero exit code.
- First, I checked the current system state.

---

# 26. Linux foundation – amit ezek után tudni kell

A Linux alapozás végére érteni kell:

```text
filesystem
paths
files/directories
users/groups
permissions
chmod/chown/sudo
processes
signals
foreground/background
services
systemd
logs
packages
pipes
grep/find
stdin/stdout/stderr
exit codes
environment variables
PATH
disk usage
mount basics
SSH
archive/compression
basic troubleshooting
```

Ezek után a következő nagy témakör:

```text
NETWORKING
```