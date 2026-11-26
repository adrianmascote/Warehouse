# Feature 2: Supplier Data Management 


## User Stories

### US-2.1: Add New Supplier 

**As a** warehouse manager 
**I want to** add a new supplier's information into the system 
**so that** we have their contact, billing, and delivery details saved when we are ready to order from them. 

**Priority:** P1 
**Independent test:** Verify a manager can submit a new supplier profile with a name, address, contact email, and delivery terms, and it appears in the active supplier list. 
**Acceptance scenarios:** see ### US-2.1 under Gherkin AC

### US-2.2: Update Supplier Details 
**As a** warehouse manager 
**I want to** update an existing supplier's address, contact details, or delivery terms 
**So that** our records are always accurate if they move or change their policies 

**Priority:** P2 
**Indpendent test:** Verify a manager can edit an existing supplier's details and the system saves the changes 
**Acceptance scenarios** see ### US-2.2 under Gherkin AC 

### US-2.3: Deactivate Supplier 
**As a** warehouse manager 
**I want to** deactivate a supplier we no longer do business with
**So that** staff don't accidentally try to order items from them. 

**Priority:** P2 
**indpendent test:** Verify a manager can deactivate a supplier, setting `is_active` to `false` so they no longer appear as an option when creating purchase orders. 
**Acceptance scnearios** see ### US-2.3 under Gherkin AC 

## Functional Requirements

* The system must allow managers to create new supplier profiles with a `name`, `address`, `contact_email`, and `terms`, assigning a unique `supplier_id`
* The system must allow managers to edit existing supplier details (`name`, `address`, `contact_email`, `terms`).
* The system must allow managers to soft-delete (deactivate) a supplier profile by setting `is_active` to `false`rather than permanently deleting the record, preserving historical `SupplierOrder` and discrepancy records (Feature 10 & 14).
* Deactivated suppliers must be hidden from active Purchase Order creation screens (Feature 14).

## Assumptions 
* Standard floor workers can view supplier names during dock receiving (Feature 15), but cannot create, edit, or deactivate supplier profiles.
* Delivery/payment `terms` are stored as descriptive strings (e.g., `Net 30`, `FOB Destination` )

## Edge Cases 
* **Deactivating a Preferred Supplier:** If a deactivated supplier is currently set as an item's `preferred_supplier_id` (Feature 4), auto-generated POs (Feature 14) must skip the inactive supplier and flag an alert for the manager to assign a new preferred supplier.
* **Deactivating a Supplier with Open POs:** Existing `Sent` or `Confirmed` purchase orders can still be received at the dock (Feature 15), but no new POs can be created for that supplier.

## Success Criteria 
* Managers can create, view, update, and deactivate supplier profiles cleanly. 
* Newly added active suppliers are immediately selectable in Item Catalog *(`preferred_supplier_id`) and Purchase Order creation (Features 4 & 14).
* Deactivated suppliers cannot be selected on new purchase orders. 

## Key Entities 
* **Supplier**
* **Manager** 


## Initial Data Model



### Supplier

| Field Name | Data Type | Constraints | Description |
| --- | --- | --- | --- |
| `supplier_id` | Integer | PK, Not Null | Primary key for the supplier | 
| `name` | String | Not Null | The name of the supplier | 
| `address` | String | Not Null | The physical or billing address of the supplier |
| `contact_email` | String | Nullable | Primary contact email for the supplier | 
| `terms` | String | Not Null | Delivery and payment terms (e.g., `Net 30`) |
| `is_active` | Boolean | Not Null, Default: `true` | `true` if currently active, `false` if deactivated |


## Gherkin AC


### US-2.1

**Scenario: Manager adds a new supplier**
**Given** the manager is logged into the Supplier Management screen
**When** they enter the supplier's name, address, contact email, and terms
**And** click "Save" 
**Then** the system should generate a new `supplier_id` and save the profile to the database with `is_active` set to `true`. 

### US-2.2

**Scenario: Manager updates a supplier's address** 
**Given** an existing supplier profile is open 
**When** the manager changes the address field to a new location 
**And** clicks "Update" 
**Then** the system overwrites the old address and displays the updated profile. 

### US-2.3

**Scenario: Manager deactivates a supplier**
**Given** a manager is viewing an active supplier profile 
**When** the manager clicks "Deactivate" and confirms the prompt 
**Then** the system should set `is_active` to `false` 
**And** hides the supplier from active purchase ordering screens. ordering screens. 


## Data Ownership & Isolation 

| Rule | Requirement | 
|---|---| 
| **Read scope** | Managers can view all active and inactive suppliers. Floor workers can view supplier names strictly for receiving/dock workflows. | 
| **Write scope** | Only Managers can update or deactivate supplier profiles. | 
| **Create scope** | Only Managers can create new supplier profiles. | 
| **Cross-user-access** | Standard worker attempting POST/PATCH -> `403 Forbidden`. | 
| **UI scope** | Supplier Management Dashboard is only visible to Managers. | 
| **Implementation** | Role-based access control (RBAC) enforced via shared helper `app/authorization/`. | 

## API Requirements 
| `GET` | `/api/suppliers` | Yes (All) | Fetches supplier list (filterable by `is_active`) |
| `POST` | `/api/suppliers` | Yes (Managers) | Creates a new supplier profile | 
| `PATCH` | `/api/supplieres/{supplier_id}` | Yes (Manager) | Updates supplier name, address, contact email, or terms | 
| `PATCH` | `/api/suppliers/{supplier_id}/deactivate` | Yes (Manager) | Sets `is_active` to `false` |

## Screen Requirements 

### [View: Supplier Management Dashboard] - route name `manage-suppliers`
* Heading / purpose: Manage Suppliers 
* Primary action: **Add New Supplier** (`oc-cta`)
* Secondary actions: **Edit**, **Deactivate** 
* Fields: Supplier Name, Address, Contact email, Delivery Terms, Active/Inactive Filter
* **Empty state:** "No active suppliers found."
* Loading / error: `<v-alert type ="error>` if required fields are missing or submission fails. 

## Test Coverage Map 
| Story | Scenario | Test file | Test name | 
|---|---|---|---|
| US-2.1 | Manager adds a new supplier | `backend/tests/suppliers.test.js` | `System creates a new active supplier profile` | 
| US-2.2 | Manager updates a supplier's address | `backend/tests/suppliers.test.js` | `System updates existing supplier address` | 
| US-2.3 | Manager deactivates a supplier | `backend/tests/suppliers.test.js` | `System sets is_active to false on deactivate` | 

## Agent Implementation Request 

```text
Implement Feature 2 from @features/feature-2-suppliers.md on branch `feature/2-suppliers`.

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests). Map every Gherkin scenario in the Test Coverage Map; run `npm test` before finishing.
```

## Definition of Done 

* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met 
* [ ] All mapped tests pass (`npm test`)
* [ ] Test Coverage Map complete 
* [ ] `features/reference/data-model.md` updated (Added Supplier Table)
* [ ] `features/reference/api.md` updated (Added Supplier CRUD endpoints)

## Out of Scope 
* Automated invoice matching or payment processing 