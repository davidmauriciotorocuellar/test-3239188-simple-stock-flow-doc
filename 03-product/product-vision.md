# Product Vision, Scope & Business Boundaries — Simple Stock Flow

## 1. Product Vision
Simple Stock Flow is a lightweight, spec-driven inventory and sales registration platform. It is built for business environments that demand strict consistency, immutable sales reporting, and atomic stock controls without unnecessary architectural bloat[cite: 2].

## 2. Inviolable Product Rules

### DP-01: Historical Report Aggregation
Sales reports aggregate data by product ID, frozen product name, and frozen category label[cite: 2]. Historical report integrity takes absolute priority over catalog updates[cite: 2].

### DP-02: Privacy & Seller Attribution
Sales reports are grouped strictly by commercial items, never broken down by seller/operator[cite: 2]. Operator attribution is preserved solely for audit purposes[cite: 2].

### DP-03: Strict Product Attribute Boundary
Catalog products consist exclusively of:
- `name`
- `price`
- `stock`
- `category_id`
- `image_key` (optional)

Attributes such as secondary descriptions, SKUs, barcode numbers, dimensions, or custom metadata are strictly forbidden[cite: 2].

### DP-05: Single-Currency Standard
The application and database operate strictly under a single-currency domain (D-05)[cite: 2]. Currency columns are omitted system-wide[cite: 2].
