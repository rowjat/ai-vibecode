# Laravel Modules Architecture

Modular Laravel applications with nwidart/laravel-modules for scalable, organized codebases.

## Overview

Laravel Modules enables building large applications as a collection of independent, self-contained modules:

```
app/Modules/
├── Billing/
│   ├── Models/
│   ├── Services/
│   ├── Http/
│   │   ├── Controllers/
│   │   ├── Requests/
│   │   └── Resources/
│   ├── Database/
│   │   ├── Migrations/
│   │   └── Factories/
│   ├── Routes/
│   │   └── api.php
│   ├── Tests/
│   ├── Events/
│   ├── Jobs/
│   └── module.json
├── Notifications/
├── Analytics/
└── Reporting/
```

## Installation

```bash
composer require nwidart/laravel-modules

php artisan module:make Billing
```

## Module Structure

Each module is self-contained with:

### `module.json`
```json
{
  "name": "Billing",
  "alias": "billing",
  "description": "Subscription billing and invoice management",
  "keywords": ["billing", "subscriptions", "invoices"],
  "priority": 0,
  "providers": [
    "Modules\\Billing\\Providers\\BillingServiceProvider"
  ],
  "aliases": {},
  "files": [],
  "requires": []
}
```

### Module Service Provider

```php
// Modules/Billing/Providers/BillingServiceProvider.php
namespace Modules\Billing\Providers;

use Illuminate\Support\ServiceProvider;

class BillingServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton(
            'Modules\Billing\Services\SubscriptionService',
            'Modules\Billing\Services\SubscriptionService'
        );
    }

    public function boot(): void
    {
        $this->loadMigrationsFrom(__DIR__ . '/../../Database/Migrations');
        $this->loadRoutesFrom(__DIR__ . '/../../Routes/api.php');
        $this->loadViewsFrom(__DIR__ . '/../../Resources/Views', 'billing');
    }
}
```

## Module Organization Patterns

### 1. Service-Per-Module

Each module owns one business domain:

```
Billing/ — Subscriptions, invoices, payment processing
├── Models/Subscription, Invoice, Payment
├── Services/SubscriptionService, InvoiceService
├── Events/SubscriptionCreated, PaymentProcessed
├── Jobs/ProcessMonthlyBilling, SendInvoice
└── Http/Controllers/SubscriptionController

Notifications/ — All notification delivery
├── Models/Notification, NotificationTemplate
├── Services/NotificationService
├── Jobs/SendEmailNotification, SendSmsNotification
└── Http/Controllers/NotificationController
```

### 2. Feature-Per-Module

Each module implements a distinct feature:

```
Authentication/ — User auth, two-factor, sessions
├── Models/User, TwoFactorCode
├── Services/AuthenticationService
├── Http/Controllers/LoginController, RegisterController
└── Routes/auth.php

MultiTenancy/ — Tenant management, isolation
├── Models/Tenant, TenantUser
├── Middleware/ResolveTenant
├── Http/Controllers/TenantController
└── Database/Migrations/create_tenants_table.php

Reporting/ — Analytics, reports, dashboards
├── Models/Report, ReportSchedule
├── Services/ReportGenerator
├── Http/Controllers/ReportController
└── Jobs/GenerateMonthlyReport
```

### 3. Layered Architecture Per Module

```
Billing/
├── Http/
│   ├── Controllers/SubscriptionController.php
│   ├── Requests/StoreSubscriptionRequest.php
│   └── Resources/SubscriptionResource.php
├── Services/
│   ├── SubscriptionService.php
│   ├── StripeIntegrationService.php
│   └── InvoiceService.php
├── Repositories/
│   ├── SubscriptionRepository.php
│   └── InvoiceRepository.php
├── Models/
│   ├── Subscription.php
│   └── Invoice.php
├── Events/
│   └── SubscriptionCreated.php
├── Listeners/
│   └── SendSubscriptionEmail.php
├── Jobs/
│   └── ProcessMonthlyBilling.php
├── Database/
│   └── Migrations/
└── Tests/
    ├── Feature/
    └── Unit/
```

## Communicating Between Modules

### Via Events (Recommended)

```php
// Billing module emits
event(new Modules\Billing\Events\SubscriptionCreated($subscription));

// Notifications module listens
class Modules\Notifications\Listeners\SendSubscriptionConfirmation {
    public function handle(SubscriptionCreated $event) {
        // Send notification
    }
}
```

### Via Service Injection

```php
// In Billing module controller
use Modules\Notifications\Services\NotificationService;

class SubscriptionController {
    public function store(StoreSubscriptionRequest $request, NotificationService $notifications) {
        $subscription = Subscription::create($request->validated());
        
        // Call another module's service
        $notifications->send('subscription_created', $subscription);
        
        return new SubscriptionResource($subscription);
    }
}
```

### Via Contracts (Interface-Based)

```php
// In Notifications module
namespace Modules\Notifications\Contracts;

interface NotificationChannelContract {
    public function send(string $template, array $data): void;
}

// In Billing module — use the interface
use Modules\Notifications\Contracts\NotificationChannelContract;

class SubscriptionService {
    public function __construct(private NotificationChannelContract $notifications) {}
    
    public function create(array $data) {
        $subscription = Subscription::create($data);
        $this->notifications->send('subscription_created', $subscription);
        return $subscription;
    }
}
```

## Routes Per Module

### API Routes
```php
// Modules/Billing/Routes/api.php
Route::prefix('api/v1')->middleware(['auth:sanctum'])->group(function () {
    Route::apiResource('subscriptions', 'SubscriptionController');
    Route::post('subscriptions/{id}/cancel', 'SubscriptionController@cancel');
    Route::apiResource('invoices', 'InvoiceController');
});
```

### Web Routes
```php
// Modules/Billing/Routes/web.php
Route::middleware(['auth', 'admin'])->group(function () {
    Route::resource('billing/subscriptions', 'Admin\SubscriptionController');
    Route::resource('billing/invoices', 'Admin\InvoiceController');
});
```

## Migrations Per Module

```bash
# Generate migration within module
php artisan module:make-migration create_subscriptions_table --module=Billing

# Run all module migrations
php artisan module:migrate

# Rollback module migrations
php artisan module:migrate-rollback
```

```php
// Modules/Billing/Database/Migrations/2024_01_15_create_subscriptions_table.php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateSubscriptionsTable extends Migration
{
    public function up(): void
    {
        Schema::create('subscriptions', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->string('stripe_id')->unique();
            $table->enum('status', ['active', 'cancelled', 'expired']);
            $table->timestamp('current_period_end')->nullable();
            $table->timestamps();
            $table->index(['user_id', 'status']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('subscriptions');
    }
}
```

## Testing Modules

```php
// Modules/Billing/Tests/Feature/SubscriptionControllerTest.php
namespace Modules\Billing\Tests\Feature;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Modules\Billing\Models\Subscription;

class SubscriptionControllerTest extends TestCase
{
    use RefreshDatabase;

    public function test_can_create_subscription(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)
            ->postJson('/api/v1/subscriptions', [
                'plan' => 'premium',
            ]);

        $response->assertCreated();
        $this->assertDatabaseHas('subscriptions', [
            'user_id' => $user->id,
            'plan' => 'premium',
        ]);
    }
}
```

## Module Dependencies

Declare module dependencies in `module.json`:

```json
{
  "name": "Billing",
  "requires": [
    "Notifications",
    "Reporting"
  ]
}
```

This ensures:
- Required modules load before dependent module
- Provides clear dependency graph
- Fails fast if dependency missing

## Module Loading & Registration

### Enable/Disable Modules

```bash
# Disable module (won't load)
php artisan module:disable Billing

# Enable module (will load)
php artisan module:enable Billing

# Check status
php artisan module:list
```

### Lazy Loading

For performance, modules can be lazy-loaded:

```php
// Modules/Billing/module.json
{
  "name": "Billing",
  "lazy": true  // Only load when explicitly needed
}
```

```php
// Access lazy module
$subscription = app('Modules\Billing\Models\Subscription');
```

## Publishing Assets

Each module can publish assets (migrations, config, views):

```bash
# Publish all module assets
php artisan vendor:publish --provider="Modules\\Billing\\Providers\\BillingServiceProvider"

# Publish specific
php artisan vendor:publish --provider="Modules\\Billing\\Providers\\BillingServiceProvider" --tag="migrations"
```

## Module Namespacing

Each module has its own namespace:

```php
// Models
namespace Modules\Billing\Models;
class Subscription { }

// Services
namespace Modules\Billing\Services;
class SubscriptionService { }

// Controllers
namespace Modules\Billing\Http\Controllers;
class SubscriptionController { }

// Events
namespace Modules\Billing\Events;
class SubscriptionCreated { }
```

## Monorepo Organization

For larger applications with 10+ modules:

```
app/Modules/
├── Core/                 # Shared utilities, base classes
│   ├── Traits/
│   ├── Contracts/
│   └── Exceptions/
├── Infrastructure/       # Modules providing infrastructure
│   ├── Authentication/
│   ├── Authorization/
│   └── MultiTenancy/
├── Domain/              # Business domain modules
│   ├── Billing/
│   ├── Reporting/
│   └── Analytics/
├── Integration/         # Third-party integrations
│   ├── Stripe/
│   ├── Slack/
│   └── Segment/
└── Admin/              # Admin/internal tools
    ├── Dashboard/
    └── Users/
```

## Best Practices

### 1. Single Responsibility
Each module owns one business domain. Don't mix concerns.

### 2. Explicit Dependencies
Use contracts and service injection. Don't reach into other modules directly.

### 3. Event-Driven Communication
Emit events for cross-module actions. Decouple via events.

### 4. Test Each Module
Write tests within the module. `Modules/Billing/Tests/` contains billing tests.

### 5. Clear Public API
Define what's public vs internal:
- Public: Models, Services, Events, Contracts
- Internal: Repositories, Traits, Helpers

### 6. Migration Isolation
Each module has its own migrations directory. Easy to extract/move.

### 7. Documentation
Each module has README.md explaining:
- What it does
- How to use it
- Dependencies
- Examples

## Extracting a Module

Modular architecture makes it easy to extract a module:

```
1. Module owns all its code (Models, Services, Migrations)
2. Public API via Contracts/Services
3. Tests validate it works independently
4. Can publish to separate package easily

# Publish Billing as separate package:
mv Modules/Billing/ packages/billing/
composer require ./packages/billing
```

## Module Checklist

Before considering a module complete:

- [ ] Models with relationships defined
- [ ] Migrations with proper reversibility
- [ ] Service layer with business logic
- [ ] HTTP layer (Controllers, Resources, Requests)
- [ ] Events for cross-module communication
- [ ] Tests (unit + feature, 80%+ coverage)
- [ ] Routes documented
- [ ] Dependencies declared in module.json
- [ ] README.md with usage examples
- [ ] Database indexes on foreign keys
