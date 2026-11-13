# Feature: Inventory Data Management 

## User Stories 

### US-1.1: Add New Inventory 
**As a** warehouse worker 
**I want to** add new inventory records 
**so that** the system tracks newly arrived products and we know exactly what is in the building 

**Priority:** P1 
**Independent test:** Verify that a worker submit new inventory and that it appears in the database view. 
**Acceptance scenarios** See ### US-1.1 under Gherkin AC 

### US-1.2: Update Inventory Amounts (Manually)
**As a** warehouse worker 
**I want to** update existing inventory quantities and locations 
**so that** the system accurately reflects when itme hsave been moved or picked for an order 

**Priority** P1 
**Independent test:** Verify a worker can change the quantity of an existing item and the system saves the new amount. 
**Acceptance scenarios:** see ### US-1.2 under Gherkin AC 

### US-1.3: Delete Inventory Records 
**As a** warehouse manager
**I want to** delete inventory records for items we no longer have 
**So that** the database stays uncluttered so that workeres aren't confused by discontinued itmes. 

**Priority** P2
**independent test:** Veriy a manager can delete an inventory record and it is completely removed from the inventory record list. 
**Acceptance scenarios:** see ### US-1.3 under Gherkin AC 

## Functional Requirements 
* The system must allow users to allow users to add new inventory records/itmes if a scanner is unavalable. 
* The system must allow users to update existing inventory amounts and locations.
* The system must allow users with manager roles to delete inventory records. 
* The system must track the quantities of each item. 



## Key Entities 
* Inventory 
* Item 
* Warehouse Location (or zone)
* Worker 



## Initial Data Model 


| --- | --- | --- |
| 'inventory_id' | Integer | Primary key for the inventory record |
| 'item_id' | Integer | Foreign Key referencing the specific item | 
| 'bin' | String | Warehouse bin location (e.g., C-N-15-2) |
| 'slot' | String | Warehouse slot location (e.g., C-N-15-1) |
| 'current_inv_qty' | Integer | The current amount of the item in stock | 
| 'min_level' | Integer | The minimum required amount for an item |
| 'max_level' | Integer | The maximum capacity amount for an item |

'''## Gherkin AC 

    ### US-1.1 

    Scenario: Worder adds new inventory
    Given a worker is is logged into the inventory management screen
    When the worker enters an item ID, a quantity of "50", and a location of "Aisle 4" 
    And clicks "Save" 
    Then the system should create a new inventory record for that item 
    And the system should show the quantity "50" at ailse 4. 

    ### US-1.2 
    Scenario: Worker updates inventory quantity** 
    Given an existing inventory record shows "50" items in Aisle 4"
    When the worker updates the  quantity to "40" 
    And clicks "Update" 
    Then the system should save the new amount 
    And the system should 
    display the quantity 40 itmes in "Ailse 4" 

    ### US-1.3 
    Scenario: Manager deletes an inventory record
    Given a manger is viewing an existing inventory record
    When the manager clicks "Delete" and confirms the prompt 
    Then the system should remove the inventory record 
    permanently
'''