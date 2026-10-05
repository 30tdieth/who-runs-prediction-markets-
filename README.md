[README Prediction market benchmark.md](https://github.com/user-attachments/files/33070428/README.Prediction.market.benchmark.md)
<a id="english"></a>

**English** · [Français](#francais)

# Who runs prediction markets — Prediction markets benchmark

A static, bilingual (English / French) website that maps the prediction market landscape: dedicated platforms, on-chain protocols, crypto exchanges, brokers, sportsbooks, wallets, and infrastructure (oracles, data). Each player is compared on what sets it apart, who built it, its regulatory status, its blockchain, its market model, its volumes and its funding.

- **Live site**: https://30tdieth.github.io/who-runs-prediction-markets-/ (GitHub Pages, `main` branch, root folder). The trailing hyphen is part of the repository name.
- **Direct French version**: append `?lang=fr` to the address.
- **Data snapshot**: collected on **October 4, 2026**: 58 players, 7 categories, 6 statuses.

> This benchmark is informational. It rests on web research as of the date above, some figures are self-reported by the companies concerned, and the sector changes every week. It is not financial or legal advice.

*This README is bilingual: the English version comes first, the French version follows [below](#francais).*

---

## 1. At a glance

| | |
|---|---|
| Main file | `index.html` (about 118 KB), a single file |
| Technology | Vanilla HTML, CSS and JavaScript. No build step, no npm dependency, no framework |
| External resource | Google Fonts only (Geist, Geist Mono, Newsreader), with fallback fonts |
| Hosting | GitHub Pages. The file must be named `index.html` to open at the root address |
| Default language | English. The choice is remembered in `localStorage` (`pm-bench-lang`) and can be forced with `?lang=en` or `?lang=fr` |
| Theme | Light or dark. Follows the system on first load, then the saved choice (`pm-bench-theme`) |

## 2. Features

- Sortable table of 58 players, with an expandable row for detailed context and a link to each player's website.
- Filters: category tabs, status, regulation, accent-insensitive full-text search, and a reset button.
- Sorting by weight, name, status, creator, regulation or blockchain.
- Interactive volume-share bar: hover or focus for a tooltip, click to jump to the player's row.
- Snapshot of the latest known valuations, a "market reading" section, and a methodology section with sources.
- EN / FR switch without reloading, and a light / dark theme switch.
- Keyboard shortcuts: `/` focuses the search box, `Esc` clears it and then leaves it.
- Accessibility: `aria-*` attributes, visible focus, `prefers-reduced-motion` respected, view transitions (`startViewTransition`) only where the browser supports them.
- Responsive layout, with a sticky command bar and a table header that adjusts beneath it.

## 3. Run locally and deploy

```bash
# Option 1: open the file directly in a browser
open index.html

# Option 2: small local server
python3 -m http.server 8000     # then http://localhost:8000
```

**Deployment**: every `push` to `main` redeploys the site through GitHub Pages (Settings → Pages → *Deploy from a branch* → `main` / `/ (root)`). Allow one to two minutes. After a change, a hard refresh (Ctrl+F5) avoids the cache.

## 4. Architecture of `index.html`

The file follows this order. The landmarks are identifiers to search for in the code, not line numbers, which change with every edit.

| Block | Landmark | Role |
|---|---|---|
| Theme script | first `<script>` in `<head>` | Applies `data-theme` before rendering to avoid a flash of the wrong theme |
| CSS | comments `/* ───── … ───── */` | Sections: Tokens, Base, Top bar, Hero, Market share, Snapshot, Market reading, command bar, table, Methodology, Motion, Responsive |
| HTML | `<header class="hero">`, `.band`, `.reading`, `<main id="bench">`, `<footer class="method">` | Empty skeleton, filled in by JavaScript |
| Interface strings | `const I18N` | EN and FR labels (titles, filters, categories, statuses, valuations, sources) |
| Added strings | `Object.assign(I18N.en, …)` and `Object.assign(I18N.fr, …)` | Second set of labels: brand, date, counters, kickers, themes |
| Sources | `const SOURCE_URLS` | Links of the methodology section, **aligned by index** with `I18N.<lang>.sources` |
| Statuses | `const STATUS_META` | CSS class and sort order of each status |
| **Data** | `const D = [ … ]` | One row per player, **written in French** (source of truth) |
| Translations | `const GEN` and `const EN` | `EN` overrides `D` in English, `GEN` translates generic phrases |
| Volume shares | `const SHARE`, `const VALO_SCALE` | Numeric values for the bar and for the valuation scale |
| Rendering | `applyStatic()`, `renderCats()`, `render()`, `detailHTML()`, `toggle()`, `jumpTo()`, `setSort()`, `setLang()` | DOM generation from `D` and the state |
| Events | `/* Events */` section | Listeners for filters, sorting, language and keyboard |

The whole UI state lives in one object: `state = {lang, cat, status, reg, q, sortK, sortDir, open}`.

## 5. Data model

Each player is an object in `D`. **The internal identifier is derived from `n`** (normalized name), so `n` must be unique.

| Field | Content | Note |
|---|---|---|
| `n` | Display name | Join key with `EN` and `SHARE` |
| `c` | Category | `pure`, `onchain`, `crypto`, `broker`, `sport`, `wallet`, `infra` |
| `s` | Status | `live`, `partial`, `pending`, `announced`, `rumor`, `closed` |
| `w` | Weight from 1 to 5 | Analyst judgment, see §8 |
| `cr` | Creator / parent company | |
| `d` | Point of differentiation | One or two sentences |
| `r` | Regulatory status (text) | |
| `rg` | Regulation (filter) | `cftc`, `sec`, `offshore`, `defi`, `na` |
| `ch` | Blockchain / rails | |
| `m` | Market model and resolution | |
| `v` | Volume, open interest, traction | Always state the period and method |
| `f` | Funding and valuation | |
| `x` | Context (expanded panel) | Key dates, partnerships, litigation |
| `sn` | Status note (optional) | Fill in for `announced`, `pending`, `partial`, `rumor`, `closed` |
| `u` | Website URL | Empty string if unknown |

**Statuses**: `live` (in service), `partial` (beta or limited access), `pending` (awaiting approval), `announced` (announced, not yet launched), `rumor` (in development, unofficial), `closed` (shut down or absorbed).

**"Empty" values**: `"n.d."` (EN: `"n/a"`), `"—"` and `""` are displayed in muted gray. Use `n.d.` when no reliable figure was found, and `—` when the field has no meaning for the player.

## 6. Internationalization

- **French is the source of truth**: `D` is written in French.
- **English is an override layer**: `EN["<exact n>"] = { field: "translation", … }`. The key must be **identical** to `n` in French.
- **Fallback chain** for an untranslated field: the `EN` value, then `GEN[French value]` (e.g. `"n.d."` → `"n/a"`), then the French text as is. A missing translation therefore shows up as French text in the English interface, with no JavaScript error.
- If `EN[...]` defines `n`, that is the display name in English (e.g. `"Meta « Arena »"` becomes `Meta “Arena”`).
- Fields that are identical in both languages (proper names, `—`) do not need to be repeated in `EN`.
- **Number formats**: EN `$60.7B`, `~6%`; FR `60,7 Md$`, `6 %` (space before the sign). The `FIG` regular expressions bold figures according to these formats. Another format still works, but without the highlighting.

## 7. Practical guides

### 7.1 Add a player

1. Add the French row to `D`, inside the group of its category (the data stays in French, as the source of truth):

```js
{n:"Exemple",c:"wallet",s:"announced",w:1,cr:"Société X",
 sn:"Lancement annoncé pour le T4 2026",
 d:"Ce qui le distingue, en une ou deux phrases.",
 r:"À préciser",rg:"na",ch:"Solana",m:"Carnet d’ordres",
 v:"n.d.",f:"n.d.",x:"Contexte et dates clés.",u:"https://exemple.com"},
```

2. Add the translation to `EN`, with a key identical to `n`:

```js
"Exemple":{cr:"Company X",sn:"Launch announced for Q4 2026",
 d:"What sets it apart, in one or two sentences.",m:"Order book",x:"Context and key dates."},
```

3. Update the counters maintained by hand (§7.3), then run the checks (§10).

### 7.2 Change a figure

Change the value in `D` **and** in `EN`, respecting each language's format. If the figure appears elsewhere (valuation snapshot, bar legend, "market reading" block), fix it everywhere: those blocks are independent texts.

### 7.3 Items to keep up to date by hand

At each data revision, check:

| Item | Where | Pitfall |
|---|---|---|
| Update date | `I18N.en.updated`, `I18N.fr.updated` | Two languages |
| Collection date | `I18N.en.snap`, `I18N.fr.snap` | "collected on …" text |
| Issue month | `issue`, `h1b`, `title`, in both languages | |
| **Player count in `lede`** | `I18N.en.lede`, `I18N.fr.lede` | **Hard-coded** ("58 players", "58 acteurs"). The counter at the top of the page is computed |
| Volume shares | `SHARE[].p`, `I18N.*.pct`, `I18N.*.novig`, `barLabel`, `caption` | `SHARE[].p` must sum to 100 |
| Valuations | `I18N.*.valos` and `VALO_SCALE` | Same order and same length; `VALO_SCALE` is in billions of dollars and sets the length of the gauges |
| Sources | `I18N.en.sources`, `I18N.fr.sources`, `SOURCE_URLS` | Three arrays aligned by index |
| Market reading | `I18N.*.reading` | Four written takeaways, to reread at each major update |

## 8. Editorial conventions

1. **Never invent.** No reliable source: `n.d.`. Doubt stays visible in the entry rather than being smoothed over.
2. **Attribute self-reported figures** ("per Chainlink", "per Pyth") and state the period. Volumes from different sources are not comparable with each other (notional, taker volume, cumulative), and the page says so.
3. **Date the statuses.** A player that has not launched is `announced`, `pending` or `rumor`, with an `sn` note saying what is confirmed.
4. **Weight (`w`) is a judgment**, not a measurement: volume, distribution and influence combined. Scale: 5 leader, 4 major, 3 significant, 2 challenger, 1 niche or pre-launch. Any change of weight should be justified in the commit message.
5. **Paraphrase sources**, with no long quotations or reproduction of protected text.
6. **Stay descriptive**: no investment recommendation and no judgment on the value of a product.
7. **EN / FR parity**: every content change touches both languages.

## 9. Design

- **Tokens**: CSS variables in `:root` (light) and `:root[data-theme="dark"]`: surfaces, inks (`--ink`, `--ink-2` to `--ink-4`), accents (`--blue`, `--green`, `--amber`), segment colors (`--seg1` to `--seg4`), spacing (`--s1` to `--s9`), radii, easing curve. Do not hard-code colors: go through the tokens.
- **Typography**: Newsreader (headings, serif), Geist (text), Geist Mono (labels and figures).
- **Motion**: reveal on scroll (`data-reveal`), table entrance animations, view transitions. Everything is disabled under `prefers-reduced-motion`.
- **Command bar** (`#cmd`): sticky; its height is measured in JavaScript and injected into `--cmd-h` to position the table header.

## 10. Check before committing

In the browser console (top-level constants are accessible there):

```js
(() => ({
  entries: D.length,
  enOrphans: Object.keys(EN).filter(k => !D.find(r => r.n === k)),             // expected: []
  duplicates: D.map(r => r.n).filter((n, i, a) => a.indexOf(n) !== i),          // expected: []
  sources: [I18N.en.sources.length, I18N.fr.sources.length, SOURCE_URLS.length],// three identical numbers
  shareTotal: SHARE.reduce((a, s) => a + s.p, 0),                               // expected: 100
  valosAligned: I18N.en.valos.length === VALO_SCALE.length
             && I18N.fr.valos.length === VALO_SCALE.length,                     // expected: true
  badEnums: D.filter(r => !I18N.en.cats[r.c] || !STATUS_META[r.s]
             || !(r.w >= 1 && r.w <= 5)).map(r => r.n),                         // expected: []
  missingEN: D.flatMap(r => ["d","x","sn"].filter(k => r[k] && (EN[r.n]||{})[k] === undefined)
             .map(k => r.n + "." + k))                                          // expected: []
}))()
```

Then, by hand:

- [ ] The console reports no error.
- [ ] Switch EN → FR → EN: no French text in the English version, and vice versa.
- [ ] Light and dark themes.
- [ ] 390 px width (mobile): the page does not scroll horizontally, and the table scrolls inside its container.
- [ ] Search for a recently added player, expand its row, click the website link.

## 11. Scope of the benchmark

**Included**: dedicated platforms (regulated or not), on-chain protocols including those announced without a date, "prediction market" segments of other products (for example HIP-4 on Hyperliquid), crypto exchanges, brokers and TradFi exchanges, sportsbooks, DFS and media, wallets and aggregators, infrastructure (oracles, data, terminals).

**Deliberately excluded**: platforms with no real money or purely forecasting-based ones (Manifold, Metaculus…). To be reopened if the scope changes.

**Rule**: a product that has been announced but has not launched appears in the table with a status that says so clearly.

## 12. Known limitations and data to verify

**Data**
- The entries for **SX Bet, Zeitgeist, Drift BET, Webull and Azuro** rest on general knowledge, without recent verification. The Drift BET entry flags this in its `v` field.
- The launches of **Kraken**, **ADI Predictstreet** (US access) and **Wealthsimple** could not be confirmed by a source as of October 4, 2026.
- **Oracles**: Pyth, RedStone and Chainlink's role at Polymarket were checked against primary sources. Other oracles active in the sector (for example API3, Switchboard, Reality.eth-style or Kleros solutions) **were not examined**.
- The volume shares (`SHARE`) cover **4 venues tracked by DeFi Rate** from September 2 to October 1, 2026, not the whole market.
- Several figures are self-reported (volume settled by Chainlink, value secured by Pyth, RedStone's positioning) and are flagged as such in the entries.

**Technical**
- Data and code live in the same file: every update goes through an edit of `index.html`.
- The player count is hard-coded in `lede` (see §7.3).
- Sources are listed at page level, not per entry: you cannot trace where a specific figure comes from without digging.
- The `<head>` contains no `meta description`, no Open Graph tags and no favicon. There is no print stylesheet.

## 13. Development ideas

In suggested order of priority:

1. **Compute the player count in `lede`** from `D.length` (quick fix, removes a source of oversights).
2. **Sources per entry**: a `src` field (list of `{label, url}`) displayed in the expanded panel, with the date of last verification.
3. **Automated verification**: turn the script from §10 into a test run on every `push` (GitHub Action with Node or Playwright).
4. **Sharing metadata**: `meta description`, Open Graph, favicon, a title per language.
5. **Separate data from code** (`data/players.json`) with a small generation script that reinjects the data into `index.html`. A decision to make first: the current site opens without a server and without a build step, and that should be either preserved or consciously given up.
6. **Filters in the URL** (category, status, search) to share a specific view.
7. **Export** of the filtered table as CSV or JSON.
8. **Complete the oracle coverage** (see §12) and verify the flagged entries.
9. **Normalize the data model** into `{fr: {…}, en: {…}}` per player, so that a missing translation becomes impossible rather than silent.
10. **Print stylesheet** and a `LICENSE` file (no license is defined at this stage).

## 14. Rules for AI assistants (Claude Code)

- **Read §5 to §8 before changing data.** The main pitfalls: an `EN` key identical to `n`, three aligned source arrays, counters typed by hand.
- **Keep the site as a single file with no build step**, unless the owner explicitly asks otherwise. Propose an architecture change before making it.
- **Never invent a figure, a date or a status.** Look for a source, otherwise write `n.d.`. Flag any unverified figure in your reply.
- **Maintain EN / FR parity** and each language's number formats.
- **Do not add any external resource** (CDN, third-party script, analytics) without approval: the site only loads Google Fonts.
- **Wrap every `localStorage` access in a `try/catch`**, as the existing code does.
- **Preserve accessibility** (ARIA roles and labels, visible focus, `prefers-reduced-motion`) and use the CSS tokens for colors.
- **Run the script from §10** after every data change and report the result.
- For a periodic update: reread the dates, volume shares, valuations and the "market reading" block (§7.3), in addition to the entries.

## License

Not defined at this stage.

---

<a id="francais"></a>

[English](#english) · **Français**

# Who runs prediction markets — Benchmark des marchés prédictifs

Site statique et bilingue (anglais / français) qui recense les acteurs des marchés prédictifs : plateformes dédiées, protocoles on-chain, exchanges crypto, brokers, sportsbooks, wallets, et briques d'infrastructure (oracles, données). Chaque acteur est comparé sur son point de différenciation, son créateur, son statut réglementaire, sa blockchain, son modèle de marché, ses volumes et ses levées de fonds.

- **Site en ligne** : https://30tdieth.github.io/who-runs-prediction-markets-/ (GitHub Pages, branche `main`, racine). Le tiret final fait partie du nom du dépôt.
- **Version française directe** : ajouter `?lang=fr` à l'adresse.
- **État des données** : relevées le **4 octobre 2026**, soit 58 acteurs, 7 catégories, 6 statuts.

> Ce benchmark est informatif. Il repose sur des recherches web à la date indiquée, certaines données sont auto-déclarées par les sociétés concernées, et le secteur évolue chaque semaine. Ce n'est pas un conseil financier ni juridique.

---

## 1. En bref

| | |
|---|---|
| Fichier principal | `index.html` (environ 118 Ko), un seul fichier |
| Technologies | HTML, CSS et JavaScript « vanilla ». Aucun build, aucune dépendance npm, aucun framework |
| Ressource externe | Google Fonts uniquement (Geist, Geist Mono, Newsreader), avec polices de secours |
| Hébergement | GitHub Pages. Le fichier doit s'appeler `index.html` pour s'ouvrir à l'adresse racine |
| Langue par défaut | Anglais. Choix mémorisé dans `localStorage` (`pm-bench-lang`) et forçable par `?lang=en` ou `?lang=fr` |
| Thème | Clair ou sombre. Suit le système au premier chargement, puis choix mémorisé (`pm-bench-theme`) |

## 2. Fonctionnalités

- Tableau triable de 58 acteurs, avec ligne dépliable pour le contexte détaillé et le lien vers le site de l'acteur.
- Filtres : onglets de catégorie, statut, régulation, recherche plein texte insensible aux accents, bouton de réinitialisation.
- Tri par poids, nom, statut, créateur, régulation ou blockchain.
- Barre de parts de volume interactive : survol ou focus pour une infobulle, clic pour sauter à la ligne de l'acteur.
- Instantané des dernières valorisations connues, section « lecture du marché » et section méthodologie avec sources.
- Bascule EN / FR sans rechargement, bascule de thème clair / sombre.
- Raccourcis clavier : `/` donne le focus à la recherche, `Échap` l'efface puis la quitte.
- Accessibilité : `aria-*`, focus visible, `prefers-reduced-motion` respecté, transitions de vue (`startViewTransition`) uniquement si le navigateur les gère.
- Responsive, avec barre de commandes collante et en-tête de tableau qui s'ajuste dessous.

## 3. Lancer en local et déployer

```bash
# Option 1 : ouvrir directement le fichier dans un navigateur
open index.html

# Option 2 : petit serveur local
python3 -m http.server 8000     # puis http://localhost:8000
```

**Déploiement** : tout `push` sur `main` redéploie le site via GitHub Pages (Settings → Pages → *Deploy from a branch* → `main` / `/ (root)`). Compter une à deux minutes. Après un changement, un rechargement forcé (Ctrl+F5) évite le cache.

## 4. Architecture de `index.html`

Le fichier suit cet ordre. Les repères sont des identifiants à rechercher dans le code, pas des numéros de ligne, qui changent à chaque édition.

| Bloc | Repère | Rôle |
|---|---|---|
| Script de thème | premier `<script>` dans `<head>` | Applique `data-theme` avant le rendu pour éviter l'éclair de mauvais thème |
| CSS | commentaires `/* ───── … ───── */` | Sections : Tokens, Base, Top bar, Hero, Market share, Snapshot, Market reading, command bar, table, Methodology, Motion, Responsive |
| HTML | `<header class="hero">`, `.band`, `.reading`, `<main id="bench">`, `<footer class="method">` | Squelette vide, rempli par le JavaScript |
| Textes d'interface | `const I18N` | Libellés EN et FR (titres, filtres, catégories, statuts, valorisations, sources) |
| Textes ajoutés | `Object.assign(I18N.en, …)` et `Object.assign(I18N.fr, …)` | Second jeu de libellés : marque, date, compteurs, kickers, thèmes |
| Sources | `const SOURCE_URLS` | Liens de la section méthodologie, **alignés par index** sur `I18N.<lang>.sources` |
| Statuts | `const STATUS_META` | Classe CSS et ordre de tri de chaque statut |
| **Données** | `const D = [ … ]` | Une ligne par acteur, **en français** (source de vérité) |
| Traductions | `const GEN` et `const EN` | `EN` surcharge `D` en anglais, `GEN` traduit les formules génériques |
| Parts de volume | `const SHARE`, `const VALO_SCALE` | Valeurs numériques de la barre et de l'échelle des valorisations |
| Rendu | `applyStatic()`, `renderCats()`, `render()`, `detailHTML()`, `toggle()`, `jumpTo()`, `setSort()`, `setLang()` | Génération du DOM à partir de `D` et de l'état |
| Événements | section `/* Events */` | Écouteurs des filtres, du tri, de la langue, du clavier |

L'état de l'interface tient dans un seul objet : `state = {lang, cat, status, reg, q, sortK, sortDir, open}`.

## 5. Modèle de données

Chaque acteur est un objet de `D`. **L'identifiant interne est dérivé de `n`** (nom normalisé), qui doit donc être unique.

| Champ | Contenu | Remarque |
|---|---|---|
| `n` | Nom affiché | Clé de jointure avec `EN` et `SHARE` |
| `c` | Catégorie | `pure`, `onchain`, `crypto`, `broker`, `sport`, `wallet`, `infra` |
| `s` | Statut | `live`, `partial`, `pending`, `announced`, `rumor`, `closed` |
| `w` | Poids de 1 à 5 | Appréciation d'analyste, voir §8 |
| `cr` | Créateur / maison-mère | |
| `d` | Point de différenciation | Une à deux phrases |
| `r` | Statut réglementaire (texte) | |
| `rg` | Régulation (filtre) | `cftc`, `sec`, `offshore`, `defi`, `na` |
| `ch` | Blockchain / rails | |
| `m` | Modèle de marché et résolution | |
| `v` | Volume, open interest, traction | Toujours préciser période et méthode |
| `f` | Levées et valorisation | |
| `x` | Contexte (panneau déplié) | Dates clés, partenariats, litiges |
| `sn` | Note de statut (optionnel) | À remplir pour `announced`, `pending`, `partial`, `rumor`, `closed` |
| `u` | URL du site | Chaîne vide si inconnue |

**Statuts** : `live` (en service), `partial` (bêta ou accès limité), `pending` (en attente d'agrément), `announced` (annoncé, pas encore sorti), `rumor` (en développement, non officiel), `closed` (fermé ou absorbé).

**Valeurs « vides »** : `"n.d."` (EN : `"n/a"`), `"—"` et `""` s'affichent en gris discret. Utiliser `n.d.` quand aucun chiffre fiable n'a été trouvé, `—` quand le champ n'a pas de sens pour l'acteur.

## 6. Internationalisation

- **Le français est la source de vérité** : `D` est écrit en français.
- **L'anglais est une couche de surcharge** : `EN["<n exact>"] = { champ: "traduction", … }`. La clé doit être **identique** à `n` en français.
- **Repli en cascade** pour un champ non traduit : valeur de `EN`, puis `GEN[valeur française]` (ex. `"n.d."` → `"n/a"`), puis le texte français tel quel. Un oubli de traduction se voit donc par du français dans l'interface anglaise, sans erreur JavaScript.
- Si `EN[...]` définit `n`, c'est le nom affiché en anglais (ex. `"Meta « Arena »"` devient `Meta “Arena”`).
- Les champs identiques dans les deux langues (noms propres, `—`) n'ont pas besoin d'être répétés dans `EN`.
- **Formats de nombres** : EN `$60.7B`, `~6%` ; FR `60,7 Md$`, `6 %` (espace avant le signe). Les expressions régulières `FIG` mettent en gras les chiffres selon ces formats. Un autre format fonctionne, mais sans mise en évidence.

## 7. Guides pratiques

### 7.1 Ajouter un acteur

1. Ajouter la fiche française dans `D`, dans le groupe de sa catégorie :

```js
{n:"Exemple",c:"wallet",s:"announced",w:1,cr:"Société X",
 sn:"Lancement annoncé pour le T4 2026",
 d:"Ce qui le distingue, en une ou deux phrases.",
 r:"À préciser",rg:"na",ch:"Solana",m:"Carnet d’ordres",
 v:"n.d.",f:"n.d.",x:"Contexte et dates clés.",u:"https://exemple.com"},
```

2. Ajouter la traduction dans `EN`, avec la clé identique à `n` :

```js
"Exemple":{cr:"Company X",sn:"Launch announced for Q4 2026",
 d:"What sets it apart, in one or two sentences.",m:"Order book",x:"Context and key dates."},
```

3. Mettre à jour les compteurs affichés à la main (§7.3), puis lancer les vérifications (§10).

### 7.2 Modifier un chiffre

Changer la valeur dans `D` **et** dans `EN`, en respectant le format de chaque langue. Si le chiffre apparaît ailleurs (instantané des valorisations, légende de la barre, bloc « lecture du marché »), le corriger partout : ces blocs sont des textes indépendants.

### 7.3 Éléments à tenir à jour manuellement

À chaque révision des données, vérifier :

| Élément | Où | Piège |
|---|---|---|
| Date de mise à jour | `I18N.en.updated`, `I18N.fr.updated` | Deux langues |
| Date de relevé | `I18N.en.snap`, `I18N.fr.snap` | Texte « relevées le … » |
| Mois du numéro | `issue`, `h1b`, `title`, dans les deux langues | |
| **Nombre d'acteurs dans `lede`** | `I18N.en.lede`, `I18N.fr.lede` | **Codé en dur** (« 58 players », « 58 acteurs »). Le compteur du haut de page, lui, est calculé |
| Parts de volume | `SHARE[].p`, `I18N.*.pct`, `I18N.*.novig`, `barLabel`, `caption` | `SHARE[].p` doit totaliser 100 |
| Valorisations | `I18N.*.valos` et `VALO_SCALE` | Même ordre et même longueur ; `VALO_SCALE` en milliards de dollars, c'est lui qui règle la longueur des jauges |
| Sources | `I18N.en.sources`, `I18N.fr.sources`, `SOURCE_URLS` | Trois tableaux alignés par index |
| Lecture du marché | `I18N.*.reading` | Quatre constats rédigés, à relire à chaque mise à jour majeure |

## 8. Conventions éditoriales

1. **Ne jamais inventer.** Pas de source fiable : `n.d.`. Un doute reste visible dans la fiche plutôt que lissé.
2. **Attribuer les chiffres auto-déclarés** (« selon Chainlink », « selon Pyth ») et préciser la période. Les volumes de plusieurs sources ne sont pas comparables entre eux (notionnel, volume taker, cumulé), et la page le dit.
3. **Dater les statuts.** Un acteur non lancé est `announced`, `pending` ou `rumor`, avec une note `sn` qui dit ce qui est confirmé.
4. **Le poids (`w`) est un jugement**, pas une mesure : volume, distribution et influence combinés. Barème : 5 leader, 4 majeur, 3 significatif, 2 challenger, 1 niche ou pré-lancement. Tout changement de poids doit être justifié dans le message de commit.
5. **Reformuler les sources**, sans citations longues ni reproduction de texte protégé.
6. **Rester descriptif** : pas de recommandation d'investissement ni de jugement sur la valeur d'un produit.
7. **Parité EN / FR** : toute modification de contenu touche les deux langues.

## 9. Design

- **Tokens** : variables CSS dans `:root` (clair) et `:root[data-theme="dark"]` : surfaces, encres (`--ink`, `--ink-2` à `--ink-4`), accents (`--blue`, `--green`, `--amber`), couleurs des segments (`--seg1` à `--seg4`), espacements (`--s1` à `--s9`), rayons, courbe d'animation. Ne pas écrire de couleur en dur : passer par les tokens.
- **Typographie** : Newsreader (titres, serif), Geist (texte), Geist Mono (étiquettes et chiffres).
- **Mouvement** : apparition au défilement (`data-reveal`), animations d'entrée du tableau, transitions de vue. Tout est désactivé sous `prefers-reduced-motion`.
- **Barre de commandes** (`#cmd`) : collante ; sa hauteur est mesurée en JavaScript et injectée dans `--cmd-h` pour caler l'en-tête du tableau.

## 10. Vérifier avant de committer

Dans la console du navigateur (les constantes de premier niveau y sont accessibles) :

```js
(() => ({
  entries: D.length,
  enOrphans: Object.keys(EN).filter(k => !D.find(r => r.n === k)),             // attendu : []
  duplicates: D.map(r => r.n).filter((n, i, a) => a.indexOf(n) !== i),          // attendu : []
  sources: [I18N.en.sources.length, I18N.fr.sources.length, SOURCE_URLS.length],// trois nombres identiques
  shareTotal: SHARE.reduce((a, s) => a + s.p, 0),                               // attendu : 100
  valosAligned: I18N.en.valos.length === VALO_SCALE.length
             && I18N.fr.valos.length === VALO_SCALE.length,                     // attendu : true
  badEnums: D.filter(r => !I18N.en.cats[r.c] || !STATUS_META[r.s]
             || !(r.w >= 1 && r.w <= 5)).map(r => r.n),                         // attendu : []
  missingEN: D.flatMap(r => ["d","x","sn"].filter(k => r[k] && (EN[r.n]||{})[k] === undefined)
             .map(k => r.n + "." + k))                                          // attendu : []
}))()
```

Puis, à la main :

- [ ] La console ne signale aucune erreur.
- [ ] Bascule EN → FR → EN : aucun texte français dans la version anglaise, et inversement.
- [ ] Thème clair et sombre.
- [ ] Largeur 390 px (mobile) : pas de défilement horizontal de la page, le tableau défile dans son conteneur.
- [ ] Recherche d'un acteur récemment ajouté, ouverture de sa ligne, clic sur le lien du site.

## 11. Périmètre du benchmark

**Inclus** : plateformes dédiées (régulées ou non), protocoles on-chain y compris ceux annoncés sans date, segments « marchés prédictifs » d'autres produits (par exemple HIP-4 sur Hyperliquid), exchanges crypto, brokers et bourses TradFi, sportsbooks, DFS et médias, wallets et agrégateurs, infrastructure (oracles, données, terminaux).

**Exclu volontairement** : plateformes sans argent réel ou purement de prévision (Manifold, Metaculus…). À rouvrir si le périmètre change.

**Règle** : un produit annoncé mais non sorti figure dans le tableau avec un statut qui le dit clairement.

## 12. Limites connues et données à vérifier

**Données**
- Les fiches **SX Bet, Zeitgeist, Drift BET, Webull et Azuro** reposent sur des connaissances générales, sans vérification récente. La fiche Drift BET le signale dans son champ `v`.
- Le lancement de **Kraken**, d'**ADI Predictstreet** (accès US) et de **Wealthsimple** n'a pas pu être confirmé par une source au 4 octobre 2026.
- **Oracles** : Pyth, RedStone et le rôle de Chainlink chez Polymarket ont été vérifiés auprès de sources primaires. D'autres oracles actifs dans le secteur (par exemple API3, Switchboard, solutions de type Reality.eth ou Kleros) **n'ont pas été examinés**.
- Les parts de volume (`SHARE`) portent sur **4 venues suivies par DeFi Rate** du 2 septembre au 1er octobre 2026, pas sur l'ensemble du marché.
- Plusieurs chiffres sont auto-déclarés (volume réglé par Chainlink, valeur sécurisée par Pyth, positionnement de RedStone) et signalés comme tels dans les fiches.

**Technique**
- Données et code vivent dans le même fichier : toute mise à jour passe par une édition de `index.html`.
- Le nombre d'acteurs est codé en dur dans `lede` (voir §7.3).
- Les sources sont listées au niveau de la page, pas par fiche : on ne peut pas retrouver d'où vient un chiffre précis sans fouiller.
- Le `<head>` ne contient ni `meta description`, ni balises Open Graph, ni favicon. Il n'y a pas de feuille de style d'impression.

## 13. Pistes de développement

Par ordre de priorité suggéré :

1. **Calculer le nombre d'acteurs dans `lede`** à partir de `D.length` (correction rapide, supprime une source d'oubli).
2. **Sources par fiche** : un champ `src` (liste de `{label, url}`) affiché dans le panneau déplié, avec la date de dernière vérification.
3. **Vérification automatisée** : transformer le script du §10 en test lancé à chaque `push` (GitHub Action avec Node ou Playwright).
4. **Métadonnées de partage** : `meta description`, Open Graph, favicon, titre par langue.
5. **Séparer les données du code** (`data/players.json`) avec un petit script de génération qui réinjecte les données dans `index.html`. Décision à prendre avant : le site actuel s'ouvre sans serveur et sans build, et il faut préserver ça ou l'abandonner consciemment.
6. **Filtres dans l'URL** (catégorie, statut, recherche) pour partager une vue précise.
7. **Export** CSV ou JSON du tableau filtré.
8. **Compléter la couverture des oracles** (voir §12) et vérifier les fiches signalées.
9. **Normaliser le modèle de données** en `{fr: {…}, en: {…}}` par acteur, pour qu'un oubli de traduction devienne impossible plutôt que silencieux.
10. **Feuille de style d'impression** et ajout d'un fichier `LICENSE` (la licence n'est pas définie à ce stade).

## 14. Règles pour les assistants IA (Claude Code)

- **Lire les §5 à §8 avant de modifier les données.** Les pièges principaux : clé `EN` identique à `n`, trois tableaux de sources alignés, compteurs saisis à la main.
- **Garder le site en un seul fichier sans build**, sauf demande explicite du propriétaire. Proposer un changement d'architecture avant de le faire.
- **Ne jamais inventer un chiffre, une date ou un statut.** Chercher une source, sinon écrire `n.d.`. Signaler tout chiffre non vérifié dans la réponse.
- **Maintenir la parité EN / FR** et les formats de nombres de chaque langue.
- **Ne pas ajouter de ressource externe** (CDN, script tiers, analytics) sans accord : le site ne charge que Google Fonts.
- **Entourer tout accès à `localStorage` d'un `try/catch`**, comme le code existant.
- **Conserver l'accessibilité** (rôles et libellés ARIA, focus visible, `prefers-reduced-motion`) et passer par les tokens CSS pour les couleurs.
- **Lancer le script du §10** après chaque modification de données et indiquer le résultat.
- Pour une mise à jour périodique : relire les dates, les parts de volume, les valorisations et le bloc « lecture du marché » (§7.3), en plus des fiches.

## Licence

Non définie à ce stade.
