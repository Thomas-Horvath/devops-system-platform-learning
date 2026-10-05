# GitHub Actions és CI/CD alapok

## 1. Mi a GitHub Actions?

A **GitHub Actions** a GitHub automatizációs rendszere.

Lehetővé teszi, hogy GitHub események hatására automatikus feladatokat futtassunk.

Például:

```text
Pull Request
↓
GitHub Actions
↓
dependency install
↓
test
↓
build
↓
eredmény
```

A GitHub Actions segítségével automatizálható például:

- tesztelés,
- build,
- lint,
- security ellenőrzés,
- Docker image készítés,
- deployment,
- package publikálás,
- release folyamat.

---

# 2. A GitHub Actions fő felépítése

Alapmodell:

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
ACTION vagy RUN parancs
```

---

# 3. Event

Az **event** egy GitHub esemény, amely elindíthat egy workflow-t.

Példák:

```text
push
pull_request
release
issue
schedule
manual trigger
```

Workflow-ban:

```yaml
on:
  push:
```

Ez azt jelenti:

```text
push esemény
↓
workflow indul
```

Több event:

```yaml
on:
  push:
  pull_request:
```

---

# 4. Workflow

A **workflow** egy automatizált folyamat.

Workflow példa:

```text
Node.js CI

checkout
↓
Node setup
↓
dependency install
↓
test
↓
build
```

Egy repositoryban több workflow is lehet.

Például:

```text
test.yml
deploy.yml
security.yml
```

---

# 5. Workflow fájl helye

A workflow fájlok:

```text
.github/workflows/
```

könyvtárban találhatók.

Példa:

```text
project/
├── src/
├── package.json
└── .github/
    └── workflows/
        └── ci.yml
```

---

# 6. YAML

A GitHub Actions workflow-k YAML formátumú fájlok.

Példa:

```yaml
name: Node CI

on:
  push:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Hello"
```

A YAML struktúráját a behúzás határozza meg.

---

# 7. Job

Egy workflow egy vagy több **jobból** áll.

Például:

```text
Workflow

├── test
├── build
└── deploy
```

YAML:

```yaml
jobs:
  test:
    ...

  build:
    ...

  deploy:
    ...
```

Egy job egy runner gépen futó feladatcsoport.

---

# 8. Runner

A **runner** az a gép vagy környezet, amelyen a job ténylegesen fut.

Példa:

```yaml
runs-on: ubuntu-latest
```

Ez azt jelenti:

```text
job
↓
GitHub által biztosított
Ubuntu runner
```

A runner lehet:

```text
GitHub-hosted
vagy
self-hosted
```

---

# 9. GitHub-hosted runner

GitHub ideiglenes futtatási környezetet biztosít.

Például:

```text
ubuntu-latest
windows-latest
macos-latest
```

Életciklusa:

```text
job indul
↓
runner létrejön
↓
steps lefutnak
↓
job befejeződik
↓
runner eldobható
```

---

# 10. Self-hosted runner

A runner lehet saját infrastruktúrán is.

Például:

```text
saját Ubuntu server
DigitalOcean droplet
céges szerver
```

Ezt:

```text
self-hosted runner
```

néven használjuk.

---

# 11. Step

A job **step-ekből** áll.

Például:

```yaml
steps:
  - run: npm ci
  - run: npm test
  - run: npm run build
```

A step-ek általában sorban futnak.

```text
step 1
↓
step 2
↓
step 3
```

---

# 12. run

A:

```yaml
run:
```

shell parancsot futtat a runneren.

Például:

```yaml
- run: npm test
```

Ez a runneren:

```bash
npm test
```

parancsot futtatja.

Linux példa:

```yaml
- run: ls -la
```

---

# 13. Action

Az **Action** egy újrafelhasználható, előre elkészített művelet.

Példa:

```yaml
uses: actions/checkout@v6
```

Ez checkoutolja a repository kódját a runnerre.

---

# 14. uses és run különbsége

```text
uses
=
kész Action használata
```

Példa:

```yaml
uses: actions/checkout@v6
```

```text
run
=
saját shell parancs
```

Példa:

```yaml
run: npm test
```

---

# 15. Checkout

A runner induláskor nem egyszerűen a saját local repositorynk másolata.

A projekt kódját checkoutolni kell.

```yaml
- uses: actions/checkout@v6
```

Folyamat:

```text
runner
↓
checkout
↓
repository kódja
```

---

# 16. Node.js környezet

Node.js projektnél Node környezet szükséges.

Példa:

```yaml
- uses: actions/setup-node@v7
  with:
    node-version: 20
```

Ez beállítja:

```text
Node.js
npm
```

környezetet a job számára.

---

# 17. Első Node.js CI workflow

```yaml
name: Node.js CI

on:
  push:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v6

      - name: Setup Node.js
        uses: actions/setup-node@v7
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build
```

---

# 18. name

```yaml
name: Node.js CI
```

A workflow neve.

Ez jelenik meg a GitHub Actions felületen.

---

# 19. on

```yaml
on:
  push:
  pull_request:
```

A workflow elindul:

```text
push
VAGY
Pull Request
```

eseményre.

---

# 20. jobs

```yaml
jobs:
```

Itt definiáljuk a workflow jobjait.

---

# 21. Job ID

```yaml
jobs:
  test:
```

A:

```text
test
```

a job azonosítója.

Lehetne például:

```text
build
ci
deploy
```

is.

---

# 22. runs-on

```yaml
runs-on: ubuntu-latest
```

A job Ubuntu runneren fut.

---

# 23. npm ci

CI környezetben Node.js projekt esetén gyakori:

```bash
npm ci
```

Ez a lockfile alapján telepíti a dependency-ket.

Folyamat:

```text
package-lock.json
↓
meghatározott dependency verziók
↓
reprodukálható telepítés
```

---

# 24. Test

Példa:

```yaml
- run: npm test
```

Ez a `package.json` megfelelő scriptjét futtatja.

Például:

```json
{
  "scripts": {
    "test": "vitest"
  }
}
```

---

# 25. Build

Példa:

```yaml
- run: npm run build
```

Next.js projektben például:

```text
next build
```

futhat.

Ha a build hibás:

```text
workflow failed
```

Ha sikeres:

```text
workflow successful
```

---

# 26. Continuous Integration – CI

A **CI = Continuous Integration**.

Lényege:

```text
kódváltozás
↓
automatikus ellenőrzés
↓
test
↓
build
↓
azonnali visszajelzés
```

Példa:

```text
developer
↓
git push
↓
GitHub
↓
Actions
↓
test
↓
build
```

---

# 27. Pull Request és CI

Tipikus folyamat:

```text
feature branch
↓
push
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
```

Siker:

```text
✓ dependencies
✓ tests
✓ build
```

Hiba:

```text
✗ tests
```

A workflow eredménye megjelenhet a Pull Requestben.

---

# 28. Több job

Példa:

```yaml
jobs:

  test:
    runs-on: ubuntu-latest
    steps:
      ...

  build:
    runs-on: ubuntu-latest
    steps:
      ...

  deploy:
    runs-on: ubuntu-latest
    steps:
      ...
```

Mentális modell:

```text
WORKFLOW

├── TEST
├── BUILD
└── DEPLOY
```

---

# 29. Job dependency

Egy job függhet másik job sikerétől.

Példa:

```yaml
build:
  needs: test
```

Ez:

```text
test
↓
siker
↓
build
```

Deployment:

```yaml
deploy:
  needs: build
```

Teljes folyamat:

```text
test
↓
build
↓
deploy
```

---

# 30. Continuous Delivery / Deployment – CD

A CI után következhet deployment.

```text
CI

test
↓
build
```

majd:

```text
CD

build
↓
deployment
↓
environment
```

Egyszerű pipeline:

```text
git push
↓
test
↓
build
↓
deploy
```

---

# 31. CI/CD pipeline

Egy tipikus pipeline:

```text
Developer
↓
Git
↓
GitHub
↓
GitHub Actions
↓
TEST
↓
BUILD
↓
PACKAGE
↓
DEPLOY
```

---

# 32. DevOps példa

Egy későbbi DevOps workflow lehet:

```text
Developer push
↓
GitHub Actions
↓
checkout
↓
Node setup
↓
npm ci
↓
lint
↓
tests
↓
build
↓
Docker image
↓
container registry
↓
deployment
↓
server / cloud
```

---

# 33. Secrets

Titkos adatokat nem szabad közvetlenül workflow fájlba írni.

Például:

```text
DATABASE_URL
API_KEY
SERVER_PASSWORD
TOKEN
```

Nem:

```yaml
password: my-secret-password
```

Hanem GitHub Secret használható.

Példa:

```yaml
${{ secrets.SERVER_PASSWORD }}
```

---

# 34. GITHUB_TOKEN

A GitHub workflow futása során rendelkezésre állhat egy automatikusan generált:

```text
GITHUB_TOKEN
```

token.

Ezzel a workflow bizonyos GitHub műveleteket végezhet.

A token jogosultságait biztonsági okból célszerű korlátozni.

---

# 35. Workflow eredmény

GitHub Actions felület:

```text
Workflow run
↓
Job
↓
Step
↓
Log
```

Példa:

```text
Node.js CI

✓ Checkout repository
✓ Setup Node.js
✓ Install dependencies
✗ Run tests
```

A hibás step logjai megnyithatók.

---

# 36. Troubleshooting

Hibánál érdemes ebben a sorrendben gondolkodni:

```text
Melyik workflow?
↓
Melyik job?
↓
Melyik step?
↓
Milyen parancs futott?
↓
Mi az error message?
```

Ez DevOps munkában alapvető hibakeresési szemlélet.

---

# 37. GitHub Actions vs Action

Fontos különbség:

```text
GitHub Actions
=
az egész automatizációs platform
```

```text
Action
=
egy újrafelhasználható művelet
```

Példa Action:

```text
actions/checkout
```

---

# 38. Teljes mentális modell

```text
GitHub EVENT
(push / PR / release)
        │
        ▼
WORKFLOW
        │
        ├── JOB: test
        │      │
        │      ▼
        │    RUNNER
        │      │
        │      ├── STEP: checkout
        │      ├── STEP: setup Node
        │      ├── STEP: npm ci
        │      └── STEP: npm test
        │
        └── JOB: deploy
               │
               ▼
             RUNNER
               │
               └── deployment
```

---

# Rövid összefoglalás

```text
Event
=
mi indítja el?
```

```text
Workflow
=
teljes automatizált folyamat
```

```text
Job
=
feladatcsoport
```

```text
Runner
=
gép, amin a job fut
```

```text
Step
=
a job egy lépése
```

```text
Action
=
újrafelhasználható kész művelet
```

```text
run
=
shell parancs futtatása
```

```text
CI
=
automatikus integrációs ellenőrzés,
például test + build
```

```text
CD
=
a kész eredmény további
szállítása vagy deploymentje
```

---

# Legfontosabb gondolat

A GitHub Actions lényege:

```text
ESEMÉNY
↓
AUTOMATIZÁLT FOLYAMAT
↓
ELLENŐRZÉS / BUILD / DEPLOYMENT
```

A DevOps Engineer így nem kézzel ismétli ugyanazokat a lépéseket minden változtatás után.

A folyamatot egyszer definiálja:

```text
workflow as code
```

formában, és a rendszer automatikusan, reprodukálható módon hajtja végre.