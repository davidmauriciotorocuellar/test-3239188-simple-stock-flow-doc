# Physical Data Dictionary — PostgreSQL 16 (`sales` schema)

All tables use singular naming conventions ASCII (`PascalCase` in domain code, `snake_case` in database)[cite: 2].

## 1. `sales.category`
Stores reference classifications[cite: 2].

| Column | Type | Nullable | Default | Description & Constraints |
|---|---|---|---|---|
| `id` | `uuid` | NO | None | Primary Key (`PK_category`)[cite: 2]. |
| `name` | `varchar(120)` | NO | None | Category label[cite: 2]. Unique index `IX_category_name`[cite: 2]. |

---

## 2. `sales.product`
Stores catalog items[cite: 2].

| Column | Type | Nullable | Default | Description & Constraints |
|---|---|---|---|---|
| `id` | `uuid` | NO | None | Primary Key (`PK_product`)[cite: 2]. |
| `name` | `varchar(200)` | NO | None | Product title[cite: 2]. |
| `price` | `numeric(18,2)` | NO | None | Unit price[cite: 2]. Monocurrency (D-05)[cite: 2]. |
| `stock` | `integer` | NO | None | Inventory stock[cite: 2]. Constraint `ck_product_stock_non_negative` (`stock >= 0`)[cite: 2]. |
| `category_id` | `uuid` | NO | None | FK to `sales.category(id)` (`ON DELETE RESTRICT`)[cite: 2]. |
| `image_key` | `varchar(512)` | YES | None | Opaque external binary key (D-08)[cite: 2]. `NULL` when absent[cite: 2]. |
| `deleted_at` | `timestamptz` | YES | None | Soft deletion timestamp (ADR-003)[cite: 2]. |
### Concurrency & Audit Notes
- **`xmin` (`xid`)**: System column managed by PostgreSQL for optimistic concurrency control (D-04 / ADR-002).
- **No Audit Columns**: The system explicitly omits `created_at` and `updated_at` columns across all tables, avoiding non-domain triggers and keeping `sold_at` as the single business timestamp[cite: 2].
---

## 3. `sales.sale`
Stores header records of sales transactions[cite: 2].

| Column | Type | Nullable | Default | Description & Constraints |
|---|---|---|---|---|
| `id` | `uuid` | NO | None | Primary Key (`PK_sale`)[cite: 2]. |
| `sold_at` | `timestamptz` | NO | None | Transaction UTC timestamp[cite: 2]. Indexed (`IX_sale_sold_at`)[cite: 2]. |
| `sold_by` | `varchar(120)` | NO | None | Operator username who registered the sale[cite: 2]. |
| `sold_by_user_id` | `uuid` | NO | None | FK to `sales.user(id)` (`ON DELETE RESTRICT`)[cite: 2]. |

---

## 4. `sales.sale_item`
Stores line items within a sale[cite: 2].

| Column | Type | Nullable | Default | Description & Constraints |
|---|---|---|---|---|
| `id` | `uuid` | NO | None | Primary Key (`PK_sale_item`)[cite: 2]. |
| `sale_id` | `uuid` | NO | None | Parent sale reference[cite: 2]. FK `FK_sale_item_sale_sale_id` (`ON DELETE CASCADE`)[cite: 2]. |
| `product_id` | `uuid` | NO | None | Catalog product reference[cite: 2]. FK `FK_sale_item_product_product_id` (`ON DELETE RESTRICT`)[cite: 2]. |
| `product_name` | `varchar(200)` | NO | None | Frozen product name snapshot[cite: 2]. |
| `quantity` | `integer` | NO | None | Sold unit count (`quantity > 0`)[cite: 2]. |
| `unit_price` | `numeric(18,2)` | NO | None | Frozen unit price snapshot[cite: 2]. |
| `category_name` | `varchar(120)` | NO | None | Frozen category label snapshot (ADR-004)[cite: 2]. |

---

## 5. `sales.user`
Stores operator identity and credentials[cite: 2].

| Column | Type | Nullable | Default | Description & Constraints |
|---|---|---|---|---|
| `id` | `uuid` | NO | None | Primary Key (`PK_user`)[cite: 2]. |
| `username` | `varchar(120)` | NO | None | Normalized unique username[cite: 2]. Unique index `IX_user_username`[cite: 2]. |
| `password_hash` | `varchar(512)` | NO | None | Irreversible credential hash (D-09)[cite: 2]. Never indexed[cite: 2]. |
| `role` | `varchar(40)` | NO | None | Authorization role (`admin` or `seller`)[cite: 2]. |
