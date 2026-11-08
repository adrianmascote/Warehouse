# Feature 17: Receive Orders from Customers 

## User Stories 

### US-17.1: Log New Customer Order 
**As a** manager 
**I want to** enter and save a new order requested by a customer 
**So that** the warehouse team kows exactly what needs to be packed and shipped 

**Priority:** P1 
**Indpendent test:** Verify the system generates a unique order ID, links it to the correct customer, and sets the initial status to `New`. 

### US-17.2: Verify Stock Availability 
**As a** manager 
**i want to** be warned if a customer orders an item we do not have eough of 
**So that** we do not promise inventory we cannot actually deliver. 

**Priority:** P1
**Indpendent test:** Verify the system checks the requested amount agaisnt the available inventory and flags the line item if the requested amount exceeds the inventory on hand. 

## Functional Requirements 

* The system must allow the user to select an existing `customer_id` to attach to the order. 
* The system must allow the manager to add specific items and amounts to the order 
* The system must dynamically calculate the `total_price` by multiplying the requested quantity of each item by its current selling price. 
* The system must check current stock levels (`qty_on_hand`) for each requested item. If the requested amount exceeds the available stock, the system must prompt the user to either reduce the quantity or flag the item as backordered. 
* The system must enforce a strict order status lifecycle : `New` -> `Backordered` (if waiting on stock) -> `Picking` -> `Shipped` -> `Cancelled`. 

## Assumptions 

* Customers and their billing / shipping profiles already exist in the database. 
* The `qty_on_hand` in the master inventory table is accurate and updated in real-time. 

## Edge Cases 

* **Concurrent Ordering:** Two different managers try to sell the final 5 remaining units of an item to two different customers at the exact same time. 
* **Partial Backorder:** A customer orders 50 units, but only 20 are in stock. 
* **Zero Dollar Order:** A customer uses a 100% discount or orders a promototional item that costs $0.00.

## Success Criteria 

* The database successfully stores the parent order and normalized child line terms 
* Out-of-stock items trigger a UI warning before the order is permanently saved. 
* The total price accurately reflects all line items combined. 

## Key Entities 
* Manager
* Customer 
* CustomerOrder
* CustomerOrderItem
* Item 

## Initial Data Model 

### CustomerOrder

| Field Name | Data Type | Constraints | Description | 
|----|----|----|----|
|`order_id` | Integer | PK, Not Null | Unique identifier for the customer's order | 
|`customer_id` | Integer | FK, Not Null | References the specific customer | 
| `order_date` | DateTime | Not Null | The exact date and time the order was placed | 
| `total_price` | Decimal | Not Null | The calculated total cost the customer will pay | 
| `status` | String | Not Null | ENUM: New, Backordered, Picking, Shipped, Cancelled | 

### CustomerOrderItem 

| Field Name | Data Type | Constraints | Description | 
|---|---|---|---|
| `order_item_id` | Integer | PK, Not Null | Unique ID for the specific line item | 
| `order_id` | Integer | FK, Not Null | Links back to the parent CustomerOrder | 
| `item_id` | Integer | FK, Not Null | The specific inventory item being ordered | 
| `qty_requested` | Integer | Not Null, >0 | Amount the customer wants to buy | 
| `unit_price` | Decimal | Not Null | Selling price per unit at the time of the order | 
| `is_backordered` | Boolean | Not Null | `True` if the item requires waiting for the supplier to restock | 


 Gherkin AC 

### US-17.1
**Scenario: Manager creates a new customer order**
**Given** the manager is logged into the order entry screen
**When** they select a customer, add items to the cart, and click "Submit Order" 
**Then** the system should create a new `CustomerOrder`
**And** calculate the total price 
**And** set the overarching status 


## US-17.2 
**Scenario: Customer orders out-of-stock items**
**Given** a manager is creating a new customer order 
**When** they enter a quantity that is higher than the current available inventory 
**Then** the system should display an `Out of Stock` warning 
**And** require the manager to explicitly flag the item for backorder before saving. 

## Data Ownership & Isolation 
 | Rule | Requirement | 
 |---|---|
 | **Read scope** | Managers and Pickers can view active customer orders. |
 | **Write Scope** | Only Managers and Sales staff can create or modify customer orders. | 
 | **Implementation** | Role-based access control (RBAC) enforced on `POST/api/customer-orders`.| 

 ## API Requirements 
 | Method | Endpoint | Auth | Purpose | 
 |---|---|---|---|
 | `POST` | `/api/customer-orders/validate-stock` | Yes | Checks cart quantities against the DB before submitting | 
 | `POST` | `/api/customer-orders` | Yes | Submits the final order and line items to the database | 
 

**Validate request body:**
```json
{
    "items": [
        {"item_id" :12, "qty_requested" : 50}
    ]
    
}
 ```
 **Stock Warning**
 ```json
 {
    "valid": false, 
    "warnings" : [
        { "item_id" : 12, "qty_on_hand": 20, "message" : "Insufficient stock." }
    ]
 }
 ```

## Screen Requirements 

### [View: Customer Order Entry] - route name `customer-order-create`
 * Heading / purpose: Create Customer Order 
 * Primary action: **Submit Order** (`oc-cta`) 
 * Fields: Customer Dropdown Menu, Item Search/Select Auto-complete, Quantity Input. 
 * **Visual indicator** A yellow `<v-alert type="warning">` inline with any item row that exceeds the available stock, containing a checkbox to "Approve Backorder" 

## Definition of Done 
* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met 
* [ ] All mapped tests pass (`npm test`) 
* [ ] `features/reference/data-model.md` updated 
* [ ] `features/reference/api.md` updated
* [ ] `features/reference/behavior.md` updated 

## Out of Scope 

* Physical processing, printing packing slips, and boxing the items (this is handled in Feature 18). 
* Processing credit card payments or generating financial invoices. 

## Test Coverage Map 

| Story | Scenario | Test file | Test name | 
|---|---|---|---|
| US-17.1 | Manager creates a new customer order | `backend/tests/customer_orders.test.js` | `System generates new customer order and calculates total` | 
| US-17.2 | Customer orders out-of-stock items | `backend/tests/customer_orders.test.js` | `System flags shortage and requires backorder approval` | 

```text 
Implement Feature 17 from @features/feature-17-receive-orders-from-customers.md on branch `feature17-customer-orders`.

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests).
Map every Gherkin scenario in the Test Converage Map; run `npm test` to verify the `/validate-stock`  successfully rejects orders that exceed `qty_on_hand` without the backorder tags. 
```