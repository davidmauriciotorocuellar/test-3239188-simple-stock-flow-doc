# Physical Data Dictionary — PostgreSQL 16 (`sales` schema)

All tables use singular naming conventions ASCII (`PascalCase` in domain code, `snake_case` in database).

## 1. `sales.category`
Stores reference classifications.

| Column | Type | Nullable | Default | Description & Constraints |
|---|---|---|---|---|
| `id` | `uuid` | NO | None | Primary Key (`PK_category`). |
| `name` | `varchar(120)` | NO | None | Category label. Unique index `IX_category_name`. |

---

## 2. `sales.product`
Stores catalog items.

| Column | Type | Nullable | Default | Description & Constraints |
|---|---|---|---|---|
| `id` | `uuid` | NO | None | Primary Key (`PK_product`). |
| `name` | `varchar(200)` | NO | None | Product title. |
| `price` | `numeric(18,2)` | NO | None | Unit price. Monocurrency (D-05). |
| `stock` | `integer` | NO | None | Inventory stock. Constraint `ck_product_stock_non_negative` (`stock >= 0`). |
| `category_id` | `uuid` | NO | None | FK to `sales.category(id)` (`ON DELETE RESTRICT`). |
| `image_key` | `varchar(512)` | YES | None | Opaque external binary key (D-08). `NULL` when absent. |
| `deleted_at` | `timestamptz` | YES | None | Soft deletion timestamp (ADR-003). |

---

## 3. `sales.sale`
Stores header records of sales transactions.

| Column | Type | Nullable | Default | Description & Constraints |
|---|---|---|---|---|
| `id` | `uuid` | NO | None | Primary Key (`PK_sale`). |
| `sold_at` | `timestamptz` | NO | None | Transaction UTC timestamp. Indexed (`IX_sale_sold_at`). |
| `sold_by` | `varchar(120)` | NO | None | Operator username who registered the sale. |
| `sold_by_user_id` | `uuid` | NO | None | FK to `sales.user(id)` (`ON DELETE RESTRICT`). |

---

## 4. `sales.sale_item`
Stores line items within a sale.

| Column | Type | Nullable | Default | Description & Constraints |
|---|---|---|---|---|
| `id` | `uuid` | NO | None | Primary Key (`PK_sale_item`). |
| `sale_id` | `uuid` | NO | None | Parent sale reference. FK `FK_sale_item_sale_sale_id` (`ON DELETE CASCADE`). |
| `product_id` | `uuid` | NO | None | Catalog product reference. FK `FK_sale_item_product_product_id` (`ON DELETE RESTRICT`). |
| `product_name` | `varchar(200)` | NO | None | Frozen product name snapshot. |
| `quantity` | `integer` | NO | None | Sold unit count (`quantity > 0`). |
| `unit_price` | `numeric(18,2)` | NO | None | Frozen unit price snapshot. |
| `category_name` | `varchar(120)` | NO | None | Frozen category label snapshot (ADR-004). |

---

## 5. `sales.user`
Stores operator identity and credentials.

| Column | Type | Nullable | Default | Description & Constraints |
|---|---|---|---|---|
| `id` | `uuid` | NO | None | Primary Key (`PK_user`). |
| `username` | `varchar(120)` | NO | None | Normalized unique username. Unique index `IX_user_username`. |
| `password_hash` | `varchar(512)` | NO | None | Irreversible credential hash (D-09). Never indexed. |
| `role` | `varchar(40)` | NO | None | Authorization role (`admin` or `seller`). |

---

## 6. Concurrency & Audit Design Notes
- **`xmin` System Column (`xid`)**: `product` relies on PostgreSQL's built-in `xmin` system column as the concurrency witness for optimistic concurrency control (D-04 / ADR-002). It is managed automatically by PostgreSQL and incremented on each `UPDATE`, so it is omitted from declared `information_schema` columns.
- **No Audit Columns (`created_at` / `updated_at`)**: The schema explicitly omits audit timestamps across all tables. Values are assigned strictly by the domain layer without database defaults or triggers. The single business timestamp for sales operations is `sale.sold_at`, and the state transition timestamp for product lifecycle is `product.deleted_at`.
