# Git + GitHub + GitHub Actions – komplex gyakorlólabor

## A labor célja

Ebben a laborban egy kis Node.js projekt teljes Git/GitHub életciklusát fogod végigcsinálni.

A cél nem az, hogy parancsokat másolj, hanem hogy gyakorlatban lásd:

```text
WSL Ubuntu
↓
új projekt
↓
Git repository
↓
commitok
↓
branchek
↓
GitHub remote
↓
Issue
↓
feature branch
↓
Pull Request
↓
Code Review / Checks
↓
GitHub Actions CI
↓
merge
↓
release / tag
```

A labor során szándékosan lesz:

- több local branch,
- remote branch,
- merge,
- merge conflict,
- hibás commit,
- visszavonás,
- GitHub Issue,
- Pull Request,
- GitHub Actions workflow,
- szándékosan hibás CI futás,
- javítás,
- sikeres check,
- merge,
- tag.

---

# 0. Fontos szabály

Ez **feladatlap**, nem teljes megoldókulcs.

Ahol konkrét Git-parancsot már megtanultunk, ott sokszor csak a feladatot kapod meg.

Használd a saját `git-commands-reference.md` jegyzetedet.

Ha elakadsz, először próbáld meg kideríteni:

1. Mi az aktuális állapot?
2. Melyik branchen vagy?
3. Mi staged?
4. Mi committed?
5. Mi csak local?
6. Mi került már remote-ra?

A legfontosabb diagnosztikai parancsaid:

```bash
git status
git branch
git branch -a
git log --oneline --graph --decorate --all
git remote -v
```

---

# 1. Előfeltételek

A laborhoz szükséges:

- Windows 11
- WSL2
- Ubuntu
- Git
- Node.js
- npm
- GitHub account
- működő GitHub authentication
- opcionálisan GitHub CLI (`gh`)
- VS Code

Ellenőrizd:

```bash
git --version
node --version
npm --version
```

Ha használod a GitHub CLI-t:

```bash
gh --version
gh auth status
```

## Biztonsági szabály

Soha ne másolj a projektbe:

- PAT-et,
- jelszót,
- private SSH key-t,
- API key-t,
- valódi secretet.

---

# 2. Projekt könyvtár létrehozása

A projekt a WSL Linux fájlrendszerében legyen, ne `/mnt/c/...` alatt.

Menj a projektmappádba:

```bash
cd ~/projects/LABS
```

Hozz létre egy új könyvtárat.

A projekt javasolt neve:

```text
git-github-actions-lab
```

## Feladat

Hozd létre a könyvtárat, majd lépj bele.

Ellenőrzés:

```bash
pwd
```

A végének körülbelül ilyennek kell lennie:

```text
/home/<user>/projects/LABS/git-github-actions-lab
```

---

# 3. Egyszerű Node.js projekt létrehozása

Inicializálj egy Node.js projektet:

```bash
npm init -y
```

Hozd létre ezt a struktúrát:

```text
git-github-actions-lab/
├── src/
│   └── calculator.js
├── test/
│   └── calculator.test.js
├── package.json
└── README.md
```

## `src/calculator.js`

Kezdésnek készíts egy egyszerű függvényt:

```javascript
function add(a, b) {
  return a + b;
}

module.exports = {
  add,
};
```

## `README.md`

Írj bele legalább:

```markdown
# Git GitHub Actions Lab

A small Node.js project for practicing Git, GitHub and CI/CD basics.
```

---

# 4. Tesztkörnyezet

Telepíts egy egyszerű teszt frameworköt.

Ehhez a laborhoz használhatod a Jestet:

```bash
npm install --save-dev jest
```

A `package.json` scripts részében legyen:

```json
"scripts": {
  "test": "jest"
}
```

## `test/calculator.test.js`

Készíts egy első tesztet az `add()` függvényhez.

A teszt ellenőrizze például:

```text
2 + 3 = 5
```

Futtasd:

```bash
npm test
```

## Ellenőrzési pont

A labor Git-része előtt a tesztnek sikeresen le kell futnia.

---

# 5. `.gitignore`

Hozz létre `.gitignore` fájlt.

Legalább ezt tartalmazza:

```text
node_modules/
.env
```

## Gondolkodási kérdés

Miért nem akarjuk a `node_modules` könyvtárat Gitben tárolni?

---

# 6. Git repository inicializálása

## Feladat

1. Inicializáld a repositoryt.
2. Ellenőrizd az állapotát.
3. Nézd meg az aktuális branchet.
4. Ha szükséges, legyen a fő branch neve `main`.

## Ellenőrzési pont

Próbáld értelmezni a:

```bash
git status
```

kimenetét.

Mely fájlok:

- untracked,
- tracked,
- staged?

---

# 7. Git identity ellenőrzése

Nézd meg:

```bash
git config user.name
git config user.email
```

Majd:

```bash
git config --list --show-origin
```

## Feladat

Azonosítsd:

- honnan jön a `user.name`,
- honnan jön a `user.email`,
- van-e local config,
- van-e global config.

## Kérdés

Mi lenne a különbség, ha ezt futtatnád?

```bash
git config user.email "work@example.com"
```

és ezt:

```bash
git config --global user.email "work@example.com"
```

---

# 8. Első staging és commit

## Feladat

Stage-eld a projekt megfelelő fájljait.

Ezután ellenőrizd:

```bash
git status
```

Majd készíts első commitot.

Javasolt commit message:

```text
Initial Node.js project
```

## Ellenőrzés

```bash
git log --oneline
```

Ezután:

```bash
git status
```

A working tree legyen tiszta.

---

# 9. Második commit

Módosítsd a README-t.

Adj hozzá egy `## Features` szekciót.

Commitold külön.

Javasolt commit message:

```text
Update README
```

## Ellenőrzési pont

```bash
git log --oneline
```

Most legalább két commitot kell látnod.

---

# 10. Commit gráf megtekintése

Futtasd:

```bash
git log --oneline --graph --decorate --all
```

## Feladat

Azonosítsd:

- hol van `HEAD`,
- melyik commitra mutat a `main`,
- melyik a legújabb commit.

---

# 11. GitHub repository létrehozása

GitHubon hozz létre egy új repositoryt:

```text
git-github-actions-lab
```

## Fontos

Mivel localban már van projekted:

- ne generálj új README-t,
- ne generálj új `.gitignore`-t,
- ne generálj licence-t.

Így a remote repo kezdetben üres lesz.

---

# 12. Remote beállítása

GitHubról másold ki a repository URL-jét.

Használhatsz HTTPS vagy SSH remote-ot.

## Feladat

Add hozzá remote-ként `origin` néven.

Majd ellenőrizd:

```bash
git remote -v
```

---

# 13. Első push

Pushold a `main` branchet GitHubra, és állítsd be az upstream kapcsolatot.

## Ellenőrzés

```bash
git branch -vv
```

A `main` mellett valami ilyesmit kell látnod:

```text
[origin/main]
```

GitHub weboldalon ellenőrizd, hogy megjelentek-e a fájlok.

---

# 14. GitHub Issue létrehozása

GitHubon menj az `Issues` részre.

Hozz létre:

```text
Title:
Add subtraction feature
```

Leírás például:

```markdown
Add a subtract(a, b) function to the calculator.

Acceptance criteria:

- subtract(5, 2) returns 3
- unit test is added
- existing tests still pass
```

## Feladat

Adj hozzá megfelelő labelt, ha szeretnél, például `enhancement`.

Jegyezd meg az Issue számát, például `#1`.

---

# 15. Feature branch az Issue-hoz

Localban először frissítsd a `main` branchet.

Majd hozz létre új branchet:

```text
feature/subtraction
```

## Ellenőrzés

```bash
git branch
```

Az aktuális branch `feature/subtraction` legyen.

---

# 16. Feature implementáció

A `calculator.js` fájlhoz adj hozzá `subtract(a, b)` függvényt.

Egészítsd ki az exportot.

Adj hozzá megfelelő tesztet.

## Feladat

Futtasd:

```bash
npm test
```

Csak akkor folytasd, ha a tesztek zöldek.

---

# 17. Feature commit

Stage + commit.

Javasolt message:

```text
Add subtraction feature
```

## Feladat

Nézd meg:

```bash
git log --oneline --graph --decorate --all
```

Figyeld meg:

- hol áll `main`,
- hol áll `feature/subtraction`.

---

# 18. Feature branch push GitHubra

Pushold a branchet remote-ra és állíts be upstreamet.

Ellenőrzés:

```bash
git branch -vv
git branch -r
```

GitHubon most már két branchnek kell léteznie:

```text
main
feature/subtraction
```

---

# 19. Pull Request létrehozása

GitHubon hozz létre Pull Requestet.

```text
head:
feature/subtraction

base:
main
```

PR title:

```text
Add subtraction feature
```

A leírásban hivatkozz az Issue-ra:

```text
Closes #<issue-number>
```

Például:

```text
Closes #1
```

## Ellenőrzési feladat

A PR-ban nézd meg:

- Conversation
- Commits
- Files changed
- Checks

Még ne merge-eld.

---

# 20. Code Review szimuláció

Mivel most egyedül dolgozol, saját magad fogod szimulálni a review-t.

## Feladat

Nézd végig a `Files changed` fület úgy, mintha más kódját néznéd.

Ellenőrizd:

- jó-e a függvénynév,
- van-e teszt,
- érthető-e a változtatás,
- nem került-e bele felesleges fájl.

Írj magadnak egy rövid megjegyzést a PR Conversation részébe, például:

```text
Review note: add another test for negative numbers.
```

---

# 21. PR módosítása új commit segítségével

Localban maradj a `feature/subtraction` branchen.

Adj hozzá új tesztet:

```text
subtract(-2, 3) === -5
```

Futtasd a teszteket.

Commit:

```text
Add subtraction edge case test
```

Push.

## Figyeld meg GitHubon

Nem készítettél új PR-t.

A meglévő PR automatikusan frissült.

---

# 22. Remote változás szimulálása

Most gyakoroljuk a `fetch` fogalmát.

GitHub webes editor segítségével módosítsd a README-t a `main` branchen.

Adj hozzá például:

```markdown
This repository is used for Git and GitHub practice.
```

Commitold közvetlenül GitHubon.

## Localban

Még ne használj `pull`-t.

Először:

```bash
git fetch
```

Majd:

```bash
git log --oneline --graph --decorate --all
```

## Feladat

Keresd meg a `main` és `origin/main` különbségét.

Ezután frissítsd a local `main` branchet megfelelően.

---

# 23. Másik remote branch szimulációja

GitHubon hozz létre egy új branchet a webes felületről:

```text
feature/multiply
```

## Localban

Futtass fetch-et.

Nézd meg:

```bash
git branch -r
```

Meg kell jelennie:

```text
origin/feature/multiply
```

## Feladat

Hozz létre belőle local tracking branchet.

Ellenőrzés:

```bash
git branch -vv
```

Ezután térj vissza a saját munkádhoz.

---

# 24. Merge conflict labor

Most szándékosan konfliktust fogunk készíteni.

## A lépés

`main` branchen módosítsd a README ugyanazon sorát egy adott szövegre.

Commitold.

Ezután egy új branchben:

```text
feature/readme-conflict
```

ugyanazt a sort módosítsd más tartalomra.

Commitold.

Ezután próbáld a branchet `main`-be merge-elni.

## Elvárt eredmény

```text
merge conflict
```

## Feladat

1. `git status`
2. nyisd meg az ütköző fájlt,
3. keresd meg a conflict marker-eket,
4. készíts egy harmadik, értelmes végleges változatot,
5. stage,
6. commit.

## Kérdés

Miért nem egyszerűen azt választottuk, hogy `ours` vagy `theirs`?

---

# 25. Stash labor

Hozz létre egy új branchet:

```text
feature/division
```

Kezdd el a `divide()` függvényt, de **ne commitold**.

Ezután tegyük fel, sürgős hotfix érkezik.

## Feladat

Használd a stash-t úgy, hogy ideiglenesen el tudd rakni a nem kész munkát.

Ezután válts `main`-re.

Majd térj vissza `feature/division` branchre és állítsd vissza a félretett munkát.

## Ellenőrzés

A `divide()` módosítás ismét legyen jelen.

---

# 26. Hibás local commit és reset

A `feature/division` branchen készíts egy szándékosan rossz commitot.

Például módosítsd a README-t:

```text
THIS IS A BAD CHANGE
```

Commitold:

```text
Bad experimental change
```

## Feladat

Ez a commit még **ne legyen pusholva**.

Próbáld ki a soft vagy mixed resetet, és figyeld meg:

- mi történik a committal,
- mi történik a fájlmódosítással,
- mit mutat `git status`.

Ezután állítsd vissza megfelelő állapotba a projektet.

---

# 27. Már publikált változtatás visszavonása – revert

Készíts egy ártalmatlan commitot egy feature branchen.

Pushold remote-ra.

Ezután tegyük fel, hogy kiderült: rossz volt a változtatás.

## Feladat

Ne írj át remote history-t.

Használd a `revert` megközelítést.

Pushold az új visszavonó commitot.

## Ellenőrzés

A historyban mindkettő látszódjon:

```text
rossz commit
↓
revert commit
```

---

# 28. Reflog megfigyelés

Futtasd:

```bash
git reflog
```

## Feladat

Keress benne:

- commit műveletet,
- branch switch-et,
- reset műveletet.

Nem kell semmit visszaállítani.

A cél annak megértése, hogy a reflog egyfajta local biztonsági nyom.

---

# 29. GitHub Actions CI létrehozása

Most automatizáljuk azt, amit eddig kézzel csináltál:

```text
npm ci
npm test
```

Hozd létre:

```text
.github/workflows/ci.yml
```

## A workflow feladata

Induljon:

```text
push
```

és:

```text
pull_request
```

eseményre.

Használjon Ubuntu runnert.

Lépések:

```text
checkout repository
↓
Node.js setup
↓
npm ci
↓
npm test
```

## Feladat

A workflow-t a saját GitHub Actions jegyzeted alapján írd meg.

Ne másold vakon.

Próbáld meg azonosítani:

- event,
- workflow,
- job,
- runner,
- step,
- action,
- run parancs.

---

# 30. GitHub Actions első futás

Commitold:

```text
Add CI workflow
```

Pushold.

GitHubon menj az `Actions` fülre.

## Feladat

Nyisd meg a workflow futását.

Nézd meg:

```text
workflow
↓
job
↓
steps
↓
logs
```

Azonosítsd:

- checkout step,
- Node setup,
- dependency install,
- test.

---

# 31. Szándékosan hibás CI

Most direkt törjük el.

Módosíts egy tesztet úgy, hogy hibás eredményt várjon.

Például:

```text
2 + 3 = 6
```

Commit:

```text
Break test intentionally
```

Push.

## GitHubon

Nézd meg az Actions futását.

Elvárt:

```text
FAILED
```

## Feladat

Ne csak a piros X-et nézd.

Találd meg:

1. melyik job hibázott,
2. melyik step hibázott,
3. mi volt a futtatott parancs,
4. mit mond a log,
5. miért hibázott.

Ez az első CI troubleshooting feladatod.

---

# 32. CI javítása

Javítsd vissza a tesztet.

Commit:

```text
Fix failing test
```

Push.

Ellenőrizd, hogy a következő workflow:

```text
SUCCESS
```

legyen.

---

# 33. Pull Request + GitHub Actions együtt

Ha a korábbi subtraction PR még nyitva van, ellenőrizd a Checks részt.

Ha már nincs, készíts egy új apró feature branchet, például:

```text
feature/modulo
```

Készíts hozzá `modulo(a, b)` függvényt és tesztet.

Push → Pull Request.

## Figyeld meg

A PR létrejötte után a GitHub Actions automatikusan lefut.

Elvárt:

```text
PR
↓
CI check
↓
tests
↓
zöld státusz
```

---

# 34. Branch protection / Ruleset – megfigyelési feladat

GitHub repository settingsben keresd meg a branch rules / rulesets lehetőséget.

## Ne állíts be vakon semmit.

Először csak keresd meg, milyen lehetőségek vannak például:

```text
Require pull request
Require status checks
Require approvals
Block force pushes
Block deletions
```

## Gondolkodási feladat

Milyen szabályokat állítanál be egy valódi production projekt `main` branchére?

Írd le a saját válaszodat a laborjegyzetedbe.

---

# 35. PR merge

Ha:

- a kód jó,
- a tesztek zöldek,
- a CI sikeres,

merge-eld a PR-t.

## Ellenőrzés

Az Issue-nál nézd meg, automatikusan lezáródott-e, ha a PR tartalmazta:

```text
Closes #...
```

---

# 36. Local repository frissítése merge után

A merge GitHubon történt.

A local `main` még lehet régi.

## Feladat

Frissítsd a local `main` branchet.

Ellenőrizd:

```bash
git log --oneline --graph --decorate --all
```

A local és remote main ugyanarra az állapotra mutasson.

---

# 37. Feature branch takarítás

Merge után:

- töröld a remote feature branchet, ha még létezik,
- töröld a local feature branchet.

## Kérdés

Miért nem vesznek el a feature commitjai, ha már merge-elve lettek?

---

# 38. Tag készítése

Tegyük fel, hogy ez az első stabil verziónk.

Hozz létre annotated taget:

```text
v1.0.0
```

Megjegyzés:

```text
First lab release
```

Pushold a taget GitHubra.

## Ellenőrzés

GitHubon nézd meg a tageket.

---

# 39. GitHub Release – opcionális

GitHubon a tag alapján készíts egy Release-t.

Példa:

```text
Release title:
v1.0.0
```

Leírás:

```markdown
First stable version of the Git and GitHub Actions lab.

Features:

- addition
- subtraction
- automated tests
- GitHub Actions CI
```

## Gondolkodási kérdés

Mi a különbség a Git tag és a GitHub Release között?

---

# 40. Végső repository ellenőrzés

Futtasd:

```bash
git status
git branch -a
git remote -v
git log --oneline --graph --decorate --all
```

A projekt GitHub oldalán ellenőrizd:

- Issues
- Pull Requests
- Actions
- Branches
- Commits
- Tags / Releases

---

# 41. Végső rendszerkép

A labor végére ezt a folyamatot kell értened:

```text
WSL Ubuntu
↓
Node.js project
↓
Git local repository
↓
branch
↓
commit
↓
push
↓
GitHub remote repository
↓
Issue
↓
feature branch
↓
Pull Request
↓
GitHub Actions
↓
runner
↓
npm ci
↓
npm test
↓
check
↓
merge
↓
main
↓
tag
↓
release
```

---

# 42. Ellenőrző kérdések

A labor végén próbálj válaszolni saját szavaiddal.

## Git

1. Mi a különbség working directory, staging area és repository között?
2. Mit csinál a `git add`?
3. Mit csinál a `git commit`?
4. Mi a branch?
5. Mi a `HEAD`?
6. Mi az `origin`?
7. Mi az upstream?
8. Mi a különbség `fetch` és `pull` között?
9. Mi a különbség `reset` és `revert` között?
10. Mire jó a `stash`?
11. Mi az a merge conflict?
12. Mire jó a reflog?

## GitHub

13. Mi a különbség Git és GitHub között?
14. Mire jó az Issue?
15. Mire jó a Pull Request?
16. Mi a base és head branch?
17. Mire való a code review?
18. Miért hasznos a branch protection?

## GitHub Actions

19. Mi az event?
20. Mi a workflow?
21. Mi a job?
22. Mi a runner?
23. Mi a step?
24. Mi a különbség `uses` és `run` között?
25. Mit automatizáltunk ebben a laborban?
26. Miért jó, hogy a teszt nem a fejlesztő saját gépén fut csak?
27. Mit néznél meg először, ha egy workflow piros?

---

# 43. Hibakeresési stratégia

Ha bárhol elakadsz:

## Git állapot

```bash
git status
```

## Branch

```bash
git branch
git branch -vv
```

## Minden branch

```bash
git branch -a
```

## History

```bash
git log --oneline --graph --decorate --all
```

## Remote

```bash
git remote -v
```

## Remote frissítése

```bash
git fetch
```

## GitHub Actions

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

Ne próbálj rögtön véletlenszerű parancsokat futtatni.

Először értsd meg az állapotot.

---

# 44. Biztonsági figyelmeztetések

A labor során különösen vigyázz:

```bash
git reset --hard
git clean -f
git push --force
git branch -D
```

Ezek adatvesztést vagy history módosítást okozhatnak.

Ne használj force push-t a `main` branchen.

Ne tárolj valódi secretet Gitben.

Ne commitolj:

```text
.env
PAT
private SSH key
password
API key
```

---

# 45. Opcionális haladó feladatok

Ha a fő labor már megy:

## A. Második remote

Adj a repositoryhoz egy második remote-ot más néven.

Ne pusholj rá, csak vizsgáld meg:

```bash
git remote -v
```

## B. Cherry-pick

Készíts külön branchen egy commitot, majd csak azt az egy commitot emeld át egy másik branchbe.

## C. Rebase

Hozz létre egy feature branchet, közben módosítsd a `main`-t is, majd próbáld ki a merge helyett a rebase-t.

A művelet előtt és után nézd meg a commit gráfot.

## D. Több Node.js verzió

Módosítsd a GitHub Actions workflow-t úgy, hogy több Node.js verzión teszteljen.

Például:

```text
Node 20
Node 22
```

Ez később a matrix strategy témához vezet.

## E. Build step

Adj a projekthez egy egyszerű build scriptet, és futtasd CI-ban a tesztek után.

---

# 46. Mikor tekinthető késznek a labor?

A labor akkor kész, ha:

- [ ] a projekt WSL alatt jött létre,
- [ ] Git repository működik,
- [ ] több commit készült,
- [ ] local és remote brancheket használtál,
- [ ] `origin` be van állítva,
- [ ] push/fetch/pull működik,
- [ ] készítettél Issue-t,
- [ ] készítettél Pull Requestet,
- [ ] szimuláltál code review-t,
- [ ] oldottál merge conflictot,
- [ ] használtál stash-t,
- [ ] kipróbáltál resetet,
- [ ] kipróbáltál revertet,
- [ ] megnézted a reflogot,
- [ ] létrehoztál GitHub Actions workflow-t,
- [ ] láttál hibás CI futást,
- [ ] log alapján megtaláltad a hibát,
- [ ] kijavítottad,
- [ ] zöld CI után merge-eltél,
- [ ] lezártál Issue-t,
- [ ] létrehoztál taget,
- [ ] érted a teljes folyamat fő részeit.

---

# 47. Fontos: nem a gyorsaság számít

Ezt a labort nem kell egy ülésben befejezni.

A cél:

```text
minden lépésnél tudd,
MI történt
és
MIÉRT történt.
```

Ha egy parancs működik, de nem érted az eredményét, állj meg és vizsgáld meg.

A DevOps/System gondolkodás szempontjából sokkal értékesebb:

```text
értem az állapotot
↓
változtatok rajta
↓
ellenőrzöm az eredményt
```

mint a parancsok gyors bemásolása.
