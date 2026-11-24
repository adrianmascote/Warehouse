# Feature 6: Route Data Management 

## User Stories 

### US_6.1: Add New Route 
**As a** warehouse manager 
**I want to** add a new delivery route to the system. 
**So that** we can assign customer orders to this new delivery path 

**Priority:** P1
**Independent test:** Verify amanager can create a new route with a name and geogrpahic area, and it saves to the database.
**Acceptance scenarios:**  see ### US-6.1 under Gherkin AC 

### US-6.2: Update Route Details 
**As a** warehouse manager 
**I want t** update an existing route's details 
**So that** the system reflects changes in delivery zones or delivery paths. 

**Priority:** P2 
**Independent test:** Verify a manager can edit a route's designated area and the system saves the new information. 
**Acceptance scnenarios:** see ### US-6.2 under Gherkin AC 

### US-6.3: Delete Route 
**As a** warehouse manager 
**I want to** delete a route that is no longer in service 
**So that** dispatchers do not accidentally assign orders to a dead route

**Priority:** P3 
**Independent test:** verify a manager can delete a route, permanently removing it from the available route list. 
**Acceptance scenarios:** see ### US-6.3 under Gherkin AC 

## Functional Requirements 
* The system must allow managers to add new delivery routes. 
* The system must allow managers to update existing route names and areas. 
* The system must allow managers to delete routes that are no longer used. 

## Key Entities 
* Route 
* Manager 

## Initial Data Model 

### Route 

| Field Name | Data Type | Description | 
|---|---|---| 
| `route_id` | Integer | Primary key for the route | 
| `route_name` | String | The name of the route |
| `geographic_area` | String | The geographic area covered by a route |

'''## Gherkin AC 

    ### US-6.1 
    Scenario: Manager adds a new route 
    Given: a manager is on the Routes managements screen
    When: the manager enters a route name like "Northside" and an area
    And: clicks "Save"
    Then: the system should create a new route profile 
    And: the route should appear in the active routes list. 

    ### US-6.2 
    Scenario: Manager updates a route 
    Given: a route profile already exists in the system 
    When: the manager changes the geographic area details 
    And: clicks "Update" 
    Then: the system should save the new area for that route 

    ### US-6.3
    Scenario: Manager deletes a route
    Given: a route profile already exists in the system
    When: the manager clicks "Delete" on the route 
    Then: the system should remove the route from the active list 
'''