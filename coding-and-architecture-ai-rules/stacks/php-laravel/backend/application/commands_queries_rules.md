# ✅ Commands & Queries (Application Rules)

**Path:** `.cursor/rules/application/commands-queries.mdc`

Design conventions for **Commands & Queries** in the Application layer to keep orchestration clean, predictable, and aligned with the **Subscription** domain.

---
## 🎯 Goals
- Ensure handlers receive **fully-validated, domain-typed inputs**.
- Keep controllers and infrastructure **thin** — no business logic.
- Force clarity of intent: **write = Command**, **read = Query**.
- Make code easy to test by making commands/queries **pure data**.

> Commands and Queries are transport-agnostic DTOs. They do not know HTTP, GraphQL, or databases.

---
## 🧠 Definitions

| Type     | Responsibility                   | Side Effects |
|----------|----------------------------------|--------------|
| **Command** | Change system state (write)      | ✅ Yes        |
| **Query**   | Retrieve system state (read)      | ❌ No         |

---
## 📌 Core Rules
- **Immutable** → `readonly` properties, no setters.
- **Typed** → Use Value Objects (`SubscriptionId`, `UserId`).
- **Self-contained** → Handler requires no external parsing.
- **No framework types** → NEVER expose `Request`, `Model`, `Paginator`, etc.
- **Validation on construction** → ensure DTO is valid *before handler runs*.

---
## 📦 Command Example (Subscription Domain)
```php
final class ActivateSubscriptionCommand
{
    public function __construct(
        public readonly SubscriptionId $subscriptionId,
        public readonly UserId $actorId,
    ) {}

    public static function fromRequest(array $input): self
    {
        if (!isset($input['subscription_id'])) {
            throw new InvalidArgumentException('subscription_id required');
        }

        return new self(
            subscriptionId: SubscriptionId::from($input['subscription_id']),
            actorId: UserId::from($input['actor_id']),
        );
    }
}
```

**Purpose:** expresses intent — *activate this subscription*.

---
## 🔎 Query Example (Read-Only)
```php
final class ListSubscriptionsQuery
{
    public function __construct(
        public readonly TeamId $teamId,
        public readonly ?SubscriptionStatus $status = null,
        public readonly int $limit = 50,
        public readonly ?string $cursor = null,
    ) {}

    public static function fromArray(array $filters, int $teamId): self
    {
        return new self(
            teamId: TeamId::fromInt($teamId),
            status: isset($filters['status'])
                ? SubscriptionStatus::from($filters['status'])
                : null,
            limit: $filters['limit'] ?? 50,
            cursor: $filters['cursor'] ?? null,
        );
    }
}
```

**Purpose:** express a filtering strategy — *list subscriptions*.

---
## ✅ Validation Patterns
- Validate DTO **at creation**, not inside handlers.
- Prefer custom errors: `InvalidItems`, `InvalidCursor`, etc.
- Prefer **typed collections / Value Objects** to reduce validation surface.

Example pseudocode:
```pseudo
if items missing → throw InvalidItems
if actorId missing → throw MissingActor
```

---
## 📤 Mapping From Transports
Controllers or resolvers **only**:
1. Parse transport (`Request`, GraphQL input, CLI args)
2. Build DTO (`Command::fromRequest($request)`)
3. Pass into handler

> Controllers should not interpret business meaning.

---
## 📥 Result DTOs (Query Output)
```php
final class SubscriptionListResult
{
    /** @param array<array{id: string, status: string}> $items */
    private function __construct(
        public readonly array $items,
        public readonly ?string $nextCursor,
    ) {}

    /** @param array<Subscription> $subs */
    public static function fromAggregates(array $subs, ?string $cursor): self
    {
        return new self(
            items: array_map(
                fn($s) => [
                    'id' => $s->id->toString(),
                    'status' => $s->status->value,
                ],
                $subs,
            ),
            nextCursor: $cursor,
        );
    }
}
```

---
## 🔁 Idempotency & Retry
Use an **idempotency key** when repeated execution is possible.

```php
final class CancelSubscriptionCommand
{
    public function __construct(
        public readonly SubscriptionId $subscriptionId,
        public readonly string $idempotencyKey,
        public readonly UserId $actorId,
    ) {}
}
```

---
## 🔐 Security & Authorization
- Commands that mutate state must include **actor identity**: `UserId`.
- Application handlers call **domain policies**.

Pseudocode:
```pseudo
handler.handle(cmd)
  → policy.check(actor, subscription)
  → subscription.activate()
  → repo.save(subscription)
```

---
## 🚫 Anti-Patterns
❌ Mutable DTOs with setters
❌ Commands/queries that accept framework types
❌ Generic untyped arrays when Value Objects exist
❌ "Kitchen sink" commands doing multiple things

> If a command name needs "and", it's the wrong abstraction.

---
## ✅ Review Checklist
- Immutable (`readonly`), typed properties
- All validation in constructors/factories
- No framework types (`Request`, `Model`, etc.)
- Queries encapsulate filtering/pagination
- Commands include actor identity if mutating data

---
## 🤖 AI Assistant Guidelines
**When generating Commands/Queries, always ask:**
- Is this a **state change** or **data retrieval**?
- Does the name express **intent**, not mechanics? (`ActivateSubscription` ✅ / `UpdateStatus` ❌)
- Does the handler need domain IDs or **Value Objects**?
- Does the DTO avoid framework contamination?

Decision Tree:
```pseudo
IF modifies state → Command
ELSE reads data → Query
```

> Commands and Queries are the language of the Application layer — concise, typed intent.
