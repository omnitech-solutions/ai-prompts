# 🐘 PHP Standards — Modern + DDD Aligned
**Path:** `.cursor/rules/core/php-standards.mdc`

Modern, type‑safe PHP 8.2+ conventions aligned with DDD and clean layered architecture.

---
## ✅ Language Foundations
- All files start with `declare(strict_types=1);`
- Require PHP 8.2+ features (readonly, enums, intersection types)
- Prefer constructor property promotion
- Typed properties **everywhere** (no untyped members)
- Follow PSR‑12 formatting

---
## 🔒 Type Safety
- Avoid `mixed` (only for boundary integration)
- Use array shapes when arrays are unavoidable: `array{id: string, qty: int}`
- Prefer explicit return types instead of `void`
- Document union types and handle branches explicitly

```php
function fetch(OfferingId|ExternalId $id): Offering { /* ... */ }
```

---
## 🧠 PHP 8.2 Feature Usage
### Enums
Use enums for fixed domain concepts.
```php
enum Status: string { case ACTIVE = 'active'; case DRAFT = 'draft'; }
```

### match (instead of switch)
```php
$label = match ($status) {
  Status::ACTIVE => 'Active',
  Status::DRAFT => 'Draft',
};
```

---
## 🚨 Error Handling
- Throw **specific domain exceptions**, not `Exception`
- Exception classes expose static named constructors

```php
final class OfferingNotFound extends DomainException {
  public static function withId(string $id): self {
    return new self("Offering not found: {$id}");
  }
}
```

---
## 🧾 Documentation Rules
- Public methods require PHPDoc when:
  - using array shapes
  - throwing exceptions
  - representing business rules

```php
/** @throws OfferingAlreadyActive */
public function activate(): void { /* ... */ }
```

---
## 🏛️ Structure & Class Organization
- One class per file
- Namespace follows directory structure
- Prefer composition > inheritance
- Method order: **public → protected → private**

**Aggregate skeleton
```php
final class Subscription extends AggregateRoot
{
    private function __construct(
        private SubscriptionId $id,
        private SubscriptionStatus $status,
        private CustomerId $customerId,
        private array $items,
    ) {}

    public static function start(CustomerId $customerId, array $items): self
    {
        if (empty($items)) {
            throw CannotStartSubscription::emptyItems();
        }

        return new self(
            id: SubscriptionId::new(),
            status: SubscriptionStatus::pending(),
            customerId: $customerId,
            items: $items,
        );
    }

    public function activate(): void
    {
        if ($this->status->isActive()) {
            throw SubscriptionAlreadyActive::for($this->id);
        }

        $this->status = SubscriptionStatus::active();
        $this->recordEvent(SubscriptionActivated::from($this));
    }

    public function isActive(): bool
    {
        return $this->status->isActive();
    }
}
```php
final class SubscriptionOffering {
  private function __construct(/* … */) {}
  public static function create(/* … */): self { /* … */ }
  public function activate(): void { /* business rule */ }
  public function isActive(): bool { /* query */ }
}
```

---
## 🧺 Collections, Arrays & Null Handling
- Avoid raw arrays when representing domain concepts
- Prefer typed collections or Value Objects
- Use `??`, `??=` for null handling

```php
$name = $this->customName ?? $this->generateDefaultName();
```

---
## 🚦 Performance
- Use generators for large datasets
- Favor early returns
- Use strict comparisons (`===`)

```php
/** @return \Generator<Offering> */
public function streamActive(): \Generator { /* ... */ }
```

---
## 🪝 Layer Alignment (DDD)
- **Domain:** business naming, invariants, immutability
- **Application:** orchestration only
- **Infrastructure:** adapters, ORM, HTTP

---
## 🚫 Anti‑Patterns
- Public mutable properties on domain objects
- Static methods on domain models (`::update()`, `::activate()`) — use factories or handlers instead
- Untyped arrays returned from domain code
- Blanket suppressions (`@phpstan-ignore-*`)
- Magic methods (`__get`, `__call`) in domain code

---
## ✅ Quick Checklist
- File begins with `declare(strict_types=1);`
- Uses PHP 8.2 features where appropriate (readonly, enums, match)
- No untyped properties; return types specified
- No public mutable properties in domain code
- No generic Exception; use domain-specific exceptions
- No raw arrays for domain concepts; use Value Objects or typed collections
- Domain is framework‑agnostic — no Eloquent, no HTTP, no infrastructure imports

---

