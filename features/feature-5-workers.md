# Feature 5: Worker Data Management 

## User Stories 

### US-5.1: Add New Worker 
**As a** warehouse manager
**I want to** add a new worker profile to the system 
**So that**new hires can be assigned to tasks and log into the warehouse system. 

**Priority:** P1 
**Independent test:** Verify that a manager can create a new worker pofile with a name and role, and that saves to the database. 
**Acceptance scenarios:** see ### US-5.1 under Gherkin AC 

### US-5.2: Update Worker Profile 
**As a** warehouse manager 
**I want to** update an existing worker's details or role 
**So that** the system reflects promotions, shift changes, or new job tasks. 

**Priority:** P2 
**Independent test:** Verify a manager can edit a worker's role and the system saves the new information. 
**Acceptance scenarios:** see ### US-5.2 under Gherkin AC 

### US-5.3: Deactive Worker Profile 
**As a** warehouse manager 
**I want to** deactivate a worker's profile 
**So that** former employees no longer have access to the warehouse system. 

**Priority:** P2 
**Independent test:** Verify a manager can deactivate a worker profile 
**Acceptance scenarios:** see ### US-5.3 under Gherkin AC 

## Functional Requirements 
* The system must allow managers to add new workers to the roster. 
* The system must allow managers to update a worker's details such as job role or shift. 
* The system must allow managers to deactivate workers who no longer work at the warehouse. 

## Key Entities 
* Worker 
* manager 

## Initial Data Model 

### Worker 

| Field Name | Data Type | Description | 
|---|---|---|
| `worder_id` | Integer | Primary key for the worker |
| `full_name` | String | The full name of the worker | 
| `job_role` | String | The worker's role (e.g., Picker, Receiver, Driver) |
| `status` | Boolean | True if the worker is employed, False if otherwise (e.g., Active, Inactive) |

'''## Gherkin AC 
    ### US-5.1 
    Scenario: Manager adds a new worker
    Given: a manager is on the Workers management screen 
    When: the manager enters a new worker's name and job role
    And: clicks "Save" 
    Then: the system should create a new worker profile 
    And: the worker should be saved and appear in the active roster. 

    ### US-5.2 
    Scenario: Manager updates a worker role 
    Given: a worker profile exists in the system 
    When: the manager changes the worker's role from "Picker" to "Receiver" 
    And: clicks "Update" 
    Then: the system should save the new role for that worker

    ### US-5.3
    Scenario: Manager deactivates a worker
    Given: a worker profile exists in the system 
    When: the manager clicks "Deactivate" on the worker profile 
    Then: the system should change the worker's status to "Inactive" 
    And: the worker should be removed from the active roster 
'''



