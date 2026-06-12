# Spisovka — landing page

Jednostránkový statický web pro **Spisovku** — elektronickou spisovou službu pro základní a střední školy od Scia. Účel: informovat školy o povinnosti od 1. 1. 2027 a sebrat kontakty zájemců (konverze = odeslaný formulář).

- **Tech:** statické HTML + Tailwind CSS v4 (build) + vanilla JS. Žádný framework, žádný backend.
- **Cílová skupina:** ZŠ a SŠ (vrstva B). Web **nenabízí** řešení vysokým školám / vrstvě A.
- **Hosting:** Vercel nebo Netlify (statický output, nasaditelné bez úprav kódu).

---

## Rychlý start (lokální náhled)

`css/styles.css` je už vygenerované a commitnuté, takže k prohlížení nic stavět nemusíš. Web je statický — stačí ho naservírovat přes HTTP (kvůli fontům a `fetch` neotvírej přes `file://`):

```bash
npx serve .
# nebo
python3 -m http.server 8000
```

Pak otevři `http://localhost:8000`. Formulář v lokálním režimu (bez nastaveného `FORM_ENDPOINT`) odeslání jen **zaloguje do konzole** — projde tak celý flow včetně úspěšné hlášky.

## Úprava stylů (rebuild CSS)

CSS se generuje z `src/input.css`. Když měníš třídy nebo design tokeny:

```bash
npm install          # jednou
npm run build        # jednorázový minifikovaný build → css/styles.css
npm run watch        # průběžný build při vývoji
```

Po úpravách stylů **commitni i `css/styles.css`** — deploy se spoléhá na vygenerovaný soubor (build krok na hostingu není potřeba).

---

## Struktura

```
.
├── index.html            # celá stránka (9 sekcí)
├── config.js             # FORM_ENDPOINT, cena, konverzní událost  ← tady se mění "živé" hodnoty
├── css/styles.css        # vygenerovaný Tailwind build (commitnuto)
├── src/input.css         # zdroj stylů + design tokeny + @font-face
├── js/
│   ├── main.js           # promítnutí ceny z configu
│   └── form.js           # validace, stavy, odeslání (pluggable)
├── assets/
│   ├── fonts/            # Montserrat (self-hostováno, woff2)
│   └── img/              # favicon.svg, og-image.svg
├── apps-script/Code.gs   # varianta A: sběr leadů do Google Sheetu
├── netlify.toml          # config pro Netlify
├── vercel.json           # config pro Vercel
└── package.json          # build skripty (Tailwind CLI)
```

---

## Nasazení

### Vercel
1. Naimportuj repozitář na [vercel.com](https://vercel.com) (New Project).
2. Framework Preset: **Other**. Build Command: *(prázdné)*. Output Directory: `.`.
3. Deploy. `vercel.json` nastaví hezké URL a cache fontů.

### Netlify
1. Na [netlify.com](https://netlify.com) → Add new site → import repozitáře.
2. Build command: *(prázdné)*. Publish directory: `.` (řídí `netlify.toml`).
3. Deploy.

Obojí funguje bez build kroku, protože `css/styles.css` je commitnuté. Pak už zbývá jen doplnit TODO (níže) a doménu.

---

## Napojení formuláře (kam se ukládají leady)

**Zatím nerozhodnuto.** Formulář je postavený jako *pluggable*: frontend pošle data `fetch` POSTem (JSON) na `FORM_ENDPOINT` z `config.js`. Změna cíle = změna jedné hodnoty.

Jednotný tvar odesílaných dat:

```json
{
  "timestamp": "2026-06-12T10:00:00.000Z",
  "name": "Jana Nováková",
  "email": "reditelka@zsxy.cz",
  "phone": "",
  "school": "ZŠ XY",
  "schoolType": "ZŠ",
  "role": "ředitel/ka",
  "gdprConsent": true,
  "source": "spisovka-landing"
}
```

**Zvolená varianta: A — Google Sheet.** Níže je i popis B a C pro případ, že by se tým rozhodl jinak.

### Varianta A — Google Sheet (přes Apps Script) ✅ ZVOLENO
Nejjednodušší bez infrastruktury. Kompletní návod je v `apps-script/Code.gs` (vytvoř Sheet → vlož skript → nasaď jako Web App → URL vlož do `config.js` → `FORM_ENDPOINT`). Funguje rovnou s aktuálním frontendem (posílá JSON). Dokud není URL doplněná, běží lokální mock.

### Varianta B — Netlify Forms
Sběr leadů přímo v Netlify, bez vlastního endpointu. Postup je okomentovaný v `netlify.toml`. Stručně:
1. K `<form>` v `index.html` přidej `name="lead" method="POST" data-netlify="true"` a skrytý `<input type="hidden" name="form-name" value="lead" />`.
2. V `js/form.js` přepni funkci `send()` na URL-encoded POST na `/`:
   ```js
   var body = new URLSearchParams(Object.assign({ "form-name": "lead" }, payload));
   return fetch("/", { method: "POST", headers: { "Content-Type": "application/x-www-form-urlencoded" }, body: body.toString() });
   ```
3. Leady najdeš v Netlify → Forms.

### Varianta C — vlastní endpoint / externí form služba
Jakákoliv služba, která přijme JSON POST (vlastní serverless funkce, Formspree, CRM webhook…). Stačí vložit její URL do `FORM_ENDPOINT`. Frontend i datový tvar zůstávají beze změny.

> Dokud je `FORM_ENDPOINT: null`, běží **lokální mock** — odeslání se zaloguje do konzole a zobrazí se poděkování. Ideální pro testování.

---

## GDPR

Pod formulářem je **povinný** checkbox souhlasu se zpracováním osobních údajů — bez něj nejde odeslat. Znění i odkaz jsou zatím placeholder (viz TODO).

## Analytika

Místo pro měřicí kód je připravené v `<head>` v `index.html` (zakomentovaný blok pro GTM / GA4 / Plausible). Konverze „odeslání formuláře" se po úspěšném odeslání pushuje do `window.dataLayer` jako událost `lead_submit` (název v `config.js`). **Nevkládej reálné ID, dokud není rozhodnut nástroj.**

---

## ✅ TODO před spuštěním (placeholdery k doplnění)

| # | Co | Kde |
|---|----|-----|
| 1 | **Logo Scio** — placeholder wordmark zatím ponecháváme (OK, rozhodnuto). Případná výměna za oficiální SVG později | `index.html` (nav + patička), `assets/img/favicon.svg`, `assets/img/og-image.svg` |
| 2 | **FORM_ENDPOINT** — nasadit Apps Script (varianta A, viz `apps-script/Code.gs`) a vložit URL Web Appu | `config.js` |
| 3 | **GDPR** — přesné znění souhlasu + odkaz na zásady zpracování Scio | `index.html` (`data-todo="gdpr-odkaz"`, text u checkboxu) |
| 4 | **Kontaktní e-mail** pro školy (zatím neurčen) | `index.html` (`data-todo="kontaktni-email"` — chybová hláška, patička) |
| 5 | **Analytics ID** — GTM/GA4/Plausible | `index.html` (zakomentovaný blok v `<head>`) |
| 6 | **Doména** — nahradit `TODO-domena.cz` v `canonical`, Open Graph a `robots.txt` | `index.html`, `robots.txt` |
| 7 | **Cena** — potvrdit `10 000 Kč` (placeholder) | `config.js` → `PRICE` |
| 8 | **OG obrázek** — pro max. kompatibilitu vyexportovat `og-image.svg` do PNG (1200×630) a nastavit absolutní URL | `assets/img/`, `index.html` |

Všechny TODO v kódu jsou označené komentářem `TODO:` nebo atributem `data-todo="…"` — snadno dohledatelné: `grep -rn "TODO" .`

---

## Poznámky k obsahu

- Veškerý text pochází ze schváleného copy (`../podklady/copy.md`). Neměnit bez domluvy.
- Termín **1. 1. 2027** a cena **10 000 Kč/rok/škola** jsou komunikovány jasně a vizuálně zvýrazněné.
- Žádné konkrétní funkce produktu, žádná konkurence jménem, žádné strašení atestací.
