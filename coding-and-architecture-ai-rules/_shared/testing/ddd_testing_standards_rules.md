# ✅ Testing Standards (Subscriptions Domain)

**Path:** `.cursor/rules/testing/testing-standards.mdc`

> Fast, reliable, deterministic tests — aligned with *Domain‑Driven Design* and the **Subscription** domain.

---
## 🎯 Goals
- **Fast feedback** — sub‑second unit tests.
- **High signal** — failures explain domain rules (not framework noise).
- **Deterministic** — no randomness, no external time/network.
- **Layer-aligned** — test the *right thing at the right layer*.

> 🔑 **Rule:** A test should tell a story about business rules.

---
## 🧱 Test Pyramid
- **Unit (Domain)** — pure business logic
  - No Laravel / no DB
  - Validate **invariants + events emitted**

- **Application (Handler)** — orchestration
  - Pseudocode:
    ```pseudo
    repo.get → policy.check → aggregate.behavior → repo.save → event.publish
    ```

- **Feature (Infrastructure)** — HTTP / Persistence
  - DB + Queue faked where needed
  - Assert resource output (shape, not internal ORM)

> 🔑 **Rule:** Domain tests must not import framework types.

---
## 🗂 Directory Layout
```
packages/{Subscription}/tests/
├── Unit/         # Domain / pure behavior
├── Application/  # Handler / orchestration
└── Feature/      # HTTP / DB / adapters
```

---
## 🧪 Domain Unit Tests
**What to test:**
- Aggregates (e.g., `Subscription`)
- Value Objects (e.g., `SubscriptionId`)
- Policies (e.g., `SubscriptionPolicy`)

**Examples:**
```pseudo
Given subscription is already ACTIVE
When activate() is called again
Then fail with SubscriptionAlreadyActive
```

```php
$sub = Subscription::start($customerId);
$sub->activate();
$this->expectException(SubscriptionAlreadyActive::class);
$sub->activate();
```

> ✅ Focus: **rules + events**, not storage or side effects.

---
## ⚙️ Application Handler Tests
**What to test:** use‑case orchestration

```pseudo
Given a valid ActivateSubscription command
When handler.handle(command)
Then subscription.activate() should be called
And repository.save() should persist changes
```

```php
$handler = new ActivateSubscriptionHandler(new FakeSubscriptionRepo());
$handler->handle(new ActivateSubscription($id));
$this->assertTrue($repo->saved($id));
```

> ✅ Mock **boundaries** (repositories, policies), never domain.

---
## 🌐 Feature Tests (HTTP / Infrastructure)
**Purpose:** contract-level confidence

```pseudo
POST /api/subscriptions/{id}/activate
→ expect { status: "active" }
```

**Rules:**
- Use `Event::fake()` + `Queue::fake()`
- Assert **JSON shape**, not ORM internals

---
## 🛠 Test Data
- Use builders in **Unit**: `SubscriptionBuilder`
- Use model factories in **Feature**

```pseudo
SubscriptionBuilder.start().active().build()
```

---
## 🧬 Determinism
- Freeze time via `Carbon::setTestNow()`
- Inject `Clock` into domain when required

> ✅ Tests should not fail tomorrow.

---
## 🧩 Assertions
- Assert business outcomes, not internals

```pseudo
assert subscription.status == ACTIVE
assert event of type SubscriptionActivated is recorded
```

---
## 🚫 Anti‑Patterns
- ❌ Testing framework behavior
- ❌ Asserting internal method calls
- ❌ Over‑mocking concrete classes
- ❌ Depending on time / randomness / external API

---
## ✅ Review Checklist
- Unit (Domain) tests added for new rules
- Handler test verifies orchestration
- Feature test verifies HTTP + mappings
- Fakes used for time / queue / events
- Tests read like a business scenario

> 🧠 **If the test doesn’t describe a rule, it doesn’t belong.**

