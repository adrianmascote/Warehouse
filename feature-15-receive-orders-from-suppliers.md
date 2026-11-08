# Feature 15: Receive Orders from Suppliers 

## User Stories 

### US-15.1: Check In Delivery 
**As a** worker 
**i want to** log the items that arrive from a supplier 
**So that** the system knows the delivery has physically entered the building 

**Priority:** P1 
**Independent test:** Verify the system allows a worker to look up an active purchase order and mark it as delivered. 

### US-15.2: Flag Missing Items 
**As a** manager 
**I want to** be alerted if a supplier delivers fewer items than we ordered 
**So that** we don't pay for inventory we never received. 

**Priority:** P2
**Independent test:** Verify the system compares the received quantitiy with the expected quantity and flags the order if the received amount was less than the expected amount. 

## Functional Requirements 

* The system must allow a worker to search for an active `SupplierOrder` (status `Sent`, `Confirmed`, or `Delayed`) by its `order_id` when a truck arrives. 
* The system must require the worker to input the actual`qty_received` for each item on the purchase order 
* The system must automaticaly evalutate the amounts. If all items match the order, status updates to `Received` if any item is short, status updates to `Partially Received`. 
* The system must record the `worker_id` of the employee checking in the delivery and the time of the check-in. 

## Assumptions 

* The `SupplierOrder` was successfully created and exists in the daatabse from Feature 14. 
* The worker checking in the items has physical access to a scanner or terminal when on the receiving dock. 

## Edge Cases 
* **Over-delivery:** The supplier sends *more* items than were originally requested on the purchase order. 
* **Unexpected Items:** The supplier includes items on the truck that were never listed on the PO. 
* **Total No-Show:** The delivery truck arrives, but a specific item on the PO is missing (qty = 0).

## Success Criteria 
* A `ReceivingLog` is saved to the database 
* The `SupplierOrder` status transitions as expected based on the business rules for partial receipts. 
* The original request amounts are never overwritten. Actual received quanitities are stored separately for comparison. 

## Key Entities 
* Worker / Manager 
* SupplierOrder (Purchase Order)
* ReceivingLog
* ReceivingLogItem


## Initial Data Model 

### ReceivingLog 

| Field Name | Data Type | Constraints | Description | 
|---|---|---|---|
| `receipt_id` | Integer | PK, Not Null | Unique identifier for this specific delivery check-in | 
| `order_id` | Integer | FK, Not Null | Foreign key referencing the original SupplierOrder |
| `worker_id` | Integer | FK, Not Null | References the worker who received the items |
| `receive_date` | DateTime | Not Null | The exact date and time the delivery was checked in |  
| `has_missing_items` | Boolean | Not Null | True if the received amount is less than the ordered amount |

## ReceivingLogItem

| Field Name | Data Type | Constraints | Description | 
|---|---|---|---|
| `receipt_item_id` | Integer | PK, Not Null | Unique ID for this specific received line item | 
| `receipt_id` | Integer | FK, Not Null | Links back to the parent ReceivingLog | 
| `item_id` | Integer | FK, Not Null | The specific inventory item being received | 
| `qty_expected` | Integer | Not Null | Amount that was supposed to arrive (from PO) | 
|`qty_received` | Integer | Not Null | Actual physical amount unloaded from the truck | 


## Gherkin AC 

 ### US-15.1 
**Scenario: Worker receives a full delivery** 
**Given** a worker is logged into the receiving screen 
**When** the worker enters the order ID and confirms all expected item amounts match the physical deliveyr 
**Then** the system should create a new `ReceivingLog`
**And** update the `SupplierOrder` status to `Received`. 

### US-15.2 
**Scenario: Supplier shorts a delivery** 
**Given** a worker is checking in an active delivery 
**When** the worker enters a `qty_received` that is lower than the `qty_expected`
**Then** the system should flag `has_missing_items` as True 
**And** automatically change the `SupplierOrder` status to `Partially Received`. 

## Data Ownership & Isolation 

| Rule | Requirement | 
|---|---|
|**Read scope**| Any warehouse worker can look up active purchase orders to receive them. | 
| **Write scope** | Any authorized floor worker can create a `ReceivingLog`.| 
|**Update scope** | Role-based access control (RBAC) 

## API Requirements 

| Method | Endpoiont | Auth | Purpose | 
|---|---|---|---|
|  `GET`| `/api/supplier-orders/{order_id}`| Yes | Fetches the expected items for the worker to count against | 
| `POST` | `/api/receiving-logs` | Yes | Submits the actual verified amounts | 

**Create request body:** 
```json 
{
    "order_id": 1055, 
    "items": [
        {"item_id": 44, "qty_received": 10}
    ]
}
```

**Success Response** 
```json
{
    "receipt_id": 892,
    "status_updated_to": "Partially Received", 
    "missing_items" : true

}
```

## Screen Requirements 

### [View: Receiving Dock Check-In] - route name `receiving-check-in`
* Heading / purpose: Receive Supplier Delivery 
* Primary action: **Log Delivery** `oc-cta`
* Fields: PO Search Bar, Expected vs. Actual Quantity Data Grid 
* **Visual indicator** A red `<v-badge>` or warning icon next to any row where any actual quantity does not match the expected quantity. 

## Definition of Done 

* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met 
* [ ] All mapped tests pass (`npm test`)
* [ ] `features/reference/data-model.md` updated 
* [ ] `features/reference/api.md` updated 
* [ ] `features/reference/behavior.md` updated 

## Out of Scope 

* Physical put-away of the received items onto the warehouse shelves 
* Authorizing payments to the supplier (handled by accounting). 

## Test Coverage Map 
| Story | Scenario | Test file | Test name | 
|----|----|----|----|
|US-15.1 | Worker receives a full delivery | `backend/tests/receiving.test.js` |`System updates PO to Received on full match` | 
| US-15.2 | Supplier shorts a delivery | `backend/tests/receiving.test.js` | `System updates PO to Partially Received on shortage` | 

## Agent Implementation request 

```text 
Implement Feature 15 from @features/feature-15-receive-orders-from-suppliers.md on branch `feature/15-receive-orders`. 

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend tests).
Map every Gherkin scenario in the Test Coverage Map; run `npm test` to verify the logic correctly transitions the PO status to `Partially Received` when a shortage occurs. 

```