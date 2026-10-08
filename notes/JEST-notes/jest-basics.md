# Jest alapok

## 1. Mi az a Jest?

A **Jest** egy JavaScript testing framework.

Arra használjuk, hogy automatikusan ellenőrizzük:

```text id="e1z1kp"
input
  ↓
function / program
  ↓
actual result
  ↓
compare
  ↓
expected result
```

Vagyis megadunk egy bemenetet, lefuttatjuk a programot vagy függvényt, majd ellenőrizzük, hogy a kapott eredmény megegyezik-e azzal, amit vártunk.

Példa:

```text id="3p9xw5"
2 + 3
  ↓
add(2, 3)
  ↓
5
  ↓
expected: 5
  ↓
PASS
```

**English:**
> Jest checks if the code works as expected.

*A Jest ellenőrzi, hogy a kód a várt módon működik-e.*

---

# 2. Miért használunk teszteket?

A tesztek segítenek észrevenni, ha egy későbbi módosítás elront egy korábban működő funkciót.

Például van egy függvényünk:

```javascript id="cdx2yn"
function add(a, b) {
  return a + b;
}
```

Ha később véletlenül így módosítjuk:

```javascript id="8j9r7a"
function add(a, b) {
  return a - b;
}
```

akkor a teszt azonnal jelezheti, hogy valami elromlott.

```text id="wcb7v4"
Expected: 5
Received: -1

FAIL
```

A teszt tehát egyfajta automatikus ellenőrzés.

**English:**
> A test can detect a broken feature.

*Egy teszt képes észlelni egy elromlott funkciót.*

---

# 3. Jest telepítése

A projektben:

```bash id="f6m9pc"
npm install --save-dev jest
```

A `--save-dev` azt jelenti, hogy a Jest fejlesztői függőségként kerül a projektbe.

A `package.json`-ban például:

```json id="w6a0pd"
"devDependencies": {
  "jest": "..."
}
```

A fejlesztői függőség olyan csomag, amelyre főleg fejlesztés vagy tesztelés közben van szükség.

---

# 4. Test script a package.json-ban

A `package.json`:

```json id="q6mn4d"
"scripts": {
  "test": "jest"
}
```

Ez azt jelenti:

```text id="kzo8ue"
npm test
   ↓
jest
```

Tehát:

```bash id="z61j9g"
npm test
```

lefuttatja a Jestet.

---

# 5. Egyszerű projektstruktúra

A laborunk:

```text id="lcy2jn"
git-github-actions-lab/
├── src/
│   └── calculator.js
├── test/
│   └── calculator.test.js
├── package.json
└── package-lock.json
```

A program:

```text id="hu09y0"
src/calculator.js
```

A teszt:

```text id="6l3efc"
test/calculator.test.js
```

A tesztfájl neve általában például:

```text id="e13ytj"
calculator.test.js
```

vagy:

```text id="yw6zmd"
calculator.spec.js
```

A Jest ezeket könnyen felismeri tesztfájlként.

---

# 6. A tesztelendő függvény

`src/calculator.js`:

```javascript id="e1d8cj"
function add(a, b) {
  return a + b;
}

module.exports = {
  add,
};
```

A `module.exports` miatt a függvényt másik fájlból is be tudjuk tölteni.

---

# 7. Függvény importálása a tesztbe

A tesztfájl:

```javascript id="gwl9h3"
const { add } = require("../src/calculator");
```

A `require()` betölti a `calculator.js` által exportált értékeket.

A:

```javascript id="u3f6kf"
{ add }
```

destructuring segítségével csak az `add` függvényt vesszük ki.

Az útvonal:

```text id="d1wsea"
test/calculator.test.js
       ↓
      ..
       ↓
project root
       ↓
src/calculator.js
```

A `../` tehát egy szinttel feljebb lép.

---

# 8. Az első Jest teszt

```javascript id="98xiwy"
test("2 + 3 should equal 5", () => {
  expect(add(2, 3)).toBe(5);
});
```

Ezt bontsuk részekre.

---

# 9. `test()`

```javascript id="vezf94"
test("2 + 3 should equal 5", () => {
```

A `test()` egy tesztesetet definiál.

Két fontos paramétere van.

## Első paraméter

```javascript id="c5fshc"
"2 + 3 should equal 5"
```

Ez a teszt neve.

Arra szolgál, hogy később könnyen felismerjük, melyik teszt sikerült vagy hibázott.

**English:**
> Two plus three should equal five.

---

## Második paraméter

```javascript id="2imqa8"
() => {
  ...
}
```

Ez egy függvény.

A Jest ezt futtatja le a teszt során.

---

# 10. `expect()`

A teszt egyik legfontosabb része:

```javascript id="svsjfg"
expect(add(2, 3))
```

Az `expect()` megkapja a tényleges eredményt.

Ebben az esetben:

```text id="c4a9f4"
add(2, 3)
   ↓
5
```

Tehát:

```javascript id="ijhkwm"
expect(5)
```

---

# 11. Matcher

A:

```javascript id="54ncqa"
.toBe(5)
```

egy **matcher**.

A matcher azt mondja meg, mit várunk az eredménytől.

Ebben az esetben:

```text id="7rxqjz"
actual value = 5
expected value = 5
```

Ezért:

```text id="kim0ib"
PASS
```

Ha például:

```javascript id="5wuhy0"
expect(add(2, 3)).toBe(6);
```

akkor:

```text id="uiueoy"
actual = 5
expected = 6
```

eredmény:

```text id="pj9w17"
FAIL
```

---

# 12. Alap Jest mental model

```text id="tg53tz"
test()
  ↓
run code
  ↓
expect(actual)
  ↓
matcher(expected)
  ↓
PASS / FAIL
```

Például:

```javascript id="42mtyn"
test("addition works", () => {
  expect(add(2, 3)).toBe(5);
});
```

Felbontva:

```text id="bn99j7"
add(2,3)
   ↓
actual result = 5
   ↓
expect(5)
   ↓
toBe(5)
   ↓
PASS
```

---

# 13. Néhány alap matcher

## `toBe()`

Egyszerű értékek összehasonlítása.

```javascript id="55ijh0"
expect(add(2, 3)).toBe(5);
```

Jól használható például:
- number
- string
- boolean

---

## `toEqual()`

Objektumok és tömbök tartalmának összehasonlításánál gyakori.

```javascript id="ixjvie"
expect([1, 2, 3]).toEqual([1, 2, 3]);
```

Egyszerű mental model:

```text id="7ehu1e"
toBe()
→ simple value / identity-style comparison

toEqual()
→ object or array contents
```

---

## `toBeTruthy()`

Azt ellenőrzi, hogy az érték JavaScript szerint truthy-e.

```javascript id="4xw2qg"
expect(true).toBeTruthy();
```

---

## `toBeFalsy()`

Falsy érték ellenőrzése.

```javascript id="0e8w1q"
expect(false).toBeFalsy();
```

---

## `toBeGreaterThan()`

```javascript id="rvu8kn"
expect(10).toBeGreaterThan(5);
```

---

## `toContain()`

Például tömbnél:

```javascript id="wrnmqz"
expect(["linux", "git", "docker"]).toContain("git");
```

---

# 14. Több teszt ugyanahhoz a függvényhez

Nem elég mindig csak egyetlen esettel tesztelni.

Például:

```javascript id="ajqx3p"
test("2 + 3 should equal 5", () => {
  expect(add(2, 3)).toBe(5);
});

test("-2 + 3 should equal 1", () => {
  expect(add(-2, 3)).toBe(1);
});

test("0 + 0 should equal 0", () => {
  expect(add(0, 0)).toBe(0);
});
```

Így több helyzetet ellenőrzünk.

Ezeket hívhatjuk **test cases**-nek.

**English:**
> We test different input values.

*Különböző bemeneti értékeket tesztelünk.*

---

# 15. `describe()`

Ha sok teszt tartozik ugyanahhoz a funkcióhoz, csoportosíthatjuk őket.

```javascript id="c5dxym"
describe("add()", () => {
  test("adds positive numbers", () => {
    expect(add(2, 3)).toBe(5);
  });

  test("adds negative numbers", () => {
    expect(add(-2, -3)).toBe(-5);
  });
});
```

A `describe()` nem kötelező.

A szerepe:

```text id="ft0oic"
describe
   ↓
group related tests
```

Vagyis kapcsolódó teszteket csoportosít.

---

# 16. A Jest futás kimenete

A laborban ezt kaptuk:

```text id="m8s2us"
PASS  test/calculator.test.js
  ✓ 2 + 3 should equal 5

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        0.295 s
Ran all test suites.
```

Nézzük meg.

## `PASS`

```text id="2fc0ox"
PASS test/calculator.test.js
```

A tesztfájl összességében sikeresen lefutott.

---

## `✓`

```text id="6xhay2"
✓ 2 + 3 should equal 5
```

Az adott teszteset sikeres.

---

## Test Suites

```text id="6g747f"
Test Suites: 1 passed, 1 total
```

Egy tesztfájlt / tesztcsoportot futtatott, és az sikerült.

---

## Tests

```text id="57n45e"
Tests: 1 passed, 1 total
```

Egy teszteset futott le.

---

## Snapshots

```text id="5utdyj"
Snapshots: 0 total
```

Ebben a projektben snapshot tesztet nem használunk.

Ezzel most nem kell mélyebben foglalkozni.

---

# 17. Mi történik hibánál?

Tegyük fel:

```javascript id="5xp2tr"
test("2 + 3 should equal 6", () => {
  expect(add(2, 3)).toBe(6);
});
```

A Jest valami ehhez hasonlót jelez:

```text id="ufqggu"
Expected: 6
Received: 5
```

Ez nagyon fontos.

A tesztlog segít megtalálni:
- melyik teszt hibázott,
- mit vártunk,
- mit kaptunk,
- melyik sorban történt a probléma.

Ez később a GitHub Actions laborban különösen fontos lesz.

---

# 18. Unit test

A mostani tesztünk egy egyszerű **unit test**.

A unit test egy kis, elkülönített programrészt tesztel.

Nálunk:

```text id="07127c"
unit
 ↓
add()
```

Tehát nem az egész alkalmazást vizsgáljuk, csak egy függvényt.

Példa:

```javascript id="6mwhb6"
expect(add(2, 3)).toBe(5);
```

---

# 19. Unit test vs integration test

Most még csak a különbséget érdemes érteni.

## Unit test

Egy kisebb egységet tesztel.

```text id="lprzyw"
function
   ↓
test
```

Például:

```text id="mcfv90"
add()
subtract()
calculateTotal()
```

## Integration test

Több komponens együttműködését teszteli.

Például:

```text id="ujsqu7"
API
 ↓
application
 ↓
database
```

Egy későbbi DevOps projektben például azt tesztelhetjük, hogy az alkalmazás valóban eléri-e az adatbázist.

---

# 20. Mi köze ennek a DevOps-hoz?

A DevOps mérnöknek nem feltétlenül az a feladata, hogy az összes unit tesztet ő írja.

Viszont gyakran neki kell kialakítania vagy karbantartania azt a folyamatot, amely ezeket automatikusan lefuttatja.

Például:

```text id="5cdgmp"
Developer pushes code
        ↓
GitHub
        ↓
GitHub Actions
        ↓
npm ci
        ↓
npm test
        ↓
Jest
        ↓
PASS / FAIL
```

Ez már **CI – Continuous Integration**.

Ha a Jest teszt hibázik:

```text id="xsyohr"
Jest FAIL
   ↓
npm test exits with error
   ↓
GitHub Actions step fails
   ↓
Job fails
   ↓
Workflow becomes red
```

Ha sikerül:

```text id="9r283r"
Jest PASS
   ↓
npm test success
   ↓
CI check success
   ↓
green check
```

Ezért fontos nekünk érteni legalább a tesztelés alapját.

---

# 21. Test exit code és CI

Parancssori programok a futás végén egy exit code-ot adnak vissza.

Általában:

```text id="afon8q"
0 → success
nem 0 → error
```

Ha a Jest minden tesztje sikerül:

```text id="t806a9"
exit code 0
```

Ha valamelyik teszt hibázik:

```text id="cu0zpt"
non-zero exit code
```

A CI rendszer ezt képes érzékelni.

Ezért tudja a GitHub Actions megállapítani:

```text id="txle37"
npm test
   ↓
successful?
   ↓
YES → continue
NO  → fail job
```

Ez az egyik legfontosabb kapcsolat a Jest és a DevOps között.

---

# 22. `npm test` és `npm ci`

Később a GitHub Actionsban gyakran ezt fogjuk látni:

```bash id="dxh5zy"
npm ci
npm test
```

## `npm ci`

A projekt függőségeit telepíti a `package-lock.json` alapján.

CI környezetben erre tervezték.

## `npm test`

Lefuttatja a `package.json` `test` scriptjét.

Nálunk:

```text id="xg8zb1"
npm test
   ↓
jest
```

A teljes folyamat:

```text id="t0xv7l"
fresh CI runner
      ↓
npm ci
      ↓
node_modules
      ↓
npm test
      ↓
Jest
      ↓
PASS / FAIL
```

---

# 23. Jó tesztnév

A teszt neve legyen érthető.

Kevésbé jó:

```javascript id="tnrbnx"
test("test1", () => {
```

Jobb:

```javascript id="y9564r"
test("2 + 3 should equal 5", () => {
```

Még általánosabb:

```javascript id="yjqv0d"
test("add returns the sum of two numbers", () => {
```

**English:**
> The test name should describe the expected behaviour.

*A tesztnév írja le az elvárt működést.*

---

# 24. Arrange – Act – Assert

Egy gyakori tesztstruktúra:

```text id="4j47qu"
Arrange
  ↓
Act
  ↓
Assert
```

## Arrange

Előkészítjük az adatokat.

```javascript id="gtg89s"
const a = 2;
const b = 3;
```

## Act

Lefuttatjuk a vizsgált kódot.

```javascript id="fejri5"
const result = add(a, b);
```

## Assert

Ellenőrizzük az eredményt.

```javascript id="cf043l"
expect(result).toBe(5);
```

Teljes példa:

```javascript id="xac9b4"
test("add returns the sum of two numbers", () => {
  const a = 2;
  const b = 3;

  const result = add(a, b);

  expect(result).toBe(5);
});
```

Ez ugyanazt csinálja, mint:

```javascript id="1ow1u2"
expect(add(2, 3)).toBe(5);
```

csak olvashatóbban szétválasztva.

---

# 25. Alap hibakeresési stratégia

Ha egy Jest teszt hibázik:

```text id="96xx3x"
1. Which test failed?
2. What was expected?
3. What was received?
4. Which line failed?
5. Is the test wrong or is the code wrong?
```

Magyarul:

```text id="l7meey"
1. Melyik teszt hibázott?
2. Mit várt?
3. Mit kapott?
4. Melyik sor hibázott?
5. A teszt rossz vagy a programkód rossz?
```

Ez az utolsó kérdés különösen fontos.

Egy piros teszt nem mindig azt jelenti, hogy a program rossz.

Lehet, hogy maga a teszt rosszul van megírva.

---

# 26. Fontos angol szókincs

| English | Magyar |
|---|---|
| test | teszt |
| testing | tesztelés |
| test case | teszteset |
| test suite | tesztkészlet |
| unit test | egységteszt |
| integration test | integrációs teszt |
| expected value | várt érték |
| actual value | tényleges érték |
| matcher | összehasonlító ellenőrzés |
| pass | sikerül |
| fail | megbukik / hibázik |
| assertion | ellenőrző állítás |
| dependency | függőség |
| dev dependency | fejlesztői függőség |
| behaviour | működés / viselkedés |
| test result | teszteredmény |
| exit code | kilépési kód |
| test runner | tesztfuttató |
| automated test | automatizált teszt |

---

# 27. B1 technikai mondatok

- Jest is a JavaScript testing framework.
- We use tests to check our code.
- This test checks the `add` function.
- The expected result is five.
- The actual result is five.
- The test passed.
- The test failed.
- Jest shows the failed test in the output.
- Unit tests check small parts of the application.
- GitHub Actions can run our tests automatically.
- The CI job fails if the test fails.
- `npm test` runs the Jest test suite.

---

# 28. Rövid összefoglalás

A jelenlegi projektben:

```text id="t3byd6"
calculator.js
     ↓
contains add()
     ↓
calculator.test.js
     ↓
calls add()
     ↓
expect(...)
     ↓
toBe(...)
     ↓
PASS / FAIL
```

A Jest feladata:

```text id="8cnt39"
run tests
   ↓
compare actual result
with expected result
   ↓
report result
```

Később:

```text id="owjyap"
GitHub Actions
       ↓
npm ci
       ↓
npm test
       ↓
Jest
       ↓
PASS / FAIL
```

Így kapcsolódik össze a fejlesztői tesztelés a CI/CD folyamattal.

---

# 29. Ellenőrző kérdések

1. Mi a Jest?
2. Miért használunk automatizált teszteket?
3. Mit jelent a `--save-dev`?
4. Mit csinál az `npm test`?
5. Mit csinál a `test()`?
6. Mire való az `expect()`?
7. Mi az a matcher?
8. Mit ellenőriz a `toBe()`?
9. Mi a különbség a tényleges és a várt eredmény között?
10. Mit jelent a PASS?
11. Mit jelent a FAIL?
12. Mi az a unit test?
13. Mi a különbség unit és integration test között?
14. Mire szolgál a `describe()`?
15. Mit jelent az Arrange – Act – Assert?
16. Miért fontos az exit code CI környezetben?
17. Mi történik GitHub Actionsban, ha `npm test` hibával lép ki?
18. Miért hasznos a DevOps mérnöknek érteni a teszteket?
19. Mit néznél meg először egy hibás Jest futásnál?
20. Mi a kapcsolat a Jest és a CI között?

---

# 30. Amit egyelőre nem kell mélyen tudni

Később külön is foglalkozhatunk ezekkel:

```text id="h038xm"
mock
spy
beforeEach
afterEach
async tests
promises
API testing
snapshot testing
coverage
```

A jelenlegi DevOps/GitHub Actions laborhoz elég stabilan érteni:

```text id="rslzk4"
test()
expect()
matcher
PASS / FAIL
npm test
exit code
CI connection
```