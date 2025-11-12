# Package Structure — AI Development Rules

**Purpose:** Define authoritative layout and naming conventions for packages in a monorepo. Mirror layered DDD architecture and keep domain boundaries explicit.

---

## 🎯 Goals

- Clear separation of **Domain**, **Application**, and **Infrastructure** layers
- Predictable paths and file naming for fast navigation
- Enable testability and code generation alignment
- Support multiple language ecosystems (Ruby, PHP, TypeScript)

---

## 📁 Standard Package Layout

### Generic Structure
```
packages/{PackageName}/
├── Domain/
│   ├── Aggregates/
│   │   └── {Aggregate}/
│   │       ├── {Aggregate}.{rb,php,ts}              # Aggregate root
│   │       └── {Aggregate}Repository.{rb,php,ts}    # Repository interface
│   ├── ValueObjects/
│   │   └── {ValueObject}.{rb,php,ts}                # Immutable domain types
│   ├── Events/
│   │   ├── Base{Aggregate}Event.{rb,php,ts}
│   │   └── {Aggregate}{Action}.{rb,php,ts}          # e.g., InvoiceSent
│   ├── Policies/
│   │   └── {Aggregate}Policy.{rb,php,ts}            # Authorization rules
│   ├── Services/
│   │   └── {Aggregate}Service.{rb,php,ts}           # Pure domain services
│   ├── DataTransferObjects/                         # Internal DTOs
│   │   └── {Concept}DTO.{rb,php,ts}
│   ├── Exceptions/
│   │   └── {Aggregate}Exception.{rb,php,ts}
│   └── README.md                                     # Domain glossary & invariants
│
├── Application/
│   ├── Commands/
│   │   └── {Aggregate}{Action}/
│   │       ├── {Aggregate}{Action}Command.{rb,php,ts}
│   │       ├── {Aggregate}{Action}Handler.{rb,php,ts}
│   │       └── {Aggregate}{Action}Result.{rb,php,ts}
│   └── Queries/
│       └── {Aggregate}{View}/
│           ├── {Aggregate}{View}Query.{rb,php,ts}
│           └── {Aggregate}{View}Handler.{rb,php,ts}
│
├── Infrastructure/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   └── {Aggregate}Controller.{rb,php,ts}
│   │   ├── Resources/                               # HTTP response DTOs
│   │   │   └── {Aggregate}Resource.{rb,php,ts}
│   │   └── routes.{rb,php,ts}                       # Route definitions
│   ├── Persistence/
│   │   ├── Models/
│   │   │   └── {Aggregate}Model.{rb,php,ts}         # ORM model
│   │   ├── Repositories/
│   │   │   └── {Aggregate}Repository.{rb,php,ts}    # Repository implementation
│   │   └── Migrations/
│   │       └── {timestamp}_create_{table}.{rb,sql}
│   ├── Jobs/
│   │   └── {Action}Job.{rb,php,ts}                  # Async workers
│   ├── Listeners/
│   │   └── {Event}Listener.{rb,php,ts}              # Event subscribers
│   ├── Integration/                                 # Cross-package APIs
│   │   ├── DataTransferObjects/
│   │   │   └── {External}DTO.{rb,php,ts}
│   │   └── {Aggregate}Api.{rb,php,ts}               # Integration surface
│   └── Providers/                                   # Service registration
│       └── {Package}ServiceProvider.{rb,php,ts}
│
├── tests/
│   ├── Unit/                                        # Pure domain tests
│   │   └── Domain/
│   └── Integration/                                 # HTTP + adapter tests
│       └── Infrastructure/
│
├── config/                                          # Package-specific config
└── README.md                                        # Package overview
```

---

## 🏷️ Naming Conventions

### Domain Layer
| Component | Pattern | Example |
|-----------|---------|---------|
| **Aggregate Root** | `{Aggregate}.{ext}` | `Subscription.php`, `WorkOrder.rb` |
| **Value Object** | `{Concept}.{ext}` | `EmailAddress.ts`, `Money.php` |
| **Domain Event** | `{Aggregate}{Action}.{ext}` | `InvoiceSent.php`, `OrderPlaced.rb` |
| **Policy** | `{Aggregate}Policy.{ext}` | `WorkOrderPolicy.ts` |
| **Repository Interface** | `{Aggregate}Repository.{ext}` | `SubscriptionRepository.php` |
| **Domain Service** | `{Aggregate}Service.{ext}` | `PricingService.rb` |
| **Internal DTO** | `{Concept}DTO.{ext}` | `InvoiceLineItemDTO.ts` |

### Application Layer
| Component | Pattern | Example |
|-----------|---------|---------|
| **Command** | `{Aggregate}{Action}Command.{ext}` | `CreateInvoiceCommand.php` |
| **Command Handler** | `{Aggregate}{Action}Handler.{ext}` | `CreateInvoiceHandler.rb` |
| **Query** | `{Aggregate}{View}Query.{ext}` | `InvoiceDetailsQuery.ts` |
| **Query Handler** | `{Aggregate}{View}Handler.{ext}` | `InvoiceDetailsHandler.php` |
| **Result DTO** | `{Aggregate}{Action}Result.{ext}` | `CreateInvoiceResult.rb` |

### Infrastructure Layer
| Component | Pattern | Example |
|-----------|---------|---------|
| **ORM Model** | `{Aggregate}Model.{ext}` | `SubscriptionModel.php`, `WorkOrderModel.rb` |
| **Repository Impl** | `{Aggregate}Repository.{ext}` | `SubscriptionRepository.ts` (impl) |
| **Controller** | `{Aggregate}Controller.{ext}` | `InvoiceController.php` |
| **Resource/Serializer** | `{Aggregate}Resource.{ext}` | `InvoiceResource.rb` |
| **Job/Worker** | `{Action}Job.{ext}` | `SendWelcomeEmailJob.ts` |
| **Listener/Subscriber** | `{Event}Listener.{ext}` | `InvoiceSentListener.php` |
| **Integration API** | `{Aggregate}Api.{ext}` | `SubscriptionApi.rb` |
| **External DTO** | `{External}{View}DTO.{ext}` | `StripeCustomerDTO.ts` |

---

## 📋 Layer Placement Rules

### Domain Layer Files
**Location:** `packages/{PackageName}/Domain/`

**What Goes Here:**
- Aggregate roots and entities
- Value objects (immutable, validated)
- Domain events (state change records)
- Repository **interfaces** (NOT implementations)
- Domain services (stateless business logic)
- Policies (authorization rules)
- Internal DTOs (domain concepts)
- Domain exceptions

**Must NOT Include:**
- Framework imports (Rails, Laravel, Express, NestJS)
- ORM models or persistence code
- HTTP or routing concerns
- External API clients
- Async job definitions

---

### Application Layer Files
**Location:** `packages/{PackageName}/Application/`

**What Goes Here:**
- Commands (write operations with validation)
- Queries (read operations)
- Handlers (use-case orchestration)
- Result DTOs (handler return types)
- Transaction boundary management
- Cross-aggregate coordination

**Must NOT Include:**
- Controllers or HTTP request/response handling
- ORM models or direct persistence
- Business rules (those belong in **Domain**)
- Job/Worker definitions

---

### Infrastructure Layer Files
**Location:** `packages/{PackageName}/Infrastructure/`

**What Goes Here:**
- Controllers (thin HTTP adapters)
- ORM models (ActiveRecord, Eloquent, TypeORM)
- Repository implementations
- Resources/Serializers (HTTP response formatting)
- Jobs/Workers (async execution)
- Listeners/Subscribers (event handlers)
- Integration APIs (cross-package communication)
- External DTOs (API contracts)
- Service providers/registration
- Route definitions
- Migrations

**Must NOT Include:**
- Business logic or domain rules
- Use-case orchestration (that's **Application**)

---

## 🔗 Integration Patterns

### Cross-Package Communication

**Integration API Location:** `Infrastructure/Integration/{Aggregate}Api.{rb,php,ts}`

**Purpose:**
- Single entry point per package
- Synchronous communication between bounded contexts
- Exposes controlled surface for other packages

**Contract Rules:**
- Accept/return **External DTOs** only (NEVER internal aggregates)
- NO business rules (delegate to **Application** handlers)
- NO authorization (handled by calling package)

**Pattern:**
```ruby
# Infrastructure/Integration/SubscriptionApi.{rb,php,ts}
class SubscriptionApi
  def get_active_subscription(user_id)
    # Convert to internal query
    query = GetActiveSubscriptionQuery.new(user_id)
    result = @query_handler.handle(query)
    
    # Map to external DTO
    SubscriptionProfileDTO.from_domain(result)
  end
end
```

```php
// Infrastructure/Integration/SubscriptionApi.{rb,php,ts}
final class SubscriptionApi {
    public function getActiveSubscription(string $userId): SubscriptionProfileDTO {
        $query = new GetActiveSubscriptionQuery($userId);
        $result = $this->queryHandler->handle($query);
        return SubscriptionProfileDTO::fromDomain($result);
    }
}
```

```typescript
// Infrastructure/Integration/SubscriptionApi.{rb,php,ts}
export class SubscriptionApi {
  async getActiveSubscription(userId: string): Promise<SubscriptionProfileDTO> {
    const query = new GetActiveSubscriptionQuery(userId);
    const result = await this.queryHandler.handle(query);
    return SubscriptionProfileDTO.fromDomain(result);
  }
}
```

---

## 🗄️ Persistence Layer

### ORM Models

**Location:** `Infrastructure/Persistence/Models/{Aggregate}Model.{rb,php,ts}`

**Naming:**
- Ruby/Rails: `{Aggregate}` (e.g., `Subscription < ApplicationRecord`)
- PHP/Laravel: `{Aggregate}Eloquent` (e.g., `SubscriptionEloquent extends Model`)
- TypeScript/TypeORM: `{Aggregate}Entity` (e.g., `@Entity() SubscriptionEntity`)

**Rules:**
- NO business logic in models
- Used ONLY in **Infrastructure** layer
- Define relationships, casts, and database mappings

### Repository Implementation

**Location:** `Infrastructure/Persistence/Repositories/{Aggregate}Repository.{rb,php,ts}`

**Responsibility:**
- Implement **Domain** repository interface
- Map between **Domain Aggregates** ↔ **ORM Models**
- Provide transaction helpers
- Handle database queries

**Pattern:**
```ruby
# Infrastructure/Persistence/Repositories/SubscriptionRepository.{rb,php,ts}
class SubscriptionRepository
  def save(subscription)
    model = to_model(subscription)
    model.save!
    to_domain(model)
  end
  
  def execute_in_transaction(&block)
    ActiveRecord::Base.transaction { block.call }
  end
  
  private
  
  def to_domain(model)
    # Map ORM → Domain
  end
  
  def to_model(subscription)
    # Map Domain → ORM
  end
end
```

```php
// Infrastructure/Persistence/Repositories/SubscriptionRepository.{rb,php,ts}
final class SubscriptionEloquentRepository implements SubscriptionRepository {
    public function save(Subscription $sub): Subscription {
        $model = $this->toModel($sub);
        $model->save();
        return $this->toDomain($model);
    }
    
    public function executeInTransaction(callable $fn): mixed {
        return DB::transaction($fn);
    }
    
    private function toDomain(SubscriptionEloquent $model): Subscription { /* ... */ }
    private function toModel(Subscription $sub): SubscriptionEloquent { /* ... */ }
}
```

```typescript
// Infrastructure/Persistence/Repositories/SubscriptionRepository.{rb,php,ts}
export class SubscriptionRepository implements ISubscriptionRepository {
  async save(subscription: Subscription): Promise<Subscription> {
    const entity = this.toEntity(subscription);
    await this.repo.save(entity);
    return this.toDomain(entity);
  }
  
  async executeInTransaction<T>(fn: () => Promise<T>): Promise<T> {
    return this.repo.manager.transaction(fn);
  }
  
  private toDomain(entity: SubscriptionEntity): Subscription { /* ... */ }
  private toEntity(sub: Subscription): SubscriptionEntity { /* ... */ }
}
```

### Migrations

**Location:** `Infrastructure/Persistence/Migrations/`

**Naming:**
- Ruby: `{timestamp}_{action}_{table}.rb`
- PHP: `{YYYY}_{MM}_{DD}_{seq}_{action}_{table}.php`
- TypeScript: `{timestamp}-{Action}{Table}.ts`

**Standards:**
- Use **UUID** primary keys for aggregates
- Always include **soft deletes** for user-created entities
- Index foreign keys and frequently queried columns
- Prefer numeric FKs for relational performance

**Pattern:**
```ruby
# {timestamp}_create_subscriptions.rb
class CreateSubscriptions < ActiveRecord::Migration[7.0]
  def change
    create_table :subscriptions, id: :uuid do |t|
      t.references :user, type: :uuid, null: false, foreign_key: true
      t.string :status, null: false, index: true
      t.timestamps
      t.datetime :deleted_at, index: true
    end
  end
end
```

```php
// {YYYY}_{MM}_{DD}_{seq}_create_subscriptions_table.php
return new class extends Migration {
    public function up(): void {
        Schema::create('subscriptions', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->foreignUuid('user_id')->constrained();
            $table->string('status')->index();
            $table->timestamps();
            $table->softDeletes();
        });
    }
};
```

```typescript
// {timestamp}-CreateSubscriptions.ts
export class CreateSubscriptions implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(new Table({
      name: 'subscriptions',
      columns: [
        { name: 'id', type: 'uuid', isPrimary: true },
        { name: 'user_id', type: 'uuid' },
        { name: 'status', type: 'varchar' },
        { name: 'created_at', type: 'timestamp' },
        { name: 'deleted_at', type: 'timestamp', isNullable: true }
      ],
      indices: [
        { columnNames: ['user_id'] },
        { columnNames: ['status'] },
        { columnNames: ['deleted_at'] }
      ]
    }));
  }
}
```

---

## 🌐 HTTP Layer

### Controllers (Thin Adapters)

**Location:** `Infrastructure/Http/Controllers/{Aggregate}Controller.{rb,php,ts}`

**Responsibility (3 steps only):**
1. Convert **HTTP Request** → **Command/Query**
2. Invoke **Application Handler**
3. Convert result → **HTTP Response** (Resource/Serializer)

**Must NOT:**
- Contain business logic
- Perform authorization (invoke via handler)
- Manage transactions
- Validate beyond basic shape

**Pattern:**
```ruby
# Infrastructure/Http/Controllers/InvoiceController.{rb,php,ts}
class InvoiceController < ApplicationController
  def create
    command = CreateInvoiceCommand.from_request(params)
    invoice = @handler.handle(command)
    render json: InvoiceResource.new(invoice), status: :created
  end
end
```

```php
// Infrastructure/Http/Controllers/InvoiceController.{rb,php,ts}
final class InvoiceController {
    public function create(Request $req, CreateInvoiceHandler $handler): JsonResponse {
        $command = CreateInvoiceCommand::fromRequest($req);
        $invoice = $handler->handle($command);
        return InvoiceResource::make($invoice)->response()->setStatusCode(201);
    }
}
```

```typescript
// Infrastructure/Http/Controllers/InvoiceController.{rb,php,ts}
export class InvoiceController {
  async create(req: Request, res: Response): Promise<void> {
    const command = CreateInvoiceCommand.fromRequest(req.body);
    const invoice = await this.handler.handle(command);
    res.status(201).json(InvoiceResource.from(invoice));
  }
}
```

### Resources/Serializers (Response DTOs)

**Location:** `Infrastructure/Http/Resources/{Aggregate}Resource.{rb,php,ts}`

**Responsibility:**
- Transform **Domain Types** → **HTTP JSON**
- Control API surface (hide internal details)
- Format dates, currencies, enums for external consumption

**Rules:**
- NEVER expose aggregates directly
- Map domain concepts to API-friendly structure
- Handle null/optional fields explicitly

---

## 🔄 Async Processing

### Jobs/Workers

**Location:** `Infrastructure/Jobs/{Action}Job.{rb,php,ts}`

**Responsibility:**
- Execute heavy/remote work asynchronously
- Call **Application Handlers** or **Domain Services**
- NO business rules directly in job

**Pattern:**
```ruby
# Infrastructure/Jobs/SendWelcomeEmailJob.{rb,php,ts}
class SendWelcomeEmailJob < ApplicationJob
  def perform(subscription_id)
    @handler.handle(SendWelcomeEmailCommand.new(subscription_id))
  end
end
```

```php
// Infrastructure/Jobs/SendWelcomeEmailJob.{rb,php,ts}
final class SendWelcomeEmailJob implements ShouldQueue {
    public function handle(SendWelcomeEmailHandler $handler): void {
        $handler->handle(new SendWelcomeEmailCommand($this->subscriptionId));
    }
}
```

```typescript
// Infrastructure/Jobs/SendWelcomeEmailJob.{rb,php,ts}
export class SendWelcomeEmailJob {
  async execute(): Promise<void> {
    await this.handler.handle(new SendWelcomeEmailCommand(this.subscriptionId));
  }
}
```

### Listeners/Subscribers

**Location:** `Infrastructure/Listeners/{Event}Listener.{rb,php,ts}`

**Responsibility:**
- Subscribe to **Domain Events**
- Translate event → dispatch **Job/Worker**
- Synchronous, lightweight (NO heavy work)

**Pattern:**
```ruby
# Infrastructure/Listeners/SubscriptionCreatedListener.{rb,php,ts}
class SubscriptionCreatedListener
  def handle(event)
    SendWelcomeEmailJob.perform_later(event.subscription_id)
  end
end
```

```php
// Infrastructure/Listeners/SubscriptionCreatedListener.{rb,php,ts}
final class SubscriptionCreatedListener {
    public function handle(SubscriptionCreated $event): void {
        SendWelcomeEmailJob::dispatch($event->subscriptionId);
    }
}
```

```typescript
// Infrastructure/Listeners/SubscriptionCreatedListener.{rb,php,ts}
export class SubscriptionCreatedListener {
  async handle(event: SubscriptionCreated): Promise<void> {
    await this.queue.add(SendWelcomeEmailJob.name, { 
      subscriptionId: event.subscriptionId 
    });
  }
}
```

---

## 🧪 Testing Structure

### Unit Tests (Domain Focus)

**Location:** `tests/Unit/Domain/`

**What to Test:**
- Aggregate invariants and business rules
- Value object validation
- Domain event creation
- Policy authorization logic
- Domain service behavior

**Characteristics:**
- NO framework dependencies
- NO database access
- Fast execution (< 100ms per test)
- Pure domain logic only

**Example Structure:**
```
tests/Unit/
├── Domain/
│   ├── Aggregates/
│   │   └── SubscriptionTest.{rb,php,ts}
│   ├── ValueObjects/
│   │   └── EmailAddressTest.{rb,php,ts}
│   ├── Policies/
│   │   └── WorkOrderPolicyTest.{rb,php,ts}
│   └── Services/
│       └── PricingServiceTest.{rb,php,ts}
```

### Integration Tests (Infrastructure Focus)

**Location:** `tests/Integration/Infrastructure/`

**What to Test:**
- HTTP controller flows (request → response)
- Repository mapping (Domain ↔ ORM)
- Integration API contracts
- Job execution
- Event listener dispatching

**Characteristics:**
- Uses framework testing utilities
- Database transactions/cleanup
- Slower than unit tests
- Tests adapter correctness

**Example Structure:**
```
tests/Integration/
├── Infrastructure/
│   ├── Http/
│   │   └── InvoiceControllerTest.{rb,php,ts}
│   ├── Persistence/
│   │   └── SubscriptionRepositoryTest.{rb,php,ts}
│   ├── Jobs/
│   │   └── SendWelcomeEmailJobTest.{rb,php,ts}
│   └── Integration/
│       └── SubscriptionApiTest.{rb,php,ts}
```

---

## 📦 Service Registration

### Service Provider/Container Setup

**Location:** `Infrastructure/Providers/{Package}ServiceProvider.{rb,php,ts}`

**Responsibility:**
- Bind **Domain Interfaces** → **Infrastructure Implementations**
- Register routes
- Load migrations
- Configure package services

**Pattern:**
```ruby
# Infrastructure/Providers/SubscriptionServiceProvider.{rb,php,ts}
class SubscriptionServiceProvider
  def register(container)
    container.bind(
      SubscriptionRepository,
      SubscriptionEloquentRepository
    )
  end
  
  def boot
    load_routes
    load_migrations
  end
end
```

```php
// Infrastructure/Providers/SubscriptionServiceProvider.{rb,php,ts}
final class SubscriptionServiceProvider extends ServiceProvider {
    public function register(): void {
        $this->app->bind(
            SubscriptionRepositoryInterface::class,
            SubscriptionEloquentRepository::class
        );
    }
    
    public function boot(): void {
        $this->loadRoutesFrom(__DIR__.'/../../routes/api.php');
        $this->loadMigrationsFrom(__DIR__.'/../Persistence/Migrations');
    }
}
```

```typescript
// Infrastructure/Providers/SubscriptionServiceProvider.{rb,php,ts}
export class SubscriptionModule {
  static register(container: Container): void {
    container
      .bind<ISubscriptionRepository>(TYPES.SubscriptionRepository)
      .to(SubscriptionRepository);
      
    this.registerRoutes(app);
    this.runMigrations();
  }
}
```

---

## ✅ Code Review Checklist

### Package Structure
- [ ] **Domain**, **Application**, **Infrastructure** folders present
- [ ] Files placed in correct layer based on responsibility
- [ ] Naming conventions followed consistently
- [ ] No framework imports in **Domain** layer

### Domain Layer
- [ ] Aggregates, VOs, Events, Policies properly organized
- [ ] Repository **interfaces** only (no implementations)
- [ ] No persistence, HTTP, or framework dependencies
- [ ] README.md with domain glossary present

### Application Layer
- [ ] Commands/Queries in separate folders
- [ ] Handlers accept Commands/Queries, return Domain Types/DTOs
- [ ] No ORM models or HTTP concerns
- [ ] Transaction boundaries explicit

### Infrastructure Layer
- [ ] Controllers are thin (< 15 lines typically)
- [ ] Repository implementations map Domain ↔ ORM
- [ ] Resources/Serializers format HTTP responses
- [ ] Jobs delegate to handlers (no business logic)
- [ ] Integration APIs use External DTOs only

### Persistence
- [ ] Migrations use UUID PKs for aggregates
- [ ] Soft deletes on user-created entities
- [ ] Foreign keys and hot columns indexed
- [ ] Repository provides transaction helpers

### Testing
- [ ] Unit tests focus on Domain (no framework)
- [ ] Integration tests cover Infrastructure adapters
- [ ] Clear separation between test types
- [ ] Tests grouped by layer/concern

---

## 🚫 Anti-Patterns to Avoid

### Structure Violations
- ❌ Mixing Domain and Infrastructure in same folder
- ❌ Controllers in Application layer
- ❌ Business logic in Infrastructure
- ❌ Repository implementations in Domain

### Naming Violations
- ❌ Generic names (`Manager`, `Helper`, `Util`)
- ❌ Framework-coupled names in Domain (`{Aggregate}Eloquent` in Domain)
- ❌ Inconsistent suffixes (mixing `Service`, `Manager`, `Handler`)

### Dependency Violations
- ❌ Domain importing from Infrastructure
- ❌ Application importing from Infrastructure (except via interfaces)
- ❌ Circular dependencies between packages

### File Placement Violations
- ❌ Business rules in ORM models
- ❌ Transaction management in controllers
- ❌ Domain logic in jobs/listeners
- ❌ HTTP concerns in Application layer

---

## 💡 AI Assistant Guidelines

### When Creating New Files

**Always Ask:**
1. Which **layer** does this belong to?
2. What is the correct **folder** within that layer?
3. Does the **naming** follow conventions?
4. Are **dependencies** pointing in the right direction?

### File Placement Decision Flow

```
Is it CORE BUSINESS LOGIC?
└─ YES → Domain/
   ├─ Entity with behavior? → Aggregates/{Aggregate}/{Aggregate}.{ext}
   ├─ Immutable type? → ValueObjects/{Name}.{ext}
   ├─ State change record? → Events/{Aggregate}{Action}.{ext}
   ├─ Authorization rule? → Policies/{Aggregate}Policy.{ext}
   └─ Persistence contract? → Aggregates/{Aggregate}/{Aggregate}Repository.{ext}

Is it USE-CASE ORCHESTRATION?
└─ YES → Application/
   ├─ Write operation? → Commands/{Aggregate}{Action}/
   └─ Read operation? → Queries/{Aggregate}{View}/

Is it ADAPTER/FRAMEWORK CODE?
└─ YES → Infrastructure/
   ├─ HTTP? → Http/Controllers/ or Http/Resources/
   ├─ Database? → Persistence/Models/ or Persistence/Repositories/
   ├─ Async? → Jobs/ or Listeners/
   └─ External API? → Integration/{Aggregate}Api.{ext}
```

### When Reviewing Structure

**Red Flags:**
- Framework imports in `Domain/` folder
- Business logic in `Infrastructure/` files
- ORM models in `Application/` or `Domain/`
- Controllers calling other controllers
- Jobs containing business rules
- Missing repository interfaces in `Domain/`

**Green Flags:**
- Clean layer separation with predictable paths
- Consistent naming across package
- Interfaces in Domain, implementations in Infrastructure
- Thin controllers (< 15 lines)
- Domain README with glossary
- Clear test separation (Unit vs Integration)

---

## 🔧 Language-Specific Adaptations

### Ruby/Rails Conventions
- Use `snake_case` for files and methods
- Models in `app/models/` (or package equivalent)
- Migrations use Rails timestamp format
- Service objects common pattern

### PHP/Laravel Conventions
- Use `PascalCase` for classes, `camelCase` for methods
- Models often suffixed with `Eloquent` to distinguish from aggregates
- Migrations in `database/migrations/`
- Service providers for registration

### TypeScript/Node Conventions
- Use `PascalCase` for classes, `camelCase` for properties
- Consider `index.ts` barrel exports for clean imports
- Entities often suffixed for TypeORM clarity
- Module-based dependency injection

**Consistency within project > strict adherence to language idioms when architectural clarity is at stake.**