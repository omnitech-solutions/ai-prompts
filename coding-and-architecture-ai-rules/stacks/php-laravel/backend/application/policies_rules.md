# ✅ Domain Policies (Authorization Rules)

**Path:** `.cursor/rules/application/policies.mdc`

Policies define **who can perform actions** on a Subscription aggregate — independent of HTTP, persistence, or framework concerns.

---
## 🎯 Goals
- Express authorization using **domain language**.
- Keep policy logic **pure and framework‑free**.
- Enforce authorization **before** any state mutation.

> Policies protect invariants at the system boundary.

---
## 🧠 Core Concepts
- **Policy** → answers: *“Is this actor allowed to do this?”*
- **Handler** → orchestrates behavior by calling: `policy.check()` → `aggregate.method()` → `repo.save()`
- **Aggregate** → enforces business rules; does **not** check authorization.

---
## 📌 Rules
- Located in: `Domain/Policies/`
- Use **Value Objects** (`UserId`, `SubscriptionId`) — no primitives.
- Return `void` on success; **throw specific exceptions** on failure.
- No framework imports (`Request`, `Model`, DB calls).

---
## 🧱 Structure (Pseudocode)

```pseudo
class SubscriptionPolicy:
    method canActivate(UserId actor, Subscription sub):
        if sub.belongsToDifferentTeam(actor):
            throw UnauthorizedSubscriptionAccess

        if sub.isCancelled():
            throw SubscriptionCannotBeActivated
```

---
## ✅ Example (PHP / Subscription Domain)

```php
final class SubscriptionPolicy
{
    public function canActivate(UserId $actor, Subscription $subscription): void
    {
        if (!$subscription->belongsToActor($actor)) {
            throw NotAuthorizedForSubscription::of($subscription->id(), $actor);
        }

        if ($subscription->isCancelled()) {
            throw SubscriptionCannotBeActivated::of($subscription->id());
        }
    }
}
```

---
## 🚦 Application Handler Usage

```pseudo
handler.handle(command):
    sub = repo.get(command.subscriptionId)

    policy.canActivate(command.actorId, sub)  // ✅ authorization

    sub.activate()                             // ✅ domain behavior

    repo.save(sub)
```
---
## 🔐 Query Policies (Reads)

Pseudocode example:

```pseudo
method canView(UserId actor, Subscription sub):
    if not sub.isVisibleTo(actor):
        throw NotAuthorizedToViewSubscription
```

> Queries also require authorization — not only commands.

---
## 🧪 Testing Policies

```pseudo
Given: Subscription owned by UserA
When:  UserB attempts activation
Then:  throw NotAuthorizedForSubscription
```

- Pure PHP tests — no DB, no HTTP, no mocks required.
- Test **happy path** and **deny path**.

---
## 🚫 Anti‑Patterns
❌ Returning `bool` and ignoring it in handlers  
✅ Throw exception on restricted actions

❌ Policies calling repositories/HTTP  
✅ Policies reason using only aggregates + VOs

❌ Embedding HTTP status or response  
✅ Keep responses in Infrastructure layer

---
## 🤖 AI Assistant Guidelines
When generating policies:

- Place them in **Domain/Policies/**
- Use method names that reflect business meaning: `canActivate`, `canCancel`
- Require `UserId` and `Subscription` aggregate
- Throw **specific exceptions**

Decision tree:

```pseudo
IF policy needs persistence → extract checks into repository layer
ELSE logic stays inside policy
```

> If it touches I/O, it's not a policy.

---
## ✅ Review Checklist
- Policy lives in **Domain**
- Uses **Value Objects** (`UserId`, `SubscriptionId`)
- Throws specific exceptions
- Handler calls policy **before behavior**
- No framework types or I/O

---
Policies express **permission**. Aggregates express **business rules**.
Together, they keep the system safe, explicit, and testable.
