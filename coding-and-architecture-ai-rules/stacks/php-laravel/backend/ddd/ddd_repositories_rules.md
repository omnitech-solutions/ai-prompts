# ✅ Repository Rules (Inventory / Warehouse Domain)
**Path:** `.cursor/rules/domain/repositories.mdc`

Repositories abstract **persistence** and expose **aggregate retrieval + storage** using the Ubiquitous Language.
They hide all technical concerns (ORM, SQL, HTTP, caching) from the Domain + Application layers.

---
## 🎯 Goals
- Keep **Domain layer persistence-agnostic**.
- Make Application handlers simple (`repo.get → aggregate.behavior → repo.save`).
- Allow changing storage layer (SQL → S3 → API) without touching business logic.

> The Repository represents a *collection of aggregates*, not a data access utility.

---
## 🧠 Core Rules
- Interface lives in **Domain**.
- Implementation lives in **Infrastructure**.
- Never return ORM models — always return **Aggregates**.
- Accept/return **Value Objects** (`InventoryItemId`, `Sku`).
- No primitive IDs.

---
## 🗂 Repository Interface (Domain)
```php
interface InventoryItemRepository
{
    public function get(InventoryItemId $id): InventoryItem;

    /** @return list<InventoryItem> */
    public function listLowStock(int $threshold): array;

    public function save(InventoryItem $item): void;
}
```

---
## 💾 Repository Implementation (Infrastructure)
```php
final class InventoryItemEloquentRepository implements InventoryItemRepository
{
    public function get(InventoryItemId $id): InventoryItem
    {
        $model = InventoryItemModel::where('id', $id->toString())->firstOrFail();
        return $this->toDomain($model);
    }

    public function save(InventoryItem $item): void
    {
        InventoryItemModel::updateOrCreate(
            ['id' => $item->id()->toString()],
            $this->fromDomain($item),
        );
    }
}
```

> Domain → **interface** only; Infrastructure → **implementation**.

---
## 🔄 Handler Usage (Application Layer)
```pseudo
handler.handle(command):
  item = repo.get(command.itemId)
  item.reserve(command.quantity)
  repo.save(item)
```

> Repo enables use case orchestration without leaking DB concerns.

---
## 🧪 Testing
- Domain tests: no repository.
- Application tests: use **InMemoryRepository**.
- Infrastructure tests: hit DB + mapping.

```php
final class InMemoryInventoryItemRepository implements InventoryItemRepository
{
    /** @var array<string, InventoryItem> */ private array $store = [];

    public function get(InventoryItemId $id): InventoryItem { return $this->store[$id->toString()]; }
    public function save(InventoryItem $item): void { $this->store[$item->id()->toString()] = $item; }
    public function listLowStock(int $threshold): array { /* ... */ }
}
```

> Use in-memory repos in handler tests for fast feedback.

---
## 🚫 Anti-Patterns
❌ Returning ORM models
❌ Query-style methods inside aggregates (belongs in repo)
❌ Accepting primitive values instead of Value Objects

---
## 🤖 AI Assistant Guidelines
When generating repositories:
- Define interface in **Domain** with Ubiquitous Language
- Use **Value Objects** for identity (`InventoryItemId`, `Sku`)
- Use **mapping** between domain & persistence
- No business logic or mapping inside aggregates

Decision tree:
```pseudo
IF persistence or lookup logic → Repository
IF validation or business rules → Aggregate
```

---
## ✅ Review Checklist
- Interface exists in Domain
- Implementation in Infrastructure
- No ORM types leave Infrastructure
- Repo returns Aggregates, not primitives
- Handler performs: repo.get → behavior → repo.save