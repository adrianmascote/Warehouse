# Feature 18: Pick Orders from Customers 

## User Stories 

### US-18.1: View Automated Pick Route 
**As a** warehouse worker 
**I want to** view an automatically generated pick list sorted by the most efficient picking path. 
**So that** I don't waste time walking back and forth across the warehouse 

**Priority:** P1 
**Indpendent test:** Verify the system sorts the items on the pick list based on the storage zones to make the most efficient walking route. 

### US-18.2: Update Order to Picked 
**As a** manager 
**I want to** know when a worker has finished collecting all items for an order 
**So that** I know it is ready to be packed and shipped

**Priority:** P1 
**Indpendent test:** Verify the system updates the order status to `Picked` once the worker confirms that all items are collected.

## Functional Requirements 

* The system must make a digital pick list that links the ordered items to their corresponding `zone_id` 
* The system must automatically sort the digital pick list to make the most efficient walking route through the warehouse locations. 
* The system must allow a worker to claim an order so that two workers do not pick the same order. 
* The system must allow the worker to check off or scan items with their device as they pick the items
* The system must automaticall change the order 'status' to `Picked` when all items are checked off. 
* The system must record the `worker_id` of the person who picked the order. 

## Assumptions 
* Workers are using mobile barcode scanners connected to the warehouse network 
* Storage zones have bene mapped with indexes. 

## Edge Cases 
* **Insufficient Inventory:** The scanner directs the worker to pick an item, but the shelf is empty. 
* **Concurrent Claiming:** Two workers attemtp to claim the same StoreOrder simultaneously. 

## Success Criteria 

* Worker successfully scans all items and the order status updates to `Picked`.
* Pick tasks log exact `start_time` and `end_time` for performance tracking. 
* The routing algorithm succesfully groups items by `zone_id` proximity. 

## Key Entities 
* Worker / Manager 
* StoreOrder
* Item 
* StorageZone 
* PickTask 

## Initial Data Model 

### PickTask 

| Field Name | Data Type | Description | 
|---|---|---|
| `pick_id` | Integer | Unique identifier for this specific picking assignment | 
| `order_id`| Integer | Foreign key referencing the Store order being picked | 
| `worker_id` | Integer | Foreign key referencing the worker gathering the items | 
| `pick_status` | String | The current state of the task (e.g., Not started, In Progress, Completed) | 
| `start_time` | DateTime | The exact date and time the worker started picking |
| `end_time` | DateTime | The exact date and time the worker finished picking | 

## Gherkin AC 

 ### US-18.1 
 **Scenario:** Scanner directs worker to the next closest item 
 **Given:** a worker has just scanned and collected an item for a PickTask
 **When:** they look at their screen for the next step 
 **Then:** the system should display the exact storage zone and amount needed for the next closest item on the optimized route 

### US-18.2 
**Scenario:** Worker finishes gathering the order 
**Given:** a worker has checked off every item on the pick list 
**When:** They tap "Complete Picking" on their scanner 
**Then:** the system should log the end time and update the CustomerOrder status to `Picked`

## Data Ownership & Isolation 

* Each user owns their assigned tasks exclusively 

| Rule | Requirement | 
| --- | --- | 
|**Read scope**| Workers only see rows where `worker_id = req.user.id` |
|**Write scope**| Only when row matches `id` and `req.user.id` | 
|**Create scope**| New rows owned by the authenticated user | 
|**Cross-user-access**| Another user's resource -> `404` (not `403`)|
|**UI scope**| UI shows only data returned for the signed-in user| 
|**Implementation**| Shared helper in `app/authorization/` - do not duplicate scope in every controller |

## API Requirements 

| Method | Endpoint | Auth | Purpose |
|---|---|---|---|
|`GET`| `/api/orders/{order_id}/`claim` | Yes | Assins the optimized list of items to pick |
|`POST`| `/api/orders/{order_id}/`claim` | Yes | Assigns the order to the `worker_id` |
| `PATCH` | `/api/orders/items/{order_item_id} | Yes | Updates picked amount |

**Create request body:**
```json 
{"qty_picked" : 3}

```


## Screen Requirements
### [View: Active Pick Screen] - route name `pick-active`

* Heading / purpose: Active order pick 
* Primary action: **Confirm Pick**(`oc-cta` when it is primary labeled CTA)
* Fields, dialogs, icon actions with `aria-label's
* **Empty state:** "No items left to pick." 
* Loading / error: skeleton/progress; `<v-alert type = "error" > for failures 

## Test Coverage Map 

| Story | Scenario | Test file | Test name | 
|-------|----------|-----------|-----------|
| US-18.1 | Scanner directs worker to the next closest item | `backend/tests/lists.test.js` |  `User creates a new list` | 
| US-18.2 | Worker finishes gathering the order | `backend/tests/picking.test.js` | `Worker finishes gathering order` |

## Agent Implementation request 

```text
Implement Feature 18 from @features/feature-18-pick-orders-from-stores.md on branch `feature/18-pick-orders-from-stores`. 

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests).
Map every Gherkin scenario in the Test Coverage Mape; run `npm test` before finishing.
If API routes, payloads, schema, or product rules changed per this spec update @features/reference/api.md, @features/reference/data-model.md, and/or @features/reference/behavior.md in the same PR to match the code. 

* **Features** 