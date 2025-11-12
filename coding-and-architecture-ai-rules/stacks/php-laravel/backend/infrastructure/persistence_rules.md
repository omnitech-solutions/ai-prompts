# 🗄️ Persistence Rules (Repositories, ORM, Transactions)
**Path:** `.cursor/rules/infrastructure/persistence-rules.mdc`

Persistence is an *infrastructure concern*. The Domain never knows how data is stored.

---
## 🎯 Goals
- Keep domain persistence-agnostic.
- Repositories **map** Domain ↔ ORM/data.
- Use transactions only in Application layer.

> The domain expresses intent — persistence executes it.

---
## 🧠 Principles
- Domain defines **Repository Interfaces**
- Infrastructure implements repository logic
- Repositories return **Aggregates**, not ORM models
- One aggregate → one repository

---
## 📦 Structure (Pseudocode — Inventory Domain)
```pseudo
Domain: InventoryItem
Repository Interface: InventoryItemRepository
Infrastructure: EloquentInventoryItemRepository
```

---
## ✅ Example (Domain Repository Interface)
```php
interface InventoryItemRepository
{
    public function get(InventoryItemId $id): InventoryItem;
    public function save(InventoryItem $item): void;
}
```

**Infrastructure Implementation (ORM Mapping)**
```php
final class EloquentInventoryItemRepository implements InventoryItemRepository
{
    public function get(InventoryItemId $id): InventoryItem
    {
        $model = ItemModel::where('id', $id->toString())->firstOrFail();
        return InventoryItemMapper::toDomain($model);
    }

    public function save(InventoryItem $item): void
    {
        ItemModel::updateOrCreate(
            ['id' => $item->id()->toString()],
            InventoryItemMapper::toPersistence($item)
        );
    }
}
```

---
## 🧪 Mapping Rules
- ORM model is not passed to domain
- Use **mapper classes** or data transformers

```pseudo
ORM Model → Mapper → Aggregate
Aggregate → Mapper → ORM Model
```

---
## 🔄 Transactions
- Only Application Handlers manage transactions (not repositories).

```php
DB::transaction(fn() => $handler->handle($command));
```

---
## 🚫 Anti-Patterns
| Anti-Pattern | Fix |
|--------------|-----|
| Domain uses ORM models | Domain uses repository interface |
| Setters instead of behavior | Call Aggregate methods |
| Repositories returning arrays | Return Aggregates / Value Objects |

---
## 🤖 AI Assistant Guidelines
When generating persistence code:
- Ask: *"Is this persistence or mapping? Then Infrastructure."*
- Domain should never import ORM/model classes.
- Repositories should call behavior, not mutate state.

Decision Tree:
```pseudo
IF DB/ORM → Infrastructure repository
IF mapping objects → Mapper
IF business rules → Aggregate
```

---
## ✅ PR Review Checklist
- Domain has repository interface only
- Implementation lives in Infrastructure
- Mapping layer converts ORM ↔ Aggregate
- No ORM types leak into domain
- Transactions handled at Application layer