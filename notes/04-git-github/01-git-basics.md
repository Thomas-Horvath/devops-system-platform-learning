# Git és GitHub alapok

## 1. Mi a Git?

A **Git** egy elosztott verziókezelő rendszer (*distributed version control system*).

Feladata, hogy egy projekt változásait nyomon kövesse.

Segítségével látható:

- milyen változtatások történtek,
- mikor történtek,
- ki végezte őket,
- mi volt a változtatás célja,
- és szükség esetén vissza lehet térni korábbi állapotokhoz.

Egyszerű modell:

```text
projekt
↓
módosítás
↓
commit
↓
új módosítás
↓
commit
↓
újabb verzió
```

A Git internetkapcsolat nélkül is működik.

---

## 2. Git és GitHub különbsége

### Git

A Git maga a verziókezelő rendszer.

A saját számítógépünkön követi a projekt változásait.

### GitHub

A GitHub egy online szolgáltatás, ahol Git repositorykat lehet tárolni és megosztani.

```text
Local repository
a saját gépen
        │
        │ git push
        ▼
Remote repository
GitHubon
```

Fontos:

**Git ≠ GitHub**

Git használható GitHub nélkül is.

---

## 3. Repository

A **repository**, röviden **repo**, egy Git által kezelt projekt.

```text
my-project/
├── README.md
├── src/
├── notes/
└── .git/
```

A `.git` könyvtár tartalmazza a Git által kezelt információkat.

Többek között:

- commit history,
- branchek,
- konfiguráció,
- remote adatok,
- verziótörténet.

Ha egy könyvtárban van `.git` könyvtár, akkor az Git repository.

---

## 4. Local és remote repository

### Local repository

A saját számítógépünkön található Git repository.

Példa:

```text
~/projects/devops-system-platform-learning
```

### Remote repository

Távoli szerveren található repository.

Például:

- GitHub
- GitLab
- Bitbucket
- saját Git szerver

```text
LOCAL REPOSITORY
       │
       │ push
       ▼
REMOTE REPOSITORY

LOCAL REPOSITORY
       ▲
       │ pull / fetch
       │
REMOTE REPOSITORY
```

---

## 5. Commit

A **commit** egy mentési pont a projekt történetében.

```bash
git add README.md
git commit -m "Update README"
```

Egyszerű történet:

```text
Commit A
   ↓
Commit B
   ↓
Commit C
```

Minden commit egyedi azonosítóval rendelkezik.

Ezt **commit hashnek** nevezzük.

Például:

```text
a41f27c
```

A commit message röviden leírja, mi történt.

```text
Add Linux notes
Fix database connection
Update README
```

---

# 6. Miért fontos a Git DevOpsban?

Gitben nem csak alkalmazáskód lehet.

DevOps környezetben például:

```text
application code
Dockerfile
docker-compose.yml
CI/CD pipeline
Terraform code
Kubernetes manifests
Nginx configuration
scripts
documentation
```

Egyszerű modell:

```text
Git repository
      ↓
változtatás
      ↓
commit
      ↓
push
      ↓
CI/CD pipeline
      ↓
build / test / deployment
```

---

# Branch, Remote és szinkronizálás

## 7. Branch

A **branch** egy külön fejlesztési ág ugyanazon repositoryn belül.

Segítségével új funkción vagy hibajavításon lehet dolgozni anélkül, hogy azonnal módosítanánk a fő ágat.

```text
A --- B --- C   main
         \
          D --- E   feature/login
```

Ebben:

- `main` = fő ág
- `feature/login` = külön fejlesztési ág

Ha elkészült, vissza lehet illeszteni a fő branchbe.

Ezt **merge-nek** nevezzük.

---

## 8. main

A `main` általában a repository fő branchének neve.

Régebben gyakran:

```text
master
```

volt használatos.

A `main` nem speciális Git-technológia, hanem egy branch neve.

```bash
git branch -M main
```

---

# 9. Több branch kezelése

Egy repositoryban egyszerre sok branch lehet.

Például:

```text
main
feature/login
feature/payment
bugfix/navbar
```

A local branchek megtekintése:

```bash
git branch
```

Példa:

```text
* main
  feature/login
  feature/payment
```

A `*` jelzi, melyik branchen állunk.

Branch váltása:

```bash
git switch feature/login
```

Régebbi forma:

```bash
git checkout feature/login
```

Új branch létrehozása és azonnali átváltás:

```bash
git switch -c feature/login
```

Példa:

```text
A --- B --- C   main
             \
              D --- E   feature/login
```

A `main` továbbra is `C`-n marad.

A feature branch új commitjai csak azon a branchen jelennek meg.

---

# 10. Remote repository

A **remote repository** egy másik helyen található Git repository.

Egy local repositoryhoz **nem csak egy remote tartozhat**.

Például:

```text
origin
company
backup
```

Ezek mutathatnak különböző szerverekre.

Például:

```text
origin  → GitHub
company → céges GitLab
backup  → saját Git szerver
```

---

# 11. origin

Az `origin` a fő remote repository szokásos neve.

Ez csak egy **alias**, vagyis rövid név.

```text
origin
↓
https://github.com/user/project.git
```

Az `origin`:

- nem maga a GitHub,
- nem kötelező név,
- tetszőlegesen átnevezhető vagy más név is használható.

Például:

```bash
git remote add github https://github.com/user/project.git
```

Ekkor:

```bash
git push github main
```

teljesen szabályos.

Az `origin` azért gyakori, mert clone esetén a Git alapértelmezetten ezt a nevet adja annak a remote-nak, ahonnan a repository származik.

---

## Remote-ok megtekintése

```bash
git remote -v
```

Példa:

```text
origin   https://github.com/user/project.git
company  https://gitlab.company.hu/project.git
backup   ssh://server/project.git
```

---

# 12. Remote hozzáadása

```bash
git remote add origin <repository-url>
```

Például:

```bash
git remote add origin https://github.com/user/project.git
```

Ha már létezik:

```text
remote origin already exists
```

Ez azt jelenti, hogy már van `origin` nevű remote.

---

# 13. Local branch és remote-tracking branch

A local és remote branch nem ugyanaz.

```text
main
```

a local branch.

```text
origin/main
```

az `origin` remote `main` branchének Git által ismert állapota.

```text
local main
    │
    │ követi
    ▼
origin/main
```

---

# 14. Upstream branch

Egy local branchhez hozzárendelhető egy remote branch.

Ezt **upstream kapcsolatnak** nevezzük.

```text
local main
↓
origin/main
```

Beállítása:

```bash
git push -u origin main
```

A `-u` beállítja az upstream kapcsolatot.

Ezután általában elég:

```bash
git push
```

vagy:

```bash
git pull
```

---

# 15. Nem-main branch feltöltése GitHubra

Feature branch ugyanúgy feltölthető, mint a `main`.

Például:

```bash
git push -u origin feature/login
```

Ekkor remote oldalon létrejön:

```text
origin/feature/login
```

és az upstream kapcsolat is beáll.

Utána:

```bash
git push
```

elég.

GitHubon tehát párhuzamosan lehet:

```text
main
feature/login
feature/payment
bugfix/navbar
```

---

# 16. push

A `git push` a local commitokat küldi fel a remote repositoryba.

Local:

```text
A --- B --- C   main
```

Remote:

```text
A --- B         main
```

Push után:

```text
A --- B --- C
```

mindkét oldalon.

```bash
git push
```

vagy:

```bash
git push origin main
```

---

## Fontos: a push commitokat küld

Tipikus workflow:

```bash
git add .
git commit -m "Add Git notes"
git push
```

A még nem commitolt változtatást a `push` nem küldi fel.

---

# 17. fetch

A:

```bash
git fetch
```

lekéri a remote repository új Git-információit.

Remote:

```text
A --- B --- C
```

Local:

```text
A --- B
```

Fetch után:

```text
main        = B
origin/main = C
```

A local `main` még nem változik.

Mentális modell:

```text
REMOTE
  │
  │ fetch
  ▼
origin/main

local main változatlan
```

A `fetch` jelentése:

> Nézd meg, mi változott a remote-on, de a saját branchemet még ne módosítsd.

---

# 18. pull

A:

```bash
git pull
```

nagyon leegyszerűsítve:

```text
fetch
+
merge / integráció
```

Tehát nemcsak lekéri a változásokat, hanem be is építi őket az aktuális local branchbe.

---

# 19. push, fetch és pull

```text
push
=
local → remote

fetch
=
remote információ → Git
local branch módosítása nélkül

pull
=
remote → local branch
és beépítés
```

Memóriafogás:

```text
push  = küldök
fetch = megnézem / lekérem
pull  = lekérem és beépítem
```

---

# 20. Ha egy kolléga új branchet készített

Tegyük fel, hogy a kolléga létrehozott és pusholt egy új branchet:

```text
feature/payment
```

Először:

```bash
git fetch
```

A remote branchek megtekintése:

```bash
git branch -r
```

Például:

```text
origin/main
origin/feature/payment
```

Ezután készíthetünk saját local branchet, amely követi a remote branchet:

```bash
git switch -c feature/payment --track origin/feature/payment
```

Újabb Git-verziókban sokszor elég:

```bash
git switch feature/payment
```

ha a Git egyértelműen megtalálja a megfelelő remote branchet.

Ekkor:

```text
origin/feature/payment
        ↓
local feature/payment
```

A branch ezután:

- futtatható,
- tesztelhető,
- módosítható,
- tovább commitolható.

---

# 21. Kolléga munkájának külön branchben tesztelése

Nem kell rögtön merge-elni a kolléga munkáját a saját branchünkbe.

Például:

```bash
git fetch
git switch -c colleague-test --track origin/feature/payment
```

Ezután külön tesztelhetjük:

```bash
npm install
npm test
npm run dev
```

Ha megfelelő:

```bash
git switch my-feature
git merge colleague-test
```

Így a saját munkánk addig változatlan marad, amíg tudatosan nem merge-eljük.

---

# 22. Merge

A **merge** két branch történetét egyesíti.

Példa:

```text
A --- B --- C   main
         \
          D --- E   feature
```

Merge után létrejöhet:

```text
A --- B --- C ------- M   main
         \           /
          D --- E ---
```

Az `M` egy merge commit lehet.

---

# 23. Merge conflict

Ha két branch ugyanazon részét eltérően módosították, konfliktus keletkezhet.

```text
           C   remote
          /
A --- B
          \
           D   local
```

A Git ilyenkor nem tudja automatikusan eldönteni, mi legyen a helyes végeredmény.

Fontos:

A konfliktus feloldása nem feltétlenül azt jelenti, hogy:

```text
enyém
VAGY
kollégáé
```

A végeredmény akár egy harmadik verzió is lehet.

Például:

```text
enyém:
price = amount * 1.27

kollégáé:
price = calculateTax(amount)

végeredmény:
price = calculateTax(amount, VAT_RATE)
```

A Git csak jelzi a konfliktust.

A szakmailag helyes végleges kódot az embernek kell meghatároznia.

---

# 24. Régi commit megtekintése

Legyen:

```text
A --- B --- C --- D   main
```

Ha a `B` állapotot szeretnénk megnézni:

```bash
git switch --detach <B-commit-hash>
```

Ekkor:

```text
A --- B --- C --- D
      ↑
     HEAD
```

Ez **detached HEAD** állapot.

---

# 25. Új branch indítása régi commitból

Ha a régi verzióból új irányba szeretnénk tovább dolgozni:

```bash
git switch --detach <B-hash>
git switch -c alternative-version
```

Eredmény:

```text
A --- B --- C --- D   main
      \
       E --- F        alternative-version
```

Ezután a két fejlesztési irány külön létezik.

Később akár merge-elhető:

```bash
git switch main
git merge alternative-version
```

---

# 26. Branch mint mutató

Fontos mentális modell:

A branch nem egy teljes projektmásolat.

A branch lényegében egy **mutató egy commitra**.

Például:

```text
                E --- F   feature-A
               /
A --- B --- C --- D       main
         \
          G --- H         feature-B
```

A mutatók:

```text
main      → D
feature-A → F
feature-B → H
```

Amikor új commit készül az aktuális branchen, annak mutatója előrelép.

Ezért nagyon gyors és olcsó új brancheket létrehozni.

---

# 27. HEAD

A `HEAD` azt jelzi, hogy jelenleg melyik branch / commit van checkoutolva.

Normál eset:

```text
HEAD
 ↓
main
 ↓
D
```

Detached HEAD esetén:

```text
HEAD
 ↓
B
```

Ilyenkor a `HEAD` közvetlenül egy commitra mutat, nem branchre.

---

# 28. Local commit visszavonása reset segítségével

Legyen:

```text
A --- B --- C   main
```

Ha `C` hibás és még csak localban van:

```bash
git reset --hard HEAD~1
```

Ekkor:

```text
A --- B   main
```

A branch visszakerül `B`-re.

---

## Soft reset

```bash
git reset --soft HEAD~1
```

A commit visszavonódik, de a változtatások staged állapotban megmaradnak.

---

## Mixed reset

```bash
git reset HEAD~1
```

A commit visszavonódik.

A fájlmódosítások megmaradnak a working directoryban.

---

## Hard reset

```bash
git reset --hard HEAD~1
```

A commit és a hozzá tartozó munkafájl-változtatások is eldobódhatnak.

Ezért a `--hard` veszélyes művelet.

---

# 29. Már pusholt commit visszavonása

Tegyük fel:

```text
LOCAL:
A --- B --- C

REMOTE:
A --- B --- C
```

Localban reset:

```bash
git reset --hard HEAD~1
```

Ekkor:

```text
LOCAL:
A --- B

REMOTE:
A --- B --- C
```

Normál:

```bash
git push
```

esetén a Git általában elutasítja a műveletet:

```text
non-fast-forward
```

Ennek oka:

A remote olyan commitot tartalmaz (`C`), amely már nincs benne a local branch történetében.

A Git nem akarja automatikusan törölni a remote történetet.

---

# 30. Force push

A remote history technikailag átírható:

```bash
git push --force
```

Biztonságosabb változat:

```bash
git push --force-with-lease
```

A `--force-with-lease` ellenőrzi, hogy a remote branch nem változott-e közben olyan módon, amelyről nem tudunk.

Fontos szabály:

> Megosztott branch, különösen `main` történetét általában nem írjuk át force push-sal.

Ennek oka, hogy más kollégák már dolgozhatnak a korábbi commitokra építve.

---

# 31. Revert

Már publikált commit visszavonására általában biztonságosabb:

```bash
git revert <commit-hash>
```

Példa:

```text
A --- B --- C --- R
```

ahol:

```text
C = hibás változtatás
R = C változtatásainak visszavonása
```

A `C` commit nem tűnik el.

A Git létrehoz egy új commitot, amely visszacsinálja a változtatásait.

---

# 32. reset vs revert

```text
reset
=
a branch történetének / mutatójának módosítása

revert
=
új commit készítése,
amely visszavon egy korábbi változtatást
```

Általános szabály:

```text
csak local commit
→ reset gyakran használható

már megosztott / pusholt commit
→ revert általában biztonságosabb
```

---

# 33. Release és branchek

Egy projekt release-e általában a fő branch egy meghatározott stabil állapotából készül.

Például:

```text
feature/login -----\
feature/payment ----> main → release
bugfix/navbar -----/
```

A feature branchek elkészülnek, tesztelődnek, majd merge-elődnek.

A végleges állapot tipikusan:

```text
main
↓
stabil commit
↓
release
```

A feature brancheket merge után gyakran törlik, mert már nincs rájuk szükség.

De technikailag nem kötelező, hogy csak egyetlen branch maradjon.

Nagy projektekben lehet például:

```text
main
develop
release/2.0
hotfix/security
```

is.

A release-t gyakran nem csak commit, hanem **tag** is jelöli.

Például:

```text
main
│
A --- B --- C --- D
              ↑
            v1.0.0
```

A tageket később külön tanuljuk.

---

# 34. Teljes Git mentális modell

```text
                 feature/login
                       │
                       ▼
                 D --- E
                /
A --- B --- C ---------------- F   main
         \
          G --- H
               ▲
               │
          feature/payment
```

Remote oldalon:

```text
origin/main
origin/feature/login
origin/feature/payment
```

Egy local branch követhet egy remote branchet:

```text
local main
↓
origin/main
```

és:

```text
local feature/login
↓
origin/feature/login
```

---

# 35. Fontos parancsok

### Local branchek

```bash
git branch
```

### Remote branchek

```bash
git branch -r
```

### Összes branch

```bash
git branch -a
```

### Branch váltás

```bash
git switch branch-name
```

### Új branch

```bash
git switch -c new-branch
```

### Branch push

```bash
git push -u origin branch-name
```

### Remote-ok

```bash
git remote -v
```

### Remote változások lekérése

```bash
git fetch
```

### Remote változások lekérése és beépítése

```bash
git pull
```

### Merge

```bash
git merge branch-name
```

### Régi commit megtekintése

```bash
git switch --detach <commit-hash>
```

### Régi commitból új branch

```bash
git switch -c new-branch <commit-hash>
```

### Utolsó local commit visszavonása, változások megtartásával

```bash
git reset --soft HEAD~1
```

### Hard reset

```bash
git reset --hard HEAD~1
```

### Publikált commit biztonságos visszavonása

```bash
git revert <commit-hash>
```

---

# Rövid összefoglalás

```text
repository
=
Git által kezelt projekt

commit
=
mentési pont / verzió

branch
=
egy fejlesztési ág

main
=
általában a fő branch

HEAD
=
ahol jelenleg állunk

remote
=
távoli repository-kapcsolat

origin
=
a fő remote szokásos neve,
de nem kötelező

origin/main
=
az origin main branchének
local Git által ismert állapota

upstream
=
local és remote branch kapcsolata

push
=
commitok feltöltése remote-ra

fetch
=
remote változások lekérése
a local branch módosítása nélkül

pull
=
fetch + változások beépítése

merge
=
branchek történetének egyesítése

merge conflict
=
a Git nem tudja automatikusan
összeegyeztetni két változtatást

reset
=
branch mutatójának / local historynak módosítása

revert
=
korábbi változtatás visszavonása
egy új commit segítségével
```

## Legfontosabb gondolat

A Gitben nem egyszerűen fájlokat másolgatunk.

A Git:

```text
commitokból
+
azok közötti kapcsolatokból
+
branch mutatókból
+
remote kapcsolatokból
```

épít fel egy verziótörténetet.

Ezért lehet ugyanabból a projektből több fejlesztési irányt létrehozni, külön tesztelni őket, majd később szükség szerint egyesíteni.


# Git konfiguráció

## 1. Mi a Git config?

A Git működését különböző konfigurációs beállításokkal lehet szabályozni.

Ezeket a:

```bash
git config
```

paranccsal kezeljük.

A Git konfiguráció tartalmazhat például:

- felhasználónevet,
- email címet,
- alapértelmezett branchet,
- editor beállítást,
- credential helper beállítást,
- remote repository információkat,
- branch tracking beállításokat.

---

# 2. Git identity

A Git commitokhoz tartozik egy szerzői identitás.

Például:

```text
Author: Thomas Horvath
Email: user@example.com
```

Ennek beállítása:

```bash
git config --global user.name "Thomas Horvath"
```

és:

```bash
git config --global user.email "user@example.com"
```

Ez az információ bekerül a commit metadata-ba.

Fontos:

```text
Git identity
≠
GitHub authentication
```

A `user.name` és `user.email` nem jelent GitHub-bejelentkezést.

Csak azt határozza meg, milyen szerzői információ kerüljön a commitba.

---

# 3. Git config szintek

A Git konfiguráció három fő szinten létezhet:

```text
SYSTEM
↓
GLOBAL
↓
LOCAL
```

A specifikusabb beállítás általában felülírja az általánosabbat:

```text
local > global > system
```

---

# 4. System config

A system konfiguráció az egész rendszer Git telepítésére vonatkozik.

Parancs:

```bash
git config --system ...
```

Linuxon tipikus konfigurációs fájl:

```text
/etc/gitconfig
```

Ez több felhasználót és repositoryt is érinthet.

Hétköznapi Git használat során ritkábban módosítjuk.

---

# 5. Global config

A global konfiguráció az adott operációs rendszer adott felhasználójára vonatkozik.

Például:

```bash
git config --global user.name "Thomas Horvath"
```

Linux / WSL alatt tipikusan:

```text
~/.gitconfig
```

fájlban található.

Például:

```text
/home/user/.gitconfig
```

A global beállítás általában minden repositoryban érvényes az adott felhasználónál.

---

# 6. Windows és WSL Git konfiguráció

A Windows és WSL külön operációs környezet.

Ezért:

```text
Windows Git config
≠
WSL Git config
```

A Windows Git és a WSL-ben futó Linux Git külön konfigurációs fájlokat használhat.

Ezért előfordulhat, hogy Windows alatt már beállítottuk:

```text
user.name
user.email
```

de WSL-ben újra be kell állítani őket.

---

# 7. Local config

A local konfiguráció csak az aktuális repositoryra vonatkozik.

Például:

```bash
git config user.name "Thomas Work"
```

Mivel nincs `--global`, a beállítás csak az aktuális repositoryban érvényes.

A konfiguráció a repository:

```text
.git/config
```

fájljában tárolódik.

---

# 8. Global és local beállítás együtt

Példa:

Global:

```text
user.name = Thomas Horvath
user.email = personal@example.com
```

Egy céges repository local configja:

```text
user.email = work@company.com
```

Ebben a repositoryban a Git a local emailt használja.

Más repositorykban továbbra is a global email érvényes.

---

# 9. Prioritás

Ha ugyanaz a beállítás több szinten is létezik:

```text
system:
user.name = Default User

global:
user.name = Thomas Horvath

local:
user.name = Thomas Work
```

akkor az adott repositoryban:

```text
Thomas Work
```

lesz használva.

Szabály:

```text
local > global > system
```

---

# 10. Konfiguráció megtekintése

Az összes aktív konfiguráció:

```bash
git config --list
```

Hasznosabb változat:

```bash
git config --list --show-origin
```

A `--show-origin` megmutatja, melyik konfigurációs fájlból származik az adott érték.

Példa:

```text
file:/home/user/.gitconfig    user.name=Thomas Horvath
file:/home/user/.gitconfig    user.email=user@example.com
file:.git/config              remote.origin.url=https://github.com/...
```

---

# 11. Egyes konfigurációs szintek megtekintése

Global:

```bash
git config --global --list
```

Local:

```bash
git config --local --list
```

System:

```bash
git config --system --list
```

Konkrét érték:

```bash
git config user.name
```

vagy:

```bash
git config user.email
```

---

# 12. Remote információ a local configban

A repositoryhoz tartozó remote beállítások a local Git configban is tárolódnak.

Például:

```bash
git remote add origin https://github.com/user/project.git
```

A:

```text
.git/config
```

fájlba kerülhet például:

```ini
[remote "origin"]
    url = https://github.com/user/project.git
    fetch = +refs/heads/*:refs/remotes/origin/*
```

Tehát a local Git konfiguráció tartalmazhat:

- remote URL-eket,
- branch tracking kapcsolatokat,
- repo-specifikus beállításokat.

---

# 13. Git identity és authentication különbsége

Git identity:

```text
Ki szerepel a commit szerzőjeként?
```

Például:

```text
Thomas Horvath
user@example.com
```

GitHub authentication:

```text
Valóban jogosult vagyok-e a GitHub account használatára?
```

Ez két külön folyamat.

A:

```bash
git config --global user.name ...
```

nem jelent GitHub-bejelentkezést.

---

# 14. Authentication és authorization

## Authentication

Jelentése:

```text
Ki vagy?
```

Példák:

```text
password
PAT
SSH key
```

---

## Authorization

Jelentése:

```text
Mit csinálhatsz?
```

Például:

```text
olvashatsz repositoryt?
pusholhatsz?
törölhetsz branchet?
adminisztrálhatod a repositoryt?
```

Memóriafogás:

```text
Authentication
=
KI VAGY?

Authorization
=
MIT SZABAD CSINÁLNOD?
```

---

# 15. Git config és GitHub kapcsolata

Egyszerű modell:

```text
LOCAL GIT

user.name
user.email
repository
branch
remote
     │
     │ push / pull
     ▼
GITHUB

authentication
authorization
remote repository
```

A Git config a helyi Git működését konfigurálja.

A GitHub authentication azt bizonyítja a távoli szolgáltatás felé, hogy valóban jogosultak vagyunk a művelet végrehajtására.

---

# Rövid összefoglalás

```text
git config
=
Git beállításainak kezelése

system
=
teljes rendszer szintje

global
=
aktuális operációs rendszer-felhasználó szintje

local
=
aktuális repository szintje

prioritás:
local > global > system

user.name
=
commit szerzőjének neve

user.email
=
commit szerzőjének email címe

Git identity
≠
GitHub authentication

authentication
=
ki vagy?

authorization
=
mit csinálhatsz?
```