# GitHub Basic és Authentication

## 1. Mi a GitHub?

A **GitHub** egy Git repository hosting és fejlesztési együttműködési platform.

Fontos különbség:

```text
Git
=
verziókezelő rendszer

GitHub
=
Git repository hosting
+
csapatmunka
+
jogosultságkezelés
+
automatizálás
+
projektkezelés
```

A Git helyben, internetkapcsolat nélkül is használható.

A GitHub egy távoli szolgáltatás, amely Git repositorykat tárol és további funkciókat biztosít.

---

# 2. Local Git és GitHub kapcsolata

Egyszerű modell:

```text
LOCAL COMPUTER

Git repository
     │
     │ push / pull / fetch
     ▼
remote: origin
     │
     ▼
GITHUB

remote repository
```

Példa:

```text
Local:

~/projects/my-project
```

GitHubon:

```text
github.com/example-user/my-project
```

A local repository és a GitHub repository két külön Git repository.

A remote kapcsolat köti össze őket.

---

# 3. Mit ad a GitHub a Githez?

A GitHub többek között:

```text
repository hosting
Pull Requests
code review
Issues
GitHub Actions
CI/CD
releases
permissions
security
team collaboration
```

funkciókat biztosít.

A Git tehát maga a verziókezelés.

A GitHub a Git köré épített együttműködési és automatizálási platform.

---

# 4. Authentication és authorization

GitHub használatakor két fontos fogalmat külön kell választani.

## Authentication

Jelentése:

```text
Ki vagy?
```

A GitHub ellenőrzi, hogy valóban az adott felhasználóként próbálunk-e kapcsolódni.

Példák:

```text
PAT
SSH key
GitHub CLI token
```

---

## Authorization

Jelentése:

```text
Mit szabad csinálnod?
```

Például:

```text
olvashatod a repositoryt?
pusholhatsz?
létrehozhatsz branchet?
adminisztrálhatod a repositoryt?
futtathatsz workflow-t?
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

# 5. GitHub Git authentication lehetőségek

Git repositoryk GitHubbal történő használatánál két fő módszer gyakori:

```text
1. HTTPS
2. SSH
```

HTTPS esetén jellemzően tokenalapú hitelesítés történik.

SSH esetén kulcspár segítségével történik a hitelesítés.

---

# 6. HTTPS authentication

HTTPS remote példa:

```text
https://github.com/example-user/my-project.git
```

Ha ezt használjuk:

```bash
git push
```

akkor a folyamat:

```text
Git
↓
HTTPS kapcsolat
↓
GitHub
↓
Authentication
↓
Authorization
↓
push
```

A GitHub a normál account jelszót nem használja Git parancssori HTTPS autentikációhoz.

Helyette például:

```text
Personal Access Token
GitHub CLI
credential manager
```

használható.

---

# 7. Personal Access Token – PAT

A **Personal Access Token**, röviden **PAT**, egy GitHub hozzáférési token.

Egyszerűen:

```text
PAT
=
egy titkos hozzáférési kulcs,
amellyel GitHub műveleteket végzünk
```

A PAT használható a normál jelszó helyett HTTPS Git műveleteknél.

Fontos:

```text
PAT ≠ account password
```

A PAT célja az, hogy:

- korlátozható legyen,
- visszavonható legyen,
- lejárati időt lehessen adni,
- csak bizonyos repositorykhoz férjen hozzá.

---

# 8. Fine-grained PAT

GitHub támogat részletesen korlátozható tokeneket.

Például létrehozhatunk egy tokent:

```text
Repository access:
my-project

Permissions:
Contents:
Read and write

Expiration:
90 days
```

Így a token nem feltétlen fér hozzá az összes repositoryhoz.

Ez biztonságosabb, mint egy túl széles jogosultságú token.

A token nem adhat több hozzáférést, mint amivel maga a GitHub-felhasználó rendelkezik.

---

# 9. PAT létrehozásának logikája

GitHub weboldalon:

```text
GitHub
↓
Settings
↓
Developer settings
↓
Personal access tokens
↓
Fine-grained tokens
↓
Generate new token
```

Itt beállítható például:

```text
Token name
Expiration
Repository access
Permissions
```

A token létrehozása után kapunk egy titkos karakterláncot.

Fontos:

> A PAT-et úgy kell kezelni, mint egy jelszót.

Nem szabad:

```text
forráskódba írni
Git repositoryba commitolni
README-be írni
chatbe bemásolni
nyilvánosan megosztani
```

---

# 10. Fiktív HTTPS + PAT példa

Tegyük fel, hogy a GitHub felhasználó:

```text
example-user
```

és a repository:

```text
devops-lab
```

GitHub repository címe:

```text
https://github.com/example-user/devops-lab.git
```

Local repositoryban remote hozzáadása:

```bash
git remote add origin https://github.com/example-user/devops-lab.git
```

Ellenőrzés:

```bash
git remote -v
```

Eredmény:

```text
origin  https://github.com/example-user/devops-lab.git (fetch)
origin  https://github.com/example-user/devops-lab.git (push)
```

Ezután:

```bash
git push -u origin main
```

A Git kérheti:

```text
Username:
```

Ide:

```text
example-user
```

kerül.

Ezután:

```text
Password:
```

Ide nem a GitHub account jelszavát írjuk.

Hanem:

```text
PERSONAL ACCESS TOKEN
```

A GitHub dokumentáció szerint HTTPS Git műveleteknél a PAT használható a password helyett.

---

# 11. Mi történik a háttérben?

```text
git push
↓
Git kapcsolatot nyit:
https://github.com
↓
GitHub credentialt kér
↓
username
+
PAT
↓
GitHub ellenőrzi a PAT-et
↓
melyik userhez tartozik?
↓
érvényes?
↓
van repository hozzáférése?
↓
van write permission?
↓
IGEN
↓
push végrehajtva
```

---

# 12. Credential

A **credential** hitelesítéshez használt információ.

HTTPS GitHub esetén például:

```text
username
+
PAT
```

együtt alkothat credentialt.

Ha a Git nem tárolja ezt valamilyen módon, minden push során újra kérheti.

---

# 13. Credential helper

A Git credential helper feladata:

```text
credential tárolása
vagy
credential előkeresése
```

Példa:

```text
git push
↓
Git
↓
credential helper
↓
credential
↓
GitHub
```

---

# 14. credential.helper store

Egyszerű megoldás:

```bash
git config --global credential.helper store
```

Ezután a Git el tudja menteni a beadott credentialt.

A probléma:

```text
store
=
a credential titkosítatlanul is tárolódhat
```

Ezért ez működőképes, de nem a legjobb hosszú távú biztonsági megoldás.

---

# 15. GitHub CLI – gh

A GitHub CLI külön program.

Parancsa:

```bash
gh
```

Fontos:

```text
git
≠
gh
```

A:

```text
git
```

a Git verziókezelő.

A:

```text
gh
```

a GitHub szolgáltatás parancssori kezelője.

---

# 16. Mire használható a gh?

Például:

```text
authentication
repository kezelés
Pull Requests
Issues
workflow-k
GitHub Actions
releases
API hívások
SSH key kezelés
```

---

# 17. gh auth login

GitHub CLI autentikáció:

```bash
gh auth login
```

A `gh` alapból böngészőalapú autentikációt tud használni, és ha elérhető megfelelő rendszer credential store, a hitelesítési tokent ott tárolja.

A parancs kérdezhet például:

```text
GitHub.com vagy Enterprise?
HTTPS vagy SSH?
Browser login?
```

---

# 18. gh auth status

A GitHub CLI aktuális authentication állapota:

```bash
gh auth status
```

Példa:

```text
Logged in to github.com account example-user
Git operations protocol: https
```

Ez azt bizonyítja, hogy a GitHub CLI autentikálva van.

---

# 19. gh és Git összekapcsolása

A:

```bash
gh auth setup-git
```

beállítja a Gitet úgy, hogy a GitHub CLI-t credential helperként használja.

A kívánt folyamat:

```text
git push
↓
Git
↓
credential helper
↓
gh auth git-credential
↓
GitHub CLI credential
↓
GitHub
```

Így nem kell minden alkalommal kézzel beírni a PAT-et.

---

# 20. Fiktív GitHub CLI HTTPS setup

Tegyük fel:

```text
GitHub account:
example-user
```

Első lépés:

```bash
gh auth login
```

Kiválasztjuk:

```text
GitHub.com
HTTPS
Login with browser
```

Sikeres login után:

```bash
gh auth status
```

Majd:

```bash
gh auth setup-git
```

Ez beállítja, hogy a Git használja a GitHub CLI credential helperét.

Ezután:

```bash
git push
```

folyamat:

```text
Git
↓
gh credential helper
↓
GitHub authentication
↓
push
```

---

# 21. SSH authentication

A másik fontos módszer:

```text
SSH
```

SSH esetén nem PAT segítségével autentikálunk.

Hanem egy:

```text
public key
+
private key
```

kulcspárral.

---

# 22. Public és private key

SSH kulcspár:

```text
PRIVATE KEY
+
PUBLIC KEY
```

## Private key

A private key:

```text
csak a saját gépen marad
```

SOHA nem szabad:

```text
GitHubra feltölteni
elküldeni másnak
repositoryba commitolni
chatbe bemásolni
```

---

## Public key

A public key:

```text
megosztható
```

Ezt adjuk hozzá a GitHub accountunkhoz.

---

# 23. SSH authentication működése

Egyszerűsített modell:

```text
LOCAL COMPUTER

private key
      │
      │ bizonyítja,
      │ hogy nálunk van
      ▼

GitHub

public key
```

A private key maga nem kerül elküldésre GitHubnak.

A titkos kulcs segítségével a kliens kriptográfiailag bizonyítja, hogy birtokolja a GitHubon regisztrált public key párját.

---

# 24. SSH kulcsok helye Linuxon

SSH kulcsok általában:

```text
~/.ssh/
```

könyvtárban vannak.

Például:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

Ebben:

```text
id_ed25519
=
PRIVATE KEY

id_ed25519.pub
=
PUBLIC KEY
```

A `.pub` jelzi a public keyt.

---

# 25. Meglévő SSH kulcsok ellenőrzése Linuxon

Először:

```bash
ls -al ~/.ssh
```

GitHub dokumentáció szerint itt érdemes keresni például:

```text
id_ed25519
id_ed25519.pub

id_rsa
id_rsa.pub
```

kulcspárokat.

Ha:

```text
~/.ssh
```

nem létezik, valószínűleg még nincs default helyen SSH kulcspárunk.

---

# 26. SSH kulcs generálása Linuxon

Modern alapértelmezett választás:

```text
Ed25519
```

Kulcs létrehozása:

```bash
ssh-keygen -t ed25519 -C "example@example.com"
```

A `-C` után megadott email csak egy megjegyzés/label a kulcshoz.

GitHub is az Ed25519 használatát mutatja elsődleges példaként modern rendszereken.

---

# 27. ssh-keygen kérdések

A parancs után:

```text
Enter file in which to save the key
(/home/user/.ssh/id_ed25519):
```

Ha megfelel az alapértelmezett hely:

```text
ENTER
```

Ezután:

```text
Enter passphrase:
```

Itt opcionálisan megadható jelszó a private key védelmére.

Biztonságosabb:

```text
private key
+
passphrase
```

Ha passphrase van, a kulcs ellopása önmagában kevésbé veszélyes.

---

# 28. Létrejövő fájlok

Példa:

```text
/home/user/.ssh/id_ed25519
/home/user/.ssh/id_ed25519.pub
```

Private:

```text
id_ed25519
```

Public:

```text
id_ed25519.pub
```

---

# 29. SSH agent

Ha a private key passphrase-szel védett, nem akarjuk minden Git műveletnél újra beírni.

Erre használható:

```text
ssh-agent
```

Az ssh-agent a kulcsokat kezeli a futó session során.

Linuxon indítás:

```bash
eval "$(ssh-agent -s)"
```

Példa eredmény:

```text
Agent pid 12345
```

GitHub dokumentáció szintén ezt a mintát használja Linux környezetben.

---

# 30. Private key hozzáadása az ssh-agenthez

```bash
ssh-add ~/.ssh/id_ed25519
```

Ekkor az agent kezeli a kulcsot.

Ha van passphrase:

```text
Enter passphrase:
```

egyszer megadhatjuk.

---

# 31. Public key megtekintése

```bash
cat ~/.ssh/id_ed25519.pub
```

Példa:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... example@example.com
```

EZ A PUBLIC KEY.

Ezt lehet GitHubhoz hozzáadni.

A private keyt:

```text
~/.ssh/id_ed25519
```

nem másoljuk sehova.

---

# 32. Public key hozzáadása GitHubon

GitHub weboldalon:

```text
Profile
↓
Settings
↓
SSH and GPG keys
↓
New SSH key
```

Megadunk:

```text
Title:
Personal Linux Laptop

Key type:
Authentication Key

Key:
a teljes public key
```

A GitHub account ezután ismeri ezt a gépet/kulcsot.

---

# 33. SSH kapcsolat tesztelése

```bash
ssh -T git@github.com
```

Első alkalommal megjelenhet host fingerprint kérdés.

Például:

```text
Are you sure you want to continue connecting?
```

A host hitelességének ellenőrzése után:

```text
yes
```

Sikeres autentikáció esetén GitHub jelzi, hogy sikeresen autentikáltunk.

---

# 34. SSH remote URL

HTTPS remote:

```text
https://github.com/example-user/devops-lab.git
```

SSH remote:

```text
git@github.com:example-user/devops-lab.git
```

---

# 35. Remote átállítása HTTPS-ről SSH-ra

Aktuális remote:

```bash
git remote -v
```

Példa:

```text
origin https://github.com/example-user/devops-lab.git
```

Átállítás:

```bash
git remote set-url origin git@github.com:example-user/devops-lab.git
```

Ellenőrzés:

```bash
git remote -v
```

Eredmény:

```text
origin git@github.com:example-user/devops-lab.git
```

---

# 36. Fiktív teljes SSH példa Linuxon

Felhasználó:

```text
example-user
```

Repository:

```text
devops-lab
```

## 1. Meglévő kulcsok ellenőrzése

```bash
ls -al ~/.ssh
```

---

## 2. Kulcs generálása

```bash
ssh-keygen -t ed25519 -C "example@example.com"
```

Alapértelmezett hely elfogadása:

```text
ENTER
```

Passphrase:

```text
erős jelszó
```

---

## 3. ssh-agent indítása

```bash
eval "$(ssh-agent -s)"
```

---

## 4. Private key hozzáadása

```bash
ssh-add ~/.ssh/id_ed25519
```

---

## 5. Public key megtekintése

```bash
cat ~/.ssh/id_ed25519.pub
```

---

## 6. Public key GitHubra

```text
GitHub
→ Settings
→ SSH and GPG keys
→ New SSH key
```

Beillesztjük a public keyt.

---

## 7. Kapcsolat tesztelése

```bash
ssh -T git@github.com
```

---

## 8. Remote beállítása

```bash
git remote set-url origin git@github.com:example-user/devops-lab.git
```

---

## 9. Push

```bash
git push
```

A folyamat:

```text
Git
↓
SSH
↓
private key
↓
GitHub public key
↓
authentication
↓
authorization
↓
push
```

---

# 37. PAT vs SSH

## HTTPS + PAT / GitHub CLI

```text
Git
↓
HTTPS
↓
PAT / credential helper
↓
GitHub
```

Előny:

- egyszerű URL,
- gyakran könnyen működik vállalati hálózatokon,
- GitHub CLI kényelmesen kezelheti.

Hátrány:

- credential kezelés szükséges,
- PAT lejárhat,
- token biztonságára figyelni kell.

---

## SSH

```text
Git
↓
SSH
↓
private/public key
↓
GitHub
```

Előny:

- kényelmes napi Git használatra,
- nincs PAT begépelés,
- kulcspáros hitelesítés,
- nagyon gyakori fejlesztői/DevOps környezetben.

Hátrány:

- első konfiguráció bonyolultabb,
- private key kezelésére nagyon figyelni kell,
- bizonyos hálózatok blokkolhatják az SSH-t.

---

# 38. HTTPS + gh vs SSH mentális modell

```text
HTTPS

git push
↓
credential helper
↓
gh / PAT
↓
GitHub
```

SSH:

```text
git push
↓
SSH client
↓
private key
↓
GitHub public key
↓
GitHub
```

Mindkettő ugyanazt a célt szolgálja:

```text
Bizonyítsuk GitHubnak,
hogy valóban jogosult felhasználók vagyunk.
```

Csak más technológiát használnak.

---

# 39. WSL és Windows különbsége

Windows és WSL két külön környezetként viselkedhet.

Például:

```text
WINDOWS

Git for Windows
Windows Credential Manager
Windows gh
Windows SSH
```

és:

```text
WSL

Linux Git
Linux gh
Linux SSH
Linux config
Linux credential handling
```

Ezért előfordulhat:

```text
Windowsban GitHub login működik

de

WSL-ben külön authentication setup szükséges
```

Ez nem GitHub-hiba.

A két környezet külön Git és authentication konfigurációval rendelkezhet.

---

# 40. Melyiket érdemes használni?

Mindkettőt érdemes érteni.

DevOps szempontból különösen fontos:

```text
HTTPS token authentication
+
SSH key authentication
```

ismerete.

Napi használatra választható például:

```text
HTTPS + gh credential helper
```

vagy:

```text
SSH
```

A választás környezet- és cégfüggő.

---

# 41. Biztonsági szabályok

## PAT

Soha ne:

```text
commitold Gitbe
írd config fájlba nyíltan
oszd meg
küldd el chatben
```

---

## Private SSH key

Soha ne oszd meg:

```text
~/.ssh/id_ed25519
```

Ez titkos.

---

## Public SSH key

Megosztható:

```text
~/.ssh/id_ed25519.pub
```

Ez kerül GitHubra.

---

## Passphrase

Érdemes használni private key védelemhez.

---

# 42. Rövid összefoglalás

```text
GitHub authentication
=
annak bizonyítása,
hogy ki vagyunk
```

Fő módszerek:

```text
HTTPS + PAT
HTTPS + GitHub CLI
SSH key
```

PAT:

```text
titkos token
HTTPS Git műveletekhez
```

Credential helper:

```text
a credential kezelését végzi
```

GitHub CLI:

```text
gh
=
GitHub parancssori kliens
```

SSH:

```text
private key
+
public key
```

Private key:

```text
csak nálunk marad
```

Public key:

```text
GitHubra kerül
```

SSH agent:

```text
a private key használatát kezeli
```

HTTPS remote:

```text
https://github.com/user/repo.git
```

SSH remote:

```text
git@github.com:user/repo.git
```

---

# 43. Teljes mentális modell

```text
                LOCAL COMPUTER
                      │
                      │
            ┌─────────┴─────────┐
            │                   │
          HTTPS                SSH
            │                   │
      credential helper      private key
            │                   │
         PAT / gh             SSH agent
            │                   │
            └─────────┬─────────┘
                      │
                      ▼
                   GitHub
                      │
              Authentication
                      │
                      ▼
               Authorization
                      │
                      ▼
               Remote repository
```