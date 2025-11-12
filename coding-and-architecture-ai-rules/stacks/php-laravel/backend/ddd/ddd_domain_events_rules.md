# ✅ Domain Events (Work Orders / Production)
**Path:** `.cursor/rules/domain/domain-events.mdc`

Domain Events represent **something that happened in the past** that the business cares about.
They are immutable facts — not commands and not requests.

---
## 🎯 Goals
- Capture **meaningful business events** (`WorkOrderCompleted`, `WorkOrderScheduled`)
- Decouple aggregates from side effects (notifications, integrations, workflows)
- Enable eventual consistency across bounded contexts
- Provide auditability and traceability

> A Domain Event is *evidence of a state change that already happened*.

---
## 🧠 Core Rules
- Named in **past tense**: `WorkOrderCompleted`
- Immutable (`readonly`, no setters)
- Created inside **aggregate methods only**
- Published by the **Application layer**, not Domain
- Do not include framework or transport types

---
## 📦 Structure (Pseudocode)
```pseudo
class WorkOrderCompleted:
  readonly workOrderId
  readonly completedAt
```

Event emission inside aggregate:
```pseudo
method complete():
  if alreadyCompleted → throw
  status = COMPLETED
  recordEvent(WorkOrderCompleted.from(this))
```

---
## ✅ Example (PHP)
```php
final class WorkOrderCompleted
{
    private function __construct(
        public readonly WorkOrderId $id,
        public readonly DateTimeImmutable $completedAt,
    ) {}

    public static function from(WorkOrder $order): self
    {
        return new self(
            id: $order->id(),
            completedAt: new DateTimeImmutable(),
        );
    }
}
```

---
## 🏛️ Emitting Events (Aggregate)
```php
final class WorkOrder
{
    public function complete(): void
    {
        if ($this->status->isCompleted()) {
            throw AlreadyCompleted::for($this->id);
        }

        $this->status = Status::Completed;
        $this->recordEvent(WorkOrderCompleted::from($this));
    }
}
```

---
## 🚦 Publishing Events (Application Handler)
```pseudo
handler.handle(cmd):
  order = repo.get(cmd.workOrderId)
  order.complete()         // event recorded inside aggregate
  repo.save(order)
  eventBus.publish(order.pullEvents())
```

---
## 🧪 Testing
```pseudo
Given: WorkOrder not completed
When: complete()
Then: event WorkOrderCompleted is emitted
```

> Domain tests assert emitted events — not side effects.

---
## 🚫 Anti-Patterns
❌ Event triggers DB writes or HTTP calls directly
❌ Event classes contain ORM IDs or arrays
❌ Event published from controller/service instead of aggregate

> Events are facts, not instructions.

---
## 🤖 AI Assistant Guidelines
When generating events:
- Use **past tense verbs** (`Completed`, `Scheduled`, `Cancelled`)
- Always emit events from **aggregate behavior**, never from a handler or controller
- Keep event DTOs **pure domain** — no framework types

Decision tree:
```pseudo
IF something already happened → Domain Event
IF something should happen     → Command
```

---
## ✅ Review Checklist
- Named in past tense
- Emitted from aggregate behavior
- Immutable (`readonly`)
- Does not depend on frameworks or infrastructure
- Application publishes events (not Domain)
