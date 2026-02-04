# Stack Research: Claude Learning Hub

**Project:** Team knowledge base for Claude/Claude Code learning resources
**Researched:** 2026-02-03
**Overall Confidence:** HIGH

---

## Recommended Stack

### Core Framework

| Technology | Version | Purpose | Rationale | Confidence |
|------------|---------|---------|-----------|------------|
| **Laravel** | 11.x | Backend framework | Current LTS version (bug fixes until Sep 2025, security until Mar 2026). Streamlined application structure with single AppServiceProvider, code-first config in bootstrap/app.php. Requires PHP 8.2+. | HIGH |
| **PHP** | 8.2+ | Runtime | Required by Laravel 11. PHP 8.3 or 8.4 recommended for best performance. | HIGH |
| **Inertia.js** | 2.0 | SPA bridge | Eliminates need for separate API - use Laravel controllers directly with Vue pages. Acts as "glue" between Laravel backend and Vue frontend. | HIGH |
| **Vue.js** | 3.5.x | Frontend framework | Latest stable (3.5.18). Includes Composition API, reactive props destructure, useTemplateRef(), improved SSR, and 56% memory reduction. | HIGH |
| **Tailwind CSS** | 4.0 | Styling | Major refactor - zero configuration needed, one-line import (`@import "tailwindcss"`), native Vite plugin for maximum performance. | HIGH |

### Authentication & Scaffolding

| Package | Version | Purpose | Rationale | Confidence |
|---------|---------|---------|-----------|------------|
| **Laravel Breeze** | Latest | Auth scaffolding | Choose over Jetstream - lighter weight, easier to customize, perfect for learning Laravel. Includes Vue/Inertia/TypeScript scaffolding out of the box. No 2FA/teams needed for this project. | HIGH |

**Installation command:**
```bash
composer require laravel/breeze --dev
php artisan breeze:install vue --typescript --ssr
```

When prompted, select:
- Stack: `vue`
- Dark mode: `yes` (optional, but nice)
- TypeScript: `yes` (strongly recommended)
- SSR: `yes` (better SEO, faster initial load)
- Pest: `yes` (modern testing)

### Slack Integration

| Package | Version | Purpose | Rationale | Confidence |
|---------|---------|---------|-----------|------------|
| **nwilging/laravel-slack-bot** | 2.1.x | Slash commands & bot | Best option for receiving slash commands. Supports bot tokens for full API access, channel search, slash command handlers. Works alongside official notification channel. | MEDIUM |
| **laravel/slack-notification-channel** | 3.7.x | Outbound notifications | Official Laravel package for sending messages TO Slack. 62.5M downloads, well-maintained. Use for confirmation messages after link capture. | HIGH |

**Why this combination:**
- `laravel-slack-bot` handles **inbound** (slash commands like `/capture https://example.com`)
- `slack-notification-channel` handles **outbound** (confirmation messages, status updates)

**Alternative considered:**
- `spatie/laravel-slack-slash-command` (v1.12.2) - Good but less active; 3-second response limit can be challenging. The nwilging package handles delayed responses better.

**Slack App Setup Requirements:**
1. Create app at https://api.slack.com/apps
2. Enable "Slash Commands" feature
3. Create bot token (for nwilging package)
4. Create incoming webhook (for notification channel)
5. Set signing secret in `.env`

```env
SLACK_BOT_TOKEN=xoxb-...
SLACK_SIGNING_SECRET=...
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/...
```

### Metadata Extraction

| Package | Version | Purpose | Rationale | Confidence |
|---------|---------|---------|-----------|------------|
| **hazaveh/php-link-preview** | Latest | URL metadata | Extracts OG tags (title, description, image) + favicon. PHP 8.2+ compatible, uses Guzzle. Lightweight, no external API dependency. | MEDIUM |

**How it works:**
```php
use Hazaveh\LinkPreview\LinkPreview;

$preview = new LinkPreview();
$result = $preview->getPreview('https://example.com');
// Returns: title, description, image, icon
```

**Fallback strategy:**
1. Primary: OG tags (og:title, og:description, og:image)
2. Secondary: Meta tags (description, title tag)
3. Tertiary: Page content analysis

**Alternative considered:**
- External APIs (Unfurl.io, Embed.ly, OpenGraph.io) - Adds external dependency, potential costs, rate limits. Self-hosted approach is better for control and learning.
- Native PHP `get_meta_tags()` - Too basic, doesn't handle OG tags well.

### Database

| Technology | Version | Purpose | Rationale | Confidence |
|------------|---------|---------|-----------|------------|
| **MySQL** | 8.0+ | Database | Better choice for learning Laravel - more documentation, tutorials use MySQL by default, simpler setup. PostgreSQL advantages (JSONB, full-text search) not needed for this project scope. DigitalOcean managed MySQL available. | HIGH |

**Why MySQL over PostgreSQL:**
- Simpler for Laravel learners (most tutorials assume MySQL)
- Laravel Forge + DigitalOcean MySQL is straightforward
- Project doesn't need PostgreSQL's advanced features (JSONB indexing, partial indexes)
- If moving to Laravel Cloud later, migration path is documented

**If you want PostgreSQL:** Laravel Cloud uses PostgreSQL with hibernation support. Consider PostgreSQL if you plan to use Laravel Cloud for hosting.

### Supporting Libraries

| Package | Version | Purpose | Rationale | Confidence |
|---------|---------|---------|-----------|------------|
| **spatie/laravel-query-builder** | 6.4.x | API query building | If you add API endpoints later. Filtering, sorting, includes via query params. Follows JSON:API spec. | MEDIUM |
| **laravel/pint** | Pre-installed | Code style | Already in Laravel 11. Run `./vendor/bin/pint` to auto-fix code style. | HIGH |
| **pestphp/pest** | 3.x | Testing | Modern PHP testing. Expressive syntax. Breeze installs this if selected. | HIGH |
| **@vueuse/core** | 11.x | Vue utilities | Collection of Vue composition utilities. `useFetch`, `useLocalStorage`, `useDark`, etc. | MEDIUM |
| **ziggy-js** | Latest | Route handling | Inertia includes this for using Laravel routes in JS (`route('resources.index')`). | HIGH |

### Development Tools

| Tool | Purpose | Rationale |
|------|---------|-----------|
| **Vite** | Build tool | Ships with Laravel. Fast HMR, native ES modules. |
| **TypeScript** | Type safety | Catches errors at compile time. Breeze sets this up. |
| **ESLint + Prettier** | JS/Vue linting | Add for consistent frontend code. |
| **Laravel Debugbar** | Debug | Essential for development. Install with `--dev`. |

---

## What NOT to Use

### Avoid These Packages

| Package | Why to Avoid |
|---------|--------------|
| **maknz/slack-laravel** | No longer maintained. Abandoned. |
| **Laravel Jetstream** | Overkill for this project. Complex, steep learning curve. Includes team management, 2FA you don't need. |
| **laravel/ui** | Legacy. Deprecated in favor of Breeze. |
| **Bootstrap** | Tailwind 4 is the Laravel ecosystem standard. Bootstrap adds unnecessary weight. |
| **Axios (standalone)** | Inertia handles HTTP. Don't add separate HTTP library. |
| **Vuex** | Vue 3 Composition API + Pinia is the modern approach. Inertia handles most state needs. |

### Avoid These Patterns

| Pattern | Why to Avoid | Do Instead |
|---------|--------------|------------|
| Building a separate API | Inertia eliminates this need | Use Inertia's controller responses |
| jQuery | Unnecessary with Vue 3 | Use Vue reactivity |
| Blade for dynamic content | Defeats Inertia's purpose | Use Vue components |
| Global state management initially | Premature optimization | Let Inertia handle page data; add Pinia if needed later |
| External metadata APIs | Adds dependency, cost, rate limits | Self-hosted extraction with hazaveh/php-link-preview |

---

## Configuration Notes

### Vite Configuration (vite.config.js)

```javascript
import { defineConfig } from 'vite'
import laravel from 'laravel-vite-plugin'
import vue from '@vitejs/plugin-vue'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
        vue({
            template: {
                transformAssetUrls: {
                    base: null,
                    includeAbsolute: false,
                },
            },
        }),
        tailwindcss(),
    ],
    resolve: {
        alias: {
            '@': '/resources/js',
        },
    },
})
```

### Tailwind CSS 4 (resources/css/app.css)

```css
@import "tailwindcss";

@source "../views/**/*.blade.php";
@source "../js/**/*.{js,ts,vue}";
```

### Environment Variables (.env additions)

```env
# Slack Integration
SLACK_BOT_TOKEN=xoxb-your-bot-token
SLACK_SIGNING_SECRET=your-signing-secret
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/...

# Optional: For queued jobs
QUEUE_CONNECTION=database
```

### Key Laravel Config Points

1. **bootstrap/app.php** - Laravel 11's new configuration hub:
   - Middleware registration
   - Exception handling
   - Route loading

2. **config/services.php** - Add Slack configuration:
```php
'slack' => [
    'notifications' => [
        'bot_user_oauth_token' => env('SLACK_BOT_TOKEN'),
        'channel' => env('SLACK_BOT_CHANNEL', '#general'),
    ],
],
```

---

## Installation Sequence

```bash
# 1. Create Laravel project
composer create-project laravel/laravel claude-learning-hub
cd claude-learning-hub

# 2. Install Breeze with Vue/TypeScript/SSR
composer require laravel/breeze --dev
php artisan breeze:install vue --typescript --ssr --pest

# 3. Install Node dependencies
npm install

# 4. Install Slack packages
composer require laravel/slack-notification-channel
composer require nwilging/laravel-slack-bot

# 5. Install metadata extraction
composer require hazaveh/php-link-preview

# 6. Install development tools
composer require barryvdh/laravel-debugbar --dev

# 7. Configure database and migrate
php artisan migrate

# 8. Start development servers (two terminals)
php artisan serve
npm run dev
```

---

## Version Matrix Summary

| Component | Version | Source |
|-----------|---------|--------|
| Laravel | 11.x | [Official Docs](https://laravel.com/docs/11.x/releases) |
| PHP | 8.2+ | Laravel 11 requirement |
| Vue.js | 3.5.x | [Vue Blog](https://blog.vuejs.org/posts/vue-3-5) |
| Inertia.js | 2.0 | [Inertia Docs](https://inertiajs.com/docs/v2/) |
| Tailwind CSS | 4.0 | [Tailwind Blog](https://tailwindcss.com/blog/tailwindcss-v4) |
| laravel/slack-notification-channel | 3.7.x | [Packagist](https://packagist.org/packages/laravel/slack-notification-channel) |
| nwilging/laravel-slack-bot | 2.1.x | [GitHub](https://github.com/nwilging/laravel-slack-bot) |
| hazaveh/php-link-preview | Latest | [GitHub](https://github.com/hazaveh/php-link-preview) |
| spatie/laravel-query-builder | 6.4.x | [Spatie Docs](https://spatie.be/docs/laravel-query-builder/v6/introduction) |

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Core Framework (Laravel 11) | HIGH | Verified via official docs |
| Vue/Inertia/Tailwind | HIGH | Verified via official docs |
| Laravel Breeze | HIGH | Official Laravel package |
| Slack Notification Channel | HIGH | Official Laravel package, verified version |
| Slack Bot (nwilging) | MEDIUM | Active package but last release May 2023; verify compatibility with Laravel 11 during implementation |
| Metadata Extraction | MEDIUM | Package works but verify edge cases during implementation |
| MySQL vs PostgreSQL | HIGH | Standard recommendation for Laravel learners |

---

## Sources

### Official Documentation
- [Laravel 11.x Release Notes](https://laravel.com/docs/11.x/releases)
- [Laravel Starter Kits](https://laravel.com/docs/11.x/starter-kits)
- [Inertia.js v2 Documentation](https://inertiajs.com/docs/v2/)
- [Vue 3.5 Announcement](https://blog.vuejs.org/posts/vue-3-5)
- [Tailwind CSS v4.0](https://tailwindcss.com/blog/tailwindcss-v4)
- [Tailwind CSS Laravel Guide](https://tailwindcss.com/docs/guides/laravel)

### Package Repositories
- [laravel/slack-notification-channel](https://packagist.org/packages/laravel/slack-notification-channel)
- [nwilging/laravel-slack-bot](https://github.com/nwilging/laravel-slack-bot)
- [hazaveh/php-link-preview](https://github.com/hazaveh/php-link-preview)
- [spatie/laravel-query-builder](https://github.com/spatie/laravel-query-builder)
- [spatie/laravel-slack-slash-command](https://github.com/spatie/laravel-slack-slash-command)

### Community Resources
- [Laravel MySQL vs PostgreSQL Comparison](https://medium.com/appfoster/database-design-patterns-when-to-use-mysql-vs-postgresql-in-your-next-laravel-project-eee38adaec17)
- [Laravel Breeze vs Jetstream](https://www.twilio.com/en-us/blog/laravel-breeze-vs-laravel-jetstream)
- [Laravel 12 Inertia Vue Setup Guide](https://chapimaster.com/programming/laravel/laravel-12-inertia-vue-setup)
