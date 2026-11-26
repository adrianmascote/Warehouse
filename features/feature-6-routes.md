# Feature 6: Route Data Management 

## User Stories 

### US_6.1: Add New Route 
**As a** warehouse manager 
**I want to** add a new delivery route to the system. 
**So that** we can assign customer orders to this new delivery path 

**Priority:** P1
**Independent test:** Verify amanager can create a new route with a name and geogrpahic area, and optional assigned driver, and it saves to the database.
**Acceptance scenarios:**  see ### US-6.1 under Gherkin AC 

### US-6.2: Update Route Details 
**As a** warehouse manager 
**I want t** update an existing route's details 
**So that** the system reflects changes in delivery zones, paths, or delivery drivers. 

**Priority:** P2 
**Independent test:** Verify a manager can edit a route's designated area or assigned driver and the system saves the new information.
**Acceptance scnenarios:** see ### US-6.2 under Gherkin AC 

### US-6.3: Delete Route 
**As a** warehouse manager 
**I want to** delete a route that is no longer in service 
**So that** dispatchers do not accidentally assign orders to a dead route

**Priority:** P3 
**Independent test:** verify a manager can delete an unused route, permanently removing it from the available route list. 
**Acceptance scenarios:** see ### US-6.3 under Gherkin AC 

## Functional Requirements 
* The system must allow managers to add new delivery routes with a unique name, geographic area, and optional assigned `driver_id`.
* The system must allow managers to update existing route names, geographic areas, and assigned drivers.
* The system must allow managers to delete routes that are no longer used. 
* The system must block the deletion of any route currently tied to active `BillOfLading` shipments (Feature 19).

## Assumptions 
* Drivers already exist in the `Worker` table (Feature 5) before being assigned to a route.
* Standard workers and drivers cannot create, edit, or delete routes. 

## Edge Cases 
* **Duplicate Route Name:** Attempting to create a route with a `route_name` that already exists returns `409 Conflict`.
* **Active Shipment Protection:** Attempting to delete a route that has `In Transit` shipments returns `409 Conflict` and prompts the manager to reassign or complete those deliveries first.

## Success Criteria 
* Managers can perform full CRUD operations on delivery routes. 
* Active routes and their assigned drivers are immediately available to the Shipping & Delivery workflow (Feature 19).

## Key Entities 
* **Route** 
* **Manager** 
* **Worker (Driver)**

## Initial Data Model 

### Route 

| Field Name | Data Type | Constraints | Description | 
|---|---|---| 
| `route_id` | Integer | PK, Not Null | Primary key for the route | 
| `route_name` | String | Unique, Not Null | The name of the route (e.g., `Northside`) |
| `geographic_area` | String | Not Null | The geographic area covered by a route |
| `driver_id` | Integer | FK, Nullable | Foreign key referencing the assigned `Worker` (Driver) |

## Gherkin AC 

### US-6.1 
**Scenario: Manager adds a new route**
**Given** a manager is on the Route Management screen
**When** the manager enters a route name like "Northside" and a geographic area
**And** clicks "Save"
**Then** the system creates a new route profile
**And** the route appears in the active routes list. 

### US-6.2 
**Scenario: Manager updates a route** 
**Given** a route profile already exists in the system 
**When** the manager changes the geographic area details 
**And** clicks "Update" 
**Then** the system should save the new area for that route 

### US-6.3
**Scenario: Manager deletes a route**
**Given** a route profile already exists in the system
**When** the manager clicks "Delete" on the route 
**Then** the system should remove the route from the active list 

## Data Ownership & Isolation 
| Rule | Requirement | 
|---|---|
| **Read scope** | Managers can view all routes. Drivers can view routes assigned to them (`driver_id` = req.user.id`). |
| **Write scope** | Only Managers can update or delete routes. | 
| **Create scope** | Only Managers can create new routes. | 
| **Cross-user-access** | Standard worker attempting POST/PATCH/DELETE -> `403 Forbidden`. | 
| **UI scope** | Route Management Dashboard is only visible to Managers. | 
| **Implementation** | Role-based access control (RBAC) enforced via shared helper in `app/authorization/`. |

## API Requirements
| Method | Endpoint | Auth | Purpose | 
|---|---|---|---|
| `GET` | `/api/routes` | Yes (All) | Fetches all delivery routes (filtered for drivers where needed) | 
| `POST` | `/api/routes` | Yes (Manager) | Creates a new delivery route | 
| `PATCH` | `/api/routes/{route_id}` | Yes (Manager) | Updates route name, area, or assigned driver | 
| `DELETE` | `/api/routes/{route_id}` | Yes (Manager) | Deletes a route (if no active shipments are attached) | 

## Screen Requirements

### [View: Route Management Dashboard] - route name `manage-routes`
* Heading/ purpose: Manage Delivery Routes 
* Primary action: **Add New Route** (`oc-cta`)
* Secondary actions: **Edit**, **Delete**
* Fields: Route Name, Geographic Area, Assigned Driver Dropdown 
* **Empty state:** "No delivery routes configured." 
* Loading / error: `<v-aler type = "error">` for duplicate route names or active shipment deletion conflicts. 

## Test Coverage Map 
| Story | Scenario | Test file | Test name | 
|---|---|---|---|
| US-6.1 | Manager adds a new route | `backend/tests/routes.test.js` | `System creates a new delivery route` | 
| US-6.2 | Manager updates a route | `backend/tests/rooutes.test.js` | `System updates existing route's details` |
| US-6.3 | Manager deletes a route | `backend/tests/routes.test.js` | `System deletes unused delivery route` | 

## Agent Implementation Request 
```text 
Implement Feature 6 from @features/feature-6-routes.md on branch `feature/6-routes`.

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests). 
Map every Gherkin scenario in the Test Coverage Map; run `npm test` before finishing.
```

## Definition of Done 
* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met 
* [ ] All mapped tests pass (`npm test`) 
* [ ] Test Coverage Map complete 
* [ ] `features/reference/data-model.md` updated (Added Route table)
* [ ] `features/reference/api.md` updated (Added route CRUD endpoints)

## Out of Scope 
* GPS turn-by-turn navigation or live traffic optimization 
* Multi-stop delivery route sequencing. 
