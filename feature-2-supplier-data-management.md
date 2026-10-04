# Feature 2: Supplier Data Management 


## User Stories

### US-2.1: Add New Supplier 

**As a** manager 
**I want to** add a new supplier's information into the system 
**so that** we have their contact and billing details saved when we are ready to order from them. 

**Priority:** P1 
**Independent test:** Verify a manager can submit a new supplier profile and it appears in the active supplier list 
**Acceptance scenarios:** see ### US-2.1 under Gherkin AC

### US-2.2: Update Supplier Details 
**As a** manager 
**I want to** update an existing supplier's address or delivery temrs 
**So that** our records are always accurate if they move or change their policies 

**Priority:** P2 
**Indpendent test:** Verify a manager can edit an existing supplier's details and the system saves the changes 
**Acceptance scenarios** see ### US-2.2 under Gherkin AC 

### US-2.3: Deactivate Supplier 
**As a** manager 
**I want to** remove or deactivate a supplier we no longer do business with
**So that** workers don't accidentally try to order items from them. 

**Priority:** P2 
**indpendent test:** Verify a manager can deactivate a supplier and they no longer appear as an option when creating purchase orders. 
**Acceptance scnearios** see ### US-2.3 under Gherkin AC 

## Functional Requirements

* The system must allow managers to create new supplier profiles. 
* The system must allow managers to edit existing supplier details (name, address, terms).
* The system must allow managers to delete or deactivate a supplier profile. 
* The system must assign a unique 'supplier_id' to every new supplier added to the database. 


## Key Entities

* Manager 
* Supplier 



## Initial Data Model



### Supplier

| Field Name | Data Type | Description |
| ;--- | ;--- | ;--- |
| 'supplier_id' | Integer | Primary key for the supplier | 
| 'name' | String | The name of the supplier | 
| 'address' | String | The physical or billing address of the supplier |
| 'terms' | String | The delivery terms |
| 'is_active' | Boolean | True if we currently do business with them, False if deactivated | 


'''## Gherkin AC



    ### US-2.1

    Scenario: Manager adds a new supplier
    Given: the manager is logged into the supplier management screen
    When: they enter the supplier's name, address, and terms
    And: click "Save" 
    Then: the system should generate a new 'supplier_id' and save the profile to the database.

    ### US-2.2

    Scenario: Manager updates a supplier's address
    Given: an existing supplier profile is open 
    When: the manager changes the address field to a new location 
    And: clicks "Update" 
    Then: the system should overwrite the old address and display the updated profile. 

    ### US-2.3

    Scenario: Manager deactivates a supplier 
    Given: a manager is viewing an active supplier profile 
    When: the manager clicks "Deactivate" and confirms the prompt 
    Then: the system should set 'is_active' to False 
    And: hide the supplier from the ordering screens. 
'''