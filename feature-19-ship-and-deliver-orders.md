# Feature 19: Ship and Deliver Orders to Customer 

## User Stories 

### US-19.1: Load Order onto Truck 
**As a** worker 
**I want to** scan an order as I load it onto the delivery truck 
**So that** the system knows the package has left the warehouse 

**Priority:** P1
**Independent test:** Verify the system updates the order status to 'Shipped' and assigns it to a specific delivery route when scanned at the loading dock 

### US-19.2: Confirm Delivery 
**As a** worker (Driver)
**I want to** log when I drop off the package at the customer's location 
**So that** the customer and manager know the order has been successfully fulfilled. 

**Priority:** P1 
**Independent test:** Verify the system records the delivery time and updates the order status to 'Delivered'. 

## Functional Requirements 

* The system must allow a worker to use a scanner to log an order onto a specific delivery 'Route' 
* The system must automatically generate a 'BillOfLading' tracking record when the item is loaded. 
* The system must allow a driver to pull up their route o a mobile device and mark specific orders as 'Delivered'. 
* The system must log the exact date and time for the final delivery 

## Key Entities 
* Worker (Driver)
* CustomerOrder 
* Route 
* BillOfLading 

## Initial Data Model 

### BillOfLading 

| Field Name | Data Type | Description | 
|---|---|---|
| 'bol_id' | Integer | Unique identifier for this shipping record | 
| 'order_id' | Integer | Foreign key referencing the customer order being shipped |
| 'route_id' | Integer | Foreign key referencing the driver assigned to the delivery route |
| 'shipped_date' | DateTime | The exact date and time the order was loaded onto the truck |
| 'delivery_date' | DateTime | The exact date and time it was dropped off to the customer | 
| 'delivery_status' | String | Current State (e.g., In Transit, Delivered, Failed Delivery) |

'''Gherkin AC 

    ### US-19.1 
    Scenario: Worker loads an order for shipping 
    Given: A packed order is waiting at the loading dock 
    When: a worker scans the package barcode and assigns it to the delivery truck 
    Then: the system should create a BillOflading record and update the order status to 'Shipped' 

    ### US-19.2 
    Scenario: Driver drops off the package 
    Given: the driver has arrived at the customer's address 
    When: they tap "Confirm Delivery" on their scanner or mobile device 
    Then: the system should log the current time as 'delivered_date' and update the status to 'Delivered'
'''