# ADR-0002: Role-based security and data isolation

**Status:** Accepted  
**Date:** 2026-11-26  
**Deciders:** Adrian Mascote (Warehouse Management System — OC CS Speckit)

## Context

The **Warehouse Management System (WMS)** serves three distinct user groups—**Warehouse Managers**, **Floor Workers** (Pickers and Receivers), and **Delivery Drivers**—accessing the same backend API from both desktop web browsers and handheld RF scanners.

We needed to decide:

1. How to authenticate users across both the Web Portal (Feature 20) and RF Scanners (Feature 13) without duplicating security logic.
2. How to prevent Floor Workers or Drivers from accessing Manager-only master data, deletions, or financial/analytics reports.
3. How to immediately revoke access when a Manager deactivates an employee (Feature 5).

Relying only on frontend route guards (hiding buttons in Vue) is unsafe because any user with a valid token could still send direct HTTP requests to restricted API endpoints.

## Decision

Adopt a **layered, server-enforced security model** combining **bcrypt password/PIN hashing**, **JWT + Session table authentication**, and **centralized Role-Based Access Control (RBAC)**:

| Security Layer | Implementation |
|----------------|----------------|
| **Credential storage** | Passwords and scanner PINs hashed with `bcrypt`; plaintext credentials are never stored or logged |
| **Session management** | Login returns a signed JWT backed by a server-side `Session` table row; logout or worker deactivation clears the session immediately |
| **Authentication middleware** | `authenticate` middleware validates the `Authorization: Bearer <token>` header, verifies the active `Session` row, and attaches `req.user.id` and `req.user.role` |
| **Role enforcement (RBAC)** | Shared authorization helpers in `app/authorization/` check `req.user.role` before executing controller logic (`403 Forbidden` on unauthorized roles) |
| **Record scoping** | Delivery Drivers are scoped strictly to routes and Bills of Lading assigned to their `driver_id` (`req.user.id`) |
| **Frontend guards** | Vue Router navigation guards hide unauthorized views and redirect to `/login` on `401 Unauthorized` |

**Access Matrix:**

| Role | Permitted Capabilities |
|------|------------------------|
| **Manager** | Full CRUD on master data (Items, Suppliers, Customers, Workers, Routes, Warehouses, Company), operational oversight, and all reports (Features 9–12) |
| **Floor Worker** | RF Scanner login, Receiving (Feature 15), Put-Away (Feature 16), and Order Picking (Feature 18) |
| **Delivery Driver** | View assigned Routes and complete Shipping/Delivery confirmations (Feature 19) |

**Invariants:**

1. Every protected API route runs `authenticate` middleware before reaching controller logic.
2. Role checks are always enforced on the server; UI hiding is strictly for user experience.
3. Workers with `status = Inactive` cannot log in and have all active sessions invalidated immediately.
4. Failed authentication returns `401 Unauthorized`; authenticated requests lacking role permission return `403 Forbidden`.

## Consequences

### Positive

- **Unified security:** Both Web Portal (Feature 20) and RF Scanner (Feature 13) share the exact same authentication and RBAC pipeline.
- **Instant revocation:** Deactivating a worker in Feature 5 immediately blocks their RF scanner and web sessions via the `Session` table.
- **Consistent enforcement:** Centralizing role checks in `app/authorization/` prevents individual feature routes from accidentally forgetting permission checks.

### Negative / tradeoffs

- Every API request requires a lightweight database lookup against the `Session` table to confirm the token has not been revoked.
- Adding new job roles in the future requires updating the shared RBAC helper definitions and route mappings.

## Alternatives considered

| Option | Why not |
|--------|---------|
| **Frontend-only role hiding** | Insecure; users could bypass the Vue UI and call `/api/suppliers` or `/api/workers` directly. |
| **Stateless JWT without a Session table** | Cannot immediately revoke an active token if an RF scanner is lost or a worker is deactivated. |
| **Separate auth systems for Web and RF Scanners** | Doubles maintenance effort and increases the risk of inconsistent password/session rules. |

## Related artifacts

- ADRs: [ADR-0001 — Client–server architecture](./0001-client-server-architecture.md), [ADR-0003 — MySQL relational database and soft-delete strategy](./0003-mysql-relational-database.md)
- Feature specs: [Feature 5 — Workers](../../features/feature-5-workers.md), [Feature 13 — RF Scanner Login](../../features/feature-13-scanner-login.md), [Feature 20 — Web Portal Login](../../features/feature-20-web-login.md)
- NFRs: [docs/nfr.md](../nfr.md)