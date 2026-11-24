# Feature 9: Out-of-Stock & Low Inventory Reporting 

## User Stories 

### US-9.1: View Out-of-Stock Items 
**As a** warehouse manager 
**I want to** generate a report of all items with an inventory count of zero. 
**So that** I know exactly which products we can no longer fulfill for customer orders. 

**Priority:** P1
**Independent test:** Verify the system analyzes the inventory database and returns all items that are out of stock. 
**Acceptance scenarios:** see ### US-9.1 under Gherkin AC 

### US-9.2: View Low Stock Items 
**As a** warehouse manager 
**I want to** view a report of items that have fallen bellow their minimum stock levels. 
**So that** I can reorder products before they completely run out. 

**Priority:** P1 
**Independent test:** Ensure the system identifies the items where the current inventory is below the 'min_level' threshold. 
**Acceptance scenarios:** see ### US-9.2 under Gherkin AC

### US-9.3: Export Inventory Report 
**As a** warehouse manager 
**I want to** export the out-of-stock and low-stock reports 
**So that** I can send the data to the finance department to create supplier orders. 

**Priority:** P2 
**Independent test:** Verify the manager can download the generated report as PDF file. 
**Acceptance scenarios:** see ### US-9.3 under Gherkin AC 

## Functional Requirements 
* The system must allow managers to filter inventory records where 'current_inv_qty' is 0. 
* The system must allow managers to filter inventory records where 'current_inv_qty' is less than or equal to the 'min_level'
* The report must display the Item SKU, Description, current quantity, minimum level, and bin location. 
* The system must provide an export function for the generated reports.  

## Assumptions 

* Every item in the database has a predefined `min_level` established during initial setup. 
* The inventory counts are accurately maintained in real-time by the inbound receiving and outbound shipping features. 

## Edge Cases 
* **Null Minimums:** Items that have a `min_level` of `0` or `null` (such as discontinued products) must be excluded from the Low Stock report to prevent false alarms. 
* **Negative Inventory:** If a system error causes `current_inv_qty` to drop below zero, the system must still flag it in the Out-of-Stock report.

## Success Criteria 
* The report accurately joins data across the Inventory, Item, and StorageZone tables. 
* The UI properly distinguishes between "Out of Stock" and "Low Stock"

## Key Entities 
* Manager 
* Inventory
* Item 
* StorageZone


## Initial Data Model 

### InventoryRecord (Referenced)

| Field Name | Data Type | Constraints | Description | 
|---|---|---|---|
| `inv_id` | Integer | PK, Not Null | Primary key for the inventory record | 
| `item_id` | Integer | PK, Not Null | Foreign key referencing the specific item | 
| `zone_id` | Integer | FK, Not Null | Foreign key referencing the specific StorageZone | 
| `current_inv_qty` | Integer | Not Null | The current quantity of the item in stock |
| `min_level` | Integer | The minimum required stock level before reordering | 


### Item (Referenced)

| Field Name | Data Type | Constraints | Description | 
|---|---|---|---|
| `item_id` | Integer | PK, Not Null | Primary key for the item | 
| `sku` | String | Not Null |  The SKU identifier for the item | 
| `description` | String | Not Null | The description of the item |

### StoragerZone (Referenced)

| Field Name | Data Type | Constraints | Description | 
|---|---|---|---|
| `zone_id` | Integer | PK, Not Null | Primary key for the storage zone | 
| `bin_location` | String | Not Null | The physical aisle/bin location string | 


## Data Ownership & Isolation 
| Rule | Requirement | 
|---|---|
| **Read scope** | Managers can view inventory added data across all zones and items.  | 
| **Write scope** | None (Reporting features are Read-Only). |
| **Create scope** | None. | 
| **Cross-user-access** | Role-based access control (RBAC) enforced via shared helper `app/authorization/`. |

## API Requirements 

| Method | Endpoint | Auth | Purpose | 
|---|---|---|---|
| `GET` | `/api/reports/inventory/out-of-stock` | Yes (Manager) | Fetches items where quantity <= min_level |
|`GET` | `/api/reports/inventory/export` | Yes (Manager) | Returns the queried report data formatted as a downloadable CSV | 

## Screen Requirements 

### [View: Inventory Shortage Dashboard] - route name `inventory-reports` 
* Heading / purpose: Inventory Depletion Reports
* Primary actions: **View Out of Stock** (`oc-cta`), **View Low Stock** (`oc-cta`)
* Secondary action: **Export as Spreadsheet** 
* Fields/Displays: Data table featuring columns for SKU, Description, Current Qty, Min Level, and Bin Location. 
* **Empty state:** "All items are currently adequately shocked." 
* Loading: Data table skeleton loader. 

## Test Coverage Map 

| Story | Scenario | Test file | Test name | 
|---|---|---|---|
| US-9.1 | Manager runs the Out-of-Stock report | `backend/tests/inventory-report.test.js` | `System returns items at or below minimum threshold` | 
| US-9.2 | Manager runs the Low Stock report | `backend/tests/inventory-report.test.js` | `System returns items at or below minimum threshold` | 
| US-9.3 | Manager exports the inventory report | `backend/tests/inventory-report.test.js` | `System formats active report data as CSV download` | 

## Agent Implementation request 

```text 
Implement Feature 9 from @features/feature-9-low-inventory.md on branch `feature/9-low-inventory`.

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests).
Map every Gherkin scenario in the Test Coverage Map; run `npm test` before finishing.
Ensure the CSV export endpoint dynamically handles whichever report (OOS or Low Stock) the user is currently viewing.
```

## Definition of Done 

* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met 
* [ ] All mapped tests pass (`npm test`)
* [ ] Test Coverage Map complete 
* [ ] `features/reference/api.md` updated (if API changed)

## Out of Scope 
* Forecasting future low stock events based on sales patterns. 


## Gherkin AC 

### US-9.1 
**Scenario:** Manager runs the Out-of-Stock report 
**Given** the manager is logged into the reporting screen
**When** the manager clicks "Generate Out-of-Stock Report" 
**Then** the system should display a list of all items with an inventory count of 0.

### US-9.2 
**Scenario:** Manager runs the Low Stock report 
**Given** the manager is on the reporting screen 
**When** the manager clicks "Generate Low Stock Report" 
**Then** the system should display all item where the inventory count is at or bellow the minimum threshold. 

### US-9.3 
**Scenario:** Manager exports the inventory report 
**Given** an inventory report is displayed on the screen 
**When** the manager clicks "Export as Spreadsheet" 
**Then** the system should download a CSV file containing the data of the active report. 


