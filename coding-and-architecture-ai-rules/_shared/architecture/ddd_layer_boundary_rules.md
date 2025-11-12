# DDD Layer Boundaries — AI Development Rules

**Purpose:** Enforce clean architecture separation in Domain-Driven Design. Protect **Domain** purity, enable **testability**, and guide precise code placement.

---

## 🎯 Core Principles

### Dependency Rule
- **Domain** NEVER depends on **Application** or **Infrastructure**
- **Application** depends on **Domain** only (via interfaces)
- **Infrastructure** depends on both **Domain** and **Application**

### Contract Enforcement
- All cross-layer communication uses **domain types** or **explicit DTOs**
- NO arrays, primitives, or framework types for business concepts
- NO direct persistence models in **Domain** or **Application**

---

## 📦 Layer Responsibilities

### Domain Layer (Core Business Logic)

**Owns:**
- **Aggregates** — cluster of entities with consistency boundary
- **Value Objects (VOs)** — immutable business primitives
- **Domain Events** — record of state changes
- **Policies** — authorization and business rule evaluators
- **Domain Services** — stateless operations across aggregates
- **Repository Interfaces** — persistence contracts (NOT implementations)

**Must NOT Know:**
- Frameworks (Laravel, Rails, Express, NestJS)
- Persistence details (databases, ORMs, ActiveRecord, Eloquent, TypeORM)
- Infrastructure concerns (caching, queues, logging)
- External APIs or integrations

**I/O Contract:**
- Accepts: **Domain Types** (Aggregates, VOs, Domain DTOs)
- Returns: **Domain Types** only

---

### Application Layer (Use-Case Orchestration)

**Owns:**
- **Commands/Queries** — intent objects with validation
- **Handlers** — execute single use case
- **Transaction Boundaries** — coordinate persistence
- **Cross-Aggregate Coordination** — manage consistency

**Must NOT Know:**
- Controllers or HTTP requests/responses
- Persistence Models (Eloquent, ActiveRecord, TypeORM entities)
- Infrastructure implementations (only interfaces)

**I/O Contract:**
- Accepts: **Commands/Queries**
- Returns: **Domain Types** or **Result DTOs**

**Rules:**
- NO handler-to-handler chaining
- Authorization via **Domain Policies** (invoke, don't implement)
- Transactions via **Repository Interface** methods

---

### Infrastructure Layer (Adapters & Framework)

**Owns:**
- **Controllers** — HTTP adapters (thin)
- **Repository Implementations** — persistence mapping
- **Jobs/Workers** — async work execution
- **Listeners/Subscribers** — event-to-job translation
- **Resources/Serializers/DTOs** — HTTP response formatting
- **External Integrations** — API clients, adapters
- **Framework Concerns** — caching, logging, queues

**Must NOT:**
- Contain business rules or domain logic
- Orchestrate use cases (that's **Application**)
- Expose aggregates directly in APIs

**I/O Contract:**
- Translates between framework types and **Domain/Application contracts**

---

## 🔗 Dependency Matrix (Quick Reference)

| From ↓ / To → | **Domain** | **Application** | **Infrastructure** |
|---------------|:----------:|:---------------:|:------------------:|
| **Domain** | ✅ | ❌ | ❌ |
| **Application** | ✅ | ❌ | ❌ (interfaces only) |
| **Infrastructure** | ✅ | ✅ | ✅ |

---

## 🧱 Boundary Implementation Patterns

### Repositories (Persistence Boundary)

**Interface (Domain):**
```ruby
# Domain/Aggregates/Subscription/SubscriptionRepository.{rb,php,ts}
interface SubscriptionRepository
  def save(subscription)
  def get_by_id(subscription_id)
  def execute_in_transaction(&block)
end
```

```php
// Domain/Aggregates/Subscription/SubscriptionRepository.{rb,php,ts}
interface SubscriptionRepository {
    public function save(Subscription $sub): Subscription;
    public function getById(SubscriptionId $id): ?Subscription;
    public function executeInTransaction(callable $fn): mixed;
}
```

```typescript
// Domain/Aggregates/Subscription/SubscriptionRepository.{rb,php,ts}
interface SubscriptionRepository {
  save(subscription: Subscription): Promise<Subscription>;
  getById(id: SubscriptionId): Promise<Subscription | null>;
  executeInTransaction<T>(fn: () => Promise<T>): Promise<T>;
}
```

**Implementation (Infrastructure):**
- Lives in `Infrastructure/Persistence/{Aggregate}Repository.{rb,php,ts}`
- Maps **Domain Aggregates** ↔ **Persistence Models**
- Handles ORM concerns, queries, transactions

**Rules:**
- Interface signatures use **Domain Types** only
- NO ORM models (ActiveRecord, Eloquent, TypeORM) in Domain layer
- Transaction helpers exposed on interface

---

### Controllers (HTTP Boundary)

**Responsibility:**
1. Convert **HTTP Request** → **Command/Query**
2. Invoke **Application Handler**
3. Convert result → **HTTP Response DTO/Resource**

**Must Be Thin:**
- NO business logic
- NO authorization checks (invoke **Domain Policy** via handler)
- NO transaction management
- NO validation beyond basic shape conversion

**Pattern:**
```ruby
# Infrastructure/Http/Controllers/WorkOrderController.{rb,php,ts}
def update
  command = UpdateWorkOrder.from_request(request, params[:id])
  work_order = @handler.handle(command)
  render json: WorkOrderResource.new(work_order)
end
```

```php
// Infrastructure/Http/Controllers/WorkOrderController.{rb,php,ts}
public function update(Request $req, string $id, UpdateWorkOrderHandler $handler) {
    $command = UpdateWorkOrder::fromRequest($req, $id);
    $workOrder = $handler->handle($command);
    return WorkOrderResource::make($workOrder);
}
```

```typescript
// Infrastructure/Http/Controllers/WorkOrderController.{rb,php,ts}
async update(req: Request, res: Response) {
  const command = UpdateWorkOrder.fromRequest(req, req.params.id);
  const workOrder = await this.handler.handle(command);
  return res.json(WorkOrderResource.from(workOrder));
}
```

---

### Integration APIs (Cross-Bounded Context)

**Purpose:** Synchronous communication between bounded contexts

**Location:** `Infrastructure/Integration/{Aggregate}Api.{rb,php,ts}`

**Contract:**
- Accept/return **External DTOs** (NOT aggregates)
- NO business rules (those live in **Domain**)
- NO authorization (handled by **Application**)

---

### Jobs & Listeners (Async Boundary)

**Listeners/Subscribers:**
- Live in **Infrastructure**
- Synchronously translate **Domain Event** → dispatch **Job/Worker**
- NO heavy work or external calls

**Jobs/Workers:**
- Live in **Infrastructure**
- Execute async/heavy work via **Application Handlers** or **Domain Services**
- NO business rules directly in job

**Pattern:**
```ruby
# Listener.{rb,php,ts}
def handle(event)
  SendWelcomeEmailJob.perform_later(event.subscription_id)
end

# Job.{rb,php,ts}
def perform(subscription_id)
  @handler.handle(SendWelcomeCommand.new(subscription_id))
end
```

```php
// Listener.{rb,php,ts}
public function handle(SubscriptionCreated $event): void {
    SendWelcomeEmail::dispatch($event->subscriptionId);
}

// Job.{rb,php,ts}
public function handle(SendWelcomeHandler $handler): void {
    $handler->handle(new SendWelcomeCommand($this->subscriptionId));
}
```

```typescript
// Listener.{rb,php,ts}
async handle(event: SubscriptionCreated): Promise<void> {
  await this.queue.dispatch(new SendWelcomeEmailJob(event.subscriptionId));
}

// Job.{rb,php,ts}
async execute(): Promise<void> {
  await this.handler.handle(new SendWelcomeCommand(this.subscriptionId));
}
```

---

### Policies (Authorization/Business Rules)

**Implementation:**
- Lives in **Domain** layer
- Pure business logic (NO framework dependencies)
- Returns capability results (can/cannot)

**Invocation:**
- **Application Handler** invokes policy BEFORE mutations
- Enforces authorization at use-case level

**Pattern:**
```ruby
# Domain/Policies/WorkOrderPolicy.{rb,php,ts}
def can_approve?(work_order, user)
  PolicyResult.new(...)
end

# Application/Handlers/ApproveWorkOrderHandler.{rb,php,ts}
result = @policy.can_approve?(work_order, command.user)
raise UnauthorizedException, result.reason unless result.allowed?
```

```php
// Domain/Policies/WorkOrderPolicy.{rb,php,ts}
public function canApprove(WorkOrder $wo, User $user): PolicyResult

// Application/Handlers/ApproveWorkOrderHandler.{rb,php,ts}
$result = $this->policy->canApprove($workOrder, $command->user);
if (!$result->allowed) {
    throw new UnauthorizedException($result->reason);
}
```

```typescript
// Domain/Policies/WorkOrderPolicy.{rb,php,ts}
canApprove(workOrder: WorkOrder, user: User): PolicyResult

// Application/Handlers/ApproveWorkOrderHandler.{rb,php,ts}
const result = this.policy.canApprove(workOrder, command.user);
if (!result.allowed) {
  throw new UnauthorizedException(result.reason);
}
```

---

## ⚠️ Anti-Patterns (Violations to Avoid)

### Domain Layer Violations
- ❌ Importing framework-specific code (Rails ActiveSupport, Laravel facades, Express middleware)
- ❌ Calling HTTP utilities or framework abstractions
- ❌ Direct database queries or ORM usage
- ❌ Accessing configuration or environment variables

### Application Layer Violations
- ❌ Using ORM models (ActiveRecord, Eloquent, TypeORM entities)
- ❌ Accessing controllers or HTTP requests
- ❌ Implementing business rules (should be in **Domain**)
- ❌ Handler-to-handler chaining (breaks transaction boundaries)

### Infrastructure Layer Violations
- ❌ Fat controllers with orchestration logic
- ❌ Business rules in ORM models or repositories
- ❌ Direct aggregate mutations in controllers/jobs
- ❌ Bypassing **Application** handlers

### Cross-Cutting Violations
- ❌ Passing arrays/primitives for business concepts (use **VOs**)
- ❌ Exposing aggregates in HTTP responses (use **Resources/Serializers/DTOs**)
- ❌ Passing ORM models through **Domain/Application** APIs
- ❌ Mixing persistence logic into domain services

---

## ✅ Code Placement Decision Tree

**Adding Business Logic?**
- Complex rules/invariants → **Domain Service** or **Aggregate method**
- Authorization check → **Domain Policy**
- Simple validation → **Value Object** constructor

**Adding Use Case?**
- Create **Command/Query** in **Application/Commands**
- Create **Handler** in **Application/Handlers**
- Handler invokes **Domain** methods and **Repositories**

**Adding HTTP Endpoint?**
- Create thin **Controller** in **Infrastructure/Http**
- Controller converts Request → Command → Handler → Resource/Serializer

**Adding Persistence?**
- Define **Interface** in **Domain**
- Implement in **Infrastructure/Persistence**
- Map Domain ↔ Persistence models in repository

**Adding External Integration?**
- Create **Adapter/Client** in **Infrastructure/Integration**
- Use **Domain DTOs** or **External DTOs** for contracts
- Inject into **Application** handlers as interface

**Adding Async Work?**
- Create **Listener/Subscriber** in **Infrastructure/Listeners** (subscribe to domain event)
- Create **Job/Worker** in **Infrastructure/Jobs** (call handler)
- NO business logic in listener or job

---

## 🔍 Code Review Checklist

### Domain Purity
- [ ] NO framework imports (Rails, Laravel, Express, NestJS)
- [ ] Uses only **Domain Types** in method signatures
- [ ] Repository **interfaces** (not implementations) only
- [ ] Business rules self-contained in aggregates/policies

### Application Correctness
- [ ] Handlers accept **Commands/Queries**
- [ ] Handlers return **Domain Types** or **Result DTOs**
- [ ] NO ORM models or persistence concerns
- [ ] Transactions managed via **Repository Interface**
- [ ] Authorization via **Domain Policy** invocation

### Infrastructure Isolation
- [ ] Controllers are thin (no business logic)
- [ ] Mapping isolated to repositories/resources/serializers
- [ ] NO aggregates exposed in HTTP responses
- [ ] Jobs/Workers/Listeners delegate to handlers
- [ ] External integrations use adapters/ACL

### Contract Adherence
- [ ] NO arrays for business concepts (use **VOs** or **DTOs**)
- [ ] Clear **Domain Type** definitions
- [ ] Proper **DTO** usage at boundaries
- [ ] Consistent mapping strategies

---

## 📝 Quick Reference: Where Does X Go?

| What | Layer | File Pattern | Notes |
|------|-------|--------------|-------|
| Aggregate | **Domain** | `Domain/Aggregates/{Name}/{Name}.{rb,php,ts}` | Core entity cluster |
| Value Object | **Domain** | `Domain/ValueObjects/{Name}.{rb,php,ts}` | Immutable business primitive |
| Domain Event | **Domain** | `Domain/Events/{Name}.{rb,php,ts}` | State change record |
| Policy | **Domain** | `Domain/Policies/{Name}Policy.{rb,php,ts}` | Authorization/business rule |
| Repository Interface | **Domain** | `Domain/Aggregates/{Name}/{Name}Repository.{rb,php,ts}` | Persistence contract |
| Command/Query | **Application** | `Application/Commands/{Name}.{rb,php,ts}` | Intent with validation |
| Handler | **Application** | `Application/Handlers/{Name}Handler.{rb,php,ts}` | Single use case |
| Controller | **Infrastructure** | `Infrastructure/Http/Controllers/{Name}Controller.{rb,php,ts}` | HTTP adapter (thin) |
| Repository Impl | **Infrastructure** | `Infrastructure/Persistence/{Name}Repository.{rb,php,ts}` | Persistence mapping |
| Job/Worker | **Infrastructure** | `Infrastructure/Jobs/{Name}Job.{rb,php,ts}` | Async work executor |
| Listener/Subscriber | **Infrastructure** | `Infrastructure/Listeners/{Name}Listener.{rb,php,ts}` | Event-to-job bridge |
| Resource/Serializer | **Infrastructure** | `Infrastructure/Http/Resources/{Name}Resource.{rb,php,ts}` | HTTP response format |
| External Client | **Infrastructure** | `Infrastructure/Integration/{Name}Client.{rb,php,ts}` | API integration adapter |

---

## 💡 AI Assistant Guidelines

### When Generating Code

1. **Always ask:** "Which layer does this belong to?"
2. **Check dependencies:** Does this violate the dependency rule?
3. **Verify I/O:** Are the right types being passed across boundaries?
4. **Question arrays:** Should this be a VO or DTO instead?
5. **Locate interfaces:** Is the interface in Domain and impl in Infrastructure?

### When Reviewing Code

1. Scan imports for framework leakage in **Domain/Application**
2. Check for ORM usage outside **Infrastructure**
3. Verify controllers are thin (< 10 lines typically)
4. Confirm handlers don't chain to other handlers
5. Ensure aggregates aren't exposed in HTTP responses

### Framework-Agnostic Principles

- Replace framework-specific terms with generic equivalents
- **ORM** = ActiveRecord, Eloquent, TypeORM, Prisma, Hibernate
- **Jobs** = Jobs, Workers, Background Tasks, Queue Processors
- **Listeners** = Listeners, Subscribers, Event Handlers, Observers
- **Resources** = Resources, Serializers, Presenters, DTOs

**Default to:** Making boundaries explicit, extracting VOs, and questioning shortcuts that blur layer responsibilities.

---

## 🔧 Language-Specific Conventions

### Ruby/Rails
- Use snake_case for methods/files
- Repository interfaces as modules or abstract classes
- Jobs via ActiveJob, Sidekiq, or Resque

### PHP/Laravel
- Use PascalCase for classes, camelCase for methods
- Interfaces with explicit `interface` keyword
- Jobs via Laravel Queues

### TypeScript/Node
- Use PascalCase for classes, camelCase for methods/properties
- Interfaces with explicit `interface` or `abstract class`
- Jobs via Bull, BullMQ, or similar queue libraries

**Consistency within project > strict adherence to language idioms when it conflicts with DDD boundaries**