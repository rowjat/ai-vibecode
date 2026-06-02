# Laravel Boost Skill

Spec-driven Laravel development harness integrating the full vibecode workflow for PHP Laravel projects.

## Overview

This skill enables Claude to research, plan, and execute Laravel features with architectural intelligence:
- **Framework-aware research** — Stack detection, package auditing, architecture mapping
- **Laravel-specific planning** — Service layer design, migration strategies, testing architecture  
- **Autonomous execution** — Artisan-aware execution, migration safety, database reversibility
- **Quality assurance** — Laravel testing (PHPUnit, Pest), linting (Pint), static analysis (Larastan)

## When This Skill Activates

This skill is auto-discovered when:
- Request contains `laravel`, `artisan`, `eloquent`, `forge`, `nova`, `livewire`, `filament`
- File `composer.json` declares `laravel/framework` dependency
- Project structure matches Laravel conventions (`app/`, `routes/`, `database/migrations/`)

## Quick Usage

### Research Laravel Architecture
```
"Add webhook support to our Laravel API"
→ vc-laravel-research activates
→ Scans for middleware patterns, queue setup, event listeners
→ Checks existing webhook implementations in other services
```

### Plan Service Layer Feature
```
"Create a billing module with subscription management"
→ vc-laravel-plan generates:
  • Service class hierarchy
  • Migration strategy (zero-downtime if needed)
  • Model relationships
  • Event/job requirements
  • Test coverage plan
```

### Execute with Safety
```
"ENTER EXECUTE MODE"
→ Validates migration reversibility
→ Runs tests before committing migrations
→ Checks model relationships for N+1 queries
→ Verifies queue/event bindings
```

## Core Capabilities

### 1. Stack Detection & Analysis

Automatically identifies:
- **Laravel version** (9, 10, 11+) with feature parity
- **Package ecosystem** — Spatie packages, Filament, Livewire, Nova
- **Database setup** — Postgres/MySQL specifics, connection pooling
- **Queue configuration** — Redis/database/sync drivers
- **Authentication** — Sanctum, Passport, Fortify
- **Testing framework** — PHPUnit vs Pest, coverage config

**Script**: `scripts/detect-laravel-stack.js`

### 2. Architecture Research

Maps:
- **Middleware stack** — Authentication, CORS, rate limiting order
- **Service providers** — Boot/register timing, package discovery
- **Event/listener architecture** — Async patterns, queue handling
- **API versioning** — Route grouping, controller namespacing
- **Database patterns** — Query scope conventions, eager loading rules
- **Testing patterns** — Test database setup, factory definitions

**Script**: `scripts/research-laravel-architecture.js`

### 3. Planning & Spec Generation

Creates Laravel-aware plans including:
- **Models & migrations** — With relationships, indexes, foreign keys
- **Controllers & routes** — RESTful naming, resource conventions
- **Service classes** — Business logic isolation
- **Jobs & events** — Async processing strategy
- **Tests** — Unit, feature, and integration test templates
- **Validation rules** — Form request classes with custom rules
- **Database reversibility** — Rollback safety, data preservation

**Script**: `scripts/generate-laravel-plan.js`

### 4. Execution & Validation

Ensures code quality:
- **Migration safety** — Validates reverse migration works
- **Test coverage** — Runs affected tests before commit
- **Query optimization** — Detects N+1 patterns via Debugbar/Clockwork integration
- **Type checking** — Larastan static analysis (if installed)
- **Linting** — PHP-CS-Fixer or Pint formatting
- **Secret detection** — Blocks `.env` leaks, API key exposure

**Script**: `scripts/validate-laravel-execution.js`

## Key Patterns

### Zero-Downtime Migration Pattern
```php
// Step 1: Add new column (nullable)
Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->nullable();
});

// Step 2: Backfill data
// Step 3: Add constraint
// Step 4: Make required in next deployment
```

### Event-Driven Architecture
```php
// Emit event
event(new SubscriptionCreated($subscription));

// Listen & queue
class SendSubscriptionConfirmation implements ShouldQueue {
    public function handle(SubscriptionCreated $event) {...}
}
```

### Repository Pattern (Optional)
```php
// Interface-first design
interface SubscriptionRepository {
    public function findActive(User $user): ?Subscription;
}

// Eloquent implementation
class EloquentSubscriptionRepository implements SubscriptionRepository {...}
```

### Testing Strategy
```php
// Feature test database transactions
class CreateSubscriptionTest extends TestCase {
    public function test_creates_subscription() {
        $response = $this->post('/api/subscriptions', [...]);
        $this->assertDatabaseHas('subscriptions', ...);
    }
}
```

## Integration Points

### Hooks That Trigger
- **Pre-execution** — `laravel-validation-check` ensures migrations are reversible
- **Post-execution** — `laravel-context-update` captures new models, services, routes in context
- **Phase transition** — `laravel-blast-radius` analyzes database/API surface changes

### Lifecycle Integration
- **RESEARCH**: `vc-laravel-research` scans composer.json, app/ structure, recent migrations
- **INNOVATE**: `vc-laravel-innovate` explores design patterns (Service/Repository/Event)
- **PLAN**: `vc-laravel-plan` generates detailed spec with model diagrams, migration strategy
- **EXECUTE**: `vc-laravel-execute` runs with migration safety, test-driven approach
- **VALIDATE**: `vc-laravel-validate` runs test suite, Larastan, Pint
- **UPDATE**: `vc-laravel-update` indexes new models/services in context, updates architecture diagram

## Scripts

### Automated

| Script | Purpose | Trigger |
|--------|---------|---------|
| `detect-laravel-stack.js` | Identifies Laravel version, packages, config | RESEARCH phase |
| `research-laravel-architecture.js` | Maps middleware, providers, patterns | RESEARCH phase |
| `generate-laravel-plan.js` | Creates migration + model + service templates | PLAN phase |
| `validate-laravel-execution.js` | Runs tests, linting, static analysis | EXECUTE → VALIDATE |

### Manual Triggers

```bash
# Analyze current stack
node scripts/detect-laravel-stack.js --analyze

# Generate plan for new feature
node scripts/generate-laravel-plan.js --feature "billing" --type "service"

# Validate execution
node scripts/validate-laravel-execution.js --check-migrations --run-tests
```

## Safety Systems

### Migration Reversibility Check
```bash
php artisan migrate
php artisan migrate:rollback  # Must succeed
php artisan migrate            # Must re-apply cleanly
```

### Database Relationship Validation
- Foreign keys must have corresponding models
- Polymorphic relations validated
- Cascade deletes logged with blast radius

### Query N+1 Detection (Optional)
When Debugbar installed, flags:
```php
// ❌ N+1 Query
$users->each->comments;  // Triggers N queries

// ✅ Eager Loaded
$users->load('comments');
```

### Secret Leak Prevention
- Blocks direct `.env` file reading
- Flags hardcoded config values
- Validates `config/` files don't contain secrets

## File Structure

```
vc-laravel-boost/
├── SKILL.md                                      # This file
├── README.md                                     # Extended usage guide
├── scripts/
│   ├── detect-laravel-stack.js                  # Stack detection
│   ├── research-laravel-architecture.js         # Architecture mapping
│   ├── generate-laravel-plan.js                 # Plan generation
│   ├── validate-laravel-execution.js            # Validation
│   ├── package.json                             # Dependencies
│   └── .env.example                             # Config template
├── templates/
│   ├── migration-template.php
│   ├── model-template.php
│   ├── service-template.php
│   ├── controller-template.php
│   ├── test-feature-template.php
│   ├── test-unit-template.php
│   ├── job-template.php
│   ├── event-template.php
│   └── event-listener-template.php
├── references/
│   ├── laravel-conventions.md                   # PSR-12, Laravel patterns
│   ├── migration-safety-patterns.md             # Zero-downtime strategies
│   ├── testing-strategies.md                    # PHPUnit, Pest, coverage
│   ├── service-architecture-patterns.md         # Service/Repository/DTO
│   ├── api-design-guide.md                      # RESTful, versioning, docs
│   ├── performance-optimization.md              # N+1, caching, queuing
│   ├── deployment-checklist.md                  # Pre-production validation
│   └── laravel-ecosystem.md                     # Popular packages guide
└── examples/
    ├── webhook-integration.md
    ├── subscription-billing.md
    ├── real-time-notifications.md
    ├── multi-tenancy-setup.md
    └── api-versioning.md
```

## Environment Setup

`.env` or `.claude/.env` can configure:

```bash
# Laravel configuration
LARAVEL_VERSION=11                    # Expected version for validation
LARAVEL_TESTING_FRAMEWORK=pest        # pest or phpunit
LARAVEL_QUEUE_DRIVER=redis            # redis, database, sync
LARAVEL_DB_TYPE=postgres              # postgres, mysql, sqlite

# Validation strictness
REQUIRE_MIGRATION_REVERSIBILITY=true
REQUIRE_TEST_COVERAGE=80
REQUIRE_LARASTAN_STRICT=false

# Code quality
REQUIRE_PINT=true                     # PHP code formatting
REQUIRE_STATIC_ANALYSIS=false         # Larastan, PHPStan
```

## Context Integration

After features complete, context updates automatically:

```
process/context/
├── all-context.md              # Includes Laravel stack summary
├── laravel/
│   └── all-laravel.md          # Models, services, routes, conventions
├── tests/
│   └── all-tests.md            # Test factories, commands, coverage
└── database/
    └── all-database.md         # Schema, relationships, indexes
```

## Common Requests

### "Add a new API endpoint"
1. RESEARCH scans existing routes, versioning strategy, authentication
2. INNOVATE explores resource vs action-based design
3. PLAN generates route, controller, form request, test
4. EXECUTE implements with validation, error handling
5. VALIDATE runs feature tests, checks response structure

### "Create background job for email sending"
1. RESEARCH checks queue setup, existing jobs, event patterns
2. INNOVATE explores queuing strategy (Redis/database), retries
3. PLAN generates job class, dispatch locations, failure handling
4. EXECUTE implements with monitoring, dead-letter handling
5. VALIDATE tests job execution, retry logic, failure cases

### "Add multi-tenancy to existing app"
1. RESEARCH scans current data model, customer isolation strategy
2. INNOVATE explores tenant scoping (middleware vs model), isolation level
3. PLAN generates migration, tenant model, scoping middleware, tests
4. EXECUTE implements with data segregation, blast radius analysis
5. VALIDATE tests cross-tenant query prevention, backup/restore

### "Optimize database queries (N+1 detection)"
1. RESEARCH profiles existing queries, identifies hot paths
2. INNOVATE explores eager loading, caching, query aggregation
3. PLAN lists specific changes with before/after queries
4. EXECUTE adds relationship loading, query scope optimization
5. VALIDATE measures query count reduction, runs benchmarks

## Contributing

To extend Laravel Boost:
1. Add script to `scripts/` (executable Node.js file)
2. Create reference doc if pattern not covered
3. Add template if new artifact type
4. Update this SKILL.md with trigger keywords

## References

- [Laravel Official Docs](https://laravel.com/docs)
- [Laravel Best Practices Repo](https://github.com/alexeymezenin/laravel-best-practices)
- [Spatie Laravel Packages](https://spatie.be/open-source?search=laravel)
- [Laracasts](https://laracasts.com)
