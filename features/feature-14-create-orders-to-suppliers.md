# Feature 14: Create Orders to Suppliers 

## User Stories 

### US-14.1 : Create Supplier Purchase Order 
**As a** manager 
**I want to** create a new purchase order for a specific supplier 
**So that** I can order more items to restock the warehouse 

**Priority:** P1 
**Independent test** Verify the system assigns a unique order ID, calculates the total cost based on the items selected, and sets the initial status to a "Draft". 

### US-14.2 : Tracking Order Status 

**As a** manager
**I want to** track the delivery status and expected arrival dates of the purchase orders. 
**So that** I know exactly when new inventory will arrive 

**Priority:** P2 
**Independent test:** Verify the system allows the user to update the order status from 'Sent' to 'Delayed' or 'Fulfilled' and accurately shows the expected delivery date. 

## Functional Requirements
* The system must allow a manager to select a valid `supplier_id` from the existing Supplier database to begin an order
* The system must automatically calcuate the `total_cost` by multiplying the amount of each item ordered by their cost. S
* The system must enforce an order status lifecycle: `Draft` -> `Sent` -> `Confirmed` (by suplier)
* The system must record the employee ID of the manager who created the order and the time of creation 

## Assumptions 
* The supplier catalog (available items and current unit costs) is already filled in the system. 
* Managers have the financial authorization to use warehouse funds to purchase orders. 

## Edge Cases 
* **Zero Item Order:** A manager attempts to submit a purchase order with no items attatched. 
* **Cost Fluctuation:** The supplier changes the unit cost of an item while the order is still in the `Draft` state. 
* **Discontinued Item:** An item is marked as inactive or dicontinued before the draft order is sent

## Success Criteria 
* The system successfully generates a unique Purchase Order (PO) number. 
* The total cost calculates accurately based on the line items. 
* Status transitions are restricted to the defined lifecyle logic. 


## Key Entities 
* Manager / Worker 
* Supplier 
* SupplierOrder 
* SupplierOrderItem
* Item



## Initial Data Model 

### SupplierOrder

| Field Name | Data Type | Constriants | Description | 
|---|---|---|---|
| `order_id` | Integer | PK, Not Null | Unique identifier for the purchase order |
| `supplier_id` | Integer | FK,Not Null | References the specfic supplier being ordered from | 
| `created_by` | Integer | FK,Not Null | The employee ID of the manager placing the order |  
| `order_date` | DateTime | Not Null | The exact date the order was created | 
| `expected_delivery` | Date | Nullable | Estimated arrival date (populated after supplier confirmation) | 
| `total_cost` | Decimal | Not Null | The calculated total cost of the order | 
| `status` | String | Not Null | ENUM: Draft, Sent, Confirmed, Delayed, Cancelled | 

### SupplierOrderItem 

| Field Name | Data Type | Constriants | Description | 
|---|---|---|---|
| `order_item_id` | Integer | PK, Not Null | Unique ID for this specific line item | 
| `order_id` | Integer | FK, Not Null | Links back to the parent SuplierOrder | 
| `item_id` | Integer | FK, Not Null | The specific inventory being ordered | 
| `qty_requested` | Integer | Not Null, > 0 | Amount requested from the supplier | 
| `unit_cost` | Decimal | Not Null | Cost per unit at the time of ordering |

## Gherkin AC 

### US-14.1 
**Scenario: Manager drafts a new purchase order** 
 **Given** the maanger is logged into the ordering screen 
**When** they select a supplier, add items to the order, and click "Save" 
**Then** the system should generate a new  `SupplierOrder` with a unique ID 
**And** calculate the total cost 
**And** sets the initial status to `Draft`

### US-14.2 
**Scenario: Manager updates delayed shipment** 
**Given** an existing `SupplierOrder` is currently marked as `Sent` or `Confirmed`
**When** the manager changes the expected delivery date to a future date
**Then** the system should prompt the manager to update the status to `Delayed`
**And** save the new timeline to the database. 

## Data Ownership & Isolation 
| Rule | Requirement | 

|---|---|
|**Read scope**| Managers and authorized inventory clerks can view all the purchase orders. |
|**Write scope**| Only Managers can create, update, or transition order statuses. | 
|**Create scope**| New orders are automatically tied to the manager's `user_id`. | 
|**Cross-user access**| Floor workers attempting to view or edit supplier POs -> `403 Forbidden`.| 
|**Implementation**| Role-based access control is enforced on all supplier-orders routes. | 

## API Requirements 

| Method | Endpoint | Auth | Purpose | 
|---|---|---|---|
|`POST`| `/api/supplier-orders` | Yes(Manager)| Creates a new draft purchase order | 
|`POST`| `/api/supplier-orders/{order_id}/items` | Yes ( Manager) | Adds a line item to a draft order | 
|`PATCH` | `/api/supplier-orders/{order_id}/status` | Yes(Manager) | Updates the order lifecycle state accordingly | 

**Create request body:** 
```json
{
    "supplier_id": 402
}

{
    "order_id": 1055, 
    "status" : "Draft",
    "message": "Draft purchase order created."
}
```

## Screen Requirements 
### [View: Create Purchase Order] - route name `supplier-order-create`
* Heading / purpose: Draft New Supplier Order 
* Primary action: **Create Draft** (`oc-cta`)
* Fields: Supplier Dropdown Menu, Item Search/Select Auto-complete, quality input
* **Empty state:** "No items added to this order yet." 

## Definition of Done 

*[ ] Backend and frontend implemented per this spec 
*[ ] Success Criteria met 
*[ ] All mapped tests pass (`npm test`)
*[ ] `features/reference/data-model.md` updated 
*[ ] `features/reference/api.md`updated 
*[ ] `features/reference/behavior.md` updated 

## Out of Scope 
* Automated reordering
* The physical receiving of the goods at the loading dock.

## Test Coverage Map 
| Story | Scenario | Test file | Test name | 
|------|------|------|------|
|US-14.1 | Manager drafts a new purchase order | `backend/tests/supplier_orders.test.js` | `Manager creates draft PO` | 
| US-14.2 | Manager updates a delayed shipment | `backend/tests/supplier_order.test.js` | `Manager updates PO timeline and status` |

## Agent Implementation request 

```text 
Implement Feature 14 from @features/feature-14-create-orders-to-suppliers.md on branch `feature/14-supplier-orders`.

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests).
Map every Gherkin scenarion in the TEst Coverage Map; run `npm test` to verify the order lifecycle strict status transitions. 
```

