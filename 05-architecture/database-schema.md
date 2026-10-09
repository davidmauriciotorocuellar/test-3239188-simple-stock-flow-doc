# Database Architecture & Structural Rules — Simple Stock Flow

## 1. Entity-Relationship Diagram

```mermaid
erDiagram
    category  ||--o{ product   : "classifies"
    sale      ||--|{ sale_item : "composes"
    product   ||--o{ sale_item : "sold_in (FK RESTRICT)"
    user      ||--o{ sale      : "registers (FK RESTRICT)"
```[cite: 2]

---

## 2. Foreign Key Policy Matrix

| Foreign Key | Source Column | Referenced Column | ON DELETE | Rationale |
|---|---|---|---|---|
| **FK-1** | `product.category_id` | `category.id` | `RESTRICT`[cite: 2] | Prevents purging categories that classify existing products[cite: 2]. |
| **FK-2** | `sale_item.sale_id` | `sale.id` | `CASCADE`[cite: 2] | Composition rule: line items have no existence outside their parent sale[cite: 2]. |
| **FK-3** | `sale_item.product_id` | `product.id` | `RESTRICT`[cite: 2] | Safeguard barrier: prevents physical purges of catalog products from corrupting historical sales records[cite: 2]. |
| **FK-4** | `sale.sold_by_user_id` | `user.id` | `RESTRICT`[cite: 2] | Operator accountability: an operator account with registered sales cannot be deleted[cite: 2]. |

---

## 3. Soft Deletion Mechanism (ADR-003)
When a product is deleted, the system populates `product.deleted_at` with the current UTC timestamp[cite: 2]. Global query filters exclude soft-deleted records from regular catalog queries[cite: 2]. Foreign keys and historical sale items remain intact[cite: 2].
