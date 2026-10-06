# ⚡ Omrežnina+

[🌐 Dostop do aplikacije](https://curious-hamster-3d9d0d.netlify.app/)  
[📘 Dokumentacija (GitBook)](https://omreznina.gitbook.io/omreznina+)  
[💻 GitHub repozitorij](https://github.com/adam8kac/Omreznina)  
[🎫 Upravljanje nalog (YouTrack)](https://omreznina.youtrack.cloud/issues)  
[🎥 Predstavitveni video spletne strani](https://youtu.be/4VQbA-ZNZQ0?si=rnjP1ZeGg3gaYNvq)


## 📖 O projektu

**Omrežnina+** je sodobna spletna aplikacija, ki uporabnikom omogoča napreden nadzor nad porabo električne energije in optimizacijo stroškov. Glavni cilji aplikacije so:

- omogočiti celovit pregled nad mesečno in dnevno porabo,
- prikazovati stroške glede na časovne bloke in dogovorjeno moč,
- zaznavati in prikazovati prekoračitve ter izračunavati posledične stroške,
- omogočiti simulacijo porabe moči,
- zagotoviti večjo varnost z uporabo MFA (dvofaktorske avtentikacije),
- avtomatsko analizirati naložene podatke iz sistema mojelektro.si,
- prikazati podatke tudi za sončne elektrarne.

Aplikacija je primerna vsakogar, ki si želi boljše in lepše razlage svoje električne porabe.


## 🏗️ Arhitektura sistema

Aplikacija je zgrajena po sodobni modularni arhitekturi z jasno ločitvijo med frontendom, backendom in podatkovno plastjo. Uporabljene so naslednje tehnologije:

- **Frontend**: React + Vite + TailwindCSS
- **Backend**: Spring Boot (Java), z uporabo Firebase SDK in AES enkripcije
- **Podatkovna baza**: Firebase Firestore (NoSQL)
- **Avtentikacija**: Firebase Auth z razširitvijo za TOTP MFA (Google Authenticator)
- **CI/CD**: GitHub Actions za avtomatsko gradnjo in testiranje, Netlify za frontend in Render za backend
- **Zunanje integracije**: OpenWeather, OpenAI za dinamične povzetke

![Arhitekturni diagram](images/architecture.png)


## 🗃️ Shema Firestore baze

Podatki so organizirani po uporabnikih (`uid`), vsak uporabnik ima naslednje ključne kolekcije in dokumente:

```
users (uid)
├── dogovorjena-moc
│   ├── 1: number
│   ├── 2: number
│   └── ...
├── et
│   └── price: number
├── mfa
│   ├── enabled: boolean
│   ├── secretHash: string
│   └── uid: string
├── optimum
│   └── 2024
│       ├── 1
│       │   └── data[0]
│       │       ├── agreedPower: number
│       │       ├── agreed_power_price: number
│       │       ├── block: number
│       │       ├── blockPrice: number
│       │       ├── maxPowerRecieved: number
│       │       ├── overrun_delta: number
│       │       ├── penalty_price: number
│       │       ├── timestamp: string
│       │       ├── optimal agreed power: number
│       │       └── total price: number
│       └── ... (bloki 2–4 enako strukturirani)
├── poraba
│   └── 2025-04
│       └── 2025-04-01
│           ├── cena energije et: number
│           ├── delta oddana delovna energija et: number
│           ├── delta prejeta delovna energija et: number
│           ├── merilno mesto: string
│           ├── oddana delovna energija et: number
│           ├── poraba et: number
│           ├── prejeta delovna energija et: number
│           ├── tarifa za et: number
│           └── vrsta stanja: string
├── prekoracitve
│   └── 2024
│       └── 01
│           ├── 1
│           │   └── data[0]
│           │       ├── agreedPower: number
│           │       ├── agreed_power_price: number
│           │       ├── block: number
│           │       ├── blockPrice: number
│           │       ├── delta power: number
│           │       ├── maxPowerRecieved: number
│           │       ├── penalty_price: number
│           │       ├── timestamp: string
│           │       └── total price: number
│           └── ... (bloki 2–4 enako strukturirani)
├── racuni
│   └── 2017
│       └── 04
│           ├── energyCost: number
│           ├── month: string
│           ├── networkCost: number
│           ├── note: string
│           ├── penalties: number
│           ├── surcharges: number
│           ├── totalAmount: number
│           ├── uploadTime: timestamp
│           └── vat: number
└── toplotna-crpalka
    ├── power: number
    └── turn on temperature: number
```


## 🎯 UML UseCase diagram

Diagram zajema naslednje funkcionalnosti:

- možnost vklopa dvofaktorskega uverjanja,
- izbris računa,
- upravljanje uporabniškega računa,
- vnos porabe/15 minutne porabe ročno iz datotek,
- ročni vnos računa,
- ogled grafov in analiz,
- simulacija porabe,
- predikcija porabe naslednjega meseca,
- vnos toplotne črpalke,
- izračun optimuma dogovorjene moči in primerjavo optimum/dejansko
- Interakcija s chatbotom

![UML UseCase diagram](images/usecase.jpg)


## 🚀 Deployment (CI/CD)

Projekt **Omrežnina+** uporablja sodoben CI/CD proces, ki temelji na GitHub Actions in integracijah z zunanjimi platformami za neprekinjeno integracijo in hitro objavo sprememb.

### ✅ Avtomatizacije preko GitHub Actions

Ob vsakem `push` ali `pull request` se izvede:

- gradnja in testiranje (cypress) **frontend** aplikacije (React + Vite),
- gradnja in testiranje **backend** aplikacije (Spring Boot),
- preverjanje pokritosti testov in kakovosti kode z **SonarCloud**,
- avtomatski **deploy frontenda na Netlify**,
- avtomatski **deploy backenda na Render**,
- opcijsko tudi **Docker build & deploy** (lokalno ali CI/CD scenarij).


### 🌐 Hosting & Deploy platforme

#### **Netlify (Frontend)**  
Netlify gostuje React aplikacijo z uporabo **Jamstack** arhitekture in omogoča:
- avtomatski deploy ob vsakem `push` na `main` branch,
- podporo za `vite.config.js` in optimizacijo statičnih vsebin,
- okoljske spremenljivke, preusmeritve in zaščitene poti,
- vgrajen CDN, ki omogoča hitro nalaganje iz katere koli lokacije.

🔗 [Netlify dokumentacija](https://docs.netlify.com/)

#### **Render (Backend)**  
Render skrbi za gostovanje Spring Boot backenda in nudi:
- avtomatski deploy iz GitHub repozitorija,
- podporo za `Dockerfile` ali gradnjo iz Maven projekta,
- HTTPS certifikat, okoljske spremenljivke in health check,
- dostop do Firestore baze v realnem času.

🔗 [Render dokumentacija](https://render.com/docs)


Backend teče na Renderju in komunicira z bazo Firestore preko Firebase Admin SDK.


## ⚙️ Lokalna vzpostavitev

### 1. Kloniranje repozitorija

```bash
git clone https://github.com/adam8kac/Omreznina.git
cd Omreznina
```

### 2. Zagon frontenda

```bash
cd frontend
npm install
npm run dev
```
Frontend bo dostopen na http://localhost:5173

### 3. Zagon backenda
```bash
cd backend
mvn spring-boot:run
```

Pred tem konfiguriraj datoteko application.properties s Firebase Admin JSON datoteko (GOOGLE_APPLICATION_CREDENTIALS).
Za lokalni razvoj se običajno nastavi okolje s potjo do .json datoteke:

```bash
export GOOGLE_APPLICATION_CREDENTIALS=path/to/your/firebase-adminsdk.json
```

Backend bo dostopen na:
http://localhost:8080

### 4. Alternativno (Docker)
```bash
cd Omreznina/build_images
```

```bash
touch firebase.json
```
Sem kopirate service account key ki ga najdeš na firestore.

In na koncu:
```bash
./build_docker_image/build.sh
```

## 🧪 Testiranje

Cypress: e2e testi (prijava, MFA, grafi, simulacije)

JUnit & Mockito: unit testi v Spring Boot

SonarCloud: analiza kode in pokritost s testi

CI/CD: vsi testi tečejo v GitHub Actions


## 🔐 MFA zaščita
Uporabniki lahko omogočijo MFA (TOTP) z uporabo Google Authenticator. Skrivnost se AES-enkriptira in shrani v Firestore. MFA se preverja ob prijavi, če je aktivirana.

Firebase TOTP MFA

Google Identity MFA


## 🌐 Uporabljena orodja in dokumentacija

| Orodje                   | Dokumentacija                                                                                                                                       |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Firebase Firestore       | [firebase.google.com/docs/firestore](https://firebase.google.com/docs/firestore)                                                                    |
| Firebase TOTP MFA        | [firebase.google.com/docs/auth/web/totp-mfa](https://firebase.google.com/docs/auth/web/totp-mfa)                                                    |
| Google Identity MFA      | [cloud.google.com/identity-platform/docs/web/mfa](https://cloud.google.com/identity-platform/docs/web/mfa)                                          |
| OpenWeather API          | [openweathermap.org/api/one-call-3](https://openweathermap.org/api/one-call-3)                                                                      |
| Netlify                  | [docs.netlify.com](https://docs.netlify.com/)                                                                                                       |
| Render                   | [render.com/docs](https://render.com/docs)                                                                                                          |
| OpenAI                   | [platform.openai.com/docs/overview](https://platform.openai.com/docs/overview)                                                                      |
| Docker                   | [docs.docker.com/get-started/introduction/build-and-push-first-image](https://docs.docker.com/get-started/introduction/build-and-push-first-image/) |
| Docker + GitHub Actions  | [docs.docker.com/build/ci/github-actions](https://docs.docker.com/build/ci/github-actions/)                                                         |
| Cypress                  | [docs.cypress.io](https://docs.cypress.io/app/get-started/why-cypress)                                                                              |
| SonarCloud               | [docs.sonarsource.com](http://docs.sonarsource.com/sonarqube-cloud/)                                                                                |
| Spring Boot              | [docs.spring.io/spring-boot/documentation](https://docs.spring.io/spring-boot/documentation.html)                                                   |
| Vite                     | [vite.dev/guide](https://vite.dev/guide/)                                                                                                           |
| Novi časovni bloki (URO) | [uro.si/prenova-omrežnine](https://www.uro.si/prenova-omreznine/novi-časovni-bloki)                                                                 |
| GEN-I Ceniki             | [gen-i.si/ceniki](https://gen-i.si/dom/elektricna-energija/ceniki-in-akcije/?utm_source=chatgpt.com)                                                |


## 📘 Celotna dokumentacija
Vse podrobnosti, opisi, tehnični diagrami in navodila so dostopni v GitBook dokumentaciji:

📖 https://omreznina.gitbook.io/omreznina+
