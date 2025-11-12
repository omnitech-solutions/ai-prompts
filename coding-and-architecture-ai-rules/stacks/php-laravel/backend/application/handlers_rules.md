# ✅ Application Handlers (Use-Case Orchestration Rules)

**Path:** `.cursor/rules/application/handlers.mdc`

Handlers coordinate a **single business use case** by invoking domain logic, policies, and persistence without containing business rules.

---
## 🎯 Goals
- Keep orchestration **explicit and traceable**
- Enforce **domain purity** — no business rules here
- Ensure **thin infrastructure** — controllers defer to handlers
- Improve testability: handlers accept Commands/Queries and return DTOs

> Handlers express *how* a use case executes — not *what* the business rules are.

---
## 🧠 Core Concepts
- **Command Handler** → performs write operations
- **Query Handler** → reads state and returns DTOs
- **Policy** → authorization guard
- **Aggregate** → enforces business invariants

Pseudocode orchestration:
```pseudo
handler.handle(command):
  aggregate = repo.get(command.id)
  policy.check(actor, aggregate)
  aggregate.behavior()
  repo.save(aggregate)
  return ResultDTO
```

---
## 📌 Rules
- Accept a **Command or Query DTO** as the **first parameter**
- Operate only on **Value Objects + Aggregates** (no primitives for IDs)
- Contain **zero business logic** — only orchestration
- No handler → handler calls
- Use **repository interface** to load + persist
- Use **transactions** for multiple changes

> If the handler contains domain logic, extract it into the Aggregate.

---
## 🏗️ Handler Skeleton (Pseudocode)
```pseudo
class ActivateSubscriptionHandler:
  constructor(repo, policy)

  method handle(ActivateSubscription cmd):
    subscription = repo.get(cmd.subscriptionId)

    policy.canActivate(cmd.actorId, subscription)

    subscription.activate()

    repo.save(subscription)

    return ActivateSubscriptionResult.from(subscription)
```

---
## ✅ Typed Command Example
```php
final class ActivateSubscriptionCommand
{
    public function __construct(
        public readonly SubscriptionId $subscriptionId,
        public readonly UserId $actorId,
    ) {}
}
```

---
## ✅ Handler Implementation (PHP)
```php
final class ActivateSubscriptionHandler
{
    public function __construct(
        private SubscriptionRepository $repo,
        private SubscriptionPolicy $policy,
    ) {}

    public function handle(ActivateSubscriptionCommand $cmd): ActivateSubscriptionResult
    {
        return $this->repo->transaction(function () use ($cmd) {
            $subscription = $this->repo->get($cmd->subscriptionId);

            $this->policy->canActivate($cmd->actorId, $subscription);

            $subscription->activate();

            $this->repo->save($subscription);

            return ActivateSubscriptionResult::from($subscription);
        });
    }
}
```

---
## 📤 Result DTO Pattern
```php
final class ActivateSubscriptionResult
{
    private function __construct(
        public readonly string $id,
        public readonly string $status,
    ) {}

    public static function from(Subscription $sub): self
    {
        return new self(
            id: $sub->id()->toString(),
            status: $sub->status()->value,
        );
    }
}
```

---
## 🔄 Transactions & Events
```pseudo
repo.transaction(fn):
  persist aggregate
  enqueue domain events
  return result
```

- open transaction in handler
- commit aggregate + events atomically
- Infrastructure dispatches events

---
## 🧪 Testing Guidance
- For handler unit tests:
  - Mock **Repository + Policy**, never the Aggregate
  - Assert orchestration order: load → authorize → mutate → save

```pseudo
test handler activates:
  repo.stubGet returns trialing subscription
  handler.handle(cmd)
  assert subscription.isActive
  assert repo.save called
```

---
## 🚫 Anti-Patterns
- Business logic **inside handlers**
- Returning Eloquent / ORM models
- Handler calling **another handler**
- Too many responsibilities (violates single use case)

---
## 🤖 AI Assistant Guidelines
When generating a handler:
- Ask: *“Is this Domain or Application?”*
  - If it mutates state → Handler calls **aggregate behavior**
  - If logic belongs to the aggregate, do **not** place in handler
- Ensure handler:
  - Accepts **Command/Query DTO**
  - Loads from **repository interface**
  - Invokes **policy → aggregate → persist** pattern
  - Returns **Result DTO**, not primitives

Decision Tree:
```pseudo
IF logic checks permissions → Policy
IF logic enforces invariants → Aggregate
IF logic coordinates flow → Handler
```

---
## ✅ Review Checklist
- Receives **Command/Query DTO** (first parameter)
- Calls **Policy before mutation**
- Calls **Aggregate behavior**, not setters
- Uses **transactions + repository**
- Returns **Result DTO** (never ORM models)
- No cross-handler calls or business logic
