# 01 - Computer and Operating System Basics

## Computer

A számítógép alapvető feladatai:

- adat fogadása
- adat feldolgozása
- adat tárolása
- adat továbbítása

Egy számítógép ezekhez különböző hardverelemeket használ. Ezek működését és együttműködését az operációs rendszer kezeli.

---

# 1. Hardware

A **hardware**, magyarul hardver, a számítógép fizikai részeit jelenti.

Fontosabb komponensek:

- Motherboard
- CPU
- RAM
- Storage
- Network Interface
- GPU
- Power Supply Unit
- Perifériák

---

## 1.1 Motherboard

### Mi ez?

A **motherboard**, magyarul alaplap, a számítógép fő nyomtatott áramköri lapja.

A legtöbb fontos hardverelem közvetlenül vagy közvetve ehhez csatlakozik.

### Mire való?

Az alaplap biztosítja a különböző hardverkomponensek közötti kommunikáció fizikai infrastruktúráját.

Például:

- CPU
- RAM
- SSD
- GPU
- hálózati kártya
- USB-eszközök

mind kapcsolatban vannak az alaplappal.

Az alaplap az áramellátás elosztásában is részt vesz, de magát az elektromos energiát a tápegység, vagyis a PSU biztosítja.

### Hogyan működik?

Az alaplapon különböző csatlakozók és adatátviteli útvonalak találhatók.

Például:

```text
CPU ↔ RAM

CPU ↔ PCIe ↔ GPU

CPU ↔ PCIe ↔ SSD

Motherboard ↔ USB ↔ perifériák
```

---

## Bus

### Mi ez?

A **bus** olyan adatátviteli útvonal vagy kommunikációs rendszer, amelyen keresztül a számítógép hardverkomponensei adatot cserélnek egymással.

Modern számítógépekben például a **PCI Express (PCIe)** egy fontos nagysebességű kommunikációs rendszer.

Például:

```text
CPU
↓
PCIe
↓
GPU
```

vagy:

```text
CPU
↓
PCIe
↓
NVMe SSD
```

### Miért fontos?

A különböző buszrendszerek sebessége befolyásolhatja az egyes hardverelemek közötti adatátvitel sebességét.

---

## BIOS / UEFI

### Mi ez?

A BIOS és az UEFI olyan firmware, amely a számítógép indulásának legelső lépéseit végzi.

Régebbi számítógépeken jellemzően **BIOS** működött, modern rendszereken inkább **UEFI** használatos.

### Mire való?

Feladata többek között:

- hardver inicializálása
- alapvető hardverellenőrzés
- boot folyamat elindítása
- boot eszköz kiválasztása
- bizonyos hardverbeállítások kezelése

### Alaplapi elem

Az alaplapon található elem nem magát a BIOS vagy UEFI firmware-t tárolja.

Az elem főleg:

- a rendszeróra működését
- bizonyos firmware-beállítások megőrzését

segíti akkor is, amikor a számítógép nincs áram alatt.

### Miért fontos System/DevOps szempontból?

Szervereknél találkozhatunk például:

- boot sorrend beállítással
- virtualizáció engedélyezésével
- hardverfelismerési problémákkal
- firmware-beállításokkal
- storage konfigurációval
- hálózati boottal

---

# 1.2 CPU - Central Processing Unit

## Mi ez?

A **CPU - Central Processing Unit** a számítógép központi feldolgozóegysége.

Fő feladata a programutasítások végrehajtása és a számítások elvégzése.

## Mire való?

A CPU hajtja végre például az olyan műveleteket, mint:

- számok összeadása
- értékek összehasonlítása
- memória olvasása
- memória írása
- logikai műveletek
- programutasítások végrehajtása

---

## Hogyan működik?

Nagyon leegyszerűsített modell:

```text
Fetch
↓
Decode
↓
Execute
```

### Fetch

A CPU lekéri a következő végrehajtandó utasítást.

### Decode

A CPU értelmezi, hogy az utasítás mit jelent.

### Execute

A CPU végrehajtja az utasítást.

Ez a folyamat rendkívül gyorsan és folyamatosan ismétlődik.

---

# 1.3 CPU Core

## Mi ez?

A **core**, magyarul processzormag, a CPU egyik fizikai végrehajtó egysége.

Egy modern CPU több magot is tartalmazhat.

Például:

```text
CPU
├── Core 1
├── Core 2
├── Core 3
└── Core 4
```

## Mire való?

Több mag segítségével több feladat valóban párhuzamosan is végrehajtható.

Nagyon leegyszerűsített példa:

```text
Core 1 → Browser
Core 2 → Antivirus
Core 3 → Node.js process
Core 4 → másik process
```

A valóság ennél összetettebb.

Az operációs rendszer **scheduler** nevű része folyamatosan eldönti, hogy mely thread melyik CPU-magon és mikor fusson.

## Miért fontos System/DevOps szempontból?

A CPU-magok száma befolyásolhatja:

- párhuzamos feladatok számát
- szerver kapacitását
- alkalmazások teljesítményét
- virtuális gépek számát
- konténerek teljesítményét

---

# 1.4 Thread

A thread fogalmának két fontos jelentése van:

- Software Thread
- Hardware Thread

---

## Software Thread

### Mi ez?

A software thread egy processen belüli végrehajtási szál.

Egy process több threadet is tartalmazhat.

```text
Process
├── Thread 1
├── Thread 2
└── Thread 3
```

A threadek ugyanazon process bizonyos erőforrásain osztoznak.

---

## Hardware Thread

### Mi ez?

A hardware thread a CPU egy logikai végrehajtási egysége.

Egy CPU specifikációja lehet például:

```text
8 physical cores
16 hardware threads
```

Ez nem azt jelenti, hogy 16 fizikai CPU-mag található benne.

Egy fizikai mag több logikai végrehajtási szálat is kezelhet, amelyek bizonyos hardvererőforrásokon osztoznak.

## Miért fontos?

System Engineer és DevOps területen gyakran találkozhatunk:

- CPU core
- vCPU
- thread
- process
- parallelism

fogalmakkal.

---

# 1.5 Clock Speed

## Mi ez?

A CPU működését órajel szinkronizálja.

Mértékegysége lehet:

```text
Hz
MHz
GHz
```

Például:

```text
3.5 GHz
```

Az 1 GHz körülbelül egymilliárd órajelciklust jelent másodpercenként.

## Fontos

A magasabb GHz önmagában nem jelenti azt, hogy egy CPU minden esetben gyorsabb.

A teljesítményt több tényező is befolyásolja:

- CPU architektúra
- CPU-magok száma
- cache mérete
- memória sebessége
- utasítások hatékonysága
- workload típusa

---

# 1.6 Instruction

## Mi ez?

Az **instruction** olyan gépi utasítás, amelyet a CPU végre tud hajtani.

Például:

- adat mozgatása
- összeadás
- kivonás
- összehasonlítás
- memória olvasása
- memória írása
- ugrás másik utasításra

A magas szintű programkód végül olyan formára kerül, amelyből a CPU végrehajtható gépi utasításokat kap.

---

# 1.7 CPU Registers

## Mi ez?

A **register** a CPU belsejében található nagyon kis méretű, rendkívül gyors memória.

## Mire való?

A CPU a közvetlenül használt adatokat, címeket és köztes eredményeket registerekben tárolhatja.

Nagyon leegyszerűsített példa:

```text
Register A = 5
Register B = 7

CPU:
A + B = 12
```

## Hogyan működik?

A register közvetlenül a CPU része, ezért a CPU rendkívül gyorsan hozzá tud férni.

## Miért fontos?

A registerek a memóriahierarchia leggyorsabb szintjét jelentik.

---

# 1.8 CPU Cache

## Mi ez?

A **cache** egy nagyon gyors memória a CPU-ban vagy annak közvetlen közelében.

## Mire való?

A CPU sokkal gyorsabb, mint a RAM.

Ha a CPU minden adatért közvetlenül a RAM-hoz fordulna, sok időt töltene várakozással.

A cache ezért olyan adatokat és utasításokat tart közel a CPU-hoz, amelyekre valószínűleg hamarosan szükség lesz.

---

## L1 Cache

Az L1 cache:

- nagyon gyors
- nagyon kis méretű
- jellemzően CPU-magonként található

Gyakran külön cache található:

- adatok számára
- utasítások számára

---

## L2 Cache

Az L2 cache:

- nagyobb, mint az L1
- valamivel lassabb
- sok processzornál magonként külön található

---

## L3 Cache

Az L3 cache:

- nagyobb, mint az L2
- lassabb, mint az L1 és L2
- gyakran több CPU-mag közösen használja

Például:

```text
        L3 Cache
       /    |    \
   Core1  Core2  Core3
```

---

# 1.9 Memory Hierarchy

A számítógép különböző sebességű és méretű memóriatípusokat használ.

Nagyon leegyszerűsített memóriahierarchia:

```text
FASTEST / SMALLEST

Registers
↓
L1 Cache
↓
L2 Cache
↓
L3 Cache
↓
RAM
↓
SSD
↓
HDD

SLOWEST / LARGEST
```

Általános szabály:

> Minél közelebb van egy memória a CPU-hoz, annál gyorsabb, de általában annál kisebb és drágább.

---

## Cache Hit

Cache hit történik, amikor a CPU által keresett adat már megtalálható a cache-ben.

Ilyenkor az adat gyorsan elérhető.

---

## Cache Miss

Cache miss történik, amikor az adat nincs az adott cache-szinten.

Ilyenkor a rendszer egy lassabb memória-szinten keresi tovább.

Például:

```text
L1 miss
↓
L2
↓
L3
↓
RAM
```

---

# 1.10 RAM - Random Access Memory

## Mi ez?

A **RAM - Random Access Memory** a számítógép fő munkamemóriája.

## Mire való?

A RAM-ban találhatók többek között:

- futó programok
- programkódok aktuálisan használt részei
- változók
- processzek adatai
- ideiglenes adatok
- bufferek
- különböző cache-ek

Egy program indítása nagyon leegyszerűsítve:

```text
Program az SSD-n
↓
operációs rendszer betölti
↓
RAM
↓
CPU végrehajtja
```

## Fontos pontosítás

A CPU nem a RAM-ban végzi a számításokat.

A számításokat a CPU végzi.

A RAM a CPU számára gyorsan elérhető munkaterületet biztosít.

---

## Volatile Memory

A RAM **volatile memory**.

Ez azt jelenti, hogy áramellátás nélkül az adatai elvesznek.

```text
Power Off
↓
RAM contents lost
```

## Miért fontos System/DevOps szempontból?

Szervereknél gyakran vizsgáljuk:

- összes RAM
- használt RAM
- szabad RAM
- processzenkénti memóriahasználat
- memóriahiány
- swap használat

---

# 1.11 Storage

## Mi ez?

A **storage** hosszú távú adattárolásra szolgáló háttértár.

Példák:

- HDD
- SATA SSD
- NVMe SSD

## Mire való?

Itt tárolódnak például:

- operációs rendszer fájljai
- programok
- dokumentumok
- adatbázisok
- logok
- backupok

---

## Non-volatile Storage

A háttértár **non-volatile**.

Ez azt jelenti, hogy áramellátás nélkül is megtartja az adatokat.

---

## HDD - Hard Disk Drive

A HDD mágneses adattároló.

Mechanikus, mozgó alkatrészeket tartalmaz.

Jellemzői:

- nagy kapacitás
- viszonylag alacsony ár
- SSD-nél lassabb
- mechanikus alkatrészeket tartalmaz

---

## SSD - Solid State Drive

Az SSD nem tartalmaz mozgó alkatrészeket.

Jellemzői:

- HDD-nél gyorsabb
- alacsonyabb hozzáférési idő
- halk működés
- jobb mechanikai ellenállás

---

## NVMe SSD

Az NVMe SSD tipikusan PCI Express kapcsolaton keresztül kommunikál.

Általában nagyobb adatátviteli sebesség és kisebb késleltetés érhető el vele, mint hagyományos SATA SSD esetén.

---

## Miért fontos System/DevOps szempontból?

Szervereken fontos lehet:

- tárhely mérete
- disk usage
- olvasási sebesség
- írási sebesség
- IOPS
- latency
- filesystem
- backup
- storage failure

---

# 1.12 RAM és Storage összehasonlítása

| Tulajdonság | RAM | SSD / HDD |
|---|---|---|
| Feladat | ideiglenes munkamemória | tartós adattárolás |
| Sebesség | nagyon gyors | lassabb |
| Áram nélkül | adat elveszik | adat megmarad |
| Tipikus tartalom | futó programok, változók | fájlok, programok, adatbázisok |
| Típus | volatile | non-volatile |

Egyszerű modell:

```text
Storage
↓
Program betöltése
↓
RAM
↓
CPU
```

---

# 1.13 Network Interface

## NIC - Network Interface Card

### Mi ez?

A **NIC - Network Interface Card** a számítógép hálózati interfésze.

Lehet:

- fizikai
- virtuális

### Mire való?

Lehetővé teszi, hogy a számítógép hálózaton keresztül más eszközökkel kommunikáljon.

Például:

```text
Computer
↓
NIC
↓
Ethernet / Wi-Fi
↓
Network
```

### Fontos fogalmak

- Ethernet
- Wi-Fi
- MAC address
- network interface

Ezeket később a networking fejezetben részletesen tanuljuk.

### Miért fontos System/DevOps szempontból?

Szinte minden modern szerver hálózaton kommunikál.

Később vizsgálni fogunk többek között:

- network interface-eket
- IP-címeket
- routingot
- DNS-t
- portokat
- hálózati kapcsolatokat

---

# 1.14 GPU - Graphics Processing Unit

## Mi ez?

A **GPU - Graphics Processing Unit** eredetileg elsősorban grafikai számításokra tervezett feldolgozóegység.

## Mire való?

A GPU nagyon sok egyszerűbb számítást képes párhuzamosan végrehajtani.
GPU = a videókártya "processzora"
Videókártya = a teljes hardveregység

Felhasználási területek:

- grafika
- videó
- 3D
- tudományos számítás
- machine learning
- AI

## Miért fontos System/DevOps szempontból?

Normál webes infrastruktúránál a GPU nem feltétlenül fontos.

AI, machine learning és nagy számításigényű rendszereknél azonban GPU-s szervereket is üzemeltethetünk.

---

# 1.15 Power Supply Unit - PSU

## Mi ez?

A **PSU - Power Supply Unit** a számítógép tápegysége.

## Mire való?

A hálózati váltóáramot a számítógép alkatrészei számára megfelelő egyenárammá alakítja.

Energiával látja el többek között:

- alaplapot
- CPU-t
- GPU-t
- háttértárakat
- ventilátorokat
- egyéb belső hardvereket

Nagyon leegyszerűsítve:

```text
230 V AC
↓
PSU
↓
DC feszültségek
↓
Motherboard / CPU / GPU / Storage
```

## Miért fontos System/DevOps szempontból?

Szerveres környezetben találkozhatunk:

- redundáns tápegységekkel
- energiafogyasztással
- UPS-sel
- áramkimaradásokkal
- hardverhibákkal

---

# 1.16 Perifériák

## Mi ez?

A perifériák a számítógéphez csatlakozó külső vagy kiegészítő be- és kimeneti eszközök.

Példák:

- billentyűzet
- egér
- monitor
- nyomtató
- mikrofon
- hangszóró
- külső adattároló

## Mire valók?

Lehetővé teszik a számítógép és a felhasználó vagy más külső eszköz közötti adatcserét.

---

# 2. Firmware

## Mi ez?

A **firmware** olyan alacsony szintű szoftver, amely közvetlenül egy hardvereszköz működését vezérli.

Nem ugyanaz, mint egy hagyományos felhasználói alkalmazás.

## Példák

- BIOS / UEFI
- SSD firmware
- hálózati kártya firmware
- router firmware

## Mire való?

A firmware biztosítja egy adott hardver alapvető működését.

## Miért fontos System/DevOps szempontból?

Szervereken firmware-frissítésre lehet szükség például:

- biztonsági hibák javításához
- stabilitási hibák javításához
- hardverkompatibilitás javításához
- teljesítményproblémák megoldásához

---

# 3. Boot Process

## Mi ez?

A **boot process** a számítógép elindulásának folyamata a bekapcsolástól az operációs rendszer használható állapotáig.

## Hogyan működik?

Nagyon leegyszerűsítve:

```text
Power On
↓
BIOS / UEFI
↓
Hardware initialization
↓
Boot device kiválasztása
↓
Bootloader
↓
Kernel betöltése
↓
Operating System indul
↓
Services indulnak
↓
Login
```

Linux esetén például:

```text
UEFI
↓
GRUB
↓
Linux Kernel
↓
systemd
↓
Services
↓
Login
```

---

## Bootloader

### Mi ez?

A bootloader olyan program, amely segít betölteni az operációs rendszer kernelét.

Linux rendszereken gyakori bootloader:

```text
GRUB
```

## Miért fontos System/DevOps szempontból?

Boot problémák esetén fontos felismerni, hogy melyik szinten akad el a rendszer:

- firmware
- bootloader
- kernel
- filesystem
- service-ek indulása

---

# 4. Device Driver (illesztőprogram)

## Mi ez?

A **device driver** olyan szoftverkomponens, amely lehetővé teszi, hogy az operációs rendszer kommunikáljon egy adott hardvereszközzel.

## Mire való?

Például:

```text
Operating System
↓
Network Driver
↓
Network Card
```

vagy:

```text
Operating System
↓
GPU Driver
↓
GPU
```

## Miért van rá szükség?

Az operációs rendszer nem ismeri automatikusan minden hardver pontos működését.

A driver biztosítja az operációs rendszer és a hardver közötti szükséges kommunikációs réteget.

## Miért fontos System/DevOps szempontból?

Driver problémák okozhatnak például:

- hálózati hibát
- storage hibát
- GPU problémát
- teljesítményproblémát
- hardverfelismerési problémát

---

# 5. Filesystem - Bevezetés

## Mi ez?

A **filesystem**, magyarul fájlrendszer, meghatározza, hogyan tárolja és szervezi az operációs rendszer a fájlokat egy háttértáron.

## Mire való?

A fájlrendszer kezeli például:

- fájlokat
- könyvtárakat
- fájlneveket
- fájlok helyét
- jogosultságokat
- metadata adatokat

## Példák

Windows rendszereken:

```text
NTFS
FAT32
exFAT
```

Linux rendszereken:

```text
ext4
XFS
Btrfs
```

## Miért fontos System/DevOps szempontból?

Szervereken gyakran dolgozunk:

- tárhellyel
- mountokkal
- fájljogosultságokkal
- logokkal
- adatbázisfájlokkal
- backupokkal

A fájlrendszereket később részletesen is tanuljuk.

---

# 6. Operating System

## Mi ez?

Az **Operating System - OS**, magyarul operációs rendszer olyan rendszerszoftver, amely:

- kezeli a számítógép hardveres erőforrásait
- szolgáltatásokat biztosít az alkalmazások számára
- kezeli a programok futását
- kezeli a felhasználókat és jogosultságokat

Példák:

- Windows
- Linux-alapú operációs rendszerek
- macOS

## Fontos

Az operációs rendszer nem egyszerűen grafikus felhasználói felület.

Egy szerver operációs rendszer grafikus felület nélkül is teljes értékűen működhet.

---

## Mire való?

Az operációs rendszer kezeli többek között:

- CPU használatát
- memóriát
- processzeket
- fájlrendszereket
- hardvereszközöket
- hálózatot
- felhasználókat
- jogosultságokat

---

## Egyszerű modell

```text
Applications
↓
Operating System
↓
Kernel
↓
Hardware
```

Az alkalmazásoknak nem kell minden hardverelemet közvetlenül vezérelniük.

Az operációs rendszer közös szolgáltatásokat biztosít számukra.

---

## Miért fontos System/DevOps szempontból?

A legtöbb infrastruktúra valamilyen operációs rendszeren fut.

System Engineer vagy DevOps Engineer munkában gyakran foglalkozunk:

- Linux szerverekkel
- processzekkel
- service-ekkel
- felhasználókkal
- jogosultságokkal
- logokkal
- networkinggel
- filesystemekkel
- CPU használattal
- memóriahasználattal

---

# 7. Kernel

## Mi ez?

A **kernel**, magyarul rendszermag, az operációs rendszer központi, alapvető része.

A kernel közvetlenül kezeli a rendszer legfontosabb erőforrásait.

## Mire való?

A kernel főbb feladatai:

- CPU kezelése
- processzek kezelése
- memória kezelése
- hardvereszközök kezelése
- device driverek kezelése
- fájlrendszerek kezelése
- hálózati kommunikáció kezelése
- rendszererőforrások hozzáférésének szabályozása

---

## Hogyan működik?

Nagyon leegyszerűsítve:

```text
Applications
↓
System Calls
↓
Kernel
↓
Hardware
```

Egy alkalmazás általában nem közvetlenül vezérli a hardvert.

Például ha egy program fájlt akar megnyitni, a kerneltől kér szolgáltatást.

---

# 8. System Call

## Mi ez?

A **system call** olyan mechanizmus, amelyen keresztül egy program szolgáltatást kérhet a kerneltől.

Például:

- fájl megnyitása
- memória foglalása
- process létrehozása
- hálózati kapcsolat létrehozása

Nagyon leegyszerűsítve:

```text
Application
↓
System Call
↓
Kernel
↓
Hardware / Resource
```

---

# 9. User Space és Kernel Space

A modern operációs rendszerek elkülönítik a normál alkalmazásokat a kernel működésétől.

## User Space

A normál alkalmazások általában user space-ben futnak.

Például:

- Browser
- Node.js
- Nginx
- Shell
- különböző programok

## Kernel Space

A kernel privilegizált környezetben fut.

Nagyon leegyszerűsítve:

```text
USER SPACE
├── Browser
├── Node.js
├── Nginx
└── Shell

──────────────

KERNEL SPACE
└── Operating System Kernel

──────────────

HARDWARE
```

A kernel közvetlenül hozzáférhet a rendszer legfontosabb erőforrásaihoz.

Ez fontos:

- biztonsági
- stabilitási
- jogosultsági

szempontból.

---

# 10. Hogyan indul el egy program?

Tegyük fel, hogy elindítunk egy alkalmazást.

Nagyon leegyszerűsítve:

```text
Program file az SSD-n
↓
Operációs rendszer betölti a szükséges részeket
↓
RAM
↓
Process létrejön
↓
CPU végrehajtja az utasításokat
```

A program tehát nem egyszerűen közvetlenül az SSD-ről fut.

A háttértáron lévő programkód szükséges részei bekerülnek a memóriába, majd az operációs rendszer létrehozza a futó processt.

---

# 11. Virtualization - Bevezetés

## Mi ez?

A **virtualization**, magyarul virtualizáció lehetővé teszi, hogy egy fizikai számítógépen több elkülönített virtuális számítógép fusson.

Ezeket:

```text
Virtual Machine
VM
```

néven nevezzük.

## Egyszerű modell

```text
Physical Hardware
↓
Hypervisor
├── Virtual Machine 1
│   └── Guest OS
│
├── Virtual Machine 2
│   └── Guest OS
│
└── Virtual Machine 3
    └── Guest OS
```

---

## Hypervisor

A **hypervisor** kezeli a virtuális gépeket és elosztja közöttük a fizikai erőforrásokat.

Például:

- CPU
- RAM
- storage
- network

## Miért fontos System/DevOps szempontból?

A cloud infrastruktúrák jelentős része virtualizációra épül.

Egy cloud szerver gyakran valójában egy virtuális gép, amely egy nagyobb fizikai szerveren fut.

---

# 12. WSL2

## Mi ez?

WSL jelentése:

```text
Windows Subsystem for Linux
```

A WSL lehetővé teszi Linux környezet futtatását Windows alatt.

Mi **WSL2-t** használunk.

## Hogyan működik?

A WSL2 valódi Linux kernelt futtat egy könnyű virtualizált környezetben.

Nagyon leegyszerűsítve:

```text
Windows 11
↓
WSL2
↓
Linux Kernel
↓
Ubuntu
↓
Bash / Git / Linux Tools
```

## Mire való?

WSL2 segítségével Windows gépen használhatunk Linuxos eszközöket.

Például:

```text
git
ssh
ps
systemctl
grep
find
curl
```

Később pedig:

```text
Docker
Terraform
Linux networking tools
```

## Miért fontos számunkra?

A tanulási projektjeink nagy részét WSL2 Ubuntu alatt fogjuk kezelni.

Így olyan környezetben dolgozunk, amely sok szempontból hasonlít egy valódi Linux szerverhez.

---

# 13. Összefoglaló rendszerkép

Az eddigieket nagyon leegyszerűsítve így rakhatjuk össze:

```text
                USER / APPLICATIONS
                       ↓
                Operating System
                       ↓
                    Kernel
                       ↓
            ┌──────────┼──────────┐
            ↓          ↓          ↓
           CPU        RAM       Storage
            ↓                     ↓
         Cache                  Filesystem
            ↓
       NIC / GPU / Devices
```

Egy program futása nagyjából:

```text
SSD
↓
Filesystem
↓
Operating System
↓
RAM
↓
Process
↓
CPU
↓
Registers / Cache
```

Hálózati kommunikáció esetén:

```text
Application
↓
Operating System
↓
Kernel
↓
Network Interface
↓
Network
```

---

# 14. System / DevOps szempontból fontos alapmodell

Egy System Engineer vagy DevOps Engineer gyakran ilyen problémákkal találkozik:

```text
CPU usage high
RAM usage high
Disk full
Disk I/O slow
Network connection failed
Process stopped
Service unavailable
Filesystem problem
Driver problem
Boot problem
```

Ezért fontos érteni, hogy a rendszer különböző rétegei hogyan kapcsolódnak egymáshoz:

```text
Application
↓
Operating System
↓
Kernel
↓
CPU / RAM / Storage / Network
↓
Physical or Virtual Hardware
```

A későbbi Linux, networking, Docker, cloud és Kubernetes témák mind erre az alapmodellre fognak épülni.

# 15. Terminal, Shell és CLI

## 15.1 Terminal

### Mi ez?

A **terminal** egy olyan alkalmazás vagy felület, amelyen keresztül parancssoros környezetet használhatunk.

A terminal önmagában általában nem értelmezi a parancsokat.

Inkább egy olyan felület, amelyben egy shell fut.

Példák:

- Windows Terminal
- GNOME Terminal
- Konsole
- macOS Terminal

### Egyszerű modell

```text
Terminal
↓
Shell
↓
Commands
```

Például WSL használatakor:

```text
Windows Terminal
↓
Ubuntu / WSL
↓
Bash
↓
Linux commands
```

### Miért fontos System/DevOps szempontból?

Szerveres környezetben nagyon gyakran terminálon keresztül dolgozunk.

Például:

```bash
ssh user@server
```

Ezzel távolról beléphetünk egy Linux szerverre, és terminálon keresztül kezelhetjük.

---

# 15.2 Shell

## Mi ez?

A **shell** egy parancsértelmező program.

Feladata, hogy a felhasználó által beírt parancsokat értelmezze, majd elindítsa a megfelelő programokat vagy műveleteket.

Példák Linuxon:

```text
bash
zsh
fish
sh
```

Windows alatt például:

```text
PowerShell
cmd
```

## Mire való?

A shell segítségével:

- programokat indíthatunk
- fájlokat kezelhetünk
- könyvtárak között mozoghatunk
- processzeket kezelhetünk
- rendszerinformációkat kérdezhetünk le
- hálózati eszközöket használhatunk
- scriptet futtathatunk
- automatizálhatunk feladatokat

Például:

```bash
pwd
```

megmutatja, melyik könyvtárban vagyunk.

```bash
ls
```

kilistázza a könyvtár tartalmát.

```bash
cd projects
```

belép a `projects` könyvtárba.

---

## Hogyan működik?

Nagyon leegyszerűsítve:

```text
User
↓
Shell
↓
Command értelmezése
↓
Program / System Call
↓
Operating System
```

Például:

```bash
ls
```

esetén a shell:

1. értelmezi a parancsot
2. megkeresi az `ls` programot
3. elindítja azt
4. az operációs rendszer végrehajtja
5. az eredmény megjelenik a terminalban

---

## Bash

A **Bash - Bourne Again Shell** az egyik legelterjedtebb Linux shell.

Mi a WSL Ubuntu környezetben főként Bash-t fogunk használni.

Például:

```bash
echo "Hello"
```

vagy:

```bash
mkdir project
```

---

## Shell és Terminal különbsége

A két fogalom nem ugyanaz.

```text
Terminal = a felület / alkalmazás
Shell = a parancsokat értelmező program
```

Példa:

```text
Windows Terminal
↓
Bash
```

Ugyanabban a terminalban akár más shell is futhatna:

```text
Windows Terminal
├── PowerShell
├── Bash
└── más shell
```

---

# 15.3 CLI - Command Line Interface

## Mi ez?

A **CLI - Command Line Interface** magyarul parancssoros felhasználói felület.

Ez egy olyan interfész, ahol a felhasználó szöveges parancsokkal kommunikál a rendszerrel vagy egy programmal.

Például:

```bash
git status
```

vagy:

```bash
docker ps
```

vagy:

```bash
systemctl status nginx
```

A CLI nem feltétlenül az egész operációs rendszer parancssoros környezete.

Egy külön programnak is lehet saját CLI-je.

Például:

```text
Git CLI
Docker CLI
AWS CLI
Azure CLI
Terraform CLI
```

---

## Mire való?

A CLI segítségével:

- parancsokat adhatunk ki
- programokat konfigurálhatunk
- rendszerállapotot kérdezhetünk le
- automatizálhatunk
- scriptet írhatunk
- távoli szervereket kezelhetünk

---

# 15.4 GUI - Graphical User Interface

## Mi ez?

A **GUI - Graphical User Interface** grafikus felhasználói felület.

A felhasználó grafikus elemek segítségével kommunikál a programmal.

Például:

- ablakok
- ikonok
- gombok
- menük
- listák
- grafikus beállítási felületek

Példák:

- Windows asztal
- VS Code grafikus felülete
- webböngésző
- File Explorer

---

# 15.5 CLI és GUI összehasonlítása

| Tulajdonság | CLI | GUI |
|---|---|---|
| Vezérlés | szöveges parancsok | grafikus elemek |
| Használat | billentyűzet-központú | egér + billentyűzet |
| Automatizálás | nagyon jó | korlátozottabb |
| Távoli kezelés | nagyon hatékony | gyakran nehezebb |
| Erőforrásigény | általában alacsony | általában magasabb |
| Kezdőknek | nehezebb lehet | gyakran egyszerűbb |
| Scriptelhetőség | kiváló | általában gyengébb |

---

# 15.6 Miért használunk sok CLI-t System/DevOps munkában?

A parancssoros eszközök több fontos előnnyel rendelkeznek.

## Automatizálható

Egy parancs scriptbe írható.

Például:

```bash
sudo apt update
sudo apt upgrade
```

Ezek később automatizálhatók.

---

## Ismételhető

Ugyanazt a parancsot több gépen is végre lehet hajtani.

Ez fontos, ha nem egyetlen szervert, hanem sok rendszert kezelünk.

---

## Dokumentálható

Egy parancs pontosan leírható.

Például:

```bash
systemctl restart nginx
```

Sokkal egyértelműbb, mint:

> kattints ide, majd arra a gombra, majd arra a menüre

---

## Távoli szervereken is használható

Egy Linux szervernek gyakran nincs grafikus felülete.

Ilyenkor SSH-n keresztül lépünk be:

```bash
ssh user@server
```

majd CLI-n keresztül kezeljük.

---

## Kevesebb erőforrást igényel

Egy szerveren gyakran nincs szükség grafikus asztali környezetre.

Ezért egy tipikus Linux szerver lehet:

```text
Linux
├── SSH
├── Nginx
├── Docker
├── Database
└── Monitoring
```

grafikus felület nélkül.

---

# 15.7 Egy teljes példa

Tegyük fel, hogy WSL Ubuntu alatt dolgozunk.

```text
Windows Terminal
↓
WSL2 Ubuntu
↓
Bash Shell
↓
CLI command
↓
Linux Operating System
↓
Kernel
↓
Hardware / Resource
```

Például:

```bash
ls
```

A folyamat nagyon leegyszerűsítve:

```text
User
↓
beírja: ls
↓
Bash értelmezi
↓
elindítja az ls programot
↓
Operating System / Kernel
↓
filesystem információ
↓
eredmény vissza a terminalba
```

---

# 15.8 Fontos különbségek

```text
Terminal
=
az alkalmazás vagy felület, ahol a shell fut
```

```text
Shell
=
a parancsokat értelmező program
```

```text
CLI
=
parancssoros felhasználói interfész
```

```text
GUI
=
grafikus felhasználói interfész
```

Egyszerűen:

```text
Terminal
↓
Shell
↓
CLI commands
```

---

# 15.9 Miért fontos System/DevOps szempontból?

System Engineer és DevOps munkában folyamatosan használunk parancssoros eszközöket.

Például:

```text
Linux CLI
Git CLI
Docker CLI
Terraform CLI
Cloud CLI
Kubernetes CLI
```

Később olyan parancsokat fogunk használni, mint:

```bash
ps
```

```bash
top
```

```bash
systemctl
```

```bash
journalctl
```

```bash
ip
```

```bash
ss
```

```bash
curl
```

```bash
git
```

```bash
docker
```

```bash
terraform
```

```bash
kubectl
```

Ezért a shell és a CLI magabiztos használata a System / DevOps / Platform Engineer egyik alapvető készsége.


# Process, Thread, Memory és System Call alapok

## 1. Program és process

A **program** egy passzív kód vagy végrehajtható állomány a háttértáron.

Például:

```text
node
nginx
postgres
chrome
```

Amikor egy programot elindítunk, az operációs rendszer létrehoz egy:

```text
process
```

folyamatot.

Egyszerű modell:

```text
Program
↓
elindítás
↓
Process
```

A process egy futó program példánya.

---

# 2. Mi tartozik egy processhez?

Egy processhez többek között tartozhat:

```text
Process
│
├── PID
├── programkód
├── memória
├── stack
├── heap
├── nyitott fájlok
├── CPU állapot
├── jogosultságok
└── threadek
```

---

# 3. PID

A **PID = Process ID**.

Az operációs rendszer minden processhez azonosítót rendel.

Példa:

```text
PID 1200 → nginx
PID 1300 → node
PID 1500 → postgres
```

Az OS ezzel tudja megkülönböztetni a futó processeket.

---

# 4. Egy programból több process

Egy programból több process is futhat.

Például:

```text
Chrome program

↓
Chrome process
Chrome process
Chrome process
Chrome process
```

Tehát:

```text
PROGRAM
=
kód / executable
```

```text
PROCESS
=
annak egy futó példánya
```

---

# 5. Thread

A **thread** egy processen belüli végrehajtási szál.

Példa:

```text
Process
│
├── Thread 1
├── Thread 2
├── Thread 3
└── Thread 4
```

A process biztosítja az erőforrásokat és a memóriateret.

A thread a végrehajtási egység.

---

# 6. Process és thread hasonlat

Éttermis hasonlat:

```text
Process
=
étterem
```

Az étteremnek vannak:

```text
helyiségei
eszközei
készletei
```

A threadek:

```text
szakácsok
```

Egy étteremben több szakács dolgozhat.

Hasonlóan:

```text
1 process
+
több thread
```

is létezhet.

---

# 7. Miért vannak threadek?

Egy alkalmazás egyszerre több feladatot kezelhet.

Példa:

```text
Process

Thread 1 → UI
Thread 2 → network
Thread 3 → calculation
```

Ez lehetővé teszi, hogy több tevékenység haladjon egymás mellett.

---

# 8. Process isolation

Külön processzek általában külön memóriaterületet kapnak.

Példa:

```text
Node process
Memory A

PostgreSQL process
Memory B

Nginx process
Memory C
```

Egy process nem írhat csak úgy egy másik process memóriájába.

Ez fontos:

```text
stabilitás
biztonság
hibaelkülönítés
```

szempontjából.

---

# 9. Processzek kommunikációja

Külön processzeknek kommunikációs mechanizmusokra van szükségük.

Ezeket összefoglalóan:

```text
IPC
=
Inter-Process Communication
```

néven ismerjük.

Példák:

```text
pipes
sockets
shared memory
signals
```

DevOps szempontból nagyon fontos:

```text
socket
```

Példa:

```text
Node process
↓
TCP socket
↓
PostgreSQL process
```

---

# 10. Scheduler

Tegyük fel:

```text
Thread A
Thread B
Thread C
Thread D
Thread E
```

de csak néhány CPU core áll rendelkezésre.

Az operációs rendszer schedulerének feladata eldönteni:

```text
melyik futtatható thread
mikor kapjon CPU-időt
```

Egyszerű modell:

```text
Threads
↓
Scheduler
↓
CPU cores
```

---

# 11. Multitasking

A CPU és az operációs rendszer nagyon gyorsan váltogat a futtatandó feladatok között.

Például:

```text
Thread A
↓
Thread B
↓
Thread C
↓
Thread A
↓
Thread D
```

Ezért úgy érzékeljük, hogy sok program egyszerre fut.

---

# 12. Context switch

Ha a CPU egyik threadről másik threadre vált:

```text
Thread A
↓
Thread B
```

az:

```text
context switch
```

Az operációs rendszernek el kell mentenie az egyik thread CPU-állapotát és vissza kell töltenie a másikét.

Egyszerűen:

```text
A fut
↓
A állapot mentése
↓
B állapot betöltése
↓
B fut
```

A context switchnek van teljesítményköltsége.

---

# 13. RAM

A **RAM** a számítógép gyors, ideiglenes munkamemóriája.

Amikor egy program fut:

```text
Storage
↓
program
↓
Process
↓
RAM használat
```

A futó processzek memóriát használnak.

---

# 14. Process memória

Egy process memóriafelépítése leegyszerűsítve:

```text
Process Memory
│
├── Code
├── Data
├── Heap
└── Stack
```

A process számára úgy tűnik, mintha saját memóriaterülete lenne.

---

# 15. Virtual memory

A process által használt címek általában:

```text
virtual addresses
```

Nem közvetlen fizikai RAM-címek.

Egyszerű modell:

```text
Process
↓
Virtual Address
↓
Memory Management
↓
Physical RAM
```

---

# 16. Miért fontos a virtual memory?

A virtual memory többek között:

```text
process isolation
egyszerűbb memory management
nagy logikai címtér
```

biztosítását segíti.

Példa:

```text
Process A
Virtual Memory A

Process B
Virtual Memory B
```

Mindkettő saját címtartományt érzékel.

---

# 17. Memory page

A memóriát az OS kisebb egységekre osztja.

Ezek:

```text
pages
```

Példa:

```text
Virtual Page 1
Virtual Page 2
Virtual Page 3
```

A fizikai RAM oldalaihoz / frame-jeihez rendelhetők.

Egyszerű modell:

```text
Virtual Page
↓
Page Table
↓
Physical Memory
```

---

# 18. Stack

A **stack** főleg a függvényhívások futási állapotához kapcsolódik.

Példa:

```text
main()
↓
functionA()
↓
functionB()
```

Stack:

```text
functionB
functionA
main
```

A legutoljára bekerült elem távozik először.

Ez:

```text
LIFO
=
Last In, First Out
```

---

# 19. Heap

A **heap** dinamikusan lefoglalt memória.

Itt lehetnek például:

```text
objectek
array-ek
dinamikus adatstruktúrák
```

Egyszerű különbség:

```text
Stack
=
függvényhívásokhoz kapcsolódó futási állapot
```

```text
Heap
=
dinamikusan létrejövő adatok
```

A pontos működés a nyelvtől és runtime-tól is függ.

---

# 20. Memory leak

Ha egy program memóriát foglal, de azt többé nem tudja megfelelően felszabadítani:

```text
memory leak
```

alakulhat ki.

Példa:

```text
App indul
200 MB

később
1 GB

később
4 GB

később
8 GB
```

Ez szervereken komoly probléma lehet.

---

# 21. Swap

Ha kevés a RAM, az operációs rendszer használhat:

```text
swap
```

területet.

Ez háttértáron található.

Egyszerű modell:

```text
RAM
↓
memory pressure
↓
ritkábban használt memóriaoldalak
↓
SWAP
↓
SSD / disk
```

---

# 22. Swap és RAM különbsége

A swap nem azonos a RAM-mal.

```text
RAM
=
nagyon gyors
```

```text
Swap
=
háttértár
=
lassabb
```

Ezért:

```text
8 GB RAM + 8 GB swap
```

nem ugyanaz, mint:

```text
16 GB RAM
```

---

# 23. Out Of Memory

Ha:

```text
RAM elfogy
+
nincs elegendő swap
```

akkor:

```text
Out Of Memory
```

helyzet alakulhat ki.

Linuxban később találkozunk az:

```text
OOM Killer
```

mechanizmussal.

---

# 24. User space

A normál alkalmazások általában:

```text
user space
```

területen futnak.

Például:

```text
Node.js
Nginx
PostgreSQL
browser
VS Code
```

---

# 25. Kernel space

A kernel privilegizált környezetben fut.

Hozzáférhet többek között:

```text
CPU
RAM
storage
network hardware
drivers
process management
```

erőforrásokhoz.

---

# 26. User space és kernel space

A szétválasztás célja:

```text
biztonság
stabilitás
hozzáférés kontrollálása
```

Egyszerű modell:

```text
USER SPACE

Node
Nginx
PostgreSQL
Browser

──────────────

KERNEL SPACE

Kernel
Drivers
Memory Management
Scheduler
Networking
Filesystem
```

---

# 27. Miért nem érheti el az alkalmazás közvetlenül a hardvert?

Ha minden alkalmazás közvetlenül kezelhetné:

```text
RAM
disk
network card
CPU
```

erőforrásokat, egyetlen programhibával az egész rendszer sérülhetne.

Ezért az alkalmazás:

```text
korlátozott környezet
```

alatt fut.

A kernel végzi a privilegizált műveleteket.

---

# 28. System call

A **system call** olyan mechanizmus, amelyen keresztül egy user-space program szolgáltatást kér a kerneltől.

Egyszerű modell:

```text
Application
↓
System Call
↓
Kernel
↓
Hardware / OS resource
```

---

# 29. System call példák

Kernel segítség szükséges például:

```text
fájl megnyitásához
fájl olvasásához
fájl írásához
process létrehozásához
memory kezeléshez
network socket használatához
```

---

# 30. Fájlolvasási példa

Alkalmazás:

```text
read config.txt
```

Egyszerűsített folyamat:

```text
Application
↓
system call
↓
Kernel
↓
Filesystem
↓
Driver
↓
Storage
```

Az adat visszafelé:

```text
Storage
↓
Kernel
↓
Application
```

---

# 31. Hálózati példa

Egy Node szerver:

```text
server.listen(3000)
```

mögött leegyszerűsítve:

```text
Node process
↓
system call
↓
Kernel network stack
↓
socket
↓
network interface
↓
network
```

A Node program nem közvetlenül a hálózati kártyát kezeli.

---

# 32. User mode és kernel mode

Normál program:

```text
user mode
```

System call során:

```text
user mode
↓
system call
↓
kernel mode
↓
kernel operation
↓
user mode
```

Ez szabályozott belépési pont a kernel funkcióihoz.

---

# 33. Teljes OS mentális modell

```text
PROGRAM
↓
PROCESS
↓
THREAD
↓
SCHEDULER
↓
CPU
```

A process közben:

```text
PROCESS
↓
VIRTUAL MEMORY
↓
RAM / SWAP
```

Ha OS-erőforrást akar:

```text
APPLICATION
↓
SYSTEM CALL
↓
KERNEL
↓
FILESYSTEM / NETWORK / MEMORY / HARDWARE
```

---

# 34. Teljes rendszerkép

```text
                   APPLICATION
                        │
                        ▼
                     PROCESS
                        │
                ┌───────┴────────┐
                │                │
             THREADS         VIRTUAL MEMORY
                │                │
                ▼                ▼
            SCHEDULER       RAM / SWAP
                │
                ▼
              CPU


APPLICATION
     │
     │ system call
     ▼
   KERNEL
     │
 ┌───┼──────────────┐
 │   │              │
 ▼   ▼              ▼
Disk Network       Memory
 │   │
 ▼   ▼
Hardware
```

---

# 35. Legfontosabb fogalmak röviden

## Program

```text
passzív kód
```

## Process

```text
futó program példánya
```

## PID

```text
process azonosító
```

## Thread

```text
végrehajtási szál
```

## Scheduler

```text
eldönti, melyik thread kap CPU-időt
```

## Context switch

```text
váltás egyik threadről a másikra
```

## RAM

```text
gyors ideiglenes memória
```

## Virtual memory

```text
a process által látott logikai memóriatér
```

## Stack

```text
függvényhívásokhoz kapcsolódó memória
```

## Heap

```text
dinamikusan lefoglalt memória
```

## Swap

```text
háttértáron lévő memória-kiegészítés
```

## User space

```text
normál alkalmazások környezete
```

## Kernel space

```text
privilegizált operációsrendszer-környezet
```

## System call

```text
az alkalmazás kérése a kernel felé
```

---

# 36. DevOps szempontból

Később ezekkel a problémákkal fogunk találkozni:

```text
high CPU usage
high memory usage
memory leak
swap usage
process crash
OOM
slow disk I/O
network socket problem
```

A hibák megértésének alapja:

```text
process
thread
scheduler
memory
kernel
system calls
```

Ezért fontos ezeket az alapokat még a Linux parancsok előtt megérteni.