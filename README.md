# Zarthbenn's TCG Vault

A static web archive for out-of-print trading card games, information collected from working on rules videos for my youtube channel. Card data is stored in per-game SQLite databases and served directly in the browser via sql.js.

---

## How it works

The site consists of three components:

- `index.html` — the entire frontend.
- `games.json` — the manifest listing every game in the archive.
- `data/*.db` — one SQLite database per game containing sets and cards.

## games.json

The manifest is a JSON array where each entry represents one game. The site reads this file on load to build the home page.

## Database structure

Each game has its own `.db` file stored in `data/`. The schema is identical across all games.

### sets table

```sql
CREATE TABLE sets (
  id           INTEGER PRIMARY KEY AUTOINCREMENT,
  name         TEXT NOT NULL,
  code         TEXT,
  release_date TEXT,
  notes        TEXT
);
```

| Column | Description |
|---|---|
| `name` | Full display name of the set or expansion. |
| `code` | Short identifier shown as a badge on the set button, e.g. `HOA`. Optional. |
| `release_date` | Release date as a plain text string, e.g. `1997-03-01` or `March 1997`. |
| `notes` | Short description shown on the set button. Optional. |

Sets are displayed in order of `id` (i.e. insertion order), so the order you insert them is the order they appear on the game page.

---

### cards table

```sql
CREATE TABLE cards (
  id               INTEGER PRIMARY KEY AUTOINCREMENT,
  set_id           INTEGER NOT NULL REFERENCES sets(id) ON DELETE CASCADE,
  collector_number TEXT,
  name             TEXT NOT NULL,
  card_type        TEXT,
  rarity           TEXT,
  rules_text       TEXT,
  flavor_text      TEXT,
  artist           TEXT,
  image_url        TEXT DEFAULT '',
  image_checksum   TEXT,
  extra_data       JSON,
  source_notes     TEXT
);
```

| Column | Description |
|---|---|
| `set_id` | Foreign key linking the card to its set. |
| `collector_number` | The card's number within the set. Used for display and sort order. Stored as text to accommodate formats like `001` or `P1`. |
| `name` | Card name. Required. |
| `card_type` | Type line as printed on the card, e.g. `Creature`, `Spell`, |
| `rarity` | Rarity string. The following values have coloured pips in the UI: `Common`, `Uncommon`, `Rare`, `Legendary`, `Mythic`, `Promo`, `Fixed`. Any other value shows a black pip. |
| `rules_text` | The card's rules or ability text. |
| `flavor_text` | Italicised flavour text shown separately in the card detail view. |
| `artist` | Artist credit. Used in the artist filter on the set view. |
| `image_url` | URL pointing to a hosted scan of the card. Cards with an empty `image_url` show a placeholder. |
| `image_checksum` | Optional. An MD5 or SHA256 hash of the image file for link-rot detection. Future proofing as I explore image hosting options. |
| `extra_data` | A JSON blob for game-specific attributes that don't fit the standard schema, e.g. `{"power": 3, "toughness": 2}`. Rendered as labelled chips in the card detail view. |
| `source_notes` | Notes on where this card's data was sourced from. Shown in the card detail view. Not used currently. |

---

## Image hosting

Im currently using google drive to link images through the "thumbnail" method. Currently exploring other options as this is rate limited and not consistent. 
