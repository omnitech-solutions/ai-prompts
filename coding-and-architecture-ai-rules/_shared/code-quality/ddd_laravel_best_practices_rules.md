# 🍁 Laravel + Domain‑Driven Design Conventions
**Path:** `.cursor/rules/core/laravel-conventions.mdc`

Guidelines for integrating Laravel with a **DDD + layered architecture**, keeping framework concerns isolated in Infrastructure while preserving domain purity.

---
## 🧩 Architectural Boundary
```
Domain  ←  Application  ←  Infrastructure (Laravel)
```
✔ Domain / Application layers must remain framework‑agnostic.
✔ All Laravel code lives in **Infrastructure**.
✔ Infrastructure adapts HTTP, persistence, queues → Domain.

> The Domain must not know Laravel exists.

---
## 🏗️ Service Provider Pattern
Each module/package exposes `{Module}ServiceProvider` responsible only for:
- route + migration loading
- repository interface to implementation binding

```php
final class SubscriptionServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        $this->loadRoutesFrom(__DIR__.'/routes/api.php');
        $this->loadMigrationsFrom(__DIR__.'/database/migrations');
    }

    public function register(): void
    {
        $this->app->bind(
            SubscriptionOfferingRepositoryInterface::class,
            SubscriptionOfferingEloquentRepository::class,
        );
    }
}
```

---
## 🗄️ Database & Migrations
- Primary key for aggregates: `uuid('id')`
- External identifiers stored as `external_id` (unique)
- Soft deletes for user‑owned aggregates
- Index aggressively on filterable columns

```php
Schema::create('subscription_offerings', function (Blueprint $table) {
    $table->uuid('id')->primary();
    $table->string('external_id')->unique();
    $table->unsignedBigInteger('team_id')->index();
    $table->string('status');
    $table->json('items');
    $table->timestamps();
    $table->softDeletes();
});
```

---
## 🏛️ Eloquent Model (Infrastructure Adapter)
Location: `Infrastructure/Persistence/Models/{Aggregate}Eloquent.php`

```php
final class SubscriptionOfferingEloquent extends Model
{
    use SoftDeletes;

    protected $table = 'subscription_offerings';
    protected $keyType = 'string';
    public $incrementing = false;

    protected $guarded = ['id'];

    protected $casts = [
        'items' => 'array',
        'created_at' => 'datetime',
        'updated_at' => 'datetime',
    ];
}
```

> Eloquent models **must not** contain business logic.

---
## 🔌 Repositories (Domain ↔ Persistence Mapping)
Location: `Infrastructure/Persistence/Repositories/{Aggregate}EloquentRepository.php`

```php
final class SubscriptionOfferingEloquentRepository
    implements SubscriptionOfferingRepositoryInterface
{
    public function findOneByExternalIdOrFail(SubscriptionOfferingExternalId $id): SubscriptionOffering
    {
        $model = SubscriptionOfferingEloquent::where('external_id', $id->toString())->firstOrFail();
        return $this->toDomain($model);
    }

    public function store(SubscriptionOffering $offering): SubscriptionOffering
    {
        $model = SubscriptionOfferingEloquent::updateOrCreate(
            ['external_id' => $offering->externalId()->toString()],
            $this->fromDomain($offering),
        );

        return $this->toDomain($model);
    }
}
```

> The Domain defines the repository **interface**; Infrastructure defines the **implementation**.

---
## 🌐 HTTP Controllers
Controllers must:
- delegate to **Application Handlers**
- contain no domain logic
- construct Commands from input

```php
final class SubscriptionOfferingController extends Controller
{
    public function updateItems(Request $request, string $externalId): JsonResponse
    {
        $command = UpdateSubscriptionOfferingItemsCommand::fromRequest($request, $externalId);
        $result = $this->handler->handle($command);
        return SubscriptionOfferingResource::make($result)->response();
    }
}
```

> Controllers should be **thin** → ideally < **15 lines**.

---
## 🔄 Request → Command Mapping
Move validation into Command constructors (not FormRequest).

```php
public static function fromRequest(Request $request, string $externalId): self
{
    $validated = $request->validate([
        'items' => 'required|array',
        'items.*.item_id' => 'required|string',
        'items.*.quantity' => 'required|integer|min:1',
    ]);

    return new self(
        externalId: SubscriptionOfferingExternalId::fromString($externalId),
        userId: UserId::fromString($request->user()->id),
        items: $validated['items'],
    );
}
```

---
## 📦 API Resources
Location: `Infrastructure/Http/Resources/{Aggregate}Resource.php`

```php
final class SubscriptionOfferingResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->externalId,
            'team_id' => $this->teamId,
            'status' => $this->status,
            'items' => $this->items,
            'created_at' => $this->createdAt,
        ];
    }
}
```

---
## 🧵 Jobs, Listeners & Events
- Listeners remain synchronous
- Jobs perform async work
- Jobs **delegate to handlers**, not contain logic

```php
final class ProcessOfferingActivationJob implements ShouldQueue
{
    public function handle(ProcessOfferingActivationCommandHandler $handler): void
    {
        $handler->handle(new ProcessOfferingActivationCommand($this->externalId));
    }
}
```

---
## 🧪 Testing
✔ **Unit:** test Domain without Laravel
✔ **Feature:** full HTTP + adapter mapping
✔ Use factories + `@group` tags

---
## 🚫 Anti‑Patterns
| Don’t | Instead |
|-------|---------|
| Business logic inside Eloquent models | Domain Entity owns behavior |
| Calling DB directly from handlers | Use Repository interface |
| FormRequests | Validate in Command/DTO |
| Fat controllers | Controller → Handler → Domain |

---
## ✅ Quick Checklist
- Model contains **no domain logic**
- Handler performs orchestration only
- Repository maps Domain ↔ persistence model
- Validation occurs in Command/DTO
- Controller < 15 lines, delegates to Handler

---
> Laravel is the delivery mechanism — **not the architecture.**
