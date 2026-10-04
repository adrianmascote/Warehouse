# Feature 3: Customer Operations 

## User Stories 

### US-3.1: Add New Customer 

**As a** manager 
**I want to** add a new customer's information into the system 
**So that** we have their shipping address and contact details saved for future orders. 

**Priority:** P1
**Indpendent test** Verify a manager can submit a new customer profile and it appears in the active customer list 
**Acceptance criteria** see ### US-3.1 under Gherkin AC 

### US-3.2: Update Customer Details 

**As a** manager 
**I want to** update an existing customer's shipping address 
**So that** our delivery drivers are always sent to the correct location. 

**Priority:** P1 
**Independent test:** Verify a manager can edit an existing customer's address and the system saves the changes. 
**Acceptance criteria** See ### US-3.2 under Gherkin AC 

### US-3.3: Deactivate Customer Profile 

**As a** manager 
**I want to** remove or deactivate a customer account 
**So that** we do not accidentally process orders for inactive accounts. 

**Priority:** P1 
**Independent test:** Verify a manager can edit an existing customer's address and the system saves the changes.
**Acceptance criteria** see ### US-3.3 under Gherkin AC 

## Functional Requirements 

* The system must allow users with manager roles to create new customer profiles 
* The system must allow managers to edit existing customer details (name, shipping address).
* The system must allow managers to delete or deactivate a customer profile. 
* The system must assign a unique 'customer_id' to every new customer added to the database.


## Key Entities 
* Manager 
* Customer 

## Initital Data Model 

### Customer 

| Field Name | Data Type | Description | 
| --- | --- | --- | 
| 'customer_id' | Integer | Primary Key for the customer |
| 'name' | String | The name of the customer | 
| 'shipping_address' | String | The delivery address for the customer |
| 'is_active' | Boolean | True if the customer is active, False if deactivated | 




'''## Gherkin AC 

    ### US-3.1 
    
    Scenario: Manager adds a new customer
    Given: the manager is logged into the customer management screen
    When: they enter the customer's name and shipping address
    And: click "Save" 
    Then: the system should generate a new 'customer_id' and save the profile to the database. 

    ### US-3.2 

    Scenario: Manager updates a customer's address 
    Given: an existing customer profile is open 
    When: the manager changes the shipping address field to a new location 
    And: clicks "Update" 
    Then: the system should overwrite the old address and display the updated profile. 

    ### US-3.3 

    Scenario: Manager deactivates a customer 
    Given: a manager is viewing an active customer profile 
    When: the manager clicks "Deactivate" and confirms the prompt 
    Then: the system should set 'is_active' to False 
    And: hide the customer from the active ordering screens
'''

