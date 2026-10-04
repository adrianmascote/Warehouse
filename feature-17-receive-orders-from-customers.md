# Feature 17: Receive Orders from Customers 

## User Stories 

### US-17.1: Log New Customer Order 
**As a** manager 
**I want to** enter and save a new order requested by a customer 
**So that** the warehouse team kows exactly what needs to be packed and shipped 

**Priority:** P1 
**Indpendent test:** Verify the system generates a unique order ID, links it to the correct customer, and sets the initial status to 'New' 

### US-17.2: Verify Stock Availability 
**As a** manager 
**i want to** be warned if a customer orders an item we do not have eough of 
**So that** we do not promise inventory we cannot actually deliver. 

**Priority:** P1
**Indpendent test:** Verify the system checks the requested amount agaisnt the available inventory and flags any shortages. 

## Functional Requirements 

* The system must allow the user to select an existing 'customer_id' to attatch to attach to the order. 
* The system must allow the user to add specific items and amounts to the order 
* The system must dynamically calculate the total price of the order based on the items ordered. 
* The system must check current stock levels and flag orders that exceeds the inventory available. 
* The system must track the order 'status' (e.g., New, Picking, Shipped)

## Key Entities 
* Manager
* Customer 
* CustomerOrder
* Item 

## Intial Data Model 

### CustomerOrder

| Field Name | Data Type | Description | 
|---|---|---|
| 'order_id' | Integer | Unique identifier for the customer's order | 
| 'customer_id' | Integer | Foreign key referencing the specific customer | 
| 'order_date' | DateTime | The exact date and time the order was placed | 
| 'items_requested' | Array | A list of the specific item ID's and quantities the customer wants | 
| 'total_price' | Decimal | The calculated total cost the customer will pay | 
| 'status' | String | The current state of the order (e.g., New, Picking, Shipped) |

''' Gherkin AC 

    ### US-17.1
    Scenario: Manager creates a new customer order 
    Given: the manager is logged into the order entry screen
    When: they select a customer, add items to the cart, and click "Submit Order" 
    Then: the system should create a new CustomerOrder and set the status to 'New'. 

    ### US-17.2
    Scenario: Customer orders out-of-stock items
    Given: a manager is creating a new customer order 
    When: they enter an amount that is higher than the current available inventory 
    Then: The system should display an 'Out of Stock' warning and ask if they want to backorder the item. 
'''