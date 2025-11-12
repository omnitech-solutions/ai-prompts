# 🏗️ Code Quality & Domain‑Driven Design Standards

**Path:** `.cursor/rules/core/code-quality-ddd-standards.mdc`

Rules that enforce **clarity, maintainability, testability, and scalability** across backend systems. Applies to all stacks (PHP, TypeScript, Ruby).

---

## 🚩 Why This Exists

- Build software that is **easy to reason about** and evolve.
- Maintain separation between **business logic** and **technical implementation**.
- Promote **small, cohesive units** and strict boundaries between layers.

> **Architecture should make the correct decision the easiest option.**

---

## 🎯 Core Principles

### 🗣️ Ubiquitous Language

Rule: Domain terms must match code, documentation, and tests.

- ✅ `activateSubscription()`
- ❌ `setSubscriptionStatus()`

### 🏗️ Layered Architecture

```
Domain  ←  Application  ←  Infrastructure
   ↑───────────────↑
   | business rules |
```

- **Domain:** Business logic, invariants, entities, value objects.
- **Application:** Use‑case orchestration, transactions, policies.
- **Infrastructure:** HTTP, DB, frameworks, adapters.

### 🧠 Behavior Over Data

- Aggregates expose **business behavior**, not setters/getters.
- Methods must describe actions: `activate()`, `changePlan()`.

---

## ⚙️ Development Standards

### ✍️ Code Quality

```php
// ✅ Expressive domain behavior
public function activate(): void
{
    if ($this->isActive()) {
        throw SubscriptionAlreadyActive::for($this->id);
    }

    $this->status = Status::ACTIVE;
    $this->recordEvent(SubscriptionActivated::from($this));
}

// ❌ Anemic model (data mutation only)
public function setStatus(string $status): void
{
    $this->status = $status;
}
```

### 📐 Structure & Method Rules

- Target **15–20 lines** max.
- One visual indentation level.
- Prefer **early returns** to nested conditions.

---

## 🔒 Type Safety

```php
declare(strict_types=1);

// ✅ Value Object instead of primitive
final class EmailAddress
{
    public function __construct(private string $value)
    {
        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
            throw InvalidEmailAddress::for($value);
        }
    }
}
```

> Avoid primitives when modeling domain concepts.

---

## 🏛 Layer Responsibilities

### **Domain Layer (Core Business)**

✅ DO

- Entities, Value Objects, Domain Events
- Enforce invariants
- Expose behavior, not data
- Declare repository **interfaces** only

❌ DON'T

- Import framework/ORM code
- Perform I/O (DB, HTTP)
- Orchestrate workflows

### **Application Layer (Use Cases)**

```php
class ActivateSubscriptionHandler
{
    public function handle(ActivateSubscription $command): void
    {
        $this->policy->canActivate($command->userId);
        $subscription = $this->repo->get($command->subscriptionId);
        $subscription->activate();
        $this->repo->save($subscription);
    }
}
```

DO:

- Policies, DTOs, transactions, event dispatch DON’T:
- Contain business rules

### **Infrastructure Layer (Adapters)**

- Controllers, persistence, external API integration
- Keep thin (< 15 lines per action)
- Map Domain ↔ ORM/Transport

---

## 🧪 Testing Strategy

### ✅ Unit Tests (Domain)

```php
public function test_cannot_activate_empty_subscription(): void
{
    $subscription = SubscriptionFactory::empty();
    $this->expectException(EmptySubscription::class);
    $subscription->activate();
}
```

- Fast, pure, no database/framework

### 🔄 Application / Feature Tests

```php
$handler = new ActivateSubscriptionHandler(new InMemorySubscriptionRepository());
$handler->handle(new ActivateSubscription($id));
```

- Test handlers using in‑memory repositories

---

## 🚫 Anti‑Patterns

| Anti‑Pattern                  | Fix                            |
| ----------------------------- | ------------------------------ |
| Anemic models                 | Add behavior to aggregates     |
| Business logic in controllers | Move to Application handler    |
| ORM models inside Domain      | Map in Infrastructure layer    |
| Generic exceptions            | Use specific domain exceptions |
| God objects                   | Use smaller aggregates         |

---

## **✅ Code Review Checklist**

### **Domain Layer**

- Uses ubiquitous language in naming
- No framework dependencies
- Rich behavior (not just getters/setters)
- Value objects for concepts
- Aggregates enforce invariants
- Repository interfaces only

### **Application Layer**

- Commands are explicit and validated
- Handlers orchestrate only
- Authorization before mutation
- Returns domain objects or DTOs

### **Infrastructure Layer**

- Controllers are thin (< 15 lines)
- Proper Domain ↔ ORM mapping
- No business logic in adapters

---

## 🛠️ Tooling & Automation

### Code Quality

```bash
# PHP
yarn php:lint
./vendor/bin/php-cs-fixer fix --dry-run
./phpstan-packages.sh

# TypeScript
eslint --fix
npm run type-check
```

### Architecture Enforcement

| Stack      | Tool                  |
| ---------- | --------------------- |
| PHP        | Deptrac               |
| TypeScript | ESLint import rules   |
| All        | Static analysis in CI |

---

## 🤖 AI Assistant Guidelines

Decision Tree: Where does code belong?

| Question                                  | Layer          |
| ----------------------------------------- | -------------- |
| Is this a **business rule or invariant**? | Domain         |
| Does it **coordinate domain objects**?    | Application    |
| Does it involve **DB, HTTP, queues**?     | Infrastructure |
| Cross‑aggregate logic?                    | Domain Service |
| Permission/authorization rule?            | Domain Policy  |

Naming conventions:

- **Commands:** `Verb + Noun` → `ActivateSubscription`
- **Events:** `Noun + PastTense` → `SubscriptionActivated`
- **Policies:** `Noun + Policy` → `SubscriptionPolicy`

> **If a method name starts with **``**, it's probably wrong.**

