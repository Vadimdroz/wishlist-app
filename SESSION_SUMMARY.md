# Wishlist App — Session Summary

## Project Overview
Single-file HTML PWA (`index.html`) — vanilla JS, no build tools, localStorage (`wl_v3` key), dark mode, Satoshi font.

**Working directory:** `/Users/vadim.drozdovski/wishlist-app/`
**Git:** initialized, `main` branch, ~20 commits.

---

## App Structure

### Key Files
- `index.html` — entire app (~1400 lines)
- `logo.svg` — SVG dandelion logo (120×120 viewBox)
- `manifest.json` — PWA manifest
- `icon-32.png`, `icon-180.png`, `icon-192.png`, `icon-512.png` — PNG icons

### Categories
Movies, TV Shows, Books, Trips (double-wide card), + more.

### Tabs (bottom sticky nav)
- **Future** — wishlist items
- **Discovery** — "Coming Soon" placeholder
- **Past** — completed items

---

## APIs Used
| Purpose | API |
|---|---|
| Movies & TV Shows | OMDB (`apikey=919ea22c`), `type=movie` or `type=series` |
| Books | Open Library |
| Trailers | YouTube Data API v3 (`AIzaSyCUhLADIiLB4AN5A3wdwXd8X_D8PnOnNBg`) |

---

## CSS Variables (Dark Mode)
```css
--bg:#0d0d14; --surface:#16161f; --surface2:#1e1e2a;
--border:rgba(255,255,255,.08); --text:#f0f0f5; --muted:#6b6b8a
```

---

## Key Features Implemented This Session

### "Just Did It" Flow (`markDone(id)`)
- Opens bottom sheet: **5-star rating** + **6 aspect pills** + **free-text review**
- Aspect pills: Storyline & writing, Direction, Acting, Thrill & excitement, Authenticity & emotion, Style & cinematography
- Saved to item: `item.rating`, `item.likedAspects[]`, `item.review`

### Add Directly to Past
- When `tab === 'done'` and adding new item, form shows same rating/aspects/review fields (`isDoneContext` flag in `buildForm`)

### Card Layout
- **Movies/TV:** director + year + stars on one non-wrapping row (actors removed)
- **Books:** two inline rows — (genres | Kindle button) and (source tag | JDI button)
- **Trips:** `grid-column:1/-1; aspect-ratio:8/3` — full width, same height as other cards
- Poster: `align-self:stretch` fills full card height

### Bottom Navigation
```html
<nav class="bottom-nav"> <!-- Future | Discovery | Past --> </nav>
```
- `setTab(t)` switches active tab and re-renders
- `goHome()` on logo click: clears `catId/editId/searchQuery`, switches from Discovery → Future

### Dandelion Logo (`logo.svg`)
- Inspired by dandelion photo: golden stem, white seed-head, seeds blowing upper-right
- Used inline in top-left nav and as favicon/PWA icons
- Favicon hierarchy: SVG → 32px PNG → 192px PNG
- Apple touch icon: 180px; PWA icons: 192px + 512px

---

## Important Field Names
| Field | Purpose |
|---|---|
| `f-imdb` | IMDb rating text input (renamed from `f-rating` to avoid conflict) |
| `f-star-rating` | Star picker hidden input (Future→Past form) |
| `f-match` | Match/interest picker (Future items) |
| `jdi-rating` | Star picker in JDI bottom sheet |
| `jdi-review` | Review textarea in JDI sheet |
| `f-aspects-wrap` | Aspect pills container in add form |

---

## Known Fixes Applied
- TV Shows Smart Update: switched from TVmaze → OMDB `type=series` (TVmaze lacked IMDb/Metacritic data)
- Kindle/JDI right-align on all cards (not just first): added `margin-left:auto` to `.item-card-bottom-right`
- Source tag gap removed: moved out of `.item-card-bottom` (which had `margin-top:auto`)
- TV Shows category image: replaced broken Unsplash URL with working one (`photo-1461151304267-38535e780c79`)

---

## Continuing Development
To resume in a new session, open `/Users/vadim.drozdovski/wishlist-app/` and read `index.html`. The app runs by opening `index.html` directly in a browser (no server needed for most features).

For live preview during development, start a local server:
```bash
cd /Users/vadim.drozdovski/wishlist-app && python3 -m http.server 8080
```
