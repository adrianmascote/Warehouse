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

| Field Name | Data Type | Constraints | Description | 
|---|---|---|
| `bol_id` | Integer | PK, Not Null | Unique identifier for this shipping record | 
| `order_id` | Integer | FK, Not Null | Foreign key referencing the customer order being shipped |
| `route_id` | Integer | Foreign key referencing the driver assigned to the delivery route |
| `shipped_date` | DateTime | Not Null | The exact date and time the order was loaded onto the truck |
| `delivery_date` | DateTime | Nullable | The exact date and time it was dropped off to the customer (Null until delivered) | 
| `delivery_status` | String | Current State (e.g., In Transit, Delivered, Failed Delivery) |

## Data Ownership & Isolation 

| Rule | Requirement |
|---|---|
| **Read scope** | Drivers only see routes where `driver_id = req.user.id` |
| **Write scope** | Only when row matches `id` and `req.user.id` |
| **Create scope** | New BOL records owned by the system / authenticated loader | 
| **Cross-user-access** | Another user's resource -> `404` (not `403`) | 
| **UI scope** | UI shows only routes 

## API Requirements 

| Method | Endpoint | Auth | Purpose | 
|---|---|---|---|
|`POST` | `/api/shipments` | Yes | Generates the BillOfLading and sets status to `Shipped` when loaded | 
| `GET` | `/api/routes/active` | Yes | Gets the driver's specific route and pending deliveries. 
| `PATCH` | `/api/shipments/{bol_id}/deliver` | Yes | Logs the delivery_date and updates status to `Delivered` | 

## Screen Requirements 

### [View: Loading Dock Scanner] - route name `shipping-dock`
* Heading / purpose: Load Truck 
* Primary action: **Generate BOL** (`oc_cta`)
* Fields, dialogs, icon actions with labels 
* **Empty state:** "No packed orders waiting for shipment."

### [View: Driver Route] - route name `driver-route`
* Heading / purpose: Active Deliveries 
* Primary action: **Confirm Delivery** (`oc-cta`)
* **Empty state:** "All packages delivered" 

## Test Coverage Map 

| Story | Scenario | Test file | Test name | 
|---|---|---|---|
| US-19.1 | Worker loads an order for shipping | `backend/tests/shipping.test.js` | `System generates BOL on truck load` | 
| US-19.2 | Driver drops off the package | `backend/tests/delivery.test.js | `System logs delivery date and status` | 

## Agent Implementation request 
```text 
Implement Feature 19 from @features/feature-19-ship-and-deliver-orders.md on branch `feature/19-ship-and-deliver`.


Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests).
Map every Gherkin scenario in the Test Coverage Mape; run `npm test` before finishing.
If API routes, payloads, schema, or product rules changed per this spec update @features/reference/api.md, @features/reference/data-model.md, and/or @features/reference/behavior.md in the same PR to match the code. 
```


## Definition of Done 
* [ ] Backend and frontend implemented per this spec (FR-00N satisfied)
* [ ] Success Criteria (SC-00N) met 
* [ ] All mapped tests pass (`npm test`)
* [ ] Test Coverage Map complete 
* [ ] `features/reference/data-model.md` updated (if schema changed)
* [ ] `features/reference/api.md` updated (If API changed)
* [ ] `features/reference/behavior.md` updated (if product rules changed)

## Out of Scope 

* GPS tracking of the delivery truck in real-time 
* Customer signature capture on the mobile device

### US-19.1 
**Scenario: Worker loads an order for shipping** 
**Given:** A packed order is waiting at the loading dock 
**When:** a worker scans the package barcode and assigns it to the delivery truck 
**Then:** the system should create a BillOflading record and update the order status to 'Shipped' 

### US-19.2 
**Scenario: Driver drops off the package**
**Given:** the driver has arrived at the customer's address 
**When:** they tap "Confirm Delivery" on their scanner or mobile device 
**Then:** the system should log the current time as 'delivered_date' and update the status to 'Delivered'
