# Pitfalls Research: Claude Learning Hub

**Domain:** Laravel + Vue/Inertia.js application with Slack integration
**Researched:** 2026-02-03
**Confidence:** MEDIUM-HIGH (multiple sources cross-referenced)

---

## Critical Pitfalls

These mistakes cause rewrites, security incidents, or major project delays.

### Slack Integration Pitfalls

| Pitfall | Warning Signs | Prevention | Phase |
|---------|---------------|------------|-------|
| **Exposed Webhook URLs** - Webhook URLs committed to Git or hardcoded in source files. Attackers can post to your Slack channels or exploit channel overrides. | URLs appear in `.env.example`, config files, or Git history. Over 600 leaked webhooks found in 30 minutes of searching public repos. | Store in `.env` only. Add to `.gitignore`. Use `config('services.slack.webhook')` pattern. Never log or expose in error messages. | Phase 1 (Foundation) |
| **No Request Signature Verification** - Accepting Slack requests without verifying `X-Slack-Signature` header allows attackers to spoof requests to your bot. | Bot accepts any POST request to its endpoint. No middleware checking signatures. | Implement Laravel middleware using HMAC-SHA256 verification. Check timestamp is within 5 minutes. Use raw request body for signature calculation. Consider `spatie/laravel-slack-slash-command` package. | Phase 2 (Slack Integration) |
| **Message Injection Attacks** - User-supplied data with large whitespace can split/truncate Slack messages, making malicious content appear as standalone bot messages. | Messages containing user input look corrupted. Users report seeing unexpected messages from your bot. | Validate and sanitize all user input. Strip excessive whitespace. Use allowlist of valid characters. Limit message length. Never allow Slack mentions or links from user input. | Phase 2 (Slack Integration) |
| **Webhook Token vs Signing Secret Confusion** - Using deprecated verification tokens instead of signing secrets for request verification. | Code checks a static "token" field instead of computing HMAC signature. Slack warns about deprecation. | Use signing secrets exclusively. Remove any verification token logic. Update to current Slack API patterns. | Phase 2 (Slack Integration) |
| **Queued Notifications in Transactions** - Slack notifications dispatched via queue inside database transactions may run before transaction commits. | Notifications reference models that "don't exist." Intermittent failures on high load. | Set `after_commit => true` in queue config, or call `->afterCommit()` on notification. Test with database driver first, then Redis. | Phase 3 (Notifications) |

### Laravel Learning Pitfalls

| Pitfall | Warning Signs | Prevention | Phase |
|---------|---------------|------------|-------|
| **N+1 Query Problem** - By far the #1 performance killer. Running hundreds of SQL queries on one page due to lazy loading relationships. | Pages slow to load. Laravel Debugbar shows 50+ queries for simple pages. Database CPU spikes. | Use eager loading with `with()` on every query with relationships. Install Laravel Debugbar from day 1. Run `php artisan model:prune --pretend` to identify issues. | Phase 1 (Foundation) |
| **Fat Controllers** - All business logic crammed into controller methods. Controllers become 500+ line monsters. | Single controller methods exceed 50 lines. Hard to understand request flow. Testing requires mocking HTTP layer. | Keep controllers thin - receive request, call service/action, return response. Extract business logic to Service classes or Actions. | Phase 1 (Foundation) |
| **Hardcoded Configuration** - API keys, URLs, file paths embedded directly in code instead of using `.env` and `config()`. | `grep` finds literal URLs/keys in PHP files. Different behavior across environments. | Use `.env` for all environment-specific values. Access via `config()` helper, never `env()` outside config files. Create config file for Slack settings. | Phase 1 (Foundation) |
| **Skipping Validation** - No or minimal request validation, leading to invalid data in database and security vulnerabilities. | Direct use of `$request->input()` without validation. Database constraint errors in production. | Always use Form Request classes for validation. Define validation rules before writing controller logic. | Phase 1 (Foundation) |
| **Not Using Route Model Binding** - Manually fetching models by ID when Laravel can inject them automatically. | Controllers start with `$resource = Resource::findOrFail($id)`. Repetitive 404 handling code. | Use route model binding: `Route::get('/resource/{resource}', ...)` with type-hinted parameter. Laravel handles 404 automatically. | Phase 1 (Foundation) |
| **Breaking MVC Architecture** - Database queries in Blade templates, business logic in routes, validation in controllers. | `@foreach(User::all()` in Blade. Complex logic in `web.php`. | Views only display data passed to them. Controllers orchestrate. Models handle data. Create dedicated query methods on models. | Phase 1 (Foundation) |
| **Not Reading Laravel Docs** - Most "advanced" tips are in the official documentation that beginners skip. | Re-implementing features Laravel provides. Missing helper functions. Suboptimal patterns. | Read docs for each feature before implementing. Keep docs bookmarked. Laravel docs are exceptionally well-written. | All Phases |
| **No Failed Jobs Table** - Queue failures silently disappear because `failed_jobs` table was never migrated. | Jobs "vanish" without errors. No visibility into what went wrong. | Run `php artisan queue:failed-table` and migrate before using queues. Implement `failed()` method on job classes. | Phase 2 (Background Jobs) |

### Inertia.js Pitfalls

| Pitfall | Warning Signs | Prevention | Phase |
|---------|---------------|------------|-------|
| **Version Mismatch** - Frontend Inertia package out of sync with Laravel adapter causing subtle bugs. | Props not updating. Routes behaving unexpectedly. Console warnings about version. | Pin both `@inertiajs/vue3` and `inertiajs/inertia-laravel` versions together. Update together. Check upgrade guide. | Phase 1 (Foundation) |
| **Layout Re-rendering** - Not using persistent layouts causes full component re-mount on every navigation. | Header/sidebar flicker on navigation. Form state lost unexpectedly. Performance issues. | Define layout at page component level using `defineOptions({ layout: MainLayout })`. Use persistent layouts for shared UI. | Phase 1 (Foundation) |
| **useForm Reactivity Loss** - Spreading `useForm` data loses Vue reactivity, causing forms to not update. | Form fields don't reflect changes. Submit sends stale data. Vue devtools shows stale state. | Don't spread form data. Use `form.field` directly. If spreading needed, use `toRefs()`. | Phase 1 (Foundation) |
| **Prop Drilling in Large Apps** - Passing data through many component layers becomes unmanageable. | Components have 10+ props. Changes require editing 5 files. Hard to trace data flow. | Use Pinia for client-side state. Keep Inertia for server->page data. Hybrid approach: Inertia for initial data, Pinia for UI state. | Phase 3+ (Scale) |
| **CSRF Token Mismatch** - Axios not configured with CSRF token for non-Inertia requests. | 419 errors on AJAX calls. Forms work but API calls fail. | Ensure `bootstrap.js` sets `axios.defaults.headers.common['X-CSRF-TOKEN']`. Inertia handles this for its requests automatically. | Phase 1 (Foundation) |
| **JSON Response When Inertia Expected** - Modal workflows returning JSON instead of Inertia responses cause page breaks. | Modal submissions cause full page reload or errors. "Expecting Inertia response" console errors. | Use `Inertia::render()` or redirects for Inertia routes. If you need JSON, make it a separate API endpoint. | Phase 2 (Features) |

### Deployment Pitfalls

| Pitfall | Warning Signs | Prevention | Phase |
|---------|---------------|------------|-------|
| **APP_DEBUG=true in Production** - Exposes stack traces, file paths, environment variables, and API keys to users. | Error pages show full stack traces. `.env` values visible in errors. Security scanners flag the site. | Triple-check before every deploy. Add CI/CD check that fails if `APP_DEBUG=true` in production `.env`. Automate this verification. | Pre-Launch |
| **Queue Workers Not Managed** - Starting workers manually means they die on deploy or crash without restart. | Jobs stop processing mysteriously. Workers don't restart after deploy. Memory leaks crash servers. | Use Supervisor or Laravel Forge to manage workers. Configure memory limits and restart policies. Run `php artisan queue:restart` after every deploy. | Pre-Launch |
| **No Cache Optimization** - Missing route/config/view caching in production hurts performance. | Slow first requests. High CPU for config parsing. | Run `php artisan optimize` in deploy script. This caches config, routes, views. Clear on deploy: `php artisan optimize:clear` then `php artisan optimize`. | Pre-Launch |
| **Migrations Not Tested** - Running untested migrations in production risks data loss. | Column not found errors after deploy. Data truncated warnings. Rollback failures. | Always test in staging first. Use `php artisan migrate --pretend` to preview. Never edit deployed migrations - create new ones to fix. | All Phases |
| **Sync Queue Driver in Production** - Using `QUEUE_CONNECTION=sync` means jobs run synchronously, blocking requests. | Slow page loads when jobs should be queued. No entries in jobs table. | Use Redis or database driver in production. Set up Horizon for Redis. Never sync in production. | Pre-Launch |
| **Exposed Sensitive Files** - `.env`, `.git/`, `storage/` accessible via web. | Direct URL access to `/.env` returns file contents. Git history exposed. | Configure web server (Nginx/Apache) to deny access to sensitive paths. Point document root to `/public` only. Test with curl. | Pre-Launch |
| **Session/Cache Driver Mismatch** - Using `file` driver for sessions/cache across multiple servers. | User logged out randomly. Session data lost. Inconsistent behavior. | Use Redis or database for sessions in production. Single source of truth across servers. | Pre-Launch |

---

## Security Considerations

### Slack-Specific Security

**Webhook URL Protection:**
- Webhook URLs are effectively bearer tokens - anyone with the URL can post messages
- Never log webhook URLs in error messages or debug output
- Rotate webhook URLs if any suspicion of compromise
- Consider using Slack Apps with OAuth instead of plain webhooks for better control

**Request Verification (CRITICAL):**
```php
// Middleware pattern for Slack signature verification
public function handle($request, Closure $next)
{
    $timestamp = $request->header('X-Slack-Request-Timestamp');
    $signature = $request->header('X-Slack-Signature');

    // Reject if timestamp > 5 minutes old (replay attack prevention)
    if (abs(time() - $timestamp) > 300) {
        abort(403, 'Request timestamp too old');
    }

    // Compute expected signature
    $sigBasestring = 'v0:' . $timestamp . ':' . $request->getContent();
    $mySignature = 'v0=' . hash_hmac('sha256', $sigBasestring, config('services.slack.signing_secret'));

    if (!hash_equals($mySignature, $signature)) {
        abort(403, 'Invalid signature');
    }

    return $next($request);
}
```

**Input Sanitization:**
- Never trust data from Slack events - sanitize before display or storage
- Slack usernames can contain special characters
- Block list is insufficient - use allowlist for valid characters

### Laravel Authentication Security (Sanctum/Breeze)

**Domain Configuration (CRITICAL for SPA):**
- SPA and API MUST share top-level domain (e.g., `app.example.com` and `api.example.com`)
- Sanctum uses HttpOnly cookies that cannot be shared across different domains
- Misconfigure this and you'll get endless 401 errors with no clear cause

**Stateful Domains:**
```php
// Correct format in sanctum.php
'stateful' => [
    'localhost:5173',      // Correct
    'yourapp.com',         // Correct
    // 'http://localhost/', // WRONG - no protocol, no trailing slash
],
```

**Session Fixation:**
- Call `$request->session()->regenerate()` after login
- Breeze handles this, but custom auth often misses it

**Rate Limiting:**
- Login and password reset are primary attack targets
- Configure throttling in `RouteServiceProvider` or use `throttle` middleware

---

## Performance Traps

### Database Performance

**The N+1 Query Monster:**
```php
// BAD - N+1 queries
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->author->name; // Query per post!
}

// GOOD - 2 queries total
$posts = Post::with('author')->get();
foreach ($posts as $post) {
    echo $post->author->name; // Already loaded
}
```

**Detection:** Install Laravel Debugbar immediately. It shows query count per page. If you see 50+ queries loading 20 records, you have N+1.

**Missing Database Indexes:**
- Foreign keys need indexes for JOIN performance
- Columns in `WHERE` clauses need indexes
- Use `EXPLAIN` on slow queries

### Queue Performance

**Unoptimized Job Serialization:**
- Passing Eloquent models to jobs serializes all attributes
- Pass only IDs, re-fetch in job if needed
- Large payloads slow down queue and may hit Redis limits

**Single Queue Bottleneck:**
- All jobs in default queue = critical jobs wait behind bulk operations
- Create separate queues: `high`, `default`, `low`
- Configure workers per queue based on priority

### Frontend Performance

**Inertia Partial Reloads:**
- Default: Full page data on every navigation
- Use `only` option to fetch only changed data
- Reduces payload size significantly

**Asset Compilation:**
- Run `npm run build` for production (minification, tree-shaking)
- Development builds are 5-10x larger

---

## What "Done Wrong" Looks Like

### Anti-Pattern 1: The God Controller

```php
// BAD: 300-line controller method
class ResourceController extends Controller
{
    public function store(Request $request)
    {
        // Validation inline
        $validated = $request->validate([...50 rules...]);

        // Business logic
        if ($validated['type'] === 'special') {
            // 50 lines of special handling
        }

        // External API calls
        $slackResponse = Http::post(env('SLACK_WEBHOOK'), [...]);

        // More business logic
        // Database operations
        // Email sending
        // Event dispatching
        // Response formatting
    }
}
```

**Why it's wrong:** Untestable, unreadable, unmaintainable. Changes require understanding 300 lines of context.

**How to fix:** Form Request for validation, Service class for business logic, Event for notifications.

### Anti-Pattern 2: Naked Slack Endpoint

```php
// BAD: No verification, no sanitization
Route::post('/slack/events', function (Request $request) {
    $event = $request->json();

    // Trust everything from "Slack"
    Resource::create([
        'content' => $event['text'],  // XSS waiting to happen
        'user' => $event['user'],     // Attacker-controlled
    ]);

    return response('ok');
});
```

**Why it's wrong:** Anyone can POST to this endpoint. No signature verification. Stores unsanitized input.

**How to fix:** Middleware for signature verification, sanitize all input, use proper controller.

### Anti-Pattern 3: Environment Confusion

```php
// BAD: env() scattered throughout codebase
class SlackService
{
    public function send($message)
    {
        $webhook = env('SLACK_WEBHOOK_URL');  // Cached in production!
        Http::post($webhook, ['text' => $message]);
    }
}
```

**Why it's wrong:** `env()` returns `null` when config is cached in production. Leads to silent failures.

**How to fix:** Use `config()` everywhere. Define in `config/services.php`:
```php
'slack' => [
    'webhook_url' => env('SLACK_WEBHOOK_URL'),
],
// Then: config('services.slack.webhook_url')
```

### Anti-Pattern 4: Synchronous Slack in Request Cycle

```php
// BAD: User waits for Slack API
public function store(Request $request)
{
    $resource = Resource::create($request->validated());

    // User waits 500ms-2s for this
    Http::post(config('services.slack.webhook'), [
        'text' => "New resource: {$resource->title}"
    ]);

    return redirect()->route('resources.show', $resource);
}
```

**Why it's wrong:** Slack API latency (500ms-2s) blocks the user. Slack outage = your app hangs.

**How to fix:** Queue the notification:
```php
$resource = Resource::create($request->validated());
SendSlackNotification::dispatch($resource);  // Immediate return
return redirect()->route('resources.show', $resource);
```

---

## Phase-Specific Warnings

| Phase | Topic | Likely Pitfall | Mitigation |
|-------|-------|---------------|------------|
| 1 | Project Setup | Not setting up Debugbar/Telescope from start | Install debugging tools in first commit |
| 1 | Database | Not using migrations, manual DB changes | All schema changes via migrations only |
| 1 | Inertia Setup | Version mismatch between packages | Document exact versions, update together |
| 2 | Slack Integration | No request signature verification | Implement middleware before first endpoint |
| 2 | Slack Integration | Webhook URL in codebase | Environment variable from day 1 |
| 2 | Queues | Using sync driver for "testing" | Use database driver even in development |
| 3 | Authentication | Domain mismatch for SPA auth | Plan domain structure before implementation |
| 3 | State Management | Prop drilling as app grows | Introduce Pinia early if >10 shared states |
| 4 | Deployment | No deployment checklist | Create and follow checklist every deploy |
| 4 | Production | APP_DEBUG=true | CI/CD check that fails on debug mode |

---

## Sources

### Slack Integration
- [Avoiding Common Slack Mistakes - Essential Guide for Remote Laravel Developers](https://moldstud.com/articles/p-avoiding-common-slack-mistakes-essential-guide-for-remote-laravel-developers)
- [Slack Web Hook Message Injection Advisory - Pulse Security](https://pulsesecurity.co.nz/advisories/slack-message-injection)
- [Overlooked Webhooks Exploit Endpoint Vulnerability in Slack Channels | CloudSEK](https://www.cloudsek.com/threatintelligence/leaked-slack-webhooks-exploit-endpoint-vulnerability-in-slack-channels)
- [Security best practices | Slack Developer Docs](https://docs.slack.dev/security/)
- [Verifying requests from Slack | Slack Developer Docs](https://docs.slack.dev/authentication/verifying-requests-from-slack/)
- [Laravel middleware for validating slack signing secret (GitHub Gist)](https://gist.github.com/pingcheng/f7500adf1b1009df3ed341f511305b0d)
- [spatie/laravel-slack-slash-command (GitHub)](https://github.com/spatie/laravel-slack-slash-command)

### Laravel Beginner Mistakes
- [5 Common Mistakes Beginner Laravel Developers Must Avoid | Medium](https://medium.com/@developerawam/5-common-mistakes-beginner-laravel-developers-must-avoid-acac327588c9)
- [Laravel: 9 Typical Mistakes Juniors Make | Laravel Daily](https://laraveldaily.com/post/laravel-typical-mistakes-juniors-make)
- [19+ Laravel Best Practices for Developers in 2025 | ButterCMS](https://buttercms.com/blog/laravel-best-practices/)
- [Laravel Good vs Bad Practices: Real Code Examples | Medium](https://codermanjeet.medium.com/laravel-good-vs-bad-practices-real-code-examples-thatll-save-your-project-7382bd57535c)
- [4 Laravel Best Practices Every New Developer Should Know | Medium](https://medium.com/@ariscool/4-laravel-best-practices-every-new-developer-should-know-6f170ce9065b)

### Inertia.js
- [Inertia.js - The Modern Monolith](https://inertiajs.com/)
- [useForm data is not reactive/looses reactivity (GitHub Issue)](https://github.com/inertiajs/inertia/issues/1304)
- [Maximizing Efficiency: Pinia vs Inertia Data Sharing](https://medalibouk.com/post/maximizing-efficiency-pinia-vs-inertia-data-sharing-in-laravel-vue-projects)
- [Avoiding Common Pitfalls in Vue.js Reactivity](https://infinitejs.com/posts/avoiding-vue-reactivity-pitfalls/)

### Queue & Jobs
- [The Ultimate Guide to Debugging Laravel Queued Jobs Like a Pro | Medium](https://masteryoflaravel.medium.com/the-ultimate-guide-to-debugging-laravel-queued-jobs-like-a-pro-95261446da57)
- [30+ Laravel Queue Mistakes You Must Avoid in Production | Medium](https://medium.com/@mdzahid.pro/30-laravel-queue-mistakes-you-must-avoid-in-production-ff259d6e067a)

### Deployment
- [Deploy Laravel To Production Like A Pro (2025 Guide & Checklist)](https://www.php-dev-zone.com/laravel-production-deployment-checklist-and-common-mistakes-to-avoid)
- [A Security Checklist for Your Laravel App Before You Hit Deploy](https://dev.to/kamruljpi/a-security-checklist-for-your-laravel-app-before-you-hit-deploy-2578)
- [Checklist: 8 Things to Do When Launching Laravel Project LIVE | Laravel Daily](https://laraveldaily.com/post/checklist-8-things-launching-laravel-project-live)

### Authentication
- [Laravel SPA Authentication: setup and common mistakes](https://cdruc.com/laravel-spa-auth-extended)
- [Best Laravel Authentication Methods in 2025](https://www.softwarebhai.com/blog/best-laravel-authentication-methods-2025)
- [Beginner to Production: Building Laravel Authentication & Authorization the Right Way](https://blog.greeden.me/en/2026/01/28/beginner-to-production-building-laravel-authentication-authorization-the-right-waylogin-registration-sanctum-policy-gate-role-design-security-and-accessible-forms-error-ui/)
