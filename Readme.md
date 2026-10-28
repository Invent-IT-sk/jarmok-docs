# E-Jarmok – Hlavná technická dokumentácia

**Verzia dokumentu:** 1.0 (December 2025)
**Stav:** Living document (pravidelne aktualizovaný)

---

## Obsah
1. [Účel a prehľad systému](#1-účel-a-prehľad-systému)
2. [Architektúra riešenia](#2-architektúra-riešenia)
3. [Frontend aplikácia](#3-frontend-aplikácia)
4. [Mapová funkcionalita](#4-mapová-funkcionalita)
5. [Formuláre a validácia](#5-formuláre-a-validácia)
6. [Backend API](#6-backend-api)
7. [Databáza a dáta](#7-databáza-a-dáta)
8. [Integrácie](#8-integrácie)
9. [Bezpečnosť a ochrana údajov](#9-bezpečnosť-a-ochrana-údajov)
10. [Multijazyčnosť (i18n)](#10-multijazyčnosť-i18n)
11. [Build, nasadenie a CI/CD](#11-build-nasadenie-a-cicd)
12. [Monitoring a analytika](#12-monitoring-a-analytika)
13. [API dokumentácia](#13-api-dokumentácia)
14. [Komponenty a custom hooks](#14-komponenty-a-custom-hooks)
15. [Výkonnostné optimalizácie](#15-výkonnostné-optimalizácie)
16. [Podporované prehliadače](#16-podporované-prehliadače)
17. [Známe problémy a plánované vylepšenia](#17-známe-problémy-a-plánované-vylepšenia)
18. [Kontakt a podpora](#18-kontakt-a-podpora)
19. [Prílohy](#19-prílohy)

---

## 1. Účel a prehľad systému
**E‑Jarmok** je cloudová webová aplikácia, ktorá digitalizuje proces organizácie jarmočných podujatí. Umožňuje registráciu predajcov, správu stánkov, vizualizáciu priestorových dát na mape, spracovanie platieb a automatizované generovanie dokumentov (faktúry, potvrdenia, oznámenia).

**Cieľové skupiny:**
- **Organizátori** (mestá, obce): správa podujatí, mapy a zóny, registrácie, platby, reporty.
- **Predajcovia**: online registrácia, výber miesta na mape, platba, sledovanie stavu.
- **Návštevníci**: interaktívna mapa stánkov, profily predajcov, informácie o programe.

**Hlavné výhody:** úspora času, transparentnosť, flexibilita, moderné responzívne rozhranie.

---

## 2. Architektúra riešenia
Systém beží v **Microsoft Azure** a používa vrstvenú architektúru:

**Vrstvy:**
- **Prezentačná vrstva (Frontend, SPA)** – React, routing, UI komponenty.
- **Biznis logika (Hooks, Context, Services)** – orchestrácia procesov, stav.
- **Dátová/prístupová vrstva (API klient, TanStack Query)** – HTTP, cache, synchronizácia.
- **Externé služby** – REST API, Mapbox, platobná brána, storage, preklady.

**Nasadené komponenty:**
- **Frontend:** Azure Static Web Apps (SPA fallback, CDN caching).
- **Backend API:** Azure Container Apps (auto-scaling, blue‑green deployment).
- **Databáza:** Azure PostgreSQL Flexible Server (RLS, zálohy, extensions).
- **Úložisko súborov:** Azure Blob Storage (SAS access, CDN, lifecycle tiers).
- **Monitorovanie:** Application Insights, Log Analytics.

---

## 3. Frontend aplikácia
**Typ:** PWA (Progressive Web App), SPA

**Technologický stack (aktuálny):**
- **React 19.1.0**
- **TypeScript 5.8.x** (strict mode)
- **Vite 7** (build tool)
- **React Router 7.6** (client‑side routing)
- **Zustand 5** (lightweight global state)
- **TanStack Query 5.84** (server state, caching)
- **React Hook Form 7.65** + **Zod 4.1** (form + schémová validácia)
- **Mapbox GL 3.14** (+ Draw, Search)
- **Recharts 3.2** (grafy)
- **i18next 25** + **react‑i18next 15** (i18n)
- Utility: **Axios 1.11**, **jwt‑decode 4**, **crypto‑js 4.2**, **Moment.js 2.30**, **immutable.js 5.1**, **lodash.debounce 4.0.8**.

**Štruktúra projektu:**
```
src/
├─ assets/          # Statické zdroje (ikony, obrázky)
├─ components/      # Znovupoužiteľné UI komponenty (50+)
├─ constants/       # Aplikačné konštanty
├─ context/         # React Context providers
├─ hooks/           # Custom React hooks (11)
├─ routes/          # Route komponenty & konfigurácia
├─ services/        # API service layer (6 služieb)
├─ types/           # TypeScript typy
├─ utils/           # Utility funkcie & helpers
├─ index.css        # Globálne štýly
└─ main.tsx         # Entry point
```

**Routing:**
- Kategórie: Public (`/`, `/event/:id`), Authenticated (`/user/*`), NotAuthenticated (`/login`), Admin (`/admin/*`).
- Ochrana trás: **SafeRoute** wrapper + role‑based access.
- 404: vlastný **NotFound** komponent.

---

## 4. Mapová funkcionalita
**Komponent:** `MapWidget` – interaktívna mapa pre kreslenie a správu plôch stánkov.

**Funkcie:** kreslenie/úprava polygónov, výpočet plochy (m²), perimetra a najdlhšej strany (Turf.js), drag&drop značiek, vyhľadávanie lokácií, responsive viewport.

**Poznámka k historickým zmenám:** Pôvodne bol v niektorých častiach dokumentácie uvedený **Leaflet**; aktuálne sa používa **Mapbox GL** (jednotný stack pre lepšiu integráciu s Draw/Search a moderné renderovanie).

---

## 5. Formuláre a validácia
- **React Hook Form** – minimálne re‑renders, uncontrolled pattern.
- **Zod** – type‑safe validácia s vlastnými chybovými správami.
- Hlavné formuláre: `ApplicationForm` (registrácia predajcu), `EventForm` (admin), `AreaForm`, `LoginForm` (multi‑step), `UserSettings`.

---

## 6. Backend API
**Runtime:** Node.js 22 + TypeScript 5.9

**Framework:** Express 5 (middleware architektúra)

**Kľúčové súčasti:**
- **Autentifikácia:** JWT (access 15 min, refresh 7 dní), email/SMS verifikácia, Facebook OAuth; príprava na eID (Slovensko.sk, STS/IAM)
- **Validácia:** Zod schémy + automatické OpenAPI
- **ORM / SQL:** Kysely (type‑safe SQL)
- **Logging:** Winston (structured) → Application Insights, Log Analytics
- **Rate limiting:** vlastná implementácia (PostgreSQL)
- **Dokumenty:** DOCX šablóny + Gotenberg (HTML/DOCX → PDF)

**Moduly API:** autentifikácia, používatelia/spoločnosti, eventy, mapy/zóny, registrácie, platby/faktúry, dokumenty/súbory, dashboard/analytika, verejné API, prekladanie, facebook integrácia.

---

## 7. Databáza a dáta
**Azure PostgreSQL Flexible Server** – relačná DB (~25 tabuliek), multi‑tenant s **Row Level Security (RLS)**.

**Extensions:** `pg_cron` (periodické úlohy), `unaccent` (vyhľadávanie bez diakritiky).

**Migrácie:** verzionované v TypeScript; auto job pri deploy (kontrola pending migrácií, transakčný run, rollback pri zlyhaní).

**Formáty dát:**
- API request/response: JSON
- Upload: `multipart/form-data`
- Preklady: JSONB štruktúra (jazykové objekty)

---

## 8. Integrácie
- **Platby:** GoPay (session, redirect, callback, webhooky, status update, e‑ticket)
- **Email:** Azure Communication Services (verifikačné kódy, potvrdenia, notifikácie)
- **SMS:** EuroSMS brána (overenie, upozornenia)
- **Facebook:** publikovanie eventov + OAuth login
- **DeepL:** automatické preklady (SK → EN/HU/PL) s možnosťou manuálnej úpravy
- **Storage:** Azure Blob Storage (obrázky, PDF; SAS, CDN; lifecycle tiers)
- **Mapbox:** GL render, Draw, Search

---

## 9. Bezpečnosť a ochrana údajov
- **Autentifikácia:** email+OTP, telefón+SMS OTP, Facebook OAuth, príprava na eID.
- **Autorizácia:** RBAC (user/admin/superadmin) + RLS izolácia tenantov.
- **API bezpečnosť:** HTTPS, JWT Bearer, CORS whitelist, Helmet (CSP, HSTS, X‑Frame‑Options), vstupná validácia.
- **Ochrana pred útokmi:** rate limiting, parametrizované SQL, XSS prevention.
- **GDPR:** právo na vymazanie, automatické čistenie starých záznamov, dáta iba v EÚ, audit trail.

---

## 10. Multijazyčnosť (i18n)
- **Framework:** i18next + react‑i18next (HTTP backend, lazy loading)
- **Ukladanie:** cookie + localStorage; fallback **slovenčina (sk)**
- **Podporované jazyky:** sk, en, hu, pl
- **Súbory prekladov:** `public/locales/{sk,en,hu,pl}.json`
- **Funkcie:** pluralizácia, interpolácia, context‑based translations, date/time formatting (Moment.js)
- **Detekcia jazyka:** nastavenia prehliadača, predchádzajúca voľba.

---

## 11. Build, nasadenie a CI/CD
- **Build:** Vite (code splitting, asset optimization; SVG sprites; WebP)
- **Skripty:** `npm run dev|build|preview|lint|lint:fix`
- **Env:**
  ```
  VITE_MAPBOX_TOKEN=<mapbox_access_token>
  VITE_API_URL=<backend_api_url>
  ```
- **Platformy:** Azure Static Web Apps (frontend), Azure Container Apps (backend)
- **IaC:** Pulumi (TS) – RG, ACR, ACA Env, ACA apps, PostgreSQL, Storage, Key Vault, App Insights, Log Analytics, App Gateway, Communication Service
- **CI/CD:** GitHub Actions – build → docker → deploy (dev/prod), health checks, manuálne approval do prod, rollback cez Pulumi
- **Testovanie:** E2E testy (Playwright) vo webe, unit a funkčné testy (Jest) v API; prehľad a odkazy na postup v [TESTING.md](TESTING.md)

---

## 12. Monitoring a analytika
- **Application Insights:** request/dependency tracking, exceptions, custom metrics (registrácie, platby), performance profiling, availability tests
- **Log Analytics (KQL):** centralizované logy, trace cez Request ID, User/Tenant ID
- **Dashboard:** finančný prehľad, obsadenosť plôch, heatmapy, export PDF/Excel

---

## 13. API dokumentácia
- **Základ:** REST API s OpenAPI/Swagger
- **Docs:** `https://api.e-jarmok.sk/docs` (Swagger UI)
- **OpenAPI JSON:** `https://api.e-jarmok.sk/api-docs`
- **Hlavné endpointy:** `/api/auth/*`, `/api/events/*`, `/api/stalls/*`, `/api/reservations/*`, `/api/users/*`, `/api/payments/*`, `/api/documents/*`, `/api/translations/*`

---

## 14. Komponenty a custom hooks
**Komponenty (výber):** Layout (`Layout`, `Header`, `Footer`, `BreadcrumbNavigation`), Form (`FormInput`, `CheckBox`, `RadioList`, `Dropdown`, `SegmentedControl`, `FileInput`, `OTPInput`), Data Display (`Table`, `PaginatedContent`, `EventCard`, `StandCard`, `InfoCard`, `Charts`), Feedback (`Alert`, `Notification`, `Modal`, `Loading`, `ProgressBar`, `Timer`), Špecializované (`Map`, `MapWidget`, `MapMarker`, `QrCode`, `ETicket`, `GoPayMethods`, `RichText`).

**Hooks (výber):** `useUser`, `useAlert`, `useLayout`, `usePageTitle`, `useParams`, `useRoute`, `usePayment`, `useQrCode`, `useServiceTypes`, `useLanguageDropdown`.

---

## 15. Výkonnostné optimalizácie
- **Code splitting:** vendor + route‑based + component lazy loading
- **Caching:** TanStack Query; Service Worker (PWA ready); HTTP cache headers
- **Bundle:** tree shaking, minification, gzip/brotli
- **Obrázky:** SVG sprites, WebP, lazy loading

---

## 16. Podporované prehliadače
- Chrome/Edge (posledné 2 verzie), Firefox (posledné 2), Safari (posledné 2), iOS Safari (14+), Chrome Android (posledné 2)

---

## 17. Známe problémy a plánované vylepšenia
- **Patch:** `@mapbox/search-js-react@1.3.0` – custom patch pre kompatibilitu s React 19 (patch-package)
- **Plán:** PWA offline režim, push notifikácie, WCAG 2.1 AA, React Native app, WebSocket realtime, pokročilá analytika.

---

## 18. Kontakt a podpora
- **Repozitár (FE):** `Invent-IT-sk/jarmok-web` (branch: `main`)
- **Podpora:** support@e-jarmok.sk
- **Dokumentácia (verejná):** https://e-jarmok.sk/docu

---

## 19. Prílohy
**A. ID-SK Design System** – `@id-sk/frontend-3.0.0-beta.css`, gov.sk branding, accessibility.

**B. Mapbox konfigurácia** – token, style URL, Search API, Draw config.

**C. Limity uploadov** – max veľkosť (default 10 MB), formáty: PDF/JPG/PNG/DOC/DOCX, storage: Azure Blob Storage.

> **Poznámka:** *Aplikačná príručka* a *Používateľská príručka* sú k tejto hlavnej technickej dokumentácii vedené **ako samostatné prílohy** (nie sú súčasťou tohto súboru).

---

<!-- Archivované zdrojové dokumenty (nepripájať ako prílohy, slúžia len na referenciu pri zlučovaní) -->
<details>
<summary>Zdroj: TECHNICAL_DOCUMENTATION.md (výňatky)</summary>

# E-Jarmok - Technická Dokumentácia Frontend Aplikácie

## 1. Prehľad Aplikácie

**E-Jarmok** je cloudová webová Single Page Application (SPA) postavená na React frameworku, ktorá digitalizuje tradičný proces organizácie jarmočných podujatí. Frontend aplikácia zabezpečuje používateľské rozhranie pre registráciu predajcov, správu stánkov, vizualizáciu priestorových dát na mapách a spracovanie platieb.

**Verzia:** 0.76.0  
**Typ aplikácie:** Progressive Web Application (PWA)  
**Deployment:** Azure Static Web Apps  

---

## 2. Technologický Stack

### 2.1 Core Framework & Runtime

| Technológia | Verzia | Účel |
|------------|--------|------|
| **React** | 19.1.0 | Hlavný UI framework |
| **TypeScript** | 5.8.3 | Typovaný JavaScript |
| **Vite** | 7.0.0 | Build tool & development server |
| **React Router** | 7.6.3 | Client-side routing |

### 2.2 State Management & Data Fetching

| Technológia | Verzia | Účel |
|------------|--------|------|
| **Zustand** | 5.0.6 | Lightweight state management |
| **TanStack Query** | 5.84.1 | Server state management, caching, synchronizácia |
| **React Hook Form** | 7.65.0 | Správa formulárov |
| **Zod** | 4.1.12 | Schema validation |

### 2.3 Mapové & Priestorové Dáta

| Technológia | Verzia | Účel |
|------------|--------|------|
| **Mapbox GL** | 3.14.0 | Renderovanie interaktívnych máp |
| **Mapbox GL Draw** | 1.5.0 | Kreslenie & editácia polygónov na mape |
| **Mapbox Search JS React** | 1.3.0 | Vyhľadávanie adries a lokácií |
| **Turf.js** | 7.2.0 | Priestorové výpočty (plocha, perimeter, geometria) |

### 2.4 UI & Vizualizácia

| Technológia | Verzia | Účel |
|------------|--------|------|
| **Recharts** | 3.2.1 | Grafy a štatistiky |
| **Draft.js** | 0.11.7 | Rich text editor |
| **QRCode.react** | 4.2.0 | Generovanie QR kódov |

### 2.5 Internacionalizácia

| Technológia | Verzia | Účel |
|------------|--------|------|
| **i18next** | 25.3.1 | i18n framework |
| **react-i18next** | 15.6.0 | React integrácia |
| **i18next-...
</details>

<details>
<summary>Zdroj: DOKUMENTACIA.md (výňatky)</summary>

# E-Jarmok - Dokumentácia systému

## Obsah
1. [O systéme](#1-o-systéme)
2. [Architektúra riešenia](#2-architektúra-riešenia)
3. [Funkcionalita](#3-funkcionalita)
4. [Technológie](#4-technológie)
5. [Bezpečnosť a ochrana dát](#5-bezpečnosť-a-ochrana-dát)
6. [Integrácie](#6-integrácie)
7. [Multijazyčnosť](#7-multijazyčnosť)
8. [Dostupnosť systému](#8-dostupnosť-systému)

---

## 1. O systéme

### Účel
**E-Jarmok** je cloudová webová aplikácia, ktorá digitalizuje tradičný proces organizácie jarmočných podujatí. Systém zjednodušuje registráciu predajcov, správu stánkov a spracovanie platieb, čím šetrí čas organizátorom aj účastníkom.

### Pre koho je systém určený

**Organizátori jarmoku** (mestá, obce)
- Správa viacerých jarmočných podujatí
- Prehľad registrácií a platieb
- Generovanie dokumentov a reportov
- Komunikácia s predajcami

**Predajcovia (jarmočníci)**
- Jednoduchá online registrácia
- Výber lokality stánku na mape
- Online platba
- Sledovanie stavu prihlášky

**Návštevníci jarmoku**
- Informácie o ponúkanom sortimente
- Interaktívna mapa stánkov
- Profily predajcov s kontaktmi

### Hlavné výhody
- **Úspora času** - Automatizácia administratívnych úkonov
- **Transparentnosť** - Všetky informácie dostupné online
- **Flexibilita** - Podpora viacerých jazykov a miest
- **Modernosť** - Responzívne rozhranie fungujúce na všetkých zariadeniach

---

## 2. Architektúra riešenia

### Cloudové riešenie
Systém beží v cloude Microsoft Azure, čo zabezpečuje:
- **Vysokú dostupnosť** - 99,9% uptime s automatickým failover mechanizmom
- **Škálovateľnosť** - Automatické prispôsobenie výkonu podľa záťaže pomocou Azure Container Apps
- **Bezpečnosť** - Dáta uložené v európskych dátových centrách s šifrovaním v pokoji aj pri prenose
- **Zálohovanie** - Automatické zálohy databázy každých 24 hodín s 30-dňovou retenciou

### Technická architektúra

**Vrstvová architektúra**
Systém je postavený na štvorvrstvovej architektúre, ktorá zabezpečuje separáciu zodpovedností a jednoduch...
</details>
