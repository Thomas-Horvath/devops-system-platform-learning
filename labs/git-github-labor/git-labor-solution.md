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

# Git + GitHub + GitHub Actions – komplex labor
## Megoldókulcs és magyarázat – 2. rész
### 21–29. feladat

---

# 21. A meglévő Pull Request frissítése új committal

A `feature/subtraction` branchhez már létezett egy Pull Request.

A review során felmerült, hogy érdemes lenne további tesztet hozzáadni, például negatív számokkal.

## Branch ellenőrzése

```bash
git branch --show-current
```

A kívánt branch:

```text
feature/subtraction
```

---

## Új teszt hozzáadása

Például:

```javascript
test("-2 - 3 should equal -5", () => {
  expect(subtract(-2, 3)).toBe(-5);
});
```

Ezután:

```bash
npm test
```

Ha minden teszt sikeres:

```bash
git add .
git commit -m "Add subtraction edge case test"
git push
```

---

## Mi történt a Pull Requesttel?

Nem kellett új PR-t létrehozni.

A GitHub Pull Request nem egyetlen commitot követ, hanem lényegében két branch közötti különbséget:

```text
base: main
head: feature/subtraction
```

Ha a feature branch új commitot kap:

```text
feature/subtraction
        ↓
    new commit
        ↓
      push
```

akkor a már létező PR automatikusan frissül.

### Mental model

```text
same feature branch
       ↓
new commit
       ↓
git push
       ↓
same Pull Request updates
```

**English:**

> I pushed another commit to the same Pull Request.

---

# 22. Remote változtatás és `git fetch`

A következő gyakorlatban GitHubon módosítottuk a remote `main` branchet.

A local repository azonban ezt nem látja automatikusan.

Kezdetben:

```text
local main   → A
origin/main  → A
GitHub main  → A
```

GitHubon készült egy új commit:

```text
GitHub main  → B
```

Localban viszont továbbra is:

```text
local main   → A
origin/main  → A
```

---

## `git fetch`

```bash
git fetch
```

Ezután:

```text
local main   → A
origin/main  → B
GitHub main  → B
```

A fontos rész:

> A `git fetch` nem mozgatja el automatikusan a local `main` branchet.

Csak frissíti azt, amit a local Git a remote állapotáról tud.

---

## Mi az `origin/main`?

Az:

```text
origin/main
```

egy **remote-tracking reference**.

Nem maga a GitHub branch.

A local repositoryban tárolt információ arról, hogy a Git legutóbbi tudása szerint hol áll az `origin` remote `main` branchje.

A:

```bash
git fetch
```

frissíti ezt az információt.

---

## `fetch` vs `pull`

Egyszerűsítve:

```text
git fetch
=
remote információ letöltése
```

míg:

```text
git pull
≈
fetch
+
változások integrálása
```

Ezért hibakereséskor gyakran jó stratégia:

```bash
git fetch
git log --oneline --graph --decorate --all
```

Először megnézzük, mi változott, és csak utána döntünk arról, hogyan akarjuk integrálni.

**English:**

> Fetch updates my view of the remote repository.

---

# 23. Remote branch és local tracking branch

GitHubon létrehoztuk:

```text
feature/multiply
```

A local Git ezt először még nem ismerte.

Ezután:

```bash
git fetch
git branch -r
```

Megjelent:

```text
origin/feature/multiply
```

---

## Fontos különbség

Ez:

```text
origin/feature/multiply
```

nem az a normál local branch, amin dolgozunk.

Ez egy:

```text
remote-tracking reference
```

A local munkához létrehoztuk:

```bash
git switch -c feature/multiply --track origin/feature/multiply
```

---

## Mit jelent ez?

```bash
-c feature/multiply
```

Létrehoz egy új local branchet.

```bash
--track origin/feature/multiply
```

Beállítja az upstream/tracking kapcsolatot.

Mental model:

```text
GitHub:
feature/multiply
        ↓
      fetch
        ↓
local Git:
origin/feature/multiply
        ↑
      tracks
        ↑
feature/multiply
```

---

## Ellenőrzés

```bash
git branch -vv
```

Példa:

```text
feature/multiply abc123 [origin/feature/multiply]
```

A:

```text
[origin/feature/multiply]
```

mutatja az upstream kapcsolatot.

---

# 24. Merge conflict gyakorlat

A cél az volt, hogy szándékosan létrehozzunk merge conflictot.

Az első próbálkozás azonban nem konfliktussal végződött.

---

## 24.1. Első próbálkozás – Fast-forward

Először a `main` branchen módosítottuk a README-t.

Ezután ebből az állapotból hoztunk létre feature branchet, majd azon is módosítottunk.

A history:

```text
A ← B ← C ← D
        ↑       ↑
       main   feature
```

A feature a main közvetlen folytatása volt.

Ezért:

```bash
git merge feature/readme-conflict
```

eredménye:

```text
Fast-forward
```

A Git csak előremozgatta a `main` pointert.

```text
A ← B ← C ← D
                ↑
              main
```

Nem volt két külön history-vonal, ezért konfliktus sem tudott kialakulni.

---

# 24.2. Valódi merge conflict létrehozása

Ehhez mindkét branchnek ugyanabból a közös pontból külön irányba kellett fejlődnie.

```text
             feature commit
            /
common commit
            \
             main commit
```

A feature branchen ugyanazt a README-sort egyik módon írtuk át.

A main branchen ugyanezt a sort másképp.

Ezután:

```bash
git merge feature/readme-conflict-2
```

eredmény:

```text
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

---

# 24.3. Conflict marker-ek

A Git valami ilyesmit írt a fájlba:

```text
<<<<<<< HEAD
main változat
=======
feature változat
>>>>>>> feature/readme-conflict-2
```

### `<<<<<<< HEAD`

Az aktuális branch változata.

### `=======`

A két változat elválasztója.

### `>>>>>>> feature/...`

A merge-elt branch változata.

---

# 24.4. Konfliktus feloldása

Nem kötelező kizárólag az egyik változatot megtartani.

Lehet:

```text
main változat
```

vagy:

```text
feature változat
```

vagy akár teljesen új, harmadik végleges változat.

Mi a harmadik lehetőséget használtuk.

Ezután:

```bash
git add README.md
git commit -m "Resolve README merge conflict"
```

A `git add` itt nem egyszerűen „fájlt tesz stage-be”, hanem azt is jelzi a Gitnek:

> Ezt a konfliktust feloldottam.

---

# 24.5. Merge commit

A gráf:

```text
*   0dd5f47 Resolve README merge conflict
|\
| * 8735906 feature change
* | a841301 main change
|/
```

A merge commitnak két parentje van:

```text
feature history
       \
        merge commit
       /
main history
```

---

# 24.6. Új változtatás ugyanazon feature branchen

Ezután tovább dolgoztunk ugyanazon feature branchen, és egy másik README-sort módosítottunk.

Új commit készült.

Majd ismét:

```bash
git merge feature/readme-conflict-2
```

Most nem lett konfliktus.

Miért?

Mert az új változtatás összeegyeztethető volt a main tartalmával.

### Fontos tanulság

A conflict nem „ragad rá” egy branchre.

Nem ezt vizsgálja a Git:

```text
Ez a branch korábban konfliktusos volt?
```

hanem:

```text
mi a közös ancestor?
↓
mi változott mainen?
↓
mi változott feature-ön?
↓
összeegyeztethetők?
```

---

# 24.7. A három fontos merge-helyzet

### Fast-forward

```text
nincs külön main history
→ csak pointer mozog
```

### Automatic three-way merge

```text
két history-vonal van
+
a változások összeegyeztethetők
→ Git automatikusan merge-el
```

### Merge conflict

```text
két history-vonal
+
összeegyeztethetetlen változtatás
→ emberi döntés kell
```

---

# 25. Stash – félkész munka ideiglenes elrakása

Létrehoztuk:

```text
feature/division
```

branchet.

Elkezdődött a:

```javascript
divide()
```

függvény fejlesztése.

A munka azonban még nem volt kész, ezért nem akartunk commitot készíteni.

---

## Fontos felismerés: lehet branch-et váltani commit nélkül?

Igen, bizonyos esetekben.

A Git engedheti:

```bash
git switch main
```

akkor is, ha vannak nem commitolt változtatások.

Ha ezek biztonságosan átvihetők a másik branch working tree-jére, akkor a változtatások „jönnek velünk”.

Ez elsőre meglepő.

### A nem commitolt módosítás nem feltétlenül „a branch része”

```text
branch
→ commit history

working tree
→ jelenlegi, még nem commitolt fájlállapot
```

Ezért egy working tree módosítás branchváltáskor akár megmaradhat.

---

## Mikor tiltja le a Git a branchváltást?

Ha a váltás felülírná a nem mentett változtatást.

Ilyenkor például:

```text
Your local changes would be overwritten by checkout.
```

---

# 25.1. `git stash`

A félkész munkát ideiglenesen elraktuk:

```bash
git stash
```

Ezután:

```bash
git status
```

a working tree ismét tiszta lett.

Most már:

```bash
git switch main
```

biztonságosan használható.

---

## Visszatérés

```bash
git switch feature/division
```

Majd:

```bash
git stash pop
```

A félkész `divide()` módosítás újra megjelent.

---

# 25.2. Mental model

```text
unfinished work
      ↓
git stash
      ↓
temporary stash storage
      ↓
clean working tree
      ↓
switch branch
      ↓
return to feature branch
      ↓
git stash pop
      ↓
unfinished work restored
```

---

## Stash vs commit

```text
commit
→ a projekt history része
```

```text
stash
→ ideiglenes félrerakás
→ nem normál project history
```

Ez nagyon hasznos például:

- sürgős hotfix miatt,
- branchváltás előtt,
- gyors kísérlet előtt,
- félkész munka elrakására.

**English:**

> I stashed my unfinished changes.

---

# 26. Hibás local commit és `git reset --soft`

A `divide()` függvényt befejeztük és commitoltuk.

Ez lett a jó commit.

Ezután a README-ben szándékosan hibás módosítást készítettünk.

Például:

```text
THIS IS A BAD CHANGE
```

Majd ezt is commitoltuk.

Fontos:

> A hibás commit még nem volt pusholva.

---

## History

```text
A
↓
Add division function
↓
Bad README change
```

A HEAD a hibás commitra mutatott.

---

# 26.1. Soft reset

Mivel csak az utolsó commitot akartuk visszavonni:

```bash
git reset --soft HEAD~1
```

vagy konkrét commit hashre:

```bash
git reset --soft <division-commit>
```

---

## Mit jelent a `HEAD~1`?

```text
HEAD
=
aktuális commit
```

```text
HEAD~1
=
az aktuális commit első parentje
```

Egyszerű lineáris history esetén:

```text
egy committal korábban
```

---

# 26.2. Mi történt soft reset után?

Előtte:

```text
division commit
      ↓
bad README commit
      ↑
     HEAD
```

Utána:

```text
division commit
      ↑
     HEAD
```

A hibás README változtatás azonban nem veszett el.

Megmaradt a:

```text
staging area
```

területén.

Ezért:

```bash
git status
```

staged változtatásként mutatta.

---

# 26.3. A soft reset mental modelje

```text
git reset --soft
```

jelentése:

```text
move branch pointer
+
keep changes staged
```

Ez nagyon hasznos, ha:

```text
commitoltam
↓
rájöttem, hogy rossz
↓
még nem pusholtam
↓
visszalépek
↓
kijavítom
↓
új, helyes commit
```

---

# 26.4. Fontos: mikor jó a reset?

Elsősorban:

```text
local / unpublished history
```

esetén.

Ha a commit már közös remote repositoryban van, a reset + force push history átírást okozhat.

Ilyenkor általában más megoldás kell.

Ez vezetett a következő feladathoz.

---

# 27. Publikált commit visszavonása – `git revert`

A `feature/division` branchet pusholtuk GitHubra.

Ezután egy új README-változtatást készítettünk.

Commitoltuk és pusholtuk.

Tehát a rossznak bizonyult commit már:

```text
remote repository
```

része volt.

Most nem akartuk átírni a közös historyt.

---

# 27.1. `git revert`

Először:

```bash
git log --oneline
```

Megkerestük a visszavonandó commit hashét.

Majd:

```bash
git revert <commit-hash>
```

A Git nem törölte ki a régi commitot.

Ehelyett új commitot készített, amely az eredeti változtatás ellenkezőjét alkalmazta.

---

## History

```text
good state
↓
bad commit
↓
revert commit
```

Például:

```text
abc123 Bad README change
def456 Revert "Bad README change"
```

Ezután:

```bash
git push
```

---

# 27.2. Reset vs revert

Ez az egyik legfontosabb Git-különbség.

### Reset

```text
history pointer mozgatása
```

Jellemző használat:

```text
local
nem publikált commit
```

---

### Revert

```text
új commit készül
ami visszavon egy korábbi commitot
```

Jellemző használat:

```text
már publikált / közös history
```

Mental model:

```text
RESET
„tegyünk úgy, mintha ez nem lenne a branch végén”
```

```text
REVERT
„ez megtörtént, de most egy új commitban visszavonom”
```

---

## Miért jobb publikált historynál a revert?

Tegyük fel, hogy Developer A és Developer B már letöltötte:

```text
A → B → C
```

Ha valaki force push-sal átírja:

```text
A → B
```

akkor a többiek historyja már eltér.

Reverttel:

```text
A → B → C → D
```

mindenki ugyanazt a történetet látja.

**English:**

> The bad commit was already pushed.

> I reverted it instead of rewriting history.

---

# 27.3. Plusz kísérlet – régi commit megtekintése

A revert gyakorlás közben kipróbáltuk a checkoutot is.

Hash alapján visszaléptünk egy konkrét commitra:

```bash
git checkout <commit-hash>
```

Ez egy új fontos Git-fogalmat hozott elő:

# Detached HEAD

---

# 27.4. Normál HEAD állapot

Normál esetben:

```text
HEAD
 ↓
feature/division
 ↓
a95d4eb
```

A logban például:

```text
a95d4eb (HEAD -> feature/division)
```

A:

```text
HEAD -> feature/division
```

azt jelenti:

> A HEAD jelenleg a `feature/division` branchre mutat.

---

# 27.5. Detached HEAD

Ha konkrét commit hashre checkoutolunk:

```bash
git checkout a95d4eb
```

akkor a HEAD közvetlenül a commitra kerül.

```text
HEAD
 ↓
a95d4eb

feature/division
 ↓
a95d4eb
```

Mindkettő ugyanarra a commitra mutathat, de a HEAD már nincs „rákapcsolva” a branch pointerre.

Ezért a log:

```text
a95d4eb (HEAD, origin/feature/division, feature/division)
```

és nem:

```text
a95d4eb (HEAD -> feature/division, ...)
```

---

## A vessző és a nyíl jelentése

```text
HEAD -> feature/division
```

Normál branch állapot.

```text
HEAD, feature/division
```

Detached HEAD: ugyanazon a commiton vannak, de a HEAD közvetlenül a commitra mutat.

---

# 27.6. Miért kell erre figyelni?

Detached HEAD állapot önmagában nem hiba.

Nagyon hasznos régi commitok:

- megtekintésére,
- tesztelésére,
- fordítására,
- hibakeresésére.

Viszont ha detached HEAD állapotban új commitokat készítünk, azok nem egy normál branch végére kerülnek.

Ezért munkához általában visszatérünk egy branchre:

```bash
git switch feature/division
```

---

# 27.7. Fontos felismerés: a HEAD nem azt jelenti, hogy „legújabb commit”

A HEAD jelentése:

> Jelenleg hol vagyok a repositoryban?

Ha régi commitra checkoutolunk:

```bash
git checkout abc123
```

akkor:

```text
HEAD = abc123
```

Tehát a HEAD folyamatosan mozog.

---

## Visszatérés branchre

```bash
git switch feature/division
```

vagy bizonyos helyzetekben:

```bash
git switch -
```

amely visszavisz az előző branchre / pozícióra.

---

# 28. `git reflog` – a local Git mozgásnaplója

Futtattuk:

```bash
git reflog
```

A reflogban olyan műveletek jelentek meg, mint:

```text
commit
checkout
switch
reset
revert
```

---

# 28.1. `git log` vs `git reflog`

Nagyon fontos különbség.

### `git log`

A commit historyt mutatja.

```text
A → B → C
```

---

### `git reflog`

Azt mutatja, hogy a local referencia, különösen a HEAD, milyen pozíciókon járt.

Például:

```text
HEAD@{0}
HEAD@{1}
HEAD@{2}
```

---

## Mit jelent ez?

```text
HEAD@{0}
```

a jelenlegi HEAD állapot.

```text
HEAD@{1}
```

az előző.

```text
HEAD@{2}
```

az azelőtti.

---

# 28.2. Az eredeti feladaton túl: elveszett commit visszaállítása

A labor eredetileg csak a reflog megfigyelését kérte.

Mi tovább mentünk.

Készítettünk egy új commitot:

```text
Add reflog practice line
```

Majd szándékosan:

```bash
git reset --hard HEAD~1
```

paranccsal visszaléptünk.

A commit eltűnt a normál:

```bash
git log
```

kimenetéből.

---

## A commit valójában még létezett

A branch pointer már nem mutatott rá:

```text
A → B
    ↑
  branch

C
```

A `C` commit „elveszettnek” tűnt.

De a reflog még emlékezett arra, hogy a HEAD korábban ott volt.

```bash
git reflog
```

Például:

```text
abc123 HEAD@{1}: commit: Add reflog practice line
```

---

# 28.3. Visszaállítás reflogból

A commit hash alapján:

```bash
git reset --hard abc123
```

Ezután:

```text
A → B → C
        ↑
      branch
```

A commit újra a branch history végére került.

---

# 28.4. Óvatosabb recovery módszer

Valódi hibamentésnél nem feltétlenül kell rögtön mozgatni a jelenlegi branchet.

Lehet:

```bash
git branch recovery abc123
```

Ekkor:

```text
recovery
   ↓
abc123
```

létrejön egy új branch az elveszett commiton.

Ezután nyugodtan megvizsgálhatjuk, hogy valóban ezt akartuk-e visszaállítani.

---

# 28.5. Reflog mental model

```text
commit
↓
reset
↓
commit eltűnik a branch historyból
↓
git reflog
↓
régi HEAD pozíció megtalálása
↓
commit hash
↓
reset vagy recovery branch
↓
commit megmentve
```

---

# 28.6. Fontos korlát

A reflog alapvetően:

```text
local Git information
```

Nem egy GitHubon tárolt általános backup-rendszer.

Ezért:

> The reflog is a local safety net.

**English:**

> I found the lost commit in the reflog.

> I recovered the commit using its hash.

---

# 29. GitHub Actions CI létrehozása

Most elérkeztünk ahhoz a részhez, ahol a Git/GitHub workflow-t automatizáljuk.

Eddig kézzel csináltuk:

```bash
npm ci
npm test
```

A cél az, hogy ezt mostantól a GitHub automatikusan elvégezze.

---

# 29.1. Workflow helye

A repositoryban létre kell hozni:

```text
.github/
└── workflows/
    └── ci.yml
```

Fontos:

```text
.github/workflows/
```

egy speciális GitHub Actions könyvtár.

A GitHub itt keresi a workflow fájlokat.

---

# 29.2. A CI célja

A workflow induljon:

```text
push
```

és:

```text
pull_request
```

eseményre.

Majd:

```text
Ubuntu runner
↓
repository checkout
↓
Node.js setup
↓
npm ci
↓
npm test
```

---

# 29.3. Példa workflow

```yaml
name: CI

on:
  push:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
```

---

# 29.4. A workflow részei

## Workflow

Az egész:

```text
ci.yml
```

egy workflow.

Mental model:

```text
WORKFLOW
└── JOB
    └── STEPS
```

---

## Event

```yaml
on:
  push:
  pull_request:
```

Ezek az események indítják el a workflow-t.

```text
EVENT
↓
WORKFLOW
```

Például:

```text
git push
↓
GitHub receives push event
↓
CI workflow starts
```

---

## Job

```yaml
jobs:
  test:
```

A workflow egy:

```text
test
```

nevű jobot tartalmaz.

Egy workflow több jobot is tartalmazhat.

---

## Runner

```yaml
runs-on: ubuntu-latest
```

A runner az a gép/környezet, ahol a job fut.

Ebben az esetben:

```text
GitHub-hosted Ubuntu machine
```

Mental model:

```text
GitHub
↓
starts runner
↓
Ubuntu environment
↓
our commands run there
```

---

## Step

A job lépésekből áll:

```yaml
steps:
```

Például:

```text
1. checkout
2. setup Node
3. npm ci
4. npm test
```

A step tehát egy jobon belüli műveleti lépés.

---

# 29.5. `uses` vs `run`

Ez különösen fontos.

## `uses`

Például:

```yaml
uses: actions/checkout@v4
```

Azt jelenti:

> használjunk egy már elkészített GitHub Actiont.

Másik példa:

```yaml
uses: actions/setup-node@v4
```

---

## `run`

Például:

```yaml
run: npm ci
```

A runner shelljében közvetlen parancsot futtat.

Másik:

```yaml
run: npm test
```

Mental model:

```text
uses
→ reusable Action
```

```text
run
→ shell command
```

---

# 29.6. Mi az `npm ci`?

CI környezetben az:

```bash
npm ci
```

általában jobb választás, mint:

```bash
npm install
```

mert a lock file alapján reprodukálható dependency telepítést végez.

A CI-nek nem új dependency-verziókat kell keresnie.

Azt akarjuk, hogy ugyanazt az állapotot telepítse, amit a projekt meghatároz.

---

# 29.7. Az Actions mental modelje

```text
EVENT
↓
WORKFLOW
↓
JOB
↓
RUNNER
↓
STEP
↓
ACTION vagy RUN command
```

A mi példánk:

```text
push / pull_request
↓
CI
↓
test job
↓
Ubuntu runner
↓
checkout
↓
setup Node.js
↓
npm ci
↓
npm test
```

---

# 29.8. Miért CI?

Eddig:

```text
developer machine
↓
npm test
```

A teszt csak akkor futott, ha mi kézzel elindítottuk.

Most:

```text
push
↓
GitHub Actions
↓
fresh runner
↓
dependencies installed
↓
tests automatically executed
```

Ez már:

```text
Continuous Integration
```

alap.

A fontos gondolat:

> Ne csak azt higgyük el, hogy „nálam működik”.

A repository saját automatizált ellenőrzést kap.

**English:**

> The CI workflow runs the tests automatically.

---

# 21–29. Fő tanulságok egyben

```text
Pull Request update
→ ugyanaz a PR követi a feature branch új commitjait

fetch
→ remote információ frissítése localban

tracking branch
→ local branch + upstream kapcsolat

merge
→ branchek historyjának integrálása

stash
→ félkész munka ideiglenes elrakása

reset
→ branch/history pointer mozgatása

revert
→ korábbi változtatás új committal történő visszavonása

detached HEAD
→ HEAD közvetlenül commitra mutat

reflog
→ local referencia-mozgások naplója

GitHub Actions
→ repository eseményekre automatikusan futó workflow
```

---

# Reset – Revert – Reflog gyors összehasonlítás

## Reset

```text
„Mozgasd vissza a branchet.”
```

Leginkább local/unpublished historyhoz.

---

## Revert

```text
„Tartsd meg a historyt, de vond vissza ezt a változtatást.”
```

Különösen hasznos már pusholt commitoknál.

---

## Reflog

```text
„Mutasd meg, hogy localban korábban hol jártak a reference-ek.”
```

Recovery eszköz.

---

# Working tree – Staging – Commit – Stash

```text
Working Tree
    ↓ git add
Staging Area
    ↓ git commit
Repository History
```

A stash oldalirányú ideiglenes tárolóként képzelhető el:

```text
Working Tree
    ↓
git stash
    ↓
temporary stash
    ↓
git stash pop
    ↓
Working Tree
```

---

# HEAD összefoglaló

Normál állapot:

```text
HEAD
 ↓
branch
 ↓
commit
```

Detached:

```text
HEAD
 ↓
commit
```

A HEAD tehát nem azt jelenti, hogy:

```text
„a repository legújabb commitja”
```

hanem:

```text
„az a pozíció, ahol jelenleg vagyok”
```

---

# B1 szakmai angol mondatok

> I pushed another commit to the same Pull Request.

> I fetched the remote changes.

> The local branch tracks the remote branch.

> I resolved the merge conflict manually.

> I stashed my unfinished work.

> I reset the local commit.

> The bad commit was already pushed.

> I reverted the bad commit.

> My HEAD was detached from the branch.

> I switched back to the feature branch.

> I found the lost commit in the reflog.

> I recovered the lost commit.

> The CI workflow runs the tests automatically.

---

# Ellenőrző kérdések

1. Miért frissül automatikusan egy meglévő PR új feature commit után?
2. Mit csinál a `git fetch`?
3. Mi az `origin/main`?
4. Mi a remote-tracking reference?
5. Mit jelent a tracking/upstream kapcsolat?
6. Miért nem lett conflict az első merge-próbálkozásnál?
7. Mi a fast-forward?
8. Mikor keletkezhet merge conflict?
9. Mit jelent a `<<<<<<< HEAD`?
10. Mit jelent a merge commit két parentje?
11. Mire használható a `git stash`?
12. A nem commitolt working tree változás feltétlenül egy branchhez tartozik?
13. Mit csinál a `git stash pop`?
14. Mit csinál a `git reset --soft HEAD~1`?
15. Mi történik soft reset után a módosításokkal?
16. Miért jobb a reset főleg még nem pusholt commitoknál?
17. Mit csinál a `git revert`?
18. Miért nem törli ki a revert az eredeti commitot?
19. Mi a különbség reset és revert között?
20. Mit jelent a detached HEAD?
21. Miért változik `HEAD -> branch` helyett `HEAD, branch` formára a log?
22. Mit jelent valójában a HEAD?
23. Hogyan térünk vissza detached HEAD állapotból?
24. Mi a `git reflog`?
25. Mi a különbség `git log` és `git reflog` között?
26. Hogyan lehet reflogból elveszett commitot visszaállítani?
27. Miért lehet recovery branchet készíteni egy elveszett commitra?
28. Mi indítja el a GitHub Actions workflow-t?
29. Mi a job?
30. Mi a runner?
31. Mi a step?
32. Mi a különbség `uses` és `run` között?
33. Miért hasznos a CI?


# Git + GitHub + GitHub Actions – komplex labor
## Megoldókulcs és magyarázat – 3. rész
### 29–45. feladat

---

# 29. GitHub Actions CI létrehozása

A cél az volt, hogy az eddig kézzel futtatott:

```bash
npm ci
npm test
```

parancsokat a GitHub automatikusan futtassa.

Ehhez létrehoztuk:

```text
.github/workflows/ci.yml
```

fájlt.

A workflow végső alapstruktúrája:

```yaml
name: first CI workflow

on:
  push:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
```

---

## 29.1. A workflow felépítése

### Workflow

Az egész:

```text
ci.yml
```

egy workflow.

---

### Event / trigger

```yaml
on:
  push:
  pull_request:
```

A workflow két eseményre indul:

```text
push
```

és:

```text
pull_request
```

Mental model:

```text
EVENT
↓
WORKFLOW
```

---

### Job

```yaml
jobs:
  test:
```

A workflow egyik jobja:

```text
test
```

Egy workflow több jobot is tartalmazhat.

---

### Runner

```yaml
runs-on: ubuntu-latest
```

A job egy GitHub által biztosított Ubuntu runneren fut.

Fontos felismerés:

> A runner nem egy saját, állandó Ubuntu szerver.

A GitHub létrehoz egy ideiglenes környezetet:

```text
workflow indul
↓
runner létrejön
↓
job lefut
↓
runner megszűnik
```

Nem kell SSH-val belépni és kézzel parancsokat futtatni rajta.

---

### Step

```yaml
steps:
```

A job egyes műveletei.

Nálunk:

```text
checkout
↓
Node setup
↓
npm ci
↓
npm test
```

---

## 29.2. `uses` és `run`

### `uses`

```yaml
uses: actions/checkout@v4
```

Egy előre elkészített GitHub Action használata.

Például:

```yaml
uses: actions/setup-node@v4
```

---

### `run`

```yaml
run: npm ci
```

A runner shelljében közvetlenül futtatott parancs.

Másik példa:

```yaml
run: npm test
```

Mental model:

```text
uses
→ reusable GitHub Action

run
→ shell command
```

---

# 29.3. Mi az `npm ci`?

Korábban fejlesztés közben általában:

```bash
npm i
```

vagy:

```bash
npm install
```

parancsot használtunk.

Az:

```bash
npm ci
```

CI környezethez jobban illő dependency-telepítés.

A `ci` jelentése gyakorlatilag:

```text
clean install
```

Fő különbség:

```text
npm install
→ fejlesztés során általános dependency kezelés
→ szükség esetén módosíthatja a lock file-t
```

```text
npm ci
→ package-lock.json alapján reprodukálható telepítés
→ nem akar új dependency-verziókat keresni
→ tiszta CI környezethez ideális
```

Mental model:

```text
developer machine
→ npm install

CI runner
→ npm ci
```

A CI célja:

> Ugyanazokat a dependency-verziókat használja, amelyeket a projekt lock file-ja rögzít.

---

# 29.4. Külön CI feature branch

A workflow készítése közben észrevettük, hogy még:

```text
feature/division
```

branchen voltunk.

A workflow azonban nem a division feature része volt.

Mivel a:

```text
.github/workflows/ci.yml
```

még nem volt commitolva, létrehoztunk egy új branchet:

```bash
git switch -c feature/ci-workflow
```

A nem commitolt fájl velünk jött az új branchre.

Ez ismét megmutatta:

> A nem commitolt working tree változás nem automatikusan „egy branch tulajdona”.

Ha nincs ütközés, branchváltáskor átvihető.

---

# 29.5. GitHub authentication probléma

A workflow pusholásakor ezt kaptuk:

```text
refusing to allow a Personal Access Token to create or update workflow
`.github/workflows/ci.yml` without `workflow` scope
```

A repositoryhoz volt push jogosultságunk, de a korábbi PAT nem rendelkezett workflow-módosítási jogosultsággal.

A:

```text
.github/workflows/
```

különösen érzékeny terület, mert az itt lévő fájlok automatikusan kódot futtathatnak.

---

## GitHub CLI authentication

Frissítettük a `gh` autentikációt:

```bash
gh auth refresh -h github.com -s workflow
```

WSL-ből a böngésző automatikus megnyitása nem működött, ezért a device login URL-t kézzel nyitottuk meg Windowsban.

Sikeres autentikáció után:

```bash
gh auth status
```

majd:

```bash
gh auth setup-git
```

---

## Credential helper ellenőrzése

```bash
git config --show-origin --get-regexp '^credential'
```

Eredmény:

```text
credential.helper store
credential.https://github.com.helper
credential.https://github.com.helper !/usr/bin/gh auth git-credential
```

Ez azt jelenti:

```text
általános fallback:
store

github.com esetén:
gh auth git-credential
```

Tehát GitHubhoz a Git már a GitHub CLI által kezelt credentialt használja.

Mental model:

```text
git push
↓
github.com remote
↓
GitHub-specific credential helper
↓
gh auth git-credential
↓
GitHub
```

---

# 30. GitHub Actions első futása

A workflow commitja:

```text
Add CI workflow
```

Push után GitHubon:

```text
Actions
↓
workflow run
↓
test job
↓
steps
↓
logs
```

Az első sikeres teszt:

```text
PASS test/calculator.test.js
Tests: 1 passed
```

Ez volt az első valódi CI futásunk.

Mental model:

```text
git push
↓
push event
↓
GitHub Actions
↓
temporary Ubuntu runner
↓
checkout
↓
Node setup
↓
npm ci
↓
npm test
↓
PASS
↓
runner megszűnik
```

**English:**

> I pushed my code and GitHub tested it automatically.

---

# 30.1. `NaN` a CI logban

Az első Actions logban megjelent:

```text
console.log
NaN
```

Ennek oka a `calculator.js` volt.

A program CLI módban:

```javascript
const a = Number(process.argv[2]);
const b = Number(process.argv[3]);

console.log(add(a, b));
```

részt futtatott.

Amikor Jest ezt importálta:

```javascript
const { add } = require("../src/calculator");
```

a teljes fájl lefutott.

Jest viszont nem adott CLI argumentumokat, ezért:

```text
process.argv
↓
nincs megfelelő szám
↓
Number(...)
↓
NaN
```

---

## Javítás: `require.main === module`

```javascript
function add(a, b) {
  return a + b;
}

if (require.main === module) {
  const a = Number(process.argv[2]);
  const b = Number(process.argv[3]);

  console.log(add(a, b));
}

module.exports = { add };
```

A:

```javascript
require.main === module
```

azt vizsgálja:

> Közvetlenül ezt a fájlt indították?

CLI futás:

```bash
npm run dev -- 3 4
```

→ true.

Jest import:

```javascript
require("../src/calculator")
```

→ false.

Így:

```text
CLI execution
→ process.argv használható

import
→ CLI rész nem fut
```

Push után új Actions run indult, és a `NaN` eltűnt.

---

# 31. Szándékosan hibás CI

Direkt elrontottuk a tesztet:

```javascript
expect(add(2, 3)).toBe(6);
```

Push után a workflow piros lett.

A log:

```text
FAIL test/calculator.test.js

Expected: 6
Received: 5
```

Majd:

```text
Error: Process completed with exit code 1.
```

---

## Mit tanultunk?

A CI hibakeresés nem csak annyi:

```text
piros X
```

hanem:

```text
workflow
↓
job
↓
failed step
↓
command
↓
log
↓
konkrét hiba
```

Nálunk:

```text
Jest test fails
↓
npm test returns exit code 1
↓
Run tests step fails
↓
job fails
↓
workflow fails
↓
red X
```

Ez összekapcsolódik a Linuxnál tanult exit code-okkal:

```text
0
→ success

non-zero
→ error/failure
```

**English:**

> The CI pipeline failed because a test failed.

---

# 32. CI javítása

A hibás tesztet visszajavítottuk:

```javascript
expect(add(2, 3)).toBe(5);
```

Majd:

```bash
npm test
git add .
git commit -m "Fix failing test"
git push
```

Új workflow run indult.

Eredmény:

```text
SUCCESS
```

Ez jól mutatta:

```text
failed CI
↓
log analysis
↓
fix
↓
new commit
↓
push
↓
new CI run
↓
green
```

---

# 32.1. Miért futott kétszer ugyanaz a CI?

A workflow:

```yaml
on:
  push:
  pull_request:
```

és a `feature/ci-workflow` branchhez nyitott Pull Request is tartozott.

Egy feature push ezért két eseményt is kiválthatott:

```text
git push
↓
push event
↓
workflow #1
```

és:

```text
same push updates open PR
↓
pull_request event
↓
workflow #2
```

Ezért láttunk két futást.

Ez nem hiba.

A trigger-konfiguráció következménye.

---

## Finomított production példa

Később használható például:

```yaml
on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main
```

Ekkor:

```text
feature push + open PR
→ PR check

merge to main
→ main push check
```

Így csökkenthető a felesleges dupla futás.

---

# 33. Pull Request + GitHub Actions együtt

Létrehoztuk:

```text
feature/modulo
```

branchet.

Új függvény:

```javascript
function modulo(a, b) {
  return a % b;
}
```

Példa:

```text
7 % 3 = 1
```

Teszt:

```javascript
test("7 modulo 3 should equal 1", () => {
  expect(modulo(7, 3)).toBe(1);
});
```

Majd:

```bash
npm test
git add .
git commit -m "Add modulo function"
git push -u origin feature/modulo
```

Pull Request készült.

---

# 33.1. Miért nem indult el először a modulo PR CI-je?

Fontos tanulság:

A `ci.yml` akkor még csak a:

```text
feature/ci-workflow
```

branchen létezett.

A:

```text
main
```

branchen még nem.

A modulo pedig a régi mainből indult.

Ezért azon az ágon nem állt rendelkezésre a workflow.

Mental model:

```text
feature/ci-workflow
└── ci.yml

main
└── nincs ci.yml

feature/modulo
└── mainből indult
```

Ezért nem tudott automatikusan ugyanaz a workflow elindulni.

---

## Tanulság

A CI konfiguráció közös infrastruktúra.

Ha minden feature PR-re használni akarjuk:

```text
CI workflow
↓
main
↓
new feature branches
```

jobb, ha először a közös alapba kerül.

---

# 34. Branch protection / Ruleset

A repository settingsben megnézhető:

```text
Rules
Rulesets
Branch protection
```

Egy valódi production `main` branchnél érdemes lehet például:

```text
Require pull request before merging
Require status checks
Require approvals
Block force pushes
Block deletions
```

---

## Miért?

### Require Pull Request

Ne lehessen közvetlenül mainre fejleszteni.

```text
feature
↓
PR
↓
review
↓
main
```

---

### Require status checks

Csak akkor merge-elhető:

```text
tests PASS
CI PASS
```

---

### Require approvals

Másik ember ellenőrzése szükséges.

---

### Block force pushes

Védi a shared historyt.

---

### Block deletion

Ne lehessen véletlenül törölni a fontos branchet.

---

# 35. Pull Requestek merge-elése

A labor végére több feature branchünk volt:

```text
feature/subtraction
feature/division
feature/multiply
feature/modulo
feature/ci-workflow
```

Cél:

```text
minden szükséges feature
↓
main
```

A CI workflow-t először merge-eltük, hogy a további PR-ek már használhassák.

---

# 35.1. Feature branch frissítése mainből

Ha egy feature PR konfliktusos lett:

```bash
git switch main
git fetch
git pull

git switch feature/subtraction
git merge main
```

Itt local `main`-t merge-eltünk a feature branchbe.

Ez tiszta mental model:

```text
GitHub main
↓
git pull
↓
local main updated
↓
switch feature
↓
merge main into feature
```

---

# 35.2. Conflict resolution

Több feature ugyanazt a:

```text
calculator.js
```

fájlt módosította.

Például az export sor különböző brancheken:

```javascript
module.exports = { add, subtract };
```

vagy:

```javascript
module.exports = { add, divide };
```

vagy:

```javascript
module.exports = { add, modulo };
```

A végleges megoldás nem feltétlenül egyik vagy másik branch változata.

Lehet:

```javascript
module.exports = {
  add,
  subtract,
  multiply,
  divide,
  modulo
};
```

Ez ismét megmutatta:

> Conflict resolution során egy harmadik, integrált végleges verzió is készíthető.

---

## Conflict feloldása után

```bash
git status
git add src/calculator.js
git add test/calculator.test.js
git commit -m "Merge main into feature/..."
git push
```

A push frissítette a már nyitott PR-t.

CI lefutott.

Ha zöld:

```text
Merge pull request
```

---

# 35.3. Issue lezárása

Ha a PR description tartalmaz:

```text
Closes #1
```

akkor megfelelő merge esetén az Issue automatikusan lezárható.

Ha nincs ilyen kapcsolat:

```text
Issue
↓
Close issue
```

A labor végére:

```text
open Pull Requests = 0
open Issues = 0
```

---

# 36. Local main frissítése

Miután GitHubon megtörtént az utolsó merge:

```bash
git switch main
git pull
```

Ellenőrzés:

```bash
git log --oneline --graph --decorate --all
```

A végeredményben:

```text
HEAD -> main
origin/main
```

ugyanarra a commitra mutatott.

Nálunk például:

```text
88c6338 (HEAD -> main, origin/main)
```

Ez azt jelenti:

```text
local main
=
remote-tracking origin/main
```

A local repository szinkronban van a remote mainnel.

---

# 37. Feature branch cleanup

A feature branchek munkája már bekerült a mainbe.

Ezért a local feature branchek törölhetők:

```bash
git branch -d feature/subtraction
git branch -d feature/division
git branch -d feature/multiply
git branch -d feature/modulo
git branch -d feature/ci-workflow
```

A:

```text
-d
```

safe delete.

Ha Git szerint a branch nincs megfelelően merge-elve, figyelmeztet.

Nem használunk reflexből:

```bash
git branch -D
```

parancsot.

---

## Remote branch törlése

```bash
git push origin --delete feature/subtraction
```

és hasonlóan a többire.

Ha GitHub UI-ban töröltük őket:

```bash
git fetch --prune
```

eltávolítja az elavult remote-tracking reference-eket.

---

# 37.1. Miért nem vesznek el a commitok?

Mert a main már eléri őket a repository commit gráfjában.

Mental model:

```text
feature commit
↓
merged into main
↓
main history eléri
↓
feature pointer már törölhető
```

A branch csak referencia/pointer.

A commit nem „a branchben lakik”.

---

# 37.2. Practice branch

A:

```text
practice/merge-conflict-lab
```

branchet szándékosan megtartottuk.

Ez archiválja a külön merge-conflict gyakorlás historyját, amit nem akartunk a valódi main workflow részévé tenni.

---

# 38. Annotated tag készítése

A végleges main lett az első stabil verziónk.

Tag:

```text
v1.0.0
```

Létrehozás:

```bash
git tag -a v1.0.0 -m "First lab release"
```

---

## Mit jelent?

```text
-a
→ annotated tag

v1.0.0
→ verzió neve

-m
→ tag message
```

Ellenőrzés:

```bash
git tag
```

Részletesen:

```bash
git show v1.0.0
```

---

## Push

```bash
git push origin v1.0.0
```

A tageket érdemes explicit pusholni.

Mental model:

```text
stable main commit
       ↑
     v1.0.0
```

A tag azt mondja:

> Ezt a konkrét commitot tekintjük az 1.0.0 verziónak.

---

# 39. GitHub Release

A `v1.0.0` tagből GitHub Release-t készítettünk.

Release title:

```text
v1.0.0
```

Példa release notes:

```markdown
First stable version of the Git and GitHub Actions lab.

Features:

- addition
- subtraction
- multiplication
- division
- modulo
- automated Jest tests
- GitHub Actions CI
```

---

# 39.1. Git tag vs GitHub Release

### Git tag

A Git része.

```text
commit
↑
v1.0.0
```

Konkrét commit megjelölése.

---

### GitHub Release

GitHub-funkció.

Általában egy taghez kapcsolódik.

Tartalmazhat:

```text
title
release notes
documentation
assets
downloadable files
```

Mental model:

```text
commit
↑
Git tag
↑
GitHub Release
```

A Release lényegében a verzió emberbarát dokumentációs/publikációs oldala.

---

# 40. Végső repository ellenőrzés

A labor végén:

```bash
git status
```

Elvárt:

```text
working tree clean
```

---

## Branchek

```bash
git branch -a
```

Ellenőrizni kell:

- main létezik,
- szükségtelen feature branchek törölve,
- practice branch csak akkor marad, ha szándékosan megtartjuk.

---

## Remote

```bash
git remote -v
```

Ellenőrizni kell az:

```text
origin
```

fetch és push URL-jét.

---

## History

```bash
git log --oneline --graph --decorate --all
```

Ellenőrizni kell:

```text
HEAD -> main
origin/main
```

ugyanott van-e.

---

## GitHub ellenőrzés

### Issues

Elkészült issue-k lezárva.

### Pull Requests

Nincs feleslegesen nyitott PR.

### Actions

Utolsó main workflow zöld.

### Branches

Felesleges remote feature branchek törölve.

### Commits

A szükséges funkciók a main historyjában vannak.

### Tags

```text
v1.0.0
```

létezik.

### Releases

```text
v1.0.0
```

release publikálva.

---

# 40.1. Végső funkcionális állapot

A kalkulátor végül tartalmazza:

```text
add()
subtract()
multiply()
divide()
modulo()
```

és a hozzájuk tartozó teszteket.

Emellett:

```text
.github/workflows/ci.yml
```

automatikusan futtatja a CI-t.

---

# 40.2. További saját kiegészítés – smoke test

A labor későbbi továbbfejlesztéseként nemcsak unit testet érdemes futtatni, hanem ellenőrizni, hogy maga az alkalmazás is elindul-e.

Példa:

```bash
npm run dev -- 6 3
```

Ez már egy egyszerű:

```text
smoke test
```

jellegű ellenőrzés.

Mental model:

```text
unit test
→ függvény működését ellenőrzi

smoke test
→ az alkalmazás alapvető futtathatóságát ellenőrzi
```

Későbbi DevOps projektekben:

```text
unit tests
↓
integration tests
↓
build
↓
container build
↓
startup
↓
health check
↓
deployment
```

---

# 41. Végső rendszerkép

A labor végére a teljes folyamat:

```text
WSL Ubuntu
↓
Node.js project
↓
Git local repository
↓
feature branch
↓
code change
↓
git add
↓
commit
↓
push
↓
GitHub remote
↓
Issue
↓
Pull Request
↓
Code Review
↓
GitHub Actions
↓
temporary runner
↓
npm ci
↓
npm test
↓
CI check
↓
merge
↓
main
↓
tag
↓
GitHub Release
```

Ez már egy egyszerű, de valós fejlesztési/DevOps workflow.

---

# 42. Ellenőrző kérdések – megoldások

## Git

### 1. Working directory, staging area, repository

```text
Working directory
→ amit jelenleg a fájlokban módosítok

Staging area
→ amit a következő commitba előkészítettem

Repository
→ commitolt project history
```

Folyamat:

```text
working tree
↓ git add
staging area
↓ git commit
repository history
```

---

### 2. Mit csinál a `git add`?

A módosítást staging area-ba teszi.

Conflict esetén azt is jelzi, hogy az adott fájl konfliktusát feloldottnak tekintjük.

---

### 3. Mit csinál a `git commit`?

Új snapshotot / history pontot hoz létre a staged változtatásokból.

---

### 4. Mi a branch?

Mozgó referencia/pointer egy commitra.

```text
branch
↓
commit
```

---

### 5. Mi a HEAD?

Azt jelzi, hol vagyunk jelenleg.

Normál eset:

```text
HEAD
↓
branch
↓
commit
```

Detached HEAD:

```text
HEAD
↓
commit
```

---

### 6. Mi az `origin`?

A remote repositoryhoz adott local név.

Általában:

```text
origin
→ GitHub repository
```

---

### 7. Mi az upstream?

Az a remote branch, amelyet egy local branch alapértelmezett partnerként követ.

Példa:

```text
feature/modulo
↕
origin/feature/modulo
```

---

### 8. `fetch` vs `pull`

```text
fetch
→ remote információ letöltése
→ local working branch nem módosul automatikusan
```

```text
pull
→ fetch + integráció
```

---

### 9. `reset` vs `revert`

```text
reset
→ branch pointer/history mozgatása
→ főleg unpublished/local history
```

```text
revert
→ új commitban visszavon egy régi commitot
→ shared/published historyhoz biztonságosabb
```

---

### 10. Mire jó a stash?

Félkész, még nem commitolandó munka ideiglenes félrerakására.

```text
working tree
↓
stash
↓
temporary storage
```

---

### 11. Mi a merge conflict?

Amikor két history-vonal változtatásait Git nem tudja automatikusan összeegyeztetni.

---

### 12. Mire jó a reflog?

Local referencia-mozgások naplója.

Segíthet elveszettnek tűnő commitok visszakeresésében.

---

## GitHub

### 13. Git vs GitHub

```text
Git
→ verziókezelő rendszer

GitHub
→ Git repository hosting + collaboration platform
```

---

### 14. Mire jó az Issue?

Feladat, bug, ötlet vagy munka nyomon követésére.

---

### 15. Mire jó a Pull Request?

Egy branch változtatásainak integrációjára vonatkozó javaslat.

Lehetővé teszi:

```text
review
discussion
CI checks
approval
merge
```

---

### 16. Base és head branch

```text
head
→ ahonnan jön a változás

base
→ ahová menni akar
```

Példa:

```text
feature/modulo → main

head = feature/modulo
base = main
```

---

### 17. Mire való a Code Review?

A változtatások emberi ellenőrzésére merge előtt.

---

### 18. Miért hasznos a branch protection?

Megakadályozhat:

```text
direct main push
merge failing CI mellett
force push
branch deletion
review nélküli merge
```

---

## GitHub Actions

### 19. Mi az event?

Az esemény, amely workflow-t indít.

Például:

```text
push
pull_request
```

---

### 20. Mi a workflow?

Automatizált folyamat, amelyet YAML fájl ír le.

---

### 21. Mi a job?

A workflow egyik végrehajtási egysége.

---

### 22. Mi a runner?

Az a gép/környezet, ahol a job fut.

Nálunk:

```text
ubuntu-latest
```

---

### 23. Mi a step?

A jobon belüli egy konkrét lépés.

---

### 24. `uses` vs `run`

```text
uses
→ előre elkészített Action

run
→ shell command
```

---

### 25. Mit automatizáltunk?

```text
dependency install
+
Jest tests
```

push és PR eseményekre.

---

### 26. Miért jobb, hogy nem csak a fejlesztő gépén fut?

Mert:

```text
fresh environment
reproducibility
automatic validation
shared result
```

Csökkenti a:

> „Nálam működik.”

problémát.

---

### 27. Mit nézzünk először piros workflow-nál?

```text
workflow
↓
failed job
↓
failed step
↓
command
↓
log
↓
exit code / error
```

---

# 43. Hibakeresési stratégia

A fő szabály:

> Először értsd meg az állapotot. Utána változtass.

Ne kezdj véletlenszerű parancsokat futtatni.

---

## 1. Hol vagyok?

```bash
pwd
```

---

## 2. Melyik branchen vagyok?

```bash
git branch --show-current
```

vagy:

```bash
git branch
```

---

## 3. Mi a repository állapota?

```bash
git status
```

Ez általában az első Git troubleshooting parancs.

---

## 4. Tracking kapcsolatok

```bash
git branch -vv
```

Megmutatja például:

```text
main [origin/main]
```

---

## 5. Összes branch

```bash
git branch -a
```

Local és remote-tracking branchek.

---

## 6. History

```bash
git log --oneline --graph --decorate --all
```

Megmutatja:

```text
commits
branches
HEAD
origin/*
merge history
```

---

## 7. Remote-ok

```bash
git remote -v
```

---

## 8. Remote állapot frissítése

```bash
git fetch
```

---

## 9. Actions troubleshooting

GitHub:

```text
Actions
↓
Workflow run
↓
Job
↓
Failed step
↓
Log
```

A logból keressük:

```text
error
failed command
expected / received
exit code
```

---

# 43.1. Gyakorlati troubleshooting mental model

```text
STATE FIRST
↓
git status
↓
branch
↓
history
↓
remote
↓
error message
↓
only then fix
```

Ez rendszerüzemeltetésnél és DevOpsnál általánosan is jó gondolkodás.

---

# 44. Biztonsági szabályok

Vannak erős Git parancsok, amelyeket nem szabad automatikusan használni.

---

## `git reset --hard`

```bash
git reset --hard ...
```

A working tree változásokat is eldobhatja.

Mindig ellenőrizd előtte:

```bash
git status
```

---

## `git clean -f`

Untracked fájlokat törölhet.

Ezeket Git nem biztos, hogy vissza tudja állítani.

---

## `git push --force`

Remote historyt írhat át.

Közös brancheken különösen veszélyes.

Production `main`:

```text
NO blind force push
```

---

## `git branch -D`

Kényszerített branch törlés.

A Git figyelmeztetését is megkerüli.

Előbb:

```bash
git branch -d
```

---

# 44.1. Secretek

Ne commitolj valódi secretet:

```text
.env
PAT
password
API key
private SSH key
```

Fontos:

> Ha egy secret egyszer bekerült Git historyba, nem elég csak a következő commitban törölni a fájlt.

A secretet kompromittáltnak kell tekinteni és rotálni/cserélni.

---

# 44.2. Workflow security

A:

```text
.github/workflows/
```

különösen érzékeny.

Ezek a fájlok automatizált kódfuttatást határoznak meg.

Ezért a GitHub authenticationnél is külön workflow permission problémába futottunk.

---

# 45. Opcionális haladó feladatok – megoldások

Ezeket a fő labor során nem volt kötelező végrehajtani, de a megoldási elv a következő.

---

# 45.A Második remote

Meglévő remote-ok:

```bash
git remote -v
```

Második remote hozzáadása:

```bash
git remote add backup <repository-url>
```

Ellenőrzés:

```bash
git remote -v
```

Példa:

```text
origin  ...
backup  ...
```

A remote név tetszőleges.

Az:

```text
origin
```

csak konvenció.

---

# 45.B Cherry-pick

Tegyük fel:

```text
feature/a
```

branchen létrehozunk egy commitot:

```text
abc123 Add useful change
```

Átváltunk másik branchre:

```bash
git switch feature/b
```

Majd:

```bash
git cherry-pick abc123
```

A Git csak ezt az egy commitot alkalmazza az aktuális branchre.

Mental model:

```text
branch A:
A → B → C

cherry-pick C

branch B:
X → Y → C'
```

Fontos:

A cherry-picked commit új commit objektum lesz, ezért általában új hash-t kap.

---

# 45.C Rebase

Kiindulás:

```text
        feature1
       /
A → B
       \
        main1
```

Feature branchen:

```bash
git switch feature/rebase-practice
git rebase main
```

A Git a feature commitjait a friss main tetejére helyezi.

Előtte:

```text
A → B → F1 → F2
     \
      M1 → M2
```

Rebase után:

```text
A → B → M1 → M2 → F1' → F2'
```

A commitok új hash-t kaphatnak.

Ezért:

> Rebase history rewriting operation.

Saját local feature branchen hasznos.

Már megosztott közös historyn óvatosan.

---

## Merge vs rebase

### Merge

```text
history megőrzése
+
merge commit
```

### Rebase

```text
lineárisabb history
+
commitok újraírása
```

---

# 45.D Több Node.js verzió GitHub Actionsben

Ehhez matrix strategy használható.

Példa:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version:
          - 18
          - 20
          - 22

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - run: npm ci
      - run: npm test
```

Ez több külön tesztkörnyezetet hoz létre:

```text
Node 18
↓
npm test

Node 20
↓
npm test

Node 22
↓
npm test
```

Ez már jól mutatja a CI egyik nagy előnyét:

> ugyanazt a projektet több környezeten automatikusan ellenőrizhetjük.

---

# Labor végeredménye

A teljes labor során végigmentünk:

```text
Node project
↓
Git initialization
↓
commits
↓
branches
↓
GitHub remote
↓
Issues
↓
Pull Requests
↓
Code Review
↓
remote tracking
↓
merge
↓
merge conflict
↓
stash
↓
reset
↓
revert
↓
detached HEAD
↓
reflog recovery
↓
GitHub Actions
↓
CI success
↓
intentional CI failure
↓
CI troubleshooting
↓
feature integration
↓
branch cleanup
↓
tag
↓
release
```

---

# Végső DevOps mental model

A labor legfontosabb tanulsága nem egy-egy Git parancs bemagolása.

Hanem a folyamat megértése:

```text
változtatás
↓
verziókezelés
↓
elkülönített feature branch
↓
review
↓
automatikus ellenőrzés
↓
integráció
↓
stabil main
↓
verziójelölés
↓
release
```

A Git nem csak „mentés”.

A GitHub nem csak „hely, ahová feltöltjük a kódot”.

A GitHub Actions pedig nem csak „lefuttat egy tesztet”.

Együtt egy olyan workflow-t alkotnak, amelyben a változtatások:

```text
nyomon követhetők
ellenőrizhetők
visszavonhatók
review-zhatók
automatizáltan tesztelhetők
és kontrolláltan integrálhatók
```

---

# Rövid szakmai angol összefoglaló

> I created feature branches for separate changes.

> I opened Pull Requests and reviewed the changes.

> I resolved merge conflicts manually.

> I used stash for unfinished work.

> I used reset for local history and revert for published commits.

> I recovered a lost commit with reflog.

> I created a GitHub Actions CI workflow.

> The CI workflow runs tests automatically.

> I investigated a failed workflow using the logs.

> I merged the tested features into main.

> I cleaned up the feature branches.

> I tagged the first stable version as v1.0.0.

> I created a GitHub Release from the tag.