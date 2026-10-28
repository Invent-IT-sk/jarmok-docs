# E-Jarmok – Testovanie

Prehľad automatizovaných testov. Podrobný postup (inštalácia, spustenie, čítanie výsledkov) je vedený priamo v repozitároch, aby bol vždy aktuálny s kódom.

## Prehľad

| Typ testov | Repozitár | Čo overuje | Kedy sa spúšťajú | Dokumentácia |
| --- | --- | --- | --- | --- |
| E2E testy (Playwright) | `jarmok-web` | Celý proces registrácie predajcu v prehliadači: stránka zóny a cena, súhlasy, základné a doplňujúce údaje, prílohy (všetky typy, kontrola uloženia v API), súhrn, platba neskôr | lokálne, pred vydaním zmien vo formulároch registrácie | [`jarmok-web/e2e/README.md`](https://github.com/Invent-IT-sk/jarmok-web/blob/main/e2e/README.md) |
| Podporné skripty pre E2E | `jarmok-auth-user` | Príprava testovacích dát (podujatie „E2E Test Jarmok“, testovací používatelia), prihlasovací token a vyčistenie dát po teste | automaticky pri každom spustení E2E testov | [`jarmok-auth-user/test/e2e/README.md`](https://github.com/Invent-IT-sk/jarmok-auth-user/blob/main/test/e2e/README.md) |
| Unit testy (Jest) | `jarmok-auth-user` | Controllery, DAO vrstva, služby, validácie a router API, bez databázy (mocky) | lokálne, pred každým PR | [`jarmok-auth-user/test/README.md`](https://github.com/Invent-IT-sk/jarmok-auth-user/blob/main/test/README.md) |
| Funkčné testy (Jest) | `jarmok-auth-user/functionalTests` | Volania bežiaceho API cez HTTP (nahrávanie dokumentov, rodné číslo, testovacie dáta finančného reportu) | lokálne, podľa potreby | [`functionalTests/README.md`](https://github.com/Invent-IT-sk/jarmok-auth-user/blob/main/functionalTests/README.md) |

## E2E testy v skratke

- Bežia **iba lokálne**: webová aplikácia (`vite --mode e2e`), API na `localhost:3000` a lokálna PostgreSQL v Dockeri. Všetky požiadavky mimo `localhost` (GoPay, Mapbox, RegisterUZ) sú v teste zablokované, takže test nikdy nezasiahne dev ani produkčné prostredie.
- Skripty v API odmietnu bežať proti nelokálnej databáze alebo pri `ENVIRONMENT=prod`.
- Prihlásenie prebieha tokenom (bez UI loginu); testovací používatelia nemajú heslo.
- Pred aj po teste sa automaticky zmažú dáta, ktoré test vytvoril (registrácie, faktúry, dokumenty, firma, údaje profilu). Testovacie podujatie a používatelia zostávajú.
- Výsledky: výpis v termináli, HTML report (`npx playwright show-report`) a pri chybe screenshot a trace.

## Stav a ďalšie kroky

- Testy zatiaľ **nebežia v CI** (GitHub Actions iba buildujú a nasadzujú); spúšťajú sa lokálne.
- E2E zatiaľ nepokrývajú: výber elektriny, PO vs. FO, ZŤP, zľavy, prílohy zóny remeselníkov, administráciu.
