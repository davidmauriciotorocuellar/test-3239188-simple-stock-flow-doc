# Database Indexing Strategy & Access Patterns — Simple Stock Flow

## 1. Primary Access Patterns

| Pattern ID | Access Description | Target Table | Filter / Condition | Frequency |
|---|---|---|---|---|
| **Q1** | Catalog product search | `product` | Partial text match, category, active (`deleted_at IS NULL`)[cite: 2] | High[cite: 2] |
| **Q7** | Sales by date range | `sale` | Date range filtering on `sold_at`[cite: 2] | High[cite: 2] |
| **Q9** | Aggregated Sales Report | `sale` ⋈ `sale_item` | Date range filter, group by product and frozen category label[cite: 2] | High (Most expensive)[cite: 2] |
| **Q10** | Authentication lookup | `user` | Exact equality on normalized `username`[cite: 2] | High (On every login)[cite: 2] |

---

## 2. Index Matrix & Design Justifications

### 2.1 Active Indices in Engine
- **`IX_category_name`**: Unique B-tree index on `category(name)`[cite: 2].
- **`IX_user_username`**: Unique B-tree index on `user(username)`[cite: 2].
- **`IX_sale_sold_at`**: B-tree index on `sale(sold_at)` serving date-range filtering (Q7, Q9)[cite: 2].

### 2.2 Optimization & Composite Covering Index
- **`IX_sale_item_composite`**: Unique composite index on `sale_item(sale_id, product_id)` including `INCLUDE (quantity, unit_price)`[cite: 2].
  - **Optimization Value:** Allows the analytical sales report (Q9) to perform **index-only scans**, calculating sales totals directly from the index tree without reading heap table blocks from disk[cite: 2].

### 2.3 Excluded Indices & Design Rationale
- **`password_hash`**: Never indexed for security and privacy compliance[cite: 2].
- **`deleted_at` standalone**: Omitted; instead integrated into partial index predicates where it adds real selectivity[cite: 2].
- **`product.stock`**: Omitted; stock is queried by product ID, never filtered as a search range[cite: 2].
