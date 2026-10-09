# System Context & Scope — Simple Stock Flow

## 1. System Overview
Simple Stock Flow is an internal inventory and sales management system designed to track catalog products, control available stock levels, record immutable sales operations, and produce aggregated analytical reports.

## 2. Business Scope & Boundaries
- **Monocurrency Design:** The system operates strictly in a single currency without multi-currency or exchange rate conversions (D-05).
- **Internal Operators Only:** Users are internal operators categorized into two closed roles: `admin` and `seller`. There is no customer or end-buyer entity in the system.
- **Opensearch/External Binary Storage:** Product images are stored as opaque string keys referencing external binary storage, avoiding blob database overhead (D-08).
- **No External Payment Gateways:** Payment processing, credit card clearing, and third-party financial integrations are strictly out of scope.
