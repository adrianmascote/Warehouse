# Feature 10: Supplier Issues & Discrepacy Reporting 

## User Stories 

### US 10-1: Identify short-shipped orders 
**As a** warehouse manager 
**I want to** view a report of supplier orders where the received quantity does not match the ordered quantity. 
**So that** I know which suppliers are failing to deliver what we paid for. 

**Priority:** P1 
**Independent tetst:** Verify the system can compare the original ordered item quantity to the quantity received. 
**Acceptance scenarios:** see ### US-10.1 under Gherkin AC 

### US-10.2: View Specific Missing Items 
**As a** warehouse manager 
**I want to** see exactly which SKUs and how many cases were missing from a flagged delivery 
**So that** I can contact the supplier to request a refund or an additional shipment. 

**Priority:** P1 
**Independent test:** Verify the discrepancy report contains the Item's SKU, description, amount ordered, and the amount received. 
**Acceptance scenarios:** see ### US-10.2 under Gherkin AC 

## Functional Requirements 
* The system must compare the original 'cases' ordered on a purchase order with the amount actually scanned in by the receiving worker. 
* The system must generate a report that flags any purchase orders that have a status of "Received" but have a mismatch in the item amounts. 
* The report must display the Supplier Name, PO Number, Item SKU, Ordered quantity, and received quantity. 

## Key Entities 
* Manager 
* Supplier 
* Supplier Order 
* Item 

## Initial Data Model 
* **SupplierOrder** (Referenced for 'po_number', 'supplier_id', 'status')
* **SupplierOrderItem** (Referenced for 'sku', expected 'cases')
* **Supplier** (Referenced for 'name')
* **Item** (Referenced for 'description')

## Gherkin AC 

### US-10.1 
**Scenario: Manager views orders with missing items** 
**Given** the manager is on the reporting screen
**When** the manager clicks "Generate Supplier Discrepancy Report" 
**Then** the system should display a list of all recent Purchase Orders where the received case count was lower than the ordered case count

### US-10.2 
**Scenario: Manager investigates a specific supplier issue** 
**Given** the supplier Discrepancy Report is generated 
**when** the manager clicks on a specific flagged PO number 
**Then** the system should display the Item SKUs that were missing and the difference in case amounts. 

