# Feature 2: Supplier Operations 

## User Stories 

### US-2.1: Create Supplier Order 
**As a** warehouse worker 
**I want to** create a new order to a supplier 
**so that** we can restock items that are running low in the warehouse 

**Priority:** P1 
**Independent test:** Verify a manager can submit a new supplier order and it appears in the system with a "Pending" status. 
**Acceptance scenarios:** see ### US-2.1 under Gherkin AC

## Functional Requirements 
* The system must allow managers to select a supplier and add specific items and amounts to a new order. 
* The system must track the status of supplier order (Pending, Received, Put Away). 
* The system must automatically increase inventory amounts when a worker marks an order as "Put Away" 

## Key Entities 
* Supplier 
* Supplier Order 
* Worker 
* Inventory 

## Initial Data Model 
* **Supplier** 
* 'supplier_id' (Primary Key)
* 'name' (String)
* 'address' (String)
* 'terms' (String)
* **SupplierOrder** 
* 'order_id' (Primary Key)
* 'supplier_id' (Foregin Key referencing Supplier)
* 'po_number' (String)
* 'order_date' (Date)
* 'authorized_by' (String)
* 'status' (String - Pending, Received, Put Away)
* **SupplierOrderItem** 
* 'order_item_id' (Primary Key)
* 'order_id' (Foreign Key)
* 'sku' (String)
* 'cases' (Integer)
* 'price' (Decimal)

## Gherkin AC 

### US-2.1 
**Scenario: Worker receives a supplier order via scanner** 
**Given** the worker is logged into the scanner
**When** the worker scans the order's PO number
**And** the system displays the expected items
**Then** the worker can begin scanning the product UPC's to mark them as received

## US-2.2 
**Scenario: Worker puts away items using location codes** 
**Given** the worker has a cart of received items 
**When** the worker scans their badge to attatch the product to themselves
**And** scans the product UPC
**And** scans the primary location code on the bin. 
**Then** the system saves the product's new location in the database. 

## US-2.3 
**Scenario: Worker puts away items** 
**Given** a supplier order exists with a "Recived" status 
**When** a worker clicks "Put Away" and confirms the storing locations
**Then** the system should update the order status to "Put Away" 
**And** the system should increase tthe inventory amounts for those items. 




