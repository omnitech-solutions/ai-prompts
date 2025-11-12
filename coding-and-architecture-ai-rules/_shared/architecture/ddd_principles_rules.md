Path: ../../_shared/architecture/ddd-principles-api-first-operational-complete.md

# DDD Principles — API‑First (Subscription Domain)
> Point‑form, execution‑focused version for engineers + AI assistants.
> Structured to guide consistent implementation inside an **apps‑first monorepo**.

---

## ✅ Purpose
- Provide **actionable rules** for designing API‑first backend domains.
- Ensure aggregates & domain logic live where they belong — **in the domain layer**, not controllers.
- Reduce drift between teams by enforcing folder structure + naming.
- Enable AI tools (Cursor / Tabby ML) to place generated code correctly.

---

## 🔗 Chain of Workflows (follow in order)
1. Define domain language (nouns, verbs, invariants).
2. Identify aggregate root (Subscription).
3. Define value objects (SubscriptionId, Money).
4. Define policies (upgrade rules, invariants spanning actors).
5. Expose behaviors through commands + handlers.
6. Persist through repository interface (domain) + implementation (infra).

> Output: behavior driven by the **domain model**, not CRUD.

---

## 🧠 Core Pillars
- **Ubiquitous Language** — code names = domain names.
- **Bounded Contexts** — each app owns its domain; no leaking types across contexts.
- **Model Continuously** — evolve the model as insights improve.
- **Tactical Patterns Serve Strategy** — aggregates + VOs reinforce boundaries.

---

## 📁 Folder Structure (apps-first monorepo)
```
/ (repo root)
├── apps/
│   ├── subscriptions/                 ← bounded context
│   │   ├── src/
│   │   │   ├── domain/                ← aggregates, VOs, events, policies, repos
│   │   │   ├── application/           ← commands, queries, handlers
│   │   │   └── infrastructure/        ← controllers, persistence, integrations
│   │   └── tests/                     ← mirrors src/
│   └── billing/                       ← separate bounded context
└── packages/                          ← shared libs (never domain)
```
- **App = bounded context**.
- **Domain never depends on Infra**.
- **Shared never contains domain logic**.

---

## 🧩 Aggregates (consistency boundary)
> Responsible for enforcing invariants and emitting domain events.

```pseudocode
aggregate Subscription {
  id: SubscriptionId
  accountId: AccountId
  plan: PlanId
  status: {PENDING, ACTIVE, PAUSED, CANCELLED}
  nextBillingAt: DateTime?
  price: Money

  static create(accountId, plan, payment)
  activate(payment)
  changePlan(newPlan)
}
```
- Only aggregate methods mutate state.
- No HTTP, DB, or queue logic.
- Emits domain events on meaningful state change.

---

## 🔑 Entities & Value Objects
**Entities** — identity over time (identity matters more than attributes).
- Example: `Subscription` exists even if plan changes.

**Value Objects** — immutable, validated, equality by value.
- Represent concepts: `Money`, `Email`, `PlanId`.

```pseudocode
value Money { amount: Decimal, currency: Currency }
value SubscriptionId { value: UUID }
value PlanId { value: UUID }
```

---

## 🏭 Factories (explicit creation)
- Use **static factories** to enforce invariants at creation.
- Avoid `new Subscription(...)` — too easy to break invariants.

```pseudocode
Subscription.create(accountId, plan, payment)
```
- Factories guarantee valid state from the moment of construction.

---

## 🧭 Policies (business rules)
- Encapsulate rules that depend on actors or context.
- Called by handlers before aggregate mutation.

```pseudocode
policy SubscriptionPolicy {
  ensureCanUpgrade(user, subscription)
}
```

---

## 🔌 Domain Services (multi‑aggregate behavior)
- Domain logic that spans aggregates or requires external domain knowledge.
- Accept/return **domain types**, not primitives.

```pseudocode
service BillingService {
  charge(accountId, money): PaymentResult
}
```

---

## 📣 Domain Events (facts)
- Describe **what happened**, not what will happen.
- Used for notifying other contexts.

Examples: `SubscriptionCreated`, `SubscriptionActivated`, `SubscriptionPlanChanged`

Events are always **past tense** and immutable.

---

## 🗄 Repositories (interfaces in domain)
> Domain defines the interface; infrastructure implements it.

```pseudocode
interface SubscriptionRepository {
  save(Subscription)
  getById(SubscriptionId): Subscription?
}
```
- Return aggregates/VOs, **never ORM models** or arrays.

---

## 🔌 Bounded Context Integration
- Synchronous → **integration APIs**
- Asynchronous → **domain events**
- No internal domain types cross context boundaries.

> Controllers call application layer → handlers → aggregate.

---

## 🧱 Anti‑Corruption Layer (ACL)
- Protects domain from external semantics.

Usage:
- When integrating with legacy / third‑party systems.
- Translate into domain terms inside ACL adapters.

---

## 🔄 Sagas / Process Managers
- For long‑running processes involving multiple aggregates/contexts.
- Maintain explicit saga state.
- React to domain events → dispatch commands.

Example: activation → provisioning → billing → notification.

---

## ✔️ Consistency & Transactions
- **Within aggregate:** strong consistency.
- **Across aggregates/contexts:** eventual (events).
- Use repository transaction helpers for multi‑step operations.
- Consider compensating actions for failure flows.

---

## 🗃 Event Sourcing (optional)
- Default: state‑based aggregates.
- Use event sourcing when:
  - Full audit trail required.
  - Time‑based reasoning ("what was true at time X?").

---

## ✅ Testing Strategy
- **Domain unit** — pure, assert invariants + events.
- **Application feature** — handlers + in‑memory repo.
- **Contract tests** — verify integration APIs and events.

---

## 🚫 Anti‑Patterns
- Anemic domain (CRUD only, no behavior).
- Domain calling HTTP/DB directly.
- Repositories returning primitives/ORM models.
- One aggregate doing everything (**God Aggregate**).
- Generic events (`UpdatedEvent`) — prefer domain‑specific names.

---

_End of operational complete version._

