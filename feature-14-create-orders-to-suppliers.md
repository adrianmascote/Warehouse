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

* **Zero Item Order:**
## Key Entities 
* Manager / Worker 
* Supplier 
* 

## Initial Data Model 

| Field Name | Data Type | Description | 
|---|---|---|
| 'order_id' | Integer | Unique identifier for the purchase order | 
| 'supplier_id' | Integer | References the specfic supplier being ordered from | 
| 'created_by' | String | The employee ID or username of the worker placing the order | 
| 'order_date' | Date | The exact date the order was created | 
| 'expected_delivery' | Date | The estimated arrival date provided by the supplier | 
| 'items_ordered' | Array | A list containing the specific item ID's and quantities requested | 
| 'total_cost' | Decimal | The calculated total cost of the order | 
| 'status' | String | The current state of the order (e.g., Draft, Sent, Delayed, Fulfilled) |


'''## Gherkin AC 

    ### US-14.1 
    **Scenario: Manager drafts a new purchase order** 
    **Given** the maanger is logged into the ordering screen 
    **When** they select a supplier, add items to the order, and click "Save" 
    **Then** the system should generate a new purchase order with a unique ID and calculate the total cost. 

    ### US-14.2 
    **Scenario: Manager updates delayed shipment** 
    **Given** an existing purchase order is currently marked as 'Sent' 
    **When** the manager changes the expected delivery date to a future date
    **Then** the system should prompt the manager to update the status to "Delayed: abd saves the new timeline. 
'''



