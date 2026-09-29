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
* The system must calculate the time between the 'order_date' and 'shipped_date'
* The system must group fulfillment times into percentage brackets to show performance 
* The system must display what percentage of orders were shipped within 24 hours, 48 hours, and 72+ hours. 

## Key Entities 
* Manager
* Customer Order
* Bill of Lading 

## Initial Data Model 
* **CustomerOrder** (Referenced for 'order_id', 'order_date', 'status')
* **BillOfLading** (Referenced for 'order_id' , 'shipped_date')

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


