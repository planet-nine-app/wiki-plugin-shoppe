# wiki-plugin-shoppe — Developer Documentation

Multi-tenant digital goods shoppe for Federated Wiki, powered by Sanora.

## Architecture

Follows the Service-Bundling Plugin Pattern. Each tenant (seller/creator) gets their own Sanora user account, identified by a UUID and an 8-emoji emojicode (same format as BDO: 3 base + 5 unique from the EMOJI_PALETTE).

### Tenant Identity

Each tenant has:
- **UUID** — their Sanora user UUID
- **Emojicode** — 8-emoji human-readable identifier (e.g. `🛍️🎨🎁🌟💎🐉📚🔥`)

The first 3 emoji are fixed per wiki instance (`SHOPPE_BASE_EMOJI`, default `🛍️🎨🎁`). The last 5 are unique per tenant.

### Tenant Lifecycle

1. Wiki owner registers a tenant: `POST /plugin/shoppe/register { name }`
   - Plugin creates a Sanora user for the tenant
   - Generates emojicode, stores `{ uuid, emojicode, name, keys }` in `.shoppe-tenants.json`
   - Returns `{ uuid, emojicode }` — tenant puts these in their `manifest.json`
2. Tenant builds their archive (see format below)
3. Tenant drags archive onto the wiki plugin widget
4. Plugin verifies `uuid + emojicode`, processes all goods into Sanora
5. Shoppe accessible at `/plugin/shoppe/:uuid` or `/plugin/shoppe/:emojicode`

## Archive Format

```
my-shoppe.zip
  manifest.json           ← required: { uuid, emojicode, name }
  books/
    My Novel/             ← subfolder per book
      my-novel.epub
      cover.jpg
      info.json           ← { "title": "…", "description": "…", "price": 0 }
    Technical Guide/
      guide.pdf
      cover.jpg
      info.json
  music/
    My Album/             ← album = subfolder
      cover.jpg
      01-track.mp3
      02-track.mp3
    Standalone Track.mp3  ← standalone track = file directly in music/
  posts/
    01-Hello World/       ← numeric prefix determines table of contents order
      post.md             ← the post content (required)
      cover.jpg           ← optional cover image
      screenshot.png      ← any assets referenced in the markdown
      info.json           ← optional: { "title": "…", "description": "…" }
    02-My Series/         ← subdirectories = multi-part series
      cover.jpg           ← optional series cover
      intro.md            ← optional series intro
      info.json           ← optional: { "title": "…", "description": "…" }
      01-Part One/
        post.md
        diagram.png
      02-Part Two/
        post.md
  albums/
    Vacation 2025/        ← photo album = subfolder
      photo1.jpg
      photo2.jpg
  products/
    T-Shirt/              ← product = subfolder with cover + info.json
      cover.jpg
      info.json
```

### manifest.json

```json
{
  "uuid": "your-uuid-from-registration",
  "emojicode": "🛍️🎨🎁🌟💎🐉📚🔥",
  "name": "My Shoppe"
}
```

### products/*/info.json

```json
{
  "title": "Planet Nine T-Shirt",
  "description": "Comfortable cotton tee with logo",
  "price": 25,
  "shipping": 5
}
```

## Routes

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `POST` | `/plugin/shoppe/register` | Owner | Register new tenant |
| `GET`  | `/plugin/shoppe/tenants` | Owner | List all tenants |
| `POST` | `/plugin/shoppe/upload` | UUID+emojicode in archive | Upload goods archive |
| `GET`  | `/plugin/shoppe/:id` | Public | Shoppe HTML page |
| `GET`  | `/plugin/shoppe/:id/goods` | Public | Goods JSON |
| `GET`  | `/plugin/shoppe/:id/goods?category=books` | Public | Filtered goods JSON |

`:id` accepts either UUID or emojicode.

## Configuration

```bash
# Base emoji for all tenant emojicodes on this wiki (default: 🛍️🎨🎁)
export SHOPPE_BASE_EMOJI="🏪🎪🎁"

# Sanora port (default: 7243)
export SANORA_PORT=7243
```

## Supported File Types

| Category | Extensions |
|----------|-----------|
| Books    | .epub, .pdf, .mobi, .azw, .azw3 |
| Music    | .mp3, .flac, .m4a, .ogg, .wav |
| Posts    | .md |
| Albums   | .jpg, .jpeg, .png, .gif, .webp, .svg |
| Products | .jpg/.png cover + info.json |

## Storage

Tenant registry: `.shoppe-tenants.json` (gitignored — contains private keys)

Each tenant's goods are stored in Sanora under their own UUID.

## Dependencies

```json
{
  "adm-zip": "^0.5.10",
  "form-data": "^4.0.0",
  "multer": "^1.4.5-lts.1",
  "node-fetch": "^2.6.1",
  "sessionless-node": "^0.9.12"
}
```
