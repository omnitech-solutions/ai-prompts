# DDD Architecture — AI Development Rules

**Purpose:** Comprehensive Domain-Driven Design (DDD) implementation guide. Enforce tactical patterns, strategic design, and architectural boundaries for maintainable, business-aligned systems.

---

## 🎯 Core DDD Philosophy

### Business-First Mindset
- **Code reflects business language** — use terms from domain experts (Ubiquitous Language)
- **Business rules live in Domain** — not scattered across controllers, services, or database
- **Protect invariants** — aggregates enforce consistency boundaries
- **Model behaviors, not data structures** — focus on what entities *do*, not just what they *are*

### Architecture Goals
- **High cohesion** within bounded contexts
- **Loose coupling** between bounded contexts
- **Testable** domain logic without framework dependencies
- **Evolvable** system that adapts to business changes
- **Explicit** boundaries and contracts

---

## 📚 DDD Building Blocks (Tactical Patterns)

### 1. Entities

**Definition:** Objects with identity that persist over time, distinguished by ID rather than attributes.

**Characteristics:**
- Has unique identifier (usually UUID for external facing, numeric for internal)
- Identity remains constant even when attributes change
- Mutable behavior through domain methods
- Tracks lifecycle and state transitions

**Implementation:**
```ruby
# Domain/Aggregates/Order/Order.{rb,php,ts}
class Order
  attr_reader :id, :status, :items
  
  def initialize(id, customer_id)
    @id = id
    @customer_id = customer_id
    @status = OrderStatus.pending
    @items = []
  end
  
  def add_item(product, quantity)
    raise OrderAlreadyPlacedError if placed?
    @items << OrderItem.new(product, quantity)
  end
  
  def place
    raise EmptyOrderError if @items.empty?
    @status = OrderStatus.placed
    @events << OrderPlaced.new(@id)
  end
  
  private
  
  def placed?
    @status == OrderStatus.placed
  end
end
```

```php
// Domain/Aggregates/Order/Order.{rb,php,ts}
final class Order {
    private OrderId $id;
    private OrderStatus $status;
    private array $items = [];
    private array $events = [];
    
    public function __construct(OrderId $id, CustomerId $customerId) {
        $this->id = $id;
        $this->customerId = $customerId;
        $this->status = OrderStatus::Pending;
    }
    
    public function addItem(Product $product, int $quantity): void {
        if ($this->isPlaced()) {
            throw new OrderAlreadyPlacedException();
        }
        $this->items[] = new OrderItem($product, $quantity);
    }
    
    public function place(): void {
        if (empty($this->items)) {
            throw new EmptyOrderException();
        }
        $this->status = OrderStatus::Placed;
        $this->events[] = new OrderPlaced($this->id);
    }
}
```

```typescript
// Domain/Aggregates/Order/Order.{rb,php,ts}
export class Order {
  private readonly id: OrderId;
  private status: OrderStatus;
  private items: OrderItem[] = [];
  private events: DomainEvent[] = [];
  
  constructor(id: OrderId, customerId: CustomerId) {
    this.id = id;
    this.customerId = customerId;
    this.status = OrderStatus.Pending;
  }
  
  addItem(product: Product, quantity: number): void {
    if (this.isPlaced()) {
      throw new OrderAlreadyPlacedError();
    }
    this.items.push(new OrderItem(product, quantity));
  }
  
  place(): void {
    if (this.items.length === 0) {
      throw new EmptyOrderError();
    }
    this.status = OrderStatus.Placed;
    this.events.push(new OrderPlaced(this.id));
  }
}
```

**Rules:**
- NO public setters — use expressive methods (`order.place()` not `order.setStatus('placed')`)
- Validate state transitions
- Emit domain events for significant state changes
- Encapsulate business rules

---

### 2. Value Objects (VOs)

**Definition:** Immutable objects defined by their attributes, with no identity. Two VOs with same attributes are considered equal.

**Characteristics:**
- **Immutable** — once created, cannot change
- **Self-validating** — constructor enforces invariants
- **Replaceable** — change by creating new instance
- **Equality by value** — not by reference

**When to Use:**
- Concepts without lifecycle (Email, Money, Address)
- Measurements or quantities
- Descriptive characteristics
- Complex attributes that need validation

**Implementation:**
```ruby
# Domain/ValueObjects/Money.{rb,php,ts}
class Money
  attr_reader :amount, :currency
  
  def initialize(amount, currency)
    raise InvalidAmountError if amount.negative?
    raise InvalidCurrencyError unless valid_currency?(currency)
    
    @amount = amount.round(2)
    @currency = currency
    freeze # Make immutable
  end
  
  def add(other)
    raise CurrencyMismatchError unless same_currency?(other)
    Money.new(@amount + other.amount, @currency)
  end
  
  def ==(other)
    other.is_a?(Money) && 
      @amount == other.amount && 
      @currency == other.currency
  end
  
  private
  
  def same_currency?(other)
    @currency == other.currency
  end
end
```

```php
// Domain/ValueObjects/Money.{rb,php,ts}
final readonly class Money {
    public function __construct(
        public float $amount,
        public Currency $currency
    ) {
        if ($amount < 0) {
            throw new InvalidAmountException();
        }
        $this->amount = round($amount, 2);
    }
    
    public function add(Money $other): Money {
        if (!$this->currency->equals($other->currency)) {
            throw new CurrencyMismatchException();
        }
        return new Money($this->amount + $other->amount, $this->currency);
    }
    
    public function equals(Money $other): bool {
        return $this->amount === $other->amount 
            && $this->currency->equals($other->currency);
    }
}
```

```typescript
// Domain/ValueObjects/Money.{rb,php,ts}
export class Money {
  private constructor(
    public readonly amount: number,
    public readonly currency: Currency
  ) {
    if (amount < 0) {
      throw new InvalidAmountError();
    }
    Object.freeze(this);
  }
  
  static create(amount: number, currency: Currency): Money {
    return new Money(Math.round(amount * 100) / 100, currency);
  }
  
  add(other: Money): Money {
    if (!this.currency.equals(other.currency)) {
      throw new CurrencyMismatchError();
    }
    return Money.create(this.amount + other.amount, this.currency);
  }
  
  equals(other: Money): boolean {
    return this.amount === other.amount 
      && this.currency.equals(other.currency);
  }
}
```

**Common VOs:**
- `EmailAddress` — validates email format
- `Money` — amount + currency with arithmetic
- `PhoneNumber` — validates and formats phone
- `Address` — street, city, postal code, country
- `DateRange` — start/end with validation
- `Percentage` — 0-100 with bounds checking
- `UUID` / `ExternalId` — identifier wrapper

**Rules:**
- Validate in constructor — fail fast
- Provide factory methods for complex creation
- Implement equality comparison
- NO setters or mutable methods
- Use for replacing primitives (String Obsession anti-pattern)

---

### 3. Aggregates

**Definition:** Cluster of entities and VOs treated as a single unit for data changes. Has a root entity (Aggregate Root) that controls access.

**Purpose:**
- Define **consistency boundaries** — what must be transactionally consistent
- Control **invariant enforcement** — business rules that must always be true
- Minimize **contention** — smaller aggregates = better concurrency
- Simplify **persistence** — save/load as a unit

**Design Principles:**
- **Small aggregates** — prefer smaller consistency boundaries
- **Reference by ID** — aggregates reference other aggregates by ID, not direct object reference
- **Eventual consistency** between aggregates — use domain events
- **Root guards invariants** — all mutations go through aggregate root
- **Single transaction per aggregate** — don't modify multiple aggregates in one transaction

**Implementation:**
```ruby
# Domain/Aggregates/Subscription/Subscription.{rb,php,ts}
class Subscription
  attr_reader :id, :status, :line_items, :events
  
  def initialize(id, customer_id, plan_id)
    @id = id
    @customer_id = customer_id
    @plan_id = plan_id
    @status = SubscriptionStatus.trial
    @line_items = []
    @events = []
  end
  
  # Aggregate root controls all changes
  def add_addon(addon_id, quantity)
    raise SubscriptionCancelledError if cancelled?
    
    # Enforce invariant: no duplicate addons
    raise DuplicateAddonError if has_addon?(addon_id)
    
    @line_items << SubscriptionLineItem.new(addon_id, quantity)
    @events << AddonAdded.new(@id, addon_id)
  end
  
  def activate(payment_method_id)
    raise InvalidStateTransition unless can_activate?
    
    @status = SubscriptionStatus.active
    @payment_method_id = payment_method_id
    @events << SubscriptionActivated.new(@id)
  end
  
  def cancel(reason)
    raise AlreadyCancelledError if cancelled?
    
    @status = SubscriptionStatus.cancelled
    @cancelled_at = Time.current
    @cancellation_reason = reason
    @events << SubscriptionCancelled.new(@id, reason)
  end
  
  private
  
  def can_activate?
    trial? || grace_period?
  end
  
  def has_addon?(addon_id)
    @line_items.any? { |item| item.addon_id == addon_id }
  end
end
```

```php
// Domain/Aggregates/Subscription/Subscription.{rb,php,ts}
final class Subscription {
    private SubscriptionId $id;
    private SubscriptionStatus $status;
    private array $lineItems = [];
    private array $events = [];
    
    private function __construct(
        SubscriptionId $id,
        CustomerId $customerId,
        PlanId $planId
    ) {
        $this->id = $id;
        $this->customerId = $customerId;
        $this->planId = $planId;
        $this->status = SubscriptionStatus::Trial;
    }
    
    public static function create(
        SubscriptionId $id,
        CustomerId $customerId,
        PlanId $planId
    ): self {
        $subscription = new self($id, $customerId, $planId);
        $subscription->events[] = new SubscriptionCreated($id);
        return $subscription;
    }
    
    public function addAddon(AddonId $addonId, int $quantity): void {
        if ($this->isCancelled()) {
            throw new SubscriptionCancelledException();
        }
        
        if ($this->hasAddon($addonId)) {
            throw new DuplicateAddonException();
        }
        
        $this->lineItems[] = new SubscriptionLineItem($addonId, $quantity);
        $this->events[] = new AddonAdded($this->id, $addonId);
    }
    
    public function activate(PaymentMethodId $paymentMethodId): void {
        if (!$this->canActivate()) {
            throw new InvalidStateTransitionException();
        }
        
        $this->status = SubscriptionStatus::Active;
        $this->paymentMethodId = $paymentMethodId;
        $this->events[] = new SubscriptionActivated($this->id);
    }
}
```

```typescript
// Domain/Aggregates/Subscription/Subscription.{rb,php,ts}
export class Subscription {
  private readonly id: SubscriptionId;
  private status: SubscriptionStatus;
  private lineItems: SubscriptionLineItem[] = [];
  private events: DomainEvent[] = [];
  
  private constructor(
    id: SubscriptionId,
    customerId: CustomerId,
    planId: PlanId
  ) {
    this.id = id;
    this.customerId = customerId;
    this.planId = planId;
    this.status = SubscriptionStatus.Trial;
  }
  
  static create(
    id: SubscriptionId,
    customerId: CustomerId,
    planId: PlanId
  ): Subscription {
    const subscription = new Subscription(id, customerId, planId);
    subscription.events.push(new SubscriptionCreated(id));
    return subscription;
  }
  
  addAddon(addonId: AddonId, quantity: number): void {
    if (this.isCancelled()) {
      throw new SubscriptionCancelledException();
    }
    
    if (this.hasAddon(addonId)) {
      throw new DuplicateAddonException();
    }
    
    this.lineItems.push(new SubscriptionLineItem(addonId, quantity));
    this.events.push(new AddonAdded(this.id, addonId));
  }
  
  activate(paymentMethodId: PaymentMethodId): void {
    if (!this.canActivate()) {
      throw new InvalidStateTransitionError();
    }
    
    this.status = SubscriptionStatus.Active;
    this.paymentMethodId = paymentMethodId;
    this.events.push(new SubscriptionActivated(this.id));
  }
}
```

**Aggregate Sizing Rules:**
- **Too large:** Multiple unrelated entities forced into same transaction
- **Too small:** Can't enforce invariants that span entities
- **Right size:** Contains exactly what must be consistent together

**Cross-Aggregate References:**
```ruby
# ✅ CORRECT: Reference by ID
class Order
  def initialize(id, customer_id)
    @id = id
    @customer_id = customer_id  # ID reference
  end
end

# ❌ WRONG: Direct object reference
class Order
  def initialize(id, customer)
    @id = id
    @customer = customer  # Direct reference creates coupling
  end
end
```

---

### 4. Domain Events

**Definition:** Record of something significant that happened in the domain. Past-tense, immutable, and carries enough data for interested parties to react.

**Purpose:**
- **Decouple** aggregates and bounded contexts
- Enable **eventual consistency** across aggregates
- Support **audit trails** and event sourcing
- Trigger **side effects** (emails, notifications, integrations)

**Characteristics:**
- **Past tense naming** — `OrderPlaced`, `PaymentProcessed`, `SubscriptionCancelled`
- **Immutable** — once created, never changes
- **Contains context** — aggregate ID, timestamp, relevant data
- **Produced by aggregates** — not by services or handlers

**Implementation:**
```ruby
# Domain/Events/SubscriptionCancelled.{rb,php,ts}
class SubscriptionCancelled
  attr_reader :subscription_id, :reason, :occurred_at
  
  def initialize(subscription_id, reason, occurred_at = Time.current)
    @subscription_id = subscription_id
    @reason = reason
    @occurred_at = occurred_at
    freeze
  end
  
  def event_type
    'subscription.cancelled'
  end
end

# Aggregate emits event
class Subscription
  def cancel(reason)
    @status = SubscriptionStatus.cancelled
    @events << SubscriptionCancelled.new(@id, reason)
  end
  
  def release_events
    events = @events.dup
    @events.clear
    events
  end
end
```

```php
// Domain/Events/SubscriptionCancelled.{rb,php,ts}
final readonly class SubscriptionCancelled {
    public function __construct(
        public SubscriptionId $subscriptionId,
        public string $reason,
        public DateTimeImmutable $occurredAt
    ) {}
    
    public static function now(SubscriptionId $id, string $reason): self {
        return new self($id, $reason, new DateTimeImmutable());
    }
    
    public function eventType(): string {
        return 'subscription.cancelled';
    }
}

// Aggregate emits event
final class Subscription {
    private array $events = [];
    
    public function cancel(string $reason): void {
        $this->status = SubscriptionStatus::Cancelled;
        $this->events[] = SubscriptionCancelled::now($this->id, $reason);
    }
    
    public function releaseEvents(): array {
        $events = $this->events;
        $this->events = [];
        return $events;
    }
}
```

```typescript
// Domain/Events/SubscriptionCancelled.{rb,php,ts}
export class SubscriptionCancelled implements DomainEvent {
  constructor(
    public readonly subscriptionId: SubscriptionId,
    public readonly reason: string,
    public readonly occurredAt: Date = new Date()
  ) {
    Object.freeze(this);
  }
  
  get eventType(): string {
    return 'subscription.cancelled';
  }
}

// Aggregate emits event
export class Subscription {
  private events: DomainEvent[] = [];
  
  cancel(reason: string): void {
    this.status = SubscriptionStatus.Cancelled;
    this.events.push(new SubscriptionCancelled(this.id, reason));
  }
  
  releaseEvents(): DomainEvent[] {
    const events = [...this.events];
    this.events = [];
    return events;
  }
}
```

**Event Flow:**
1. **Aggregate** produces event during business operation
2. **Repository** persists aggregate + publishes events
3. **Event Dispatcher** notifies listeners
4. **Listeners** dispatch async jobs or update read models
5. **Jobs** execute side effects (emails, external APIs)

**Rules:**
- Events are **immutable** — use readonly/frozen
- Events are **facts** — can't be "undone", only compensated
- Events carry **sufficient data** — listeners shouldn't need to query aggregate
- NO business logic in event listeners — delegate to handlers/services

---

### 5. Domain Services

**Definition:** Stateless operations that don't naturally belong to a single entity or VO. Encapsulate domain logic that spans multiple aggregates.

**When to Use:**
- Logic involves multiple aggregates
- Behavior doesn't fit naturally on an entity
- Requires external domain knowledge or calculations
- Coordinating cross-aggregate workflows (within single transaction)

**When NOT to Use:**
- Could be an entity method
- Orchestrating use cases (that's Application layer)
- Calling external APIs (that's Infrastructure)

**Implementation:**
```ruby
# Domain/Services/PricingService.{rb,php,ts}
class PricingService
  def calculate_subscription_price(subscription, pricing_rules)
    base_price = pricing_rules.base_price_for(subscription.plan_id)
    
    addon_price = subscription.line_items.sum do |item|
      pricing_rules.addon_price_for(item.addon_id) * item.quantity
    end
    
    discount = apply_discount(subscription, base_price + addon_price)
    
    Money.new(base_price + addon_price - discount, 'USD')
  end
  
  private
  
  def apply_discount(subscription, total)
    return 0 unless subscription.discount_code
    # Domain logic for discount calculation
  end
end
```

```php
// Domain/Services/PricingService.{rb,php,ts}
final readonly class PricingService {
    public function calculateSubscriptionPrice(
        Subscription $subscription,
        PricingRules $rules
    ): Money {
        $basePrice = $rules->basePriceFor($subscription->planId());
        
        $addonPrice = array_reduce(
            $subscription->lineItems(),
            fn($sum, $item) => $sum + $rules->addonPriceFor($item->addonId()) 
                * $item->quantity(),
            0
        );
        
        $discount = $this->applyDiscount($subscription, $basePrice + $addonPrice);
        
        return new Money($basePrice + $addonPrice - $discount, Currency::USD);
    }
    
    private function applyDiscount(Subscription $sub, float $total): float {
        if (!$sub->hasDiscountCode()) {
            return 0;
        }
        // Domain logic for discount calculation
    }
}
```

```typescript
// Domain/Services/PricingService.{rb,php,ts}
export class PricingService {
  calculateSubscriptionPrice(
    subscription: Subscription,
    rules: PricingRules
  ): Money {
    const basePrice = rules.basePriceFor(subscription.planId);
    
    const addonPrice = subscription.lineItems.reduce(
      (sum, item) => sum + rules.addonPriceFor(item.addonId) * item.quantity,
      0
    );
    
    const discount = this.applyDiscount(subscription, basePrice + addonPrice);
    
    return Money.create(basePrice + addonPrice - discount, Currency.USD);
  }
  
  private applyDiscount(subscription: Subscription, total: number): number {
    if (!subscription.hasDiscountCode()) {
      return 0;
    }
    // Domain logic for discount calculation
  }
}
```

**Rules:**
- **Stateless** — no instance variables
- **Pure domain logic** — no framework dependencies
- **Inject via constructor** — when needed by other domain objects
- **Named after business capability** — `PricingService`, `ShippingCalculator`, not `SubscriptionManager`

---

### 6. Repositories

**Definition:** Abstraction for accessing and persisting aggregates. Hides persistence details from domain.

**Purpose:**
- **Aggregate lifecycle management** — retrieve, save, delete
- **Query interface** — find aggregates by business criteria
- **Transaction control** — manage consistency boundaries
- **Persistence abstraction** — domain doesn't know about database

**Pattern:**
```ruby
# Domain/Aggregates/Subscription/SubscriptionRepository.{rb,php,ts}
# Interface in Domain
module SubscriptionRepository
  def save(subscription)
  def find_by_id(subscription_id)
  def find_active_for_customer(customer_id)
  def execute_in_transaction(&block)
end

# Implementation in Infrastructure
class SubscriptionActiveRecordRepository
  def save(subscription)
    model = to_model(subscription)
    model.save!
    
    # Publish domain events
    subscription.release_events.each do |event|
      EventDispatcher.publish(event)
    end
    
    to_domain(model)
  end
  
  def find_by_id(subscription_id)
    model = SubscriptionModel.find_by(id: subscription_id.to_s)
    return nil unless model
    to_domain(model)
  end
  
  def execute_in_transaction(&block)
    ActiveRecord::Base.transaction { block.call }
  end
  
  private
  
  def to_domain(model)
    # Reconstitute aggregate from ORM model
    Subscription.reconstitute(
      id: SubscriptionId.new(model.id),
      customer_id: CustomerId.new(model.customer_id),
      # ... map all attributes
    )
  end
  
  def to_model(subscription)
    # Map aggregate to ORM model
    SubscriptionModel.find_or_initialize_by(id: subscription.id.to_s).tap do |model|
      model.customer_id = subscription.customer_id.to_s
      model.status = subscription.status.to_s
      # ... map all attributes
    end
  end
end
```

```php
// Domain/Aggregates/Subscription/SubscriptionRepository.{rb,php,ts}
// Interface in Domain
interface SubscriptionRepository {
    public function save(Subscription $subscription): Subscription;
    public function findById(SubscriptionId $id): ?Subscription;
    public function findActiveForCustomer(CustomerId $id): array;
    public function executeInTransaction(callable $fn): mixed;
}

// Implementation in Infrastructure
final class SubscriptionEloquentRepository implements SubscriptionRepository {
    public function save(Subscription $subscription): Subscription {
        $model = $this->toModel($subscription);
        $model->save();
        
        // Publish domain events
        foreach ($subscription->releaseEvents() as $event) {
            $this->eventDispatcher->publish($event);
        }
        
        return $this->toDomain($model);
    }
    
    public function findById(SubscriptionId $id): ?Subscription {
        $model = SubscriptionEloquent::find($id->toString());
        return $model ? $this->toDomain($model) : null;
    }
    
    public function executeInTransaction(callable $fn): mixed {
        return DB::transaction($fn);
    }
    
    private function toDomain(SubscriptionEloquent $model): Subscription {
        // Reconstitute aggregate
        return Subscription::reconstitute(
            id: new SubscriptionId($model->id),
            customerId: new CustomerId($model->customer_id),
            // ... map all attributes
        );
    }
    
    private function toModel(Subscription $sub): SubscriptionEloquent {
        // Map aggregate to Eloquent model
        return SubscriptionEloquent::findOrNew($sub->id()->toString())->fill([
            'customer_id' => $sub->customerId()->toString(),
            'status' => $sub->status()->value,
            // ... map all attributes
        ]);
    }
}
```

```typescript
// Domain/Aggregates/Subscription/SubscriptionRepository.{rb,php,ts}
// Interface in Domain
export interface ISubscriptionRepository {
  save(subscription: Subscription): Promise<Subscription>;
  findById(id: SubscriptionId): Promise<Subscription | null>;
  findActiveForCustomer(customerId: CustomerId): Promise<Subscription[]>;
  executeInTransaction<T>(fn: () => Promise<T>): Promise<T>;
}

// Implementation in Infrastructure
export class SubscriptionRepository implements ISubscriptionRepository {
  async save(subscription: Subscription): Promise<Subscription> {
    const entity = this.toEntity(subscription);
    await this.repo.save(entity);
    
    // Publish domain events
    for (const event of subscription.releaseEvents()) {
      await this.eventDispatcher.publish(event);
    }
    
    return this.toDomain(entity);
  }
  
  async findById(id: SubscriptionId): Promise<Subscription | null> {
    const entity = await this.repo.findOne({ where: { id: id.toString() } });
    return entity ? this.toDomain(entity) : null;
  }
  
  async executeInTransaction<T>(fn: () => Promise<T>): Promise<T> {
    return this.repo.manager.transaction(fn);
  }
  
  private toDomain(entity: SubscriptionEntity): Subscription {
    // Reconstitute aggregate
    return Subscription.reconstitute(
      new SubscriptionId(entity.id),
      new CustomerId(entity.customerId),
      // ... map all attributes
    );
  }
  
  private toEntity(sub: Subscription): SubscriptionEntity {
    // Map aggregate to TypeORM entity
    const entity = new SubscriptionEntity();
    entity.id = sub.id.toString();
    entity.customerId = sub.customerId.toString();
    entity.status = sub.status.value;
    // ... map all attributes
    return entity;
  }
}
```

**Repository Rules:**
- **One repository per aggregate root** — not per entity
- **Interface in Domain** — implementation in Infrastructure
- **Aggregate-level operations** — save/find whole aggregates, not individual entities
- **Collection-like interface** — think "in-memory collection" metaphor
- **Handle events** — publish domain events after successful save
- **Transaction support** — provide helper for multi-aggregate transactions

---

### 7. Factories

**Definition:** Encapsulate complex object creation logic. Useful when construction involves multiple steps, validation, or dependencies.

**When to Use:**
- Complex aggregate reconstitution from persistence
- Creation requires external dependencies
- Multiple creation methods (fromDTO, fromAPI, etc.)
- Invariant validation spans multiple objects

**Implementation:**
```ruby
# Domain/Aggregates/Subscription/SubscriptionFactory.{rb,php,ts}
class SubscriptionFactory
  def initialize(pricing_service)
    @pricing_service = pricing_service
  end
  
  def create_from_registration(customer_id, plan_id, addons)
    subscription = Subscription.new(
      SubscriptionId.generate,
      customer_id,
      plan_id
    )
    
    addons.each do |addon|
      subscription.add_addon(addon.id, addon.quantity)
    end
    
    # Complex logic involving domain service
    price = @pricing_service.calculate_subscription_price(
      subscription,
      PricingRules.current
    )
    
    subscription.set_price(price)
    subscription
  end
end
```

```php
// Domain/Aggregates/Subscription/SubscriptionFactory.{rb,php,ts}
final readonly class SubscriptionFactory {
    public function __construct(
        private PricingService $pricingService
    ) {}
    
    public function createFromRegistration(
        CustomerId $customerId,
        PlanId $planId,
        array $addons
    ): Subscription {
        $subscription = Subscription::create(
            SubscriptionId::generate(),
            $customerId,
            $planId
        );
        
        foreach ($addons as $addon) {
            $subscription->addAddon($addon->id, $addon->quantity);
        }
        
        // Complex logic involving domain service
        $price = $this->pricingService->calculateSubscriptionPrice(
            $subscription,
            PricingRules::current()
        );
        
        $subscription->setPrice($price);
        return $subscription;
    }
}
```

```typescript
// Domain/Aggregates/Subscription/SubscriptionFactory.{rb,php,ts}
export class SubscriptionFactory {
  constructor(private readonly pricingService: PricingService) {}
  
  createFromRegistration(
    customerId: CustomerId,
    planId: PlanId,
    addons: AddonSelection[]
  ): Subscription {
    const subscription = Subscription.create(
      SubscriptionId.generate(),
      customerId,
      planId
    );
    
    for (const addon of addons) {
      subscription.addAddon(addon.id, addon.quantity);
    }
    
    // Complex logic involving domain service
    const price = this.pricingService.calculateSubscriptionPrice(
      subscription,
      PricingRules.current()
    );
    
    subscription.setPrice(price);
    return subscription;
  }
}
```

---

### 8. Policies (Domain Authorization)

**Definition:** Encapsulate authorization and complex business rules that don't belong on aggregates. Answer "can this user/system perform this action?"

**Purpose:**
- **Separate authorization from business logic**
- **Reusable rules** across multiple use cases
- **Testable in isolation**
- **Explicit capability checking** before mutations

**Implementation:**
```ruby
# Domain/Policies/WorkOrderPolicy.{rb,php,ts}
class WorkOrderPolicy
  def can_approve?(work_order, user)
    return PolicyResult.denied("Work order already approved") if work_order.approved?
    return PolicyResult.denied("Insufficient permissions") unless user.can_approve_work_orders?
    return PolicyResult.denied("Amount exceeds approval limit") if exceeds_limit?(work_order, user)
    
    PolicyResult.allowed
  end
  
  def can_edit?(work_order, user)
    return PolicyResult.denied("Cannot edit approved work order") if work_order.approved?
    return PolicyResult.denied("Not the owner") unless work_order.owned_by?(user.id)
    
    PolicyResult.allowed
  end
  
  private
  
  def exceeds_limit?(work_order, user)
    work_order.total_amount > user.approval_limit
  end
end

# PolicyResult value object
class PolicyResult
  attr_reader :allowed, :reason
  
  def self.allowed
    new(true, nil)
  end
  
  def self.denied(reason)
    new(false, reason)
  end
  
  def allowed?
    @allowed
  end
end
```

```php
// Domain/Policies/WorkOrderPolicy.{rb,php,ts}
final readonly class WorkOrderPolicy {
    public function canApprove(WorkOrder $workOrder, User $user): PolicyResult {
        if ($workOrder->isApproved()) {
            return PolicyResult::denied("Work order already approved");
        }
        
        if (!$user->canApproveWorkOrders()) {
            return PolicyResult::denied("Insufficient permissions");
        }
        
        if ($this->exceedsLimit($workOrder, $user)) {
            return PolicyResult::denied("Amount exceeds approval limit");
        }
        
        return PolicyResult::allowed();
    }
    
    public function canEdit(WorkOrder $workOrder, User $user): PolicyResult {
        if ($workOrder->isApproved()) {
            return PolicyResult::denied("Cannot edit approved work order");
        }
        
        if (!$workOrder->isOwnedBy($user->id())) {
            return PolicyResult::denied("Not the owner");
        }
        
        return PolicyResult::allowed();
    }
    
    private function exceedsLimit(WorkOrder $wo, User $user): bool {
        return $wo->totalAmount()->greaterThan($user->approvalLimit());
    }
}

// PolicyResult value object
final readonly class PolicyResult {
    private function __construct(
        public bool $allowed,
        public ?string $reason
    ) {}
    
    public static function allowed(): self {
        return new self(true, null);
    }
    
    public static function denied(string $reason): self {
        return new self(false, $reason);
    }
}
```

```typescript
// Domain/Policies/WorkOrderPolicy.{rb,php,ts}
export class WorkOrderPolicy {
  canApprove(workOrder: WorkOrder, user: User): PolicyResult {
    if (workOrder.isApproved()) {
      return PolicyResult.denied("Work order already approved");
    }
    
    if (!user.canApproveWorkOrders()) {
      return PolicyResult.denied("Insufficient permissions");
    }
    
    if (this.exceedsLimit(workOrder, user)) {
      return PolicyResult.denied("Amount exceeds approval limit");
    }
    
    return PolicyResult.allowed();
  }
  
  canEdit(workOrder: WorkOrder, user: User): PolicyResult {
    if (workOrder.isApproved()) {
      return PolicyResult.denied("Cannot edit approved work order");
    }
    
    if (!workOrder.isOwnedBy(user.id)) {
      return PolicyResult.denied("Not the owner");
    }
    
    return PolicyResult.allowed();
  }
  
  private exceedsLimit(workOrder: WorkOrder, user: User): boolean {
    return workOrder.totalAmount.greaterThan(user.approvalLimit);
  }
}

// PolicyResult value object
export class PolicyResult {
  private constructor(
    public readonly allowed: boolean,
    public readonly reason: string | null
  ) {
    Object.freeze(this);
  }
  
  static allowed(): PolicyResult {
    return new PolicyResult(true, null);
  }
  
  static denied(reason: string): PolicyResult {
    return new PolicyResult(false, reason);
  }
}
```

**Usage in Application Handler:**
```ruby
# Application/Commands/ApproveWorkOrder/ApproveWorkOrderHandler.{rb,php,ts}
class ApproveWorkOrderHandler
  def initialize(repository, policy)
    @repository = repository
    @policy = policy
  end
  
  def handle(command)
    work_order = @repository.find_by_id(command.work_order_id)
    user = command.user
    
    # Check policy before mutation
    result = @policy.can_approve?(work_order, user)
    raise UnauthorizedException.new(result.reason) unless result.allowed?
    
    # Execute business operation
    work_order.approve(user.id)
    
    @repository.save(work_order)
  end
end
```

**Policy Rules:**
- Lives in **Domain** layer
- Returns **PolicyResult** (not boolean) to provide reasons
- NO framework dependencies
- Invoked by **Application** handlers before mutations
- Can access domain concepts (aggregates, user roles)

---

## 🏗️ Strategic Design Patterns

### Bounded Contexts

**Definition:** Explicit boundary within which a domain model is defined and applicable. Different contexts can have different models for the same business concept.

**Purpose:**
- **Manage complexity** by dividing large systems
- **Enable autonomy** for teams
- **Isolate models** that serve different purposes
- **Allow evolution** independently

**Example Contexts:**
```
E-commerce System:
├── Sales Context
│   ├── Order (with pricing, discounts, fulfillment)
│   └── Customer (with billing address, payment methods)
│
├── Inventory Context
│   ├── Product (with stock levels, warehouses)
│   └── Reservation (with allocation rules)
│
├── Shipping Context
│   ├── Shipment (with tracking, carrier details)
│   └── Address (with validation, geocoding)
│
└── Billing Context
    ├── Invoice (with line items, taxes)
    └── Customer (with credit terms, payment history)
```

**Key Insights:**
- "Customer" means different things in Sales vs Billing contexts
- Each context has its own model, database, and codebase
- Models are optimized for their specific use case

**Context Mapping:**

| Pattern | Description | When to Use |
|---------|-------------|-------------|
| **Shared Kernel** | Two contexts share subset of domain model | Small, stable shared concepts between tight teams |
| **Customer/Supplier** | Downstream depends on upstream; upstream defines API | Clear power dynamic; upstream serves downstream needs |
| **Conformist** | Downstream conforms to upstream model | Upstream won't change; downstream has no negotiation power |
| **Anti-Corruption Layer (ACL)** | Translation layer protects downstream from upstream | Upstream model is poor fit; need to shield domain |
| **Open Host Service** | Well-defined protocol for integration | Multiple consumers; need stable public API |
| **Published Language** | Standardized integration language (e.g., JSON schema) | Industry standards; multiple systems integrate |
| **Separate Ways** | No integration; duplicate functionality | Integration cost > duplication cost |

**Implementation Patterns:**

**Anti-Corruption Layer (ACL):**
```ruby
# Infrastructure/Integration/Stripe/StripeAdapter.{rb,php,ts}
class StripeAdapter
  def fetch_customer(stripe_customer_id)
    # Call Stripe API
    stripe_customer = Stripe::Customer.retrieve(stripe_customer_id)
    
    # Translate to domain model
    Customer.new(
      id: CustomerId.new(stripe_customer.id),
      email: EmailAddress.new(stripe_customer.email),
      name: CustomerName.new(stripe_customer.name)
      # Map only what we need, ignore Stripe-specific details
    )
  end
  
  def create_subscription(subscription)
    # Translate domain model to Stripe API format
    Stripe::Subscription.create(
      customer: subscription.customer_id.to_s,
      items: subscription.line_items.map { |item| translate_line_item(item) }
      # Only send what Stripe needs
    )
  end
  
  private
  
  def translate_line_item(item)
    { price: item.price_id.to_s, quantity: item.quantity }
  end
end
```

```php
// Infrastructure/Integration/Stripe/StripeAdapter.{rb,php,ts}
final readonly class StripeAdapter {
    public function fetchCustomer(string $stripeCustomerId): Customer {
        // Call Stripe API
        $stripeCustomer = \Stripe\Customer::retrieve($stripeCustomerId);
        
        // Translate to domain model
        return new Customer(
            id: new CustomerId($stripeCustomer->id),
            email: new EmailAddress($stripeCustomer->email),
            name: new CustomerName($stripeCustomer->name)
            // Map only what we need, ignore Stripe-specific details
        );
    }
    
    public function createSubscription(Subscription $subscription): string {
        // Translate domain model to Stripe API format
        $stripeSubscription = \Stripe\Subscription::create([
            'customer' => $subscription->customerId()->toString(),
            'items' => array_map(
                fn($item) => $this->translateLineItem($item),
                $subscription->lineItems()
            )
        ]);
        
        return $stripeSubscription->id;
    }
    
    private function translateLineItem(SubscriptionLineItem $item): array {
        return [
            'price' => $item->priceId()->toString(),
            'quantity' => $item->quantity()
        ];
    }
}
```

```typescript
// Infrastructure/Integration/Stripe/StripeAdapter.{rb,php,ts}
export class StripeAdapter {
  async fetchCustomer(stripeCustomerId: string): Promise<Customer> {
    // Call Stripe API
    const stripeCustomer = await stripe.customers.retrieve(stripeCustomerId);
    
    // Translate to domain model
    return new Customer(
      new CustomerId(stripeCustomer.id),
      new EmailAddress(stripeCustomer.email),
      new CustomerName(stripeCustomer.name)
      // Map only what we need, ignore Stripe-specific details
    );
  }
  
  async createSubscription(subscription: Subscription): Promise<string> {
    // Translate domain model to Stripe API format
    const stripeSubscription = await stripe.subscriptions.create({
      customer: subscription.customerId.toString(),
      items: subscription.lineItems.map(item => this.translateLineItem(item))
    });
    
    return stripeSubscription.id;
  }
  
  private translateLineItem(item: SubscriptionLineItem): Stripe.SubscriptionCreateParams.Item {
    return {
      price: item.priceId.toString(),
      quantity: item.quantity
    };
  }
}
```

**ACL Rules:**
- Lives in **Infrastructure** layer
- Translates between external models and domain models
- Protects domain from external API changes
- Never exposes external types to domain

---

### Ubiquitous Language

**Definition:** Shared language between developers and domain experts. Terms used in code match terms used by the business.

**Implementation Strategy:**

**Bad Example (Technical Language):**
```ruby
# ❌ Technical terms that don't match business language
class OrderProcessor
  def process_transaction(data)
    entity = OrderEntity.new
    entity.set_status("PENDING")
    entity.persist
  end
end
```

**Good Example (Ubiquitous Language):**
```ruby
# ✅ Business terms that domain experts use
class Order
  def place
    raise OrderAlreadyPlacedException if placed?
    @status = OrderStatus.placed
    @events << OrderPlaced.new(@id)
  end
end
```

**Language Evolution:**
- Document terms in **Domain/README.md** glossary
- Update code when business language changes
- Hold "model exploration workshops" with domain experts
- Reject technical jargon in favor of business terms

**Example Glossary:**
```markdown
# Domain Glossary

**Subscription**: A recurring agreement between customer and company for service delivery.
  - States: Trial, Active, Paused, Cancelled, Expired
  - Invariant: Cannot add addons after cancellation
  
**Work Order**: Request for service delivery at a property.
  - States: Draft, Submitted, Approved, Scheduled, Completed, Rejected
  - Invariant: Requires approval before scheduling if amount > $5000
  
**Invoice**: Bill for services rendered, sent to customer.
  - States: Draft, Sent, Paid, Overdue, Void
  - Invariant: Cannot void paid invoices
```

---

### Context Integration Patterns

**1. Domain Events (Eventual Consistency)**

Best for: Loose coupling, async workflows, cross-context communication

```ruby
# Subscription Context produces event
class Subscription
  def activate
    @status = SubscriptionStatus.active
    @events << SubscriptionActivated.new(@id, @customer_id)
  end
end

# Billing Context listens and reacts
class SubscriptionActivatedListener
  def handle(event)
    CreateInvoiceJob.perform_later(event.customer_id, event.subscription_id)
  end
end
```

**2. Integration API (Synchronous)**

Best for: Read operations, immediate consistency needs

```ruby
# Infrastructure/Integration/SubscriptionApi.{rb,php,ts}
class SubscriptionApi
  def get_active_subscription(customer_id)
    query = GetActiveSubscriptionQuery.new(customer_id)
    subscription = @query_handler.handle(query)
    
    # Return external DTO
    SubscriptionProfileDTO.from_domain(subscription)
  end
end

# Usage from another context
class InvoiceFactory
  def initialize(subscription_api)
    @subscription_api = subscription_api
  end
  
  def create_for_customer(customer_id)
    subscription = @subscription_api.get_active_subscription(customer_id)
    # Use subscription data to create invoice
  end
end
```

---

## 🎨 Application Layer Patterns

### Commands & Queries (CQRS)

**Definition:** Separate read operations (Queries) from write operations (Commands).

**Benefits:**
- **Optimized models** — reads use different model than writes
- **Scalability** — scale reads and writes independently
- **Clarity** — intent is explicit (am I reading or writing?)
- **Security** — different permissions for reads vs writes

**Command Pattern:**
```ruby
# Application/Commands/CreateSubscription/CreateSubscriptionCommand.{rb,php,ts}
class CreateSubscriptionCommand
  attr_reader :customer_id, :plan_id, :addon_ids, :payment_method_id
  
  def initialize(customer_id, plan_id, addon_ids, payment_method_id)
    @customer_id = customer_id
    @plan_id = plan_id
    @addon_ids = addon_ids
    @payment_method_id = payment_method_id
    validate!
  end
  
  def self.from_request(params)
    new(
      CustomerId.new(params[:customer_id]),
      PlanId.new(params[:plan_id]),
      params[:addon_ids]&.map { |id| AddonId.new(id) } || [],
      PaymentMethodId.new(params[:payment_method_id])
    )
  end
  
  private
  
  def validate!
    raise ValidationError, "Customer ID required" if @customer_id.nil?
    raise ValidationError, "Plan ID required" if @plan_id.nil?
    raise ValidationError, "Payment method required" if @payment_method_id.nil?
  end
end

# Application/Commands/CreateSubscription/CreateSubscriptionHandler.{rb,php,ts}
class CreateSubscriptionHandler
  def initialize(repository, factory, policy)
    @repository = repository
    @factory = factory
    @policy = policy
  end
  
  def handle(command)
    # Authorization
    result = @policy.can_create_subscription?(command.customer_id)
    raise UnauthorizedException.new(result.reason) unless result.allowed?
    
    # Business operation
    subscription = @factory.create(
      command.customer_id,
      command.plan_id,
      command.addon_ids
    )
    
    subscription.set_payment_method(command.payment_method_id)
    
    # Persistence
    @repository.execute_in_transaction do
      @repository.save(subscription)
    end
    
    # Return result
    CreateSubscriptionResult.new(subscription.id)
  end
end
```

**Query Pattern:**
```ruby
# Application/Queries/GetSubscriptionDetails/GetSubscriptionDetailsQuery.{rb,php,ts}
class GetSubscriptionDetailsQuery
  attr_reader :subscription_id, :requesting_user_id
  
  def initialize(subscription_id, requesting_user_id)
    @subscription_id = subscription_id
    @requesting_user_id = requesting_user_id
  end
end

# Application/Queries/GetSubscriptionDetails/GetSubscriptionDetailsHandler.{rb,php,ts}
class GetSubscriptionDetailsHandler
  def initialize(repository, policy)
    @repository = repository
    @policy = policy
  end
  
  def handle(query)
    subscription = @repository.find_by_id(query.subscription_id)
    raise SubscriptionNotFoundException unless subscription
    
    # Authorization check
    result = @policy.can_view?(subscription, query.requesting_user_id)
    raise UnauthorizedException.new(result.reason) unless result.allowed?
    
    # Return domain object or DTO
    subscription
  end
end
```

**CQRS Rules:**
- **Commands** change state, return minimal data (ID or Result DTO)
- **Queries** read state, never modify
- Separate models for reads (optimized projections) and writes (aggregates)
- Commands are task-based: `PlaceOrder`, not `UpdateOrderStatus`
- Queries can bypass domain layer for performance (read from projections/views)

---

### Application Services vs Domain Services

**Application Service (Handler):**
- **Orchestrates** use case workflow
- Coordinates multiple aggregates/services
- Manages transactions
- Enforces authorization
- Lives in **Application** layer

**Domain Service:**
- **Encapsulates** domain logic that doesn't fit on entities
- Pure business rules
- Stateless operations
- Lives in **Domain** layer

**Example:**
```ruby
# Domain Service (pure business logic)
class PricingService
  def calculate_total(order)
    subtotal = order.items.sum(&:price)
    discount = calculate_discount(order, subtotal)
    tax = calculate_tax(subtotal - discount, order.shipping_address)
    subtotal - discount + tax
  end
end

# Application Handler (orchestration)
class PlaceOrderHandler
  def initialize(order_repo, inventory_repo, pricing_service)
    @order_repo = order_repo
    @inventory_repo = inventory_repo
    @pricing_service = pricing_service
  end
  
  def handle(command)
    # Transaction boundary
    @order_repo.execute_in_transaction do
      # Load aggregates
      order = @order_repo.find_by_id(command.order_id)
      
      # Reserve inventory (cross-aggregate coordination)
      order.items.each do |item|
        @inventory_repo.reserve(item.product_id, item.quantity)
      end
      
      # Domain service for complex calculation
      total = @pricing_service.calculate_total(order)
      order.set_total(total)
      
      # Execute business operation
      order.place
      
      # Persist
      @order_repo.save(order)
    end
  end
end
```

---

## 🛡️ Maintaining Boundaries

### Dependency Injection

**Purpose:** Decouple layers and enable testing

**Pattern:**
```ruby
# Domain defines interface
module SubscriptionRepository
  def save(subscription)
  def find_by_id(id)
end

# Infrastructure implements
class SubscriptionActiveRecordRepository
  include SubscriptionRepository
  # implementation
end

# Application depends on interface
class CreateSubscriptionHandler
  def initialize(repository: SubscriptionRepository)
    @repository = repository
  end
end

# Wire up in service provider/container
container.register(:subscription_repository, SubscriptionActiveRecordRepository.new)
container.register(:create_subscription_handler) do
  CreateSubscriptionHandler.new(
    repository: container.resolve(:subscription_repository)
  )
end
```

---

### Testing Strategy

**Unit Tests (Domain Layer):**
```ruby
# tests/Unit/Domain/Subscription_test.{rb,php,ts}
RSpec.describe Subscription do
  describe '#activate' do
    it 'transitions from trial to active' do
      subscription = Subscription.new(id, customer_id, plan_id)
      subscription.activate(payment_method_id)
      
      expect(subscription.status).to eq(SubscriptionStatus.active)
    end
    
    it 'emits SubscriptionActivated event' do
      subscription = Subscription.new(id, customer_id, plan_id)
      subscription.activate(payment_method_id)
      
      events = subscription.release_events
      expect(events).to include(an_instance_of(SubscriptionActivated))
    end
    
    it 'raises error when already cancelled' do
      subscription = Subscription.new(id, customer_id, plan_id)
      subscription.cancel("Customer request")
      
      expect { subscription.activate(payment_method_id) }
        .to raise_error(InvalidStateTransitionException)
    end
  end
end
```

**Integration Tests (Application Layer):**
```ruby
# tests/Integration/Application/CreateSubscriptionHandler_test.{rb,php,ts}
RSpec.describe CreateSubscriptionHandler do
  it 'creates subscription and persists to database' do
    command = CreateSubscriptionCommand.new(customer_id, plan_id, [], payment_method_id)
    
    result = handler.handle(command)
    
    # Verify persistence
    subscription = repository.find_by_id(result.subscription_id)
    expect(subscription).not_to be_nil
    expect(subscription.status).to eq(SubscriptionStatus.trial)
  end
  
  it 'publishes SubscriptionCreated event' do
    command = CreateSubscriptionCommand.new(customer_id, plan_id, [], payment_method_id)
    
    expect { handler.handle(command) }
      .to have_published_event(SubscriptionCreated)
  end
  
  it 'raises UnauthorizedException when policy denies' do
    # Setup: customer has active subscription
    existing_subscription = create_active_subscription(customer_id)
    
    command = CreateSubscriptionCommand.new(customer_id, plan_id, [], payment_method_id)
    
    expect { handler.handle(command) }
      .to raise_error(UnauthorizedException)
  end
end
```

**Test Pyramid:**
```
         /\
        /  \  E2E Tests (few, slow, expensive)
       /----\
      /      \ Integration Tests (some, medium)
     /--------\
    /          \ Unit Tests (many, fast, cheap)
   /------------\
```

**Testing Rules:**
- **Unit tests** — Domain layer, no framework, no database
- **Integration tests** — Application layer, test handlers with real repositories
- **E2E tests** — HTTP layer, full stack through controllers
- **Mock external dependencies** — payment gateways, email services
- **Don't mock domain objects** — test real aggregates/VOs

---

## 🚨 Common DDD Anti-Patterns

### Anemic Domain Model
**Problem:** Entities are just data containers with getters/setters. Business logic lives in services.

```ruby
# ❌ WRONG: Anemic model
class Order
  attr_accessor :status, :items, :total
end

class OrderService
  def place_order(order)
    # Validation
    raise "Empty order" if order.items.empty?
    
    # Business logic in service
    order.status = "PLACED"
    order.total = calculate_total(order.items)
    order.save
  end
end

# ✅ CORRECT: Rich domain model
class Order
  def place
    raise EmptyOrderError if @items.empty?
    
    @status = OrderStatus.placed
    @total = calculate_total
    @events << OrderPlaced.new(@id)
  end
  
  private
  
  def calculate_total
    @items.sum(&:price)
  end
end
```

---

### God Aggregate
**Problem:** Aggregate tries to do everything, becomes massive and slow.

```ruby
# ❌ WRONG: God aggregate
class Customer
  attr_reader :orders, :subscriptions, :invoices, :payments, :support_tickets
  # Too many responsibilities
end

# ✅ CORRECT: Focused aggregates
class Customer
  attr_reader :id, :email, :billing_address
  # Just customer identity and core attributes
end

class Order
  attr_reader :id, :customer_id  # Reference customer by ID
  # Order-specific logic
end

class Subscription
  attr_reader :id, :customer_id  # Reference customer by ID
  # Subscription-specific logic
end
```

---

### Leaking Domain Logic
**Problem:** Business rules duplicated in controllers, services, or database.

```ruby
# ❌ WRONG: Logic in controller
class OrdersController
  def place
    order = Order.find(params[:id])
    
    # Business logic in controller
    if order.items.empty?
      return render json: { error: "Cannot place empty order" }, status: 422
    end
    
    order.update(status: "PLACED")
    render json: order
  end
end

# ✅ CORRECT: Logic in domain
class OrdersController
  def place
    command = PlaceOrderCommand.new(params[:id])
    result = @handler.handle(command)  # Handler delegates to aggregate
    render json: OrderResource.new(result)
  end
end

class Order
  def place
    raise EmptyOrderError if @items.empty?
    @status = OrderStatus.placed
  end
end
```

---

### Transaction Script Pattern
**Problem:** Each use case is a procedure that operates on data, not domain model.

```ruby
# ❌ WRONG: Transaction script
def place_order(order_id)
  # Just database operations and procedures
  order = OrderModel.find(order_id)
  order.status = "PLACED"
  order.placed_at = Time.now
  order.save
  
  OrderMailer.confirmation(order).deliver_later
  AnalyticsService.track("order_placed", order.id)
end

# ✅ CORRECT: Domain-driven
class PlaceOrderHandler
  def handle(command)
    order = @repository.find_by_id(command.order_id)
    
    # Rich domain behavior
    order.place  # Encapsulates rules, emits events
    
    @repository.save(order)  # Events trigger side effects
  end
end
```

---

## ✅ DDD Code Review Checklist

### Domain Layer
- [ ] **Ubiquitous Language** used in all class/method names
- [ ] **No framework dependencies** (Rails, Laravel, Express)
- [ ] **Rich behavior** — entities have methods, not just getters/setters
- [ ] **Value Objects** for concepts (not primitive obsession)
- [ ] **Aggregates** enforce invariants and emit events
- [ ] **Repository interfaces** only (no implementations)
- [ ] **Domain events** are immutable and past-tense
- [ ] **Policies** return PolicyResult with reasons
- [ ] **No persistence logic** in domain objects

### Application Layer
- [ ] **Commands/Queries** are explicit and validated
- [ ] **Handlers** orchestrate, don't contain business logic
- [ ] **Authorization** via domain policies before mutations
- [ ] **Transactions** explicitly managed
- [ ] **Cross-aggregate** coordination is minimal
- [ ] **No direct ORM** usage (only via repository interfaces)
- [ ] **Returns** domain objects or result DTOs

### Infrastructure Layer
- [ ] **Controllers** are thin (< 15 lines)
- [ ] **Repositories** map Domain ↔ ORM
- [ ] **Events published** after successful persistence
- [ ] **ACL** translates external APIs to domain models
- [ ] **No business logic** in controllers/jobs/listeners
- [ ] **Resources/Serializers** format responses

### General
- [ ] **Boundaries respected** — layers depend correctly
- [ ] **Aggregates sized** appropriately (not too big/small)
- [ ] **Consistency** is transactional within aggregate, eventual across
- [ ] **Tests** focus on behavior, not implementation
- [ ] **Glossary** updated when language changes

---

## 💡 AI Assistant DDD Guidelines

### When Generating Code

**Always Ask:**
1. Is this **core business logic** (Domain) or **orchestration** (Application) or **adapter** (Infrastructure)?
2. Does this name match the **Ubiquitous Language**?
3. Should this be an **Entity**, **Value Object**, or **Domain Service**?
4. Where is the **consistency boundary** (aggregate)?
5. Is this a **Command** (write) or **Query** (read)?

### Decision Trees

**Where does business logic go?**
```
Is it a validation/calculation for a single concept?
  └─ YES → Value Object constructor

Does it involve state and identity?
  └─ YES → Entity method or Aggregate method

Does it span multiple aggregates?
  └─ YES → Domain Service

Is it orchestration of use case?
  └─ YES → Application Handler

Is it authorization?
  └─ YES → Domain Policy
```

**How do I model this?**
```
Does it have identity that persists?
  └─ YES → Entity (possibly Aggregate Root)
  └─ NO → Value Object

Does it cluster related entities?
  └─ YES → Aggregate

Does it represent something that happened?
  └─ YES → Domain Event

Does it encapsulate a choice/permission?
  └─ YES → Policy
```

### Red Flags in Code Review
- Entity with only getters/setters (anemic model)
- Business logic in controllers/jobs/listeners
- Aggregates referencing other aggregates by object (should be ID)
- Setters on aggregates (use expressive methods)
- Value Objects that are mutable
- Domain layer importing framework code
- Multiple aggregates modified in single transaction
- Commands with generic names (`UpdateOrder` instead of `PlaceOrder`)

### Green Flags
- Rich domain behavior with expressive method names
- Clear aggregate boundaries
- Immutable value objects with self-validation
- Domain events for significant state changes
- Thin controllers that delegate to handlers
- Repository interfaces in Domain, implementations in Infrastructure
- Explicit transaction boundaries in Application layer
- Ubiquitous Language throughout codebase