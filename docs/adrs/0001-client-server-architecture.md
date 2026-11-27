# ADR-0001: Client-server architecture for Warehouse Management System 

**Status:** Accepted  
**Date:** 2026-11-26  
**Deciders:** Adrian Mascote (Warehouse Management System - OC CS SpecKit)

## Context

The **Warehouse Management System (WMS)** is a multi-role operational application supporting Warehouse Managers, Floor Workers, (Pickers and Receivers), Unlike a single-user app, multiple employees read and update shared warehouse data simultaneously-including `InventoryRecord`, `SupplierOrder`, `CustomerOrder`, `PickTask`, and `BillOfLading`- from both desktop web browsers and handheld RF scanners.

We needed to decide:

1. Whether client devices hold authoritative state or communicate with a centralized server. 
2. How to identify the caller and enforce role permissions (Manager Vs. Worker vs. Driver) on every API request.
3. How to keep real-time inventory counts consistent across receiving, put-away, picking, and shipping workflows. 

An offline-first or client-storage design (`localStorage` as source of truth) woulc cause inventory desynchronization between workers and could not enforce role-based access control (RBAC) across shared relational data. 

## Decision

Adopt a **classic client–server split** with a **stateless REST API**, **centralized MySQL persistence**, and **server-enforced RBAC**:

| Layer | Choice |
|-------|--------|
| **Client** | Vue 3 SPA (Vite), Vuetify 4, axios (responsive for both desktop dashboards and RF scanner viewports) |
| **Server** | Node.js + Express + Sequelize (ES modules) |
| **Database** | MySQL — single shared relational database with normalized tables (`Item`, `StorageZone`, `InventoryRecord`, etc.) |
| **Transport** | JSON over HTTPS; API base path `/api/` |
| **Auth** | Username + password; bcrypt hashes; **JWT + Session table** (token stored server-side, revocable on logout) |
| **Client session hint** | Login response stored in `localStorage` key `user`; axios attaches `Authorization: Bearer <token>` on every request |
| **Authorization** | `authenticate` middleware sets `req.user.id` and `req.user.role`; shared RBAC helpers in `app/authorization/`enforce Manage-only CRUD (`403 Forbidden` for unatuhorized roles) and Driver route scoping (`driver_id = req.user.id`) |
| **Repo layout** | Monorepo: `frontend/` + `backend/` + `features/` + `docs/` |

```text
Browser / RF Scanner (Vue SPA)                   Express API                 MySQL
─────────────────                                ───────────                 ─────
localStorage["user"]  ──Bearer──►               authenticate middleware  ──► sessions, users
router guards (UI)                              app/authorization/ (RBAC)    items,inventory,zones
                                                controllers + Sequelize      orders, pick_tasks,bols
```

**Invariants** (must hold in every feature):

1. The server and MySQL database are the **single source of truth** for all inventory, orders, and operational logs
2. Every authenticated request resolves to **exactly one** active `worker_id` and `job_role` from a valid session row.
3. **No endpoint** allows a standard worker to execute Manager-only mutations (master data CRUD, deletions, or financial/analytics reports) - unauthorized role attempts return `403 Forbidden`.
4. Deactivated workers (`status = Inactive`) are immediately blocked from authenticating on both web and RF scanner endpoints. 

## Consequences

### Positive

- Clear Spec-Driven Development (SDD) boundaries; feature specs (`features/feature-1` through `feature-20`) -> REST API contract -> Sequelize models -> Jest/Vitest tests.
- Real-time inventory consistency: barcode scans in Receiving (Feature 15), Put-Away (Feature 16), and Picking (Feature 18) immediately reflect in Out-of-Stock Reports (Feature 9).
- Session revocation on logout or worker deactivation (Feature 5) is enforced server-side via the `Session` table.
- One unified backend serves both the Manager Web Portal (Feature 20) and RF Scanner workflows (Feature 13).

### Negative / tradeoffs

- Requires a running MySQL instance and Express backend for all operations (RF scanners cannot operate offline if warehouse Wi-Fi drops).
- `localStorage` session is a **convenience cache** for UI routing only; it is not authoritative (`401 Unauthorized` clears it and redirects to login).
- Running separate dev servers for frontend and backend requires CORS configuration during local development. 


## Alternatives considered

| Option | Why not |
|--------|---------|
| **localStorage / offline-first client** | Cannot synchronize live inventory counts across multiple simultaneous pickers and receivers; no real RBAC enforcement. |
| **JWT in cookie only, no Session table** | Cannot immediately revoke an active session when a manager deactivates a worker(Feature 5). |
| **Separate backends for Web vs. RF Scanner** | Duplicates business logic and database models; a single Express API handles both clients cleanly. |
| **GraphQL** | Adds schema complexity unnecessary for standard CRUD and operational REST workflows. |


## Related artifacts

- ADRs: [ADR-0002 — Role-based security and data isolation](./0002-security-architecture.md), [ADR-0003 — MySQL relational database](./0003-mysql-relational-database.md), [ADR-0004 — Vue 3 as the frontend framework](./0004-vue-frontend-framework.md), [ADR-0006 — Node.js and Express as the API runtime](./0006-node-express-api.md), [ADR-0007 — REST and flat JSON](./0007-rest-json-api.md)
- C4 diagrams: [docs/arch_diagrams/](../arch_diagrams/README.md) (context, container, component)
- Feature specs: [Feature 1 — User Authentication](../../features/feature-1-user-auth.md) (identity foundation); Features 2–3 (list/todo isolation)
- Cursor rules: [auth-patterns.mdc](../../.cursor/rules/auth-patterns.mdc), [security.mdc](../../.cursor/rules/security.mdc), [frontend-services.mdc](../../.cursor/rules/frontend-services.mdc), [project-structure.mdc](../../.cursor/rules/project-structure.mdc)
- Reference: [api.md](../../features/reference/api.md), [data-model.md](../../features/reference/data-model.md)