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

* The system must allow a worker to search for a 'SupplierOrder' by its 'order_id' when a truck arrives. 
* The system must let the worker set the actual amount of items unloaded from the truck 
* The system must update the purchase order status to 'Received' or 'Partially Received' depending on if all the items arrived. 
* The system must record which worker checked in the delivery and the date and time. 

## Key Entities 
* Worker / Manager 
* SupplierOrder (Purchase Order)
* ReceivingLog

## Initial Data Model 

### ReceivingLog 

| Field Name | Data Type | Description | 
|---|---|---|
| 'receipt_id' | Integer | Unique identifier for this specific delivery check-in | 
| 'order_id' | Integer | Foreign key referencing the original SupplierOrder |
| 'worker_id' | Integer | Foreign key referencing the worker who received the items |
| 'receive_date' | DateTime | The exact date and time the delivery was checked in | 
| 'items_received' | Array | A list of the specific item IDs and the amounts received | 
| 'has_missing_items' | Boolean | True if the received amount is less than the ordered amount |

'''## Gherkin AC 
    Feature: Receive Orders from Suppliers 

    ### US-15.1 
    Scenario: Worker receives a full delivery
    Given a worker is logged into the receiving screen
    When the worker enters the order ID and confirms all expected items have arrived 
    Then the system should create a new ReceivingLog and update the order status to 'Received' 
    
    ### US-15.2 
    Scenario: Supplier shorts a delivery
    Given a worker is checking in a delivery 
    When the worker enters a received amount that is lower than the expected amount 
    Then the system should flag 'has_missing_items' as True and alert the manager. 
'''