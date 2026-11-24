# Feature 10: Supplier Issues & Discrepacy Reporting 

## User Stories 

### US 10-1: Identify short-shipped orders 
**As a** warehouse manager 
**I want to** view a report of supplier orders where the received quantity does not match the ordered quantity. 
**So that** I know which suppliers are failing to deliver what we paid for. 

**Priority:** P1 
**Independent test:** Verify the system can compare the original ordered item quantity to the quantity received. 
**Acceptance scenarios:** see ### US-10.1 under Gherkin AC 

### US-10.2: View Specific Missing Items 
**As a** warehouse manager 
**I want to** see exactly which SKUs and how many cases were missing from a flagged delivery 
**So that** I can contact the supplier to request a refund or an additional shipment. 

**Priority:** P1 
**Independent test:** Verify the discrepancy report contains the Item's SKU, description, amount ordered, and the amount received. 
**Acceptance scenarios:** see ### US-10.2 under Gherkin AC 

## Functional Requirements 
* The system must compare the original 'cases' ordered on a purchase order with the amount actually scanned in by the receiving worker. 
* The system must generate a report that flags any purchase orders that have a status of "Received" but have a mismatch in the item amounts. 
* The report must display the Supplier Name, PO Number, Item SKU, Ordered quantity, and received quantity. 

## Assumptions 
* The receiving flow (Feature 15) successfuly record the exact `cases_received` before marking an order as "Received."
* Supplier and Item records are consistently maintained and linked in the database. 

## Edge Cases 
* **Over-shipments:** A supplier accidentally sends more cases than were ordered (mismatch where received > ordered).
* **Open Orders:** Orders that are still in "Pending" or "In Transit" status must be exluded from this report since they haven't been fully received yet. 

## Success Criteria 
* The report accurately joins data across the Supplier, Order, and Item tables. 
* Managers can easily identify the exact (difference) between ordered and received amounts
* Standard workers cannot access the report






## Key Entities 
* Manager 
* Supplier 
* Supplier Order 
* Item 

## Initial Data Model 

### SupplierOrder (Referenced)

| Field Name | Data Type | Constraints | Description | 
|---|---|---|---|
| `order_id` | Integer | PK, Not Null | Primary key for the supplier order |
| `po_number` | String | The purchase order number | 
| `supplier_id` | Integer | FK, Not Null | Foreign key referencing the supplier | 
| `status` | String | Not Null | The order status | 

### SupplierOrderItem (Referenced)

| Field Name | Data Type | Constraints | Description | 
|---|---|---|---|
| `order_id` | Integer | FK, Not Null | Foreign key referencing the SupplieirOrder |
| `item_id` | Integer | FK, Not Null | Foreign key referencing the Item |  
| `cases` | Integer | Not Null | The number of cases ordered | 
| `cases_received` | Integer | Not Null | The number of cases received at the dock |

### Supplier (Referenced)

| Field Name | Data Type | Description | 
|---|---|---|---|
| `supplier_id` | Integer | PK, Not Null | Primary key for the supplier
| `name` | String | Not Null | The name of the supplier | 

### Item (Referenced)

| Field Name | Data Type | Description | 
|---|---|---|---|
| `item_id` | Integer | PK, Not Null | Primary key for the Item | 
| `description` | String | Not Null | The description of the item |

## Gherkin AC 

### US-10.1 
**Scenario: Manager views orders with missing items** 
**Given** the manager is on the reporting screen
**When** the manager clicks "Generate Supplier Discrepancy Report" 
**Then** the system should display a list of all recent Purchase Orders where the received case count was lower than the ordered case count

### US-10.2 
**Scenario: Manager investigates a specific supplier issue** 
**Given** the supplier Discrepancy Report is generated 
**When** the manager clicks on a specific flagged PO number 
**Then** the system should display the Item SKUs that were missing and the difference in case amounts. 


## Data Ownership & Isolation 

| Rule | Requirement | 
|---|---|
| **Read scope** | Managers can view discrepancy aggregated data across all suppliers and orders | 
| **Write scope** | None (Reporting features are Read-Only). | 
| **Create scope** | None. | 
| **Cross-user-access** | Standard worker attemtping to view -> `403 Access Denied`. | 
| **UI scope** | Report menu is visible to authenticated Manager roles. | 
| **Implementation** | Role-based access control (RBAC) enforced through helper in `app/authorization/`.| 

## API Requirements 
| Method | Endpoint | Auth | Purpose | 
|---|---|---|---|
| `GET` | `/api/reports/supplier-discrepancies` | Yes (Manager) | Fetches the joined discrepancy data for flagged POs | 

## Screen Requirements 

### [View: Supplier Issues Dashboard] - route name `supplier-reports` 
* Heading / purpose: Supplier Discrepancy Report 
* Primary action: **Generate Report** (`oc-cta`)
* Fields/ Displays: Data table featuring columbs for Supplier Name, PO Number , SKU, Ordered Qty, Received Qty, and Difference. 
* **Empty state:** "All received orders match expected quantities."
* Loading: Data table loader. 

## Test Coverage Map 

| Story | Scenario | Test file | Test name | 
|---|---|---|---|
| US-10.1 | Manager views orders with missing items | `backend/tests/reporting.test.js` | `Report accurately joins item SKU's and calculates the difference in items` | 
| US-10.2 | Manager investigates a specific supplier issue | `backend/tests/reporting.test.js` | `Report accurately joins item SKUs and calculates difference`| 

## Agent Implementation Request 

```text 
Implement Feature 10 from @features/feature-10-supplier-issues.md on branch `feature/10-supplier-issues`. 

Follow Layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests). Map every Gherkin scenario in the Test Coverage Map; run `npm test` before finishing. 
Ensure the SQL database only queries SupplierOrders with a "Received' status. 
```

## Definition of Done 
* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met 
* [ ] All mapped tests pass (`npm test`) 
* [ ] Test Coverage Map complete 
* [ ] `features/references/api.md` updated (if API changed)

## Out of Scope 
* Financial Automatic chargebacks. 
* Automatically emailing the supplier about the discrepancy. 