# Requirements Specification — Simple Stock Flow

## 1. Functional Requirements (FR)

### FR-01: Immutable Category Seed
The system must pre-seed 5 fixed category records in the initial database migration (`General`, `Herramientas`, `Electricidad`, `Fontanería`, `Pinturas`)[cite: 2]. Categories are read-only[cite: 2].

### FR-02: Product Catalog CRUD & Soft Delete
Operators must be able to create, update, and search products[cite: 2]. Physical deletion is prohibited; product deletion must execute soft deletion via `deleted_at` (ADR-003)[cite: 2].

### FR-03: Stock Management & Invariant Enforcement
Stock decrement (`Withdraw`) and restock (`Restock`) operations must atomically validate that `stock >= 0`[cite: 2]. Any operation attempting to drop stock below zero must fail immediately[cite: 2].

### FR-04: Immutable Sale Registration
Register sales containing one or more line items[cite: 2]. The operation must atomically:
1. Validate product active status[cite: 2].
2. Decrement product stock[cite: 2].
3. Copy and freeze current product name, price, and category label into `sale_item`[cite: 2].

### FR-05: Aggregated Analytical Sales Report
Produce aggregated sales reports over a specified date range[cite: 2]. Grouping must strictly execute by `product_id`, `product_name`, and frozen `category_name`[cite: 2].

---

## 2. Non-Functional Requirements (NFR)

### NFR-01: Engine-Level Data Integrity
Integrity constraints must be enforced directly inside PostgreSQL 16+ engine (`CHECK`, `FOREIGN KEY`, `UNIQUE`)[cite: 2], ensuring raw SQL scripts or direct database operations cannot bypass domain rules[cite: 2].

### NFR-02: Password Security & Privacy
User passwords must be hashed before persistence (D-09)[cite: 2]. `password_hash` must never be logged, indexed, or exposed in API payloads[cite: 2].

### NFR-03: Query Performance & Covering Indexing
Sales report aggregation queries must execute via covering indexes (`sale_item(sale_id, product_id) INCLUDE (quantity, unit_price)`) to allow index-only scans without reading underlying heap tables[cite: 2].
