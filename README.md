# DÜNYA KAHRAMANLARI — WORLD HEROES
**Global Heroes & Historical Biography Encyclopedia**

> *“Dünya tarihinin insan hikâyeleri.”*

---

## Architecture Overview

- **Framework**: Next.js (App Router), React 19, TypeScript, Tailwind CSS
- **Database**: PostgreSQL with Drizzle ORM (`src/db/schema.ts`, `src/db/index.ts`)
- **Design System**: Editorial Light (`#FAF9F6` parchment background, `#FFFFFF` cards, `#263548` archival slate headings, `#66717D` secondary copy, `#B28B50` antique gold accents, `Playfair Display` + `Inter` + `JetBrains Mono`)
- **Vector Atlas**: Interactive ISO 3166-1 SVG World Map powered by `d3-geo` (`geoNaturalEarth1`), `topojson-client`, and `world-atlas`

---

## Key Features Implemented

1. **Multilingual Encyclopedia (`/[lang]/...`)**
   - Supports 12 initial languages (`tr`, `en`, `ku`, `ar`, `fa`, `de`, `fr`, `es`, `it`, `ru`, `zh`, `ja`) with automatic RTL support (`ar`, `fa`) and dynamic `hreflang` metadata.
   - Untranslated biographies transparently display a fallback banner stating the verified source language instead of misrepresenting untranslated content.
2. **19-Section Dynamic Homepage**
   - Hero Editorial Showcase, Featured Heroes, World Countries, Explore by Continent, Most Influential Figures, Independence Struggles, Women Heroes, Commanders & State Founders, Today in History, Chronological Flow, Recently Added, Most Read, Photo Gallery, and About Encyclopedia.
   - Every section can be enabled, disabled, reordered, and renamed from `/admin`.
3. **Interactive World Map & ISO 3166-1 Countries System**
   - Explicitly separates **Sovereign States** (`sovereign_state`) from **Dependent Territories / Special Status Regions** (`dependent_territory`).
   - Dedicated Country Dossier pages (`/[lang]/country/[slug]`) with founding history, periods, wars, categorized heroes, photos, and chronological timelines.
4. **Comprehensive Biography Pages (`/[lang]/hero/[slug]`)**
   - Sticky Table of Contents (`İçindekiler`), reading time, share action, Wikidata QID reference, family/childhood/education/youth/life story, political & military career, wars, struggles, awards, balanced historical controversies, quotes, works, books, monuments, verified Wikimedia Commons media attributions, bibliography, related figures, Schema.org `Person` + `Article` JSON-LD, and moderated reader notes.
5. **Advanced Search & BCE–Present Timeline**
   - AJAX autocomplete & full search (`GET /api/search`) with Turkish diacritics folding, Levenshtein typo tolerance, and filters across countries, continents, categories, wars, eras (BCE/CE), years, sorting, and pagination.
   - Chronological Timeline (`/[lang]/timeline`) linking wars, events, state foundings, and hero lifespans.
6. **Automated Data Collection Engine (`src/lib/wikimedia.ts`)**
   - Live **Wikidata API** (`wbgetentities` & `wbsearchentities`), **Wikidata SPARQL** country discovery queries, **Wikipedia REST API** summaries, and **Wikimedia Commons** `extmetadata` license & attribution verification.
   - Duplicate `wikidata_qid` prevention, country citizenship (`P27`) verification, quality score gate, and license verification (images without verified open licenses are excluded from auto-publishing).
7. **26-Module Admin Panel (`/admin`)**
   - Protected by salted `scrypt` password hashing, HMAC-SHA256 HTTP-only sessions, CSRF tokens, brute-force account lockout, and activity audit logs.
   - Default demo administrator: `admin@dunyakahramanlari.org` / `Tarih2026!`.
8. **System Installer (`/install`) & SEO (`sitemap.xml`, `robots.txt`)**
   - Installation diagnostics and lock state at `/install`.

## Cultural heritage and personal story collection

- `/[lang]/culture` presents a curated introduction to cultural places in the current database, with links to the relevant country, source, and photographer. The selection is editorial, not a claim to represent every culture or country.
- `/[lang]/collection` shows published biographies the visitor bookmarks. The list of saved slugs stays in that browser's local storage; the displayed biographies are fetched from the database through `/api/collection`.
- Cultural photographs in `public/images/heritage/` are sourced from Pexels. Credits and links appear in the interface and are recorded in `src/lib/heritage.ts`. The hero's family-album photo is credited to Thiago José Amaral; the photograph of flowers and old photographs is credited to suju-foto.
- Seeded Wikimedia Commons portrait and historical-object credits, including license versions and source pages, are recorded in `src/lib/portrait-attribution.ts`. The Saladin image depicts a historical coin, not a photographic portrait.

## World data: every country and a compiled set of notable people

Two generated snapshots in `data/` are loaded into the database on first start by `src/lib/world-data.ts` (idempotent; existing countries and people are never overwritten).

### `data/world-countries.json` — 250 ISO 3166-1 entries

- **195 sovereign states**: the 193 UN member states plus the two UN observer states (Vatican City, Palestine).
- **51 dependent territories** and **4 entries with a special status** (Kosovo, Taiwan, Western Sahara, Antarctica), listed separately. The grouping follows UN membership as a criterion and is not a political judgement.
- Built by `scripts/build-world-countries.mjs` from the `world-countries` package (mledoze/countries, **ODbL**), Wikidata identifiers and Turkish labels (**CC0**), and short Wikipedia introductions (**CC BY-SA 4.0**, attributed on each country page).

### `data/world-figures.json` — people compiled from Wikidata

Built by `scripts/build-world-figures.mjs`. This is a compiled selection, **not a complete list of every notable person**: up to 8 scientists/inventors, 5 writers/thinkers, 3 artists, 3 Nobel Peace laureates and 2 additional women per country, ranked by the number of Wikipedia language editions.

Editorial policy applied by the script:

- A person is linked to a country only through a Wikidata citizenship statement (P27) for that state. Historical states (for example the Ottoman Empire or the Kingdom of England) are deliberately not mapped to modern countries, so many pre-modern figures are not included automatically.
- Only deceased people are imported automatically.
- Scientists, inventors, writers, thinkers, artists and Nobel laureates are **published**.
- Statespeople, commanders, monarchs, activists and religious figures are imported with the status **`needs_review`** (up to 6 per country). Whether someone is a national hero is an editorial judgement; these records wait in **Admin → Biyografi Yönetimi → "İnceleme Bekliyor"** and can be published in bulk.
- People linked to atrocities or violent crime, athletes and screen-only performers are not imported.
- Portraits are used only when Wikimedia Commons reports an open license; otherwise a monogram placeholder is shown.

To rebuild: `npm i --no-save world-countries && node scripts/build-world-countries.mjs && node scripts/build-world-figures.mjs` (network access to Wikidata, Wikipedia and Commons required; responses are cached in `/tmp/wd/cache`). Quotas are set at the top of the figures script.

Portrait URLs point at Wikimedia Commons thumbnails where one exists; files narrower than 250 px are linked as small originals. `node scripts/refresh-thumbnails.mjs` re-resolves original-file URLs to thumbnails in both the snapshot and the database. When a remote image cannot be loaded, the interface shows the person's initials (`PortraitImage`).

## Honours, works, inventions and quotes

`scripts/build-legacy.mjs` writes `data/world-legacy.json`, loaded by `src/lib/legacy-data.ts` into `hero_honors`, `hero_works` and `hero_quotes`:

- **Honours**: Wikidata "award received" (P166) with the year qualifier; medal images come from the award item when Commons reports an open license, otherwise a vector medal is drawn.
- **Works**: items whose author, composer, creator, architect, director or discoverer/inventor is the person, plus the person's "notable work" list. Kinds: invention, discovery, theory, book, poem, writing, composition, artwork, building, film. Slogans, ideologies, titles, characters and anything classed as a crime are excluded. Up to 14 works and 12 honours per person.
- **Quotes**: first-level bullets from the person's Wikiquote page (Turkish first, then English), with reference markup removed; sections about the person or marked disputed are skipped. Hand-written quotes in the editor take precedence.
- Coverage depends on Wikidata: 3,547 of 5,583 people have at least one record. Many writers have only a few catalogued works, and most pre-20th-century honours have no image.

## Biography pages

`/[lang]/hero/[slug]` uses a dedicated reading layout (`.atlas-bio` in `src/app/atlas.css`):

- **Header**: portrait with its Commons credit, breadcrumb, name, epithet, alternative names, lifespan, categories, Wikidata link, share/save actions and five quick figures (honours, works, wars, reads, reading time).
- **Grouped chapters**: origins, career, struggles/achievements and historical legacy, with a labelled "controversial aspects" callout instead of a plain paragraph.
- **Structured legacy**: honours as a medal wall (open-licence award image or a drawn medal), works grouped as inventions, discoveries, theories, books, poems, writings, compositions, artworks, buildings and films; each item links to its Wikidata record.
- **Wars and courage**: each linked war shows years, place and outcome; linked historical events and birth/death form a chronological "Life Chronology".
- **Quotes, gallery, monuments, sources, related figures, moderated reader notes**, a sticky identity card, an ordered table of contents and language links.
- The sidebar and article carry configurable advertisement areas (see below).
- Person, Article and BreadcrumbList JSON-LD, canonical URL and hreflang alternates are emitted.

## Advertisements (admin controlled)

Ad slots live in the `ad_slots` table and are managed in **Admin → 27. Reklam Yönetimi**:

- Seven seeded placements: home (below the hero, between discovery blocks), biography (intro, sidebar, after sources), directories (books, honours, countries), and a footer banner on every page.
- Each slot has its own on/off switch, device targeting, format, publisher id, AdSense unit id, optional custom HTML and display order. A global `ca-pub-` publisher id can be saved once and inherited.
- Nothing is rendered for visitors until a slot is enabled **and** a valid unit or creative exists; enabled-but-empty slots show a small labelled notice so editors can see the position. Every area is labelled "Reklam" and is styled as a quiet frame to avoid interfering with reading.
- AdSense `adsbygoogle` script is loaded only when a unit is actually rendered. No secret keys are stored; AdSense needs only the public publisher and slot ids.

## Translation infrastructure

- **UI strings**: English is the canonical key set (`src/lib/ui-translations.ts`, derived from `UiDictionary`); Turkish and English ship in code. Overrides for every other language are stored in the `translations` table with namespace `ui` and are merged into the dictionary used by the navigation and footer. New admin module **28. Arayüz Çevirileri** lists every key per language, allows editing one by one, shows coverage per language, configures a provider and runs bulk translation.
- **Providers**: none (manual), self-hosted or public **LibreTranslate** (no key required) and **DeepL** (server-side`DEEPL_API_KEY` or a key saved in settings). The adapter lives in `translateTexts()`; adding another provider is one branch.
- **Content**: the same module can machine-translate the next unpublished-language biographies (four per request) from their English or Turkish source. Results are stored as `translationStatus = 'auto'` and remain editor-reviewable; the Wikipedia link and source language are preserved. Existing biography and country translations are still edited in the separate Translation Center.
- **Honest limits**: word-for-word translation of every biography into all twelve languages requires a running provider and time — roughly 3,600 published biographies per language in batches of four. Without a provider, missing translations fall back to English and the interface states the source language. Selecting another language never presents untranslated text as translated.
- Every menu item, page, category, country, event, war, period, media item, source, homepage section, theme value, SEO record, user, editor, comment, cron log, backup, ad slot and UI string is reachable from the admin panel; the panel has 28 modules.

## Atlas-style visual system (desktop + mobile)

The illustrated atlas UI is defined in `src/app/atlas.css`; existing biography and database behavior remains in place.

- **Desktop:** dark navy icon-and-label navigation; wide image-led world/history vitrine; cream quick-discovery tiles; country flags on a map panel; four editorial feature cards; period timeline with a sourced quotation. Database-backed sections remain editable and toggleable from Admin → Sayfa & Bölüm Yönetimi.
- **Mobile:** compact centered brand, separate hamburger/search controls, tailored hero backdrop, four-column quick tiles, a compact flag strip, two-column discovery cards, and a fixed five-item bottom navigation. The menu includes all other real destinations, languages, and a working appearance toggle.
- **Imagery:** `scripts/build-home-collage.py` composes **decorative** desktop and mobile backgrounds in `public/images/redesign/` from NASA's public-domain Earth photograph (`https://commons.wikimedia.org/wiki/File:The_Blue_Marble_(remastered).jpg`), Roman Colosseum photography by Görkem Özdemir (`https://www.pexels.com/photo/the-colosseum-rome-italy-15562413/`), Giza photography by Diego F. Parra (`https://www.pexels.com/photo/silhouette-of-pyramids-during-sunset-15126861/`), and night-sky photography by Koma Tang (`https://www.pexels.com/photo/stars-in-the-sky-during-night-time-11858855/`). Verified Wikimedia Commons portraits are separate clickable elements, not baked into the background. The visual is explicitly a modern editorial collage, not an archival photograph. The rendered `.webp` files are included, so Pillow is **not** a runtime dependency; only regenerating the images with `python3 scripts/build-home-collage.py` requires Pillow.
- **Feature photography:** book and quill by Atlantic Ambience (`https://www.pexels.com/photo/old-book-and-quill-pen-on-wooden-table-12955808/`); Parthenon by Uiliam Nörnberg (`https://www.pexels.com/photo/the-sun-is-setting-over-the-parthenon-28300999/`). The historical period thumbnails illustrate present-day sites and do not claim to be photographs from the eras shown.
- **New working destinations:** `/[lang]/honors` lists sourced honours and medals with filters, search, pagination and biography links; `/[lang]/books` similarly lists books, poems and writings; `/[lang]/category/sairler` lists poets tagged from occupation data. The world map, 195 sovereign countries, search, biography pages, and admin modules remain connected to PostgreSQL.

## Homepage composition

`getHomepageData` returns a pool of candidates for each block rather than a final list, and `src/app/[lang]/page.tsx` places every person once:

- Blocks with a narrow subject (independence, founders, women, science) choose first; general blocks (featured, notable, recently added, gallery) choose from who is left.
- Within a block, people from different countries and people whose text exists in the page language are preferred.
- Pools marked as daily are reshuffled every day, so the selection changes without editor work.
- "Most read" is a ranked text list without portraits, because it must show the real ranking even when those people already appear above.
- A block with fewer than two people is not rendered. Blocks, their order and their titles remain editable in Admin → Sayfa & Bölüm Yönetimi (`collections_index` and `daily_quote` were added).

Inner pages follow the same rule for images: a biography's gallery lists only images other than its portrait, and a country page does not repeat portraits already shown as cards.
