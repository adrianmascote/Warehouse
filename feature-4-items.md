# Feature 4: Item Data Management 

## User Stories 

### US-4.1: Add New Item Profile 
**As a** warehouse manager
**I want to** add a new item to the system's main list. 
**So that** the warehouse can start receiving and tracking this new product. 

**Priority:** P1 
**Independent test:** Verify a user can create a new create a new item profile with a name and barcode number, and it saves to the database. 
**Acceptance scenarios:** see ### US-4.1 under Gherkin AC 

## US-4.2: Update Item Profile 
**As a**  warehouse manager 
**I want to** update an existing item's details such as it's name or description. 
**So that** the product catalog remains accurate even if details change over time 

**Priority:** P2 
**Independent test:** Verify that a user can edit an item's details and the system saves the updated information. 
**Acceptance scenarios:** see ### US-4.2 under Gherkin AC 

### US-4.3 : Delete Item Profile 
**As a** warehouse manager 
**I want to** delete an item from the master catalog 
**So that** we do not accidentally order or track items that we no longer sell

**Priority:** P3
**Independent test:** Verify a user can delete an item's profile and it is removed from the system completely. 
**Acceptance scenarios:** see ### US-4.3 under Gherkin AC 

## Functional Requirements 
* The system must allow authorized users to add items to the catalog. 
* The system must allow users to update item names and descriptions. 
The system must allow users to delete an item from the catalog entirely. 

## Key Entities 
* Item 
* Manager 

## Initial Data Model 
* **Item** 
* 'item_id' (Primary Key)
* 'sku' (String)
* 'item_upc' (String)
* 'case_upc' (String)
* 'description' (String)
* 'case_cost' (Decimal)
* 'price' (Decimal)

## Gherkin AC 

### US-4.1 
**Scenario: Manager adds a new item** 
**Given** the manager is on the Items management screen 
**When** the manager enters a new item name and barcode number
**And** clicks "Save" 
**Then** the system mshould create a new item in the database. 

### US-4.2 
**Scenario: Manager updates an item** 
**Given** an item already exists in the system
**When** the manager changes the item's description
**And** clicks "Update" 
**Then** the system should save the new description of that item. 

### US-4.3 
**Scenario: Manager deletes an item** 
**Given** an item exists in the system 
**When** the manager clicks "Delete" on the item's profile 
**Then** the system should remove the item completely from the database. 



