# ✅ Value Objects (Billing / Money Domain)
**Path:** `.cursor/rules/domain/value-objects.mdc`

Value Objects represent **what something is**, not **how it changes**. They are immutable, self-validating, and always valid.

---
## 🎯 Goals
- Improve correctness by using **rich types instead of primitives** (e.g., `Money`, `Currency`, `Percentage`).
- Reduce validation clutter in the domain.
- Make illegal states **unrepresentable**.

> A primitive (int/string) only holds data. A Value Object holds *meaning*.

---
## 🧠 Core Rules
- Immutable (`readonly`, no setters)
- Validated on construction (never allow invalid state)
- Equality by **value**, not identity
- Contain **behavior**, not just data

---
## 📦 Value Object Structure (Pseudocode)
```pseudo
class Money:
  readonly amount
  readonly currency

  constructor(amount, currency):
    if amount < 0 → throw

  method add(other): return Money(amount + other.amount)
  method multiply(rate): return Money(amount * rate)
```

---
## ✅ Example (PHP: Money Value Object)
```php
final class Money
{
    public function __construct(
        public readonly int $amountInCents,
        public readonly Currency $currency,
    ) {
        if ($amountInCents < 0) {
            throw new InvalidMoneyValue($amountInCents);
        }
    }

    public function add(self $other): self
    {
        $this->assertSameCurrency($other);
        return new self($this->amountInCents + $other->amountInCents, $this->currency);
    }

    private function assertSameCurrency(self $other): void
    {
        if (!$other->currency->equals($this->currency)) {
            throw CurrencyMismatch::between($this->currency, $other->currency);
        }
    }
}
```

---
## 🔁 Always Return a New Instance
Value Objects **never mutate**:
```pseudo
wrong: money.add(another) changes current
correct: return new Money(amount + other.amount)
```

---
## 🧪 Testing Value Objects
```pseudo
Given $money = Money(100, USD)
When add(Money(50, USD))
Then result.amount == 150
```

Unit test only the **rules**, not persistence.

---
## 🚫 Anti-Patterns
❌ Using primitives: `int $price`
✅ Use a Value Object: `Money $price`

❌ Public setters or mutation methods
✅ Recreate a new instance instead

❌ Duplicating validation across aggregates
✅ Validation lives inside the Value Object

---
## 🤖 AI Assistant Guidelines
When generating Value Objects:
- Think: *“Is this a domain concept or just data?”*
- Prefer VOs over primitives when domain meaning exists
- Validate in constructor and fail early
- Provide **behavior** (formatting, math, normalization)

Decision tree:
```pseudo
IF validation + meaning + operations → Value Object
ELSE → primitive
```

---
## ✅ Review Checklist
- Immutable (`readonly`)
- Validates input at creation
- No setters or mutation
- Contains behavior relevant to domain
- Equality based on **value**, n