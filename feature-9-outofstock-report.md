# Feature 9: Out-of-Stock & Low Inventory Reporting 

## User Stories 

### US-9.1: View Out-of-Stock Items 
**As a** warehouse manager 
**I want to** generate a report of all items with an inventory count of zero. 
**So that** I know exactly which products we can no longer fulfill for customer orders. 

**Priority:** P1
**Independent test:** Verify the system analyzes the inventory database and returns all items that are out of stock. 
**Acceptance scenarios:** see ### US-9.1 under Gherkin AC 

### US-9.2: View Low Stock Items 
**As a** warehouse manager 
**I want to** view a report of items that have fallen bellow their minimum stock levels. 
**So that** I can reorder products before they completely run out. 

**Priority:** P1 
**Independent test:** Ensure the system identifies the items where the current inventory is below the 'min_level' threshold. 
**Acceptance scenarios:** see ### US-9.2 under Gherkin AC

### US-9.3: Export Inventory Report 
**As a** warehouse manager 
**I want to** export the out-of-stock and low-stock reports 
**So that** I can send the data to the finance department to create supplier orders. 

**Priority:** P2 
**Independent test:** Verify the manager can download the generated report as PDF file. 
**Acceptance scenarios:** see ### US-9.3 under Gherkin AC 

## Functional Requirements 
* The system must allow managers to filter inventory records where 'current_inv_qty' is 0. 
* The system must allow managers to filter inventory records where 'current_inv_qty' is less than or equal to the 'min_level'
* The report must display the Item SKU, Description, current quantity, minimum level, and bin location. 
* The system must provide an export function for the generated reports.  

## Key Entities 
* Manager 
* InventoryRecord 
* Item 

## Initial Data Model 

* **InventoryRecord** (Referenced for 'current_inv_qty', 'min_level', 'bin', 'slot')
* **Item** (Referenced for 'sku', 'description')

## Gherkin AC 

### US-9.1 
**Scenario: Manager runs the Out-of-Stock report** 
**Given** the manager is logged into the reporting screen
**When** the manager clicks "Generate Out-of-Stock Report" 
**Then** the system should display a list of all items with an inventory count of 0 

### US-9.2 
**Scenario: Manager runs the Low Stock report** 
**Given** the manager is on the reporting screen
**When** the manager clicks "Generate Low Stock Report" 
**Then** the system should display all items where the inventory count at or below the minimum threshold. 

### US-9.3 
**Scenario: Manager exports the inventory report** 
**Given** an inventory report is displayed on the screen
**When** the manager clicks "Export as Spreadsheet" 
**Then** the system should download a file containing the data of the report. 