# Feature 11: Shipping & Performance Reporting 

## User Stories 

### US-11.1: View Average Fulfillment Time 
**As a** warehouse manager 
**I want to** view the average time it takes an order to go from "New" to "Shipped" 
**So that** I can measure the overall speed and efficiency for the picking and packing process. 

**Priority:** P1
**Independent test:** Verify the system can calculate the time difference between an order's 'order_date' and its 'shipped_date'
**Acceptance scenarios:** see ### US-11.1 under Gherkin AC 

### US-11.2: View Shipping Quantiles 
**As a** warehouse manager
**I want to** see our shipping times grouped into percentage brackets (e.g., top 25%, median, lower 25%, etc.)
**So that** I can identify if a specific chunk of orders is getting delayed and investigate the cause. 

**Priority:** P1
**Indpendent test:** Verify the system can sort shipped orders by fulfillment speed and group them into percentage brackets. 
**Acceptance scenarios:** see ### US-11.2 under Gherkin AC 

## Functional Requirements 
* The system must calculate the time between the `order_date` and `shipped_date`
* The system must group fulfillment times into percentage brackets (quantiles) to show performance 
* The system must display what percentage of orders were shipped within 24 hours, 48 hours, and 72+ hours. 

## Assumptions 

* All database timestamps are stored in standard UTC to prevent timezone calculation errors. 
* There is enough historical order data in the system to calculate accurate percentiles 

## Edge Cases 
* **Canceled Orders:** Orders that were cancelled before shipping must be excluded from the math so they don't influence the averages 
* **Same-Day Anomalies:** Orders where `order_date` and `shipped_date` are identical down to the second (likely a system error or manual override) should be flagged or exluded. 

## Success Criteria 
* The dashboard successfully renders the average fulfillment time and percentiles. 
* The math accurately reflects the data in the database. 
* The report generates in under 3 seconds even with thousands of order records. 

## Key Entities 
* Manager
* CustomerOrder
* BillOfLading 

### CustomerOrder (Referenced)

| Field Name | Data Type | Constraints | Description | 
|---|---|---|
| `order_id` | Integer | Primary key for the customer order | 
| `order_date` | DateTime | Not Null | The date and time the order was created | 
| `status` | String | Not Null | The current order status | 

### BillOfLading (Referenced)

| Field Name | Data Type | Description | 
|---|---|---|
| `bol_id` | Integer | PK, Not Null | Primary key for the shipping record | 
| `order_id'` | Integer | Foreign key referencing the associated CustomerOrder |
| `shipped_date` | DateTime | Not Null | The date and time the order was shipped | 

## Gherkin AC 

### US-11.1 
**Scenario: Manager checks average fulfillment speed** 
**Given** the manager is on the reporting screen 
**When** the manager clicks "Generate Speed Report" 
**Then** the system should display the average time taken to ship an order 

### US-11.2 
**Scenario: Manager views shipping brackets** 
**Given** the speed report is generated
**When** the manager selects "View Brackets" 
**Then** the system should group the orders to show what eprcentage shipped in under 24, 48, and over 72 hours. 


## Data Ownership & Isolation 
| Rule | Requirement | 
|---|---|
| **Read scope** | Managers can view aggregated data for all completed orders. Standard workers cannot view this report. | 
| **Write scope** | None (Reporting features are Read-Only). |
| **Create scope** | None. | 
| **Cross-user-access** | Standard worker attempting to view `403 Access Denied`. | 
| **UI scope** | Report menu is only visible to the authenticated Manager roles |
| **Implementation** | Role-based access control (RBAC) enforced through shared helper in `app/authorization/`. | 

## API Requirements 
| Method | Endpoint | Auth | Purpose | 
|---|---|---|---|
| `GET` | `/api/reports/shipping-performance` | Yes (Manager) | Fetches the calculated averages and brackets for the UI | 

## Screen Requirements 

### [View: Shipping Performance Dashboard] - route name `shipping-reports`
* Heading / purpose: Shipping Speed & Performance
* Primary action: **Generate Report** (`oc-cta`)
* Fields / Displays: Average Time, Percentage Bracket Bar Chart (24h, 48h, 72h+).
* **Empty state:** "Not enough shipping data to generate report." 
* Loading: Skeleton loader while calculations run. 

## Test Coverage Map 

| Story | Scenario | Test file | Test name | 
|---|---|---|---|
| US-11.1 | Manager checks average fulfillment speed | `backend/tests/reporting.test.js` | `System calculates accurate shipping average` | 
| US-11.2 | Manager views shipping brackets | `backend/tests/reporting.test.js` | `System groups orders into accurate time brackets` | 

## Agent Implementation Request 

```text
Implement Feature 11 from @features/feature-11-shipping-percentiles.md on branch `feature/11-shipping-percentiles`. 

Follow Layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests). Map every Gherkin scenario in the Test Coverage Map; run `npm test` before finishing. 
Ensure the SQL databases exclude canceled orders from the averages. 
```

## Definition of Done 
* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met 
* [ ] All mapped test pass (`npm test`)
* [ ] Test Coverage Map complete 
* [ ] `features/reference/api.md` updated (if API changed)

## Out of Scope 

* Real-time estimates for future unshipped orers 
* Driver specific delivery speed profiles (handled in a separate report)
