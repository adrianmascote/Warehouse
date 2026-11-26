# Feature 4: Item Data Management 

## User Stories 

### US-4.1: Add New Item Profile 
**As a** warehouse manager
**I want to** add a new item to the system's master catalog 
**So that** the warehouse can start receiving, storing, and tracking this new product. 

**Priority:** P1 
**Independent test:** Verify a manager can create a new item profile with a SKU, description, UPC barcode, cost, and price, and it saves to the database.
**Acceptance scenarios:** see ### US-4.1 under Gherkin AC 

## US-4.2: Update Item Profile 
**As a**  warehouse manager 
**I want to** update an existing item's details such as it's description, pricing, or reorder thresholds
**So that** the product catalog remains accurate even if details change over time. 

**Priority:** P2 
**Independent test:** Verify that a user can edit an item's details and the system saves the updated information. 
**Acceptance scenarios:** see ### US-4.2 under Gherkin AC 

### US-4.3 : Delete Item Profile 
**As a** warehouse manager 
**I want to** delete an item from the master catalog 
**So that** we do not accidentally order or track items that we no longer sell

**Priority:** P3
**Independent test:** Verify a user can delete an unreferenced item's profile and it is removed from the system completely. 
**Acceptance scenarios:** see ### US-4.3 under Gherkin AC 

## Functional Requirements 
* The system must allow managers to add new items to the catalog with a unique `SKU`, UPC barcodes, description, `case_cost`, `price`, `required_zone_type`, and optional reorder settings (`preffered_supplier_id`, `min_level`, `reorder_qty`) 
* The system must allow managers to update item descriptions, pricing, barcodes, and reorder thresholds.
* The system must allow managers to delete an item from the catalog if it has no active inventory or open orders. 
* The system must block the deletion of any item currently referenced by active `InventoryRecord`, `SupplierOrderItem`, or `CustomerOrderItem` rows.

## Assumptions 
* Standard floor workers can read item details (e.g., during scanning) but cannot create, edit, or delete catalog items. 
* Each item has a  unique `sku`

## Edge Cases
* **Duplicate SKU or UPC:** Attempting to create an item with a `sku`, `item_upc`, or `case_upc` that already exists returns `409 Conflict`.
* **Active Reference Protection:** Attempting to delete an item that has existing inventory or order history returns `409 Conflict`, instructing the manager to clear inventory or mark the item inactive instead. 

## Success Criteria 
* Managers can perform full CRUD operations on the master item catalog. 
* Barcode scans in Receiving (Feature 15), Put-Away (Feature 16), and Picking (Feature 18) immediately resolve newly added items. 

## Key Entities 
* **Item**
* **Manager**
* **Supplier (Referenced)** 

## Initial Data Model 

### Item 

| Field Name | Data Type | Constraints | Description |
|---|---|---|---|
| `item_id` | Integer | PK, Not Null | Primary key for the item | 
| `sku` | String | Unique, Not Null | The SKU identifier for the item | 
| `item_upc` | String | Unique, Nullable | The UPC barcode for an individual unit | 
| `case_upc` | String | Unique, Nullable | The UPC barcode for full case | 
| `description` | String | Not Null | The name/description of the item | 
| `case_cost` | Decimal | Nullable | The purchase cost per case | 
| `price` | Decimal | The selling price of the item |
| `required_zone_type` | String | Not Null, Default: `Standard` | Required storage type (e.g., Standard, Refrigerated) |
| `preferred_supplier_id` | Integer | FK, Nullable | Default `Supplier` used for auto-generated POs (Feature 14) | 
| `min_level` | Integer | Nullable | Low stock threshold for alerts and auto-ordering (Features 9 & 14) | 
| `reorder_qty` | Integer | Nullable | Default quantity requested on auto-generated POs (Feature 14) |


## Gherkin AC 
### US-4.1 
    
**Scenario: Manager adds a new item** 
**Given** the manager is on the Item management screen 
**When** the manager enters a new item SKU, description, UPC barcode, cost, and price
**And** clicks "Save"
**Then** the system creates a new item in the database.


### US-4.2 
**Scenario: Manager updates an item**
**Given** an item already exists in the system
**When** the manager changes the item's description
**And** clicks "Update" 
**Then** the system saves the new description of that item. 

### US-4.3 
**Scenario: Manager deletes an item** 
**Given** an unreferenced item exists in the system 
**When** the manager clicks "Delete" on the item's profile 
**Then** the system should remove the item completely from the database. 

## Data Ownership & Isolation 
 
| Rule | Requirement | 
|---|---|
| **Read scope** | All authenticated users (Managers and Workers) can view/lookup items. |
| **Write scope** | Only Managers can update or delete catalog items. | 
| **Create scope** | Only Managers can create new catalog items. | 
| **Cross-user-access** | Standard worker attempting POST/PATCH/DELETE -> `403 Forbidden`. |
| **UI scope** | Item Management CRUD controls are only visible to Managers. | 
| **Implementation**| Role-based access control (RBAC) enforced via shared helper in `app//authorization/`. |

## API Requirements 
| Method | Endpoint | Auth | Purpose |
|---|---|---|---|
| `GET` | `/api/items` | Yes (All) | Fetches catalog items (searchable by `sku`, `item_upc`, `case_upc`, or `description`) |
| `POST` | `/api/items` | Yes (Manager) | Creates a new item profile |
| `PATCH` | `/api/items/{item_id}` | Yes (Manager) | Updates item attributes, pricing, or reorder settings |
| `DELETE` | `api/items/{item_id}` | Yes (Manager) | Deletes an unreferenced item profile |

## Screen Requirements 

### [View: Item Management Dashboard] - route name `manage-items`
* Heading / purpose: Manage Item Catalog 
* Primary action: **Add New Item** (`oc-cta`)
* Secondary actions: **Edit**, **Delete** 
* Fields: SKU, Description, Item UPC, Case UPC, Case Cost, Selling Price, Required Zone Type, Preferred Supplier, Min Level, Reorder Qty
* **Empty state:** "No items in the catalog yet."
* Loading / error: `<v-alert type = "error">` for duplicate SKU/ UPC or deletion conflicts. 

## Test Coverage Map 
| Story | Scenario | Test file | Test name |
|---|---|---|---|
| US-4.1 | Manager adds a new item | `backend/tests/items.test.js` | `System creates a new item profile` |
| US-4.2 | Manager updates an item | `backend/tests/items.test.js` | `System updates existing item details` | 
| US-4.3 | Manager deletes an item | `backend/tests/items.test.js` | `System deletes unreferenced item profile` | 

## Agent Implementation Request 

```text 
Implement Feature 4 from @features/feature-4-items.md on branch `feature/4-items`.

Follow layer order in @features/framework.md(models -> routes -> backend tests -> frontend -> frontend tests).
Map every Gherkin scenario in the Test Coverage Map; run `npm test` before finishing.
```

## Definition of Done 
* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met 
* [ ] All mapped tests pass (`npm test`)
* [ ] Test Coverage Map complete 
* [ ] `features/reference/data-model.md` updated (Added Item table)
* [ ] `features/reference/api.md` updated (Added item CRUD endpoints)

## Out of Scope 
* Automated barcode label printing 
* Dynamic promotional pricing or bulk discount tiers. 


