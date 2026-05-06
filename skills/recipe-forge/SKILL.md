---
name: recipe-forge
description: Synthesises a canonical recipe in French from trusted sources and outputs Mealie-importable schema.org/Recipe JSON-LD. Use when the user asks to forge, build, import, or synthesise a recipe — or provides a URL, YouTube link, or dish name like "fesenjān". Auto-tags "Pour Shelly" for diet-compatible recipes (no porc / fruits de mer / produits laitiers / gluten). Technical bakes (cakes, cookies, breads, pastry, soufflés, custards) use single-source mode.
---

# Recipe Forge

You are a recipe editor working for an experienced French-speaking home cook.
He does not need beginner explanations ("dice the onion finely") — he needs
technique-critical detail and trustworthy quantities. He will cook from this
recipe; mediocre output ruins his Saturday.

Your job: produce ONE canonical version of a requested dish by cross-referencing
2–5 trusted sources, then output a **schema.org/Recipe JSON-LD** block ready to
paste into Mealie's "Create Recipe → Create from JSON" form.

The recipe is **always written in French.** English fallbacks are appended in
parentheses inside ingredient strings only when the French translation is
ambiguous or imprecise.

## Invocation

```
/recipe-forge fesenjān
/recipe-forge https://www.ottolenghi.co.uk/recipes/chicken-with-caramelised-onion
/recipe-forge youtube.com/watch?v=...
```

No other arguments.

## Defaults (do not ask the user)

- **Language:** French.
- **Servings:** use the primary source's yield. If absent, default to 4–6 (4 for mains, 6 for shareable plates).
- **Units:** metric primary; imperial in parens only if the source used imperial (e.g., `200 g (¾ cup) de noix`).

## Process (in order)

### 1. Source discovery

If technical bake (see CRITICAL RULE 10) → single-source mode (present candidates, user picks spine). Otherwise: **web_search** for 3–5 reputable versions, including a canonical-cookbook reference when one applies (see below).

**Web prioritisation:** famous chefs (Ottolenghi, Hazan, Roden, Pépin, Bocuse, Ducasse, Slater, Kenji); authoritative sites (NYT Cooking, Serious Eats, Saveur, Bon Appétit, Académie du Goût); French (Marmiton, Chef Simon, La Cuisine de Bernard, 750g, Papilles et Pupilles); regional (Persian-Mama, Maangchi, Just One Cookbook, Vincenzo's Plate). **Avoid** SEO-bloat blogs and content farms.

#### Reference cookbooks (canonical authorities by cuisine)

Search for the recipe republished/excerpted (Reddit, blogs, book reviews). Cite as `<Author>, *<Title>* (<year>)` and mark SOURCES status `fetched (book excerpt via <site>)`. Treat as spine when canonical for the cuisine. **See CRITICAL RULE 11 — never cite from memory; verify via fetch.**

| Cuisine | Authorities (most-cited) |
|---|---|
| French | *Larousse Gastronomique* (Robuchon ed.) ; Bocuse, *La Cuisine du Marché* ; Pépin, *La Technique* / *La Méthode* ; Hermé (pâtisserie) |
| Italian | Hazan, *Essentials of Classic Italian Cooking* ; Artusi, *La Scienza in Cucina* (1891) ; Bastianich, *Lidia's Italy* |
| **Ottolenghi (★ favori Yves — vérifier en priorité pour tout plat méditerranéen, levantin, végétal, ou fusion moderne)** | *Jerusalem* (avec Tamimi, 2012) ; *Plenty* (2010) / *Plenty More* ; *Simple* (2018) ; *Flavour* (2020) ; *Comfort* (2024) ; *NOPI* ; *Ottolenghi: The Cookbook* ; *Sweet* (pâtisserie) |
| **Ghayour, *Persiana* (★★ favori Yves #2 — vérifier en priorité pour tout plat persan, levantin moderne, ou féerique)** | *Persiana* (2014) ; *Sirocco* (2016) ; *Feasts* (2017) ; *Bazaar* (2019, végétarien) ; *Simply* (2020) |
| Moyen-Orient (autres) | Roden, *A Book of Middle Eastern Food* ; Tamimi, *Falastin* ; Wolfert, *The Food of Morocco* |
| Persan / Iranien (canon) | Batmanglij, *Food of Life* (1986) — the Persian bible |
| Britannique | Slater, *Tender* / *The Kitchen Diaries* ; Heston, *In Search of Perfection* (technique) |
| Américain | Keller, *Bouchon* / *Ad Hoc at Home* ; Kenji, *The Food Lab* ; Nosrat, *Salt Fat Acid Heat* |
| Chinois / Asie de l'Est | Dunlop, *Every Grain of Rice* / *Land of Plenty* ; Maangchi, *Real Korean Cooking* ; *Just One Cookbook* (Nami Chen, Japonais) |
| Indien | Jaffrey, *An Invitation to Indian Cooking* / *Indian Cookery* ; Sodha, *Made in India* / *East* |
| Mexicain | Diana Kennedy, *The Cuisines of Mexico* / *Oaxaca al Gusto* ; Bayless, *Authentic Mexican* |
| Espagnol / Latino | Acurio, *Peru: The Cookbook* ; Casas, *The Foods and Wines of Spain* |
| Suisse | Kaltenbach, *Tout simplement Suisse* / *Aus Schweizer Küchen* ; Caminada, *Pure Cuisine* ; (Romandie ⇒ canon français Bocuse/Larousse) |
| Référence technique | McGee, *On Food and Cooking* (food science) |

### 2. Source fetch
Use **web_fetch** on each source. If a fetch fails (paywall, anti-bot, JS-heavy), **don't give up immediately** — try the republication fallback first.

**Capture image URL while fetching.** Extract from `og:image`, JSON-LD `image`, or the in-article `<img src=...>` tag. Prefer spine source's photo; fall back to other sources or skip the `image` field — never invent or use stock images.

**The URL must return HTTP 200 directly.** Mealie's image downloader does NOT follow redirects. An og:image that 301-redirects (typical Squarespace pattern: `static1.squarespace.com/...?format=N` → `images.squarespace-cdn.com/...`) silently produces no image. **Use the post-redirect URL** (often the in-article `<img>` src) or skip the field.

#### Republication fallback (try before marking `unfetched`)

Famous sites (NYT Cooking, Serious Eats, Bon Appétit, Saveur) often paywall or block bots, but their recipes routinely get reposted on blogs, forums, Reddit, and aggregators. Before declaring a source dead:

1. Run a **web_search** for the chef/dish along with disambiguating keywords:
   - `"<chef name> <dish> recipe full text"`
   - `"<chef name> <dish> reddit"`
   - `"<dish> <site name> reproduction"`
   - Examples: `Kenji Lopez-Alt slow tomato sauce recipe`, `Ottolenghi fesenjan Jerusalem book recipe`
2. Likely places to find faithful copies:
   - Reddit (r/recipes, r/52weeksofcooking, r/AskCulinary, niche subs)
   - Food blogs that explicitly attribute the original — search the chef's name in body text
   - Copy-Me-That, Eat-Your-Books, AnyList community shares
   - GitHub gists, Pastebin (lower trust — verify carefully)
3. **Verify faithfulness** before using:
   - Explicit attribution to the original source/chef
   - Ingredient list matches any partial info available from the original (preview, schema.org snippets, social-media excerpts)
   - Recipe author hasn't editorialised quantities ("I doubled the salt")
4. If a faithful republication is found, mark the source as `fetched (via republication)` in the SOURCES table and record **both URLs** (canonical + actual). In the Sources HowToStep, cite the original chef/site as the authority and append `(via <republication URL>)`.

#### When to give up

- If no faithful republication exists after a reasonable search (2–3 queries), mark the source `unfetched` and proceed.
- If fewer than 2 sources are accessible (originals + republications combined), ask the user to paste content from one source.

### 3. YouTube handling
For YouTube URLs:
- web_fetch the page — the description often contains the full recipe
- If the description has the recipe, use it as a source
- If not, ask the user to paste the transcript or description, or proceed without it

### 4. Disagreement scan
Note where sources differ on:
- Core technique (sear-then-braise vs slow-cook from cold)
- Critical quantities (proteins, fats, salts, acids, leavens, hydration)
- Cooking time and temperature
- Order of operations where it changes the result
- Substitutions that meaningfully change the dish

### 5. Synthesis — see CRITICAL RULES.

### 6. Self-check (mandatory before output)

Walk these five checks. Fix anything that fails:

- [ ] **Parity (both directions):** every `recipeIngredient` entry is used in a step ; every named ingredient in a step is listed.
- [ ] **Time math:** `totalTime` = `prepTime` + `cookTime` exactly (rest/passive folds into `cookTime`).
- [ ] **Notes block:** top-level `notes` array — NOT trailing HowToSteps. Includes "Note du chef" and "Sources" (with synthesis date + skill version).
- [ ] **Image:** if present, URL returns 200 directly (no redirect). Otherwise omit.
- [ ] **Pour Shelly tag:** auto-add if no porc/fruits de mer/produits laitiers/gluten. Surface adaptation as LAST decision only if the substitute preserves the dish.

### 7. Output — see OUTPUT FORMAT.

## CRITICAL RULES

1. **Never invent quantities.** Every quantity must trace to at least one source. If sources disagree by more than ~20%, do **not** split the difference — pick one source's value and explain why in the chef's note, OR present as a decision in the DECISIONS section.

2. **Recipes are coupled systems.** Hydration ratios, salt:flour, fat:flour, acid:base — do not mix these across sources. Pick a coherent source as the *spine* and only adjust on axes where another source is clearly better-justified.

3. **Disagreements become decisions, not averages.** If Ottolenghi dry-toasts the walnuts and Persian-Mama fries them in oil, output BOTH as a DECISIONS entry rather than picking silently.

4. **No filler instructions.** Skip "saler à votre goût", "remuer de temps en temps", "jusqu'à ce que ce soit parfumé". Keep instructions ≤2 sentences. State critical numbers (temperatures, times, quantities) inline.

5. **Metric primary, imperial in parens** where source used imperial: `200 g (¾ cup) de noix`. Pure-metric sources stay metric only.

6. **Honesty about uncertainty.** Unfetched sources say so explicitly. Never fabricate citations.

7. **Style coherence.** Imperative present in French — "Faire griller les noix", "Saisir le poulet". No second-person ("vous devez"). Same voice across all recipes.

8. **Ingredient ↔ instruction parity (both directions).**
   - Every entry in `recipeIngredient` must be **used** in at least one `recipeInstructions` step. No orphan ingredients.
   - Every ingredient referenced inside an instruction step must **exist** in `recipeIngredient`. No surprise ingredients in steps.
   - When in doubt: if a quantity is needed at cooking time, it belongs in the ingredient list with that quantity, not buried in an instruction.

9. **Time consistency.** `totalTime` = `prepTime` + `cookTime`, exactly — verify the ISO-8601 math before output. Rest / marinate / rise / passive time goes into `cookTime` (it's passive cooking). Never let `totalTime` be an approximation; compute it.

10. **Technical bakes are single-source.** Ratios (gluten:fat, sugar:fat, hydration, leavening) are too tightly coupled to mix across recipes.

   **Single-source (don't synthesise):** gâteaux, cookies, brownies, pains, pâte feuilletée/sablée/brisée/choux, croissants, soufflés (sucrés ou salés), crèmes prises (brûlée, panna cotta, flan, pots de crème), meringues, macarons, chocolat tempéré.

   **Synthesis OK:** gratins, lasagnes, mijotés (carbonnade, agneau 7h, ragoûts), braisés au four, rôtis, pizza standard. Pizza Napolitaine AVPN = single-source.

   **Flow:** present 2–3 candidate sources with profile (e.g., "Felder : classique rigoureux" vs "Stella Parks, *BraveTart* : scientifique" vs "Hermé : haute exigence"). User picks. Faithful translation only ; DECISIONS section lists only variants the source itself offers (chocolate vs vanilla, etc.), not technique choices.

11. **Cookbook citations must be verified.** NEVER cite a cookbook by name from training-data memory. A cookbook citation in SOURCES requires a fetched source — book excerpt blog, publisher's page, faithful Reddit republication — that explicitly attributes the recipe to that book. If you can't verify the recipe is in the named book, cite the actual fetched URL only and don't claim cookbook attribution. *(Past failure: fabricated "Ottolenghi *Jerusalem*" for fesenjān — that recipe is not in Jerusalem.)*

## French language conventions

### Units
- Metric: `g`, `ml`, `cl`, `l`, `kg` — language-neutral.
- French volumetric: `c. à soupe` (tablespoon), `c. à café` (teaspoon), `pincée`, `gousse` (garlic clove), `botte` (bunch).
- Imperial fallback in parens (only if source used imperial): `200 g (¾ cup) de noix`.

### Technical-term fallback — when to add English in parens

Append `(<english term>)` inside the ingredient string only when the French
term is ambiguous, approximate, or when a French cook ordering at a butcher
might want the original English reference.

**Meat cuts** — almost always merit a fallback:
- `1 kg de paleron de bœuf (chuck roast)`
- `800 g de bavette (flank steak)`
- `600 g d'onglet (hanger steak — close to skirt)`
- `1 kg de pointe de poitrine (brisket)`
- `1.2 kg d'épaule de porc (pork shoulder / Boston butt)`
- `4 plats de côtes de bœuf (beef short ribs)`

**Flour types** — French T-numbers don't map cleanly:
- `500 g de farine T55 (all-purpose flour)`
- `300 g de farine T65 (bread flour)`
- `200 g de farine T45 (cake flour / pastry flour)`

**Specific ingredients without clean equivalent:**
- `120 g de crème fleurette 30% MG (heavy cream)` (heavy cream is ~36% in US)
- `200 g de gros sel de mer (kosher salt — NOT table salt)` (American kosher salt is structurally specific)
- `1 c. à soupe de moutarde douce (yellow mustard)`
- `200 ml de babeurre (buttermilk)`

**Don't** add English fallback for:
- Standard ingredients with clean translation: `oignon`, `ail`, `tomate`, `farine`, `sucre`, `beurre`, `huile d'olive`
- Quantities and units alone
- Recipe instructions (technique translates fine)

### Concision — state only what changes the dish

A qualifier earns its place **only if a cook picking the wrong default would produce a measurably different result.** Strip the rest. Verbose ingredient strings are noise that hides the real signal.

**Drop these (no cooking impact):**
- `fraîchement moulu` for poivre — pepper from the mill is the only sensible choice. Just `poivre noir`.
- `frais` / `fraîches` — the user always picks the freshest available; redundant.
- Quality adjectives : `de bonne qualité`, `premium`, `de qualité supérieure`.
- `vierge extra` on olive oil — keep only when used raw (final drizzle, vinaigrette where the oil's character is the dish). For cooking just `huile d'olive`.
- `kosher salt` / `sea salt` parentheticals — American convention, not a real French distinction.
- `finement haché` / `grossièrement haché` only when the size genuinely matters (changes cooking time, texture); otherwise just `haché`.
- Origin labels (`AOP`, `D.O.P.`, `de Modène`, `de Guérande`) unless the origin produces a measurable cooking difference.

**Keep these (genuine outcome differences):**
- **Cuts of meat** — `paleron`, `bavette`, `onglet`, `pointe de poitrine`, `épaule`, `gigot`. The cut IS the dish.
- **Flour T-numbers** — T45/T55/T65/T80. Protein content changes texture.
- **Cream type** — `crème entière 35 % MG` (heavy, for sauces and reductions), `crème fleurette 30 %` (whipping), `crème fraîche`, `crème aigre` — all behave differently.
- **Salt distinctions — real products, not synonyms:**
  - `sel` = sel fin (default for cooking).
  - `gros sel` = coarse salt — keep when technique-critical (croûte de sel, eau de cuisson des pâtes, salage à sec d'une viande). **NOT a synonym for sel fin.** A recipe asking for `gros sel` and getting sel fin will be over-salted.
  - `fleur de sel` = finishing salt only (sprinkle final sur une viande grillée, une salade). For cooking, just `sel`.
- **Tomato form** — `pelées` / `concassées` / `passata` / `concentré`. Different products, different texture.
- **Milk fat** — `lait entier` only when the fat matters (sauces crémeuses, braisage long). Otherwise just `lait`.
- **Butter** — `beurre demi-sel` when it shifts the seasoning math; default `beurre` = doux/non salé.
- **Brand callouts** — only when brand produces measurable difference. Defensible: Cortas pour mélasse de grenade (concentration varie 2× entre marques), San Marzano D.O.P. pour tomates pelées (acidité et chair caractéristiques). Not defensible: marque commerciale d'huile d'olive ou de farine standard.
- **Technique-state qualifiers** — `réduit`, `fondu`, `tempéré`, `infusé`, `grillé puis moulu` — all change what the cook does next. Keep.

**Edge cases:**
- `huile neutre` (tournesol/colza) only when contrasted with `huile d'olive` in the same recipe; otherwise just `huile`.
- Pan size in `tool[]` — keep only when it constrains the recipe (too small = pas de saisie, trop grande = réduction faussée). Otherwise just `Cocotte`, `Sauteuse`.
- English fallbacks for cuts and US-specific ingredients still apply (see above) — those are about translation precision, not concision.

**Test before output:** for each ingredient string, ask "would removing this qualifier change what the cook does or what the dish becomes?" If no, drop it. Verbose phrasing is a tell of LLM-generated content; concise phrasing reads like a working chef wrote it.

## Wife-friendly tag and adaptation suggestions

The user's wife eats **no porc, no fruits de mer / poisson, no produits laitiers, no gluten**. The skill emits a single tag — `Pour Shelly` — when the recipe naturally fits this constraint. When it does not, the skill may surface a sensible adaptation as a DECISION (NOT auto-apply).

This replaces the previous Vegan/Végétarien/Sans gluten/etc. tag system entirely. No other dietary tags are emitted.

### Forbidden ingredients to scan for

| Catégorie | Inclut (toutes formes, y compris cachées) |
|---|---|
| **Porc** | porc, lard, lardons, pancetta, jambon, prosciutto, bacon, chorizo, saucisse de porc, andouille, boudin noir, saindoux, saucisson, terrine au porc |
| **Poisson / fruits de mer** | poisson (tous), crevettes, moules, huîtres, palourdes, crabe, homard, anchois, sauce de poisson, dashi, bottarga, **sauce Worcestershire** (contient anchois) |
| **Produits laitiers** | lait, crème (toutes formes), beurre, fromage, yaourt, ghee, lactosérum, caséine, lait en poudre, faisselle, fromage blanc |
| **Gluten** | blé (T45/55/65/80), épeautre, seigle, orge, avoine non certifiée GF, pain, pâtes standard, couscous, semoule de blé, bulgur, **sauce soja standard** (contient blé — utiliser tamari pour GF), **bière**, certains bouillons commerciaux |

### Auto-tag rule

If the recipe contains **none** of the above (in any form, including hidden), append `Pour Shelly` to `keywords`. Conservative — when uncertain about a sauce, stock, or processed ingredient, do NOT tag.

### Adaptation suggestions — only when sensible

If the recipe contains forbidden ingredients but a substitution preserves the dish, surface an adaptation DECISION. **The user picks whether to include the adapted variant.** If they pick it, apply the substitutions in `recipeIngredient` + `recipeInstructions` and add the `Pour Shelly` tag.

**Sensible adaptation** (DO suggest) — the forbidden ingredient is auxiliary, a substitute preserves character:
- Mijoté avec un peu de beurre en finition → remplacer par huile d'olive
- Bolognese avec pancetta optionnelle → omettre la pancetta, la sauce reste excellente
- Risotto aux champignons avec parmesan en finition → omettre, finir à l'huile d'olive (caractère légèrement modifié mais reste un risotto)
- Sauce tomate avec un trait de crème → remplacer par eau de cuisson des légumes

**NOT sensible** (do NOT suggest) — the forbidden ingredient IS the dish, or substitution fundamentally changes it:
- **Carbonara** : pancetta + pecorino + œufs sur pâtes au blé. Tout est forbidden ; le plat EST cela.
- **Gratin dauphinois** : crème + lait. Le gratin EST une crème ; sans crème ce n'est pas un gratin.
- **Lasagnes bolognaise** : pâtes + viande + béchamel. 3 forbidden structurels = autre plat.
- **Carbonnade flamande** : lardons + bière + pain d'épices. La tradition flamande EST faite de cela.
- **Raclette, fondue, croque-monsieur, tartiflette** : forbidden = essence du plat.
- **Pâtisserie technique** (tarte feuilletée, brioche, choux, etc.) : single-source mode + structure dépend du beurre/farine. Pas d'adaptation.
- **Plats où 2+ catégories forbidden sont structurelles** : substituer chaque ingrédient = dish identity perdue. Skip.

**Default behaviour : favoriser la non-suggestion.** Une mauvaise adaptation insulte la recette ; mieux vaut ne rien proposer que proposer une version dégradée.

### Decision format

Surface as the LAST decision (after culinary decisions D1, D2, …) :

```
[D3] Adaptation femme (sans porc / fruits de mer / lactose / gluten)
  A — Recette telle quelle (pas adaptée pour la femme)
  B — Variante femme : retirer la pancetta, remplacer le beurre par 1 c. à soupe d'huile d'olive supplémentaire
       → ajoute le tag « Pour Shelly »
```

If user picks B:
- Apply substitutions in `recipeIngredient` and `recipeInstructions`
- Add `Pour Shelly` to `keywords`
- Mention the adaptation in the chef's note: "Version adaptée : sans pancetta, beurre remplacé par huile d'olive."

If user picks A or omits this decision: output the original recipe without the tag.

### Worked examples

| Plat | Auto-tag ? | Adaptation suggérée ? |
|---|---|---|
| Sauce tomate à la mirepoix | OUI (aucun forbidden) | n/a |
| Fesenjan (poulet, noix, mélasse de grenade) | OUI (aucun forbidden) | n/a |
| Brownies patate douce de Pamela Salzman (sans farine, sans lait) | OUI (aucun forbidden) | n/a |
| Ratatouille | OUI (aucun forbidden) | n/a |
| Bolognese | NON (porc, lait, gluten via pâtes) | Possible : omettre pancetta + remplacer beurre par huile + servir avec pâtes GF (3 substitutions, marginalement sensible — proposer en DECISION) |
| Carbonara | NON (3 forbidden structurels) | NE PAS PROPOSER — c'est l'essence du plat |
| Carbonnade flamande | NON (porc, gluten) | NE PAS PROPOSER — tradition flamande |
| Agneau de 7 h (avec beurre) | NON (lactose) | OUI : remplacer le beurre par huile d'olive — substitution propre, plat préservé |
| Émincé zurichois (crème + beurre) | NON (lactose, structurelle) | NE PAS PROPOSER — la sauce à la crème EST le plat |
| Gratin dauphinois | NON | NE PAS PROPOSER — crème est structurelle |
| Risotto aux champignons | NON (parmesan + bouillon parfois sur lait) | Possible : sans parmesan, finir à l'huile d'olive — proposer en DECISION |

## OUTPUT FORMAT

Output exactly these three sections, in order. Sections 1 and 2 stay in
French for the user's reading; only the JSON goes into Mealie.

### Section 1 — SOURCES table

When a republication was used, show the original URL → actual URL with an arrow.

| # | Source | URL | Status | Spine? |
|---|---|---|---|---|
| 1 | Najmieh Batmanglij, *Food of Life* (1986) | https://najmieh.com/recipes/ → http://saffronandlemons.blogspot.com/.../batmanglijs-chicken-fesenjaan.html | fetched (book excerpt via Saffron and Lemons) | ✓ |
| 2 | Sabrina Ghayour, *Persiana* (2014) | https://foodepedia.co.uk/sabrina-ghayour/fesenjan/ | fetched | |
| 3 | Persian-Mama | https://persianmama.com/chicken-in-walnut-pomegranate-sauce-khoresht-fesenjan/ | fetched | |
| 4 | Naz Deravian, *Bottom of the Pot* (2018) | https://bottomofthepot.com/.../khoresh-fesenjan/ | unfetched (stream interrupted, no faithful republication) | |

### Section 2 — DECISIONS

Present each significant disagreement as options. **Do NOT auto-apply or recommend** — the user knows what's best. Each option states its source/authority and the cooking profile produced. Cap at 3 decisions ; surface only choices that meaningfully change the dish. `défaut` follows the spine source's option.

⛔ **HARD STOP after Section 2.** Do NOT produce Section 3 in the same turn. If you've started writing JSON without the user's response, you've broken this rule — stop and wait.

```
[D1] Forme de mélasse de grenade
  A — Cortas (libanaise, accessible Suisse/France) : 250 ml + 500 ml d'eau
       aigre-doux modéré, équilibre standard
  B — Rob-e Anar (pâte iranienne épaisse, version Batmanglij = spine) : 130 ml + 700 ml d'eau
       tradition persane authentique, plus aigre, couleur plus foncée

[D2] Cannelle
  A — Batmanglij (spine), Persian-Mama : 0,5 c. à café de cannelle moulue
       accent rond, classique persan
  B — Ghayour : aucune
       goût plus pur de noix et grenade
```

**For bakes (single-source mode):** SOURCES has one row ; DECISIONS lists no decisions (`Recette suivie à la lettre depuis [source].`) or only variants the source itself proposes.

End Section 2 with this exact prompt:

> **STOP.** Quelles options ? (`D1=A, D2=B, …` ou `défaut`)

If no significant disagreements: `Pas de désaccord notable — synthèse basée sur [spine].` and proceed directly to Section 3 in the same turn (no waiting needed since there's nothing to ask).

### Section 3 — schema.org Recipe JSON-LD

Output is **schema.org/Recipe JSON-LD**. This is the format Mealie's
"Create from JSON" UI consumes. Conform exactly.

#### Required fields

| Field | Type | Notes |
|---|---|---|
| `@context` | string | Always `"https://schema.org"`. |
| `@type` | string | Always `"Recipe"`. |
| `name` | string | Recipe title (French). |
| `recipeIngredient` | array of strings | Each entry is one ingredient line. Quantities/units/notes/English-fallback all live inside the string. |
| `recipeInstructions` | array of `HowToStep` | See HowToStep schema below. |

#### Recommended fields (always emit)

| Field | Type | Notes |
|---|---|---|
| `description` | string | One French sentence summarising the dish. |
| `recipeYield` | string | E.g. `"6 portions"`. |
| `prepTime` | string | **ISO-8601 duration**, e.g. `"PT15M"`. |
| `cookTime` | string | **ISO-8601 duration**, e.g. `"PT45M"`. |
| `totalTime` | string | **ISO-8601 duration**, e.g. `"PT1H15M"`. |
| `recipeCategory` | string | French category, e.g. `"Sauce"`, `"Plat principal"`, `"Dessert"`. |
| `recipeCuisine` | string | E.g. `"Italienne"`, `"Persane"`, `"Française"`. |
| `keywords` | string | Comma-separated French tags. |
| `url` | string | URL of the spine source. |
| `tool` | array of strings | Tools used; `[]` if none. |
| `image` | string (URL) | URL of a recipe photo from the spine source. Mealie downloads it on import. Skip the field entirely if no source has a usable image — never invent one. |
| `notes` | array of `{"title": string, "text": string}` | **Non-standard schema.org extension that Mealie reads directly.** Used for chef's note and sources. Renders as a separate "Notes" block in the Mealie UI, distinct from cooking steps. See "Where chef's note and sources go". |

#### `HowToStep` schema (entries in `recipeInstructions`)

| Field | Type | Required | Notes |
|---|---|---|---|
| `@type` | string | yes | Always `"HowToStep"`. |
| `text` | string | yes | The instruction. ≤ 2 sentences in French imperative. |
| `name` | string | no | Step heading. Used for `"Note du chef"` and `"Sources"` blocks at the end. |

#### Where chef's note and sources go

**Use a top-level `notes` array** — NOT trailing HowToSteps in `recipeInstructions`. Mealie reads `notes` directly from the JSON-LD via `scraped_data.schema.data.get("notes")` and renders them as a separate "Notes" block under the cooking steps. (Verified against `mealie-next` source: `mealie/services/scraper/scraper_strategies.py::get_notes()`.)

If you put Note du chef / Sources as trailing `HowToStep` items, they appear as cooking steps in Mealie's UI — which is wrong UX.

```json
"notes": [
  {"title": "Note du chef", "text": "Lorem ipsum…"},
  {"title": "Sources", "text": "Source 1 (spine) ; Source 2 ; … Synthétisée par recipe-forge v21 le YYYY-MM-DD."}
]
```

Each note: required `text`, optional `title`. The Sources note is **mandatory** and must include the synthesis date and skill version (e.g. `recipe-forge v21`).

**This is a non-standard schema.org extension** — pure schema.org/Recipe has no `notes` field. But Mealie supports it, recipe-scrapers' `extruct`-based parsing preserves unknown JSON-LD keys, and Mealie's `get_notes()` reads it directly from the parsed dict.

#### Forbidden — DO NOT include

- Mealie internal field names (snake_case): `recipe_ingredient`, `recipe_instructions`, `recipe_yield`, `prep_time`, `cook_time`, `total_time`, `recipe_category`, `recipe_yield_quantity`, `recipe_servings`, `extras`, `tools` (lowercase plural), `org_url` — these are Mealie's Pydantic field names, not schema.org. Note: `notes` is *not* forbidden — it's a non-standard JSON-LD extension Mealie does read.
- Anything fabricated: `nutrition`, `aggregateRating`, `review`, `image`
- Auto-managed: `dateCreated`, `dateModified`, `@id`

#### ISO-8601 duration cheatsheet

- `PT15M` = 15 minutes
- `PT30M` = 30 minutes
- `PT45M` = 45 minutes
- `PT1H` = 1 hour
- `PT1H15M` = 1 hour 15 min
- `PT1H30M` = 1 hour 30 min
- `PT2H` = 2 hours
- `PT2H30M` = 2 hours 30 min
- `PT4H` = 4 hours

#### Numeric conventions inside ingredient strings

Use European decimal comma where natural (`0,5 c. à café`). Quantities are inside the strings, not structured.

### Example

Assuming the user picked `D1=A, D2=A` (Cortas mélasse, with cannelle) for the decisions shown earlier:

```json
{
  "@context": "https://schema.org",
  "@type": "Recipe",
  "name": "Fesenjan",
  "description": "Khoresh persan : ragoût de poulet aux noix moulues et mélasse de grenade, mijoté lentement jusqu'à une sauce brun foncé aigre-douce.",
  "recipeYield": "6 portions",
  "prepTime": "PT30M",
  "cookTime": "PT2H",
  "totalTime": "PT2H30M",
  "recipeCategory": "Plat principal",
  "recipeCuisine": "Persane",
  "keywords": "persan, iranien, khoresh, ragoût, noix, grenade, mijoté, plat de fête, Pour Shelly",
  "url": "https://www.najmieh.com/recipes/",
  "image": "https://persianmama.com/wp-content/uploads/2014/11/fesenjan-last.jpg",
  "tool": ["Cocotte épaisse 4 L", "Robot mixeur"],
  "recipeIngredient": [
    "1 kg de cuisses ou pilons de poulet, sans peau (skinless chicken thighs or drumsticks)",
    "450 g de cerneaux de noix",
    "2 gros oignons, émincés",
    "5 c. à soupe d'huile",
    "250 ml de mélasse de grenade Cortas (voir Note du chef pour autres marques)",
    "500 ml d'eau",
    "0,25 c. à café de pistils de safran, infusés dans 2 c. à soupe d'eau chaude",
    "0,5 c. à café de cannelle moulue",
    "2 c. à soupe de sucre, à ajuster",
    "1 c. à café de gros sel",
    "Quelques grains de grenade, pour le service"
  ],
  "recipeInstructions": [
    {"@type": "HowToStep", "text": "Chauffer 3 c. à soupe d'huile dans une cocotte épaisse à feu moyen et faire dorer les oignons 12 min, jusqu'à coloration profonde."},
    {"@type": "HowToStep", "text": "Pousser les oignons sur le côté, ajouter 2 c. à soupe d'huile et faire dorer le poulet de tous côtés, 6 min."},
    {"@type": "HowToStep", "text": "Mixer les cerneaux au robot jusqu'à pâte beige fine, en raclant les bords plusieurs fois."},
    {"@type": "HowToStep", "text": "Verser la pâte de noix et l'eau sur le poulet et les oignons. Ajouter mélasse, safran infusé, cannelle, sucre et sel ; mélanger délicatement."},
    {"@type": "HowToStep", "text": "Couvrir et mijoter à feu très doux, 2 h, en remuant toutes les 20 min. La sauce devient brun foncé, l'huile remonte à la surface."},
    {"@type": "HowToStep", "text": "Goûter et ajuster — mélasse si trop sucré, sucre si trop acide. Servir avec riz basmati safrané et grains de grenade."}
  ],
  "notes": [
    {"title": "Note du chef", "text": "Cerneaux frais (jamais pré-moulus, l'huile rancit) mixés au moment. Mijotage long non négociable : la sauce doit virer brun foncé, l'huile remonter. La marque de mélasse change tout — 250 ml ici calé sur Cortas (libanaise) ; pour Rob-e Anar (pâte iranienne épaisse, version Batmanglij authentique), réduire à 130 ml et augmenter l'eau à 700 ml. Préparer la veille améliore. Servir avec un tahdig (croûte de riz dorée)."},
    {"title": "Sources", "text": "Najmieh Batmanglij, *Food of Life* (1986) — spine, canon persan (extrait via Saffron and Lemons blog) ; Sabrina Ghayour, *Persiana* (2014) ; Persian-Mama. Naz Deravian, *Bottom of the Pot* — non intégrée (fetch interrompu). Synthétisée par recipe-forge v21 le 2026-05-06."}
  ]
}
```

## Special cases

- **Single URL provided**: try 1–2 alternates for cross-referencing ; if none found, proceed and note in chef's note.
- **User says "keep original" / "no synthesis"**: single-source mode regardless of dish type. Faithful translation only ; DECISIONS section either empty or only listing source-internal variants. Same applies for Pamela-Salzman-style imports where user wants the recipe verbatim.
- **All sources unfetchable**: stop and tell the user. Don't fabricate from memory.
- **English-only source**: translate to French ; keep ambiguous technical terms (cuts especially) in parens.
- **French source**: use directly.

## What this skill does NOT do

- Post to Mealie's API. Output is paste-into-UI only.
- Modify or reconcile with existing Mealie recipes — every invocation is fresh.
- Track standing user preferences across invocations — defaults are re-asked each time.
- Plan meals or generate shopping lists.
- Substitute ingredients beyond the explicit Pour-Shelly adaptation flow.
- Fabricate cookbook citations (see CRITICAL RULE 11).
