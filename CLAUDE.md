# ansoncht.github.io — Agent Guidelines

This is a personal GitHub Pages site used primarily as a **travel planner** for Anson's parents (from Hong Kong). All visible content must be in **Cantonese (廣東話, Traditional Chinese)**. Japanese is reserved for proper nouns only (hotel names, place names, attraction names as they appear on signs in Japan).

---

## Repository Structure

```
ansoncht.github.io/
├── CLAUDE.md               ← you are here
├── README.md
└── 2026/
    └── Tohoku/
        └── index.html      ← 2026 Tohoku trip planner
```

Each trip lives in `{year}/{TripName}/index.html` — always a single self-contained HTML file (no build step, no framework).

---

## Language Rules

| Content type | Language |
|---|---|
| UI labels, buttons, headers | Cantonese 廣東話 |
| Descriptions, notes | Cantonese 廣東話 |
| Area / region names | Chinese (東京, 秋田縣, 青森市) |
| Hotel names | Japanese (as they appear on the hotel's signage/website) |
| Attraction names | Japanese proper noun + Chinese subtitle line |
| Restaurant names | Japanese proper noun + Chinese subtitle/address |
| Day of week | Chinese (星期一 … 星期日) |
| Dates | Chinese numeric (8月27日) |

**Never use Simplified Chinese.** The audience reads Traditional Chinese only.

---

## Design System

### Colours

```css
--bg:           #f5f0eb   /* warm off-white page background */
--surface:      #ffffff   /* sidebar / cards */
--header:       #1a1a2e   /* dark navy — header & day number circles */
--accent:       #e8364a   /* red — active states, links, toggles */
--accent-hover: #c5203a
--text:         #2c2c2c
--text-muted:   #888888
--text-light:   #a0a0c0   /* header subtitle */

/* Hotel type marker colours */
--tokyo:        #6b21a8   /* purple */
--onsen:        #b45309   /* amber */
--city:         #1a1a2e   /* navy */
--resort:       #065f46   /* green */
--transit:      #9ca3af   /* grey */

/* Restaurant cards */
--restaurant-bg:     #f0faf4
--restaurant-border: #16a34a
--restaurant-text:   #14532d
```

### Typography

```css
font-family: 'PingFang HK', 'Hiragino Sans', 'Noto Sans HK', system-ui, sans-serif;
```

PingFang HK must be first — it is the correct Hong Kong Cantonese font on Apple devices and renders Traditional Chinese at the right weight.

### Spacing

- Sidebar width (desktop): `340px`
- Card inner padding: `13–14px 18px`
- Border radius on cards/badges: `6–8px`
- Section separators: `1px solid #f0ebe4`

---

## Itinerary Data Structure

Each day entry follows this shape. **Always include all fields**, even if empty arrays.

```js
{
  date: "8月27日",          // display date
  dow: "星期四",             // day of week in Chinese
  day: 2,                   // trip day number (1-indexed)
  hotel: "ホテル八重の緑 東京",  // Japanese hotel name
  location: "東京",          // Chinese area name shown under hotel
  lat: 35.6896,             // hotel coordinates for map marker
  lng: 139.6923,
  price: "$181",            // nightly rate (keep original currency)
  url: "https://...",       // hotel booking / info URL
  type: "tokyo",            // marker colour key: tokyo | onsen | city | resort | transit
  carPickup: true,          // optional — shows blue car callout
  carReturn: true,          // optional — shows blue car callout
  restaurants: [            // booked restaurants for this day
    {
      emoji: "🍣",
      name: "満天鮨",               // Japanese restaurant name
      sub: "Manten Sushi · 丸の内ブリックスクエアB1F",  // address/subtitle
      time: "晚上 7:00",
      url: "https://..."
    }
  ],
  recs: [                   // nearby attraction recommendations
    {
      emoji: "⛩️",
      name: "明治神宮",       // Japanese attraction name
      sub: "Meiji Shrine",   // English or Chinese subtitle
      desc: "莊嚴神社，隱於蔥鬱森林之中。旁邊係代々木公園，非常適合清晨散步"
      // desc is always Cantonese, conversational tone
    }
  ]
}
```

### Adding a restaurant

Find the matching day in `itinerary[]` and push to its `restaurants` array. Always include: `emoji`, `name` (Japanese), `sub` (address or subtitle), `time`, `url`.

### Adding a recommendation

Push to the day's `recs` array. `name` = Japanese proper noun, `sub` = short Chinese/English label (shown in red), `desc` = Cantonese description in a conversational tone (like a friend recommending it).

---

## Map

- Library: **Leaflet 1.9.4** via CDN (unpkg)
- Tiles: OpenStreetMap (`https://{s}.tile.openstreetmap.org/`)
- Route: dashed red polyline connecting all hotel coords in day order
- Markers: circular `L.divIcon` with day number, coloured by `type`
- Default center: `[39.5, 140.8]` zoom `7` (shows all of Tohoku + Tokyo)
- When a day is selected → `map.flyTo(coords, 11–12, { duration: 1 })`
- Always call `map.invalidateSize()` after showing the map panel on mobile

Geocode unknown addresses with Nominatim:
```
https://nominatim.openstreetmap.org/search?q=<address>&format=json&limit=1
```
Include `User-Agent: travel-planner/1.0` header.

---

## Layout

### Desktop (≥ 768px)
- `header` (fixed height) → `main` (flex row, fills remaining height)
- Left: `sidebar` 340px, scrollable
- Right: `#map` fills remaining width

### Mobile (< 768px)
- `header` → `main` (fills remaining, panels are `position: absolute; inset: 0`) → `tab-bar`
- Bottom tab bar with two tabs: 📋 行程 | 🗺️ 地圖
- Active panel has `.tab-active` class (display: block)
- Tapping a day in 行程 automatically switches to 地圖 and flies to the hotel
- Tab switching uses the **View Transitions API** (`document.startViewTransition`) when available, with a plain class-swap fallback
- `body` height: `100dvh` (not `100vh`) — essential for iOS Safari

---

## Commit Convention

Always commit to the `main` branch of `ansoncht.github.io`. Typical flow:

```bash
git -C ~/Documents/ansoncht.github.io add 2026/Tohoku/index.html
git -C ~/Documents/ansoncht.github.io commit -m "short description"
git -C ~/Documents/ansoncht.github.io push
```

After editing, also sync the local working copy at:
`~/Documents/interview/stripe/tohoku-travel-v3.html`

---

## Tone & Content Guidelines

- Descriptions are written **like a friend recommending a place** — warm, direct, Cantonese conversational style (e.g. "一定要去", "非常好食", "一大清早出發")
- Never use formal written Chinese (書面語) — keep it spoken Cantonese
- Rec descriptions should mention **distance / travel time** from the hotel where relevant
- Hotel prices stay in the **original booking currency** (USD $ or JPY ¥)
- Do not invent restaurant or attraction names — only add what the user has actually booked or explicitly requested
