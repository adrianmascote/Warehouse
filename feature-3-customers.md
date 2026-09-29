# Feature 3: Customer Operations 

## User Stories 

### US-3.1: Receive Customer Order

**As a** warehouse system
**I want to** receive orders from customers 
**So that** the warehouse team knows exactly what items need to be fulfilled 

**Priority:** P1
**Independent test:** Verify that a submitted customer order successfully appears in the system's "New Orders" queue. 
**Acceptance scenarios** see ### US-3.1 under Gherkin AC 

### US-3.2 
**As a** warehouse worker 
**I want to** pick items from the shelves for a customer order 
**So that** the items are gathered and ready for shopping. 

**Priority:** P1 
**Independent test:** Verify a worker can mark an order as "Picked", which takes away the items from the available inventory. 
**Acceptance scenarios** see ### US-3.2 under Gherkin AC 

### US-3.3: Pick Customer Order 

**As a** warehouse worker 
**I want to** mark a picked order as shipped 
**So that** the system records that the items have left the building and are on a delivery route. 

**Priority:** P1
**Independent test:** Verify that an order's status can successfuly transition from "Picked" to "Shipped". 
**Acceptance scenarios:** see ### US-3.3 under Gherkin aC 

### US-3.4: Deliver Customer Order

**As a** delivery driver 
**I want to** mark a shipped order as delivered 
**So that** the customer and the company know that the process is 100% complete 

**Priority:** P2 
**Independent test:** Verify a shipped order can be updated to the "Delivered" status once it reaches the customer. 
**Acceptance scenarios:** see ### US-3.4 under Gherkin AC 

## Functional Requirements 
* The system must track incoming customer orders with specific item requests and amounts. 
* The system must track the status of customer orders (New, Picked, Shipped, Delivered). 
* The system must decrease the item count from the active warehouse inventory count once an order is marked as "Picked". 
* The system must allow assigning a delivery route to an order when it is to be shipped. 

## Key Entities 
* Customer 
* Customer Order 
* Worker 
* Inventory 
* Route 

## Initital Data Model 
* **Customer** 
* 'customer_id' (Primary Key)
* 'name' (String)
* ' shipping_address' (String)
**CustomerOrder** 
* 'order_id' (Primary Key)
* 'customer_id' (Foreign Key referencing Customer)
* 'po_number' (String)
* 'order_date' (Date)
* 'status' (String - New, Picked, Shipped, Delivered)
* 'route_id' (Foreign Key referencing Route)
* **CustomaerOrderItem** 
* 'order_item_id' (Primary Key)
* 'order_id' (Foreign Key)
* 'sku' (String)
* 'qty' (Integer)
* 'price' (Decimal)
* **BillOfLading**
* 'bol_id' (Primary Key)
* 'order_id' (Foreign Key)
* 'shipped_date' (Date)
* 'driver_name' (String)
* 'delivery_date' (Date)
* 'received_by' (String)

## Gherkin AC 

### US-3.1 
**Scenario: System receives a new customer order** 
**Given** the warehouse system is running 
**When** a customer enters a valid order for items 
**Then** the order should appear in the system 
**And** the order status should be set to "New" 

### US-3.2 
**Scenario: Worker picks an order using a scanner** 
**Given** a customer order exists with the "New" status 
**And** the worker is logged into the scanner 
**When** the worker scans a pick label for the order
**And** walks to the Primary location code and scans the bin
**And** scans the product's UPC
**Then** the order status should change to "Picked" 
**And** the specific item amounts shoud be subtracted from the warehouse inventory. 

### US-3.3 
**Scenario: Worker ships an order** 
**Given** a customer order exists with a "Picked" status 
**When** a worker assigns a route to an order and clicks "Ship" 
**Then** the order status should change to "Shipped" 

### US-3.4 
**Scenario: Driver delivers an order**
**Given** a customer order exists with a "Shipped status 
**When** a driver clicks "Mark as Delivered" 
**Then** the order status should be changed to "Delivered"


