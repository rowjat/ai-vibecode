# Laravel Conventions & Best Practices

Guidelines that guide Laravel Boost planning and code generation.

## Directory Structure

```
laravel-project/
├── app/
│   ├── Models/
│   ├── Http/
│   │   ├── Controllers/
│   │   ├── Requests/
│   │   ├── Resources/
│   │   └── Middleware/
│   ├── Services/
│   ├── Jobs/
│   ├── Events/
│   ├── Listeners/
│   ├── Mail/
│   └── Observers/
├── routes/
│   ├── api.php
│   └── web.php
├── database/
│   ├── migrations/
│   ├── factories/
│   └── seeders/
├── resources/
│   └── views/
├── tests/
│   ├── Feature/
│   └── Unit/
├── config/
├── storage/
├── bootstrap/
└── composer.json
```

## Naming Conventions

| Artifact | Convention | Example |
|----------|-----------|---------|
| Model | Singular, PascalCase | `User`, `Subscription`, `Invoice` |
| Migration | snake_case, descriptive | `create_users_table`, `add_subscription_id_to_invoices` |
| Controller | PascalCase, append Controller | `UserController`, `SubscriptionController` |
| Service | PascalCase, append Service | `SubscriptionService`, `PaymentService` |
| Job | PascalCase, append Job | `SendInvoiceEmail`, `ProcessRefund` |
| Event | PascalCase | `SubscriptionCreated`, `PaymentFailed` |
| Listener | PascalCase, append Listener | `SendSubscriptionConfirmation` |
| Form Request | PascalCase, append Request | `StoreSubscriptionRequest`, `UpdateUserRequest` |
| Test | PascalCase, append Test | `UserControllerTest`, `SubscriptionServiceTest` |
| Database Seeder | PascalCase, append Seeder | `UserSeeder`, `SubscriptionSeeder` |
| Middleware | PascalCase, append Middleware | `VerifyApiToken`, `EnsureJsonResponse` |
| Mail | PascalCase, append Mail | `InvoiceMail`, `SubscriptionConfirmationMail` |
| Policy | PascalCase, append Policy | `UserPolicy`, `SubscriptionPolicy` |

## Database Conventions

### Table Names
- Plural, snake_case: `users`, `subscriptions`, `invoices`
- Pivot tables: `{singular1}_{singular2}` (alphabetical) — `role_user`

### Column Names
- snake_case: `user_id`, `subscription_id`, `created_at`
- Boolean prefix with `is_` or `has_`: `is_active`, `has_payment`
- Foreign keys: `{table_singular}_id` — `user_id`, `subscription_id`
- Timestamps: Always include `created_at`, `updated_at`
- Soft deletes: Add `deleted_at` if needed

### Indexes
- Foreign keys always indexed
- Frequently queried columns (status, email): indexed
- Index naming: `{table}_{columns}_index`

```php
Schema::create('subscriptions', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->string('stripe_id')->unique();
    $table->string('status'); // active, cancelled, expired
    $table->timestamp('current_period_end')->nullable();
    $table->timestamps();
    
    $table->index(['user_id', 'status']);
    $table->index('current_period_end');
});
```

## Model Conventions

### Relationships
```php
class User extends Model {
    // One-to-many
    public function subscriptions(): HasMany {
        return $this->hasMany(Subscription::class);
    }
    
    // Many-to-many
    public function roles(): BelongsToMany {
        return $this->belongsToMany(Role::class);
    }
}
```

### Casting & Attributes
```php
class Subscription extends Model {
    protected $casts = [
        'current_period_end' => 'datetime',
        'is_active' => 'boolean',
    ];
    
    // Accessor for display
    public function getStatusBadgeAttribute(): string {
        return match($this->status) {
            'active' => 'success',
            'cancelled' => 'danger',
            default => 'secondary',
        };
    }
}
```

### Scopes
```php
class Subscription extends Model {
    public function scopeActive($query): Builder {
        return $query->where('status', 'active')
                     ->where('current_period_end', '>', now());
    }
    
    public function scopeForUser($query, User $user): Builder {
        return $query->where('user_id', $user->id);
    }
}

// Usage
Subscription::active()->forUser($user)->get();
```

## Controller Conventions

### Resource Controller
```php
class SubscriptionController extends Controller {
    // List all
    public function index() {}
    
    // Show form for create
    public function create() {}
    
    // Store new
    public function store(StoreSubscriptionRequest $request) {}
    
    // Show specific
    public function show(Subscription $subscription) {}
    
    // Show edit form
    public function edit(Subscription $subscription) {}
    
    // Update
    public function update(UpdateSubscriptionRequest $request, Subscription $subscription) {}
    
    // Delete
    public function destroy(Subscription $subscription) {}
}
```

### API Controller (JSON only)
```php
class SubscriptionController extends Controller {
    // List all
    public function index() {
        return SubscriptionResource::collection(
            Subscription::paginate()
        );
    }
    
    // Store
    public function store(StoreSubscriptionRequest $request) {
        $subscription = Subscription::create($request->validated());
        return new SubscriptionResource($subscription);
    }
    
    // Update
    public function update(UpdateSubscriptionRequest $request, Subscription $subscription) {
        $subscription->update($request->validated());
        return new SubscriptionResource($subscription);
    }
    
    // Delete
    public function destroy(Subscription $subscription) {
        $subscription->delete();
        return response()->json(null, 204);
    }
}
```

## Service Layer Pattern

Isolate business logic from controllers:

```php
class SubscriptionService {
    public function __construct(
        private StripeService $stripe,
        private NotificationService $notification,
    ) {}
    
    public function create(User $user, array $data): Subscription {
        // Validate
        // Create with Stripe
        // Emit event
        // Return result
    }
    
    public function cancel(Subscription $subscription): void {
        // Cancel with Stripe
        // Update local record
        // Emit event
        // Send notification
    }
}

// In controller
public function store(StoreSubscriptionRequest $request) {
    $subscription = $this->subscriptionService->create(
        Auth::user(),
        $request->validated()
    );
    return new SubscriptionResource($subscription);
}
```

## Events & Jobs Pattern

Async processing for side effects:

```php
// Emit from service
event(new SubscriptionCreated($subscription));

// Listen & queue
class SendSubscriptionConfirmation implements ShouldQueue {
    use Queueable;
    
    public function __construct(private Subscription $subscription) {}
    
    public function handle(): void {
        Mail::send(new SubscriptionConfirmationMail($this->subscription));
    }
}
```

## Testing Conventions

### Feature Tests (Integration)
```php
class SubscriptionControllerTest extends TestCase {
    use RefreshDatabase; // Rollback after each test
    
    public function test_can_create_subscription() {
        $user = User::factory()->create();
        
        $response = $this->actingAs($user)
            ->post('/api/subscriptions', [...]);
        
        $response->assertCreated();
        $this->assertDatabaseHas('subscriptions', [...]);
    }
}
```

### Unit Tests
```php
class SubscriptionServiceTest extends TestCase {
    public function test_creates_subscription_with_stripe() {
        $service = new SubscriptionService(
            $stripe = Mockery::mock(StripeService::class),
        );
        
        $stripe->shouldReceive('createSubscription')
               ->once()
               ->andReturn(['id' => 'sub_123']);
        
        $result = $service->create($user, $data);
        
        $this->assertNotNull($result->id);
    }
}
```

## API Response Format

**Success (200)**
```json
{
    "data": {
        "id": 1,
        "user_id": 5,
        "status": "active",
        "created_at": "2024-01-15T10:30:00Z"
    }
}
```

**Paginated (200)**
```json
{
    "data": [...],
    "meta": {
        "current_page": 1,
        "total": 50,
        "per_page": 15
    }
}
```

**Error (4xx/5xx)**
```json
{
    "message": "Subscription not found",
    "errors": {
        "subscription_id": ["The subscription does not exist"]
    }
}
```

## Migration Patterns

### Adding Columns (Zero-Downtime)
```php
// Step 1: Add nullable column
Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->nullable();
});

// Step 2: Backfill in separate migration or job
// Step 3: Make required in separate migration
Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->change(); // Make required
});
```

### Removing Columns (Zero-Downtime)
```php
// Step 1: Stop using column in code
// Step 2: Remove column after deployment
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('legacy_field');
});
```

### Renaming Columns (Zero-Downtime)
```php
// Step 1: Create new column, copy data
Schema::table('users', function (Blueprint $table) {
    $table->string('phone_number')->nullable();
});
// Run: UPDATE users SET phone_number = phone;

// Step 2: Update code to use new column
// Step 3: Drop old column
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('phone');
});
```

## Queue Job Patterns

### Async Email
```php
class SendInvoiceEmail implements ShouldQueue {
    use Queueable;
    public $tries = 3;
    public $backoff = [60, 300, 900]; // 1m, 5m, 15m
    
    public function handle(): void {
        Mail::send(new InvoiceMail($this->invoice));
    }
    
    public function failed(Throwable $exception): void {
        // Alert team if fails after retries
    }
}
```

### Scheduled Task
```php
// In app/Console/Kernel.php
protected function schedule(Schedule $schedule) {
    $schedule->job(new ProcessMonthlyBilling)
             ->monthlyOn(1, '00:00');
}
```

## Security Conventions

### Authentication
- Always use `Auth::user()` in controllers (not `request()->user()` in services)
- Use policies for authorization: `$this->authorize('update', $subscription)`

### Input Validation
- Use form request classes, not inline validation
- Validate in `authorize()` and `rules()` methods

### Database
- Use soft deletes if data shouldn't be permanently removed
- Use foreign keys with cascading deletes where appropriate

### API
- Always use HTTPS in production
- Require API tokens (Sanctum or Passport)
- Rate limit: `throttle:60,1` (60 requests per minute)

---

## Related Guides

- [Migration Safety Patterns](./migration-safety-patterns.md)
- [Testing Strategies](./testing-strategies.md)
- [Service Architecture Patterns](./service-architecture-patterns.md)
