# GitHub workflow, Pull Request és csapatmunka

## 1/a. Mi a GitHub workflow?

A GitHub workflow azt a folyamatot jelenti, ahogyan egy csapat a Git repositoryban együtt dolgozik.

Egy tipikus fejlesztési folyamat:

```text
Issue
↓
feature branch
↓
development
↓
commit
↓
push
↓
Pull Request
↓
Code Review
↓
automatikus checks
↓
Approval
↓
Merge
↓
main
```

A cél az, hogy a `main` branchbe ne kerüljenek ellenőrizetlen változtatások.

---

## 1/b. GitHub Issue

A **GitHub Issue** egy feladat, hiba, fejlesztési igény vagy ötlet nyilvántartására szolgáló elem.

Egyszerűen:

```text
Issue
=
valami, amit a projektben meg kell oldani,
meg kell javítani vagy meg kell valósítani
```

Például:

```text
Issue #42

Add user login
```

vagy:

```text
Issue #43

Fix database connection timeout
```

Az Issue nem maga a kódváltoztatás.

Az Issue azt írja le:

```text
MI a probléma vagy feladat?
```

A branch és a Pull Request pedig azt mutatja:

```text
HOGYAN oldottuk meg?
```

---

### Mire használható egy Issue?

Például:

```text
bug
feature request
fejlesztési feladat
technikai adósság
dokumentációs feladat
security probléma
üzemeltetési feladat
```

DevOps projektben például ilyen Issue-k lehetnek:

```text
Configure Nginx reverse proxy
```

```text
Add PostgreSQL backup
```

```text
Create CI pipeline
```

```text
Fix failed production deployment
```

---

### Mit tartalmazhat egy Issue?

Egy Issue-ban lehet például:

```text
Title
Description
Assignee
Labels
Milestone
Comments
Links to Pull Requests
```

#### Title

A feladat rövid neve.

Például:

```text
Add login functionality
```

#### Description

Részletesebb leírás arról, mit kell megoldani.

Például:

```text
A felhasználók tudjanak
email és jelszó segítségével belépni.

Elvárások:
- hibás jelszó esetén error
- sikeres belépés esetén token
- legyen hozzá unit test
```

#### Assignee

Az a személy, aki dolgozik a feladaton.

#### Labels

Az Issue kategorizálására használható.

Például:

```text
bug
feature
security
documentation
priority-high
```

#### Milestone

Nagyobb célhoz vagy release-hez rendelhető.

Például:

```text
Version 1.0
```

---

### Issue alapú fejlesztési workflow

Például létrejön:

```text
Issue #42
Add user login
```

Ezután készül hozzá branch:

```text
feature/login
```

A folyamat:

```text
Issue #42
↓
feature/login
↓
development
↓
commit
↓
push
↓
Pull Request
↓
review
↓
tests
↓
merge
↓
Issue lezárva
```

Így később vissza lehet keresni:

```text
Mi volt a feladat?
↓
Ki dolgozott rajta?
↓
Melyik branch tartozott hozzá?
↓
Milyen commitok készültek?
↓
Melyik Pull Request oldotta meg?
```

---

### Issue és Pull Request kapcsolata

Egy Pull Requestben hivatkozhatunk egy Issue-ra.

Például:

```text
Fixes #42
```

vagy:

```text
Closes #42
```

Ez azt jelenti, hogy a Pull Request az adott Issue megoldására készült.

Ha a PR megfelelően merge-elődik a megfelelő alapbranchbe, a GitHub az Issue-t automatikusan le is zárhatja.

Egyszerű modell:

```text
Issue #42
   │
   │ megoldja
   ▼
Pull Request #57
   │
   │ merge
   ▼
main
   │
   ▼
Issue #42 CLOSED
```

---

### Miért fontos az Issue?

Issue nélkül könnyen ilyen lehet a fejlesztés:

```text
"Valaki mondta Slack-en,
hogy javítsuk meg a login hibát."
```

Pár hét múlva senki nem tudja:

```text
ki kérte?
mi volt pontosan a hiba?
ki javította?
melyik commit volt?
```

Issue-val:

```text
feladat
↓
dokumentált
↓
hozzárendelve
↓
branch
↓
Pull Request
↓
merge
↓
lezárva
```

A teljes munka visszakövethető.

---

### DevOps szempontból

Az Issue nem csak fejlesztői feladatokra használható.

Például:

```text
Issue #101
Add monitoring for PostgreSQL
```

ebből lehet:

```text
Issue
↓
infra branch
↓
Prometheus config
↓
Pull Request
↓
review
↓
GitHub Actions validation
↓
merge
↓
monitoring deployment
```

Így az infrastruktúra-változtatások is ugyanúgy nyomon követhetők, mint az alkalmazáskód.

---

### Rövid memóriafogás

```text
Issue
=
MIT kell megoldani?

Branch
=
HOL dolgozunk rajta?

Commit
=
MIT változtattunk?

Pull Request
=
BE AKARJUK illeszteni?

Review
=
JÓ a megoldás?

Merge
=
BEKERÜL a fő branchbe.
```

# 2. Példa egy fejlesztési feladatra

Tegyük fel, hogy van egy Node.js alkalmazásunk.

A fő branch:

```text
main
```

Kapunk egy feladatot:

```text
Add user login
```

Nem közvetlenül a `main` branchen dolgozunk.

Létrehozunk egy feature branchet:

```bash
git switch -c feature/login
```

A projekt története például:

```text
A --- B --- C   main
             \
              D --- E   feature/login
```

A login fejlesztése a külön `feature/login` branchen történik.

---

# 3. Feature branch feltöltése GitHubra

Ha elkészültünk vagy szeretnénk megosztani a munkát:

```bash
git push -u origin feature/login
```

Ekkor GitHubon már létezik:

```text
main
feature/login
```

A `feature/login` azonban még nem része a `main` branchnek.

---

# 4. Pull Request

A **Pull Request**, röviden **PR**, egy kérés arra, hogy egy branch változtatásait egy másik branchbe illesszük.

Például:

```text
feature/login
      │
      │ Pull Request
      ▼
     main
```

A PR lényegében ezt mondja:

> Szeretném a `feature/login` branch változtatásait a `main` branchbe merge-elni.

A Pull Request lehetőséget ad arra, hogy a változtatást merge előtt:

- átnézzék,
- kommenteljék,
- teszteljék,
- automatikus ellenőrzések fussanak rajta,
- jóváhagyják.

---

# 5. Base és head branch

Pull Request esetén két fontos fogalom van.

## Base branch

Az a branch:

```text
AHOVÁ
```

a változtatásokat merge-elni szeretnénk.

Például:

```text
main
```

---

## Head branch

Az a branch:

```text
AHONNAN
```

a változtatások érkeznek.

Például:

```text
feature/login
```

Tehát:

```text
head:
feature/login

↓ Pull Request

base:
main
```

---

# 6. Mit tartalmaz egy Pull Request?

Egy Pull Requestben általában látható többek között:

```text
Conversation
Commits
Checks
Files changed
```

---

## Conversation

Itt található például:

- PR leírása,
- általános kommentek,
- review-k,
- státuszváltozások.

---

## Commits

Itt láthatók a Pull Requesthez tartozó commitok.

Például:

```text
Add login endpoint
Add password validation
Add login tests
```

---

## Files changed

Megmutatja:

```text
mely fájlok változtak
mely sorokat adták hozzá
mely sorokat törölték
```

Ez a code review egyik legfontosabb felülete.

---

## Checks

Itt jelenhetnek meg az automatikusan futó ellenőrzések.

Például:

```text
✓ lint
✓ unit tests
✓ build
✓ security scan
```

vagy:

```text
✗ unit tests
```

A checkeket például GitHub Actions workflow-k futtathatják.

---

# 7. Code Review

A **code review** során egy vagy több másik fejlesztő átnézi a változtatásokat.

Vizsgálhatják például:

```text
helyes működés
kódminőség
olvashatóság
biztonság
teljesítmény
tesztelhetőség
architektúra
```

A reviewer konkrét sorokhoz is írhat megjegyzéseket.

Például:

```text
"Ezt a password validation részt érdemes külön függvénybe tenni."
```

---

# 8. Mi történik review után?

A fejlesztő visszatér a saját branchére:

```text
feature/login
```

Módosítja a kódot.

Majd:

```bash
git add .
git commit -m "Refactor password validation"
git push
```

Fontos:

Nem kell új Pull Requestet készíteni.

A meglévő Pull Request automatikusan frissül, mert ugyanahhoz a branchhez tartozik.

Folyamat:

```text
Pull Request
↓
review
↓
javítás localban
↓
commit
↓
push
↓
Pull Request automatikusan frissül
↓
új review
```

---

# 9. Approval

Ha a reviewer megfelelőnek találja a változtatást:

```text
Approve
```

jóváhagyást adhat.

Ez azt jelenti:

```text
A változtatást szakmailag elfogadom.
```

Csapattól függően lehet például:

```text
1 approval szükséges
```

vagy:

```text
2 approval szükséges
```

a merge előtt.

---

# 10. Automatikus checks

A Pull Requesthez automatikus ellenőrzések kapcsolhatók.

Például:

```text
Pull Request
↓
install dependencies
↓
lint
↓
unit tests
↓
build
↓
security checks
```

Node.js projektnél például:

```bash
npm ci
npm run lint
npm test
npm run build
```

Ha minden sikeres:

```text
✓ lint
✓ test
✓ build
```

Ha valamelyik hibás:

```text
✗ test
```

akkor a PR nem tekinthető teljesen sikeresnek.

---

# 11. CI kapcsolat

Ez már a **Continuous Integration**, röviden **CI**, egyik tipikus példája.

A cél:

```text
minden fontos kódváltozás
↓
automatikusan ellenőrizve legyen
```

Példa:

```text
git push
↓
GitHub
↓
automatikus workflow
↓
tests
↓
build
↓
eredmény megjelenik a PR-ban
```

Ezt később GitHub Actions segítségével fogjuk megvalósítani.

---

# 12. Merge

Ha:

```text
Code Review OK
+
Approval OK
+
Checks OK
```

akkor a Pull Request merge-elhető.

Példa:

```text
A --- B --- C -------- M   main
         \            /
          D --- E ----
          feature/login
```

A merge után a login funkció már a `main` branch része.

---

# 13. Feature branch merge után

Ha a feature branch már bekerült a `main` branchbe, általában nincs rá tovább szükség.

GitHubon törölhető:

```text
Delete branch
```

Localban:

```bash
git branch -d feature/login
```

A branch törlése nem jelenti azt, hogy a merge-elt változtatások elvesznek.

A commitok már a `main` történetének részei.

---

# 14. Miért nem dolgozunk közvetlenül mainen?

Képzeljük el:

```text
Developer A
↓
push main

Developer B
↓
push main

Developer C
↓
push main
```

Ha mindenki közvetlenül a `main` branchre pusholhat, könnyen bekerülhet:

```text
hibás kód
nem tesztelt változtatás
build hiba
security probléma
nem review-zott kód
```

A Pull Request workflow kontrollpontot hoz létre:

```text
feature branch
↓
Pull Request
↓
review
↓
checks
↓
approval
↓
merge
↓
main
```

---

# 15. Branch protection

A fontos brancheket, például a:

```text
main
```

branchet védeni lehet.

Ez a **branch protection**.

Például beállítható:

```text
main

direct push:
TILTVA

Pull Request:
KÖTELEZŐ

Review:
KÖTELEZŐ

Status checks:
KÖTELEZŐ

Force push:
TILTVA

Delete:
TILTVA
```

Így valaki nem tudja egyszerűen megkerülni a workflow-t:

```bash
git push origin main
```

művelettel.

---

# 16. Miért fontos a branch protection?

Branch protection segítségével technikailag is kikényszeríthető a csapat szabályzata.

Például:

```text
main
↓
csak Pull Request
↓
legalább 1 approval
↓
tests sikeresek
↓
build sikeres
↓
merge engedélyezett
```

Ez különösen fontos nagyobb csapatoknál és production rendszereknél.

---

# 17. Rulesets

A GitHub újabb és rugalmasabb szabálykezelési lehetősége a:

```text
Rulesets
```

A ruleset segítségével szabályokat lehet alkalmazni például:

```text
branchekre
tagekre
repository műveletekre
```

Például:

```text
main branch
↓
PR required
↓
2 reviewers
↓
status checks required
↓
force push forbidden
```

A branch protection és a rulesets hasonló problémát old meg:

```text
fontos repository részek védelme
+
csapat szabályainak kikényszerítése
```

---

# 18. Issue

A GitHub **Issue** használható például:

```text
hiba
fejlesztési feladat
feature request
technikai feladat
ötlet
```

nyilvántartására.

Példa:

```text
Issue #42

Add login functionality
```

---

# 19. Issue és branch kapcsolata

Egy issue-ból indulhat egy fejlesztési folyamat:

```text
Issue #42
↓
feature/login
↓
development
↓
commit
↓
push
↓
Pull Request
↓
merge
↓
Issue lezárva
```

Ez segít követni:

```text
mi volt a feladat?
ki dolgozott rajta?
melyik branch tartozott hozzá?
melyik Pull Request oldotta meg?
```

---

# 20. Teljes csapat workflow

Tipikus GitHub-alapú fejlesztési folyamat:

```text
Issue
↓
feature branch
↓
development
↓
commit
↓
push
↓
Pull Request
↓
Code Review
↓
automatikus Checks
↓
Approval
↓
Merge
↓
main
↓
feature branch törlése
```

---

# 21. DevOps nézőpont

DevOps szempontból különösen fontos ez a rész:

```text
git push
↓
GitHub
↓
Pull Request
↓
GitHub Actions
↓
test
↓
build
↓
security checks
↓
merge
↓
deployment
```

Itt kapcsolódik össze:

```text
development
+
version control
+
testing
+
automation
+
deployment
```

---

# 22. Későbbi CI/CD példa

Egy későbbi workflow például így működhet:

```text
Developer
↓
git push feature branch
↓
Pull Request
↓
GitHub Actions
↓
npm ci
↓
npm test
↓
npm run build
↓
minden sikeres
↓
review
↓
merge main
↓
új GitHub Actions workflow
↓
production deployment
```

Ez már egy egyszerű:

```text
CI/CD pipeline
```

alapja.

---

# 23. Fontos fogalmak

```text
Pull Request
=
kérés egy branch változtatásainak
másik branchbe merge-elésére
```

```text
base branch
=
ahová merge-elünk
```

```text
head branch
=
ahonnan a változtatás jön
```

```text
Code Review
=
másik fejlesztő átnézi
a változtatásokat
```

```text
Approval
=
a reviewer elfogadja a változtatást
```

```text
Checks
=
automatikus ellenőrzések
```

```text
Merge
=
branchek változásainak egyesítése
```

```text
Branch protection
=
fontos branchek védelme
```

```text
Ruleset
=
repository szabályok
központi meghatározása
```

```text
Issue
=
feladat, hiba vagy fejlesztési igény
```

---

# 24. Rövid mentális modell

```text
FEJLESZTŐ

feature branch
      │
      │ push
      ▼
GITHUB
      │
      ▼
Pull Request
      │
      ├── Code Review
      │
      ├── Checks
      │
      └── Approval
              │
              ▼
             Merge
              │
              ▼
             main
```

---

# 25. Legfontosabb gondolat

A GitHub csapatmunka célja nem egyszerűen az, hogy:

```text
feltöltsük a kódot
```

hanem hogy a változtatás kontrollált folyamaton menjen keresztül:

```text
változtatás
↓
áttekintés
↓
automatikus ellenőrzés
↓
jóváhagyás
↓
merge
```

Ez csökkenti annak az esélyét, hogy hibás vagy ellenőrizetlen kód kerüljön a fő branchbe.

DevOps szempontból ez azért különösen fontos, mert erre a folyamatra később automatizált:

```text
tesztelés
build
security check
deployment
```

lépések építhetők.