# Feature 18: Pick Orders from Customers 

## User Stories 

### US-18.1: View Automated Pick Route 
**As a** warehouse worker 
**I want to** view an automatically generated pick list sorted by Aisle, Slot, and Bin in the most efficient walking path
**So that** I don't waste time walking back and forth across the warehouse.

**Priority:** P1 
**Indpendent test:** Verify the system sorts the items on the pick list sequentially by `aisle`, `slot`, and `bin` to generate the shortest walking route.
**Acceptance scenarios:** see ### US-18.1 under Gherkin AC 

### US-18.2: Update Order to Picked 
**As a** manager 
**I want to** know when a worker has finished collecting all items for an order 
**So that** I know it is ready to be packed and shipped

**Priority:** P1 
**Indpendent test:** Verify the system updates the order status to `Picked`, logs `end_time`, and decrements inventory once the worker confirms all items are collected. 
**Acceptance scenarios:** see ### US-18.2 under Gherkin AC 

## Functional Requirements 

* The system must generate a digital pick list that links each ordered item to its corresponding `StorageZone` (`aisle`, `slot`, and `bin`).
* The system must automatically sort the pick list sequentially by `aisle`, `slot`, and `bin` (ascending) to create the most efficient walking route with minimal backtracking.
* The system must allow a worker to claim an unclaimed order (returning `409 Conflict` if another worker claims it first) so two workers do not pick the same order
* The system must require the worker to scan the physical bin barcode and item barcode as they pick each item, decrementing `qty_on_hand` in inventory. 
* The system must automatically change the `CustomerOrder` status to `Picked` and record `end_time` when all items are checked off. 
* The system must record the `worker_id` of the person who picked the order. 

## Assumptions 
* Workers are using mobile barcode scanners connected to the warehouse network 
* Warehouse locations are mapped in the `StorageZone` table using the `aisle`, `slot`, and `bin` hierarchy (Feature 8).

## Edge Cases 
* **Insufficient Inventory (Short Pick):** The scanner directs the worker to a bin, but the shelf is empty or has fewer units than ordered. The worker can flag a shortage, which alerts the manager and pauses order completion.
* **Concurrent Claiming:** Two workers attemtp to claim the same `CustomerOrder` simultaneously; the database locks the claim and returns `409 Conflict` to the second worker. 
* **Wrong Bin or Item Scan:** If the worker scans a bin or SKU that does not match the directed pick step, the scanner displays an error and blocks confirmation.


## Success Criteria 

* Worker successfully follow the Aisle/Slot/Bin pick route, scan all items, and update the order status to `Picked`. 
* Pick tasks log exact `start_time` and `end_time` for performance tracking. 
* The routing algorithm succesfully groups items by `aisle`, `slot`, and `bin` order.

## Key Entities 
* **Worker / Manager** 
* **CustomerOrder**
* **Item** 
* **StorageZone** 
* **PickTask** 
* **InventoryMovement**

## Initial Data Model 

### PickTask 

| Field Name | Data Type | Constraints | Description | 
|---|---|---|
| `pick_id` | Integer | PK, Not Null | Unique identifier for this specific picking assignment | 
| `order_id`| Integer | FK, Not Null | Foreign key referencing the `CustomerOrder` being picked | 
| `worker_id` | Integer | FK, Not Null | Foreign key referencing the worker gathering the items | 
| `pick_status` | String | Not Null | The current state of the task (e.g., Not started, In Progress, Completed) | 
| `start_time` | DateTime | Not Null | The exact date and time the worker started picking |
| `end_time` | DateTime | Nullable | The exact date and time the worker finished picking | 

### InventoryMovement 

| Field Name | Data Type | Constraints | Description | 
|---|---|---|---|
| `movement_id` | Integer | PK, Not Null | Unique identifier for this specific inventory transaction | 
| `item_id` | Integer | FK, Not Null | References the specific item being moved | 
| `zone_id` | Integer | FK, Not Null | References the `StorageZone` (Aisle/Slot/Bin)the item is being picked from | 
| `worker_id` | Integer | FK, Not Null | The quantity picked off the shelf (subtracts from `qty_on_hand`)|
| `movement_date` | DateTime | Not Null | Exact timestamp of the pick movement | 

### StorageZone (Referenced from Feature 8) 
| Field Name | Data Type | Constraints | Description | 
|---|---|---|---|
| `zone_id` | Integer | PK, Not Null | Primary key for the storage location | 
| `aisle` | String | Not Null | Aisle identifier used for route sorting (e.g., `A4`) |
| `slot` | String | Not Null | Slot identifier used for route sorting (e.g., `S12`) |
| `bin` | String | Not Null | Bin identifier used for route sorting (e.g., `B2`) |




## Gherkin AC 

 ### US-18.1 
 **Scenario:** Scanner directs worker to the next closest item 
 **Given:** a worker has just scanned and collected an item for a PickTask
 **When:** they look at their screen for the next step 
 **Then:** the system displays the next closest item on the optimized route with its exact Aisle, Slot, Bin, and quantity needed (e.g.,"Go to Aisle A2 -> Slot S04 -> Bin B1 - Pick 3") 

### US-18.2 
**Scenario:** Worker finishes gathering the order 
**Given:** a worker has checked off every item on the pick list 
**When:** They tap "Complete Picking" on their scanner 
**Then:** the system should log the end time and update the CustomerOrder status to `Picked`

## Data Ownership & Isolation 

* Each user owns their assigned tasks exclusively 

| Rule | Requirement | 
| --- | --- | 
|**Read scope**| Workers see unclaimed orders plus their own active `PickTask` rows (`worker_id = req.user.id`); Managers can view all tasks. |
|**Write scope**| Only when row matches `id` and `req.user.id` | 
|**Create scope**| New rows owned by the authenticated user | 
|**Cross-user-access**| Another user's resource -> `404` (not `403`)|
|**UI scope**| UI shows only data returned for the signed-in user| 
|**Implementation**| Shared helper in `app/authorization/` - do not duplicate scope in every controller |

## API Requirements 

| Method | Endpoint | Auth | Purpose |
|---|---|---|---|
|`GET`| `/api/orders/{order_id}/pick-list` | Yes | Fetches the order's items paired with `aisle`, `slot`, and `bin`, sorted in optimal walking-route order |
|`POST`| `/api/orders/{order_id}/claim` | Yes | Assigns the order to the `worker_id` and creates the `PickTask`|
| `POST` | `/api/inventory-movements` | Yes | Records the exact amount picked off the shelf (updates the stock) |

**Create request body:**
```json 
{"qty_picked" : 3}

```


## Screen Requirements
### [View: Active Pick Screen] - route name `pick-active`

* Heading / purpose: Active order pick Route 
* Primary action: **Confirm Pick**(`oc-cta`) and **Complete Picking** once all items are scanned
* Fields / UI: Directed Location Banner (`Aisle * Slot * Bin`), Ordered Pick List, Bin & Item Barcode Scan Input, Quantity Input, and **Report Shortage** button
* **Empty state:** "No items left to pick." 
* Loading / error: skeleton/progress; `<v-alert type = "error" >` for conflicts

## Test Coverage Map 

| Story | Scenario | Test file | Test name | 
|-------|----------|-----------|-----------|
| US-18.1 | Scanner directs worker to the next closest item | `backend/tests/Picking.test.js` |  `User creates a new list` | 
| US-18.2 | Worker finishes gathering the order | `backend/tests/picking.test.js` | `Worker finishes gathering order` |

## Agent Implementation request 

```text
Implement Feature 18 from @features/feature-18-pick-orders-from-customers.md on branch `feature/18-pick-orders-from-stores`. 

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests).
Map every Gherkin scenario in the Test Coverage Map; run `npm test` before finishing.
If API routes, payloads, schema, or product rules changed per this spec update @features/reference/api.md, @features/reference/data-model.md, and/or @features/reference/behavior.md in the same PR to match the code. 
```
## Definition of Done 

* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met 
* [] All mapped tests pass (`npm test`)
* [] Test Coverage Map complete 
* [] `features/reference/data-model.md` updated (if schema changed)
* [] `features/reference/api.md` updated (if API changed)
* [] `features/reference/behavior.md` updated (if product rules changed)

## Out of Scope 
* Physical robotic picking automation
* Packing and shipping logic (handled in Feature 19)
