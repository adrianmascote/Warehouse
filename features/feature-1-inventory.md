# Feature: Inventory Data Management 

## User Stories 

### US-1.1: Add New Inventory 
**As a** warehouse worker 
**I want to** manually add new inventory records
**so that** the system tracks newly arrived products even if an RF scanner is unavailable

**Priority:** P1 
**Independent test:** Verify that a worker can submit a new inventory record with an item, storage zone, and quantity, and it appears in the database view. 
**Acceptance scenarios** See ### US-1.1 under Gherkin AC 

### US-1.2: Update Inventory Amounts (Manually)
**As a** warehouse worker 
**I want to** update existing inventory quantities and storage zones 
**so that** the system accurately reflects when items hsave been moved, recounted, or picked for an order. 

**Priority** P1 
**Independent test:** Verify a worker can change the quantity of an existing inventory record and the system saves the new amount. 
**Acceptance scenarios:** see ### US-1.2 under Gherkin AC 

### US-1.3: Delete Inventory Records 
**As a** warehouse manager
**I want to** delete inventory records for items we no longer carry
**So that** the database stays uncluttered so that workeres aren't confused by discontinued itmes. 

**Priority** P2
**Independent test:** Veriy a manager can delete an inventory record and it is completely removed from the active inventory list. 
**Acceptance scenarios:** see ### US-1.3 under Gherkin AC 

## Functional Requirements 
* The system must allow authenticated workers and managers to manually add new inventory records if a scanner is unavailable. 
* The system must allow authenticated workers and managers to update existing inventory quantities (`current_inv_qty`) and storage zones (`zone_id`)
* The system must allow only managers to delete inventory records. 
* The system must track `current_inv_qty`, `min_level`, and `max_level` for each item in a given `StorageZone`.

## Assumptions 
* Items (Feature 4) and Storage Zones (Feature 8) already exist before an `InventoryRecord` is created for them. 
* Automated receiving (Feature 15), put-away (Feature 16), and picking (Feature 18) update this same `InventoryRecord` table 

## Edge Cases 
* **Duplicate Item + Zone:** Attempting to create a second `InventoryRecord` for an `(item_id, zone_id)` pair that already exists returns `409 Conflict` and prompts the user to update the existing record instead. 
* **Negative Quantities:** Manual input of a negative `current_inv_qty` is rejected `400 Bad Request`. 
* **Exceeding Max Capacity:** If a manual update sets `current_inv_qty > max_level`, the system displays a capacity warning before saving. 

## Success Criteria 
* Workers and managers can create and update inventory recoreds cleanly without a scanner.
* Only managers can permanently delete inventory records. 
* Updates immediately reflect in Out-of-Stock / Low Inventory Reports (Feature 9) and Picking (Feature 18).

## Key Entities 
* **InventoryRecord**
* **Item (Referenced)** 
* **StorageZone (Referenced)**
* **Worker / Manager**



## Initial Data Model 

### InventoryRecord

| Field Name | Data Type | Constraints | Description | 
| --- | --- | --- | --- | 
| `inventory_id` | Integer | PK, Not Null | Primary key for the inventory record |
| `item_id` | Integer | PK, Not Null | Foreign Key referencing `Item` (Feature 4) | 
| `zone_id` | Integer | FK, Not Null | Foreign key referencing `StorageZone` (Feature 8) |
| `current_inv_qty` | Integer | Not Null, Default: `0` | The current quantity of the item in stock at this zone | 
| `min_level` | Integer | Nullable | Minimum required stock level before low-stock alerts (Feature 9) |
| `max_level` | Integer | Nullable | Maximum storage capacity for this item in this zone |

## Gherkin AC 

### US-1.1 

**Scenario: Worder adds new inventory**
**Given** a worker is logged into the Inventory Management screen
**When** the worker selects an item, enters a quantity of "50", and selects storage zone "Aisle 4, Slot S01, Bin B1"
**And** clicks "Save" 
**Then** the system creates a new `InventoryRecord` for that item
**And** displays the quantity "50" at that storage zone. 

### US-1.2 
**Scenario: Worker updates inventory quantity** 
**Given** an existing inventory record shows "50" items in Aisle 4, Slot S01, Bin B1"
**When** the worker updates the quantity to "40" 
**And** clicks "Update" 
**Then** the system saves the new amount 
**And** displays the quantity "40" at that storage zone.

### US-1.3 
**Scenario: Manager deletes an inventory record**
**Given** a manger is viewing an existing inventory record
**When** the manager clicks "Delete" and confirms the prompt 
**Then** the system permanently removes the inventory record from the database. 

## Data Ownership & Isolation 

| Rule | Requirement | 
|---|---| 
| **Read scope** | All authenticated workers and managers can view inventory records. | 
| **Write scope** | Workers and Managers can update quantities/zones. Only Managers can delete records. | 
| **Create scope** | Workers and Managers can create new inventory records. | 
| **Cross-user-access** | Standard worker attempting `DELETE` -> `403 Forbidden`. | 
| **UI scope** | The "Delete" button is hidden for standard workers and visible only to Managers. | 
| **Implementation** | Role-based access control (RBAC) enforced via shared helper `app/authorization/`. | 

## API Requirements 

| Method | Endpoint | Auth | Purpose | 
|---|---|---|---|
| US-1.1 | Worker adds new inventory | `backend/tests/inventory.test.js` | `System creates a new inventory record` | 
| US-1.2 | Workers updates inventory quantity | `backend/tests/inventory.test.js` | `System updates existing inventory quantity` | 
| US-1.3 | Manager deletes an inventory record | `backend/tests/inventory.test.js` | `Manager deletes inventory record while worker receives 403` |

## Screen Requirements 
### [View: Inventory Management Dashboard] - route name `manage-inventory`

* Heading / Purpose: Manage Warehous Inventory 
* Primary action: **Add Inventory** (`oc-cta`)
* Secondary actions: **Edit**, **Delete** (Manager only)
* Fields: Item Dropdown (SKU/Description), Storage Zone Dropdown, Current Qty, Min Level, Max Level
* **Empty state:** "No inventory records found."
* Loading/error: `<v-alert type = "error">` for negative quantities or duplicate item/zone conflicts.

## Test Coverage Map 

| Story | Scenario | Test file | Test name | 
|---|---|---|---|---|
| US-1.1 | Worker adds new inventory | `backend/tests/inventory.test.js` | `System creates a new inventory record` | 
| US-1.2 | Worker updates inventory quantity | `backend/tests/inventory.test.js` | `System updates existing inventory quantity` | 
| US-1.3 | Manager deletes an inventory record | `backend/tests/inventory.test.js` | `Manager deletes inventory record while worker receives 403` | 


## Agent Implementation Request 

```text 
Implement Feature 1 from @features/feature-1-inventory.md on branch `feature/1-inventory`.

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests). Map every Gherkin scenario in the Test Coverage Map; run `npm test` before finishing. 
```
## Definition of Done 

* [ ] Backend and frontend implemented per this spec
* [ ] Success Criteria met 
* [ ] All mapped tests pass (`npm test`)
* [ ] Test Coverage Map complete 
* [ ] `features/reference/data-model.md` updated (Added InventoryRecord Table)
* [ ] `features/reference/api.md` updated (Added inventory CRUD endpoints)

## Out of Scope 
* Automated cycle counting schedules 
* Lot/batch expiration date tracking 