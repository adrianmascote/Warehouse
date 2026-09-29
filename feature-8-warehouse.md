# Feature 8: Warehouse Data Management 

## User Stories 

### US-8.1: Add New Warehouse Location 

**As a** warehouse manager 
**I want to** add a new storage zone to the system 
**So that** we can start assigning invenotry and staff to the new Ailses and shelves. 

**Priority:** P1 
**Independent test:** Verify an admin can create a new storage zone with a ame and type, and it saves to the system.  
**Acceptance scenarios:** see ### US-8.1 under Gherkin AC 

### US-8.2: Update Warehouse Details 
**As a** warehouse manager 
**I want to** update an existing storage zone's details
**So that** the system reflects changes accurately if an aisle is renamed or changed locations. 

**Priority:** P2 
**Independent test:** Verify a manager can edit a zone's name or location and the system saves the update. 
**Acceptance scenarios:** see ### US-8.2 under Gherkin AC 

### US-8.3: Delete Warehouse Location 
**As a** warehouse manager
**I want to** delete a storage zone from the system 
**So that** workers don't accidentally assign inventory to an ailse that doesne't exist. 

**Priority:** P3
**Independent test:** Verify a maanger can permanently remove a deleted zone from the active locations list. 
**Acceptance scenarios:** see ### US-8.3 under Gherkin AC 

## Functional Requirements 
* The system must allow managers to add new storage zones or aisles to the database. 
* The system must allow managers to update existing zone names and locations 
* The system must allow managers to delete a storage zone when it is dismantled. 

## Key Entities 
* Storage Zone 
* Manager 

## Initial Data Model 
* **StorageZone** 
* 'zone_id' (Primary Key)
* 'zone_name' (String - "Ailse 4, "BinB")
* 'zone_type' (String - e.g., "Standard", "Refrigerated")

## Gherkin AC 

### US-8.1 
**Scenario: Manager adds a new storage zone** 
**Given** a manager is on the Zone management screen
**When** the manager enters a new zone name like "Ailse 7" 
**And** clicks "Save" 
**Then** the system should create a new storage zone in the database. 

### US-8.2 
**Scenario: Manager updates a storage zone** 
**Given** a storage zone profile already exists 
**When** the manager updates the zone type to "Refrigerated" 
**And** clicks "Update" 
**Then** the system should save the enw type for that zone 

### US-8.3 
**Scenario: Maager deletes a storage zone** 
**Given** a storage zone exists in the system 
**When** the manager clicks "Delete" on the zone record 
**Then** the system should remove the zone from the active database. 