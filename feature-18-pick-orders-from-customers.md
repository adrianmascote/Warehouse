# Feature 18: Pick Orders from Customers 

## User Stories 

### US-18.1: View Optimized Pick List 
**As a** worker 
**I want to** see a list of items in the most efficient walking order 
**So that** I don't waste time walking back and forth across the warehouse 

**Priority:** P1 
**Indpendent test:** Verify the system sorts the items on the pick list based on the storage zones to make the most efficient walking route. 

### US-18.2: Update Order to Picked 
**As a** manager 
**I want to** know when a worker has finished collecting all items for an order 
**So that** I know it is ready to be packed and shipped

**Priority:** P1 
**Indpendent test:** Verify the system updates the order status to 'Picked' once the worker confirms that all items are collected.

## Functional REquirements 

* The system must make a digital pick list that links the ordered items to their corresponding 'zone_id' 
* The system must automatically sort the digital pick list to make the most efficient walking route through the warehouse locations. 
* The system must allow a worker to claim an order so that two workers do not pick the same order. 
* The system must allow the worker to check off or scan items with their device as they pick the items
* The system must automaticall change the order 'status' to 'Picked' when all items are checked off. 
* The system must record the 'worker_id' of the person who picked the order. 

## Key Entities 
* Worker / Manager 
* CustomerOrder 
* Item 
* StorageZone 
* PickTask 

## Initial Data Model 

### PickTask 

| Field Name | Data Type | Description | 
|---|---|---|
| 'pick_id' | Integer | Unique identifier for this specific picking assignment | 
| 'order_id' | Integer | Foreign key referencing the customer order being picked | 
| 'worker_id' | Integer | Foreign key referencing the worker gathering the items | 
| 'pick_status' | String | The current state of the task (e.g., Not started, In Progress, Completed) | 
| 'start_time' | DateTime | The exact date and time the worker started picking |
| 'end_time' | DateTime | The exact date and time the worker finished picking | 

'''## Gherkin AC 
    ### US-18.1 
    Scenario: Scanner directs worker to the next closest item 
    Given: A worker has just scanned and collected an item for a PickTask 
    When: they look at their scren for the next step 
    Then: the system should display the exact storage zone and amount needed for the next closest item on the optimized route 

    ### US-18.2 
    Scenario: Worker finishes gathering the order 
    Given: a worker has checked off every item on the pick list 
    When: They tap "Complete Picking" on their scanner 
    Then: the system should log the end time and update the CustomerOrder status to 'Picked'
'''
