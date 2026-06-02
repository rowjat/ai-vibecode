# Laravel Boost Skill — Implementation Guide

Complete spec-driven development system for Laravel projects integrated with vibecode-pro-max-kit.

## Table of Contents

1. [Quick Start](#quick-start)
2. [How It Works](#how-it-works)
3. [Workflow Examples](#workflow-examples)
4. [Scripts Reference](#scripts-reference)
5. [Integration Points](#integration-points)
6. [Safety Systems](#safety-systems)
7. [Troubleshooting](#troubleshooting)

---

## Quick Start

### Installation

Laravel Boost activates automatically when:
- Project has `composer.json` with `laravel/framework`
- You mention Laravel-specific terms in your request

No additional setup needed — the skill integrates seamlessly into vibecode workflows.

### Your First Request

```
"Add a webhook dispatcher to our API"
```

The harness will:
1. **RESEARCH** — Scans existing event/listener architecture, queue setup, API patterns
2. **INNOVATE** — Explores webhook delivery strategies (sync vs async, retry logic)
3. **PLAN** — Generates migration, model, event/listener, controller, tests
4. **EXECUTE** — Implements with validation, runs test suite, commits atomically
5. **UPDATE** — Captures webhook patterns in context for future features

---

## How It Works

### Phase-Locked Execution

Each phase has distinct capabilities and restrictions:

```
RESEARCH (Read-Only)
  ↓ Scans: models, migrations, routes, providers, config
  ↓ Cannot write or execute code
  ↓ Output: Architecture summary + findings
  
INNOVATE (Analysis-Only)
  ↓ Explores 2-3 design approaches
  ↓ Cannot modify codebase
  ↓ Output: Comparison of approaches with trade-offs

PLAN (Write to process/ Only)
  ↓ Generates spec with migrations, models, tests
  ↓ Cannot modify app/ or routes/
  ↓ Output: Detailed spec for review

EXECUTE (Full Access, Test-Driven)
  ↓ Implements exactly to plan
  ↓ Validates migrations are reversible
  ↓ Runs affected tests before commit
  ↓ Output: Working code with evidence pack
```

### Skill Discovery

When you request a Laravel feature, the orchestrator auto-matches:
- **vc-laravel-research** — Architecture scanning
- **vc-laravel-innovate** — Design pattern exploration
- **vc-laravel-plan** — Spec generation with templates
- **vc-laravel-execute** — Implementation with safety checks
- **vc-security** — Secret detection in config
- **vc-scenario** — Edge case analysis for migrations

---

## Workflow Examples

### Example 1: Add API Versioning

**Your Request:**
```
"Add API v2 endpoint structure alongside v1, keeping both active for 6 months"
```

**RESEARCH Output:**
```
Current state:
  • Routes: routes/api.php (flat structure)
  • Controllers: app/Http/Controllers/ (no grouping)
  • API responses: Using custom ApiResponse class
  • Versioning: None currently

Related patterns found:
  • API versioning could use route prefix + middleware
  • Controller grouping by version recommended
  • Response transformer for v2 specs
```

**INNOVATE Output:**
```
Approach 1: Route prefix versioning
  Pros: Simple routing, clear URL structure
  Cons: Duplicated controllers

Approach 2: Request middleware versioning
  Pros: Single codebase, negotiation-based
  Cons: More complex, hidden API differences

Recommendation: Approach 1 (prefix) for your use case
  Reason: Clear deprecation path, easier testing, v1/v2 can diverge safely
```

**PLAN Output:**
```
Touchpoints:
  • routes/api.php — Add v2 route group
  • app/Http/Controllers/Api/ — Create V2 subdirectory
  • app/Http/Middleware/ — New ApiVersionMiddleware
  • tests/Feature/Api/ — v2 endpoint tests

Migrations: None (pure routing)

Files to Create:
  • routes/api/v2.php (new)
  • app/Http/Controllers/Api/V2/UserController.php
  • app/Http/Middleware/ApiVersionMiddleware.php
  • tests/Feature/Api/V2/UserApiTest.php

Blast Radius:
  • No database changes
  • No existing endpoint breaks
  • Old tests run unchanged
  • New v2 tests validate new structure
```

**EXECUTE Output:**
```
✅ Created routes/api/v2.php with v2 route group
✅ Created V2 controllers with API resource methods
✅ Added ApiVersionMiddleware for accept header
✅ Implemented v2 response formatting
✅ All v1 tests pass (16/16)
✅ All v2 tests pass (8/8)
✅ Created atomic commits:
   1. "chore: add API v2 route structure"
   2. "feat: implement V2 resource controllers"
   3. "feat: add API version middleware"
```

---

### Example 2: Create Subscription Billing Module

**Your Request:**
```
"Build a subscription management system with Stripe integration, handle recurring charges and cancellations"
```

**RESEARCH Output:**
```
Current state:
  • Queue setup: Redis (queue driver)
  • Event system: Using Laravel Events
  • User model: Basic auth setup
  • Database: PostgreSQL 14

Packages found:
  • laravel-stripe (not installed) — recommend stripe/stripe-php
  • laravel-cashier (not installed) — optional, heavyweight
  • laravel-jobs (built-in)

Related patterns in codebase:
  • PaymentService class exists (one-time payments)
  • WebhookController routes webhooks
  • User model has polymorphic relations
```

**INNOVATE Output:**
```
Approach 1: Use Laravel Cashier
  Pros: Battle-tested, complete
  Cons: Opinionated structure, heavy dependencies

Approach 2: Custom with stripe-php
  Pros: Full control, lightweight, matches existing PaymentService
  Cons: More code to write and test

Recommendation: Approach 2 (custom)
  Reason: Your PaymentService already exists, team familiar with pattern
  Strategy: SubscriptionService extends PaymentService, reuses webhook handling
```

**PLAN Output:**
```
Models & Migrations:
  • Subscription model (id, user_id, stripe_id, status, plan, current_period_end)
  • subscription_items table (for multiple products per subscription)
  • Reverse migration rollback: deactivate active subscriptions, preserve records

Services:
  • SubscriptionService — Create, update, cancel
  • StripeWebhookService — Handle charge, cancellation events
  • BillingNotificationService — Email confirmation/failure

Events & Jobs:
  • SubscriptionCreated event → SendSubscriptionConfirmation job
  • SubscriptionCanceled event → ProcessRefund job
  • charge.succeeded webhook → UpdateSubscriptionStatus job

Controllers & Routes:
  • SubscriptionController (store, update, destroy, show)
  • API routes: POST /api/subscriptions, PATCH /api/subscriptions/{id}, DELETE
  • Webhook route: POST /api/webhooks/stripe (already exists)

Tests:
  • SubscriptionServiceTest — Create, cancel, handle edge cases
  • StripeWebhookTest — Verify webhook event handling
  • SubscriptionControllerTest — API endpoints
  • BillingNotificationTest — Email delivery

Blast Radius:
  • New tables: subscriptions, subscription_items (no breaking changes)
  • User model: Add HasMany subscription relation (additive)
  • Webhook route: Extend existing (backward compatible)
  • Tests required: All SubscriptionService, Controller, Webhook tests
```

**EXECUTE Output:**
```
✅ Created Subscription model + migration
✅ Created SubscriptionService with full CRUD
✅ Integrated Stripe via stripe-php package (composer require)
✅ Added webhook event handling in StripeWebhookService
✅ Created SubscriptionController with API endpoints
✅ Queued SubscriptionCreated job for async notifications
✅ All new tests pass (24/24)
✅ All existing tests pass (128/128)
✅ Migration rollback verified
✅ Atomic commits (6 total):
   1. "feat: add Subscription model and migration"
   2. "feat: create SubscriptionService"
   3. "feat: integrate Stripe API client"
   4. "feat: add SubscriptionController API endpoints"
   5. "feat: implement webhook event handling"
   6. "feat: add notification jobs and tests"
```

---

### Example 3: Optimize Database Queries (N+1 Fix)

**Your Request:**
```
"I think we have N+1 query problems in the user dashboard, can you analyze and fix?"
```

**RESEARCH Output:**
```
Dashboard queries analyzed:
  • UserController@dashboard calls:
    $users = User::all();
    foreach ($users as $user) {
      $user->subscription     // N queries (1 per user)
      $user->invoices()->count()  // Another N queries
    }

Current query count: 1 + 100 + 100 = 201 queries (with 100 users)

Related routes identified:
  • /dashboard uses this pattern
  • /admin/users also affected
```

**INNOVATE Output:**
```
Approach 1: Add eager loading
  $users = User::with('subscription', 'invoices')->get();
  
  Reduces to: 1 + 1 + 1 = 3 queries
  Pros: Minimal code change, immediate 98% reduction
  Cons: Loads all invoices (could be large)

Approach 2: Use select subqueries for counts
  Select inverse count, add as relationship
  
  Reduces to: 1 query via join
  Pros: Most efficient, single query
  Cons: Requires Model-level query scope

Recommendation: Approach 1 first, then approach 2 for invoices count
```

**PLAN Output:**
```
Changes:
  • UserController@dashboard — Add with('subscription')
  • User model — Add invoices count via withCount()
  • Migration — None

Current query:
  $users = User::all();
  Queries: 1 + 100 + 100 = 201

Optimized query:
  $users = User::with('subscription')
             ->withCount('invoices')
             ->get();
  Queries: 1 + 1 + 1 = 3

Blast Radius:
  • View expects $user->subscription (still works)
  • View expects $user->invoices_count (new, test needed)
  • No database changes
```

**EXECUTE Output:**
```
✅ Updated UserController to use eager loading
✅ Added withCount for invoice count
✅ Dashboard load time: 1200ms → 85ms (93% faster)
✅ All tests pass (156/156)
✅ Verified with Debugbar: 201 queries → 3 queries
✅ Atomic commit: "perf: optimize N+1 queries in user dashboard"
```

---

## Scripts Reference

Each phase can trigger scripts automatically:

### RESEARCH Phase

**`detect-laravel-stack.js`**
```bash
# Auto-runs during RESEARCH
# Outputs: Laravel version, packages, queue driver, testing framework
```

Detects:
- Laravel version (`composer show laravel/framework`)
- Key packages: Spatie, Filament, Livewire, Nova, Sanctum, Passport
- Database type (PostgreSQL/MySQL/SQLite)
- Queue driver (Redis/database/sync)
- Testing framework (PHPUnit/Pest)

**`research-laravel-architecture.js`**
```bash
# Auto-runs during RESEARCH
# Outputs: Middleware stack, providers, events/listeners, routes
```

Maps:
- Middleware execution order
- Service providers (boot/register timing)
- Event/listener bindings
- Route structure and versioning
- API conventions (controller naming, response format)

### PLAN Phase

**`generate-laravel-plan.js`**
```bash
# Auto-runs during PLAN
# Outputs: Models, migrations, services, controllers, tests
```

Generates templates for:
- Eloquent models with relationships
- Reversible migrations with rollback
- Service classes
- Controllers (resource vs custom)
- Test classes (unit + feature)
- Form request classes with validation

### EXECUTE Phase

**`validate-laravel-execution.js`**
```bash
# Auto-runs before EXECUTE commits
# Validates: Migration reversibility, tests pass, linting clean
```

Checks:
- Run migrations forward
- Run migrations backward (rollback)
- Run migrations forward again (re-apply)
- Execute test suite for affected paths
- Run Pint linting (if configured)
- Run Larastan static analysis (if configured)

---

## Integration Points

### Lifecycle Hooks

**Pre-Research Hook: `laravel-stack-detection`**
```
Triggers when: Laravel project detected
Action: Populate with `LARAVEL_VERSION`, `LARAVEL_PACKAGES`
Context available: Stack metadata for RESEARCH phase
```

**Pre-Execution Hook: `laravel-migration-check`**
```
Triggers when: Plan includes database changes
Action: Validate reversibility requirements met
Safety: Prevents non-reversible migrations
```

**Post-Execution Hook: `laravel-context-update`**
```
Triggers when: Feature complete and tested
Action: Index new models, services, routes in context
Memory: Next feature reads updated architecture
```

**Phase Transition Hook: `laravel-blast-radius`**
```
Triggers when: Moving PLAN → EXECUTE
Action: Analyze what could break
Output: Test coverage recommendations
```

### Context Files

After execution, context auto-updates:

```
process/context/laravel/
├── all-laravel.md              # Models, services, routes index
├── models.md                   # Model relationships, scopes
├── services.md                 # Service layer patterns
├── routes.md                   # API structure, middleware
├── migrations-log.md           # Historical migrations + learnings
└── conventions.md              # Team's Laravel patterns
```

---

## Safety Systems

### 1. Migration Reversibility

Every migration is tested:

```bash
# Step 1: Run up
php artisan migrate

# Step 2: Run down (must succeed)
php artisan migrate:rollback

# Step 3: Run up again (must succeed)
php artisan migrate
```

If rollback fails → Plan is blocked with error details.

### 2. Test-Driven Execution

Before each commit, the system:
1. Identifies affected files (modified/created)
2. Maps to test suite (test file discovery)
3. Runs only affected tests
4. Auto-escalates to full suite if >70% of codebase touched

### 3. Query N+1 Detection

If Debugbar installed:
```php
// Flagged during EXECUTE validation
$users->each->comments;  // ❌ N+1 detected

// Suggestion provided
$users->load('comments');  // ✅ Recommended
```

### 4. Secret Leak Prevention

Blocked actions:
- Reading `.env` directly → Asks for explicit confirmation
- Hardcoded API keys in config → Raises security violation
- Database credentials in code → Blocks commit

### 5. Blast Radius Analysis

Plans include impact assessment:
- **Database**: New/altered tables, indices, foreign keys
- **API**: New endpoints, breaking changes, deprecations
- **Queue**: New jobs, event listeners, async patterns
- **Auth**: New permissions, policy changes
- **Dependencies**: New packages, version constraints

---

## Troubleshooting

### "Migration reversibility check failed"

**Error**: `Rollback failed on users_add_subscription_column`

**Cause**: Migration down() doesn't cleanly reverse up()

**Fix**:
```php
// ❌ Bad: Doesn't undo original change
Schema::dropIfExists('users'); // Too aggressive

// ✅ Good: Mirrors up() in reverse
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('subscription_id');
});
```

### "N+1 query detected in dashboard"

**Error**: `201 queries in UserController@dashboard`

**Fix**: Add eager loading in controller or model:
```php
// In UserController
$users = User::with('subscription')->withCount('invoices')->get();

// Or in User model scope
public function scopeWithBillingInfo($query) {
    return $query->with('subscription')->withCount('invoices');
}
```

### "Static analysis failed: Larastan strict mode"

**Error**: `Larastan detected undefined method 'subscriptions'`

**Cause**: Doctrine annotations missing on model

**Fix**:
```php
class User extends Model {
    /**
     * @return HasMany<Subscription>
     */
    public function subscriptions(): HasMany {
        return $this->hasMany(Subscription::class);
    }
}
```

### "Test suite timeout on large migration"

**Error**: `Migration test exceeded 30s timeout`

**Cause**: Data seeding takes too long

**Fix**:
```php
// Use minimal seeding in tests
Schema::table('invoices', function (Blueprint $table) {
    $table->index('user_id');  // Add index for data migration
});

// Then backfill via job, not migration
class BackfillInvoiceData implements ShouldQueue {
    public function handle() {
        // Process in batches
        Invoice::cursor()->each(fn($inv) => $inv->update(...));
    }
}
```

### "Webhook validation failing in tests"

**Error**: `Stripe webhook signature invalid`

**Cause**: Test using real webhook signature, not test key

**Fix**:
```php
// In tests, use test webhook secret
public function test_webhook() {
    $payload = ['type' => 'charge.succeeded', ...];
    $signature = hash_hmac(
        'sha256',
        json_encode($payload),
        config('services.stripe.webhook_secret_test')  // Use test secret
    );
    
    $this->post('/api/webhooks/stripe', $payload, [
        'Stripe-Signature' => $signature
    ])->assertOk();
}
```

---

## Next Steps

1. **In Your First Laravel Project**:
   - Run: `curl -fsSL https://raw.githubusercontent.com/rowjat/ai-vibecode/main/install.sh | bash`
   - Say: `"Add user authentication with email verification"`
   - Watch the full RESEARCH → PLAN → EXECUTE workflow

2. **Customize for Your Team**:
   - Add team patterns to `process/context/laravel/conventions.md`
   - Create company-specific templates in `.claude/skills/vc-laravel-boost/templates/`
   - Document API versioning strategy

3. **Extend the Skill**:
   - Add scripts for your infrastructure (Forge, Nova, custom packages)
   - Create example plans for common features
   - Share learnings back to the context system

---

## References

- [SKILL.md](./SKILL.md) — Core skill definition
- `references/` — Detailed guides (conventions, migrations, testing, architecture)
- `templates/` — Code templates auto-populated in plans
- `examples/` — Real-world feature examples
