# Linux alapok

## 1. Linux és Linux kernel

Szigorúan véve a Linux a **kernel**, vagyis az operációs rendszer központi része.

A kernel többek között:

- kezeli a CPU-t,
- kezeli a memóriát,
- kezeli a folyamatokat,
- kezeli az eszközöket,
- kezeli a fájlrendszert,
- kezeli a hálózati műveleteket,
- rendszerhívásokat biztosít a programok számára.

Egyszerű modell:

```text
Application
    ↓
User space
    ↓
System call
    ↓
Linux kernel
    ↓
Hardware
```

**English:**  
The Linux kernel manages system resources.

---

## 2. Linux distribution

A Linux distribution egy Linux kernelre épülő teljes operációs rendszer-környezet.

Példák:

- Ubuntu
- Debian
- Fedora
- Rocky Linux
- Red Hat Enterprise Linux

Egy disztribúció többek között tartalmaz:

```text
Linux kernel
+ shell
+ system tools
+ libraries
+ package manager
+ configuration
+ applications
```

Az Ubuntu Debian-alapú Linux distribution.

---

## 3. WSL2 és Ubuntu

Windows 11 alatt a WSL2 lehetővé teszi Linux környezet futtatását.

Egyszerű modell:

```text
Windows 11
    ↓
WSL2
    ↓
Linux kernel
    ↓
Ubuntu
```

A WSL-ben megtanult Linux parancsok és alapelvek nagy része ugyanúgy használható valódi Linux szervereken és cloud VM-eken is.

---

# 4. Linux filesystem

Linuxban nincs `C:\` és `D:\` típusú meghajtóstruktúra.

A teljes fájlrendszer egyetlen gyökérből indul:

```text
/
```

Ezt **root directorynak** nevezzük.

Példa:

```text
/
├── home
├── etc
├── var
├── usr
├── tmp
├── dev
└── proc
```

Fontos:

```text
/       = filesystem root
/root   = root user home directory
```

---

# 5. Fontos Linux könyvtárak

## `/home`

A normál felhasználók saját könyvtárai.

Példa:

```text
/home/user
```

**English:**  
My project is in my home directory.

---

## `/etc`

Főleg rendszer- és programkonfigurációk.

Példák:

```text
/etc/nginx
/etc/ssh
/etc/systemd
/etc/hosts
```

Egyszerű mental model:

```text
/etc ≈ configuration
```

**English:**  
The configuration file is in `/etc`.

---

## `/var`

Futás közben változó adatok.

Példák:

```text
/var/log
/var/lib
/var/cache
```

A `/var/log` sok rendszer- és alkalmazáslog helye.

---

## `/usr`

Telepített programok, libraryk és egyéb rendszerkomponensek jelentős része.

Példák:

```text
/usr/bin
/usr/lib
/usr/local
```

---

## `/tmp`

Ideiglenes fájlok.

```text
/tmp
```

---

## `/dev`

Eszközök fájlszerű reprezentációi.

Példák:

```text
/dev/null
/dev/sda
```

Hasznos Unix/Linux mental model:

> Everything is a file.

---

## `/proc`

A kernel által biztosított virtuális fájlrendszer.

Információkat tartalmazhat például:

- processzekről,
- CPU-ról,
- memóriáról,
- kernel állapotról.

Példa:

```text
/proc/1234
```

ahol `1234` egy process PID-je lehet.

---

# 6. Path — útvonal

## Absolute path

Mindig a `/` root directoryból indul.

Példa:

```text
/home/user/projects/app
```

## Relative path

Az aktuális könyvtárhoz képest értendő.

Ha az aktuális könyvtár:

```text
/home/user
```

akkor:

```bash
cd projects
```

a következőt jelenti:

```text
/home/user/projects
```

---

# 7. Speciális útvonalak

```text
.   current directory
..  parent directory
~   user's home directory
```

Példák:

```bash
cd ..
cd ~
cd ~/projects
```

---

# 8. Alap Linux parancsok

## `pwd`

Print working directory.

```bash
pwd
```

Megmutatja az aktuális könyvtárat.

**English:**  
`pwd` shows the current directory.

---

## `ls`

Könyvtár tartalmának listázása.

```bash
ls
ls -l
ls -a
ls -la
```

- `-l` = **long listing**
- `-a` = **all**, rejtett fájlok is

---

## `cd`

Change directory.

```bash
cd directory
cd /etc
cd ~
cd ..
```

---

## `mkdir`

Make directory.

```bash
mkdir linux-lab
```

Többszintű könyvtárfa létrehozása:

```bash
mkdir -p ~/projects/linux/lab
```

A `-p` jelentése:

```text
parents
```

A szükséges szülőkönyvtárakat is létrehozza.

Például:

```text
projects
└── linux
    └── lab
```

akkor is létrejöhet, ha a `projects` és a `linux` még nem létezett.

**English:**  
`-p` creates parent directories if needed.

---

## `touch`

Fájl létrehozása vagy timestamp frissítése.

```bash
touch notes.txt
```

---

## `cp`

Copy.

Fájl másolása:

```bash
cp notes.txt notes-copy.txt
```

Másolás könyvtárba:

```bash
cp notes.txt docs/
```

Könyvtár másolásához:

```bash
cp -r docs backup/
```

A `-r` jelentése:

```text
recursive
```

A parancs végigmegy a teljes könyvtárfán és annak tartalmán.

**English:**  
`-r` copies directories recursively.

---

## `mv`

Move vagy rename.

Átnevezés:

```bash
mv notes.txt linux-notes.txt
```

Mozgatás:

```bash
mv linux-notes.txt docs/
```

Ugyanaz a parancs használható:

```text
rename
move
```

---

## `rm`

Remove.

Fájl törlése:

```bash
rm file.txt
```

Könyvtár törlése:

```bash
rm -r directory
```

A `-r` itt is **recursive**.

A shell `rm` parancsa nem egy Windows-szerű Lomtárba helyezi a fájlokat.

Ezért törlés előtt jó szokás:

```bash
pwd
ls
```

Majd csak ezután:

```bash
rm ...
```

**English:**  
Always check the path before deleting files.

---

## `cat`

Fájl tartalmának gyors kiírása.

```bash
cat file.txt
```

---

## `less`

Hosszabb fájl olvasása.

```bash
less file.txt
```

Kilépés:

```text
q
```

---

## `head`

A fájl elejének megjelenítése.

```bash
head file.txt
```

Alapból az első 10 sort mutatja.

Megadhatjuk a sorok számát:

```bash
head -n 5 file.txt
```

A `-n` jelentése:

```text
number of lines
```

---

## `tail`

A fájl végének megjelenítése.

```bash
tail file.txt
```

Alapból az utolsó 10 sort mutatja.

Például:

```bash
tail -n 5 file.txt
```

Folyamatos követés:

```bash
tail -f application.log
```

A `-f` jelentése:

```text
follow
```

Ez folyamatosan megmutatja a fájlhoz hozzáadott új sorokat.

Ez később logok figyelésénél nagyon fontos.

**English:**  
`tail -f` follows new log entries.

---

# 9. Output redirection — kimenetátirányítás

A shell képes egy parancs kimenetét fájlba irányítani.

## `>`

Felülírja a célfájlt.

```bash
echo "Linux basics" > notes.txt
```

Egyszerűen:

```text
stdout
  ↓
file
```

A fájl korábbi tartalma felülíródik.

**English:**  
`>` overwrites the file.

---

## `>>`

Hozzáfűz a fájlhoz.

```bash
echo "Filesystem practice" >> notes.txt
```

Ez nem törli a korábbi tartalmat.

**English:**  
`>>` appends to the file.

---

# 10. Relative és absolute path gyakorlati példa

Tegyük fel, hogy itt vagyunk:

```text
/home/user/linux-lab
```

és létezik:

```text
/home/user/linux-lab/docs/linux-notes.txt
```

Relatív útvonal:

```bash
cat docs/linux-notes.txt
```

Szintén relatív:

```bash
cat ./docs/linux-notes.txt
```

A `.` az aktuális könyvtár.

Abszolút útvonal:

```bash
cat /home/user/linux-lab/docs/linux-notes.txt
```

A `~` használatával:

```bash
cat ~/linux-lab/docs/linux-notes.txt
```

A `~`-t a shell a felhasználó home directory-jára bővíti ki.

---

# 11. Fontosabb flagek eddig

Nem kell minden flaget fejből tudni.

A gyakoriak használat közben rögzülnek.

| Flag | Jelentés | Példa |
|---|---|---|
| `-l` | long listing | `ls -l` |
| `-a` | all | `ls -a` |
| `-p` | parents | `mkdir -p` |
| `-r` | recursive | `cp -r`, `rm -r` |
| `-n` | number of lines | `head -n 5` |
| `-f` | follow | `tail -f` |

Egy parancs kapcsolóit így lehet megnézni:

```bash
command --help
```

Például:

```bash
mkdir --help
```

Részletes dokumentáció:

```bash
man mkdir
```

A `man` jelentése:

```text
manual
```

---

# 12. Alap mental model

```text
Linux distribution
      │
      ├── Linux kernel
      │
      ├── filesystem
      │
      ├── shell
      │
      ├── commands
      │
      └── applications
```

Egy shell parancs például:

```bash
cat /etc/hosts
```

egyszerűsítve:

```text
Bash
 ↓
cat program
 ↓
system calls
 ↓
Linux kernel
 ↓
filesystem
```

---

# 13. Alap angol szókincs

| English | Magyar |
|---|---|
| kernel | kernel / rendszermag |
| distribution / distro | disztribúció |
| filesystem | fájlrendszer |
| directory | könyvtár |
| file | fájl |
| path | útvonal |
| absolute path | abszolút útvonal |
| relative path | relatív útvonal |
| root directory | gyökérkönyvtár |
| current directory | aktuális könyvtár |
| parent directory | szülőkönyvtár |
| home directory | saját könyvtár |
| command | parancs |
| option / flag | kapcsoló / opció |
| configuration | konfiguráció |
| log | napló |
| process | folyamat |
| copy | másolás |
| move | mozgatás |
| rename | átnevezés |
| remove | törlés |
| recursive | rekurzív |
| overwrite | felülírás |
| append | hozzáfűzés |
| follow | követés |

---

# 14. B1 technikai mondatok

- The Linux kernel manages system resources.
- Ubuntu is a Linux distribution.
- My project is in my home directory.
- The configuration file is in `/etc`.
- Logs are often stored in `/var/log`.
- `pwd` shows the current directory.
- `ls` lists files and directories.
- `cd` changes the current directory.
- This is an absolute path.
- This path is relative to the current directory.
- `-p` creates parent directories if needed.
- `-r` copies directories recursively.
- `>` overwrites the file.
- `>>` appends to the file.
- `tail -f` follows new log entries.
- Always check the path before deleting files.