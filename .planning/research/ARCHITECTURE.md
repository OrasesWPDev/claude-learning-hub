# Architecture Research: Claude Learning Hub

**Domain:** Team knowledge base with Slack integration
**Researched:** 2026-02-03
**Confidence:** HIGH (verified via official documentation and established patterns)

## System Overview

```
+-------------------+      +----------------------+      +------------------+
|                   |      |                      |      |                  |
|   Slack Workspace | ---> |  Laravel Backend     | ---> |  Vue Frontend    |
|                   |      |                      |      |  (Inertia.js)    |
+-------------------+      +----------------------+      +------------------+
        |                          |                            |
        |                          v                            |
        |                  +---------------+                    |
        |                  |   MySQL/      |                    |
        |                  |   PostgreSQL  | <------------------+
        |                  +---------------+
        |                          ^
        v                          |
+-------------------+      +---------------+
|  Slack Events API | ---> | Queue Worker  |
|  (link_shared)    |      | (metadata     |
+-------------------+      |  extraction)  |
                           +---------------+
```

## Components

### 1. Slack Integration Layer

**Purpose:** Receive links shared in Slack channels and capture them for the knowledge base.

#### Event Flow: Slack to Laravel

1. **Slack App Configuration**
   - Create Slack App with Event Subscriptions enabled
   - Subscribe to `link_shared` event (triggers when URLs are posted)
   - Configure Request URL pointing to Laravel endpoint

2. **Laravel Webhook Endpoint**
   - Receives POST requests from Slack Events API
   - Must respond with HTTP 200 within 3 seconds (per Slack requirements)
   - Handles URL verification challenge during setup
   - Validates requests using signing secret

3. **Recommended Package:** `lisennk/laravel-slack-events-api`
   - Provides Laravel event integration
   - Auto-handles URL verification
   - Default endpoint: `/api/slack/event/fire`

#### Key Implementation Notes

```php
// Slack sends link_shared event when URL is posted
// Your listener should implement ShouldQueue for 3-second response requirement

class LinkSharedListener implements ShouldQueue
{
    public function handle(LinkShared $event)
    {
        // Queue the link processing immediately
        ProcessSharedLink::dispatch($event->links, $event->user, $event->channel);
    }
}
```

**Sources:**
- [Slack Events API Documentation](https://docs.slack.dev/apis/events-api/)
- [lisennk/laravel-slack-events-api](https://packagist.org/packages/lisennk/laravel-slack-events-api)

---

### 2. Core Application (Laravel)

**Purpose:** Business logic, data persistence, API for frontend, background processing coordination.

#### Directory Structure (Recommended)

```
app/
├── Http/
│   ├── Controllers/
│   │   ├── LinkController.php        # CRUD for links
│   │   ├── CategoryController.php    # Category management
│   │   └── DashboardController.php   # Dashboard/stats
│   ├── Middleware/
│   │   └── HandleInertiaRequests.php # Shared props
│   └── Requests/
│       ├── StoreLinkRequest.php
│       └── UpdateLinkRequest.php
├── Models/
│   ├── Link.php
│   ├── Category.php
│   ├── Tag.php
│   └── User.php
├── Services/
│   ├── Slack/
│   │   ├── SlackEventService.php     # Event handling
│   │   └── SlackNotificationService.php
│   └── Metadata/
│       └── LinkMetadataService.php   # OpenGraph extraction
├── Jobs/
│   ├── ProcessSharedLink.php         # From Slack
│   ├── ExtractLinkMetadata.php       # Fetch OG data
│   └── NotifyLinkAdded.php           # Optional notifications
├── Events/
│   └── LinkCreated.php
├── Listeners/
│   └── SlackLinkSharedListener.php
└── Enums/
    └── LinkStatus.php                # new, reviewed, archived
```

#### Service Layer Pattern

For external integrations (Slack, metadata extraction), use dedicated service classes:

```php
// app/Services/Metadata/LinkMetadataService.php
class LinkMetadataService
{
    public function extract(string $url): array
    {
        // Uses shweshi/opengraph package
        $data = OpenGraph::fetch($url, true);

        return [
            'title' => $data['title'] ?? null,
            'description' => $data['description'] ?? null,
            'image' => $data['image'] ?? null,
            'type' => $data['type'] ?? 'website',
        ];
    }
}
```

**Package:** `shweshi/opengraph` for metadata extraction
- Fetches Open Graph, Twitter Cards, and standard meta tags
- Language-specific metadata support
- Error handling via `FetchException`

**Sources:**
- [Laravel Service Layer Pattern](https://medium.com/@shaunthornburgh/design-patterns-and-best-practices-for-interacting-with-external-apis-in-laravel-3f9ab70ce59e)
- [shweshi/OpenGraph](https://github.com/shweshi/OpenGraph)

---

### 3. Frontend (Vue + Inertia.js)

**Purpose:** Single-page application experience with server-side routing.

#### Directory Structure

```
resources/
├── js/
│   ├── app.js                        # Entry point
│   ├── Pages/
│   │   ├── Dashboard.vue
│   │   ├── Links/
│   │   │   ├── Index.vue             # Browse/search links
│   │   │   ├── Show.vue              # Single link detail
│   │   │   └── Create.vue            # Manual add (optional)
│   │   └── Categories/
│   │       └── Index.vue
│   ├── Components/
│   │   ├── LinkCard.vue              # Reusable link display
│   │   ├── CategoryBadge.vue
│   │   ├── TagPill.vue
│   │   ├── SearchBar.vue
│   │   └── StatusDropdown.vue
│   └── Layouts/
│       └── AuthenticatedLayout.vue   # Main app layout
├── css/
│   └── app.css
└── views/
    └── app.blade.php                 # Root template
```

#### Inertia Data Flow

**Shared Data (via HandleInertiaRequests middleware):**

```php
// app/Http/Middleware/HandleInertiaRequests.php
public function share(Request $request): array
{
    return [
        ...parent::share($request),
        'auth' => [
            'user' => fn () => $request->user()
                ? $request->user()->only('id', 'name', 'email')
                : null,
        ],
        'categories' => fn () => Category::orderBy('name')->get(),
        'flash' => [
            'message' => fn () => $request->session()->get('message'),
        ],
    ];
}
```

**Page Props (from controllers):**

```php
// app/Http/Controllers/LinkController.php
public function index(Request $request)
{
    return Inertia::render('Links/Index', [
        'links' => Link::with(['category', 'tags'])
            ->filter($request->only('search', 'category', 'status'))
            ->paginate(20),
        'filters' => $request->only('search', 'category', 'status'),
    ]);
}
```

**Vue Component Access:**

```vue
<script setup>
import { usePage } from '@inertiajs/vue3'
import { computed } from 'vue'

const page = usePage()
const user = computed(() => page.props.auth.user)
const categories = computed(() => page.props.categories)

defineProps({
    links: Object,
    filters: Object,
})
</script>
```

**Key Inertia 2.0 Features to Leverage:**
- **Prefetching:** Pre-load link details on hover for instant navigation
- **Polling:** Auto-refresh dashboard to show new links from Slack
- **Deferred Props:** Load link metadata after initial page render

**Sources:**
- [Inertia.js Shared Data](https://inertiajs.com/docs/v2/data-props/shared-data)
- [Laravel Inertia Best Practices](https://www.bacancytechnology.com/blog/laravel-inertia)

---

### 4. Background Processing (Queue Workers)

**Purpose:** Handle time-consuming tasks without blocking requests.

#### Job Flow

```
Slack Event Received
        │
        v
SlackLinkSharedListener (sync - must respond in 3s)
        │
        └──> ProcessSharedLink::dispatch() ──> Queue
                                                 │
                                                 v
                                      ProcessSharedLink Job
                                                 │
                                                 ├──> Create Link record (status: pending)
                                                 │
                                                 └──> ExtractLinkMetadata::dispatch()
                                                                 │
                                                                 v
                                                      ExtractLinkMetadata Job
                                                                 │
                                                                 ├──> Fetch OpenGraph data
                                                                 │
                                                                 └──> Update Link record
                                                                      (title, description, image)
```

#### Queue Configuration

```php
// config/queue.php - Use database driver for simplicity
'default' => env('QUEUE_CONNECTION', 'database'),

// Jobs with priorities
class ExtractLinkMetadata implements ShouldQueue
{
    public $queue = 'metadata';  // Separate queue for isolation
    public $tries = 3;           // Retry on failure
    public $timeout = 30;        // Metadata fetch timeout
}
```

#### Running Workers

```bash
# Development
php artisan queue:work

# Production (via Supervisor)
php artisan queue:work --queue=high,default,metadata --sleep=3 --tries=3
```

**Sources:**
- [Laravel Queues Documentation](https://laravel.com/docs/12.x/queues)
- [Queue Workers - How They Work](https://themsaid.com/queue-workers-how-they-work)

---

## Data Flow

### Complete Flow: Slack Message to Displayed Link

```
1. User posts link in Slack channel
        │
        v
2. Slack sends link_shared event to /api/slack/event/fire
        │
        v
3. SlackLinkSharedListener receives event
   - Responds HTTP 200 immediately (required)
   - Dispatches ProcessSharedLink job to queue
        │
        v
4. ProcessSharedLink job runs
   - Creates Link record with:
     - url: extracted URL
     - slack_user_id: who shared it
     - slack_channel_id: where it was shared
     - status: 'pending'
   - Dispatches ExtractLinkMetadata job
        │
        v
5. ExtractLinkMetadata job runs
   - Fetches URL using shweshi/opengraph
   - Updates Link record with:
     - title: from og:title or <title>
     - description: from og:description or meta description
     - image_url: from og:image
     - status: 'new'
        │
        v
6. User visits /links in browser
   - LinkController::index() queries database
   - Returns Inertia response with paginated links
        │
        v
7. Vue renders Links/Index.vue
   - Displays LinkCard components
   - Shows metadata (title, description, image preview)
   - Allows filtering by category, status, search
```

---

## Database Schema (Draft)

### Entity Relationship Diagram

```
+------------------+       +------------------+       +------------------+
|      users       |       |      links       |       |   categories     |
+------------------+       +------------------+       +------------------+
| id               |       | id               |       | id               |
| name             |       | url              |       | name             |
| email            |       | title            |       | slug             |
| password         |       | description      |       | description      |
| created_at       |       | image_url        |       | color            |
| updated_at       |       | category_id (FK) |<------| created_at       |
+------------------+       | submitted_by (FK)|       | updated_at       |
        |                  | slack_user_id    |       +------------------+
        |                  | slack_channel_id |
        +----------------->| status           |       +------------------+
                           | notes            |       |       tags       |
                           | created_at       |       +------------------+
                           | updated_at       |       | id               |
                           +------------------+       | name             |
                                   |                  | slug             |
                                   |                  | created_at       |
                                   v                  | updated_at       |
                           +------------------+       +------------------+
                           |    link_tag      |               |
                           +------------------+               |
                           | link_id (FK)     |<--------------+
                           | tag_id (FK)      |
                           +------------------+
```

### Table Definitions

```sql
-- Links table (core entity)
CREATE TABLE links (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    url VARCHAR(2048) NOT NULL,
    title VARCHAR(500),
    description TEXT,
    image_url VARCHAR(2048),
    category_id BIGINT UNSIGNED,
    submitted_by BIGINT UNSIGNED,           -- User who added (if manual)
    slack_user_id VARCHAR(50),              -- Slack user ID (if from Slack)
    slack_channel_id VARCHAR(50),           -- Slack channel ID
    slack_message_ts VARCHAR(50),           -- Slack message timestamp
    status ENUM('pending', 'new', 'reviewed', 'archived') DEFAULT 'pending',
    notes TEXT,                             -- Admin notes
    created_at TIMESTAMP,
    updated_at TIMESTAMP,

    FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE SET NULL,
    FOREIGN KEY (submitted_by) REFERENCES users(id) ON DELETE SET NULL,
    INDEX idx_status (status),
    INDEX idx_category (category_id),
    INDEX idx_created (created_at)
);

-- Categories table
CREATE TABLE categories (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    color VARCHAR(7),                       -- Hex color for UI
    sort_order INT DEFAULT 0,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Tags table (many-to-many with links)
CREATE TABLE tags (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    slug VARCHAR(50) NOT NULL UNIQUE,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Pivot table
CREATE TABLE link_tag (
    link_id BIGINT UNSIGNED NOT NULL,
    tag_id BIGINT UNSIGNED NOT NULL,
    PRIMARY KEY (link_id, tag_id),
    FOREIGN KEY (link_id) REFERENCES links(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
);
```

### Model Relationships

```php
// app/Models/Link.php
class Link extends Model
{
    protected $casts = [
        'status' => LinkStatus::class,  // Enum
    ];

    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }

    public function tags(): BelongsToMany
    {
        return $this->belongsToMany(Tag::class);
    }

    public function submittedBy(): BelongsTo
    {
        return $this->belongsTo(User::class, 'submitted_by');
    }
}

// app/Models/Category.php
class Category extends Model
{
    public function links(): HasMany
    {
        return $this->hasMany(Link::class);
    }
}

// app/Models/Tag.php - Consider using spatie/laravel-tags
class Tag extends Model
{
    public function links(): BelongsToMany
    {
        return $this->belongsToMany(Link::class);
    }
}
```

**Optional Enhancement:** Use `spatie/laravel-tags` for more advanced tagging features (tag types, translations, ordering).

---

## Directory Structure (Complete)

```
claud-learnings/
├── app/
│   ├── Console/
│   │   └── Kernel.php
│   ├── Enums/
│   │   └── LinkStatus.php
│   ├── Events/
│   │   └── LinkCreated.php
│   ├── Exceptions/
│   │   └── Handler.php
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── CategoryController.php
│   │   │   ├── DashboardController.php
│   │   │   ├── LinkController.php
│   │   │   └── TagController.php
│   │   ├── Middleware/
│   │   │   └── HandleInertiaRequests.php
│   │   └── Requests/
│   │       ├── StoreLinkRequest.php
│   │       └── UpdateLinkRequest.php
│   ├── Jobs/
│   │   ├── ExtractLinkMetadata.php
│   │   └── ProcessSharedLink.php
│   ├── Listeners/
│   │   └── SlackLinkSharedListener.php
│   ├── Models/
│   │   ├── Category.php
│   │   ├── Link.php
│   │   ├── Tag.php
│   │   └── User.php
│   ├── Providers/
│   │   └── AppServiceProvider.php
│   └── Services/
│       ├── Metadata/
│       │   └── LinkMetadataService.php
│       └── Slack/
│           └── SlackEventService.php
├── config/
│   ├── app.php
│   ├── database.php
│   ├── queue.php
│   └── slackEvents.php              # Slack events config
├── database/
│   ├── factories/
│   │   ├── CategoryFactory.php
│   │   ├── LinkFactory.php
│   │   └── TagFactory.php
│   ├── migrations/
│   │   ├── create_users_table.php
│   │   ├── create_categories_table.php
│   │   ├── create_tags_table.php
│   │   ├── create_links_table.php
│   │   └── create_link_tag_table.php
│   └── seeders/
│       ├── CategorySeeder.php
│       └── DatabaseSeeder.php
├── resources/
│   ├── css/
│   │   └── app.css
│   ├── js/
│   │   ├── app.js
│   │   ├── Components/
│   │   │   ├── CategoryBadge.vue
│   │   │   ├── LinkCard.vue
│   │   │   ├── Pagination.vue
│   │   │   ├── SearchBar.vue
│   │   │   ├── StatusDropdown.vue
│   │   │   └── TagPill.vue
│   │   ├── Layouts/
│   │   │   └── AuthenticatedLayout.vue
│   │   └── Pages/
│   │       ├── Categories/
│   │       │   └── Index.vue
│   │       ├── Dashboard.vue
│   │       └── Links/
│   │           ├── Create.vue
│   │           ├── Index.vue
│   │           └── Show.vue
│   └── views/
│       └── app.blade.php
├── routes/
│   ├── api.php                      # Slack webhook endpoint
│   └── web.php                      # Inertia routes
├── tests/
│   ├── Feature/
│   │   ├── LinkControllerTest.php
│   │   └── SlackWebhookTest.php
│   └── Unit/
│       └── LinkMetadataServiceTest.php
├── .env
├── composer.json
├── package.json
└── vite.config.js
```

---

## Build Order

Based on component dependencies, here is the recommended implementation sequence:

### Phase 1: Foundation (Week 1)

**Build first - everything else depends on this.**

1. **Laravel project setup**
   - Install Laravel with Breeze/Inertia/Vue starter kit
   - Configure database connection
   - Set up basic authentication

2. **Database schema**
   - Create migrations for categories, tags, links, link_tag
   - Create models with relationships
   - Create seeders for initial categories

3. **Basic CRUD**
   - LinkController (index, show, create, store, update, destroy)
   - CategoryController
   - Basic Inertia pages (Links/Index, Links/Show)

**Dependency chain:** Database -> Models -> Controllers -> Views

### Phase 2: Core UI (Week 2)

**Make the browsing experience work.**

1. **Vue Components**
   - LinkCard component
   - SearchBar with filters
   - CategoryBadge, TagPill
   - Pagination

2. **Link browsing**
   - Filter by category
   - Filter by status
   - Search by title/description
   - Sort options

3. **Link detail view**
   - Full metadata display
   - Edit capability
   - Status management

### Phase 3: Slack Integration (Week 3)

**External integration - can be developed in parallel after Phase 1.**

1. **Slack App configuration**
   - Create Slack App in workspace
   - Enable Event Subscriptions
   - Subscribe to link_shared event

2. **Laravel webhook endpoint**
   - Install slack-events package
   - Configure verification token
   - Create event listener

3. **Link capture flow**
   - ProcessSharedLink job
   - Store link with Slack metadata

### Phase 4: Metadata Extraction (Week 3-4)

**Background processing - depends on Phase 3.**

1. **Queue setup**
   - Configure database queue driver
   - Create jobs table migration

2. **Metadata service**
   - Install shweshi/opengraph
   - Create LinkMetadataService
   - ExtractLinkMetadata job

3. **Worker configuration**
   - Supervisor setup for production
   - Error handling and retries

### Phase 5: Polish & Deploy (Week 4)

1. **Production configuration**
   - Laravel Forge setup
   - DigitalOcean provisioning
   - Queue worker daemon
   - SSL certificate

2. **Testing**
   - Feature tests for controllers
   - Unit tests for services
   - Slack webhook mocking

3. **Refinements**
   - Inertia prefetching
   - Polling for real-time updates
   - Error handling UI

---

## Build Order Rationale

| Order | Component | Why This Order |
|-------|-----------|----------------|
| 1 | Database + Models | Everything stores/retrieves from database |
| 2 | Basic CRUD | Need to verify data layer works |
| 3 | Vue UI | Need CRUD to have something to display |
| 4 | Slack Integration | Can develop parallel to UI after Phase 1 |
| 5 | Metadata Extraction | Requires links to exist first |
| 6 | Queue Workers | Metadata extraction dispatches jobs |
| 7 | Production Deploy | Everything must work locally first |

**Critical Path:** Database -> Models -> CRUD -> Integration -> Background Jobs

**Parallel Work Possible:**
- After Phase 1, UI (Phase 2) and Slack Integration (Phase 3) can proceed simultaneously
- Metadata extraction naturally follows Slack integration

---

## Sources Summary

### Official Documentation (HIGH confidence)
- [Inertia.js v2 Documentation](https://inertiajs.com/)
- [Laravel 12 Queues](https://laravel.com/docs/12.x/queues)
- [Slack Events API](https://docs.slack.dev/apis/events-api/)

### Verified Packages (HIGH confidence)
- [lisennk/laravel-slack-events-api](https://packagist.org/packages/lisennk/laravel-slack-events-api)
- [shweshi/opengraph](https://github.com/shweshi/OpenGraph)
- [spatie/laravel-tags](https://github.com/spatie/laravel-tags) (optional)

### Architecture Patterns (MEDIUM confidence)
- [Laravel Service Layer Patterns](https://medium.com/@shaunthornburgh/design-patterns-and-best-practices-for-interacting-with-external-apis-in-laravel-3f9ab70ce59e)
- [Laravel Inertia Best Practices 2025](https://www.bacancytechnology.com/blog/laravel-inertia)
- [Laravel Bookmarks Example](https://github.com/jjcosgrove/laravel-bookmarks)

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Inertia + Vue structure | HIGH | Official docs, established patterns |
| Laravel backend | HIGH | Standard Laravel architecture |
| Slack Events API | HIGH | Official Slack documentation |
| Queue architecture | HIGH | Laravel official docs |
| Metadata extraction | HIGH | Verified with shweshi/opengraph docs |
| Database schema | MEDIUM | Standard patterns, may need adjustment |
| Build order | MEDIUM | Logical dependencies, actual timing varies |
