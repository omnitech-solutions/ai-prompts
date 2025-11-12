# ✅ Aggregates (Support Ticket Domain)
**Path:** `.cursor/rules/domain/aggregates.mdc`

Aggregates are the **consistency boundary** of the Domain. They protect business invariants and emit domain events when state changes.

---
## 🎯 Goals
- Model business rules in one place
- Maintain **consistency** of state within a boundary
- Prevent invalid state transitions
- Express behavior using Ubiquitous Language (e.g., `escalate()`, `resolve()`)

> An Aggregate is not data — it is **behavior + rules + invariants**.

---
## 🧠 Core Rules
- Has an **Aggregate Root** (`SupportTicket`)
- Exposes behavior methods — not setters
- Ensures invariants before changing state
- Emits **Domain Events** on state changes
- Refer to other aggregates by **ID**, never by direct reference

```pseudo
SupportTicket.aggregate:
  method escalate():
    if priority already Max → throw
    change priority
    recordEvent(TicketEscalated)
```

---
## 📦 Aggregate Structure (Pseudocode)
```pseudo
class SupportTicket:
  readonly ticketId
  status
  priority
  assignedTo
  events = []

  method escalate():
    if status == RESOLVED → throw
    priority = priority.nextLevel()
    recordEvent(TicketEscalated.from(this))
```

---
## ✅ Example (PHP: Support Ticket)
```php
final class SupportTicket
{
    private array $events = [];

    private function __construct(
        private TicketId $id,
        private TicketPriority $priority,
        private TicketStatus $status,
        private ?UserId $assignedTo,
    ) {}

    public static function open(TicketId $id, UserId $creator): self
    {
        $ticket = new self($id, TicketPriority::normal(), TicketStatus::open(), $creator);
        $ticket->recordEvent(TicketOpened::from($ticket));
        return $ticket;
    }

    public function escalate(): void
    {
        if ($this->status->isResolved()) {
            throw TicketAlreadyResolved::for($this->id);
        }

        $this->priority = $this->priority->next();
        $this->recordEvent(TicketEscalated::from($this));
    }

    private function recordEvent(object $event): void
    {
        $this->events[] = $event;
    }

    public function pullEvents(): array
    {
        $events = $this->events;
        $this->events = [];
        return $events;
    }
}
```

---
## 🔁 State Transition Rules
- Methods must reflect business terminology
  - `escalate()` ✅
  - `setPriority()` ❌
- Methods enforce **invariants**
  - "cannot escalate a resolved ticket"
  - "cannot assign to inactive agent"

---
## 🧪 Testing Aggregates
```pseudo
Given: a NEW ticket
When: escalate()
Then: priority increases
And: event TicketEscalated is emitted
```

- Test domain logic **without database**
- Assert **events emitted**, not persistence

---
## 🚫 Anti-Patterns
❌ Public setters (`setStatus`, `setPriority`)
❌ Mutating multiple aggregates inside one transaction
❌ Aggregates calling repositories or HTTP

> Aggregates do **not** orchestrate workflows.

---
## 🤖 AI Assistant Guidelines
When generating Aggregate code:
- Use **behavioral names**: `resolve()`, `escalate()`, `assignTo()`
- Ensure state changes happen only after **invariant checks**
- Emit **Domain Events** from methods, not handlers
- If logic involves persistence → send to Handler / Repository

Decision tree:
```pseudo
IF enforces a rule → Aggregate
IF coordinates objects → Handler
IF persists objects → Repository
```

---
## ✅ Review Checklist
- Behavior methods (no setters)
- Invariants enforced before state change
- Event emitted on every state transition
- Other aggregates referenced by ID
- No repository/HTTP/framework code