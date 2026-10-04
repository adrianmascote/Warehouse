# Feature 12: Delivery Mismatch Reporting 

## User Stories 

### US-12.1: Identify Short_Shipped Orders 
**As a** warehouse manager 
**I want to** view a report comparing the items a customer ordered agaisnt what was actually packed on the bill of lading. 
**So that** I can see if the warehouse failed to complete the order accurately. 

**Priority:** P1
**Indpendent test:** Verify the system highlights orders where the shipped item amount is less than the amount requested by the customer. 
**So that** I can investgate goods that may have been dmaged or lost on the delivery route. 

**Priority:** P2
**Indpendent test:** Verify the system compares the shipped amounts on the Bill of Lading against the final delivery confirmation amount. 
**Acceptance scenarios:** see ### US-12.2 under Gherkin AC 

## Functional Requirements

* The system must compare the original amount requested on a 'CustomerOrder' with the amount shipped on the 'BillOfLading' 
* The system must track discrepancies between the shipped inventory and the final items signed for by the customer. 

* The report must display the Customer Name, Order Number, expected quantities, and the actual delivered quantities. 

## Key Entities 
* manager 
* Customer Order
* Bill of Lading 

## Initial Data Model 

### CustomerOrder (Referenced)

| Field Name | Data Type | Description | 
|---|---|---| 
| 'qty_requested' | Integer | The original item quantities requested by the customer | 

### BillOfLading (Referenced)

| Field Name | Data Type | Description |
|---|---|---|
| 'qty_shipped' | Integer | The amount of times actually loaded and shipped |
| 'qty_delivered' | Integer | The final amount of items signed for and delivered | 

'''## Gherkin AC 

    ### US-12.1 
    Scenario: Manager views incomplete shipments 
    Given: the manager is on the reporting screen
    When: the manager clicks "Generate Delivery Mismatch Report" 
    Then: the system should display a list of customer orders where the warehouse shipped either fewer or more items than requested. 

    ### US-12.2 
    Scenario: Manager investigates route losses** 
    Given: the Delivery Mismatch Report is generated
    When: the manager filters by "Driver Issues" 
    Then: the system should display orders where the customer received fewer items than what was loaded on the truck. 
'''
