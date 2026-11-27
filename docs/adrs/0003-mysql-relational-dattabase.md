# ADR-0003: MySQL relational database and soft-delete strategy

**Status:** Accepted  
**Date:** 2026-11-26  
**Deciders:** Adrian Mascote (Warehouse Management System — OC CS Speckit)

## Context

The **Warehouse Management System (WMS)** manages heavily interconnected operational data: `Item`, `StorageZone`, `InventoryRecord`, `Supplier`, `Customer`, `Worker`, `Route`, `SupplierOrder`, `CustomerOrder`, `PickTask`, and `BillOfLading`.

We needed to decide:

1. Which database engine and data-modeling approach best enforces integrity across inventory, orders, and shipping records.
2. How to handle record removal without breaking historical reports (Out-of-Stock, Supplier Issues, Shipping Percentiles, and Delivery Mismatches in Features 9–12).
3. How to store timestamps consistently across all receiving, picking, and shipping logs.

Allowing hard deletes (`DELETE FROM ...`) on master records referenced by past orders would either fail with foreign-key errors or cascade-delete historical audit data.

## Decision

Adopt **MySQL** accessed through **Sequelize ORM** with a **normalized relational schema**, **UTC timestamps**, and a **soft-delete / `409 Conflict` protection strategy**:

| Database Rule | Implementation |
|---------------|----------------|
| **Engine & ORM** | MySQL 8+ with Sequelize models and migrations |
| **Normalization** | Master entities (`Item`, `Supplier`, `Customer`, `StorageZone`, `Warehouse`) are separated from transactional records (`InventoryRecord`, `SupplierOrder`, `CustomerOrder`, `ReceivingLogItem`, `PickTask`, `BillOfLading`) via explicit foreign keys |
| **Soft Deletes** | Master records (`Worker`, `Supplier`, `Customer`, `Item`, `Route`, `Warehouse`) use a `status` field (`Active` / `Inactive`) rather than physical deletion when historical references exist |
| **Conflict Protection** | Attempting to hard-delete a record with active dependencies or create a duplicate unique key (e.g., duplicate `sku`, `email`, or `item_id + storage_zone_id` pair) returns **`409 Conflict`** |
| **Timestamps** | All transactional timestamps (`created_at`, `updated_at`, `received_at`, `shipped_at`, `delivered_at`) are stored in **UTC** (`ISO-8601`) |
| **Precision** | Monetary and quantity values use exact numeric types (`DECIMAL` / `INTEGER`) to prevent floating-point rounding errors |

**Invariants:**

1. Foreign-key relationships are enforced at the database level so no orphaned `InventoryRecord`, `PickTask`, or `BillOfLading` rows can exist.
2. Historical orders, receiving logs, and shipping reports (Features 9–12) remain intact even after a `Worker`, `Supplier`, `Customer`, or `Item` is deactivated.
3. All inventory mutations (Receiving in Feature 15, Put-Away in Feature 16, Picking in Feature 18) execute atomically so stock counts never drift out of sync.

## Consequences

### Positive

- **Referential integrity:** MySQL foreign keys guarantee that every `InventoryRecord` points to a valid `Item` and `StorageZone`, and every `BillOfLading` points to a valid `CustomerOrder` and `Route`.
- **Auditability:** Managers can still generate accurate Shipping Percentiles (Feature 11) and Delivery Mismatch reports (Feature 12) for past months even if a supplier or worker is later deactivated.
- **Predictable API errors:** Clear `409 Conflict` responses prevent accidental data corruption and give the frontend actionable error messages.

### Negative / tradeoffs

- Active operational queries (such as dropdowns for creating new orders or assigning routes) must explicitly filter by `status = 'Active'` so deactivated records do not appear.
- Schema changes require Sequelize migrations to keep development and production databases aligned.

## Alternatives considered

| Option | Why not |
|--------|---------|
| **Hard deletes with `ON DELETE CASCADE`** | Permanently destroys historical orders, pick logs, and shipping reports whenever a supplier, item, or worker is removed. |
| **NoSQL document database (e.g., MongoDB)** | Weaker built-in relational integrity across 20 interconnected features and complex reporting joins (Features 9–12). |
| **SQLite** | File-based locking is unsuitable for concurrent multi-user writes from multiple RF scanners and manager dashboards. |

## Related artifacts

- ADRs: [ADR-0001 — Client–server architecture](./0001-client-server-architecture.md), [ADR-0002 — Role-based security and data isolation](./0002-security-architecture.md)
- C4 diagrams: [docs/arch_diagrams/](../arch_diagrams/README.md)
- Feature specs: [Feature 1 — Inventory](../../features/feature-1-inventory.md), [Feature 9–12 — Reporting](../../features/feature-9-outofstock-report.md)
- NFRs: [docs/nfr.md](../nfr.md)