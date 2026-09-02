# Headless Multi-Tenant CMS

A headless, multi-tenant content management system built with **Laravel** and **Filament**, serving structured content to any frontend framework (React, Next.js, Vue, Nuxt, Angular) over a clean JSON API.

Think of it as a self-hosted, developer-first **WordPress + ACF** — familiar content concepts, but fully decoupled and API-driven.

---

## Concept

Content is modeled as **fixed types** (Page, Post, …) with two layers:

- **Fixed columns** for the queryable, routable, SEO-relevant data — `slug`, `title`, `status`, `published_at`, and meta fields.
- **A JSON `content` column** holding an ordered array of **blocks** (hero, gallery, text, CTA, …) that make up the body.

Blocks are **developer-defined in PHP**. Each block class owns both its Filament admin form schema *and* its API transformation, so there's a single source of truth from the editing UI to the API response. Editors compose and reorder blocks visually with Filament's **Builder** field; the frontend maps each block `type` to a component and renders them in order.

---

## Stack

| Layer | Technology |
|-------|-----------|
| Framework | Laravel |
| Admin panel | Filament (native multi-tenancy) |
| API auth | Laravel Sanctum |
| API layer | Laravel API Resources |
| Media | Spatie Media Library |
| Media storage | S3-compatible (AWS S3 / Cloudflare R2 / DigitalOcean Spaces / MinIO) |
| Database | MySQL / PostgreSQL |
| Frontend | Any (React, Next.js, Vue, Nuxt, Angular) — consumes the API |

---

## Architecture

- **Multi-tenancy** — Handled natively by Filament. All content is scoped to the tenant a user belongs to, so multiple organizations manage content independently within one installation.
- **Block registry** — A central registry composes a **shared block library** plus **per-type blocks**. Content types resolve to `general + type-specific` blocks. The same registry powers both the admin form and the API serialization.
- **Media** — Blocks store media **references (IDs)**, never baked-in URLs. URLs, conversions, and responsive sources are resolved at API output time, so storage/CDN changes never leave stale content.
- **API** — Public read API via Laravel API Resources, secured with Sanctum. Serves clean, frontend-ready JSON with blocks emitted in order.

### Example content payload

```json
{
  "slug": "about-us",
  "title": "About Us",
  "status": "published",
  "seo": {
    "meta_title": "About Us",
    "meta_description": "..."
  },
  "blocks": [
    { "order": 0, "type": "hero", "data": { "title": "...", "image": { "url": "..." } } },
    { "order": 1, "type": "text", "data": { "body": "..." } },
    { "order": 2, "type": "gallery", "data": { "images": [ ... ] } }
  ]
}
```

---

## Requirements

- PHP 8.2+
- Composer
- Node.js & npm
- MySQL or PostgreSQL
- An S3-compatible storage bucket (or MinIO for local dev)

---

## Installation

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>

# 2. Install PHP dependencies
composer install

# 3. Install frontend dependencies (for the Filament panel assets)
npm install

# 4. Environment
cp .env.example .env
php artisan key:generate
```

Configure your `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=cms
DB_USERNAME=root
DB_PASSWORD=

# S3-compatible storage
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=
AWS_BUCKET=
AWS_ENDPOINT=          # e.g. R2 / Spaces / MinIO endpoint
AWS_USE_PATH_STYLE_ENDPOINT=true   # true for MinIO
```

Then:

```bash
# 5. Run migrations
php artisan migrate

# 6. Create an admin user
php artisan make:filament-user

# 7. Build assets
npm run build

# 8. Serve
php artisan serve
```

Visit `/admin` to log in to the Filament panel.

---

## Usage

### Defining a block

Each block is a class that describes its admin form and its API output:

```php
class HeroBlock
{
    public static string $type = 'hero';

    public static function make(): Block
    {
        // Filament Builder block schema
    }

    public static function toApi(array $data): array
    {
        // Resolve image IDs -> URLs, shape the output
    }
}
```

Register it in the block config, mapping it to `general` or a specific type:

```php
'blocks' => [
    'general' => [HeroBlock::class, TextBlock::class, GalleryBlock::class],
    'page'    => [PageHeaderBlock::class],
    'post'    => [AuthorBioBlock::class],
],
```

### Consuming the API

Fetch published content by slug, receive ordered blocks as JSON, and map each block `type` to a frontend component.

---

## Roadmap

- [ ] Filament panel + native tenancy setup
- [ ] `Entry` model & migration (fixed SEO/routing columns + JSON content)
- [ ] Block registry + first blocks (hero, text, gallery)
- [ ] Spatie Media Library on S3-compatible storage
- [ ] Sanctum + public read API resources
- [ ] Block schema versioning strategy
- [ ] Own admin/frontend consumer

---

## License

TBD
