# Git parancsok – részletes referencia

Ez a jegyzet a Git verziókezelés során leggyakrabban használt parancsokat gyűjti össze.

Alapvető munkafolyamat:

```text
Working Directory
      ↓
git add
      ↓
Staging Area
      ↓
git commit
      ↓
Local Repository
      ↓
git push
      ↓
Remote Repository
```

---

# 1. Git verzió ellenőrzése

```bash
git --version
```

Megmutatja a telepített Git verzióját.

Példa:

```text
git version 2.43.0
```

---

# 2. Súgó

Általános segítség:

```bash
git help
```

Egy konkrét parancs dokumentációja:

```bash
git help commit
```

vagy:

```bash
git commit --help
```

Rövid segítség:

```bash
git commit -h
```

---

# 3. Repository létrehozása

## git init

Új Git repository létrehozása az aktuális könyvtárban.

```bash
git init
```

Létrejön:

```text
.git/
```

könyvtár.

Példa:

```bash
mkdir my-project
cd my-project
git init
```

---

# 4. Repository klónozása

## git clone

Egy remote repository teljes másolatának letöltése.

```bash
git clone <repository-url>
```

HTTPS példa:

```bash
git clone https://github.com/example-user/my-project.git
```

SSH példa:

```bash
git clone git@github.com:example-user/my-project.git
```

A clone általában:

```text
repository letöltése
+
Git history
+
branchek információi
+
origin remote létrehozása
```

műveleteket végzi el.

---

# 5. Repository állapotának megtekintése

## git status

Megmutatja a repository aktuális állapotát.

```bash
git status
```

Látható például:

- melyik branchen vagyunk,
- milyen fájlokat módosítottunk,
- mely fájlok staged állapotúak,
- mely fájlokat nem követi még a Git.

Rövidebb változat:

```bash
git status -s
```

---

# 6. Fájl hozzáadása a staging area-hoz

## git add

Egy fájl:

```bash
git add README.md
```

Több fájl:

```bash
git add file1.txt file2.txt
```

Minden aktuális módosítás:

```bash
git add .
```

Az összes változtatás:

```bash
git add -A
```

Interaktív kiválasztás:

```bash
git add -p
```

A `-p` segítségével egy fájlon belül is kiválaszthatjuk, mely módosításokat szeretnénk stage-elni.

---

# 7. Commit létrehozása

## git commit

Staged változtatások commitolása:

```bash
git commit -m "Add login feature"
```

A:

```text
-m
```

kapcsoló után adjuk meg a commit message-et.

---

## Commit editor használatával

```bash
git commit
```

Megnyitja a konfigurált szövegszerkesztőt a commit message számára.

---

## Módosított tracked fájlok automatikus stage-elése

```bash
git commit -am "Fix login bug"
```

Fontos:

Ez csak már tracked fájlokat ad hozzá.

Új fájlokat nem.

---

# 8. Utolsó commit módosítása

## git commit --amend

Ha az utolsó commitból valami kimaradt:

```bash
git add forgotten-file.txt
git commit --amend
```

Commit message módosítása:

```bash
git commit --amend -m "Better commit message"
```

Fontos:

A `--amend` új commit hash-t hoz létre.

Már publikált commitnál óvatosan használjuk.

---

# 9. Commit történet

## git log

Teljes commit history:

```bash
git log
```

Rövidített:

```bash
git log --oneline
```

Példa:

```text
6b12a91 Add login
90bc213 Add database
f85de11 Initial commit
```

---

## Branch gráf megjelenítése

```bash
git log --oneline --graph --all
```

Hasznosabb forma:

```bash
git log --oneline --graph --decorate --all
```

Példa:

```text
* 8ad23f1 (feature/login) Add login validation
| * 2cf9172 (main) Update README
|/
* a612fc1 Initial commit
```

Ez nagyon hasznos a branch struktúra megértéséhez.

---

# 10. Commit részleteinek megtekintése

## git show

Az aktuális commit:

```bash
git show
```

Konkrét commit:

```bash
git show <commit-hash>
```

Példa:

```bash
git show a41f27c
```

Megmutatja:

- szerző,
- dátum,
- commit message,
- változtatások.

---

# 11. Módosítások összehasonlítása

## git diff

Nem staged változtatások:

```bash
git diff
```

Staged változtatások:

```bash
git diff --staged
```

vagy:

```bash
git diff --cached
```

Két commit:

```bash
git diff <commit1> <commit2>
```

Példa:

```bash
git diff a41f27c b78a931
```

Két branch:

```bash
git diff main feature/login
```

---

# 12. Branchek megtekintése

## git branch

Local branchek:

```bash
git branch
```

Példa:

```text
* main
  feature/login
  bugfix/navbar
```

A `*` jelzi az aktuális branchet.

Remote branchek:

```bash
git branch -r
```

Minden branch:

```bash
git branch -a
```

---

# 13. Új branch létrehozása

```bash
git branch feature/login
```

Ez létrehozza a branchet, de nem vált át rá.

---

# 14. Branch váltása

Modern parancs:

```bash
git switch feature/login
```

Régebbi forma:

```bash
git checkout feature/login
```

---

# 15. Branch létrehozása és azonnali váltás

```bash
git switch -c feature/login
```

Régebbi forma:

```bash
git checkout -b feature/login
```

---

# 16. Branch létrehozása egy régi commitból

```bash
git switch -c alternative-version <commit-hash>
```

Példa:

```bash
git switch -c old-version-test a41f27c
```

Ezzel egy régebbi projektállapotból új fejlesztési irányt indíthatunk.

---

# 17. Branch átnevezése

Aktuális branch:

```bash
git branch -m new-name
```

Példa:

```bash
git branch -m feature/authentication
```

Másik branch:

```bash
git branch -m old-name new-name
```

---

# 18. Branch törlése

Biztonságos törlés:

```bash
git branch -d feature/login
```

A Git nem engedi, ha a branch nincs merge-elve.

Kényszerített törlés:

```bash
git branch -D feature/login
```

Figyelem:

A `-D` elveszíthet nem merge-elt munkát.

---

# 19. Merge

## git merge

Másik branch beillesztése az aktuális branchbe.

Példa:

```bash
git switch main
git merge feature/login
```

Folyamat:

```text
feature/login
      ↓
    merge
      ↓
main
```

---

# 20. Merge megszakítása

Ha merge közben probléma van:

```bash
git merge --abort
```

Visszaállítja a merge előtti állapotot, ha lehetséges.

---

# 21. Merge conflict

Conflict után:

```bash
git status
```

megmutatja az ütköző fájlokat.

A fájlokat kézzel javítjuk.

Ezután:

```bash
git add <file>
```

majd:

```bash
git commit
```

---

# 22. Remote-ok megtekintése

## git remote

```bash
git remote
```

Részletesen:

```bash
git remote -v
```

Példa:

```text
origin  https://github.com/user/project.git (fetch)
origin  https://github.com/user/project.git (push)
```

---

# 23. Remote hozzáadása

```bash
git remote add origin <url>
```

Példa:

```bash
git remote add origin https://github.com/user/project.git
```

Az `origin` nem kötelező név.

Például:

```bash
git remote add github https://github.com/user/project.git
```

---

# 24. Remote URL módosítása

```bash
git remote set-url origin <new-url>
```

HTTPS → SSH példa:

```bash
git remote set-url origin git@github.com:user/project.git
```

---

# 25. Remote törlése

```bash
git remote remove origin
```

vagy:

```bash
git remote rm origin
```

---

# 26. Remote átnevezése

```bash
git remote rename origin github
```

---

# 27. Remote részletes információ

```bash
git remote show origin
```

Megmutatja például:

- URL,
- tracked branchek,
- upstream kapcsolatokat.

---

# 28. Fetch

## git fetch

Remote változások lekérése anélkül, hogy a local branchet módosítaná.

```bash
git fetch
```

Konkrét remote:

```bash
git fetch origin
```

Minden remote:

```bash
git fetch --all
```

Régi remote branch referenciák takarítása:

```bash
git fetch --prune
```

---

# 29. Pull

## git pull

Remote változások lekérése és beépítése.

```bash
git pull
```

Konkrét remote és branch:

```bash
git pull origin main
```

Nagyon leegyszerűsítve:

```text
git pull
=
git fetch
+
merge / rebase
```

---

# 30. Pull rebase használatával

```bash
git pull --rebase
```

A remote változásokat lekéri, majd a local commitokat azok fölé helyezi.

Ezzel gyakran lineárisabb history készül.

---

# 31. Push

## git push

Commitok feltöltése remote repositoryba.

```bash
git push
```

Konkrét branch:

```bash
git push origin main
```

---

# 32. Első push + upstream

```bash
git push -u origin main
```

A:

```text
-u
```

beállítja az upstream kapcsolatot.

Utána elég:

```bash
git push
```

---

# 33. Feature branch push

```bash
git push -u origin feature/login
```

GitHubon létrejön például:

```text
origin/feature/login
```

---

# 34. Remote branch törlése

```bash
git push origin --delete feature/login
```

---

# 35. Force push

```bash
git push --force
```

A remote history felülírására használható.

VESZÉLYES.

Biztonságosabb:

```bash
git push --force-with-lease
```

A `--force-with-lease` ellenőrzi, hogy a remote branch nem változott-e közben váratlanul.

Megosztott `main` branchen általában kerülendő.

---

# 36. Remote branch local használata

Először:

```bash
git fetch
```

Remote branchek:

```bash
git branch -r
```

Local tracking branch létrehozása:

```bash
git switch -c feature/payment --track origin/feature/payment
```

Sok esetben egyszerűen:

```bash
git switch feature/payment
```

is működik.

---

# 37. Upstream megtekintése

```bash
git branch -vv
```

Példa:

```text
* main  a41f27c [origin/main] Update README
```

---

# 38. Upstream beállítása

```bash
git branch --set-upstream-to=origin/main main
```

vagy első pushnál:

```bash
git push -u origin main
```

---

# 39. Upstream eltávolítása

```bash
git branch --unset-upstream
```

---

# 40. Working directory módosítás visszavonása

## git restore

Egy fájl visszaállítása az utolsó commit állapotára:

```bash
git restore README.md
```

Minden tracked fájl:

```bash
git restore .
```

Figyelem:

A nem commitolt változtatások elveszhetnek.

---

# 41. Fájl kivétele stagingből

```bash
git restore --staged README.md
```

A fájl módosítása megmarad, de már nem staged.

---

# 42. Régebbi commitból fájl visszaállítása

```bash
git restore --source=<commit-hash> README.md
```

Példa:

```bash
git restore --source=a41f27c README.md
```

---

# 43. Reset

A `reset` a branch pozícióját és opcionálisan a staging/working directory állapotát módosítja.

## Soft reset

```bash
git reset --soft HEAD~1
```

Commit eltűnik, de a változtatás staged marad.

---

## Mixed reset

```bash
git reset HEAD~1
```

vagy:

```bash
git reset --mixed HEAD~1
```

Commit eltűnik.

A módosítások megmaradnak, de már nem staged állapotban.

---

## Hard reset

```bash
git reset --hard HEAD~1
```

Commit és working directory változtatások is elveszhetnek.

VESZÉLYES.

---

# 44. Konkrét commitra reset

```bash
git reset --hard <commit-hash>
```

Példa:

```bash
git reset --hard a41f27c
```

---

# 45. Revert

## git revert

Egy már létező commit változtatásainak visszavonása új committal.

```bash
git revert <commit-hash>
```

Példa:

```bash
git revert a41f27c
```

Történet:

```text
A --- B --- C --- R
```

ahol `R` visszavonja `C` változtatását.

Publikált history esetén általában biztonságosabb, mint a reset.

---

# 46. Reset vs revert

```text
reset
=
history / branch pointer módosítása
```

```text
revert
=
új commitban visszavonás
```

Általános szabály:

```text
local, még nem publikált commit
→ reset használható
```

```text
már pusholt / megosztott commit
→ revert gyakran jobb
```

---

# 47. Detached HEAD

Régi commit megtekintése:

```bash
git switch --detach <commit-hash>
```

Példa:

```bash
git switch --detach a41f27c
```

Ekkor nem egy branchen vagyunk.

Ha ebből új munkát szeretnénk:

```bash
git switch -c alternative-version
```

---

# 48. Stash

## git stash

Ideiglenesen félrerakja a még nem commitolt változtatásokat.

```bash
git stash
```

Ez hasznos például, ha:

```text
dolgozunk valamin
↓
gyorsan át kell váltani másik branchre
↓
még nem akarunk commitot készíteni
```

---

# 49. Stash listázása

```bash
git stash list
```

Példa:

```text
stash@{0}: WIP on feature/login
stash@{1}: WIP on main
```

---

# 50. Stash visszatöltése

```bash
git stash apply
```

A stash megmarad.

---

## Stash visszatöltése és törlése

```bash
git stash pop
```

Alkalmazza és eltávolítja a stash listából.

---

# 51. Konkrét stash használata

```bash
git stash apply stash@{1}
```

---

# 52. Stash törlése

Egy:

```bash
git stash drop stash@{0}
```

Mind:

```bash
git stash clear
```

---

# 53. Stash üzenettel

```bash
git stash push -m "Login feature in progress"
```

---

# 54. Új fájlok stash-elése is

```bash
git stash -u
```

A `-u` az untracked fájlokat is félrerakja.

---

# 55. Fájl törlése Gitből

## git rm

Fájl törlése:

```bash
git rm old-file.txt
```

Majd:

```bash
git commit -m "Remove old file"
```

---

# 56. Fájl megtartása, Git tracking megszüntetése

```bash
git rm --cached .env
```

A fájl megmarad a gépen, de Git többé nem követi.

Tipikus `.env` eset.

Ezután:

```text
.env
```

kerüljön `.gitignore` fájlba.

---

# 57. Fájl átnevezése vagy mozgatása

## git mv

```bash
git mv old.txt new.txt
```

Majd:

```bash
git commit -m "Rename file"
```

---

# 58. Nem követett fájlok törlése

## git clean

Először mindig preview:

```bash
git clean -n
```

Megmutatja, mit törölne.

Tényleges törlés:

```bash
git clean -f
```

Mappák is:

```bash
git clean -fd
```

VESZÉLYES, mert untracked fájlokat végleg törölhet.

---

# 59. .gitignore

Nem parancs, hanem fontos Git fájl.

Példa:

```text
node_modules/
.env
dist/
.next/
*.log
```

A Git ezeket nem fogja normál esetben trackelni.

Fontos:

Ha egy fájl már tracked, `.gitignore` önmagában nem szünteti meg a trackinget.

Ilyenkor például:

```bash
git rm --cached .env
```

---

# 60. Tag

## git tag

A tagek konkrét commitokat jelölnek.

Gyakori release-eknél.

Például:

```bash
git tag v1.0.0
```

---

## Annotated tag

```bash
git tag -a v1.0.0 -m "Version 1.0.0"
```

---

# 61. Tagek listája

```bash
git tag
```

---

# 62. Tag konkrét commitra

```bash
git tag v1.0.0 <commit-hash>
```

---

# 63. Tag feltöltése

Egy:

```bash
git push origin v1.0.0
```

Minden tag:

```bash
git push origin --tags
```

---

# 64. Local tag törlése

```bash
git tag -d v1.0.0
```

Remote:

```bash
git push origin --delete v1.0.0
```

---

# 65. Rebase

## git rebase

Egy branch commitjait másik commitlánc tetejére helyezi.

Példa:

```bash
git switch feature/login
git rebase main
```

Előtte:

```text
A --- B --- C main
     \
      D --- E feature
```

Rebase után:

```text
A --- B --- C main
             \
              D' --- E' feature
```

A commit hash-ek megváltoznak.

---

# 66. Rebase conflict folytatása

Conflict javítása után:

```bash
git add .
git rebase --continue
```

---

# 67. Rebase megszakítása

```bash
git rebase --abort
```

---

# 68. Interactive rebase

```bash
git rebase -i HEAD~3
```

Az utolsó három commit szerkesztése.

Lehet például:

```text
pick
reword
edit
squash
fixup
drop
```

Segítségével:

- commitokat összevonhatunk,
- átnevezhetünk,
- törölhetünk,
- sorrendet módosíthatunk.

Publikált brancheken óvatosan.

---

# 69. Cherry-pick

## git cherry-pick

Egy konkrét commit átemelése egy másik branchre.

```bash
git cherry-pick <commit-hash>
```

Példa:

```bash
git switch main
git cherry-pick a41f27c
```

Hasznos, ha nem akarunk egy egész branchet merge-elni, csak egy konkrét commitot.

---

# 70. Cherry-pick megszakítása

```bash
git cherry-pick --abort
```

---

# 71. Reflog

## git reflog

Nagyon fontos helyreállító eszköz.

Megmutatja, hogy a `HEAD` és branchek korábban milyen commitokra mutattak.

```bash
git reflog
```

Példa:

```text
a41f27c HEAD@{0}: reset: moving to HEAD~1
b59af31 HEAD@{1}: commit: Add login
```

Ha véletlenül reseteltünk egy commitot, sokszor a reflogból vissza lehet találni.

---

# 72. Elveszett commit visszaállítása reflogból

Megkeressük:

```bash
git reflog
```

Majd például:

```bash
git switch -c recovered-work b59af31
```

---

# 73. Blame

## git blame

Megmutatja, egy fájl egyes sorait melyik commit / szerző módosította.

```bash
git blame app.js
```

Hasznos annak kiderítésére:

```text
mikor került ide ez a sor?
melyik commit változtatta?
```

---

# 74. Keresés repositoryban

## git grep

```bash
git grep "DATABASE_URL"
```

Megkeresi a tracked fájlokban a szöveget.

---

# 75. Commit history keresése

Például commit message alapján:

```bash
git log --grep="login"
```

Egy fájl története:

```bash
git log -- README.md
```

---

# 76. Egy fájl változásainak követése

```bash
git log -p -- README.md
```

---

# 77. Commit szerzők összesítése

## git shortlog

```bash
git shortlog
```

Commit számokkal:

```bash
git shortlog -sn
```

Példa:

```text
42 Thomas Horvath
18 Developer Two
```

---

# 78. Commit azonosítás tag alapján

## git describe

```bash
git describe
```

Hasznos release/version információ generálására.

---

# 79. Git konfiguráció

## git config

Global username:

```bash
git config --global user.name "Thomas Horvath"
```

Global email:

```bash
git config --global user.email "user@example.com"
```

---

# 80. Config listázása

```bash
git config --list
```

Forrással együtt:

```bash
git config --list --show-origin
```

Global:

```bash
git config --global --list
```

Local:

```bash
git config --local --list
```

---

# 81. Konkrét config lekérdezése

```bash
git config user.name
```

```bash
git config user.email
```

---

# 82. Config érték törlése

```bash
git config --global --unset credential.helper
```

---

# 83. Default branch beállítása

```bash
git config --global init.defaultBranch main
```

Ezután az új repositoryk:

```bash
git init
```

esetén alapból `main` branchet használhatnak.

---

# 84. Credential helper

Példa:

```bash
git config --global credential.helper store
```

Ez HTTPS credentialök tárolására használható.

Biztonsági szempontból a `store` nem a legjobb megoldás.

GitHub CLI esetén:

```bash
gh auth setup-git
```

külön credential helper konfigurációt állíthat be.

---

# 85. Credential elutasítása / törlése

Példa Git credential protokoll használatával:

```bash
printf "protocol=https\nhost=github.com\n\n" | git credential reject
```

Ez használható eltárolt credential érvénytelenítésére a konfigurált helperen keresztül.

---

# 86. Git repository ellenőrzése

Megmutathatjuk a repository gyökérkönyvtárát:

```bash
git rev-parse --show-toplevel
```

Aktuális commit hash:

```bash
git rev-parse HEAD
```

Rövid hash:

```bash
git rev-parse --short HEAD
```

---

# 87. Aktuális branch neve

```bash
git branch --show-current
```

Példa:

```text
feature/login
```

---

# 88. Commit szülője

```text
HEAD
```

az aktuális commit.

Előző:

```text
HEAD~1
```

Kettővel korábbi:

```text
HEAD~2
```

Példa:

```bash
git show HEAD~2
```

---

# 89. Archive készítése repositoryból

## git archive

Például ZIP:

```bash
git archive --format=zip HEAD -o project.zip
```

Ez a repository adott verziójáról készít archívumot `.git` könyvtár nélkül.

---

# 90. Submodule

Másik Git repository beágyazása.

Hozzáadás:

```bash
git submodule add <repository-url>
```

Inicializálás:

```bash
git submodule init
```

Frissítés:

```bash
git submodule update
```

Clone után együtt:

```bash
git submodule update --init --recursive
```

Ez már haladóbb használat.

---

# 91. Worktree

Ugyanannak a repositorynak több branchét lehet egyszerre külön mappákban használni.

Példa:

```bash
git worktree add ../project-login feature/login
```

Így:

```text
project/
→ main

project-login/
→ feature/login
```

egyszerre lehet checkoutolva.

Haladó, de nagyon hasznos eszköz.

---

# 92. Worktree lista

```bash
git worktree list
```

---

# 93. Bisect

## git bisect

Hibakeresés commitok között bináris kereséssel.

Indítás:

```bash
git bisect start
```

Aktuális commit hibás:

```bash
git bisect bad
```

Egy régi jó commit:

```bash
git bisect good <commit-hash>
```

Git ezután commitokat választ ki tesztelésre.

Ha jó:

```bash
git bisect good
```

Ha rossz:

```bash
git bisect bad
```

Végül megtalálja azt a commitot, ahol a hiba bekerült.

Kilépés:

```bash
git bisect reset
```

Ez különösen érdekes későbbi hibakeresésnél.

---

# 94. Git notes

Commitokhoz külön megjegyzéseket lehet kapcsolni:

```bash
git notes add -m "Production tested"
```

Megtekintés:

```bash
git notes show
```

Ritkábban használt funkció.

---

# 95. Sparse checkout

Nagy repositoryból csak bizonyos könyvtárakat checkoutolhatunk.

Bekapcsolás:

```bash
git sparse-checkout init
```

Könyvtár kiválasztása:

```bash
git sparse-checkout set frontend
```

Nagy monorepóknál lehet hasznos.

---

# 96. A leggyakoribb napi workflow

Munka kezdése:

```bash
git switch main
git pull
```

Új feladat:

```bash
git switch -c feature/login
```

Munka:

```bash
git status
git add .
git commit -m "Add login"
```

Remote-ra:

```bash
git push -u origin feature/login
```

Ezután GitHub:

```text
Pull Request
↓
Review
↓
Checks
↓
Merge
```

Local takarítás:

```bash
git switch main
git pull
git branch -d feature/login
```

---

# 97. Gyors referencia

## Repository

```bash
git init
git clone <url>
git status
```

## Stage

```bash
git add <file>
git add .
git restore --staged <file>
```

## Commit

```bash
git commit -m "message"
git commit --amend
git log
git log --oneline
git show
```

## Branch

```bash
git branch
git branch -a
git switch <branch>
git switch -c <branch>
git branch -d <branch>
```

## Merge

```bash
git merge <branch>
git merge --abort
```

## Remote

```bash
git remote -v
git remote add origin <url>
git remote set-url origin <url>
git remote remove origin
```

## Sync

```bash
git fetch
git pull
git push
git push -u origin <branch>
```

## Visszavonás

```bash
git restore <file>
git reset --soft HEAD~1
git reset HEAD~1
git reset --hard HEAD~1
git revert <commit>
```

## Ideiglenes munka

```bash
git stash
git stash list
git stash pop
```

## Release

```bash
git tag
git tag -a v1.0.0 -m "Version 1.0.0"
git push origin --tags
```

## Haladó

```bash
git rebase
git rebase -i
git cherry-pick
git reflog
git blame
git bisect
```

---

# 98. Veszélyes parancsok

Különösen figyelni kell ezekre:

```bash
git reset --hard
```

Nem commitolt változtatások elveszhetnek.

```bash
git clean -f
```

Untracked fájlokat töröl.

```bash
git branch -D
```

Nem merge-elt branch is törölhető.

```bash
git push --force
```

Remote history felülírható.

```bash
git rebase
```

Commit history újraírását okozhatja.

Megosztott brancheken mindig gondoljuk át használatukat.

---

# 99. Mentális modell

```text
WORKING DIRECTORY
      │
      │ git add
      ▼
STAGING AREA
      │
      │ git commit
      ▼
LOCAL REPOSITORY
      │
      │ git push
      ▼
REMOTE REPOSITORY
```

Visszafelé:

```text
REMOTE
   │
   │ git fetch
   ▼
REMOTE-TRACKING BRANCH
   │
   │ merge / rebase
   ▼
LOCAL BRANCH
```

vagy:

```text
git pull
=
fetch + integráció
```

---

# 100. A legfontosabb napi parancsok

Ha csak a legfontosabbakat kell megjegyezni:

```bash
git status
git add .
git commit -m "message"
git log --oneline
git switch
git switch -c
git branch
git merge
git fetch
git pull
git push
git remote -v
git diff
git restore
git stash
git revert
```

Ezekkel a hétköznapi Git munka jelentős része elvégezhető.