# Feature 16: Put Away Orders from Suppliers 

## User Stories 

### US-16.1: Store Received Items 
**As a** worker
**I want to** know exactly which storage zone to put the new received items in 
**So that** the warehouse stays organized and items are easy to find

**Priority:** P1
**Independent test:** Verify the system assigns a valid storage zone for the received items and updates the item's location status 

### US-16.2: Update Available Inventory 
**As a** manager 
**I want to** ensure inventory amounts update automatically when items are put away. 
**So that** I have an accurate count of what is available to sell to customers. 

**Priority:** P1 
**Independent test**Verify the system increases the available inventory for the specific item put away 

## Functional Requirements 
* The system must allow a worker to scan or select a received item and suggest an appropriate 'zone_id' for storage
* The system must record the 'worker_id' of the person who moved the items 
* The system must automatically update the overall inventory count once al the items are stored. 
* The system must track the exact date and time the items were put on the shelves 

## Key Entities 
* Worker 
* Item 
* StorageZone 
* InventoryMovement 

## Initial Data Model 

### InventoryMovement 

| Field Name | Data Type | Description | 
|---|---|---|
| 'movement_id' | Integer | Unique identifier for the specific put away task | 
| 'item_id' | Integer | foreign key referencing the specific item being moved | 
| 'zone_id' | Integer | Foreign key referencing the storage zone it was | 
| 'worker_id' | Integer | Foreign key referencing the worker who stored it | 
| 'quantity' | Integer | The amount of the item that was put away | 
| 'movement_date' | DateTime | the exact date and time the items were put on the shelves | 

'''## Gherkin AC 
    ### US-16.1 
    Scenario: Worker Stores items in a specific zone
    Given: a worker has a cart of newly receivied items 
    When: They scan the item and confirm they placed it in the suggested storage zone
    Then: The system should log the new location and mark the put-away task as complete 
    ### US-16.2 
    Scenario: System updates available stock
    Given a worker has just completed a put-away task for 50 boxes 
    When the task is saved to the database 
    Then the system should automatically increase the total available inventory for that item by 50. 
'''