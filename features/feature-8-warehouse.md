# Feature 8: Warehouse Data Management 

## User Stories 

### US-8.1: Add New Warehouse Location 

**As a** warehouse manager 
**I want to** add a new storage location defined by aisle, slot, and bin to the system
**So that** we can start assigning inventory and staff to specific warehouse coordinates.

**Priority:** P1 
**Independent test:** Verify a manager can create a new storage location with an aisle, slot, bin, and storage type, and it saves to the system.
**Acceptance scenarios:** see ### US-8.1 under Gherkin AC 

### US-8.2: Update Warehouse Details 
**As a** warehouse manager 
**I want to** update an existing storage location's aisle, slot, bin, or storage type
**So that** the system reflects changes accurately if a warehouse layout is reorganized.

**Priority:** P2 
**Independent test:** Verify a manager can edit a location's aisle, slot, bin, or type and the system saves the update.
**Acceptance scenarios:** see ### US-8.2 under Gherkin AC 

### US-8.3: Delete Warehouse Location 
**As a** warehouse manager
**I want to** delete a storage location from the system 
**So that** workers don't accidentally assign inventory to a bin that doesn't exist.

**Priority:** P3
**Independent test:** Verify a manager can permanently remove an empty location from the active locations list.
**Acceptance scenarios:** see ### US-8.3 under Gherkin AC 

## Functional Requirements 
* The system must allow managers to add new storage locations using a three-tier hierarchy: **Aisle**, **Slot**, and **Bin**.
* The system must allow managers to update existing location coordinates (aisle, slot, bin) and storage types
* The system must allow managers to delete a storage location when it is dismantled.
* The system must reject the deletion of any storage location that currently has active inventory records tied to it. 

## Assumptions 
* This feature applies to a single physical warehouse facility (no multi-warehouse mapping required). 
* Standard workers do **not** have permission to manage storage zones. 

## Edge Cases 
* **Deletion Protection (Foreign Key Constraint):** If a manager attempts to delete a location containing active inventory, the system must block the action and throw a `409 Conflict` error, instructing the manager to transfer the inventory first. 
* **Duplicate Coordinates:** The system must prevent creating a new location with an `(aisle, slot, bin)` combination that already exists.

## Success Criteria 
* Managers can successfully perform full CRUD operations (Create, Read, Update, Delete) on storage zones. 
* The UI immediately reflects any updates or deletions without requiring a hard page refresh. 
* The database strictly enforces relational integrity so inventory never points at an empty zone. 

## Key Entities 
* **StorageZone** 
* **Manager** 

## Initial Data Model 

### StorageZone 

| Field Name | Data Type | Constraints | Description | 
|---|---|---|---| 
| `zone_id` | Integer | PK, Not Null | Primary key for the storage location | 
| `aisle` | String | Not Null, Unique (with slot, bin) | The warehouse aisle identifier (e.g., `A4`) | 
| `slot` | String | Not Null, Unique (with aisle, bin) | The rack/slot identifier within the aisle (e.g., `S12`) |
| `bin` | String | Not Null, Unique (with aisle, slot) | The specific bin/shelf level within the slot (e.g., `B2`) |
| `zone_type` | String | Not Null | The type of storage location (e.g., Standard, Refrigerated) |


## Gherkin AC 

### US-8.1 
**Scenario: Manager adds a new storage location**
**Given** a manager is on the Zone management screen
**When** the manager enters Aisle "A7", Slot "S03", Bin "B1", and type "Standard"
**And** clicks "Save" 
**Then** the system should create a new storage location in the database.

### US-8.2 
**Scenario: Manager updates a storage location** 
**Given** a storage location profile already exists
**When** the manager updates the zone type to "Refrigerated"
**And** clicks "Update" 
**Then** the system should save the new type for that location

### US-8.3 (Standard)
**Scenario: Manager deletes an empty storage location**
**Given** an empty storage location exists in the system
**When** the manager clicks "Delete" on the location record 
**Then** the system should remove the location from the active database.

### US-8.3 (Edge Case)
**Scenario: Manager attempts to delete a populated storage location**
**Given** a storage location exists in the system and has active inventory items
**When** the manager clicks "Delete" on the location record 
**Then** the system should block the deletion and display an error stating inventory must be moved first.

## Data Ownership & Isolation 

| Rule | Requirement | 
|---|---|
| **Read scope** | Managers can view all zones. Standard workers fetch zones strictly for picking/put-away forms. |
| **Write scope** | Only Managers can update or delete zones. | 
| **Create scope** | Only Managers can create new zones. | 
| **Cross-user-access** | Standard worker attempting POST/PATCH/DELETE -> `403 Forbidden`. |
| **UI scope** | The Zone Management Dashboard is hidden from standard workers. | 
| **Implementation** | Role-based access control (RBAC) enforced via a shared helper in `app/authorization/`. |

## API Requirements 

| Method | Endpoint | Auth | Purpose | 
|---|---|---|---|
| `GET` | `/api/zones` | Yes (All) | Fetches the list of all storage zones | 
| `POST` | `/api/zones` | Yes (Manager) | Creates a new storage zone | 
| `PATCH` | `/api/zones/{zone_id}` | Yes (Manager) | Updates a location's aisle, slot, bin, or type |
| `DELETE` | `/api/zones/{zone_id}` | Yes (Manager) | Deletes a location (if empty) | 

## Screen Requirements

### [View: Zone Management Dashboard] - route name `manage-zones`
* Heading / purpose: Manage Warehouse Locations 
* Primary action: **Add New Zone** (`oc-cta`) - opens creation page.
* Secondary actions: **Edit**, **Delete** (icon buttons next to each row in the data table). 
* Loading / error: `<v-alert type = "error">` if trying to delete a populated zone. 
* **Empty state:** "No storage zones configured." 

## Test Coverage Map 
| Story | Scenario | Test file | Test name | 
|---|---|---|---|
| US-8.1 | Manager adds a new storage zone | `backend/tests/zones.test.js` | `System creates a new storage zone` | 
| US-8.2 | Manager updates a storage zone | `backend/tests/zones.test.js` | `System updates existing zone attributes` | 
| US-8.3 | Manager deletes an empty storage zone | `backend/tests/zones.test.js` | `System deletes empty zone` | 
| US-8.3 | Manager attempts to delete a populated storage zone | `backend/tests/zones.test.js` | `System rejects deletion of populated zone with 409 Conflict` | 

## Agent Implementation request 

```text 

Implement Feature 8 from @features/feature-8-warehouse.md on branch `feature/8-warehouse`.

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests).
Map every Gherkin scenario in the Test Coverage Map; run `npm test` before finishing.
Ensure the DELETE route explicitly checks the Inventory table and throws a 409 Conflict if items are present in the zone.
```

## Definition of Done 

* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met 
* [ ] All mapped tests pass (`npm test`)
* [ ] Test Coverage Map complete 
* [ ] `features/reference/data-model.md` updated (Added StorageZone table)
* [ ] `features/reference/api.md` updated (Added zone CRUD endpoints)

## Out of Scope 
* 3D visual mapping of the warehouse floor 
* Automated suggestions for rearranging storage zones. 
