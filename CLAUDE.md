# Wishlist App — CLAUDE.md

## Project Overview
Single-file HTML PWA. Everything lives in `index.html` (~2400 lines). No build tools, no framework, no dependencies — vanilla JS, CSS, and localStorage. Open directly in a browser.

**Repo:** `vadimdroz/wishlist-app` (GitHub)
**Local path (user's Mac):** `/Users/vadim.drozdovski/Claude/Projects/wishlist-app/`
**Branch:** `main`
**Storage key:** `localStorage('wl_v3')`

---

## Running the App
```bash
open index.html          # opens in default browser
# or serve locally:
python3 -m http.server 8080
```
After any push, user must run `git pull origin main` then hard-refresh browser (**Cmd+Shift+R**). GitHub Pages is the recommended way to avoid this (Settings → Pages → main / root).

---

## Architecture

### Key Constants (top of `<script>`)
| Constant | Value |
|---|---|
| `OMDB_KEY_DEFAULT` | `919ea22c` |
| `YT_KEY` | YouTube Data API v3 key |
| `ROUTES_KEY_DEFAULT` | Google Routes API key |
| `CLAUDE_KEY_DEFAULT` | Anthropic API key (stored/used for trip info fetch) |
| `CATS` | Array of category objects (see below) |
| `BOOK_GENRES` | Sci-fi, Health, Business, Investing, History, Biography, Philosophy, Geo-Politics, Self Improvement |

### Categories (`CATS` array)
| id | Label | Subs |
|---|---|---|
| `movies` | Movies 🎬 | — |
| `tv-shows` | TV Shows 📺 | — |
| `experiences` | Experiences 🎭 | — |
| `books` | Books 📚 | — |
| `stuff` | Stuff 🛍️ | Vadim, Ira, Kids, Other |
| `restaurants` | Restaurants 🍽️ | — |
| `local-trips` | Local Trips 🚗 | Day Trips, Hotels & Overnight |
| `intl-trips` | International Trips ✈️ | — |

### State Variables
```js
let items = [];      // all wishlist items (loaded from localStorage)
let tab = 'future';  // 'future' | 'done'
let catId = null;    // currently open category
let editId = null;   // item being edited
let searchQuery = '';
```

### CSS Variables (dark mode)
```css
--bg:#0d0d14; --surface:#16161f; --surface2:#1e1e2a;
--border:rgba(255,255,255,.08); --text:#f0f0f5; --muted:#6b6b8a
```

---

## Key Functions

### Navigation & Rendering
- `setTab(t)` — switch Future/Past tab
- `goHome()` — logo click: clear catId/editId/search, return to grid
- `openCat(id)` — open a category list view
- `renderContent()` — main render dispatcher
- `renderGrid()` — category grid on home screen
- `renderList()` — item list for a category
- `buildItemCard(item, c)` — builds a single card DOM element
- `openDetail(id)` — opens item detail bottom sheet

### Forms
- `openForm(id)` — opens add/edit form (id=null for new)
- `buildForm(body, c, item)` — renders form fields based on category
- `saveItem()` — collects form values and saves to items[]
- `appendFormRow(body, label, id, type, value)` — helper to add a form field
- `buildPosterSection(posterUrl)` — file upload + URL paste for poster image (NOT shown for `stuff`)
- `setFormPoster(url)` — sets poster preview in form
- `buildSourceField(body, c, sourceVal)` — source of recommendation field (dropdown for movies/TV/books, text for others)

### Item Lifecycle
- `save()` — writes items[] to localStorage
- `markDone(id)` — opens JDI (Just Did It) bottom sheet: 5-star rating + aspect pills + review
- `deleteItem()` — deletes current item

### Smart Fetch / APIs
| Function | Purpose |
|---|---|
| `smartUpdate()` | Movies/TV → OMDB; Books → Open Library |
| `searchOmdb(key, title, type)` | OMDB search (type: `movie`\|`series`) |
| `fetchOmdbById(key, imdbId)` | OMDB detail fetch |
| `fetchFromMaps()` | Google Places API fetch for restaurants |
| `fetchFromMapsLocalTrip()` | Google Places fetch for local trips |
| `fetchIntlTripInfo()` | Claude API fetch for international trip info |
| `fetchProductInfo(url)` | Stuff: Shopify .json → Microlink → JSONLink |
| `stuffFetchProduct()` | Stuff fetch handler; falls back to Pollinations.ai for image |
| `addResearchOption()` | Adds a product option to the Research comparison list |
| `calcCommute()` | Google Routes API commute time calculation |

### Currency Conversion (Stuff)
- `getIlsRate()` — fetches live USD/ILS rate; tries open.er-api.com → fawazahmed0 → Frankfurter; fallback: `3.0`
- `convertStuffPrice()` — `₪⇄$` button handler; auto-detects direction from symbol (₪/$) or suffix (NIS/ILS/USD)

### Stuff Mode
- `setStuffMode('to-buy' | 'to-research')` — toggles buy/research panels
- Buy panel: Product URL + Fetch, Price + ₪⇄$ button, Pros, Cons
- Research panel: topic, list of product options to compare, each fetchable via URL

---

## Item Data Model
```js
{
  id, cat, status,          // 'future' | 'done'
  title, year, poster,
  source, doneDate,
  match,                    // 1-5 interest stars (future items)
  rating,                   // 1-5 rating stars (done items)
  review, likedAspects[],

  // movies / tv-shows
  director, actors, language, imdbRating, metacritic,
  synopsis, trailer, forKids, forWife,

  // books
  author, pages, kindleLink, genres[],

  // restaurants
  cuisine, city, country, mapsLink, priceRange, notes,

  // stuff
  mode,           // 'to-buy' | 'to-research'
  productUrl, price, pros, cons,
  options[],      // research mode: [{url, name, image, price, pros, desc}]

  // local-trips
  mapsLink, tripType, commute,

  // intl-trips
  country, city, notes,
}
```

---

## Stuff Category — Full Details
The most recently built category. No poster upload — image comes from:
1. Product URL fetch (Shopify `.json` → Microlink → JSONLink)
2. Pollinations.ai AI-generated image if no image found (`https://image.pollinations.ai/prompt/...`)

Card shows: type badge (🛒 Buy / 🔍 Research), price, pros snippet, 🔗 Link button.

---

## External APIs Used
| API | Purpose | Key |
|---|---|---|
| OMDB | Movies & TV metadata | `919ea22c` (hardcoded default) |
| Open Library | Book metadata | No key |
| YouTube Data v3 | Trailer search | Hardcoded |
| Google Places | Restaurant/trip auto-fill | Stored in `localStorage('places_key')` |
| Google Routes | Commute calculation | Hardcoded default |
| Anthropic Claude | International trip info | Stored in `localStorage('claude_key')` |
| Microlink | Product page scraping | No key (rate-limited) |
| JSONLink | OG tag extraction fallback | No key |
| Pollinations.ai | AI product image generation | No key |
| open.er-api.com | USD/ILS live rate | No key |
| Frankfurter | USD/ILS rate fallback | No key |

---

## Auto-backup
Every 24h the app auto-exports a JSON backup to a dated file (`wishlist-backup-YYYY-MM-DD.json`) and a CSV. Triggered on load via `localStorage('lastBackupDate')`.

---

## Recent Session Changes (for continuity)
- Removed poster upload from Stuff form (image from fetch/AI only)
- Added Shopify `.json` endpoint as primary fetch strategy (no proxy needed)
- Added Pollinations.ai AI image generation fallback
- Added NIS ⇄ USD price conversion with live rate from 3 sources
- Fixed fetch chain: Shopify direct → corsproxy → allorigins → Microlink → JSONLink
