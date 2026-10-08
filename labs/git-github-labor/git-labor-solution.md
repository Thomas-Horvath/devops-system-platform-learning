# Git + GitHub + GitHub Actions – komplex labor
## Megoldókulcs és magyarázat – 1. rész (1–19. feladat)

**Állapot:** 19. feladat teljesítve – Pull Request létrehozva  
**Következő feladat:** 20. Code Review szimuláció  
**Környezet:** Windows 11, WSL2 Ubuntu, VS Code

**Használt verziók:**
- Git: 2.34.1
- Node.js: 20.19.6
- npm: 11.12.1

---

# 1. A labor célja

A cél nem kizárólag a Git-parancsok gyakorlása, hanem egy fejlesztési folyamat megértése a projekt létrehozásától a GitHubon történő együttműködésig.

A teljes folyamat:

```text
Local development
       ↓
Git repository
       ↓
Commit
       ↓
GitHub repository
       ↓
Issue
       ↓
Feature branch
       ↓
Code changes + tests
       ↓
Commit + Push
       ↓
Pull Request
       ↓
Code Review
       ↓
CI Checks
       ↓
Merge
       ↓
Release
```

A DevOps szempontjából fontos, hogy megértsük, hogyan kerül a fejlesztő munkája egy közös repositoryba, és hogyan ellenőrizhető a kód automatikusan.

**English:**
> We use Git to track changes in our code.

*A Git segítségével követjük a kódban történt változásokat.*

---

# 2. Projekt létrehozása WSL alatt

A projekt tényleges helye:

```text
~/projects/LABS/git-github-actions-lab
```

Belépés:

```bash
cd ~/projects/LABS/git-github-actions-lab
```

Ellenőrzés:

```bash
pwd
```

A `pwd` a *print working directory* rövidítése.

A projektet a WSL Linux fájlrendszerében hoztuk létre, nem a Windows `/mnt/c/` csatolása alatt.

Ennek előnye, hogy Linux-fejlesztői környezetben dolgozunk, és elkerülhetünk bizonyos fájlrendszerbeli és teljesítményproblémákat.

---

# 3. Node.js projekt inicializálása

Parancs:

```bash
npm init -y
```

**Mit csinál?**

Létrehozza a `package.json` fájlt.

A `-y` jelentése **yes**: automatikusan elfogadja az alapértelmezett válaszokat.

A `package.json` többek között tartalmazhatja:
- a projekt nevét és verzióját,
- a függőségeket (*dependencies*),
- a fejlesztői függőségeket (*devDependencies*),
- a futtatható scripteket (*scripts*).

Például:

```json
{
  "name": "git-github-actions-lab",
  "version": "1.0.0",
  "scripts": {
    "test": "jest",
    "dev": "node src/calculator.js"
  }
}
```

Ez szemléltető részlet; a valódi `package.json` a Jest telepített verzióját is tartalmazza.

## Projektstruktúra

```text
git-github-actions-lab/
├── src/
│   └── calculator.js
├── test/
│   └── calculator.test.js
├── node_modules/
├── package.json
├── package-lock.json
├── README.md
└── .gitignore
```

A `src` tartalmazza az alkalmazás kódját, a `test` a tesztjeit.

---

# 4. Az alkalmazás elkészítése

A feladat egy egyszerű összeadó függvény létrehozása volt.

```javascript
function add(a, b) {
  return a + b;
}

module.exports = {
  add,
};
```

A `module.exports` segítségével a függvény más CommonJS-modulból is importálható.

Például:

```javascript
const { add } = require("../src/calculator");
```

## Parancssori argumentumok

A gyakorlat során továbbfejlesztettük a programot, hogy a felhasználó két számot adhasson át.

```javascript
const a = Number(process.argv[2]);
const b = Number(process.argv[3]);

function add(a, b) {
  return a + b;
}

console.log(add(a, b));

module.exports = {
  add,
};
```

A `process.argv` a Node.js által rendelkezésre bocsátott argumentumtömb.

Példa:

```bash
node src/calculator.js 3 4
```

Az argumentumok:

```text
process.argv[0] → Node executable
process.argv[1] → JavaScript file path
process.argv[2] → "3"
process.argv[3] → "4"
```

Mivel a parancssori argumentumok szövegek, `Number()` segítségével számmá alakítjuk őket.

Az eredmény:

```text
7
```

## Futtatás npm scriptből

A `package.json`-ban:

```json
"scripts": {
  "test": "jest",
  "dev": "node src/calculator.js"
}
```

Futtatás:

```bash
npm run dev -- 3 4
```

A `--` után megadott argumentumokat az npm továbbadja a scriptnek.

A tényleges futtatás így értelmezhető:

```bash
node src/calculator.js 3 4
```

**English:**
> The application reads two command-line arguments.

*Az alkalmazás két parancssori argumentumot olvas be.*

## Javasolt későbbi tisztítás

A labor előrehaladtával érdemes különválasztani az üzleti logikát és a parancssori futtatást.

Például:

```text
src/calculator.js → functions
src/index.js      → CLI entry point
```

Így a `calculator.js` importálása nem futtat automatikusan `console.log()` hívást.

Ez nem szükséges a jelenlegi feladat teljesítéséhez, de tisztább alkalmazásfelépítés.

---

# 5. Jest telepítése és az első teszt

Telepítés:

```bash
npm install --save-dev jest
```

A `--save-dev` hatására a Jest a `devDependencies` közé kerül.

A Jest egy JavaScript testing framework.

A teszt célja, hogy ellenőrizzük, egy függvény a várt eredményt adja-e.

## Tesztfájl

`test/calculator.test.js`:

```javascript
const { add } = require("../src/calculator");

test("2 + 3 should equal 5", () => {
  expect(add(2, 3)).toBe(5);
});
```

### `require()`

Betölti a tesztelendő függvényt.

### `test()`

Meghatároz egy tesztesetet.

Első argumentuma a teszt neve, a második pedig a végrehajtandó függvény.

### `expect()`

Megadja a ténylegesen vizsgált értéket.

### `toBe()`

Ellenőrzi a várt értékkel való egyezést, a Jest `Object.is` alapú összehasonlításával.

A teszt logikája:

```text
Input: 2, 3
     ↓
add(2, 3)
     ↓
Actual result: 5
     ↓
Expected result: 5
     ↓
PASS
```

## Teszt futtatása

```bash
npm test
```

A tényleges eredmény:

```text
PASS test/calculator.test.js
✓ 2 + 3 should equal 5

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
```

A teszt sikeresen lefutott.

**Fontos:** a Jest működését később külön jegyzetben részletesebben is feldolgozzuk.

### DevOps-kapcsolat

DevOps környezetben nem feltétlenül mi írjuk az összes unit tesztet, de gyakran nekünk kell biztosítani, hogy ezek automatikusan lefussanak a CI pipeline-ban.

**English:**
> The test checks if the function returns the correct result.

*A teszt ellenőrzi, hogy a függvény helyes eredményt ad-e.*

---

# 6. `.gitignore`

A `.gitignore` fájl megadja, mely fájlokat és könyvtárakat hagyja figyelmen kívül a Git, ha azok még nincsenek követés alatt.

Tartalom:

```gitignore
node_modules/
.env
```

## Miért nem commitoljuk a `node_modules` könyvtárat?

A `node_modules` telepített csomagokat tartalmaz.

Ezeket a `package.json` és `package-lock.json` segítségével újra lehet telepíteni.

```bash
npm ci
```

A `node_modules` sok fájlt tartalmazhat, ezért feleslegesen növelné a repository méretét.

A `package-lock.json` viszont általában **kerüljön be a Gitbe**, mivel rögzíti a telepítési függőségi gráfot.

A `.env` fájl gyakran konfigurációs értékeket, akár titkokat is tartalmazhat.

**Fontos:** a `.gitignore` nem távolítja el automatikusan a már követett fájlokat a Gitből.

---

# 7. Git repository inicializálása

Parancs:

```bash
git init
```

Ez létrehozza a projektben a `.git` könyvtárat.

A `.git` tárolja többek között:
- a Git objektumokat,
- a commitokhoz és branchekhez tartozó referenciákat,
- a repository konfigurációját,
- az indexet (*staging area*).

Állapot ellenőrzése:

```bash
git status
```

Branchek megtekintése:

```bash
git branch
```

A fő branch neve legyen `main`.

Szükség esetén:

```bash
git branch -M main
```

A `-M` a branch átnevezését kényszerítve is lehetővé teszi; ezért olyan helyzetben használjuk, ahol biztosan ezt akarjuk.

---

# 8. Git identity

A Git minden commitban rögzíti a szerző adatait.

Ellenőrzés:

```bash
git config user.name
git config user.email
```

Konfiguráció és annak forrása:

```bash
git config --list --show-origin
```

A `--show-origin` megmutatja, hogy az egyes beállításokat mely konfigurációs fájlokból olvasta a Git.

## Local vs global configuration

Global beállítás:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Local beállítás:

```bash
git config user.email "work@example.com"
```

Az utóbbi csak az aktuális repositoryra vonatkozik.

**Fontos különbség:**

A Git identity nem azonos a GitHub authenticationnel.

- **Identity:** ki a commit szerzője?
- **Authentication:** ki próbál hozzáférni a GitHubhoz?
- **Authorization:** mire van jogosultsága?

---

# 9. Staging és első commit

A Git egyik legfontosabb folyamata:

```text
Working directory
        ↓ git add
Staging area / Index
        ↓ git commit
Local repository history
```

A working directory a projekt aktuális fájljait tartalmazza.

A staging area a következő commit előkészített tartalmát reprezentálja.

## Fájlok stagingelése

```bash
git add .
```

A `.` az aktuális könyvtárat jelenti; a Git itt és az alkönyvtárakban található változásokat készíti elő, a Git szabályainak megfelelően.

Ellenőrzés:

```bash
git status
```

A megfelelő fájlok a `Changes to be committed` részben jelennek meg.

## Első commit

```bash
git commit -m "Initial Node.js project"
```

A `-m` jelentése itt **message**.

A commit a staged állapotból új verziót készít.

Ellenőrzés:

```bash
git log --oneline
git status
```

Az első commit későbbi rövid hash-e a laborban:

```text
02857cf initial Node.js project
```

**English:**
> A commit records a version of the project.

*A commit rögzíti a projekt egy verzióját.*

---

# 10. Második commit

A README fájlt kibővítettük egy `Features` szekcióval.

Például:

```markdown
## Features

- Addition
- Automated unit tests
```

Staging:

```bash
git add README.md
```

Commit:

```bash
git commit -m "Update README"
```

A labor tényleges második commitja:

```text
96f7bee Update README.md
```

A Git history:

```text
02857cf ← 96f7bee
```

A nyíl itt azt jelzi, hogy a második commit az elsőt tartalmazza parent referenciaként.

---

# 11. Commit gráf és HEAD

Parancs:

```bash
git log --oneline --graph --decorate --all
```

Kapcsolók:

- `--oneline`: rövid commitmegjelenítés
- `--graph`: ASCII commitgráf
- `--decorate`: branch- és tagnevek
- `--all`: minden releváns referencia által elérhető előzmény

Például:

```text
* 96f7bee (HEAD -> main) Update README.md
* 02857cf initial Node.js project
```

## Mit jelent a HEAD?

A `HEAD` normál esetben az aktuális branchre mutató szimbolikus referencia.

```text
HEAD
  ↓
main
  ↓
96f7bee
  ↓
02857cf
```

A `main` nem tárolja külön az összes commitot. A branch neve egy adott commitra mutat.

A commitok pedig a parent kapcsolatokkal összekapcsolódnak.

---

# 12. GitHub repository létrehozása

A GitHubon elkészítettük a következő repositoryt:

```text
git-github-actions-lab
```

A GitHub egy Git repository hosting platform, amely együttműködési és automatizációs funkciókat is biztosít.

Például:
- Issues
- Pull Requests
- Code Review
- GitHub Actions
- Branch protection
- Releases

A repositoryt kezdetben üresen kellett létrehozni, mert localban már volt README és Git history.

---

# 13. Remote beállítása

A GitHub repositoryt a local repositoryhoz kapcsoljuk.

```bash
git remote add origin <repository-url>
```

Az `origin` a remote neve, nem egy külön GitHub-szolgáltatás.

Ellenőrzés:

```bash
git remote -v
```

A parancs a remote URL-eket mutatja fetch és push irányban.

## Local és remote

```text
WSL Ubuntu
└── Local Git repository
        │
        │ push / fetch
        ↓
GitHub
└── Remote Git repository
```

**English:**
> Origin is the name of our remote repository.

*Az origin a távoli repositorynk neve.*

---

# 14. Első push és upstream

Az első `main` push tipikus parancsa:

```bash
git push -u origin main
```

Felbontva:

- `git push`: commitok és referenciák küldése a remote felé
- `-u`: upstream kapcsolat beállítása
- `origin`: remote neve
- `main`: pusholandó branch

Az upstream kapcsolat segítségével a Git tudja, mely remote branchhez kapcsolódik a local branch.

Ellenőrzés:

```bash
git branch -vv
```

Például:

```text
* main 96f7bee [origin/main] Update README.md
```

## Mi az `origin/main`?

Az `origin/main` egy **remote-tracking reference**, amely localban tárolja a Git által utoljára ismert távoli `main` állapotát.

Ez nem élő kapcsolat.

```bash
git fetch
```

frissítheti az `origin/main` referenciát a távoli repository alapján.

---

# 15. GitHub Issue

Ez az első igazán fontos GitHub-együttműködési fogalom.

## Mi az Issue?

Az **Issue** egy nyilvántartott feladat, probléma, fejlesztési igény vagy megbeszélendő kérdés.

Nem maga a kódmódosítás.

Példa a laborból:

```text
Title: Add subtraction feature
```

Leírás:

```markdown
Add a subtract(a, b) function to the calculator.

Acceptance criteria:

- subtract(5, 2) returns 3
- unit test is added
- existing tests still pass
```

Az **acceptance criteria** az elfogadási feltételeket jelenti.

Vagyis előre rögzítjük, milyen feltételek teljesülése esetén tekinthető késznek a feladat.

## Miért jó az Issue?

Egy csapatban nem minden feladat kezdődik azzal, hogy a fejlesztő egyszerűen kódolni kezd.

Először tudni kell:
- mit kell elkészíteni,
- miért van rá szükség,
- ki foglalkozik vele,
- milyen feltételeknek kell megfelelnie.

### Példa valós DevOps környezetből

```text
Issue #24

Title:
Add health check to the application

Description:
The monitoring system needs an endpoint
to check if the application is healthy.

Acceptance criteria:
- /health endpoint exists
- returns HTTP 200 when healthy
- automated test is added
```

Az Issue tehát a munka nyomon követhető kiindulópontja.

**English:**
> An Issue describes a task or a problem.

*Az Issue leír egy feladatot vagy problémát.*

---

# 16. Feature branch

A laborban a kivonás funkcióhoz külön branchet kellett létrehozni.

```text
feature/subtraction
```

## Miért dolgozunk külön branchen?

Mert a `main` branchet általában stabil állapotban szeretnénk tartani.

A fejlesztő egy külön branchen dolgozik, amelyet később ellenőrzés után lehet beolvasztani.

Általános folyamat:

```text
main
  │
  ├── feature/subtraction
  │       ↓
  │   development
  │       ↓
  │     tests
  │       ↓
  │      PR
  │       ↓
  └──── merge
```

Feature branch létrehozása:

```bash
git switch -c feature/subtraction
```

Ez létrehozza a branchet, és át is vált rá.

Alternatíva:

```bash
git branch feature/subtraction
git switch feature/subtraction
```

A kettő együtt hasonló eredményt ad.

---

# 17. A labor valódi hibája – commit rossz branchre

A labor során véletlenül másik terminálban, rossz repositoryban hoztuk létre a feature branchet.

Az eredeti projektben emiatt továbbra is a `main` branchen dolgoztunk.

A `subtract()` függvényt elkészítettük, majd commitoltuk.

A probléma:

**Az `Add subtraction feature` commit a `main` branchre került a `feature/subtraction` helyett.**

Ez volt a tényleges Git állapot:

```text
* 913aeaf (HEAD -> main) Add subtraction feature
* 96f7bee (origin/main) Update README.md
* 02857cf initial Node.js project
```

A `git status` üzenete:

```text
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
nothing to commit, working tree clean
```

## Mit jelentett ez?

- A `HEAD` a local `main` branchre mutatott.
- A local `main` a `913aeaf` commitra mutatott.
- Az `origin/main` még a `96f7bee` commitnál állt.
- Az új commit még nem volt pusholva.
- A working tree tiszta volt.

Ez egy biztonságosan helyreállítható helyzet volt.

## 17.1. A branch és a commit kapcsolata

A Git commitok objektumok.

Egy commit többek között tartalmazza:
- a projekt fájlfájának referenciáját,
- a parent commit referenciáját,
- a szerzői adatokat,
- a commit üzenetét.

A branch egy névvel rendelkező, módosítható referencia egy commitra.

Egyszerű példa:

```text
HEAD
 ↓
main
 ↓
C
 ↓
B
 ↓
A
```

Ha létrehozunk egy másik branchet:

```bash
git branch feature/test
```

akkor:

```text
       main
         ↓
         C
         ↑
    feature/test
```

A commit nem másolódik le.

**Két branch mutathat ugyanarra a commitra.**

## 17.2. Helyreállítás – feature branch létrehozása

Az első lépés:

```bash
git branch feature/subtraction
```

Mivel a `main` akkor a `913aeaf` commiton állt, az új branch is erre a commitra kezdett mutatni.

Állapot:

```text
               main
                 ↓
02857cf ← 96f7bee ← 913aeaf
                 ↑       ↑
           origin/main  feature/subtraction
```

Ebben az állapotban a `HEAD` továbbra is a `main` branchre mutatott.

**Fontos:** a `git branch` önmagában nem vált át az új branchre.

## 17.3. A main helyreállítása

Ezután futtattuk:

```bash
git reset --hard origin/main
```

### Mit jelent a parancs?

- `git reset`: az aktuális branch pozíciójának módosítása
- `--hard`: a Git indexét és a working tree-t is a célcommit állapotához igazítja
- `origin/main`: a célcommitot megadó referencia

Mivel ekkor még a `main` branchen voltunk, a Git **a local `main` pointerét** mozgatta vissza.

A cél az a commit volt, amelyre az `origin/main` mutatott.

Ez a következő volt:

```text
96f7bee
```

Az eredmény:

```text
                  feature/subtraction
                          ↓
02857cf ← 96f7bee ← 913aeaf
              ↑
           main
           origin/main
```

A `913aeaf` commit **nem veszett el**, mert a `feature/subtraction` branch továbbra is rá mutatott.

Ezután:

```bash
git switch feature/subtraction
```

A `HEAD` átkerült a feature branchre.

## 17.4. Miért működött a reset?

A `reset` nem általánosan az `origin`-t állítja vissza.

A megadott commitra állítja az aktuális branch pozícióját.

Ebben az esetben:

```text
Current branch: main
Target: origin/main
```

Tehát:

```text
main → 96f7bee
```

Ugyanezt a célcommitot közvetlen hash segítségével is megadhattuk volna:

```bash
git reset --hard 96f7bee
```

Az adott pillanatban a két parancs eredménye ugyanaz lett volna.

**Fontos biztonsági megjegyzés:** a `--hard` eldobhat nem mentett fájlmódosításokat és staged változtatásokat. Itt azért volt alkalmazható, mert a working tree tiszta volt, a commitot már megtartottuk egy másik branchen, és még nem pusholtuk.

## 17.5. Tanulság

A hibát azért tudtuk biztonságosan javítani, mert először megvizsgáltuk az állapotot.

Hasznos ellenőrzés commit előtt:

```bash
pwd
git branch --show-current
git status
```

A `pwd` megmutatja, melyik könyvtárban vagyunk.

A `git branch --show-current` megmutatja az aktuális branch nevét.

A `git status` megmutatja a repository állapotát.

**English:**
> I committed the change to the wrong branch.

*Rossz branchre commitoltam a változtatást.*

> I created a feature branch to keep the commit.

*Létrehoztam egy feature branchet, hogy megtartsam a commitot.*

> I reset the local main branch to origin/main.

*Visszaállítottam a local main branchet az origin/main állapotára.*

---

# 18. A subtraction feature és annak publikálása

A feature branch most már tartalmazta a `subtract()` függvényt és a hozzá kapcsolódó módosításokat.

Példa:

```javascript
function subtract(a, b) {
  return a - b;
}
```

Export:

```javascript
module.exports = {
  add,
  subtract,
};
```

Tesztpélda:

```javascript
test("5 - 2 should equal 3", () => {
  expect(subtract(5, 2)).toBe(3);
});
```

A tesztfájl elején a `subtract` függvényt is importálni kell.

Ellenőrzés:

```bash
npm test
```

A feature branchen lévő commit már készen állhatott a GitHubra küldésre.

## Feature branch push

```bash
git push -u origin feature/subtraction
```

Eredmény:

```text
Local repository:
  main
  feature/subtraction

GitHub repository:
  main
  feature/subtraction
```

Ellenőrzés:

```bash
git branch -vv
git branch -r
```

A `git branch -r` a remote-tracking brancheket listázza.

**English:**
> I pushed my feature branch to GitHub.

*Feltöltöttem a feature branchemet a GitHubra.*

---

# 19. Pull Request létrehozása

A Pull Request rövidítése: **PR**.

## 19.1. Mi az a Pull Request?

A Pull Request egy javaslat arra, hogy egy branch változtatásait integráljuk egy másik branchbe.

Például:

```text
feature/subtraction
        │
        │ Pull Request
        ↓
       main
```

Fontos:

**A Pull Request létrehozása még nem merge.**

Csak megnyitottunk egy változtatási javaslatot, amelyet ellenőrizni lehet.

## 19.2. Mi a base és head?

A PR-ban két branch van:

**Base branch**

Az a branch, amelybe szeretnénk integrálni a változtatásokat.

Nálunk:

```text
main
```

**Head branch**

Az a branch, amelynek változtatásait szeretnénk integrálni.

Nálunk:

```text
feature/subtraction
```

Egyszerűen:

```text
HEAD / SOURCE                  BASE / TARGET

feature/subtraction  ───────►  main
        változás                cél
```

Ez nem ugyanaz a fogalomhasználat, mint amikor a local Git `HEAD` referenciájáról beszélünk, bár a PR felület is a `head` szót használja.

## 19.3. Mit csinál a GitHub PR létrehozásakor?

A GitHub összehasonlítja a két branchet.

Megvizsgálhatóvá teszi például:
- a hozzáadott és módosított fájlokat,
- a commitokat,
- a változtatások különbségeit (*diff*),
- a megjegyzéseket,
- az automatikus ellenőrzéseket.

A Pull Request segít abban, hogy a változtatásokat ne ellenőrzés nélkül integráljuk.

## 19.4. A labor Pull Requestje

PR cím:

```text
Add subtraction feature
```

Base:

```text
main
```

Head:

```text
feature/subtraction
```

A leírás tartalmazhatja:

```markdown
## Summary

Add a subtraction function to the calculator.

## Changes

- Add subtract(a, b)
- Add a unit test
- Keep existing functionality working

Closes #1
```

A `#1` csak példa: a laborban a tényleges Issue számát kell használni.

## 19.5. Mit jelent a `Closes #1`?

Ez a Pull Requestet összekapcsolja az adott Issue-val.

Ha a PR-t az alapértelmezett branchbe megfelelően merge-elik, a GitHub a hivatkozott Issue-t automatikusan lezárhatja.

A kapcsolat:

```text
Issue #1
"Add subtraction feature"
        │
        │ work is implemented
        ↓
Pull Request
"Add subtraction feature"
        │
        │ review + checks
        ↓
Merge into main
        │
        ↓
Issue #1 closed
```

Ez egy fontos nyomonkövethetőségi kapcsolat.

Látható, hogy:
- miért indult a fejlesztés,
- mely PR oldotta meg,
- mely commitok tartalmazták a változtatást.

## 19.6. Issue vs Pull Request

| Tulajdonság | Issue | Pull Request |
|---|---|---|
| Fő cél | Feladat/probléma nyilvántartása | Kódváltoztatás integrálásának javaslata |
| Tartalmazhat kódváltoztatást? | Nem ez a szerepe | Igen, branchek közti változásokat mutat |
| Lehetnek hozzászólások? | Igen | Igen |
| Kapcsolódhat másikhoz? | Igen | Igen |
| Lehet lezárni? | Igen | Igen, merge vagy lezárás útján |
| DevOps-felhasználás | Feladatkezelés, incidensek, fejlesztési igények | Konfigurációk, infrastruktúra és alkalmazáskód felülvizsgálata |

## 19.7. Miért fontos ez DevOps területen?

DevOps és Platform Engineering környezetben nemcsak alkalmazáskódot kezelünk Gitben.

Például:
- Dockerfile
- Docker Compose konfiguráció
- Nginx konfiguráció
- Terraform kód
- Kubernetes manifestek
- GitHub Actions workflow-k

Ezeken a változtatásokon is dolgozhatunk feature branchen, majd Pull Requesten keresztül integrálhatjuk őket.

Példa:

```text
Issue:
Enable HTTPS for the application
          ↓
Branch:
feature/https-configuration
          ↓
Change:
Nginx configuration
          ↓
Pull Request
          ↓
Review + automated checks
          ↓
Merge
          ↓
Deployment process
```

Ez a megközelítés segíti az ellenőrizhetőséget, a csapatmunkát és a változtatások visszakövethetőségét.

## 19.8. A Pull Request fő részei

### Conversation

Itt találhatók a PR leírása, hozzászólások és az események.

### Commits

Itt láthatók a feature branch PR-hoz tartozó commitjai.

### Files changed

Itt látható a változtatások összehasonlítása.

Ez különösen fontos a **code review** során.

### Checks

Itt jelenhetnek meg az automatikus ellenőrzések eredményei.

Például később:

```text
GitHub Actions
    ↓
npm ci
    ↓
npm test
    ↓
PASS / FAIL
```

Jelenleg még nem építettük ki a teljes GitHub Actions CI folyamatot.

Ezért nem feltétlenül jelenik meg működő automatizált check.

## 19.9. Mi történik a PR megnyitása után?

A következő munkafolyamat:

```text
Open Pull Request
        ↓
Code Review
        ↓
Requested changes
        ↓
New commits
        ↓
Automated checks
        ↓
Approval / acceptance
        ↓
Merge
```

Az automatizált ellenőrzések és jóváhagyások konkrét követelményei a repository beállításaitól függenek.

Egy új commit ugyanazon PR head branchre történő pusholása általában frissíti a meglévő Pull Requestet.

Nem kell minden commit után új PR-t létrehozni.

**English:**
> I opened a Pull Request to merge my feature into main.

*Megnyitottam egy Pull Requestet, hogy a fejlesztésemet beolvasszuk a mainbe.*

> The Pull Request is waiting for review.

*A Pull Request ellenőrzésre vár.*

---

# 20. Állapot a mai labor végén

A feladatok alapján a következő állapotot céloztuk meg:

```text
Local repository
│
├── main
│     └── base project
│
└── feature/subtraction
      └── subtraction feature

                 PUSH
                   ↓

GitHub
│
├── main
│
├── feature/subtraction
│
├── Issue
│     └── Add subtraction feature
│
└── Open Pull Request
      ├── Base: main
      ├── Head: feature/subtraction
      └── Status: awaiting review
```

A Pull Request nyitva van, de még nem történt meg a merge.

---

# 21. Fontos Git-parancsok – összefoglalás

| Parancs | Feladat |
|---|---|
| `git init` | Repository inicializálása |
| `git status` | Aktuális állapot |
| `git add .` | Változtatások stagingelése |
| `git commit -m "..."` | Commit készítése |
| `git log --oneline` | Rövid history |
| `git branch` | Local branchek |
| `git branch -a` | Local és remote-tracking branchek |
| `git branch -vv` | Branchek és upstream kapcsolatok |
| `git branch feature/x` | Branch létrehozása |
| `git switch feature/x` | Branchváltás |
| `git switch -c feature/x` | Branch létrehozása és váltás |
| `git reset --hard origin/main` | Aktuális branch és munkafájlok célcommitra állítása |
| `git remote -v` | Remote-ok ellenőrzése |
| `git push -u origin main` | Main push és upstream |
| `git push -u origin feature/x` | Feature branch push és upstream |
| `git fetch` | Remote-tracking referenciák frissítése |
| `git log --oneline --graph --decorate --all` | Commitgráf |

**Megjegyzés:** a táblázatban szereplő veszélyes parancsokat, különösen a `reset --hard` használatát mindig az aktuális repository állapota alapján kell eldönteni.

---

# 22. Fontos angol szókincs

| English | Magyar |
|---|---|
| repository | verziókezelt tárhely |
| commit | mentett Git-verzió |
| branch | fejlesztési ág / commitra mutató referencia |
| pointer / reference | mutató / referencia |
| current branch | aktuális branch |
| parent commit | szülőcommit |
| staging area | előkészítési terület |
| working tree | munkafájlok aktuális állapota |
| remote | távoli repository |
| upstream | követett távoli branch |
| push | feltöltés |
| fetch | távoli változások lekérése |
| Issue | feladat / probléma |
| acceptance criteria | elfogadási feltételek |
| Pull Request | beolvasztási javaslat |
| code review | kódellenőrzés |
| base branch | célbranch |
| head branch | forrásbranch a PR-ban |
| merge | ágak egyesítése |
| automated checks | automatikus ellenőrzések |
| test suite | tesztkészlet |
| passed | sikeresen teljesült |
| failed | sikertelen |
| troubleshooting | hibakeresés |

---

# 23. Ellenőrző kérdések

A jegyzet ismétlésekor próbáld ezeket saját szavaiddal megválaszolni.

1. Mi a különbség a Git és a GitHub között?
2. Mi történik `git init` után?
3. Mire szolgál a `.gitignore`?
4. Mi a staging area?
5. Mit tartalmaz egy commit?
6. Hogyan kapcsolódnak egymáshoz a commitok?
7. Miért mondjuk, hogy a branch egy pointer?
8. Mi a HEAD?
9. Mi a különbség a `git branch` és a `git switch` között?
10. Miért maradt meg a subtraction commit a reset után?
11. Mit jelent az `origin/main`?
12. Mi a különbség a local és a remote branch között?
13. Mi az upstream?
14. Mire szolgál a GitHub Issue?
15. Mik az acceptance criteria?
16. Mi az a Pull Request?
17. Mi a különbség a PR base és head branch között?
18. Mit jelent a `Closes #1`?
19. Miért nem merge-eljük automatikusan az összes változtatást?
20. Miért hasznos a code review és az automatizált tesztelés?

---

# 24. Folytatás a következő alkalommal

**Következő lépés: 20. Code Review szimuláció.**

A nyitott Pull Requesten gyakoroljuk:

1. A `Files changed` fül értelmezését.
2. A commitok és a változtatások ellenőrzését.
3. Review megjegyzés írását.
4. Új teszt hozzáadását a feature branchhez.
5. A már nyitott PR frissítését új committal.
6. A `fetch` és `origin/main` működésének további gyakorlását.

**Fontos: a Pull Requestet még nem merge-eljük.**

A következő alkalommal innen folytatjuk.

---

# 25. A labor legfontosabb tanulsága eddig

A Git használatánál mindig tudnunk kell:

```text
WHERE AM I?
     ↓
WHICH BRANCH?
     ↓
WHAT CHANGED?
     ↓
WHAT IS STAGED?
     ↓
WHAT IS COMMITTED?
     ↓
WHAT IS PUSHED?
```

Magyarul:

```text
Hol vagyok?
     ↓
Melyik branchen?
     ↓
Mi változott?
     ↓
Mi van stagingelve?
     ↓
Mi van commitolva?
     ↓
Mi került a GitHubra?
```

**A cél nem a Git-parancsok bemagolása, hanem a repository állapotának megértése.**

Ez a szemlélet a későbbi DevOps, CI/CD és infrastruktúra-üzemeltetési feladatoknál is alapvető lesz.



# 20. Code Review szimuláció

A Pull Request már elkészült.

A következő feladat annak szimulálása, hogy most nem fejlesztőként, hanem **reviewerként** vizsgáljuk meg a változtatást.

A Pull Requestben ehhez elsősorban a:

```text
Files changed
```

fület használjuk.

---

## 20.1. Mi a Code Review?

A **Code Review** során egy másik fejlesztő vagy csapattag ellenőrzi a módosításokat, mielőtt azok bekerülnek a célbranchbe.

Tipikus workflow:

```text
Developer
   ↓
feature branch
   ↓
Pull Request
   ↓
Reviewer
   ↓
Code Review
   ↓
comments / approval / requested changes
   ↓
merge
```

A reviewer például ezeket nézi:

```text
érthető-e a kód?
jók-e a nevek?
van-e megfelelő teszt?
nem került-e be felesleges fájl?
nem került-e be secret?
megfelel-e a feladat követelményeinek?
```

---

## 20.2. Conversation comment és Review comment

A Pull Requestben többféle komment létezik.

### Conversation comment

A PR `Conversation` részében írt komment egy általános hozzászólás.

Például:

```text
I think we should add another test.
```

Ez teljesen érvényes kommunikáció, de nem ugyanaz, mint a formális Code Review.

Mental model:

```text
Conversation comment
=
general Pull Request discussion
```

---

### Line comment a Files changed részen

A:

```text
Files changed
```

fülön konkrét kódsorhoz lehet megjegyzést írni.

Például:

```text
Could we also test negative numbers?
```

Ez közvetlenül az adott kódrészlethez kapcsolódik.

Több line comment is összegyűjthető egy review részeként, majd egyszerre elküldhető a:

```text
Submit review
```

gombbal.

---

## 20.3. Submit review lehetőségei

Normál esetben egy reviewer a review végén háromféle eredményt választhat:

```text
Comment
Approve
Request changes
```

### Comment

A reviewer megjegyzéseket küld, de nem ad hivatalos jóváhagyást és nem blokkolja formálisan a PR-t.

Példa:

```text
Looks good overall.
```

---

### Approve

A reviewer hivatalosan jóváhagyja a Pull Requestet.

Jelentése:

> Átnéztem a változtatásokat, és szerintem merge-elhetők.

**English:**

> I reviewed the changes and approved the Pull Request.

---

### Request changes

A reviewer azt jelzi, hogy a PR jelenlegi állapotában még módosítást igényel.

Példa:

```text
Please add a test for negative numbers before merging.
```

---

## 20.4. Fontos: a saját Pull Requestet nem lehet Approve-olni

A laborban ugyanazzal a GitHub felhasználóval:

- létrehoztuk a Pull Requestet,
- és reviewerként is ugyanazzal a felhasználóval próbáltuk átnézni.

Ez fontos különbséget okoz.

A GitHub **nem engedi, hogy a Pull Request szerzője saját magának hivatalos approvalt adjon**.

Ezért a saját PR esetén előfordulhat, hogy a:

```text
Submit review
```

ablakban csak:

```text
Comment
```

választható.

Az:

```text
Approve
```

nem érhető el.

Ez nem hiba, hanem GitHub-szabály.

---

## 20.5. Miért van ez így?

A Code Review lényege az, hogy egy **másik ember** ellenőrizze a változtatást.

Ha a PR szerzője saját magának approvalt adhatna:

```text
Developer
↓
creates PR
↓
approves own PR
```

akkor a review nem jelentene valódi független ellenőrzést.

Valós csapatban:

```text
Developer A
↓
creates Pull Request

Developer B
↓
reviews Pull Request
↓
Approve / Request changes
```

---

## 20.6. Mit csináltunk a laborban?

Mivel egyedül dolgozunk, a Code Review-t csak **szimulálni** tudjuk.

A laborban ezt csináltuk:

```text
Pull Request
↓
Files changed
↓
line comments
↓
Submit review
↓
Comment
```

A végső review comment lehet például:

```text
Everything looks good. The changes are ready to merge.
```

Ez tartalmilag azt szimulálja, hogy a reviewer szerint a PR rendben van.

Viszont a GitHub nem fogja:

```text
Approved
```

státuszba tenni, mert ugyanaz a felhasználó a PR szerzője is.

---

## 20.7. Saját PR vs másik reviewer

### Saját Pull Request

```text
Author
↓
Files changed
↓
line comments
↓
Submit review
↓
Comment
```

Formális approval:

```text
nem adható
```

---

### Másik reviewer

```text
Developer A creates PR
↓
Developer B reviews
↓
Files changed
↓
Submit review
↓
Comment / Approve / Request changes
```

Itt már valódi:

```text
Approved
```

státusz is létrejöhet.

---

## 20.8. A mi laborunk helyes megoldása

A 20. feladatot ebben az egyszemélyes laborban így tekintjük teljesítettnek:

```text
1. Pull Request megnyitása
2. Files changed fül
3. kód átnézése
4. line commentek hozzáadása
5. Submit review
6. Comment kiválasztása
```

Például:

```text
Everything looks good. The changes are ready to merge.
```

A valódi csapathelyzetben ezt egy másik reviewer akár:

```text
Approve
```

review-ként küldené el.

---

## 20.9. DevOps kapcsolat

Code Review nem csak alkalmazáskódnál fontos.

DevOps környezetben review tárgya lehet például:

```text
Dockerfile
docker-compose.yml
Terraform
Kubernetes YAML
Nginx configuration
GitHub Actions workflow
shell script
```

Példa:

```text
Engineer changes Terraform
        ↓
Pull Request
        ↓
another engineer reviews
        ↓
CI checks
        ↓
Approve
        ↓
merge
```

---

# 21–24.

A 21–24. feladatok megoldása változatlanul a korábban kidolgozott rész szerint folytatódik.