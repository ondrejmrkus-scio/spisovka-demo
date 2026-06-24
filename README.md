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

Pak otevři `http://localhost:8000`. Kontaktní formulář je vložený Google Formulář (iframe) — data padají do propojeného Google Sheetu, žádný vlastní backend ani konfigurace není potřeba.

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
├── config.js             # cena (PRICE)  ← tady se mění "živé" hodnoty
├── css/styles.css        # vygenerovaný Tailwind build (commitnuto)
├── src/input.css         # zdroj stylů + design tokeny + @font-face
├── js/
│   └── main.js           # promítnutí ceny z configu
├── assets/
│   ├── fonts/            # Montserrat (self-hostováno, woff2)
│   └── img/              # favicon.svg, og-image.svg
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

## Sběr leadů

Kontaktní formulář je **vložený Google Formulář** (iframe v sekci kontaktu, `#lead-iframe` v `index.html`). Odpovědi padají rovnou do Google Sheetu propojeného s formulářem. Web nemá vlastní backend ani endpoint — výměna formuláře = výměna URL iframu.

---

## GDPR

Souhlas se zpracováním osobních údajů je součástí vloženého Google Formuláře. Znění zásad zpracování je na stránce `zasady-zpracovani.html`.

## Analytika

Místo pro měřicí kód je připravené v `<head>` v `index.html` (zakomentovaný blok pro GTM / GA4 / Plausible). Konverzi (odeslání formuláře) eviduje samotný Google Formulář; cross-origin iframe se z webu měřit nedá. **Nevkládej reálné ID, dokud není rozhodnut nástroj.**

---

## ✅ TODO před spuštěním (placeholdery k doplnění)

| # | Co | Kde |
|---|----|-----|
| 1 | **Logo Scio** — placeholder wordmark zatím ponecháváme (OK, rozhodnuto). Případná výměna za oficiální SVG později | `index.html` (nav + patička), `assets/img/favicon.svg`, `assets/img/og-image.svg` |
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
