# ✅ Jobs & Listeners (Message-Driven Use Cases)
**Path:** `.cursor/rules/application/jobs-listeners.mdc`

Message-driven execution for async processes such as email delivery, invoice generation, or data synchronization.

---
## 🎯 Goals
- Decouple **domain changes** from **side effects** (emails, notifications, integrations).
- Ensure handlers are **idempotent**, retry-safe, and small.
- Maintain clarity between **domain events**, **application events**, and **infrastructure jobs**.

> Jobs/Listeners react to *something already decided by the domain*.

---
## 🧠 Principles
- **Domain Events trigger side effects.** (never business rules)
- **Listeners map → Jobs**, Jobs do the work.
- **Idempotency matters** for retries and eventual consistency.
- Keep them **infrastructure**, never part of the domain model.

---
## 📐 Structure (Pseudocode)
```pseudo
Domain Event: InvoiceGenerated
Listener: ScheduleInvoiceEmail
Job: SendInvoiceEmail
```

---
## ✅ Example (PHP / Billing Domain)
```php
final class InvoiceGenerated
{
    public function __construct(
        public readonly InvoiceId $invoiceId,
        public readonly CustomerId $customerId,
    ) {}
}
```

**Listener — schedules work**
```php
final class ScheduleInvoiceEmail
{
    public function handle(InvoiceGenerated $event): void
    {
        SendInvoiceEmail::dispatch(
            invoiceId: $event->invoiceId,
            customerId: $event->customerId
        );
    }
}
```

**Job — executes work**
```php
final class SendInvoiceEmail implements ShouldQueue
{
    public function handle(): void
    {
        $invoice = $this->repo->get($this->invoiceId);
        $email = InvoiceEmail::fromInvoice($invoice);
        $this->mailer->send($email);
    }
}
```

---
## 🧪 Testing
- Listener tests: assert Job was dispatched.
- Job tests: assert side effect happened.

```pseudo
Test Listener: when InvoiceGenerated → job dispatched
Test Job: given job execution → email sent
```

---
## 🚫 Anti-Patterns
| Anti-Pattern | Fix |
|--------------|-----|
| Listener contains business logic | Move to domain or app handler |
| Job mutates domain state | Domain handler should do the mutation |
| Job catches & hides errors | Let queue retry handle failures |
| Jobs accept primitives over Value Objects | Keep typed IDs |

---
## 🤖 AI Assistant Guidelines
When generating Jobs/Listeners:
- Use **event name** that reflects domain change: `ShipmentPacked`, `UserRegistered`.
- Listener triggers work → Job performs work.
- Ask: *“Is this a side effect? Then it's a job.”*

Decision Tree:
```pseudo
IF reacts to domain event → Listener
IF performs a side effect → Job
```

---
## ✅ PR Review Checklist
- Listener contains no business logic
- Job is idempotent and retry-safe
- Accepts Value Objects or IDs, not primitives
- Errors are allowed to bubble (retry)
- Job < 20 lines of actual logic