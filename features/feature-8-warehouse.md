# Feature 8: Warehouse Data Management 

## User Stories 

### US-8.1: Add New Warehouse Location 

**As a** warehouse manager 
**I want to** add a new storage zone to the system 
**So that** we can start assigning inventory and staff to the new Ailses and shelves. 

**Priority:** P1 
**Independent test:** Verify an admin can create a new storage zone with a name and type, and it saves to the system.  
**Acceptance scenarios:** see ### US-8.1 under Gherkin AC 

### US-8.2: Update Warehouse Details 
**As a** warehouse manager 
**I want to** update an existing storage zone's details
**So that** the system reflects changes accurately if an aisle is renamed or changed locations. 

**Priority:** P2 
**Independent test:** Verify a manager can edit a zone's name or location and the system saves the update. 
**Acceptance scenarios:** see ### US-8.2 under Gherkin AC 

### US-8.3: Delete Warehouse Location 
**As a** warehouse manager
**I want to** delete a storage zone from the system 
**So that** workers don't accidentally assign inventory to an ailse that doesne't exist. 

**Priority:** P3
**Independent test:** Verify a maanger can permanently remove a deleted zone from the active locations list. 
**Acceptance scenarios:** see ### US-8.3 under Gherkin AC 

## Functional Requirements 
* The system must allow managers to add new storage zones or aisles to the database. 
* The system must allow managers to update existing zone names and locations 
* The system must allow managers to delete a storage zone when it is dismantled. 
* The system must reject the deletion of any storage zone that currently has active inventory records tied to it. 

## Assumptions 
* This feature applies to a single physical warehouse facility (no multi-warehouse mapping required). 
* Standard workers do have permission to manage storage zones. 

## Edge cases 
* **Deletion Protection (Foreign Key Constraint):** If a manager attempts delete a zone containing active inventory, the system must block the action and throw a `409 Conflict` error, instructing the manager to transfer the inventory first. 
* **Duplicate Names:** The system must prevent creating a new zone with a `zone_name` that already exists. 

## Success Criteria 
* Managers can successfully perform full CRUD operations (Create, Read, Update, Delete) on storage zones. 
* The UI immediately reflects any updates or deletions without requiring a hard page refresh. 
* The database strictly enforces relational integrity so inventory never points at a empty zone. 

## Key Entities 
* Storage Zone 
* Manager 

## Initial Data Model 

### StorageZone 

| Field Name | Data Type | Constraints | Description | 
|---|---|---|---| 
| `zone_id` | Integer | PK, Not Null | Primary key for the storage zone | 
| `zone_name` | String | Not Null, Unique | The name of the zone (e.g., Ailse 4, Bin B) | 
| `zone_type` | String | Not Null | The type of storage zone (e.g., Standard, Refrigerated) |




## Gherkin AC 

### US-8.1 
**Scenario: Manager adds a new storage zone**
**Given** a manager is on the Zone management screen
**When** the manager enters a new zone name like "Ailse 7" 
**And** clicks "Save" 
**Then** the system should create a new storage zone in the database. 

### US-8.2 
**Scenario: Manager updates a storage zone** 
**Given** a storage zone profile already exists 
**When** the manager updates the zone type to "Refrigerated" 
**And** clicks "Update" 
**Then** the system should save the enw type for that zone 

### US-8.3 (Edge Cases)
**Scenario: Manager attempts to delete a populated storage zone**
**Given** a storage zone exists in the system and has active inventory items
**When** the manager clicks "Delete" on the zone record 
**Then** the system should block the deletion and display an error stating inventory must be moved first.

## Data Ownerhsip & Isolation 

| Rule | Requirement | 
|---|---|
| **Read scope** | Managers can view all zones. Standard workers fetch zones strictly for picking/put-away forms. |
| **Write scope** | Only Managers can update or delete zones. | 
| **Create scope** | Only Managers can create new zones. | 
| **Cross-user-access** | Standard worker attempting POST/PATCH/DELETE -> `403 Access Denied`. |
| **UI scope** | The Zone Management Dashboard is hidden from standard workers. | 
| **Implementation** | Role-based access control (RBAC) enforced via a shared helper in `app/authorization/`. |

## API Requirements 

| Method | Endpoint | Auth | Purpose | 
|---|---|---|---|
| `GET` | `/api/zones` | Yes (All) | Fetches the list of all storage zones | 
| `POST` | `/api/zones` | Yes (Manager) | Creates a new storage zone | 
| `PATCH` | `/api/zones/{zone_id} | Yes (Manager) | Yes (Manager) | Deletes a zone (if empty) | 

## Screen Requirement 

### [View: Zone Management Dashboard] - route name `manage-zones`
* Heading / purpose: Manage Warehouse Locations 
* Primary action: **Add New Zone** (`oc-cta`) - opens creation page.
* Secondary actions: **Edit**, **Delete** (icon buttons next to each row in the data table). 
* Loading / error: `<v-alert type = "error">` if trying to delete a populated zone. 
* **Empty state:** "No storage zones configured." 

## Test Coverage Map 
| Story | Scenario | Test file | Test name | 
|---|---|---|---|
| US-8.1 | Manager adds a new storage zone | `backend/tests/zones.teset.js` | `System creates a new storage zone` | 
| US-8.2 | Manager updates a storage zone | `backend/tests/zones.test.js` | `System updates existing zone attributes` | 
| US-8.3 | Manager deletes an empty storage zones | `backend/tests/zones.test.js` | `System deletes ampty zone` | 
| US-8.3 | Manager attempts to delete a populated storage zone | `backend/tests/zones.test.js` | `System rejects deletion of populated zone with 409 Conflict` | 

## Agent Implementation request 

```text 

Implement Feature 8 from @features/feature-8-warehouse-data.md on branch `feature/8-warehouse-data`.

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests).
Map every Gherkin scenario in the Test Coverage Map; run `npm test` before finishing.
Ensure the DELETE route explicitly checks the Inventory table and throws a 409 Conflict if items are present in the zone.
```

## Definition of Done 

* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met (`npm test`)
* [ ] All mapped tests pass(`npm test`)
* [ ] Test Coverage Map complete 
* [ ] `features/reference/data-model.md` updated (Added StorageZone table)
* [ ] `features/reference/api.md` updated (Added zone CRUD endpoints)

## Out of Scope 
* 3D visual mapping of the warehouse floor 
* Automated suggestions for rearranging storage zones. 
