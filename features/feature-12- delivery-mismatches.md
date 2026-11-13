# Feature 12: Delivery Mismatches

## User Stories 

### US-12.1: Log Delivery Discrepancy 
**As a** delivery driver
**I want to** log if the physical items delivered do not match the digital record 
**So that** I can see if the warehouse failed to complete the order accurately. 

**Priority:** P1
**Indpendent test:** verify that a driver can submit a discrepancy reason and the system updates the item's status to `Diputed`.  
**So that** I can investgate goods that may have been dmaged or lost on the delivery route. 

### US-12.2: Resolve Disputed Deliveries 

**As a** Manager 
**I want to** review logged delivery discrepancies 
**So that** I can approve inventory adjustments and finalize the store's order status

**Priority:** P2
**Indpendent test:** Verify the manager can update a discrepancy to `Resolved`, which automatically triggers an adjustment in inventory
**Acceptance scenarios:** see ### US-12.2 under Gherkin AC 

## Functional Requirements
* The system must allow drivers to flag individual items within a `StoreOrder` as Missing, Damaged, or Overage. 
* The system must require the driver to input the actual `qty_delivered` if it differs from the delivery receipt. 
* The system must automatically change the order status to `Disputed` if any discrepancy is logged
* The system must provides managers a dashboard to review, approve, and reject discrepancy claims. 

## Assumptions 
* Drivers have network connection at the Delivery site 
* Stores physically inspect the delivery before the driver leaves. 

## Edge Cases 

* **Complete Rejection:** The store refuses the entire order due to severe damage.
* **Driver Input Error:** The driver accidentally logs a discrepancy on the wront item, 
* **Overage:** The driver delivers 5 cases but the digital receipt only listed 4

## Success Criteria 
* Discrepancies are successfully recorded with a timestamp and reason code. 
* The system prevents an order from closing if unresolved discrepancies exist. 
* Manager resolution automatically updates the inventory counts. 



## Key Entities 
* Driver / Manager 
* StoreOrder
* OrderItem
* DiscrepancyLog

## Initial Data Model 

### DiscrepancyLog 


| Field Name | Data Type | Constraints | Description | 
|---|---|---|---|
| `log_id` | Integer | PK, Not Null | Unique identifier for the mismatch record |
| `order_id` | Integer | FK, Not Null | Reference to the disputed StoreOrder | 
| `item_id` | Integer | FK, Not Null | Reference to the specific item | 
| `driver_id` | Integer | FK, Not Null | The driver who reported the issue | 
| `qty_expected` | Integer | Not Null | What the digital records say |
| `qty_delivered` | Integer | Not Null | What was physically handed over | 
| `reason_code` | String | Not Null | What was physically handed over | 
| `reason_code` | String | Not Null | ENUM: Missing, Damaged, Overage | 
| `status` | String | Not Null | ENUM: Pending, Resolved, Rejected | 

## Gherkin AC 

### US-12.1 
**Scenario: Driver reports damaged goods** 
**Given** Given the driver is completing an active delivery. 
**When** the driver inputs that 2 cases of an item are damaged 
**Then** the system creates a `DiscrepancyLog` with the status `Pending`
**And** the overarching `StoreOrder` status changes to `Disputed`

### US-12.2 
**Scenario: Manager approves the discrepancy** 
**Given** a discrepancy is in the `Pending` state 
**When** the manager clicks "Approve Adjustment" 
**Then** the system updates the log status to `Resolved`
**And** updates the master inventory count to reflect the damaged goods. 

## Data Ownership & Isolation 


| Rule | Requirement | 
|---|---|
|**Read scope**| Drivers only see discrepancies where `driver_id = req.user.id`. Managers see all.|
| **Write scope** | Drivers can only create. Only Managers can update `status` to `Resolved`. | 
| **Create Scope** | New logs auatomatically tied to the authenticated driver's ID. | 
| **Cross-user access** | Driver accessing another driver's log -> `404`. Driver attempting to resolve -> `403`.| 
| **UI scope** | Manager dashboard shows all `Pending` logs. Driver screen shows their active dlivery. | 
| **Implementation** | Role-based access control (RBAC) enforced at the route level.| 

## API Requirements 

| Method | Endpoint | Auth | Purpose | 
|---|---|---|---|
|`POST`| `/api/orders/{order_id}/discrepancies` | Yes (Driver) | Submits a new discrepancy |
|`GET`| `/api/discrepancies/pending`| Yes (Manager) | Fetches all unresolved logs |
|`PATCH` | `/api/discrepancies/{log_id}/resolve` | Yes (Manager) | Updates status and inventory | 


**Create request body:** 
```json 
{
    "item_id":88,
    "qty_expected":5,
    "qty_delivered":3, 
    "reason_code":"Missing"
}
```
## Screen Requirements 

### [View: Log Discrepancy Dialog] - route name `delivery-mismatch`
* Heading/purpose: Report Delivery Issue 
* Primary action: **Submit Report** (`oc_cta`)
* Fields: Reason Dropdown (Missing, Damaged, Overage), Actual Qty Input. 
* Loading/error: `<v-alert type="error">` for missing reason codes. 



## Test Coverage Map 

| Story | Scenario | Test file | Test name | 
|-------|----------|-----------|-----------|
| US-12.1 | Driver reports damaged goods | `backend/tests/discrepancy.test.js` | `Driver submits new mismatch log` | 
| US-12.2 | manager approves the discrepancy | `backend/tests/discrepancy.test.js` | `Manager resolves pending mismatch` |


## Agent Implementation request 

```text
Implement Featuere 12 from @feature-12-delivery-mismatches.md on branch `feature/12-delivery-mismatches`.

Follow Layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests). Map every Gherkin scenario in the Test Coverage Map; run `npm test` to verify the resolved state updates tbe master inventory.
```

## Definition of Done 

* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met 
* [] All mapped tests pass (`npm test`)
* [] `features/reference/data-model.md` updated (if schema changed)
* [] `features/reference/api.md` updated (if API changed)
* [] `features/ reference/behavior.md` updated (if product rules changed)

## Out of Scope 
* Automatic financial invoicing or refunds to the store (handled by accounting).
* Physical return shipping logistics. 
