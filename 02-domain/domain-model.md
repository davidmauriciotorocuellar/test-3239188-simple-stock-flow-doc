# Domain Model, Entities & Business Invariants — Simple Stock Flow

## 1. Domain Glossary & Technical Mapping

| Business Term | Functional Definition | Technical Mapping & Scope |
|---|---|---|
| **Product** | Catalog article defined strictly by name, price, stock, category, and optional image key (DP-03)[cite: 2]. | `Product` entity · `product` table[cite: 2] |
| **Category** | Classification group for products. Read-only, pre-seeded set of 5 categories without CRUD management (D-10)[cite: 2]. | `Category` entity · `category` table[cite: 2] |
| **Price** | Current monetary value of a product in the catalog. Strictly positive (`price > 0`)[cite: 2]. | `Money` Value Object · `product.price` column[cite: 2] |
| **Stock** | Available inventory units for a product. Non-negative (`stock >= 0`)[cite: 2]. | `product.stock` column[cite: 2] |
| **Product Image** | Opaque external storage string key (D-08). Never raw binary or local filesystem path[cite: 2]. | `product.image_key` column[cite: 2] |
| **Sale** | Immutable, fully consumed commercial transaction recording timestamp, operator, and line items[cite: 2]. | `Sale` aggregate root · `sale` table[cite: 2] |
| **Sale Line Item** | Individual transaction line featuring product ID, sold quantity, and frozen historical attributes[cite: 2]. | `SaleItem` entity · `sale_item` table[cite: 2] |
| **Quantity** | Sold units per line item. Strictly positive (`quantity > 0`)[cite: 2]. | `Quantity` Value Object · `sale_item.quantity`[cite: 2] |
| **User** | Internal system operator authenticated to register sales and manage stock. No customer entity exists[cite: 2]. | `User` entity · `user` table[cite: 2] |
| **User Role** | Authorization attribute restricted to a closed set of two values: `admin` or `seller`[cite: 2]. | `user.role` column[cite: 2] |
| **Password Hash** | Irreversible credential hash (D-09)[cite: 2]. The domain layer never sees plaintext passwords[cite: 2]. | `user.password_hash` column[cite: 2] |

---

## 2. Frozen Values & Historical Report Stability (ADR-004)

When a sale is recorded, `SaleItem` creates a **frozen snapshot** of product data[cite: 2]:
1. **`unit_price`**: Frozen price snapshot at transaction time[cite: 2].
2. **`product_name`**: Frozen product title snapshot[cite: 2].
3. **`category_name`**: Frozen category label at transaction time (`sale_item.category_name`)[cite: 2].

### Business Justification
Product names or prices may change over time in the catalog. Freezing these attributes ensures historical analytical reports (e.g., September sales) remain completely immutable and mathematically stable, preventing past records from altering whencatalog updates occur[cite: 2].

---

## 3. Aggregate Roots & Detailed Entity Invariants

### 3.1 `Category` (Reference Entity)
- **Name Invariant:** Required non-empty string[cite: 2]. Enforced by domain `Category.Rename` and database unique index `IX_category_name`[cite: 2].
- **Read-Only Nature:** No domain repository port creates, updates, or deletes categories[cite: 2].

### 3.2 `Product` (Catalog Aggregate Root)
- **Name Invariant:** Mandatory trimmed string[cite: 2].
- **Price Invariant:** `price > 0` validated by `Money` value object and `Product.ChangePrice`[cite: 2].
- **Stock Invariant:** `stock >= 0` enforced by domain methods (`Withdraw`/`Restock`) and database check constraint `ck_product_stock_non_negative`[cite: 2].
- **Category Requirement:** Mandatory valid reference to `category.id` via foreign key `FK_product_category_category_id` (`ON DELETE RESTRICT`)[cite: 2].
- **Soft Deletion:** Products are never physically deleted. Soft deletion is handled via `deleted_at` timestamp (ADR-003)[cite: 2].

### 3.3 `Sale` (Sales Aggregate Root)
- **Operator Attribution:** Mandatory non-empty username of registering operator[cite: 2].
- **Completeness:** Must contain at least one line item to be confirmed (`Sale.EnsureConfirmable`)[cite: 2].
- **Line Deduplication:** A single sale cannot duplicate the same product across multiple lines[cite: 2]. Enforced by database unique composite index `(sale_id, product_id)`[cite: 2].
- **Stock Decrement:** Adding a line item automatically executes `Product.Withdraw` in the same atomic transaction[cite: 2].

### 3.4 `User` (Identity Aggregate Root)
- **Username Normalization:** Usernames are converted to lowercase, trimmed, and must be unique (`IX_user_username`)[cite: 2].
- **Password Security:** Plaintext passwords are never stored or logged[cite: 2].
- **Role Validation:** Must be strictly `admin` or `seller`[cite: 2].
