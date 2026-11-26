# Feature 3: Customer Data Management  

## User Stories 

### US-3.1: Add New Customer 

**As a** warehouse manager 
**I want to** add a new customer's information into the system
**So that** we have their shipping address and contact details saved for future orders.

**Priority:** P1
**Indpendent test** Verify a manager can submit a new customer profile with a name, shipping address, and contact email, and it appears in the active customer list. 
**Acceptance criteria** see ### US-3.1 under Gherkin AC 

### US-3.2: Update Customer Details 

**As a** warehouse manager
**I want to** update an existing customer's shipping address or contact details
**So that** our delivery drivers are always sent to the correct location

**Priority:** P1 
**Independent test:** Verify a manager can edit an existing customer's address and the system saves the changes. 
**Acceptance criteria** See ### US-3.2 under Gherkin AC 

### US-3.3: Deactivate Customer Profile 

**As a** warehouse manager 
**I want to** deactivate a customer account
**So that** we do not accidentally process orders for inactive accounts. 

**Priority:** P2 
**Independent test:** Verify a manager can deactivate a customer profile, setting `is_active` to `false` and hiding them from active order creation. 
**Acceptance criteria** see ### US-3.3 under Gherkin AC 

## Functional Requirements 

* The system must allow managers to create new customer profiles with a `name`, `shipping_address`, and `contact_email`, assigning a unique `customer_id`.
* The system must allow managers to edit existing customer details (`name`, `shipping_address`, `contact_email`).
* The system must allow managers to soft-delete (deactivate) a customer profile by setting `is_active` to `false` rather than permanently deleting the record, preserving historical `CustomerOrder` records
* Deactivated customers must be hidden from active ordering screens (Feature 17).

## Assumptions 
* Standard floor workers and drivers can view customer shipping addresses during fulfillment/delivery, but cannot create, edit, or deactivate customer profiles. 
* Every customer record requires a valid `shipping_address` before orders can be shipped. 

## Edge Cases 
* **Deactivating a Customer with Open Orders:** If a customer currently has `Pending`, `In Progress`, or `In Transit` orders, the system still allows existing orders to finish delivering, but blocks any new orders from being created for that customer. 
* **Duplicate Customer Name:** Warn the manager if a customer with the exact same name and shipping address already exists to prevent accidental duplicates.

## Success Criteria 
* Managers can create, view, update, and deactivate customer profiles cleanly.
* Updated shipping adresses immediately reflect on new `CustomerOrder` and `BillOfLading`records (Feature 17 & 19).
* Deactivated customers cannot be selected when creating new customer orders. 


## Key Entities 
* **Manager** 
* **Customer** 

## Initital Data Model 

### Customer 

| Field Name | Data Type | Constraints | Description | 
| --- | --- | --- | ---|
| `customer_id` | Integer | PK, Not Null | Primary Key for the customer |
| `name` | String | Not Null | The name of the customer | 
| `shipping_address` | String | Not Null | The delivery address for the customer |
| `contact_email` | String | Nullable | Primary contact email for the customer |
| `is_active` | Boolean | Not Null, Default: `true` | `true` if the customer is active, `false` if deactivated |


## Gherkin AC 

### US-3.1 
    
**Scenario: Manager adds a new customer**
**Given** the manager is logged into the Customer Management screen
**When** they enter the customer's name, shipping address, and contact email 
**And** click "Save" 
**Then** the system generates a new `customer_id` and save the profile to the database with `is_active` set to `true`.


### US-3.2 

**Scenario: Manager updates a customer's address**
**Given** an existing customer profile is open 
**When** the manager changes the shipping address field to a new location 
**And** clicks "Update" 
**Then** the system overwrites the old address and displays the updated profile

### US-3.3 

**Scenario: Manager deactivates a customer** 
**Given** a manager is viewing an active customer profile 
**When** the manager clicks "Deactivate" and confirms the prompt 
**Then** the system sets `is_active` to `false`
**And** hides the customer from the active ordering screens

## Data Ownership & Isolation 

| Rule | Requirement | 
| **Read scope** | Managers can view all active and inactive customers. Drivers/Workers can view customer details strictly for assigned orderes. |
| **Write scope** | Only Managers can update or deactivate customer profiles. | 
| **Create scope** | Only Managers can create new customer profiles. | 
| **Cross-user-access** | Standard worker attempting POST/PATCH -> `403 Forbidden`. |
| **UI scope** | Customer Management Dashboard is only visible to Managers. | 
| **Implementation** | Role-based access control (RBAC) enforced via shared helper in `app/authorization/`.|

## API Requirements

| Method | Endpoint | Auth | Purpose |
|---|---|---|---|
| `GET` | `/api/customers` | Yes (All) | Fetches customer list (filterable by `is_active`) | 
| `POST` | `/api/customers` | Yes (Manager) | Creates a new customer profile | 
| `PATCH` | `/api/customers/{customer_id}` | Yes (Manager) | Updates customer name, shipping address, or contact email | 
| `PATCH` | `/api/customers/{customer_id}/deactivate` | Yes (Manager) | Sets `is_active` to `false` | 

## Screen Requirements 

### [View: Customer Management Dashboard] - route name `manage-customers`
* Heading / purpose: Manage Customers 
* Primary action: **Add New Customer** (`oc-cta`)
* Secondary actions: **Edit**, **Deactivate**
* Fields: Customer Name, Shipping Address, Contact Email, Active/Inactive Filter 
* **Empty state:** "No active customers found."
* Loading / error: `<v-alert type = "error">` if required fields are missing or submission fails.

## Test Coverage Map 
| Story | Scenario | Test file | Test name | 
|---|---|---|---|
| US-3.1 | Manager adds a new customer | `backend/tests/custoemres.test.js` | `System creates a new active customer profile` | 
| US-3.2 | Manager updates a customer's address | `backend/tests/customers.test.js` | `System updates existing customer shipping address.` |
| US-3.3 | Manager deactivates a customer | `backend/tests/customers.test.js` | `System sets is_active to false on deactivate` | 

## Agent Implementation Request

```text 
Implement Feature 3 from @features/feature-3-customer-data-management.md on branch `feature/3-customer-data-management`.

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests)
Map every Gherkin scenario in the Test Coverage Map; run `npm test` before finishing.
```

## Definition of Done 
* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met 
* [ ] All mapped tests pass (`npm test`)
* [ ] Test Coverage Map complete 
* [ ] `features/reference/data-model.md` updated (Added Customer table)
* [ ] `features/reference/api.md` updated (Added customer CRUD endpoints)

## Out of Scope 
* Customer self-service web portal for placing orders directly 
* Credit limit checks or automated billing/invoicing

