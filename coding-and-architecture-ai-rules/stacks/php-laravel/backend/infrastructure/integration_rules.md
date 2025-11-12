# 🌉 Integration Rules (External Systems & Cross-Context Boundaries)
**Path:** `.cursor/rules/integration/integration-rules.mdc`

Integrations adapt **our domain** to external systems (SaaS APIs, third-party carriers, authentication providers). They are *not* domain logic.

---
## 🎯 Goals
- Prevent external system complexity from leaking into the domain.
- Apply **Anti-Corruption Layer (ACL)** to translate models.
- Keep retry + idempotency logic **in Infrastructure**, not Domain.

> Integrations build a protective layer around the domain.

---
## 🧠 Principles
- Domain talks only to **integration interfaces**.
- External schemas → mapped to **Value Objects / DTOs**.
- Never throw raw HTTP exceptions into the Domain.
- Do not mutate domain entities inside integrations.

---
## 📐 Structure (Pseudocode — Shipping Domain)
```pseudo
Domain Event: ShipmentRequested
↓
Listener → calls Integration Service
↓
Integration sends request to Carrier API
↓
CarrierResponse mapped → Domain DTO
```

---
## ✅ Example (Shipping/Carrier Integration)
```php
interface CarrierApi {
    public function createShippingLabel(ShipmentId $id): ShippingLabelDto;
}
```

**Implementation (Infrastructure)**
```php
final class ShipEngineApi implements CarrierApi
{
    public function createShippingLabel(ShipmentId $id): ShippingLabelDto
    {
        $response = $this->client->post('/labels', [ 'shipment_id' => $id->toString() ]);

        if ($response->failed()) {
            throw CarrierUnavailable::withContext($id);
        }

        return new ShippingLabelDto(
            trackingNumber: $response['tracking'],
            pdfUrl: $response['label_url'],
        );
    }
}
```

---
## 🛑 Error & Retry Strategy
- Retry **transport failures**, not business rule failures.
- Map external errors → Stable domain errors.

```pseudo
HTTP 500 → CarrierUnavailable
HTTP 409 → RemoteConflict
HTTP 429 → RetryAfter
```

---
## 🔁 Idempotency
```pseudo
Idempotency-Key: shipment.id
```

---
## 🚫 Anti-Patterns
| Anti-Pattern | Fix |
|--------------|-----|
| Domain calling an HTTP client | Domain calls **integration interface** |
| Passing raw arrays | Use DTOs / Value Objects |
| Handling retries manually in controller | Queue/Job handles retries |

---
## 🤖 AI Assistant Guidelines
When generating integration code:
- Ask: *"Is this external? Then put it behind an ACL."*
- Generate **interface + implementation**, not direct HTTP in handlers.
- Always map external shapes into our own DTOs.

Decision Tree:
```pseudo
IF external request/response → Integration layer
IF mapping schemas → DTO / Value Object
IF state change in domain → Handler/Aggregate
```

---
## ✅ PR Review Checklist
- External calls behind interface
- No raw arrays or framework types leak into domain
- Errors mapped to domain exceptions
- Idempotency/Retry strategy defined
- Responses mapped to DTOs / Value Objects