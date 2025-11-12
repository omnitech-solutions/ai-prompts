# ✅ Controllers (Thin Transport Adapters)
**Path:** `.cursor/rules/application/controllers.mdc`

Controllers adapt transports (HTTP, GraphQL, CLI) into **Commands** or **Queries**. They never contain business logic.

---
## 🎯 Goals
- Keep controllers **thin** and readable.
- Delegate behavior to **Application Handlers**.
- Keep domain and business logic **out of controllers**.

> A controller should be boring. If it’s interesting, it’s wrong.

---
## 🧠 Principles
- 10–15 lines max
- No domain logic, no decision making
- Convert request → Command or Query
- Pass result to presenter/response

---
## 📐 Structure (Pseudocode)
```pseudo
Controller
  ↓ constructs
Command / Query
  ↓ sent to
Application Handler
  ↓ loads and invokes domain
Domain
```

---
## ✅ Example (Support Domain)
**HTTP → Command → Handler → Domain**

```php
final class AssignSupportTicketController
{
    public function __invoke(Request $request, string $ticketId)
    {
        $command = AssignTicketCommand::fromRequest($request, $ticketId);
        $this->handler->handle($command);

        return Response::json(['status' => 'assigned']);
    }
}
```

```php
final class AssignTicketCommand
{
    public function __construct(
        public readonly TicketId $ticketId,
        public readonly AgentId $agentId,
    ) {}

    public static function fromRequest(Request $request, string $ticketId): self
    {
        return new self(
            TicketId::fromString($ticketId),
            AgentId::fromString($request->get('agent_id')),
        );
    }
}
```

---
## 🚫 Anti‑Patterns
| Anti‑Pattern | Fix |
|--------------|-----|
| Controller loads ORM models | Use Handler + Repository |
| Controller performs domain logic | Put logic in aggregate/handler |
| Controller maps raw arrays | Use Value Objects/DTOs |

---
## 🧪 Testing
- Controller tests verify **route + serialization** only.
- Handlers and domain tested separately.

```pseudo
Test Controller: HTTP 201 + correct JSON
Test Handler: domain rules executed
```

---
## 🤖 AI Assistant Guidelines
When generating controllers:
- Think: *“What transport am I adapting?”*
- Never return ORM models; return DTOs.
- Always route controller → Command → Handler

Decision Tree:
```pseudo
IF logic = <15 lines → good
IF logic contains decisions → push down to handler
IF ORM used → move to repository
```

---
## ✅ PR Review Checklist
- Controller < 15 lines
- Uses Command/Query object
- No domain logic or repositories
- No primitives where a Value Object exists
- Response formatting delegated to presenter

